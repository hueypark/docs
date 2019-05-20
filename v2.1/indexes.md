---
title: Indexes
summary: Indexes improve your database's performance by helping SQL locate data without having to look through every row of a table.
toc: true
toc_not_nested: true
---

Indexes improve your database's performance by helping SQL locate data without
having to look through every row of a table.

## How do indexes work?

When you create an index, SQL "indexes" the columns you specify, which creates
a copy of the columns and then sorts their values (without sorting the values
in the table itself), storing this information along with a pointer to each
row.

After a column is indexed, SQL can easily filter the column using the index to look up the rows of the table that match, rather than scanning each row in the entire table one-by-one. On large tables, this greatly reduces the number of rows that must be read, executing queries orders of magnitude faster.

For example, if you index an `INT` column and then filter it <code>WHERE &lt;indexed column&gt; = 10</code>, SQL can use the index to find values that match. Without an index, SQL would have to evaluate _every_ row in the table checking each to see if the value in this column is equal to 10. This is known as a "full table scan", and it can be very bad for query performance.

### Creation

Each table automatically has an index created called `primary`, which indexes either its [primary key](primary-key.html) or&mdash;if there is no primary key&mdash;a unique value for each row known as `rowid`. We recommend always defining a primary key that can be used by your application's queries. If your primary index can be used instead of a secondary index, it will provide better performance (for both reads and writes) than letting CockroachDB use `rowid`.

The `primary` index helps to filter a table by its primary key but cannot help SQL to filter the table for every possible query. To get around this limitation, build secondary indexes to improve the performance of such queries. You can create them:

