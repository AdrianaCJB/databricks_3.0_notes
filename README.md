# Databricks Spark 3.0 Notes

This is my study notes and summary about the Databricks Certified Associate Developer for Apache Spark 3.0 - Python


# Summary

The Databricks Certified Associate Developer for Apache Spark 3.0certification exam assesses an understanding of the basics of the Spark architecture and the ability to apply the Spark DataFrame API to complete individual data manipulation tasks.

# Prerequisites

- have a basic understanding of the Spark architecture, including Adaptive Query Execution (AQE)
- be able to apply the Spark DataFrame API to complete individual data manipulation task, including: 
  - selecting, renaming and manipulating columns
  - filtering, dropping, sorting, and aggregating rows
  - joining, reading, writing and partitioning DataFrames
  - working with UDFs and Spark SQL functions
  
# Spark Fundamentals

## Spark's Basic Architecture

- Application: A user program built on Spark using its APIs.

- SparkSession: An object that provides a point of entry to interact with underlying Spark functionality and allows programming Spark with its APIs. You control your Spark Application through a driver process called the SparkSession.

- Driver: Is the machine in which the application runs. Runs `main()` function. It is responsible for:
  - maintaining information about spark application.
  - respond to user's program or input.
  - analizing, distributing, and scheduling work across the executors in parallel.

- Executors: Responsible for:
  - executing code assigned by the driver.
  - reporting state back to the driver.

- Jobs: In general, there should be one Spark job for one action. Actions always return results. Each job breaks down into a series of stages, the number of which depends on how many shuffle operations need to take place.

**Note: During interactive sessions with Spark shells, the driver converts the Spark application into one or more Spark jobs. It then transforms each job into a
`DAG`. This, in essence, is Spark’s execution plan, where each node within a DAG
could be a single or multiple Spark stages.**

- Stages: Each job gets divided into smaller sets of tasks called stages that depend on each other. A stage represent groups of tasks that can be executed together to compute the same operation on multiple machines. 

- Tasks: A task is just a single unit of work or execution that applied to a unit of data (the partition). Each task corresponds to a combination of blocks of data and a set of transformations that will run on a single executor. If there is one big partition in our dataset, we will have one task. If there are 1,000 little partitions, we will have 1,000 tasks that can be executed in parallel.

- Slots: Each executor has a number of slots. Then, tasks are assigned to slots for parallel execution.

- Partition: A chuck of data (collection of rows) that sit on a physical machine in a cluster.
  - A good rule of thumb is that the number of partitions should be larger than the number of executors on your cluster, potentially by multiple factors depending on the workload.
  
- Shuffles: Are triggered when data needs to move between executors. Represents a physical repartitioning of the data and requires coordinating across executors to move data around.
    - The spark.sql.shuffle.partitions default value is 200, which means that when there is a shuffle performed during execution, it outputs 200 shuffle partitions by default. 
    - You can change this value, and the number of output partitions will change.
`spark.conf.set("spark.sql.shuffle.partitions", 5)`

## Spark Concepts

### Transformations, Actions and Lazy Evaluation

Spark operations on distributed data can be classified into two types: transformations and actions.

- Transformation: transform a Spark DataFrame into a new DataFrame without altering the original data (each transformation creates a new structure), giving it the property of **immutability**.

Two types of transformations:
  - Narrow dependencies: Any transformation where a single output partition can be computed from a single input partition. 
    - Also called **pipelining**
    - performed **in-memory**
    - e.g: `where(), filter() and contains()`
  - Wide dependencies:
    - Also called **shuffle**
    - data from other partitions is read in, combined, and written to disk. 
    - performed **on disk**
    - e.g: `count(), sort(), groupBy() and orderBy()`

- Actions: An action instructs Spark to compute a result from a series of transformations. Nothing in a query plan is executed until an action is
invoked. There are three kinds of actions:
  - Actions to view data in the console.
  - Actions to collect data to native objects in the respective language.
  - Actions to write to output data sources.

    | Transformations |  Actions  |
    | --------------- | --------- |
    | orderBy()       | show()    |
    | groupBy()       | take()    |
    | filter()        | count()   |
    | select()        | collect() |
    | join()          | save()    |

