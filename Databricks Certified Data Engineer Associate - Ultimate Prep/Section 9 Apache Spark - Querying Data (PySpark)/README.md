# Section 9: Apache Spark — Querying Data (PySpark)

This section focuses on programmatic data extraction, storage decoupling, and distributed API interactions using the **PySpark DataFrame API**. In distributed architectures, PySpark serves as the primary interface for production pipelines, matching Scala execution speeds through optimizations applied by the **Catalyst Optimizer** and the **Tungsten** physical execution engine.

Refer to `image_66d9bb.png` for the lesson timeline and curriculum sequence.

---

## Section Overview

* **Total Duration:** 46 minutes
* **Total Lessons:** 7
* **Primary Focus:** Distributed driver-worker cluster topologies, Spark Connect client decoupling, structured schema mapping, and parallel multi-partition JDBC extractions.

---

## Curriculum Breakdown

### 60. Introduction to PySpark (3 min)

* **Spark Connect Architecture**: Modern runtimes decouple client sessions from the Spark driver via the **Spark Connect** client-server protocol. Commands compile into lightweight, language-agnostic DataFrame plans and transmit via gRPC to the remote driver, eliminating local Java Virtual Machine (JVM) dependencies.
* **Lazy Evaluation**: Separates execution steps into **Transformations** (which construct a logical Directed Acyclic Graph without loading data into memory) and **Actions** (which trigger Catalyst optimization, compile physical Tungsten plans, and materialize or write dataset outputs).

* **DataFrame Abstraction**: Deprecates low-level Resilient Distributed Datasets (RDDs) in favor of strictly typed **DataFrames**, enabling whole-stage code generation, memory off-heap storage, and vectorized expression evaluations across worker nodes.

### 61. Extract Customers Data — Simple JSON (17 min)

* **Strict Schema Definition**: Production workloads require explicitly defined `StructType` models over dynamic schema inference (`inferSchema`). Declaring types explicitly bypasses expensive initial metadata scans across cloud storage objects and protects downstream stages from schema drift.
* **Code Implementation Pattern**:
```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

# Explicit schema definition for safe production ingestion
customer_schema = StructType([
    StructField("customer_id", StringType(), False),
    StructField("profile_name", StringType(), True),
    StructField("signup_epoch", IntegerType(), True)
])

df_customers = (spark.read
    .schema(customer_schema)
    .json("abfss://raw-zone@storageaccount.dfs.core.windows.net/customers/*.json"))

```



### 62. Extract Orders Data — Complex JSON as Text (5 min)

* **Semi-Structured Parsing**: Direct string indexing and regex operations on nested JSON strings introduce CPU overhead. Binding raw strings to an explicit `from_json()` schema expression evaluates nested elements inline with minimal memory consumption.
* **Relational Normalization**: Utilizing `explode()` to transpose nested array elements into individual vertical rows, combined with standard dot notation (`orders.items`) to un-nest complex struct maps into tabular records.

### 63. Extract Memberships Data — Binary File (4 min)

* **Unstructured Ingestion**: Loading raw binary assets (images, PDFs, audio streams, geospatial raster layers) directly into DataFrames using the `binaryFile` format reader.
* **Structural Metadata Fields**: Ingestion automatically maps file payloads into a four-column metadata schema: `path` (`StringType`), `modificationTime` (`TimestampType`), `length` (`LongType`), and `content` (`BinaryType`).

### 64 & 65. Extract Addresses & Payments (TSV / CSV) (5 min + 8 min)

* **Delimiter & Format Options**: Reading character-separated flat files by tuning parsing configurations (`sep`, `header`, `quote`, `escape`).
* **Data Corruption Handling Modes**:
* `PERMISSIVE` (Default): Inserts corrupted rows by populating malformed values with `NULL` or directing invalid payloads into a designated `columnNameOfCorruptRecord` field while execution continues.
* `DROPMALFORMED`: Discards invalid or unparseable records during the read phase, emitting only compliant rows.
* `FAILFAST`: Aborts execution immediately upon encountering the first malformed record, returning a runtime parsing exception.



### 66. Extract Refunds Data — SQL Table via JDBC (4 min)

* **Secret Management Integration**: Isolates operational credentials using **Databricks Secret Scopes** via `dbutils.secrets.get()`, preventing raw passwords from leaking into source code or execution logs.
* **Parallel Multi-Executor Reads**: Default JDBC connections route through a single network socket on one executor node. Setting numeric partition boundaries forces parallel socket connections across multiple worker threads to saturate network bandwidth.
* **Code Implementation Pattern**:
```python
df_refunds = (spark.read
    .format("jdbc")
    .option("url", "jdbc:postgresql://rds-cluster-prod.internal:5432/finance")
    .option("dbtable", "transaction_refunds")
    # Parallelization boundaries
    .option("partitionColumn", "refund_date")
    .option("lowerBound", "2026-01-01")
    .option("upperBound", "2026-12-31")
    .option("numPartitions", "16")
    .option("user", dbutils.secrets.get(scope="jdbc-scope", key="db-user"))
    .option("password", dbutils.secrets.get(scope="jdbc-scope", key="db-pass"))
    .load())

```



---

## Important Exam Considerations

* **Transformation Classification**: Narrow transformations (`select()`, `filter()`, `withColumn()`) compute entirely within a local partition without inter-node data transfers. Wide transformations (`groupBy()`, `join()`, `distinct()`, `repartition()`) require a cluster-wide shuffle stage, defining the physical boundary between separate Spark execution stages.
* **Broadcast Join Mechanics**: When joining large fact tables with small dimension tables ($\le 10\text{ MB}$ default threshold), wrapping the small DataFrame in `broadcast(small_df)` distributes a full copy of the dimension data to all executors, converting an expensive Shuffle Hash Join or Sort-Merge Join into a low-latency Broadcast Hash Join.
* **Partition Size Targets**: Over-partitioning causes object store metadata bottlenecks, while under-partitioning leads to CPU starvation and Out-Of-Memory (`OOM`) errors. The standard target file size for partition tuning in production Delta Lake environments is **100 MB to 1 GB** per file block.

---

[← Back to Section 8: Apache Spark — Transforming Data (SQL)](https://www.google.com/search?q=./section08-readme.md) | [Next Section: Section 10: Advanced Transformations & Complex Data Structures →](https://www.google.com/search?q=./section10-readme.md)