- At the same time as the table with the `INDEX` clause of [`CREATE TABLE`](create-table.html#create-a-table-with-secondary-and-inverted-indexes). In addition to explicitly defined indexes, CockroachDB automatically creates secondary indexes for columns with the [`UNIQUE` constraint](unique.html).
- For existing tables with [`CREATE INDEX`](create-index.html).
- By applying the `UNIQUE` constraint to columns with [`ALTER TABLE`](alter-table.html), which automatically creates an index of the constrained columns.

To create the most useful secondary indexes, you should also check out our [best practices](#best-practices).

### Selection

Because each query can use only a single index, SQL chooses the index that it
expects to scan the fewest rows (i.e., the fastest). For more detail, check out
our blog post [Index Selection in
CockroachDB](https://www.cockroachlabs.com/blog/index-selection-cockroachdb-2/),
which shows how to use the
[`EXPLAIN`](https://www.cockroachlabs.com/docs/v19.1/explain.html) statement
for a query to see which index is being used.

To override CockroachDB's index selection, you can also force [queries to use a specific index](table-expressions.html#force-index-selection) (also known as "index hinting").

### Storage

CockroachDB stores indexes directly in your key-value store. You can find more information in our blog post [Mapping Table Data to Key-Value Storage](https://www.cockroachlabs.com/blog/sql-in-cockroachdb-mapping-table-data-to-key-value-storage/).

### Locking

Tables are not locked during index creation thanks to CockroachDB's [schema change procedure](https://www.cockroachlabs.com/blog/how-online-schema-changes-are-possible-in-cockroachdb/).

### Performance

Indexes often create a trade-off: they can greatly improve the speed of reads,
but often (and sometimes simultaneously) will slightly slow down write that
include an indexed column, because new values have to be written for both the
table _and_ the index.

Here's how this works: First an `INSERT` is the simplest case. It will always
be a bit slower for each additional index because, in addition to creating one
or more rows for the table, each index will _also_ have to create a new index
point for each new row in the table.

On the other hand, writes that require a lookup (such as `UPDATE` and `DELETE`
statements that include a `WHERE` clause) don't just write; they also contain a
lookup stage that an index can potentially make more efficient.

Note that, while both an `INSERT` and the write phase of a `DELETE` statement
will always be slowed down a bit for each additional index on a table, an
`UPDATE` statement will only require an update to indexes on columns that are
actually updated.

To maximize your indexes' performance, we recommend following a few [best practices](#best-practices).

## Best practices

We **strongly** recommend creating indexes for all of your common queries. To
design the most useful indexes, look at the `WHERE` and `FROM` clauses of each
query, and create indexes that:

- [Index all columns](#indexing-columns) in the `WHERE` clause.
- [Store columns](#storing-columns) that are _only_ in the `FROM` clause.

{{site.data.alerts.callout_success}}
For more information about how to tune CockroachDB's performance, see [SQL Performance Best Practices](performance-best-practices-overview.html) and the [Performance Tuning](performance-tuning.html) tutorial.
{{site.data.alerts.end}}

### Choosing columns to index

When designing indexes, it's important to consider which columns you use to
filter your queries and the order you list them in the index. Here are a few
guidelines to help you make the best choices:

- Each table's [primary key](primary-key.html) (which we recommend always
  [defining explicitly](create-table.html#create-a-table-primary-key-defined))
  is automatically indexed. The unique index it creates (called `primary`) can
  never be modified by an `UPDATE` statement, nor can the primary key for a
  table be changed. Choosing a primary key is a critical decision for every
  table, with long-term consequences.
- A query can only benefit from using one index (`primary` or secondary),
  typically chosen by the optimizer.
- When filtering by two or more columns in a query, efficiency of the query can
  be increased by using an index on two or more columns.
- Queries can benefit from an index if they filter a prefix of its columns,
  even if they don't filter by the rest.
- Columns filtered in the `WHERE` clause with the equality operators (`=` or
  `IN`) should come first in the index, before those referenced with inequality
  operators (`<`, `>`).
- Indexes of the same columns in different orders can produce different results for each query. For more information, see [our blog post on index selection](https://www.cockroachlabs.com/blog/index-selection-cockroachdb-2/)&mdash;specifically the section "Restricting the search space."

#### Example: Leveraging One Index with Two Queries

Suppose you want to optimize two queries.

The first filters by both `first_name` AND `last_name` :

{% include copy-clipboard.html %}
~~~ sql
> SELECT primary_phone_number
    FROM user_info
   WHERE last_name = 'Adams' AND first_name = 'Anne';
~~~

The second only filters by `last_name` :

{% include copy-clipboard.html %}
~~~ sql
> SELECT primary_phone_number FROM user_info WHERE last_name = 'Adams';
~~~

In this case, you should create only a single index that touches both columns, and starts with `last_name`:

{% include copy-clipboard.html %}
~~~ sql
> CREATE INDEX ON user_info (last_name, first_name);
~~~

You can then use this single index to service both queries.

**Note**: Ordering is important. This index **cannot** be used for the following query:

{% include copy-clipboard.html %}
~~~ sql
> SELECT primary_phone_number FROM user_info WHERE first_name = 'Anne';
~~~

This is because the query filters for a column, `first_name`, that is not a prefix of the index.

### Covered queries

In some cases, a query can be answered without touching the underlying table at all. This is called a "covered" query, and it will be performed in cases where the index has all of the information required of a query.

#### Example: Covered Query

Consider the index created by:

{% include copy-clipboard.html %}
~~~ sql
> CREATE INDEX ON user_info (last_name, first_name);
~~~

Suppose we wanted to know the list of first names for a given last name:

{% include copy-clipboard.html %}
~~~ sql
> SELECT first_name FROM user_info WHERE last_name = 'Adams';
~~~

There is no reason to look up the table rows because the index points have all the information needed to fulfill the query. Therefore, CockroachDB will not touch the table rows at all for this query.

### Storing columns for covered queries

Because covered queries are so efficient, it is sometimes useful to store a value with an index even if that is never used in a lookup. This is done with the `STORING` command.

#### Example: Stored column

If you wanted to optimize the performance of the following query:

{% include copy-clipboard.html %}
~~~ sql
> SELECT login_ip_address FROM login WHERE user_id = 'wcross';
~~~

With no index at all, this would require a table scan, and would not be performant.

A big improvement would involve creating an index on just `user_id`:

{% include copy-clipboard.html %}
~~~ sql
> CREATE INDEX ON (user_id);
~~~

This alone would make the table lookup efficient.

An better approach would be to also store the `login_ip_address` field in the index. Since we never query for this field, we will not index it, but will instead store it:

{% include copy-clipboard.html %}
~~~ sql
> CREATE INDEX ON login (user_id) STORING (login_ip);
~~~


## See also

- [Inverted Indexes](inverted-indexes.html)
- [SQL Performance Best Practices](performance-best-practices-overview.html)
- [Select from a specific index](select-clause.html#select-from-a-specific-index)
- [`CREATE INDEX`](create-index.html)
- [`DROP INDEX`](drop-index.html)
- [`RENAME INDEX`](rename-index.html)
- [`SHOW INDEX`](show-index.html)
- [SQL Statements](sql-statements.html)