- Lazy Evaluation: Refers to the idea that Spark waits until the last moment to execute a series of operations. That is, their results are not computed immediately, but they are recorded or remembered as a lineage. Allows to rearrange certain transformations. 
  - The plan is executed when you call an action. 
  - The expression build a logical plan represented by a directed acyclic graph (DAG).



## Spark's Structured APIs

### RDD

The RDD is the most basic abstraction in Spark. There are three vital characteristics associated with an RDD:
- Dependencies
- Partitions (with some locality information)
- Compute function: Partition => Iterator[T]




## DataFrames API 

**- SparkContext**
  - Candidates are expected to know how to use the SparkContext to control basic configuration settings such as spark.sql.shuffle.partitions.

**- SparkSession**
  - Create a DataFrame/Dataset from a collection (e.g. list or set). Dataset is only for Scala and Java.
  - Create a DataFrame for a range of numbers. e.g: `spark.range(10).toDF("value")`
  - Access the DataFrameReaders. e.g: `spark.read.format("<format>").load("<path>")`
  - Register User Defined Functions (UDFs). e.g: `spark.udf.register(<name_udf_sql>, <name_funtion>, <output_datatype>)`


**- DataFrameReader**
  - Read data for the "core" data formats (CSV, JSON, JDBC, ORC, Parquet, text and tables). 
  e.g: 
  `spark.read.csv("<path>")`, <br>
  `spark.read.json("<path>")`, <br>
  `spark.read.jdbc("<path>")`, <br>
  `spark.read.orc("<path>")`, <br>
  `spark.read.parquet("<path>")`, <br>
  `spark.read.text("<path>")`
  - How to configure options for specific formats. e.g: 
```
spark.read.option("inferSchema","true")
          .option("header","true")
          .option("sep",";")    or   .option("delimiter", "\t")
          .option("timestampFormat","mm/dd/yyyy hh:mm:ss a")
```
  - How to read data from non-core formats using format() and load(). e.g: `spark.read.format("<format>").load("<path>")`
  - Read from the database by passing the URL, table name, and connection properties into. e.g: 
  ```
  spark.read.jdbc(url, table, column=None, 
      lowerBound=None, upperBound=None, numPartitions=None, 
      predicates=None, properties=None)
  ```
  Properties is a dictionary of JDBC database at least properties "user" and "password". For example { ‘user’ : ‘SYSTEM’, ‘password’ : ‘mypassword’ }
  - How to specify a DDL-formatted schema. e.g: `schema = "name STRING, title STRING, pages INT"`
  - How to construct and specify a schema using the StructType classes. e.g: 
  ```
  schema = StructType ([ 
              StructField("name", StringType(), False),
              StructField("title", StringType(), False),
              StructField("pages", IntegerType(), False) ])
  ```
  
  - JSON inside other JSON in a StructType class: 
  
    e.g. JSON file:
    ```
    { "name" : 
        { "firstName": "Adriana",
          "lastName": "Jimenez"
         }
    }     
    ```
    e.g. Schema JSON:
    ```
    schema = StructType ([ 
                StructField("name", StructType([
                    StructField("firstName", StringType(), False),
                    StructField("lastName", StringType(), False)
                ]), False)) ])
    ```
    
  - Corrupt Record Handling for CSV and JSON: 
    - PERMISSIVE: Includes corrupt records in a "_corrupt_record" column (by default)
    - DROPMALFORMED:Ignores all corrupted records
    - FAILFAST: Throws an exception when it meets corrupted records
  
    ```
    df = (spark.read
              .option("mode", "PERMISSIVE").json(<path>)
              .option("columnNameOfCorruptRecord", "_corrupt_record")
         )
    ```


