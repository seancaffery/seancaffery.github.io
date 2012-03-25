---
layout: post
title: Optimising merge inserts with SQLServer 2008
tags: Ruby
status: publish
type: post
published: true
---

At C3, I have to support a number of different databases, and for a number of reasons,
I use PostgreSQL for development when one of our main clients uses SQL Server 2008
(we have CI builds that test all of our supported databases, of course).
This means that I do not always see the problems that clients face when using our product.
A client complained to my team that the time taken to stage records was unacceptable and
that they would not take a new release of our software until we fixed this issue.
This posed a serious problem to my team as none of us are SQL Server experts and we had
learn very quickly how to make it the fastest database that we supported.

Firstly, we gained some intelligence from the client; what files were they trying to load,
how many columns, how many rows they were loading, and we ended up with the following:

  * 56 columns
  * ~ 10,000 rows of a moderately complex table

Armed with this information, I set out, with another team member, to profile
what were currently doing and looking at ways that we could improve.

Our system maintains audit logs for all records that are updated, deleted, or
added to the system, this means that staging 10,000 rows into a table that
already contains those rows will generate:

  * an update statement for any rows that currently exist, in the audit table
  * 10,000 [upsert](http://en.wikipedia.org/wiki/Upsert) (using a
  [MERGE](http://technet.microsoft.com/en-us/library/bb510625.aspx)
  statement in the case of SQL Server 2008) statements in the staging table
  * an insert statement for records that are new in the latest upload, in the audit table
  * a delete statement for records that should not exist, in the audit table
  * another set of update records, in the audit table

We were partially on the right track; using a merge statement is the correct
way to perform an update / insert in SQL Server, our mistake was using so many
of them. The SQL Server 2008 query planner does not cache query plans. So,
every time that you execute a query, it must be parsed, compiled, and a query
plan generated, therefore you want to keep the number of queries needed to
perform an operation to a minimum wherever possible, especially if you need to
perform a particular operation thousands of times.

Our second mistake was not
having the correct indexes on the table that we were merging into. In fact,
our client had no indexes on the table that we were merging into. We used the
tools available with SQL Server to try and optimise our queries but it seems
that our merge statement was too complex for the query optimiser, as it
offered us the not very helpful hint of adding all of the columns in our
target table to an index. Not surprisingly, this considerably slowed our
inserts down. After reading up on merge
[statement best practices](http://technet.microsoft.com/en-us/library/cc879317.aspx),
we added the appropriate indexes to our join columns.

It was suggested by someone in
another team that the best way to process this number of records was to use a
temporary table, and have the database merge the temporary table into the
target table. This is the technique we ended up using and it is super fast,
but first, we had to get our new records into the temporary table.

SQL Server
supports a number of [insert syntaxes](http://technet.microsoft.com/en-
us/library/ms174335.aspx), some of which are non standard and allow you to
exclude column names or insert multiple rows with one statement. We
experimented with the T-SQL insert that allowed multiple rows to be inserted,
which is expressed in the following manner:

``` sql
INSERT INTO my_table_name (column_1, column_2, column_2)
    VALUES ('some', 'string', 'values'), ('some', 'other', 'string');
```

If the row data is inserted in the same order as the columns in the target
table, you do not have to include the column names in your query, but we chose
to include them for clarity.

This syntax allows you insert up to 1000 records
at a time, using one insert statement. This was an interesting discovery. We
had found the way to reduce the number of insert queries we had to produce and
using this technique we would have 10 inserts into our temporary table, and
the merge from our temporary table into our target table would be super fast.
Mission accomplished!

We were wrong.

After some experimentation with the
number of insert records, we found that as we increased the number of records
inserted, the query slowed down considerably. We found for our situation that
10 records per INSERT statement was the magic number.

Our new procedure for
staging records generated:

  * an update statement for any rows that currently exist, in the audit table
  * a temporary table that represented the target table
  * 1000 T-SQL INSERT statements, in the temporary table
  * 1 MERGE statement into the staging table
  * a drop statement for the temporary table
  * an insert statement for records that are new in the latest upload, in the audit table
  * a delete statement for records that should not exist, in the audit table
  * another set of update records, in the audit table

After implementing the changes mentioned above, and some heavy handed code
optimisation, we managed to take our staging time for the given records from
16 minutes to sub 1 minute.

During our investigation, we also changed the
backup strategies, how much was allocated for logging, and disabled auto-
resize for all logs. Before you modify your backup strategy, you should
[read](http://msdn.microsoft.com/en-us/library/ms187048.aspx) and fully
understand the implications for your organisation in doing so. Doing this
allowed us to focus on optimising our code and be confident that we were using
our hardware to its full capacity. These settings were not changed in
production.

