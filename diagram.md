# Diagrams

## Graphviz

This is a graphviz diagram that I found in the Ceph docs. For some reason, the diagram is not rendering on that page for me, so I copied it, and [here is a link to a rendered version][1]

```graphviz
   digraph object_store {
    size="7,7";
    node [color=lightblue2, style=filled, fontname="Serif"];

    "testrados" -> "librados"
    "testradospp" -> "librados"

    "rbd" -> "librados"

    "radostool" -> "librados"

    "radosgw-admin" -> "radosgw"

    "radosgw" -> "librados"

    "radosacl" -> "librados"

    "librados" -> "objecter"

    "ObjectCacher" -> "Filer"

    "dumpjournal" -> "Journaler"

    "Journaler" -> "Filer"

    "SyntheticClient" -> "Filer"
    "SyntheticClient" -> "objecter"

    "Filer" -> "objecter"

    "objecter" -> "OSDMap"

    "ceph-osd" -> "PG"
    "ceph-osd" -> "ObjectStore"

    "crushtool" -> "CrushWrapper"

    "OSDMap" -> "CrushWrapper"

    "OSDMapTool" -> "OSDMap"

    "PG" -> "PrimaryLogPG"
    "PG" -> "ObjectStore"
    "PG" -> "OSDMap"

    "PrimaryLogPG" -> "ObjectStore"
    "PrimaryLogPG" -> "OSDMap"

    "ObjectStore" -> "BlueStore"

    "BlueStore" -> "rocksdb"
  }
```

## Mermaid diagram

