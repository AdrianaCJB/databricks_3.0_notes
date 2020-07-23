# Databricks Spark 3.0 Notes

This is my study notes about the Databricks Certified Associate Developer for Apache Spark 3.0 - Python


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

- Stages: Each job gets divided into smaller sets of tasks called stages that depend on each other. A stage represent groups of tasks that can be executed together to compute the same operation on multiple machines. 

- Tasks: A task is just a single unit of work or execution that applied to a unit of data (the partition). Each task corresponds to a combination of blocks of data and a set of transformations that will run on a single executor. If there is one big partition in our dataset, we will have one task. If there are 1,000 little partitions, we will have 1,000 tasks that can be executed in parallel.

- Slots: Each executor has a number of slots. Then, tasks are assigned to slots for parallel execution.

- Partition: A chuck of data (collection of rows) that sit on a physical machine in a cluster.
  - A good rule of thumb is that the number of partitions should be larger than the number of executors on your cluster, potentially by multiple factors depending on the workload.
  
- Shuffles: Are triggered when data needs to move between executors. Represents a physical repartitioning of the data and requires coordinating across executors to move data around.
    - The spark.sql.shuffle.partitions default value is 200, which means that when there is a shuffle performed during execution, it outputs 200 shuffle partitions by default. 
    - You can change this value, and the number of output partitions will change.
`spark.conf.set("spark.sql.shuffle.partitions", 5)`

## DataFrames API 

### Transformations, Actions and Lazy Evaluation

Spark operations on distributed data can be classified into two types: transformations and actions.

- Transformation: transform a Spark DataFrame into a new DataFrame without altering the original data (each transformation creates a new structure), giving it the property of **immutability**.

Two types of transformations:
  - Narrow dependencies: Any transformation where a single output partition can be computed from a single input partition. 
    - Also called **pipelining**
    - performed **in-memory**
    - e.g: `filter() and contains()`
  - Wide dependencies:
    - Also called **shuffle**.
    - data from other partitions is read in, combined, and written to disk. 
    - performed **on disk**
    - e.g `groupBy() and orderBy()`

- Actions: An action instructs Spark to compute a result from a series of transformations. Nothing in a query plan is executed until an action is
invoked. There are three kinds of actions:
  - Actions to view data in the console.
  - Actions to collect data to native objects in the respective language.
  - Actions to write to output data sources.

    | Transformations | Actions |
    | ------------- | --------- |
    | orderBy()     |   show()  |
    | groupBy()     |   take()  |
    | filter()      |   count() |
    | select()      | collect() |
    | join()        |  save()   |

- Lazy Evaluation: Refers to the idea that Spark waits until the last moment to execute a series of operations. That is, their results are not computed immediately, but they are recorded or remembered as a lineage. Allows to rearrange certain transformations. The plan is executed when you call an action. 




- Partitions:
- Caching:

## The Spark UI 

In local mode, you can access this interface at http://<localhost>:4040 in a web
browser.
  
Running by default on port 4040, where you can view metrics and details such as:
- A list of scheduler stages and tasks
- A summary of RDD sizes and memory usage
- Information about the environment
- Information about the running executors
- All the Spark SQL queries











