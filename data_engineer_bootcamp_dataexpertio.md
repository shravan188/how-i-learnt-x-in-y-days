# Data Engineer bootcamp by DataExpert-io (Data with Zach)

## Day 1

### Duration : 1 hour

### Learnings

* pgadmin : A tool for managing postgressql. It provides a graphical interface for creating, maintaining and using database objects

* Used docker compose to start docker container with this (docker_compose.yml)[https://github.com/DataExpert-io/data-engineer-handbook/blob/main/bootcamp/materials/1-dimensional-data-modeling/docker-compose.yml], then opened http://localhost:5050 to open pgadmin

```
cd bootcamp\materials\1-dimensional-data-modeling

docker compose up -d

# get running containers and their id
docker ps

# 
docker exec -it <container_id> bash 

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
### Duration : 1 hour

### Learnings
* Bucketing is used primarily to improve the performance of join operations. Before you perform a join, we have to bucket the records **on the join column**

* bucketBy became available in version 2.3 of Spark

* On doing saveAsTable, spark-warehouse folder is created

```
from pyspark.sql import SparkSession
# create spark session
spark = SparkSession.builder.appName("homework").getOrCreate()

matches = spark.read.option("header","true").csv("matches.csv")
match_details = spark.read.options("header", "true").csv("match_details.csv")

matches.write.bucketBy(16, "match_id").saveAsTable("matches_bucketed", format="parquet")
match_details.write.bucketBy(16, "match_id").saveAsTable("match_details_bucketed", format="parquet")

# spark.table same as spark.read.table
# load the bucketed tables
matches_bucketed = spark.table("matches_bucketed")
match_details_bucketed = spark.table("match_details_bucketed")

matches_bucketed.join(match_details_bucketed, matches_bucketed.match_id == match_details_bucketed.match_id, "left").count()

```

### Doubts

1. How does bucketing happen and is there any way to check the bucketing in the parquet files?
2. Should we sort the table after bucketing and if so why?
3. Should we save the bucketed table using saveAsTable or can we use the bucketed data directly without saving as table?
4. How does repartitioning before bucketing help optimize bucketing?

### References
1. https://www.reddit.com/r/GoogleColab/comments/1edmrxl/how_to_turn_off_code_complete_in_notebooks/?rdt=49194
2. https://luminousmen.com/post/the-5-minute-guide-to-using-bucketing-in-pyspark/
3. https://stackoverflow.com/questions/73346771/why-does-spark-re-sort-the-data-when-the-join-of-the-two-tables-are-bucketed-and
