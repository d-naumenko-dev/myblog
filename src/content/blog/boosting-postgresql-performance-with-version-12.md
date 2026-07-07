---
title: "Boosting Database Performance by Upgrading to PostgreSQL 12"
description: "How upgrading to PostgreSQL 12 and rebuilding your indexes can speed up your applications with very little effort."
pubDate: "Nov 28 2019"
heroImage: "/blog-placeholder-5.jpg"
---

Every year the PostgreSQL development team ships a major release that brings new capabilities to the database. PostgreSQL 12 lets you speed up your applications without much effort. There are only two things you need to do:

1. Upgrade PostgreSQL — for example, by following [these instructions](/blog/upgrading-postgresql-11-to-12/).
2. Rebuild the existing indexes with the command:

```sql
REINDEX DATABASE CONCURRENTLY database_name;
```

In earlier versions of PostgreSQL, if your database contained large indexes, rebuilding them would have required restricting access to the corresponding tables. Fortunately, in PostgreSQL 12 you can rebuild indexes in parallel using the `CONCURRENTLY` option. That way your application won't sit idle while the rebuild is running.

## Some reasons to upgrade

- Faster queries that use B-tree indexes
- Reduced disk space used by B-tree indexes (-40% on a benchmark test!)
- Faster queries against partitioned tables
- The ability to create covering indexes on GiST indexes
- JIT compilation enabled by default
- The ability to query JSONB documents using the expressions defined in the SQL/JSON standard

…and much more. You can find the full list of changes here: <https://www.postgresql.org/docs/12/release-12.html>
