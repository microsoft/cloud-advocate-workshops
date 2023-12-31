# Real Time Transactional and Analytical Processing on Azure Database for PostgreSQL - Hyperscale (Citus)

Azure Database for PostgreSQL is a fully managed database-as-a-service based on the open-source Postgres relational database engine. The Hyperscale (Citus) deployment option enables you to scale queries horizontally- across multiple machines, to serve applications that require greater scale and performance. Citus transforms Postgres into a distributed database with features like [co-location](https://docs.citusdata.com/en/stable/get_started/concepts.html#co-location), a distributed SQL engine, reference tables, distributed tables and many more. The combination of [parallelism](https://docs.citusdata.com/en/stable/get_started/concepts.html#parallelism), keeping more data in memory, and higher I/O bandwidth can lead to significant performance improvements

With the latest release, Citus 10 is now available in preview on Azure Hyperscale (Citus) with new capabilities like Columnar Storage, sharding on a single node Postgres machine, Joins between Local PostgreSQL & Citus tables and much more. With Basic Tier, you can now build applications that are scale ready from day one.

In this lab, we will learn about some of the superpowers that Citus brings in to the table by distributing data across multiple nodes. We will explore:

- How to create an Azure Database for PostgreSQL-Hyperscale (Citus) using Azure Portal
- Concepts of Sharding on Hyperscale (Citus) Basic Tier
- Creating schemas and ingesting data into an Hyperscale (Citus) instance
- Using Columnar Storage to reduce storage cost and speedup analytical queries
- Scaling the Hyperscale (Citus)-Basic Tier to Standard Tier
- Rebalancing the data and capturing performance improvements

To test the new features of Citus you can either use:

- [Citus 10 Open Source](https://www.citusdata.com/download/) or;
- [Hyperscale (Citus) on Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/hyperscale-overview)

**Note:** You can even run Citus on [Docker](https://docs.citusdata.com/en/v10.0/installation/single_node_docker.html). But please note that the docker image is intended to be used for development or testing purposes only and not for production workloads.

```bash
# run PostgreSQL with Citus on port 5500
docker run -d --name citus -p 5500:5432 -e POSTGRES_PASSWORD=mypassword citusdata/citus
```

## Prerequisites

- Azure Subscription (e.g. [Free](https://aka.ms/azure-free-account) or [Student](https://aka.ms/azure-student-account))
- An **Azure Database for PostgreSQL-Hyperscale Server-Basic Tier** (Detailed steps are listed [here](https://docs.microsoft.com/azure/postgresql/quickstart-create-hyperscale-basic-tier)). For this lab, we will start with Azure Basic Tier- run queries & capture performance benchmarks and later scale it to Standard Tier to see the performance improvements introduced by horizantal scaling of nodes.
- You will also need [psql](https://www.postgresql.org/download/) (Ver 11 is recommended), which is included in [Azure Cloud Shell](https://docs.microsoft.com/en-ca/azure/cloud-shell/overview).
- [Optional] If you want you can also run Citus open source on your laptop as a single Docker container!
    ```bash
    # run PostgreSQL with Citus on port 5500
    docker run -d --name citus -p 5500:5432 -e POSTGRES_PASSWORD=mypassword citusdata/citus
    ```

## Connecting to the Hyperscale (Citus) Database

Connecting to an Azure Database for PostgreSQL-Hyperscale (Citus) database requires the fully qualified server name and login credentials. You can get this information from the Azure portal.

1. In the [Azure portal](https://portal.azure.com/), search for and select your Azure Database for PostgreSQL-Hyperscale (Citus) server name. 
1. On the server's **Overview** page, copy the fully qualified **Server name**. The fully qualified **Server name** is always of the form *\<my-server-name>.postgres.database.azure.com*. For Hyperscale (Citus) the default **Admin username** is always **'Citus'**.
1. You will also need your **Admin password** which you chose when you created the server, otherwise you can reset it using the `Reset password` button on `Overview` page.

Note: Make sure you have created a [server-level firewall rule](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal#configure-a-server-level-firewall-rule) to allow traffic from the IP address of the machine you will be using to connect to the database. If you are connected to a remote machine via SSH, you can find your current IP address via the terminal using `dig +short myip.opendns.com @resolver1.opendns.com`.

## Creating Schema and Data Distribution on Citus

As we are now able to connect to the Hyperscale (Citus) server, let us move forward and define the table structure. For this lab, we will use a small sample of Covid-19 time-series data released by the UK government (part of [OGL license](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/))  and try to get some insights on the vaccination drive. For more details please refer [coronavirus.data.gov.uk](https://coronavirus.data.gov.uk/).


You can create the tables by using standard PostgreSQL CREATE TABLE commands as shown below:

```sql
-- re-initializing database
DROP OWNED BY citus;

CREATE SCHEMA IF NOT EXISTS covid19;

SET search_path='covid19';

-- Sequences

CREATE SEQUENCE covid19.area_reference_id_seq
    INCREMENT 1
    START 1
    MINVALUE 1
    MAXVALUE 2147483647
    CACHE 1;

CREATE SEQUENCE covid19.metric_reference_id_seq
    INCREMENT 1
    START 1
    MINVALUE 1
    MAXVALUE 2147483647
    CACHE 1;

CREATE SEQUENCE covid19.release_reference_id_seq
    INCREMENT 1
    START 1
    MINVALUE 1
    MAXVALUE 2147483647
    CACHE 1;

-- Tables

CREATE TABLE covid19.area_reference
(
    id integer NOT NULL DEFAULT nextval('area_reference_id_seq'::regclass),
    area_type character varying(15) COLLATE pg_catalog."default" NOT NULL,
    area_code character varying(12) COLLATE pg_catalog."default" NOT NULL,
    area_name character varying(120) COLLATE pg_catalog."default" NOT NULL,
    unique_ref character varying(26) COLLATE pg_catalog."default" NOT NULL DEFAULT "substring"(((now())::character varying)::text, 0, 26),
    CONSTRAINT area_reference_pkey PRIMARY KEY (area_type, area_code),
    CONSTRAINT area_reference_id_key UNIQUE (id),
    CONSTRAINT unq_area_reference_ref UNIQUE (unique_ref)
);


CREATE TABLE covid19.metric_reference
(
    id integer NOT NULL DEFAULT nextval('metric_reference_id_seq'::regclass),
    metric character varying(120) COLLATE pg_catalog."default" NOT NULL,
    released boolean NOT NULL DEFAULT false,
    metric_name character varying(150) COLLATE pg_catalog."default",
    source_metric boolean NOT NULL DEFAULT false,
    CONSTRAINT metric_reference_pkey PRIMARY KEY (id),
    CONSTRAINT metric_reference_metric_key UNIQUE (metric)
);


CREATE TABLE covid19.release_reference
(
    id integer NOT NULL DEFAULT nextval('release_reference_id_seq'::regclass),
    "timestamp" timestamp without time zone NOT NULL,
    released boolean NOT NULL DEFAULT false,
    CONSTRAINT release_reference_pkey PRIMARY KEY (id),
    CONSTRAINT release_reference_timestamp_key UNIQUE ("timestamp")
);


CREATE TABLE covid19.time_series
(
    hash character varying(24) COLLATE pg_catalog."default" NOT NULL,
    partition_id character varying(26) COLLATE pg_catalog."default" NOT NULL,
    release_id integer NOT NULL,
    area_id integer NOT NULL,
    metric_id integer NOT NULL,
    date date NOT NULL,
    payload jsonb DEFAULT '{"value": null}'::jsonb
) PARTITION BY RANGE (date) ;


-- Partitions SQL

CREATE TABLE covid19.time_series_250421_to_290421 PARTITION OF covid19.time_series
    FOR VALUES FROM ('2021-04-25') TO ('2021-04-30');

CREATE TABLE covid19.time_series_300421_to_040521 PARTITION OF covid19.time_series
    FOR VALUES FROM ('2021-04-30') TO ('2021-05-05');
```

Now that the schema is ready, we can focus on deciding the right distribution strategy to shard tables across nodes on Citus cluster and data ingestion. Below table describes the different types of table on Citus cluster:

| Sr. | Table Type        | Description |
|-----|-------------------|-------------|
| 1   | [Distributed Table](https://docs.citusdata.com/en/stable/get_started/concepts.html#type-1-distributed-tables) | Large tables that are horizontally partitioned across worker nodes.Helps in scaling and parallelism. |
| 2   | [Reference Table](https://docs.citusdata.com/en/stable/get_started/concepts.html#type-2-reference-tables)   | Tables that are replicated on each node. Generally, tables which are smaller in size but are used frequently in JOINs|
| 3   | [Local Table](https://docs.citusdata.com/en/stable/get_started/concepts.html#type-3-local-tables)       | Tables that stays on coordinator node. Generally, the ones with no dependencies or JOINS. |

In our case `time_series` is the largest table that holds real time Covid19 data for various metrics across different areas in UK, and others are supporting tables with less data- which when joined with `time_series` helps in building useful analytics.


Next we’ll take these Postgres tables on the coordinator node and tell Hyperscale(Citus) to either distribute or replicate them across the workers. To do so, we’ll run a query for each table specifying the key to shard it on. In the current example we’ll shard `time_series` table on `area_id`, and make other three tables are reference tables to avoid cross shard operations.

```sql
SELECT create_distributed_table('time_series', 'area_id');

SELECT create_reference_table('area_reference');
SELECT create_reference_table('metric_reference');
SELECT create_reference_table('release_reference');
```

We're now ready to load the data. In psql, shell out to download the files:

```bash
curl -O https://raw.githubusercontent.com/Azure-Samples/azure-python-labs/postgres-1/4-postgres-citus/data/area_reference.csv
curl -O https://raw.githubusercontent.com/Azure-Samples/azure-python-labs/postgres-1/4-postgres-citus/data/metric_reference.csv
curl -O https://raw.githubusercontent.com/Azure-Samples/azure-python-labs/postgres-1/4-postgres-citus/data/release_reference.csv
curl -O https://raw.githubusercontent.com/Azure-Samples/azure-python-labs/postgres-1/4-postgres-citus/data/time_seriesaa.csv
curl -O https://raw.githubusercontent.com/Azure-Samples/azure-python-labs/postgres-1/4-postgres-citus/data/time_seriesab.csv
curl -O https://raw.githubusercontent.com/Azure-Samples/azure-python-labs/postgres-1/4-postgres-citus/data/time_seriesac.csv
curl -O https://raw.githubusercontent.com/Azure-Samples/azure-python-labs/postgres-1/4-postgres-citus/data/time_seriesad.csv
curl -O https://raw.githubusercontent.com/Azure-Samples/azure-python-labs/postgres-1/4-postgres-citus/data/time_seriesae.csv
curl -O https://raw.githubusercontent.com/Azure-Samples/azure-python-labs/postgres-1/4-postgres-citus/data/time_seriesaf.csv
curl -O https://raw.githubusercontent.com/Azure-Samples/azure-python-labs/postgres-1/4-postgres-citus/data/time_seriesag.csv
```

Next, connect to the database server again and load the data from the files into the distributed tables:

```bash
\copy covid19.area_reference from 'area_reference.csv' WITH CSV
\copy covid19.metric_reference from 'metric_reference.csv' WITH CSV
\copy covid19.release_reference from 'release_reference.csv' WITH CSV
\copy covid19.time_series from 'time_seriesaa.csv' WITH CSV
\copy covid19.time_series from 'time_seriesab.csv' WITH CSV
\copy covid19.time_series from 'time_seriesac.csv' WITH CSV
\copy covid19.time_series from 'time_seriesad.csv' WITH CSV
\copy covid19.time_series from 'time_seriesae.csv' WITH CSV
\copy covid19.time_series from 'time_seriesaf.csv' WITH CSV
\copy covid19.time_series from 'time_seriesag.csv' WITH CSV
```

## Running Queries

Now it's time for the fun part, actually running some queries. Let's start with a simple `count (*)` to see how much data we loaded:

```sql
SELECT count(*) from covid19.time_series;
```
That worked nicely. We'll move on to some complex queries in a bit, but for now we will see the benefit that we get with Columnar storage introduced with Citus 10. We have partitioned `time_series` table into two- `time_series_250421_to_290421` that holds data from 25 April till 29 April 2021 and `time_series_300421_to_040521` holds more recent data from 30 April till 04 May 2021.

Let us now check the disk space consumed by the table `time_series`:

```sql
SELECT pg_size_pretty(citus_total_relation_size('time_series_250421_to_290421') + citus_total_relation_size('time_series_300421_to_040521'));
SELECT pg_size_pretty(citus_total_relation_size('time_series_250421_to_290421'));
```
Output:
```text
citus=> SELECT pg_size_pretty(citus_total_relation_size('time_series_250421_to_290421') + citus_total_relation_size('time_series_300421_to_040521'));
 pg_size_pretty
----------------
 153 MB
(1 row)

citus=> SELECT pg_size_pretty(citus_total_relation_size('time_series_250421_to_290421'));
 pg_size_pretty
----------------
 121 MB
(1 row)
```


With time as data grows, Hyperscale (Citus) gives you a flexibility to compress your old partitions to save storage cost just by running below simple command that uses table access method to compress the data:

```sql
SELECT alter_table_set_access_method('time_series_250421_to_290421', 'columnar');
```
Please note that we have compressed only the first partition which is created to simulate historical data in real life scenario. Now that we have converted partition `time_series_250421_to_290421` into columnar, let us verify the table size again. 

```sql
SELECT pg_size_pretty(citus_total_relation_size('time_series_250421_to_290421'));
```
Output:
```text
 pg_size_pretty
----------------
 23 MB
(1 row)
```

Can you see the benefit of using **Columnar** storage- we got a compression ratio of about 5x for `time_series_250421_to_290421` partition. Another important aspect to notice here is that, the relation `time_series` now has both columnar storage as well as row-based storage. This is what we call as **HTAP**-(Hybrid Transactional/Analytical Processing) wherein the same database can be used for both analytical and transactional workloads.

We see that relation `time_series` has a attribute called `payload` of `jsonb` type which stores time-series metrics related to Covid-19 in UK. It also stores Foreign Keys to other tables like `area_reference`, `metric_reference` and `release_reference`. We can use this dataset to identify the no. of Covid-19 tests done in an area on a given date:

```sql
SELECT 
area_code,
area_name,
date,
MAX((payload -> 'value')::INT) AS Tests_Conducted
FROM covid19.time_series AS ts
JOIN covid19.area_reference AS ar ON ar.id = ts.area_id
JOIN covid19.metric_reference AS mr ON mr.id = ts.metric_id
JOIN covid19.release_reference AS rr ON rr.id = ts.release_id
WHERE date = '2021-04-27' 
AND metric = 'newVirusTestsRollingSum'
AND (payload -> 'value')  NOTNULL 
GROUP BY area_code, area_name, date;
```
Output:
```text
 area_code |    area_name     |    date    | tests_conducted
-----------+------------------+------------+-----------------
 S92000003 | Scotland         | 2021-04-27 |          127272
 N92000002 | Northern Ireland | 2021-04-27 |           77090
 W92000004 | Wales            | 2021-04-27 |           66246
 E92000001 | England          | 2021-04-27 |         6680473
 K02000001 | United Kingdom   | 2021-04-27 |         6951081
(5 rows)

Time: 615.612 ms
```

That was quick, isn't it - that too when we are using Citus on single node machine. You can imagine the performance we will get when we will add more nodes ot the cluster.If we look at the query above, we will observe that the query ran efficiently because we have distributed our tables such that the data is [co-located](https://docs.citusdata.com/en/stable/get_started/concepts.html#co-location) with minimal cross-shard operations.

Let's run another query that will generate stats for total no. of first dose vaccinations given across various areas in UK.

```sql
SELECT 
area_type,
area_code,
MAX(date) AS date,
MAX((payload -> 'value')::FLOAT) AS first_dose
FROM (
	SELECT *
	FROM covid19.time_series AS tm
	JOIN covid19.release_reference AS rr ON rr.id = release_id
	JOIN covid19.metric_reference AS mr ON mr.id = metric_id
	JOIN covid19.area_reference AS ar ON ar.id = tm.area_id
	 ) AS ts
WHERE date > (now() - INTERVAL '30 days')
AND metric = 'cumPeopleVaccinatedFirstDoseByPublishDate'
AND (payload -> 'value') NOTNULL
GROUP BY area_type, area_code;
```
Output:
```text
 area_type | area_code |    date    | first_dose
-----------+-----------+------------+------------
 nation    | E92000001 | 2021-05-03 |   29025049
 overview  | K02000001 | 2021-05-03 |   34667904
 nation    | S92000003 | 2021-05-03 |    2833761
 nation    | W92000004 | 2021-05-03 |    1864400
 nation    | N92000002 | 2021-05-03 |     944694
(5 rows)

Time: 1106.661 ms (00:01.107)
```

Let's try to see how a transactional query will perform on the same cluster.

```sql
UPDATE covid19.time_series
SET payload = '{"value": 1.0}'
WHERE metric_id=101 AND date='2021-04-30' AND area_id=873 AND release_id=29795 ;
```
Output:
```text
UPDATE 1
Time: 5.237 ms
```

So we see that with Hyperscale (Citus)- you can run both transactional and analytical workloads on the same machine.
Now that we are familiar with columnar and how to query data on Hyperscale (Citus), lets move on to explore another important (infact most important) capability of Hyperscale (Citus):

**The Power of Horizontal Scaling**

For this, I would request you to goto the [Azure portal](https://portal.azure.com/) again, select your Azure Database for PostgreSQL-Hyperscale (Citus) server and under **Compute + storage** section upgrade from **Basic** tier to **Standard** & increase **Worker node count** to **4** nodes as shown in screenshot below. 

![image](https://user-images.githubusercontent.com/41684987/117833371-ebbe8380-b293-11eb-82ee-77a0243dd4e3.png)

`Please note that this will force Citus cluster to restart. Also, changing the tier back from Standard to Basic is not supported.`


Once it's done and the new cluster with 4 worker nodes is available, the first thing we will have to do it rebalance the data across the new nodes that were added. This activity needs to be done only for distributed tables and not referenced ones. Reference tables automatically gets copied to the new node when the node is created.

Rebalancing distributed tables can be easily acheived by running below command:

```sql
SELECT rebalance_table_shards('time_series',shard_transfer_mode=>'force_logical');
```

Once the data is rebalanced across the new nodes, we can again re-run the above queries and compare the run times with benchmarks captured earlier for Basic Tier.

```sql
SELECT 
area_code,
area_name,
date,
MAX((payload -> 'value')::INT) AS Tests_Conducted
FROM covid19.time_series AS ts
JOIN covid19.area_reference AS ar ON ar.id = ts.area_id
JOIN covid19.metric_reference AS mr ON mr.id = ts.metric_id
JOIN covid19.release_reference AS rr ON rr.id = ts.release_id
WHERE date = '2021-04-27' 
AND metric = 'newVirusTestsRollingSum'
AND (payload -> 'value')  NOTNULL 
GROUP BY area_code, area_name, date;
```
Output:
```text
 area_code |    area_name     |    date    | tests_conducted
-----------+------------------+------------+-----------------
 S92000003 | Scotland         | 2021-04-27 |          127272
 N92000002 | Northern Ireland | 2021-04-27 |           77090
 W92000004 | Wales            | 2021-04-27 |           66246
 E92000001 | England          | 2021-04-27 |         6680473
 K02000001 | United Kingdom   | 2021-04-27 |         6951081
(5 rows)

Time: 138.908 ms
```

Did you observed the difference?
The same query is now taking only 1/4th of the time that it was taking earlier on a single node machine.

Let's cross-check if the second query also shows similar behaviour.

```sql
SELECT 
area_type,
area_code,
MAX(date) AS date,
MAX((payload -> 'value')::FLOAT) AS first_dose
FROM (
	SELECT *
	FROM covid19.time_series AS tm
	JOIN covid19.release_reference AS rr ON rr.id = release_id
	JOIN covid19.metric_reference AS mr ON mr.id = metric_id
	JOIN covid19.area_reference AS ar ON ar.id = tm.area_id
	 ) AS ts
WHERE date > (now() - INTERVAL '30 days')
AND metric = 'cumPeopleVaccinatedFirstDoseByPublishDate'
AND (payload -> 'value') NOTNULL
GROUP BY area_type, area_code;
```
Output:
```text
 area_type | area_code |    date    | first_dose
-----------+-----------+------------+------------
 nation    | E92000001 | 2021-05-03 |   29025049
 overview  | K02000001 | 2021-05-03 |   34667904
 nation    | S92000003 | 2021-05-03 |    2833761
 nation    | W92000004 | 2021-05-03 |    1864400
 nation    | N92000002 | 2021-05-03 |     944694
(5 rows)

Time: 205.218 ms
```

For this query as well, we see similar improvements in the overall run time. 
As you can see, we've got perfectly normal SQL running in a distributed environment with no changes to our actual queries. This is a very powerful tool for scaling PostgreSQL to any size you need without dealing with the traditional complexity of distributed systems.

## Next steps

If you do not want to keep and continue to be billed for the Azure database for Postgres-Hyperscale (Citus) server that we provisioned at the beginning of the lab, you can [delete](https://docs.microsoft.com/azure/postgresql/howto-hyperscale-read-replicas-portal#:~:text=To%20delete%20a%20server%20group,Select%20Delete.) it via the Azure Portal.

You have successfully completed this lab. If you are interested in learning more about Hyperscale (Citus) please refer to our [Quickstart](https://docs.microsoft.com/azure/postgresql/hyperscale/) guide.
