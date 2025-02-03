---
date:
  created: 2025-02-02
  updated: 2025-02-02
draft: false
categories:
  - blog
draft: false
comments: false
---

# Please, Order Your Data

Over the last five years, I've worked on various projects dealing with 'large' datasets in the analytics space. While I've fully embraced the benefits of ordering data, I've realized that many projects overlook this crucial aspect.

In this post, I'll summarize the benefits of ordering data in case I need to revisit them or if they prove helpful to others.

Let's start with a fictional dataset of cell tower events with values over time.

<!-- more -->

!!! warning

    The examples here use DuckDB for convenience, but the concepts apply to the dataset itself, not the tool.
    
    We are working with a relatively small data sample, but it should be enough to demonstrate the benefits.

## First Step: Know Your Data

```sql
describe select * from 's3://bucket/tower/data_unordered.parquet'
```

```
column_name         |column_type|null|key|default|extra|
--------------------+-----------+----+---+-------+-----+
eventdate           |TIMESTAMP  |YES |   |       |     |
ECGI                |VARCHAR    |YES |   |       |     |
LOCATION            |VARCHAR    |YES |   |       |     |
VENDOR              |VARCHAR    |YES |   |       |     |
VALUE_BIN_1         |DOUBLE     |YES |   |       |     |
VALUE_BIN_2         |DOUBLE     |YES |   |       |     |
VALUE_BIN_3         |DOUBLE     |YES |   |       |     |
VALUE_BIN_4         |DOUBLE     |YES |   |       |     |
VALUE_BIN_5         |DOUBLE     |YES |   |       |     |
VALUE_BIN_6         |DOUBLE     |YES |   |       |     |
VALUE_BIN_7         |DOUBLE     |YES |   |       |     |
VALUE_BIN_8         |DOUBLE     |YES |   |       |     |
VALUE_BIN_9         |DOUBLE     |YES |   |       |     |
VALUE_BIN_10        |DOUBLE     |YES |   |       |     |
...
VALUE_BIN_30        |DOUBLE     |YES |   |       |     |
```

```sql
select count(*) from 's3://bucket/tower/data.parquet'
-- 7,781,925
SELECT sum(total_compressed_size)/(1024*1024) as MiB FROM parquet_metadata('s3://bucket/tower/data_unordered.parquet');
-- 573 MiB
```

- **eventdate**: When the event occurred.
- **ECGI**: A global and public identifier for cell towers.
- **LOCATION** and **VENDOR**: Attributes of the tower, which should remain the same for the same ECGI.
- **VALUE_BIN_X**: These are the values we measure at different distances.

The source data has no inherent order.

A typical query might look like this:

```sql
select sum(VALUE_BIN_2) from 's3://bucket/tower/data_unordered.parquet'
where eventdate = '2024-02-02' and ecgi in ('XXX', 'YYYY')
-- done in 5.1s
```

The table has a wide format, meaning multiple columns represent values. This structure has some downsides:
  - We need to specify the column name in the SELECT statement.
  - Joins with this table using selected bins become more complex.

So, we transform and order the data as follows:

```sql
COPY (
  WITH unpivoted AS (
    SELECT
      eventdate,
      ECGI,
      LOCATION,
      VENDOR,
      UNNEST(RANGE(1, 31))::UTINYINT as BIN,
      UNNEST(ARRAY[
        VALUE_BIN_1, VALUE_BIN_2, VALUE_BIN_3, VALUE_BIN_4, VALUE_BIN_5,
        VALUE_BIN_6, VALUE_BIN_7, VALUE_BIN_8, VALUE_BIN_9, VALUE_BIN_10
      ])::DOUBLE as BIN_VALUE
    FROM 's3://bucket/tower/data_unordered.parquet'
  )
  SELECT * FROM unpivoted
  ORDER BY eventdate, ECGI, TIMING_ADV_SET_INDEX, BIN
)
TO 's3://bucket/tower/cells_unpivot_ordered.parquet' (FORMAT PARQUET);
```

**Note:** We could use the `UNPIVOT` function instead of `UNNEST`.

Now, let's check the transformed data:

```sql
select count(*) from 's3://bucket/tower/cells_unpivot_ordered.parquet'
-- 233,457,750
SELECT sum(total_compressed_size)/(1024*1024) as MiB FROM parquet_metadata('s3://bucket/tower/data_unordered.parquet');
-- 591 MiB
```

Querying the ordered and transformed dataset:

```sql
select sum(BIN_VALUE) from 's3://bucket/tower/cells_unpivot_ordered.parquet'
where eventdate = '2024-02-02 00:00:00' and ecgi in ('XXX', 'YYYY') and bin = 2
-- done in 2s
```

With the same storage size, we now store **30x more rows** (since there are 30 `VALUE_BIN_X` in the real dataset), and queries run **2.5x faster** than before.

### Bonus: Compression Optimization

DuckDB's default compression library is **Snappy**, but we can configure it to use **Zstandard (zstd)**, which generally provides better compression ratios while maintaining good decompression speed.

```sql
COPY
  (SELECT * FROM 's3://bucket/tower/cells_unpivot_ordered.parquet')
  TO 's3://bucket/tower/cells_unpivot.zstd.parquet'
  (FORMAT 'parquet', COMPRESSION 'zstd');
```

```sql
SELECT SUM(BIN_VALUE) FROM 's3://bucket/tower/cells_unpivot.zstd.parquet'
WHERE eventdate = '2024-04-12 00:00:00' AND ecgi IN ('214050460033012','214050460033022') AND bin=2;
-- 2.1s

SELECT sum(total_compressed_size)/(1024*1024) as MiB FROM parquet_metadata('s3://bucket/tower/cells_unpivot.zstd.parquet');
-- 248 MiB
```

### Results Summary

| State    | Rows        | Size    | Query Time (s) |
|----------|------------|---------|---------------|
| Original | 7,781,925  | 573 MiB | 5.1 s         |
| After    | 233,457,750 | 248 MiB | 2.1 s         |

### Conclusion

Ordering data significantly improves performance by optimizing storage layout and query execution. In this case, we saw **30x more rows stored** while reducing query time and size **by more than half**.

However, this is only part of the optimization story. Other important techniques include:

- **Optimizing data types**: Using more compact data types reduces memory usage and speeds up queries.
- **Custom codecs**: Choosing the right codec before compression can further improve storage efficiency. In this case, DuckDB is doing it [automatically](https://duckdb.org/2022/10/28/lightweight-compression.html)

These additional steps can further enhance data efficiency, reducing costs and improving performance even more.