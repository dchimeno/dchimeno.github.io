---
title: ""
date: 2025-07-24
description: "Snippets, tips and tricks about ClickHouse"
draft: false
---


## See data sizes

```sql
SELECT
    database,
    table,
    count() AS parts,
    uniqExact(partition_id) AS partition_cnt,
    sum(rows),
    formatReadableSize(sum(data_compressed_bytes) AS comp_bytes) AS comp,
    formatReadableSize(sum(data_uncompressed_bytes) AS uncomp_bytes) AS uncomp,
    uncomp_bytes / comp_bytes AS ratio
FROM system.parts
WHERE active AND table like '%name%'
GROUP BY
    table
ORDER BY comp_bytes DESC
```


## Show table definitions

```sql
select name, engine, data_paths, engine_full, partition_key, sorting_key, primary_key
from  system.tables WHERE database !='system' FORMAT Vertical;
```


## view size of columns

```sql
SELECT
    table,
    name,
    type,
    compression_codec,
    formatReadableSize(data_compressed_bytes) as comp,
    formatReadableSize(data_uncompressed_bytes) as unc
FROM system.columns
WHERE table LIKE '%raw%'
ORDER BY table ASC, data_compressed_bytes desc
```

## view data distribution

```sql
SELECT
   count(),
   * APPLY (uniq),
   * APPLY (max),
   * APPLY (min),
   * APPLY(topK(5))
FROM tablename
FORMAT Vertical
```