**- DataFrameWriter**
  - Write data to the "core" data formats (csv, json, jdbc, orc, parquet, text and tables)
  - Overwriting existing files. e.g: `spark.write.mode("overwrite").parquet(<path>)`
  - How to configure options for specific formats. e.g: 
  ```
  df.write.format("csv").mode("overwrite").option("sep","\t").save("my-tsv-file.tsv")
  df.write.jdbc(newPath, table_name, mode="append", properties=props)
  ```
  - Saving Dataframe as a SQL table: `spark.write.format("parquet").saveAsTable(parquet_table)`
  - How to write a data source to 1 single file or N separate files. 
  - Writing data in parallel - Partitioning by column:
  `df.write.mode("overwrite").partitionBy("DEST_COUNTRY_NAME").save(partitioned_files_csv_in_parquet.parquet)`
  - How to write partitioned data. e.g: `spark.write.repartition(N).mode("overwrite").parquet(<path>)`
  - How to bucket data by a given set of columns. (Bucketing is supported only for Spark-managed tables). e.g:
  ```
  numberBuckets = 10
  columnToBucketBy = "count"

   ## It will save in " /user/hive/warehouse/bucketedfiles/part-000-....snappy.parquet " 
  df.write.format("parquet").mode("overwrite").bucketBy(numberBuckets, columnToBucketBy).saveAsTable("bucketedFiles")
  ```

**- Manipulating Dataframe**
  - Columns and expressions: `col("name")` , `expr("salary * 5")`
  - Selecting columns:
  ```
  df.select("col1","col2").show()
  df.selectExpr("*", "(DEST_COUNTRY_NAME = ORIGIN_COUNTRY_NAME) as compare").show(5)
  df.selectExpr("avg(count) as average",
                "sum(count) as sum",
                "max(count) as maximum",
                "min(count) as minimum",
                "count(distinct(DEST_COUNTRY_NAME)) as 
                 count_distinct_dest_country").show(5)
  ```
  - Create columns:
  ```
  df.withColumn("newColumn", count * 100 )  
  df.withColumn("flag", expr("col1 == col2")) # return true or false
  ```
  - Rename columns: 
  ```
  df.withColumnRenamed("name", "newColumName")
  df.select("DEST_COUNTRY_NAME",
            "count", 
            expr("count") > lit(100))\
      .withColumnRenamed("(count > 100)","higherThan100")\
      .show(5)
  ```
  
  - Filtering columns:
  ```
  df.filter(col("DEST_COUNTRY_NAME") == "United States")
  df.where("count < 20")
  df.where(col("name") == 'DOT' & (col("avg") < 3 | col("avg") > 200))
  ```
  - Dropping: `df.drop("col")
  - Unique rows: 
  ```
  df.select("col1").distinct().show()
  df.select(countDistinct("col1")).show()
  ```
  - Cast columns: `df.select(col("count").cast("int").alias("countCast"))`
  - Sorting rows: 
  ```
  df.select("*").sort(expr("DEST_COUNTRY_NAME"))
  df.orderBy(desc("count"))
  df.orderBy(col("count").desc(),col("DEST_COUNTRY_NAME").asc()).show(5)
  ```
  - Limit rows: `df.orderBy(expr("count").asc()).limit(1).show()`
  - Using sample to extract random sample from a dataframe:
  ```
  withReplacement = False
  fraction = 0.3
  seed = 4

  df.sample(withReplacement, fraction, seed).show(5)
  df.sample(withReplacement, fraction, seed).count()
  ```
  - Union dataframes: `df1.union(df2)`
  - Creates or replaces a local temporary view with this DataFrame: `df.createOrReplaceTempView("complexDF")`


**- Window functions**
You can also use window functions to carry out some unique aggregations by either computing some aggregation on a specific “window” of data, which you define by using a reference to the current data. This window specification determines which rows will be passed in to this function.

`pyspark.sql.functions.window(timeColumn, windowDuration, slideDuration=None, startTime=None)`

Durations are provided as strings, e.g. ‘1 second’, ‘1 day 12 hours’, ‘2 minutes’. Valid interval strings are ‘week’, ‘day’, ‘hour’, ‘minute’, ‘second’, ‘millisecond’, ‘microsecond’. 

```
windowSpec = Window.partitionBy("CustomerId","date")\
                   .orderBy(desc("Quantity"))\
                   .rowsBetween(Window.unboundedPreceding, Window.currentRow)
                   
