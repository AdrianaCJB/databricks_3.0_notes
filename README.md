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

## Spark's Architecture

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
  
- Shuffles: Represents a physical repartitioning of the data and requires coordinating across executors to move data around.
    - The spark.sql.shuffle.partitions default value is 200, which means that when there is a shuffle performed during execution, it outputs 200 shuffle partitions by default. 
    - You can change this value, and the number of output partitions will change.
`spark.conf.set("spark.sql.shuffle.partitions", 5)`
