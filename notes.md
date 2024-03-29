# Recap

## Day 1 (03/26)

- cloned the ceph repo
- read the docs
  - [dev guide](https://docs.ceph.com/en/latest/dev/developer_guide/)
  - [development-mode cluster](https://docs.ceph.com/en/latest/dev/developer_guide/essentials/#development-mode-cluster)
  - [quick guide](https://docs.ceph.com/en/latest/dev/quick_guide/)
- static analysis of the Objecter::_calc_target() function
- investigate gdb setup [here](https://docs.ceph.com/en/latest/dev/developer_guide/debugging-gdb/)
- compiled ceph following the quick guide
- compiling took more than 2 hours
- build directory is over 80GB
- worked on setting up debugger in vscode
- made launch task which attached to the ceph-osd process
- set breakpoints in the `_calc_target` function and others
- only managed to get debugger to stop on `Objecter::_scan_requests`
  - code that calls `_calc_target` in `_scan_requests` did not run
  - `_scan_requests` was running frequently though
- tried increasing logging
- found a nice diagram of ceph architecture which included the `Objecter` class [here](diagram.md) originally from [here](https://docs.ceph.com/en/latest/dev/object-store/)
  - online graphviz visual *\[[1]\]*, because the original diagram was not rendering properly in the page

the `Objecter::_calc_target()` function is a member of the `Objecter` class. It is defined in the `src/osdc/Objecter.cc` file. The function is used to calculate the target OSD for an object. The function is called by the a variety of functions in the `Objecter` class.

- `Objecter::_op_submit`
- `Objecter::handle_osd_map`
- `Objecter::_scan_requests`
- `Objecter::_linger_submit`
- `Objecter::_recalc_linger_op_target`
- `Objecter::_map_session`
- `Objecter::_calc_command_target`
- `Objecter::linger_notify`

## Day 2 (03/27)

- still having trouble getting debugger to stop in `_calc_target`
- tried setting breakpoints in other functions that call `_calc_target`
- did reads, writes, and deletes in the `rados` command line tool
- tried `rbd` commands with a block device, thinking maybe rados just wasn't hitting the right code path
- tried getting linger ops to run by running `rados` commands
  - `rados -p mypool watch myobject` + `rados -p mypool notify myobject 'hello'`
- tried running `rados` commands with `--debug-objecter=20` and `--debug-osd=20`
- tailed the logs searching for anything related to `_calc_target`

### Breakthrough

- realized that the breakpoint needs to be set in the `rados` command
- added launch task for `rados` command for a read operation
- breakpoint in `_calc_target` was hit
- came through `Objecter::_op_submit` -> `Objecter::_calc_target`
- looked through call stack and investigated variables and values
- makes sense because the original information I was given was:

```txt
We'd like you to study the code path that is exercised by a client when calculating the target OSD to send a read operation. The Objecter::_calc_target(op_target_t *t, Connection *con, bool any_change) function is used for this and can be found in src/osdc/Objecter.cc. For this exercise, you can assume:
there is no tiering in use
the target pool in question is a replicated pool
the placement group id is not precalculated
```

- clearly states that the `_calc_target` function is called by the **client** when calculating the target OSD to send a read operation
- the information really started to make sense once I had dug into ceph and familiarized myself with some of the commands
- set up a *replicated* pool
  - default is erasure coded

## Day 3 (03/28)

- dug into the `_calc_target` function more deeply
- read about CRUSH algorithm
  - [CRUSH](https://docs.ceph.com/en/latest/architecture/#crush-introduction)
- tried to figure out where CRUSH is used in the `_calc_target` function
- `lookup_pg_mapping` function in `src/osd/OSDMap.cc` is a kind of a caching mechanism for storing `up_primary` and `acting_primary` + `up` and`acting` vectors

```cpp
    std::vector<int> up; ///< set of up osds for last pg we mapped to
    std::vector<int> acting; ///< set of acting osds for last pg we mapped to
    int up_primary = -1; ///< last up_primary we mapped to
    int acting_primary = -1;  ///< last acting_primary we mapped to
```

- Objecter->osdmap is a pointer to the current OSDMap
- osdmap->pg_to_up_acting_osds called if the `lookup_pg_mapping` function returns false, which means the mapping is not cached and needs to be recalculated

```cpp
lookup_pg_mapping(const pg_t& pg, epoch_t epoch, std::vector<int> *up,
                         int *up_primary, std::vector<int> *acting,
                         int *acting_primary) {
...
    auto it = pg_mappings.find(pg.pool());
    if (it == pg_mappings.end())
      return false;
    auto& mapping_array = it->second;
    if (pg.ps() >= mapping_array.size())
      return false;
    if (mapping_array[pg.ps()].epoch != epoch) // stale
      return false;
...
}
```

- if the mapping is not cached, the `pg_to_up_acting_osds` function is called, which in turn calls `OSDMap::_pg_to_up_acting_osds`
- `OSDMap::_pg_to_up_acting_osds` is a helper function that fills in the `up` and `acting` vectors for a given `pg_t` object
- `OSDMap::_pg_to_raw_osds` is where the CRUSH algorithm is used to calculate the OSDs for the `pg_t` object

```cpp
void OSDMap::_pg_to_raw_osds(
  const pg_pool_t& pool, pg_t pg,
  vector<int> *osds,
  ps_t *ppps) const
{
  // map to osds[]
  ps_t pps = pool.raw_pg_to_pps(pg);  // placement ps
  unsigned size = pool.get_size();

  // what crush rule?
  int ruleno = pool.get_crush_rule();
  if (ruleno >= 0)
    crush->do_rule(ruleno, pps, *osds, size, osd_weight, pg.pool());

  _remove_nonexistent_osds(pool, *osds);

  if (ppps)
    *ppps = pps;
}
```

- `crush->do_rule` is where the CRUSH algorithm is called

[1]: <https://dreampuf.github.io/GraphvizOnline/#%20%20%20digraph%20object_store%20%7B%0A%20%20%20%20size%3D%227%2C7%22%3B%0A%20%20%20%20node%20%5Bcolor%3Dlightblue2%2C%20style%3Dfilled%2C%20fontname%3D%22Serif%22%5D%3B%0A%0A%20%20%20%20%22testrados%22%20-%3E%20%22librados%22%0A%20%20%20%20%22testradospp%22%20-%3E%20%22librados%22%0A%0A%20%20%20%20%22rbd%22%20-%3E%20%22librados%22%0A%0A%20%20%20%20%22radostool%22%20-%3E%20%22librados%22%0A%0A%20%20%20%20%22radosgw-admin%22%20-%3E%20%22radosgw%22%0A%0A%20%20%20%20%22radosgw%22%20-%3E%20%22librados%22%0A%0A%20%20%20%20%22radosacl%22%20-%3E%20%22librados%22%0A%0A%20%20%20%20%22librados%22%20-%3E%20%22objecter%22%0A%0A%20%20%20%20%22ObjectCacher%22%20-%3E%20%22Filer%22%0A%0A%20%20%20%20%22dumpjournal%22%20-%3E%20%22Journaler%22%0A%0A%20%20%20%20%22Journaler%22%20-%3E%20%22Filer%22%0A%0A%20%20%20%20%22SyntheticClient%22%20-%3E%20%22Filer%22%0A%20%20%20%20%22SyntheticClient%22%20-%3E%20%22objecter%22%0A%0A%20%20%20%20%22Filer%22%20-%3E%20%22objecter%22%0A%0A%20%20%20%20%22objecter%22%20-%3E%20%22OSDMap%22%0A%0A%20%20%20%20%22ceph-osd%22%20-%3E%20%22PG%22%0A%20%20%20%20%22ceph-osd%22%20-%3E%20%22ObjectStore%22%0A%0A%20%20%20%20%22crushtool%22%20-%3E%20%22CrushWrapper%22%0A%0A%20%20%20%20%22OSDMap%22%20-%3E%20%22CrushWrapper%22%0A%0A%20%20%20%20%22OSDMapTool%22%20-%3E%20%22OSDMap%22%0A%0A%20%20%20%20%22PG%22%20-%3E%20%22PrimaryLogPG%22%0A%20%20%20%20%22PG%22%20-%3E%20%22ObjectStore%22%0A%20%20%20%20%22PG%22%20-%3E%20%22OSDMap%22%0A%0A%20%20%20%20%22PrimaryLogPG%22%20-%3E%20%22ObjectStore%22%0A%20%20%20%20%22PrimaryLogPG%22%20-%3E%20%22OSDMap%22%0A%0A%20%20%20%20%22ObjectStore%22%20-%3E%20%22BlueStore%22%0A%0A%20%20%20%20%22BlueStore%22%20-%3E%20%22rocksdb%22%0A%20%20%7D>
