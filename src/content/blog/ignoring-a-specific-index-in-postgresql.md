---
title: "How to Make a Query Ignore a Specific Index in PostgreSQL"
description: "How to temporarily disable the use of a single index in PostgreSQL while still keeping it up to date."
pubDate: "Dec 19 2019"
heroImage: "/blog-placeholder-4.jpg"
---

Sometimes, for one reason or another, you need to temporarily turn off the use of an index — but in a way that still keeps the index up to date as the data changes. For example, you might want to see how a `SELECT` behaves without one of its indexes.

To do this, set the `indisvalid` flag to `false`:

```sql
UPDATE
    pg_index
SET
    indisvalid = FALSE
WHERE
    indexrelid = 'my_index'::regclass;
```

Here `'my_index'` is the name of the index you want to ignore. From now on, queries against the data will ignore that index — while it still continues to be updated whenever the underlying data changes.

To enable the index again, run:

```sql
UPDATE
    pg_index
SET
    indisvalid = TRUE
WHERE
    indexrelid = 'my_index'::regclass;
```

This trick comes in handy when you're deciding whether an index is worth keeping — see [Finding and Removing Unused Indexes in PostgreSQL](/blog/finding-unused-postgresql-indexes/) for more on that.
