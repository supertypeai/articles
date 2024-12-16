---
title: "Optimizing queries in Postgres and BigQuery | Part 1 of Optimized Analytics Applications"
post_excerpt: A whirlwind tour of query optimization strategies ft. query plans, index scans, and BigQuery-specific optimizations
taxonomy:
  category:
    - knowledge
    - notes
  post_tag:
    - postgres
    - bigquery
    - analytics
    - database
    - dataops
---

## Key Points (_TL;DR_)

1. You don't optimize queries until you've seen the Query Plan. This is your `EXPLAIN ANALYZE`.

2. Sequential Scan is for when you have no other good options left. Your most common queries need to be Index Scans. This is the difference between a 14 second execution time and a 0.5 seconds execution time (99.996% improvement in performance ⚡)

3. But both (1) and (2) don't work on Google BigQuery. No `EXPLAIN`, no `ANALYZE`, no index scans. Every query in BigQuery is a full table scan unless you use BigQuery's specialized optimization features.

4. BigQuery compensates by adding more nodes (more parallelism) to help you achieve near constant time as your data scale.

5. Instead, use BigQuery's Query Execution Graph to look for bottlenecks and find optimization opportunities

6. Also use BigQuery's Clustered Tables and Partitioned Tables for optimization

The article contains screenshots, benchmark tests, and code snippets for each of the aforementioned points, so dive right in.

---

## Building optimized analytics applications: from Postgres to BigQuery

This is the first in a series of articles on building highly optimized analytics applications. We're going to start with the basic mental model: thinking about the stack of software that makes up a modern analytics application. We'll see where bottlenecks can occur at each layer of the analytics stack, and work through some concrete examples of how to diagnose and fix them.

### Background: The analytics stack

For the purpose of this article, we'll define the analytics stack as our collection of application layers responsible for ingesting, storing, querying, and finally an interface to the front end for search and query. Supposed the end users are business analysts relying on our web application to query for results, after they have provided a set of filters and search terms.

This stack is made up of the following components:

- A database for storing the data
- A query engine for querying the data
- A web application for querying the data

Few months into the project, our data analysts start complaining that the web application feels sluggish. Usage of the application has increased, and the analysts are now spending more time waiting for the results of their queries. Where do we start?

Welps! A few discussions with the data architects and analytics engineers on the team reveal that the database might be the bottleneck. In fact, we were told, that the team is in the process of a multi-month migration from an on-premise PostgreSQL database to Google's BigQuery. The migration is still in progress, and we are asked to help diagnose the performance issues in the meantime.

- PostgreSQl: the world's most advanced open source database and perhaps the most popular choice for mature startups
- BigQuery: Google's fully managed and highly scalable data warehouse

In the next few articles of this series, we'll look at each components of the Analytics stack in turn, and see how we can diagnose and fix performance issues. Our goal is to construct a mental model of the analytics stack, and gain an overview of the tools at our disposal at each layer of the stack. We'll start with the database (focusing on on-premise databases, and then later BigQuery), and work our way up to the web analytics application layer.

### Migrating our database: PostgreSQL to BigQuery

At the bottom of our analytics stack is the database. Currently it is the trusty PostgreSQL database responsible for storing the data, and providing a query interface for querying the data. Being the most fundamental component of the analytics stack, it is naturally the first place to look when diagnosing performance issues.

Fortunately for us, databases today are extremely well optimized. Most modern databases are designed to be fast, and are optimized for a wide variety of workloads. For the most part, troubleshooting performance issues in a database is a matter of using these provided tools, i.e.

- the query planner (sometimes called the query analyzer)
- the query optimizer (sometimes called the query executor, and sometimes it's bundled with the query planner)
- the execution engine to diagnose the problem.

Let's start of by looking at the query planner of our PostgreSQL database. The query planner is responsible for taking a query, and formulating a plan for how to execute that query. PostgreSQL has this to say about the query planner:

> The task of the planner/optimizer is to create an optimal execution plan. A given SQL query (and hence, a query tree) can be actually executed in a wide variety of different ways, each of which will produce the same set of results. If it is computationally feasible, the query optimizer will examine each of these possible execution plans, ultimately selecting the execution plan that is expected to run the fastest.

Given that the query planner is directly responsible for making sure that our queries are executed in the most efficient way possible, databases provide a way for us to inspect the query plan. This is done by using the EXPLAIN command. For example, let's say we have a table called `users` with the following schema:

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name TEXT,
  email TEXT,
  age INTEGER
);
```

And we have the following query:

```sql
SELECT * FROM users WHERE age > 30;
```

We can use the EXPLAIN command to see the query plan for this query:

```sql
EXPLAIN SELECT * FROM users WHERE age > 30;
```

Which will output the following:

```
QUERY PLAN
Seq Scan on users  (cost=0.00..10.00 rows=1000 width=16)
  Filter: (age > 30)