maxPurchaseQuantity = max(col("Quantity")).over(windowSpec)                   
```
**- Partitions**
  - Display the number of partitions in each DataFrame. e.g: `df.rdd.getNumPartitions()`
  - Set to up the number of partitions. e.g: `df.repartition(5)`
  - Set to down the number of partitions. e.g: `df.coalesce(2)`
  
**- Caching**
  - You can mark an RDD to be persisted using the persist() or cache() methods on it.
  - Default storage level is `StorageLevel.MEMORY_ONLY`
  - The available storage levels in Python include `MEMORY_ONLY, MEMORY_ONLY_2, MEMORY_AND_DISK, MEMORY_AND_DISK_2, DISK_ONLY, and DISK_ONLY_2`
  - To get the storage level: e.g `df.getStorageLevel()`
  - Once data is cached, the Catalyst optimizer will only reach back to the location where the data was cached. 
  - Cache: `df.cache()`
  - Uncache: `df.uncache()`


**- Manipulating Data**
  - Imputing null or missing data: There are some techniques such as dropping these records, adding a placeholder (e.g. -1 or -999), basic imputing by using the mean of non-missing data, and advanced imputing such as clustering ML algorithms or oversampling techniques. 
  e.g. `df.dropna("any")` , `df.na.fill({"col1" : 5})`
  - Deduplicating data: `df.dropDuplicates(["id"])`
  - Other helpful data manipultion functions: e.g `explode(), pivot(), cube(), rullop()`


## Spark SQL

### Catalyst Optimizer

Is at the core of Spark SQL's power and speed. It automatically finds the most efficient plan for applying your transformations and actions you called for in your code. Here’s an overview of the steps:

1. Write DataFrame/Dataset/SQL Code.
2. If valid code, Spark converts this to a Logical Plan.
3. Spark transforms this Logical Plan to a Physical Plan, checking for optimizations along
the way.
4. Spark then executes this Physical Plan (RDD manipulations) on the cluster.



## Util Commands in Databricks

- `%timeit` to compare .
- `%fs ls <path>` to list files inside of path. 
- `%fs head <file>` to print the bottom of file.

## The Spark UI 

In local mode, you can access this interface at http://<localhost>:4040 in a web
browser.
  
Running by default on port 4040, where you can view metrics and details such as:
- A list of scheduler stages and tasks
- A summary of RDD sizes and memory usage
- Information about the environment
- Information about the running executors
- All the Spark SQL queries


# What's new in Spark 3.0???

### * Performance

Achieve high performance for interactive, batch, streaming and ML workloads.

1. **Adaptative Query Execution:** Based on statistics of the finished plan nodes, re-optimize the execution plan of the remaining queries. Adaptative planning is between `Logical Plan` and finally `Query Execution`.

  - It's like a coalesce partition because the plan it adjust for tasks and stages, and ignore the 200 numbers of shuffle partitions by default.
  - Set `spark.conf.set("spark.sql.adaptative.enable","true")`
  
  1.1. Convert Sort Merge Join to Broadcast Hash join: 
  * Switch join strategies. 
    - Increase `spark.sql.autoBroadcastJoinThreshold` or use `broadcast()` hint.

  1.2. Shrink the number of reducers.
  * Dynamically Coalesce Shuffle Partitions:
    - Set the initial partition number high to accommodate the largest data size of the entire query execution. 
    - Automatically coalesce partitions if needed after each query stage.

  1.3. Handle skew join.
  * Dynamically Optimize Skew Joins:
    - Detect skew from partition sizes using runtime statistics.
    - Split skew partitions into smaller sub-partition.
  
**2. Dynamic Partition Pruning**

The goal of Dynamic Partition Pruning (DPP) is to allow you to read only as much data as you need.

  - Simply static partition pruning.
  - Star schema queries.
  - Denormalized queries.
  - Set `spark.sql.optimizer.dynamicPartitionPruning.enabled = true`
  
**3. Query Compilation Speedup**
  - Avoid partition scanning based on the query results of the other query fragments. 
  - Important for star-schema queries.
  - Significant speedup in TPC-DS

**4. Join Hints**

Join hints influence optimizer to choose the join strategies.
Should be used with extreme caution. Difficult to manage over time.

  - Broadcast hash join: Requeries one side to be small. No shuffle, no sort, very fast.
  
  - Shuffle Sort-merge join *(NEW)*: Robust. Can handle any data size. Need to shuffle and sort data, slower in some cases when table size is small.
  
  - Shuffle hash join *(NEW)*: Needs to shuffle data but no sort. Can handle large tables, bit will OOM too if data is skewed.
  
  - Shuffle nested loop join *(NEW)*: A Shuffle Nested Join (aka Cartesian Product Join) does not shuffle data. Instead, it does an all-pairs comparison between all join keys of each executor. This is useful for very small workloads. Doesn't require join keys.
  

### * Richer APIs

Enable new uses cases and simplify the Spark application development, new capabilities and new features.

1. **Accelerator-aware Scheduler:** Widely used for accelerating special workloads., e.g, deep learning and signal processing. Supports Standalone, YARNG, K8S. Supports GPU, FPGA, TPU. Needs to specify requires resources by configs. Application Level.

2. **Built-in Functions:** 32 new built-in functions in Scala.
Examples: map_filter, bit_count, count_if, acosh, map_zip_with, typeof, xxhash64, from_csv, date_part, bit_and, make_timestamp, min_by, make_interval, make_date, every, transform_values, etc.


3. **Pandas UDF Enhancements:**
  Supported type hints include: 
    - Series to Series
    - Iterator of Series to Iterator of Series
    - Iterator of Multiple Series to Iterator of Series
    - Series to Scalar (a single value)
  Supported function APIs include: 
    - Grouped Map
    - Map
    - Co-grouped Map
  
  Summary:
  - Scalar Pandas UDF [pandas.Series to pandas.Series]
  - Grouped Map Pandas Function API [pandas.Dataframe to pandas.Dataframe]
  - Grouped Aggregate Pandas UDF [pandas.Series to pandas.Scalar]

4. **DELETE/UPDATE/MERGE in Catalyst**



### * Monitoring ad Debbuggability

Make monitoring and debbugging in Spark applications more comprehensive and stable.

1. **Structured Streaming UI**
2. **Observable Metrics**
3. **Event Log Rollover**
  - For any stream you can access real-time metrics about:
    - Input rate
    - Process Rate
    - Input rows
    - Batch duration
    - Operation duration
  - Click over any graph to get details.

4. **DDL/DML Enhancements:**

4.1. Formatted Explain plan: You can access a formatted version of the explain plan with EXPLAIN FORMATTED. There have now a Header (shows the basic operating tree for the execution plan with numbers id for each one), a Footer (use the id number given in the header to trace identity of the operator and see additional features) and a subqueries.

  ```
  EXPLAIN FORMATTED
  SELECT *
  FROM table
  WHERE key = (SELECT ...)
  
  
  df.explain("FORMATTED")
  ```

4.2. SQL Improvements - Better ANSI SQL Compliance: 
  SQL Compatibility: 
  
  - Reduce the time and complexity of enabling applications that were written for other relational database products to run in Spark SQL.
  - ANSI SQL compliance is critical for workload migration from other SQL engines to Spark SQL.
  - ANSI Store Assignment.
  - Overflow checking.
  - Reserved Keywords in Parser.
  - To improve complience, switches to Proleptic Gregorian Calendar.
  - The following new options are experimental and off by default. Set `spark.sql.ansi.enabled = true` and `spark.sql.storeAssignmentPolicy = ANSI`
  

5. **Built-in Data Sources V2:**

Enhance the performance and functionalities pf the built-in data sources.
  - It provides unified APIs for streaming and batch processing of data sources, and supports pushdown and pruning for many common file sources, including ORC, Kafka, Cassandsa, Delta Lake, and Apache Iceberg. 
  - This release offers a catalog plugin API, which allows users to register customized catalogs and then use Spark to access/manipulate table metadata directly. 

  - Parquet/ORC Nested Column pruning
  - CSV Filter Pushdown
  - Parquet: Nested Column. Filter pushdown
  - New binary Data Source
  
### * Extensibility and Ecosystem

  Improve the plug-in interface and extend the deployment environments.
  
  - Data Source V2 API + Catalog Support
  - JDK 11 Support
  - Hadoop 3 Support
  - Hive 2.x Metastore. Hive 2.3 execution












