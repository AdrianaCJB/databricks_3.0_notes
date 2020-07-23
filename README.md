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
  - How to configure options for specific formats
  - How to write a data source to 1 single file or N separate files. e.g: 
  - How to write partitioned data
  ```
  spark.write.repartition(N).mode("overwrite").parquet(<path>)
  ```
  - How to bucket data by a given set of columns

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
  withColumnRenamed("name", "newColumName")
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
  ```
  - Dropping: `df.drop("col")
  - Unique rows: `df.select("col1").distinct().show()`
  - Cast columns: `df.select(col("count").cast("int").alias("count_cast"))`
  
  



**- Partitions**
  - Display the number of partitions in each DataFrame. e.g: `df.rdd.getNumPartitions()`
  - Set to up the number of partitions. e.g: `df.repartition(5)`
  - Set to down the number of partitions. e.g: `df.coalesce(2)`
  
**- Caching**


**- Manipulating Data**
    - Imputing null or missing data: There are some techniques such as dropping these records, adding a placeholder (e.g. -1 or -999), basic imputing by using the mean of non-missing data, and advanced imputing such as clustering ML algorithms or oversampling techniques. e.g. `df.dropna("any")` , `df.na.fill({"col1" : 5})`
    - Deduplicating data: `df.dropDuplicates(["id"])`
    - Other helpful data manipultion functions: e.g `explode(), pivot(), cube(), rullop()`


## Spark SQL

### Catalyst Optimizer

Is at the core of Spark SQL's power and speed. It automatically finds the most efficient plan for applying your transformations and actions you called for in your code.



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