```

The query plan is a tree structure, where each node represents a step in the execution of the query. In this case, the query plan consists of a single node, which is a sequential scan of the `users` table. The sequential scan is the most basic form of table scan, where the database will scan the table from the beginning to the end, and return the rows that match the query. The sequential scan is the default table scan, and is used when the database is unable to determine a better way to execute the query.

The Query Plan you see will depend on the database you are using, and the query you are executing. For example, if you are using PostgreSQL, and you have an index on the `age` column, the query plan will look like this:

```
QUERY PLAN
Index Scan using users_age_idx on users  (cost=0.00..10.00 rows=1000 width=16)
  Index Cond: (age > 30)
(2 rows)
```

Notably, the query plan has changed from a sequential scan to an index scan. The index scan is a more efficient way to execute the query, because it allows the database to avoid scanning the entire table. Instead, the database will use the index to find the rows that match the query, and return those rows. This is a much more efficient way to execute the query, and is the reason why we use indexes in the first place.

The `EXPLAIN` command shows the generated query plan but it does not actually execute the query. To actually execute the query, we can use the `EXPLAIN ANALYZE` command:

```sql
EXPLAIN ANALYZE SELECT * FROM receipts WHERE amount > 5000;
```

Which will output the following (again, assuming we have an index on the `amount` column):

```
QUERY PLAN
Index Scan using receipts_amount_idx on receipts  (cost=0.00..10.00 rows=1000 width=16) (actual time=0.012..0.012 rows=0 loops=1)
  Index Cond: (amount > 5000)
Planning time: 0.000 ms
Execution time: 0.013 ms
(17 rows)
```

#### Comparing sequential scan vs index scan

Loading this [dataset](https://data.montgomerycountymd.gov/api/views/4mse-ku6q/columns.json) into our database (expect >1.84 million rows and 43 columns), we can use `EXPLAIN ANALYZE` to compare the performance of a sequential scan vs an index scan. First, the sequential scan:

```sql
EXPLAIN ANALYZE SELECT * FROM traffic WHERE serial_id = 1;
```

The output:

```
QUERY PLAN
Gather  (cost=1000.00..393206.18 rows=1 width=1988) (actual time=13050.187..13243.14 rows=0 loops=1)
  Workers Planned: 2
  Workers Launched: 2
  ->  Parallel Seq Scan on traffic  (cost=0.00..393206.18 rows=1 width=1988) (actual time=10584.42..13070.38 rows=0 loops=3)
        Filter: (serial_id = 1)
        Rows Removed by Filter: 507306
Planning time: 0.431 ms
Execution time: 13910.25 ms
(9 rows)
```

This query uses 2 workers to perform a parallel sequential scan and took 13.9 seconds to execute with a start up time of 1 second (1000ms), resulting in a total time of 14.9 seconds (14910ms). PostgreSQL further explain the output:

> Actually two numbers are shown: the start-up time before the first row can be returned, and the total time to return all the rows. For most queries the total time is what matters, but in contexts such as a subquery in EXISTS, the planner will choose the smallest start-up time instead of the smallest total time (since the executor will stop after getting one row, anyway)

Now, let's try the same query with an index scan:

```sql
CREATE INDEX traffic_serial_id_idx ON traffic(serial_id);
EXPLAIN ANALYZE SELECT * FROM traffic WHERE serial_id = 1;
```

The output:

```
QUERY PLAN
Index Scan using traffic_serial_id_idx on traffic  (cost=0.461..8.39 rows=1 width=381) (actual time=0.513..0.530 rows=1 loops=1)
  Index Cond: (serial_id = 1)
