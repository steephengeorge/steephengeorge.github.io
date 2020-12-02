---
layout: inner
position: center
title: 'PostgreSQL Multi-Version Concurrency Control and its impacts'
date: 2020-07-18 16:15:00
categories: Databases PostgreSQL
tags: PostgreSQL Autovacuum Troubleshooting
lead_text: 'On a fine morning, one of the application queries hanged forever.
Ultimately the screen is frozen. The application is useless. And interestingly, the screen is interactive for some other set of queries.
That confirms there is no system-wide issue. While researching, you narrow down to the specific query.
One possible doubt, is the index corrupted? Analyze the indexes of all the tables part of the SQL query. 
Did we miss any index to deploy? Did we place the conditional columns in an appropriate order to enhance the performance? 
Is there any deadlock scenario behind the scene? None of them poped up...'
project_link: 'http://steephengeorge.github.io/databases/postgresql/2020/07/18/postgresql-mvcc-impacts.html'

---

# PostgreSQL: Multi-Version Concurrency Control and its impacts

![Postgresql_mvcc](/img/posts/PostgreSQL_MVCC.jpg)

Relational databases follow the ACID Guarantee. ACID stands for:

- Atomicity: A transaction considers as a single unit irrespective of actions within a transaction.
- Consistency: Transaction changes database from a valid state to another.
- Isolation: The impact of concurrent transactions would be the same as they execute sequentially.
- Durability: Committed transactions saved in permanent storage.

Relational databases use different strategies to manage concurrent transactions to maintain ACID guarantees. Most of them use the lock on a table or a row of records. PostgreSQL is different here. It uses a Multi-Version Concurrency Control (MVCC). In the case of MVCC, locks on reading will not impact the one on writing. Each transaction works on a snapshot of the data set at the time of execution. So if there are concurrent read and write transactions, PostgreSQL maintains two versions of data to avoid a deadlock scenario. That means each UPDATE and DELETE operations generate different versions of the same data.

## Data version management

Now you may have a question? If PostgreSQL keeps multiple versions of the same data over UPDATE or DELETE, how is it managing its bookkeeping up to date and provides consistency? It means every UPDATE and DELETE are soft-operations within PostgreSQL. They are going to generate a lot of dead tuples within the database.

To remove the dead tuples from the database over time, PostgreSQL provides a mechanism called VACUUM.

## VACUUM Operation

Manual Vacuum execution removes the dead tuples from the database. We can execute it at the table level. Unfortunately, manual vacuum execution locks the table and block query execution against the table. And there is no guarantee to evaluate how long it will take to finish this operation. It is not an acceptable scenario for production systems.

## AUTOVACUUM Operation

As you know, databases have internal engines to evaluates the statistics and make decisions. This process is quite evident in the scenario of selecting an execution plan for SQL queries. Databases collect the query execution statistics and, over time, fine-tune the execution plan to make it fast.

In the case of the vacuum process, the database has a lot of statistics in hand. How can we utilize those and automate the vacuum process? The answer to that question is autovacuum.

PostgreSQL provides a bunch of configuration parameters to fine-tune the autovacuum process. By using appropriate values, we can manage to attain more fine-grained, more frequent vacuum execution. You can find a detailed explanation of all available parameters [here](https://www.postgresql.org/docs/12/runtime-config-autovacuum.html).

Let me introduce some of the parameters for autovacuum fine-tuning here with an example. Assume you have a table with thousands of records. You set the configuration parameters as follows. Let's consider how does it work in this case.

_autovacuum\_vacuum\_threshold = 100_

_autovacuum\_vacuum\_scale\_factor = 0.3_

The count of obsolete records within your table reaches _100+ (0.3 \* 1000)_, the autovacuum execution will trigger.

Yes, we are all set. Everything is going well.

Do you remember I explained above, manual vacuum locks the table and blocks the query execution against the table? How is the same scenario going to be managed by autovacuum? Autovacuum by design stops execution and exit the process if a query hits the table.

## One potential problem scenario

All are happy the application is running pretty well. On a fine morning, one of the application queries hanged forever. Ultimately the screen is frozen. The application is useless. And interestingly, the screen is interactive for some other set of queries.

That confirms there is no system-wide issue. While researching, you narrow down to the specific query. One possible doubt, the index is corrupted. Analyze the indexes of all the tables part of the SQL query. Did we miss any index to deploy? Did we place the conditional columns in an appropriate order to enhance the performance? Is there any deadlock scenario behind the scene? None of them poped up.

Later we will conclude, all these easy targets are out of luck. We need to check something on a grand scale to nail down this issue. The next two things that came up to mind are table bloating and index bloating. The pgx\_scripts [repository](https://github.com/pgexperts/pgx_scripts)from pgexperts has some outstanding scripts available to help. That revealed the culprit behind the issue. A table is bloated, which is part of the query we narrow down earlier. And every time we try to execute the query manually, we observed PostgreSQL is canceling the autovacum execution to avoid a table lock. So it was quite obvious, autovacuum configuration is not working for this table.

## Tablewise AUTOVACUUM Tuning

On further investigation, it was quite evident why system-level autovacuum configurations failed to tackle this odd table. It was one of the smallest tables in the database. The total number of records was lesser than _autovacuum\_vacuum\_threshold._

You can use the same set of autovacuum parameters with specific values to set the tablewise autovacuum.

_Ex:_

_ALTER TABLE employee SET (autovacuum\_vacuum\_scale\_factor = 0.1, autovacuum\_vacuum\_threshold = 20);_

To verify the changes use following query:

_select pg\_options\_to\_table(reloptions) from pg\_class where relname='employee ' ;_

_pg\_options\_to\_table_

_--------------------_

_autovacuum\_vacuum\_scale\_factor = 0.1_

_autovacuum\_vacuum\_threshold = 20_
