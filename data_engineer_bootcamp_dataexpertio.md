# Data Engineer bootcamp by DataExpert-io (Data with Zach)

## Day 1

### Duration : 1 hour

### Learnings

* pgadmin : A tool for managing postgressql. It provides a graphical interface for creating, maintaining and using database objects

* Used docker compose to start docker container with this (docker_compose.yml)[https://github.com/DataExpert-io/data-engineer-handbook/blob/main/bootcamp/materials/1-dimensional-data-modeling/docker-compose.yml], then opened http://localhost:5050 to open pgadmin. Logged into using .env variables (postgres@postgres.com and postgres)

```
cd bootcamp\materials\1-dimensional-data-modeling

docker compose up -d

# get running containers and their id
docker ps

# 
docker exec -it <container_id> bash 

# 
pg_restore -U $POSTGRES_USER -d $POSTGRES_DB /docker-entrypoint-initdb.d/data.dump


psql -U postgres

\l

# connect to postgres database
\c postgres

\dt

select count(*) from events;

# exit psql
\q

# exit docker container
exit

```
* pg_restore : Allows you to restore a database from a .dump file (which is a logical backup file created using the command pg_dump)


### Doubts
1. What are volumes in docker and why do we use the -v flag when we do docker compose down?

### References

1. https://stackoverflow.com/questions/37694987/connecting-to-postgresql-in-a-docker-container-from-outside
2. https://stackoverflow.com/questions/129445/postgresql-psql-i-how-to-execute-script-in-a-given-path
3. https://www.timescale.com/learn/a-guide-to-pg_restore-and-pg_restore-example
4. https://simplebackups.com/blog/postgresql-pgdump-and-pgrestore-guide-examples/

## Day 2
### Learnings
Watched course lectures - Dimensional modeling theory and Lab

### Doubts
1. Why do we need separate data modeling for transactional and analytical processing?
2. What is an OLAP cube?
3. What is the difference bw struct, map and array?
4. What is cumulative data design? What is the compactness vs usability tradeoff?
5. What are the different scd types and how is it differernt from daily snapshots?
6.


## Day 3
### Duration : 0.5 hours
### Learnings
* Setup postgres and pgadmin 4 with docker using steps in day 1 

* Created new server connection with Host Name : host.docker.internal

* Viewed existing tables by opening Servers > pgtest > Databases > postgres > schemas > public > tables in the Object Explorer pane

* Opened Query Tool and then ran the following SQL queries

```
-- to see postgres version
select version();

select * from actor_films;

```

### Doubts
1. What does host name host.docker.internal mean?

### References
1. https://www.youtube.com/watch?v=UjQiwonRMas

## Day 4
### Duration : 3 hours

### Learnings
* Postgres has multiple built in types. Some of the main ones are
   * INTEGER, REAL (numeric types)
   * BOOLEAN
   * CHAR, VARCHAR, TEXT (character types)
   * DATE, TIME, TIMESTAMP (temporal types)
   * Array
   * User defined types (including enum)

* We can create PostgreSQL user-defined data type using CREATE DOMAIN or CREATE TYPE statements. CREATE TYPE is used to create a composite type

* Enum type is a custom data type that allows you to define a list of possible values for a column.If you attempt to insert or update a row with a value not in the list, PostgreSQL will issue an error. When we do ORDER BY with an enum field, it orders the rows based on the order in which we list them when we define the enum.

*

* The goal of this exercise is to create a new cumulative table called players which has only 1 row per player - and all the fields that change or keep getting added with time such as season, pts, assists, rebounds, etc are stored as an array in a single column. This new table design helps compress information and makes joins easier

```
-- create a custom type that contains all the temporal fields i.e. fields that change with time
CREATE TYPE season_stats AS (
      season INTEGER,
      pts REAL,
      ast REAL,
      reb REAL,
      weight INTEGER
)

-- 
CREATE TYPE scoring_class as ENUM('bad', 'average', 'good', 'star')

-- seasons field stores an array of season_stats, so similar to an array of tuples in python
-- scoring_class, years_since_last_active, is_active and current_season are derived columns
CREATE TABLE players (
     player_name TEXT,
     height TEXT,
     college TEXT,
     country TEXT,
     draft_year TEXT,
     draft_round TEXT,
     draft_number TEXT,
     seasons season_stats[],
     scoring_class scoring_class,
     years_since_last_active INTEGER,
     is_active BOOLEAN,
     current_season INTEGER,
     PRIMARY KEY (player_name, current_season)
)

-- today cte is the seed query, as it populates the database with initial data
-- we have to run this repeatedly with successive years to get the entire data
WITH yesterday AS (
   select * from players
   where current_season = 1995
),
today AS (
   select * from player_seasons
   where season = 1996
)
INSERT INTO players
SELECT
   COALESCE(y.player_name, t.player_name) as player_name,
   COALESCE(y.height, t.height) as height,
   COALESCE(y.college, t.college) as college,
   COALESCE(y.country, t.country) as country,
   COALESCE(y.draft_year, t.draft_year) as draft_year,
   COALESCE(y.draft_round, t.draft_round) as draft_round,
   COALESCE(y.draft_number, t.draft_number) as draft_number,
   COALESCE(y.seasons, ARRAY[]::season_stats[]) ||
            ARRAY[ROW(
               t.seasons,
               t.pts,
               t.ast,
               t.reb, 
               t.weight)::season_stats
            ]
            as seasons,
   CASE
             WHEN t.season IS NOT NULL THEN
                 (CASE WHEN t.pts > 20 THEN 'star'
                    WHEN t.pts > 15 THEN 'good'
                    WHEN t.pts > 10 THEN 'average'
                    ELSE 'bad' END)::scoring_class
             ELSE y.scoring_class
      END,

   CASE WHEN t.season IS NOT NULL THEN 0
   ELSE y.years_since_last_active + 1
   END as years_since_last_active,
   t.season IS NOT NULL as is_active,
   COALESCE(t.season, y.current_season + 1) as current_season,

FROM yesterday y FULL OUTER JOIN today t
ON y.player_name = t.player_name

```
### Doubts
1. What are functions in Postgres and how do we define and use them?
2. Why is the order in coalesce previous followed by current?


### References
1. https://neon.tech/postgresql/postgresql-tutorial/postgresql-data-types


## Day N
### Duration : 1.5 hours

### Learnings
* Partition : Atomic chunk of data (which is stored on a node in a cluster)
* Partitioning enables parallelism i.e. a data transformation can be performed on multiple data chunks in parallel, thus job finishes faster
* RDD (Resilient Distributed dataset) : A large dataset split across multiple machines/nodes. Each unit which the data is split into is called a partition
* Repartition : Change number of partitions data is divided into. Done to improve performance (more details later). Can be done in 2 ways : 
    * repartion (recreates new equal sized partitions from scratch, can increase or decrease total no. of partitions, involves shuffling across nodes) 
    * coalesce (combines existing partitions to create larger partitions, decreases total no. of partitions, no shuffling)
    (check interview questions in 11)


* Memory/inmemory vs disk : Memory is where computer stores data temporarily, while storage is where you save files permanently. Memory is much faster than other forms of storage, such as a disk (or even ssd) due to various reasons (refer 3)

* Spark does all computation in memory

* Forked repo https://github.com/DataExpert-io/data-engineer-handbook and cloned it locally. Uploaded devices.csv, events.csv and event_data_pyspark.ipynb on Databricks using Databricks community edition. Since DataBricks uses DBFS(databricks file system), had to replace file paths with /FileStore/tables/events.csv and /FileStore/tables/devices.csv

* Since pyspark 3.4.0, you can use the withColumnsRenamed() method to rename multiple columns at once. For version before that, we can either use withColumnRenamed() multiple times or use select as shown below (along with get function of dictionary)

```
from pyspark.sql import SparkSession
from pyspark.sql.functions import expr, col

spark = SparkSession.builder.appName("Jupyter").getOrCreate()

events = spark.read.option("header", "true").csv("/FileStore/tables/events.csv").withColumn("event_date", expr("DATE_TRUNC('day', event_time)"))
devices = spark.read.option("header","true").csv("/FileStore/tables/devices.csv")

df = events.join(devices, on="device_id", how="left")

mapping = dict(zip(['browser_type', 'os_type'], ['browser_family', 'os_family']))
mapping = {'browser_type': 'browser_family', 'os_type': 'os_family'}

# if the column name is not in mapping, just return column name, else return the mapped one
# this is where .get function of dictionary is so useful
df = df.select([col(c).alias(mapping.get(c, c)) for c in df.columns])

print(df.count()) # 404814
```

* Spark 3 can create tables in any Iceberg catalog with the clause USING iceberg. To use Iceberg in Spark, first configure Spark catalogs. (refer 7)

### Doubts
1. What exactly is special about HDFS and DBFS?
2. How exactly does repartition work when we specify a column name, especially when number of categories in the column is less than number of partitions? (refer 6)
3. When we create database using spark where and how is it stored? How does it compare to a postgres database? What is difference b/w operational and analytoc workload?

### References
1. https://medium.com/@zaiderikat/apache-spark-repartitioning-101-f2b37e7d8301
2. https://www.geeksforgeeks.org/difference-between-memory-and-hard-disk/
3. https://superuser.com/questions/1696557/what-in-the-hardware-makes-ram-faster-than-drive
4. https://stackoverflow.com/questions/40732962/spark-rdd-is-partitions-always-in-ram?rq=1
5. https://stackoverflow.com/questions/38798567/rename-more-than-one-column-using-withcolumnrenamed
6. https://stackoverflow.com/questions/58286502/spark-repartitioning-by-column-with-dynamic-number-of-partitions-per-column
7. https://iceberg.apache.org/docs/1.7.0/spark-ddl/
8. https://spark.apache.org/docs/3.5.3/sql-ref-syntax-ddl-create-database.html
9. https://www.reddit.com/r/dataengineering/comments/1g03cyw/how_does_spark_compare_to_postgres_in_large_scale/
10. https://stackoverflow.com/questions/76242658/if-spark-isnt-a-storage-system-how-do-tables-work
11. https://www.java-success.com/5-apache-spark-coalesce-vs-repartition-scenarios-interview-qas/


## Day N + 1

* Repartitioning : Redistributing the data across different partitions in a spark rdd

* Repartitioning using column : When we reparition using column, we may get skewed partitions since we are using the column values to do the partitioning

```
from pyspark.sql.functions import spark_partition_id


os_df = spark.createDataFrame(["Windows","Windows","Windows","Windows","Windows","Windows","Windows","Linux","Mac","Mac"], "string").toDF("os")

os_df = os_df.withColumn("partitionId", spark_partition_id())
print(os_df.rdd.getNumPartitions()) # 8
os_df.show() # partition ids from 0 to 7

os_df = os_df.repartition(2)
print(os_df.rdd.getNumPartitions()) # 2
os_df = os_df.withColumn("partitionId", spark_partition_id())
os_df.show() # partition ids 0 (six rows, combination of all 3 categories) and 1 (four rows)

## repartition by column with 2 paritions
os_df = os_df.repartition(2, col("os"))
print(os_df.rdd.getNumPartitions()) #2
os_df = os_df.withColumn("partitionId", spark_partition_id())
os_df.show() # all windows and linux rows have partition id 0, mac has partition id 1

## repartition by column with 3 paritions

os_df = os_df.repartition(3, col("os"))
print(os_df.rdd.getNumPartitions()) # 3 (but 1 partition is empty)
os_df = os_df.withColumn("partitionId", spark_partition_id()) 
os_df.show() # linux row has index 0, all windows and 

```

* To distribute data across partitions spark needs somehow to convert value of the column to index of the partition. There are two default partitioners in Spark - HashPartitioner and RangePartitioner. We can also implement our own partitioner if we want better distribution (refer 3)

* Sorting data : There are 2 ways of sorting:
    * sort() : does a global sort i.e. sorts data across all partitions. Slower as it involves shuffling
    * sortWithinPartitions() : sorts data only within each partition. We can imagine a fourth column called partitionId as the primary sorting column

```
sorted = df.repartition(10, col("event_date"))\
    .sortWithinPartitions(col("event_date"), col("host"))\
    .withColumn("event_time", col("event_time").cast("timestamp")) 


sortedTwo = df.repartition(10, col("event_date"))\
    .sort(col("event_date"), col("host"))\
    .withColumn("event_time", col("event_time").cast("timestamp")) 


```
* If we do df.repartition(1).sortWithinPartitions() it is equivalent to global sort i.e. df.sort()

* We can use .explain() to see the query plan 

empty room tomorrow

### Doubts
1. How and when to implement custom partitioning in Pyspark?
2. How to implement SCD Type 1 upsert in Pyspark? How about SCD Type 2?
3. How to write code to detect empty partitions in Pyspark?
4. How does hash partitioning work for date columns or any non categorical column?
5. Does repartitioning create near equal sized partitions?

### References
1. https://stackoverflow.com/questions/47674311/how-to-create-a-sample-single-column-spark-dataframe-in-python
2. https://stackoverflow.com/questions/46032320/apache-spark-get-number-of-records-per-partition
3. https://stackoverflow.com/questions/50694848/why-do-i-get-so-many-empty-partitions-when-repartionning-a-spark-dataframe
4. https://stackoverflow.com/questions/66534193/how-does-sortwithinpartitions-sort
5. https://stackoverflow.com/questions/31610971/spark-repartition-vs-coalesce


## Day N + 2

### Duration : 1 hour

### Learnings

* Join strategies in Spark when we join two datasets : shuffle join, broadcast join

* Broadcast join : When one of the 2 tables to be joined is small, that small table is sent to all the worker nodes in the cluster. This allows join to be performed locally within each node (thus avoiding redistributing data amongst nodes i.e. shuffling)

* spark.sql.autoBroadcastJoinThreshold : This threshold is the maximum size in bytes for a table to be automatically broadcast by spark to all worker nodes. Default value is 10485760 i.e. 10MB. By setting this value to -1, broadcasting can be disabled.

* We can explicitly perform broadcast join using the broadcast function

```
from pyspark.sql import SparkSession
# create spark session
spark = SparkSession.builder.appName("homework").getOrCreate()
# disable automatic broadcast join
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)
# check if configuration is done as expected
print(spark.conf.get("spark.sql.autoBroadcastJoinThreshold"))

maps = spark.read.option("header","true").csv("maps.csv")
medals = spark.read.option("header","true").csv("medals.csv")
matches = spark.read.option("header","true").csv("matches.csv")

print(maps.count(), medals.count(), matches.count())

matches = matches.join(maps, matches.mapid == maps.mapid, "left")
matches.explain()

from pyspark.sql.functions import broadcast
maps = spark.read.option("header","true").csv("maps.csv")
matches = spark.read.option("header","true").csv("matches.csv")
matches = matches.join(broadcast(maps), matches.mapid == maps.mapid, "left")
matches.explain()

```

* The different query plans we get are as follows

```
### Without broadcast join
== Physical Plan ==
AdaptiveSparkPlan isFinalPlan=false
+- SortMergeJoin [mapid#762], [mapid#697], LeftOuter
   :- Sort [mapid#762 ASC NULLS FIRST], false, 0
   :  +- Exchange hashpartitioning(mapid#762, 200), ENSURE_REQUIREMENTS, [plan_id=1112]
   :     +- FileScan csv [match_id#761,mapid#762,is_team_game#763,playlist_id#764,game_variant_id#765,is_match_over#766,completion_date#767,match_duration#768,game_mode#769,map_variant_id#770] Batched: false, DataFilters: [], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/content/matches.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<match_id:string,mapid:string,is_team_game:string,playlist_id:string,game_variant_id:string...
   +- Sort [mapid#697 ASC NULLS FIRST], false, 0
      +- Exchange hashpartitioning(mapid#697, 200), ENSURE_REQUIREMENTS, [plan_id=1113]
         +- Filter isnotnull(mapid#697)
            +- FileScan csv [mapid#697,name#698,description#699] Batched: false, DataFilters: [isnotnull(mapid#697)], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/content/maps.csv], PartitionFilters: [], PushedFilters: [IsNotNull(mapid)], ReadSchema: struct<mapid:string,name:string,description:string>


### With broadcast join
== Physical Plan ==
AdaptiveSparkPlan isFinalPlan=false
+- BroadcastHashJoin [mapid#890], [mapid#866], LeftOuter, BuildRight, false
   :- FileScan csv [match_id#889,mapid#890,is_team_game#891,playlist_id#892,game_variant_id#893,is_match_over#894,completion_date#895,match_duration#896,game_mode#897,map_variant_id#898] Batched: false, DataFilters: [], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/content/matches.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<match_id:string,mapid:string,is_team_game:string,playlist_id:string,game_variant_id:string...
   +- BroadcastExchange HashedRelationBroadcastMode(List(input[0, string, false]),false), [plan_id=1172]
      +- Filter isnotnull(mapid#866)
         +- FileScan csv [mapid#866,name#867,description#868] Batched: false, DataFilters: [isnotnull(mapid#866)], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/content/maps.csv], PartitionFilters: [], PushedFilters: [IsNotNull(mapid)], ReadSchema: struct<mapid:string,name:string,description:string>




```

### Doubts

1. What is a worker node?


### References
1. https://spark.apache.org/docs/3.5.3/sql-performance-tuning.html
2. https://aspinfo.medium.com/what-is-broadcast-join-how-to-perform-broadcast-in-pyspark-699aef2eff5a
3. https://github.com/DataExpert-io/data-engineer-handbook/tree/main/bootcamp/materials/3-spark-fundamentals

## Day N + 3
### Duration : 1.5 hours

### Learnings
Watched course lectures
    * Spark + Iceberg in 1 Hour - Memory Tuning, Joins, Partition 
    * High Performance Spark in 1 hour - DataFrame, Dataset, UDFs, Caching

### Doubts
1. How is bucketing different from partitioning?
2. What is executor and how is it related to the partitions created?
3. What is salting and how exactly does it solve the skew issue?
4. Is the main selling point of Databricks using notebooks in production instead of spark submit?
5. Why do api call happen on driver?
6. While bucketing why does overwrite not work but append does?

### References
1. https://stackoverflow.com/questions/19128940/what-is-the-difference-between-partitioning-and-bucketing-a-table-in-hive

## Day N + 4
### Duration : 1.75 hour

### Learnings
* Bucketing is used primarily to improve the performance of join operations. Before you perform a join, we have to bucket the records **on the join column**

* Spark applies a hash function to the specified column and divides the data into buckets corresponding to the hash values.

* bucketBy became available in version 2.3 of Spark

* Number of bucketing files = number of buckets * number of partitions

* Number of partitions depends on the cluster manager, the default parallelism is the number of CPU cores. To find number of cpu cores, we can use the multiprocessing library as follows

```
import multiprocessing

cores = multiprocessing.cpu_count() # Count the number of cores in a computer
print(cores)


```

* On doing saveAsTable, spark-warehouse folder is created

```
from pyspark.sql import SparkSession
# create spark session
spark = SparkSession.builder.appName("homework").getOrCreate()

matches = spark.read.option("header","true").csv("matches.csv")
match_details = spark.read.options("header", "true").csv("match_details.csv")

# creates 16 buckets based on match_id which is the join column
matches.write.bucketBy(16, "match_id").saveAsTable("matches_bucketed", format="parquet")
match_details.write.bucketBy(16, "match_id").saveAsTable("match_details_bucketed", format="parquet")

print(matches.rdd.getNumPartitions()) # 2 
# Google Colab has 2 cores by default, hence default parallelism is 2
# Total number of bucketing files created = num partitions * num buckets = 2 * 16 = 32
# Hence 32 files created in match_details_bucketed folder (excluding _SUCCESS file) inside spark-warehouse folder (refer 5)

# spark.table same as spark.read.table
# load the bucketed tables
matches_bucketed = spark.table("matches_bucketed")
match_details_bucketed = spark.table("match_details_bucketed")

matches_bucketed.join(match_details_bucketed, matches_bucketed.match_id == match_details_bucketed.match_id, "left").count()

```



```

== Physical Plan ==
AdaptiveSparkPlan isFinalPlan=false
+- SortMergeJoin [match_id#160], [match_id#197], LeftOuter
   :- Sort [match_id#160 ASC NULLS FIRST], false, 0
   :  +- Exchange hashpartitioning(match_id#160, 200), ENSURE_REQUIREMENTS, [plan_id=678]
   :     +- FileScan csv [match_id#160,mapid#161,is_team_game#162,playlist_id#163,game_variant_id#164,is_match_over#165,completion_date#166,match_duration#167,game_mode#168,map_variant_id#169] Batched: false, DataFilters: [], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/content/matches.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<match_id:string,mapid:string,is_team_game:string,playlist_id:string,game_variant_id:string...
   +- Sort [match_id#197 ASC NULLS FIRST], false, 0
      +- Exchange hashpartitioning(match_id#197, 200), ENSURE_REQUIREMENTS, [plan_id=679]
         +- Filter isnotnull(match_id#197)
            +- FileScan csv [match_id#197,player_gamertag#198,previous_spartan_rank#199,spartan_rank#200,previous_total_xp#201,total_xp#202,previous_csr_tier#203,previous_csr_designation#204,previous_csr#205,previous_csr_percent_to_next_tier#206,previous_csr_rank#207,current_csr_tier#208,current_csr_designation#209,current_csr#210,current_csr_percent_to_next_tier#211,current_csr_rank#212,player_rank_on_team#213,player_finished#214,player_average_life#215,player_total_kills#216,player_total_headshots#217,player_total_weapon_damage#218,player_total_shots_landed#219,player_total_melee_kills#220,... 12 more fields] Batched: false, DataFilters: [isnotnull(match_id#197)], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/content/match_details.csv], PartitionFilters: [], PushedFilters: [IsNotNull(match_id)], ReadSchema: struct<match_id:string,player_gamertag:string,previous_spartan_rank:string,spartan_rank:string,pr...


== Physical Plan ==
AdaptiveSparkPlan isFinalPlan=false
+- SortMergeJoin [match_id#371], [match_id#391], LeftOuter
   :- Sort [match_id#371 ASC NULLS FIRST], false, 0
   :  +- FileScan parquet spark_catalog.default.matches_bucketed[match_id#371,mapid#372,is_team_game#373,playlist_id#374,game_variant_id#375,is_match_over#376,completion_date#377,match_duration#378,game_mode#379,map_variant_id#380] Batched: true, Bucketed: true, DataFilters: [], Format: Parquet, Location: InMemoryFileIndex(1 paths)[file:/content/spark-warehouse/matches_bucketed], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<match_id:string,mapid:string,is_team_game:string,playlist_id:string,game_variant_id:string..., SelectedBucketsCount: 16 out of 16
   +- Sort [match_id#391 ASC NULLS FIRST], false, 0
      +- Filter isnotnull(match_id#391)
         +- FileScan parquet spark_catalog.default.match_details_bucketed[match_id#391,player_gamertag#392,previous_spartan_rank#393,spartan_rank#394,previous_total_xp#395,total_xp#396,previous_csr_tier#397,previous_csr_designation#398,previous_csr#399,previous_csr_percent_to_next_tier#400,previous_csr_rank#401,current_csr_tier#402,current_csr_designation#403,current_csr#404,current_csr_percent_to_next_tier#405,current_csr_rank#406,player_rank_on_team#407,player_finished#408,player_average_life#409,player_total_kills#410,player_total_headshots#411,player_total_weapon_damage#412,player_total_shots_landed#413,player_total_melee_kills#414,... 12 more fields] Batched: true, Bucketed: true, DataFilters: [isnotnull(match_id#391)], Format: Parquet, Location: InMemoryFileIndex(1 paths)[file:/content/spark-warehouse/match_details_bucketed], PartitionFilters: [], PushedFilters: [IsNotNull(match_id)], ReadSchema: struct<match_id:string,player_gamertag:string,previous_spartan_rank:string,spartan_rank:string,pr..., SelectedBucketsCount: 16 out of 16




```


### Doubts

1. How does bucketing happen and is there any way to check the bucketing in the parquet files?
2. Should we sort the table after bucketing and if so why?
3. Should we save the bucketed table using saveAsTable or can we use the bucketed data directly without saving as table?
4. How does repartitioning before bucketing help optimize bucketing?
5. What is bucket pruning?

### References
1. https://www.reddit.com/r/GoogleColab/comments/1edmrxl/how_to_turn_off_code_complete_in_notebooks/?rdt=49194
2. https://luminousmen.com/post/the-5-minute-guide-to-using-bucketing-in-pyspark/
3. https://stackoverflow.com/questions/73346771/why-does-spark-re-sort-the-data-when-the-join-of-the-two-tables-are-bucketed-and
4. https://medium.com/@diehardankush/what-all-about-bucketing-and-partitioning-in-spark-bc669441db63
5. https://books.japila.pl/spark-sql-internals/bucketing/#creating-bucketed-tables
6. https://stackoverflow.com/questions/67135876/how-many-cpu-cores-does-google-colab-assigns-when-i-keep-n-jobs-8-is-there-an
7. https://www.reddit.com/r/apachespark/comments/1d57daf/how_to_decide_optimal_number_of_partitions/
8. https://aspinfo.medium.com/how-to-improve-performance-with-bucketing-in-pyspark-c66a899e70b5