Planning time: 3.983 ms
Execution time: 0.531 ms
(4 rows)
```

Now instead of taking ~14 seconds to return a single row, it took 0.5 seconds to return a single row. This represents a 99.996% improvement in performance.

The query planner, in addition to making sure that the query is executed in the most efficient way possible, is also responsible for making sure that the query is executed in a way that is safe for the database. For example, the query planner will make sure that the query is executed in a way that does not cause a deadlock.

When your analytics application is running slow independent of the interface you are using (raw queries, SQL Lab, Superset, ORM, some API client, etc.), hopping in with the `EXPLAIN` command to investigate the query plan is often a great place. If necessary, you can also use the `EXPLAIN ANALYZE VERBOSE` command to see the query plan, and the time it took to execute each step of the query plan. This will help you identify the bottleneck in your query.

### Google BigQuery Optimizations and Query Plans

Many modern analytics stacks are built on top of cloud data warehouses like Google BigQuery, Amazon Redshift, and Snowflake instead of traditional, on-premise, self-hosted database servers. In those scenarios, the enterprise analytics stack might look a little different from our hypothetical stack above. The diagnostic tools and lessons are still largely applicable, even though the underlying technology might be different and the means to access these tools might differ.

Let's see a concrete example with Google BigQuery, the technology of choice for our ongoing migration to a cloud data warehouse.

With BigQuery, your diagnostic query plan and timing information is embedded within query jobs; This means that you won't be able to use the `EXPLAIN` or `EXPLAIN ANALYZE` commands explictly, as they are not supported.

> Statement not supported: ExplainStatement

Instead, if you run your queries in BigQuery's web UI, you can see the query plan and timing information in the query's **Execution Details** tab:

![Query Execution BigQuery](/_images/queryopt_1.png)
We'll see that BigQuery does a pretty remarkable job at processing 17,717,491 rows of records in under 395 milliseconds.

The query is as follows and it essentially prints the top search term in the US each day for the last 2 weeks:

```sql
SELECT
   refresh_date AS Day,
   term AS Top_Term,
       -- These search terms are in the top 25 in the US each day.
   rank,
FROM `bigquery-public-data.google_trends.top_terms`
WHERE
   rank = 1
       -- Choose only the top term each day.
   AND refresh_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 2 WEEK)
       -- Filter to the last 2 weeks.
GROUP BY Day, Top_Term, rank
ORDER BY Day DESC
   -- Show the days in reverse chronological order.
