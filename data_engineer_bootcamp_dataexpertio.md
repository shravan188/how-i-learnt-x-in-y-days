# Data Engineer bootcamp by DataExpert-io (Data with Zach)


## Day N
### Duration : 1.5 hours

### Learnings
* Partition : Atomic chunk of data (which is stored on a node in a cluster)
* RDD (Resilient Distributed dataset) : A large dataset split across multiple machines/nodes. Each unit which the data is split into is called a partition
* Repartition : Change number of partitions data is divided into. Done to improve performance (more details later). Can be done in 2 ways : 
    * repartion (recreates new partitions from scratch, can increase or decrease total no. of partitions) 
    * coalesce (combines existing partitions to create larger partitions, decreases total no. of partitions)

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

### References
1. https://medium.com/@zaiderikat/apache-spark-repartitioning-101-f2b37e7d8301
2. https://www.geeksforgeeks.org/difference-between-memory-and-hard-disk/
3. https://superuser.com/questions/1696557/what-in-the-hardware-makes-ram-faster-than-drive
4. https://stackoverflow.com/questions/40732962/spark-rdd-is-partitions-always-in-ram?rq=1
5. https://stackoverflow.com/questions/38798567/rename-more-than-one-column-using-withcolumnrenamed
6. https://stackoverflow.com/questions/58286502/spark-repartitioning-by-column-with-dynamic-number-of-partitions-per-column
7. https://iceberg.apache.org/docs/1.7.0/spark-ddl/