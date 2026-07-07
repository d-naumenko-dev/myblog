---
title: "Finding and Removing Unused Indexes in PostgreSQL"
description: "How to track down indexes that are never used in PostgreSQL and decide whether it's safe to drop them."
pubDate: "Aug 18 2019"
heroImage: "/blog-placeholder-4.jpg"
---

Indexes are not always beneficial. Adding an index to a table speeds up reads, but it comes with a side effect: whenever the data in that table changes, the index has to be updated at the same time. In other words, every insert, update, or delete becomes slower than it was before the index existed. Essentially, you sacrifice performance in one place for a speed-up in another.

Sometimes an index gets added for one reason or another, but it is never used — and never will be. It's worth getting rid of indexes like these to free up disk space and to speed up modifications to the table. The following query will help you find them:

```sql
SELECT
    idstat.relname AS table_name,
    indexrelname AS index_name,
    idstat.idx_scan AS index_scans_count,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    tabstat.idx_scan AS table_reads_index_count,
    tabstat.seq_scan AS table_reads_seq_count,
    tabstat.seq_scan + tabstat.idx_scan AS table_reads_count,
    n_tup_upd + n_tup_ins + n_tup_del AS table_writes_count,
    pg_size_pretty(pg_relation_size(idstat.relid)) AS table_size
FROM
    pg_stat_user_indexes AS idstat
JOIN
    pg_indexes
    ON
    indexrelname = indexname
    AND
    idstat.schemaname = pg_indexes.schemaname
JOIN
    pg_stat_user_tables AS tabstat
    ON
    idstat.relid = tabstat.relid
WHERE
    indexdef !~* 'unique'
ORDER BY
    idstat.idx_scan DESC,
    pg_relation_size(indexrelid) DESC;
```

It returns a list of all indexes, sorted by the number of scans (the third column). If any index has a scan count (`index_scans_count`) of zero, it's worth thinking about removing it. But wait…

You'll often come across other versions of this query online. Some of them select only the indexes that have never been scanned, and return nothing more than the index name and the name of the table it belongs to. That's not enough.

## What to check before dropping an index

When reviewing candidates for removal, you should check the following:

- Maybe the table isn't used at all, or the planner is avoiding the index for some other reason?
- How much space does the index take up? Perhaps it isn't worth worrying about right now.
- How many sequential scans is the table getting? You may need to add extra columns to the index in order to eliminate those sequential scans.

## Resetting the statistics

It happens that an index was used at some point, but you can't be sure whether it's still being used today. In that case you need to gather the statistics from scratch. Clear the statistics in the database:

```sql
SELECT pg_stat_reset();
```

Now, if you run the index-usage query again, the table names will still be there, but all of the scan counts will be reset to zero. You can carry on working — the database will start collecting statistics again, and after a while you'll be able to identify the unused indexes once more.