```

Since the most recent day has yet to conclude, we expect this query to return 13 rows (one for each day in the last 2 weeks minus the most recent one). We see the Elapsed time of 395 milliseconds, and how many bytes were processed. We can also see the query plan, which is a DAG (Directed Acyclic Graph) of the steps that BigQuery took to execute the query (Execution Graph tab).

We observe how BigQuery breaks this query into 3 stages, and how each stage is executed in parallel. When a stage is distributed and executed in parallel, there will be a difference between the average length of the stage execution or the longest execution per stage. BigQuery's query planner will distribute the query ("S00:Input" and "S01:Aggregate") across multiple nodes so they can be executed in parallel on each node. This query planner will then combine the results from each node, and return the final result ("S02:Ouput"). Toggling between "Show Average Time" and "Show Maximum Time" will provide a difference between the average length of the stage execution and the longest execution per stage.

In each stage, we can observe from the Execution Details and from the Execution Graph the number of records that it takes as input, and the number of records that it outputs.

![Query Execution BigQuery](/_images/queryopt_2.png)

- In S00:Input, becase of the `FROM bigquery-public-data.google_trends.top_terms` clause, BigQuery will read 17,717,491 rows from the table. But because of the `GROUP BY` `WHERE rank = 1 AND refresh_date > DATE_SUB(CURRENT_DATE(), INTERVAL 2 WEEK)` clause, BigQuery will filter out everything except the 13 rows that meet the criteria. Unsurprisingly, the Read and Compute time are also longest here.

- In S02: Ouput, BigQuery will return the 13 rows that meet the criteria after sorting them due to the presence of our `ORDER BY` clause. This takes longer than the previous Aggregate phase, but is still much faster than the Input phase.

In essence, from an optimization point of view there are a couple of things you want to keep an eye on when looking at this query plan:

- the less bytes a phase has to process, the faster it will be. This in part explains why the Input phase is typically the longest in a classic ETL pipeline.
- the amount of bytes that are shuffled between nodes is also a good indicator of how much data is being processed. If there is a bottleneck here, you will want to consult with your cloud architect to make sure that your data is partitioned and clustered properly.
- In each phase, you should be looking also at the number of records that are being processed; This often presents opportunities to optimize your query. For example, you may want to re-arrange your query to filter out records that you don't need before you perform a `GROUP BY` or `JOIN` operation.
- In terms of time spent, click away from "Show Average Time" to "Show Maximum Time" as well to reveal the longest execution time in each phase. This again presents opportunity to optimize your query

If you're a developer, you can also access the query plan and timing information using the `google.cloud.bigquery` library (available in Python, the `bq` commmand line tool, Go etc). The results of the query will be returned as a `google.cloud.bigquery.job.QueryJob` object. You can then access the query plan and timing information using the `query_plan` and `total_bytes_processed` attributes, respectively.

Finally, I'd like the point out that most data warehouses, BigQuery included, have a query optimizer that will automatically optimize your query for you. This is a great feature, but it's important to understand how it works so that you can make sure that it's doing what you expect it to do. Additionally, just like the handy `--dry-run` option in PostgreSQL, BigQuery also has a `--dry-run` option (or `dry_run=True` parameter when using the Python client) that will run your query, but will not actually execute it. This is a great way to test out your query and make sure that it's doing what you expect it to do, and that the time it will take to execute is what you expect it to be, before actually running it and incurring any costs.

The dry run feature is enabled by default in BigQuery's web UI, and you can see the amount of data that the query will process automatically even before you execute it.

#### What about the aforementioned Index scans?

Depending on the underlying database architecture, your data warehouse may not support index scans. This varies from database to database, and might be a little out of scope for this post.

BigQuery does not support indexes, and every query is a full table scan. The advantage is that you do not have to maintain a separate index table, or hiring a database consultant to plan out your indexing strategy, and you don't have to worry about the query optimizer not using the index that you think it should be using. The disadvantage is that

- you have to be mindful of the size of your tables, and of course, the size of your queries
- the optimization will be done for you, but the underlying storage systems may feel like a black box.

Where we can reach for trusty optimization techniques like indexing with out PostgreSQL, our post-migration options for active optimization will be limited (due to the black box nature of BigQuery's built-in query optimizer). In exchange of that loss in flexibility, BigQuery offers a more direct solution: adding more nodes to your cluster to increase the amount of parallelism that your query can take advantage of.

#### Speeding up your queries with Clustered Tables and Partitioned Tables in BigQuery

BigQuery also introduces its own innovation, known as the Clustered Tables. Supposed the data analysts on our team are primarily querying for shipment status by date and country, we can create a table that is clustered by `date` and `country` to optimize for these queries.

![](/_images/queryopt_3.png)

This will allow BigQuery to optimize for these queries, but will not allow us to optimize for queries that are not clustered by `date` and `country`. A design decision like these are typically made by the data architects in relation to the queries that are being run by the data analysts and business users. It's also a reason why at Supertype we advocate for a full-cycle approach to building and scaling data architectures for our clients.

Running some benchmarks using the same query on a Clustered Table and a non-Clustered Table, we yield the same result but with significantly different query times and costs.

![](/_images/queryopt_4.png)

The query on the left processed 2.2 TB in 20.7 seconds, while the one on the right does it in 5.4 seconds and with one-tenth of the cost.

When you combine this technique with Partitioned Tables, you can further optimize your queries. Partitioning a table allows you to split a table into smaller chunks, and to optimize for queries that are clustered by a specific partition. When used together, the analytic process will see your data engineers first segment your data into partitions, before clustering the data within each partition by the clustering columns -- with respect to the queries that are commonly being run by the data analysts and business users.

### Closing: Overcoming performance bottlenecks on the database architectural layer

In our hypothetical scenario, we start our diagnosis from the bottom of the analytics stack, and that is the database layer. Given that we're in the midst of migrating the data infrastructure from an on-premise PostgreSQL database to a cloud-based BigQuery data warehouse, we looked at various strategies to increase the performance of our queries.

With a little bit of planning and forethought, we can employ strategies like indexing, clustering, and partitioning to optimize our queries. We can also use the query plan and timing information to identify bottlenecks in our query, and use dry runs to test out our queries before we actually run them.

In the next article in this series, we'll move up the stack to the "search" layer, and look at how we can further optimize our search queries so that our analysts can spend less time goofing around with "faux coffee breaks" in between waiting for their queries to finish; and actually spend more time analyzing the data.

This article is part of a series on building optimized analytics infrastructure. The PDF version is also available for download here (first available on our [LinkedIn page](https://www.linkedin.com/company/supertype-ai/))

- [Optimizing queries in Postgres and BigQuery](https://supertype.ai/wp-content/uploads/2023/03/optimizing-postgres-bigquery-1.pdf)