I made this diagram based on Ceph docs on [CRUSH maps](https://docs.ceph.com/en/latest/rados/operations/crush-map/)

### Static image

[![Diagram of CRUSH](https://mermaid.ink/img/pako:eNptk11v2jAUhv-K5WtAQCCIXEwCQkcLDJq0u5jpxalj4IgkjhynW4r47zNO0pWpvkn0vs_58LF9plxGgnp0H8vf_AhKkyd_lxKzJmwWPIcLMokPUqE-Ji-k3f5GprW8hix_qcipNWbMF2_IRU4gjci04CehbwmfBUUsbrV5nW4lOWiU6Y15x2ZFrmXy4ZKFlKcmwcwy39km9G-lBaure4bPdd4iAfCT-QjNOzXpW_KebWPgIhGpJlsZI8eP9irggQUiM3pVPdQKtDj8Dy3ZXEFeKEFmMsL0UJtza67YUpTtnxAXgmwBVRN6Z901C7nCTOdkLxX5chQrC_64bvPfHNCMAoUCxY9lza0tt2F-mUKC_Otk9xbaMh80EB9zrfC1-OQ_WP-x8j9tvbaX1g4am8s3oUp73qGWCg6CzPf76xRT3rS1tSFhFTLh5n7k-Iox6gZ4tMBTkzNGuLEDaz-zpsAm05jge9MVbdFEqAQwMpf4fA3ZUX00J7qjnvmNQJ12dJdeDAeFlmGZcuppVYgWLbLInKaPcFCQUG8PcW7UDFLqnekf6rmdkeP2uiNnMHbGw9Gw36Il9RzXMbrTdfujwaA36LqXFn2X0iTodsbV6o_7vZ47dNwWFRGavtfVG7NPzVb4ZQOubVz-Ao39Emc?type=png)](https://mermaid.live/edit#pako:eNptk11v2jAUhv-K5WtAQCCIXEwCQkcLDJq0u5jpxalj4IgkjhynW4r47zNO0pWpvkn0vs_58LF9plxGgnp0H8vf_AhKkyd_lxKzJmwWPIcLMokPUqE-Ji-k3f5GprW8hix_qcipNWbMF2_IRU4gjci04CehbwmfBUUsbrV5nW4lOWiU6Y15x2ZFrmXy4ZKFlKcmwcwy39km9G-lBaure4bPdd4iAfCT-QjNOzXpW_KebWPgIhGpJlsZI8eP9irggQUiM3pVPdQKtDj8Dy3ZXEFeKEFmMsL0UJtza67YUpTtnxAXgmwBVRN6Z901C7nCTOdkLxX5chQrC_64bvPfHNCMAoUCxY9lza0tt2F-mUKC_Otk9xbaMh80EB9zrfC1-OQ_WP-x8j9tvbaX1g4am8s3oUp73qGWCg6CzPf76xRT3rS1tSFhFTLh5n7k-Iox6gZ4tMBTkzNGuLEDaz-zpsAm05jge9MVbdFEqAQwMpf4fA3ZUX00J7qjnvmNQJ12dJdeDAeFlmGZcuppVYgWLbLInKaPcFCQUG8PcW7UDFLqnekf6rmdkeP2uiNnMHbGw9Gw36Il9RzXMbrTdfujwaA36LqXFn2X0iTodsbV6o_7vZ47dNwWFRGavtfVG7NPzVb4ZQOubVz-Ao39Emc)

### Mermaid code

This should render in GitHub. If not, see the static image above.

```mermaid
flowchart TD
    A[CRUSH Algorithm] --> B[CRUSH Maps]
    B --> C[Devices and Buckets]
    B --> D[Rules]
    B --> E[CRUSH Location]
    B --> F[Custom Location Hooks]
    C --> G[OSDs]
    C --> H[Buckets: Hosts, Racks, etc.]
    D --> I[Placement Policies]
    D --> J[Replication Strategies]
    D --> K[Erasure Coding]
    E --> L[Key-Value Pairs]
    F --> M[Scripts for CRUSH Location]
    L --> N[OSD Location in Hierarchy]
    M --> O[Dynamic CRUSH Location]
    I --> P[Data Distribution]
    J --> Q[Data Replication]
    K --> R[Data Recovery and Storage Efficiency]
    P --> S[Data Accessibility]
    Q --> T[Data Reliability]
    R --> U[Storage Optimization]
```

[1]: <https://dreampuf.github.io/GraphvizOnline/#%20%20%20digraph%20object_store%20%7B%0A%20%20%20%20size%3D%227%2C7%22%3B%0A%20%20%20%20node%20%5Bcolor%3Dlightblue2%2C%20style%3Dfilled%2C%20fontname%3D%22Serif%22%5D%3B%0A%0A%20%20%20%20%22testrados%22%20-%3E%20%22librados%22%0A%20%20%20%20%22testradospp%22%20-%3E%20%22librados%22%0A%0A%20%20%20%20%22rbd%22%20-%3E%20%22librados%22%0A%0A%20%20%20%20%22radostool%22%20-%3E%20%22librados%22%0A%0A%20%20%20%20%22radosgw-admin%22%20-%3E%20%22radosgw%22%0A%0A%20%20%20%20%22radosgw%22%20-%3E%20%22librados%22%0A%0A%20%20%20%20%22radosacl%22%20-%3E%20%22librados%22%0A%0A%20%20%20%20%22librados%22%20-%3E%20%22objecter%22%0A%0A%20%20%20%20%22ObjectCacher%22%20-%3E%20%22Filer%22%0A%0A%20%20%20%20%22dumpjournal%22%20-%3E%20%22Journaler%22%0A%0A%20%20%20%20%22Journaler%22%20-%3E%20%22Filer%22%0A%0A%20%20%20%20%22SyntheticClient%22%20-%3E%20%22Filer%22%0A%20%20%20%20%22SyntheticClient%22%20-%3E%20%22objecter%22%0A%0A%20%20%20%20%22Filer%22%20-%3E%20%22objecter%22%0A%0A%20%20%20%20%22objecter%22%20-%3E%20%22OSDMap%22%0A%0A%20%20%20%20%22ceph-osd%22%20-%3E%20%22PG%22%0A%20%20%20%20%22ceph-osd%22%20-%3E%20%22ObjectStore%22%0A%0A%20%20%20%20%22crushtool%22%20-%3E%20%22CrushWrapper%22%0A%0A%20%20%20%20%22OSDMap%22%20-%3E%20%22CrushWrapper%22%0A%0A%20%20%20%20%22OSDMapTool%22%20-%3E%20%22OSDMap%22%0A%0A%20%20%20%20%22PG%22%20-%3E%20%22PrimaryLogPG%22%0A%20%20%20%20%22PG%22%20-%3E%20%22ObjectStore%22%0A%20%20%20%20%22PG%22%20-%3E%20%22OSDMap%22%0A%0A%20%20%20%20%22PrimaryLogPG%22%20-%3E%20%22ObjectStore%22%0A%20%20%20%20%22PrimaryLogPG%22%20-%3E%20%22OSDMap%22%0A%0A%20%20%20%20%22ObjectStore%22%20-%3E%20%22BlueStore%22%0A%0A%20%20%20%20%22BlueStore%22%20-%3E%20%22rocksdb%22%0A%20%20%7D>
