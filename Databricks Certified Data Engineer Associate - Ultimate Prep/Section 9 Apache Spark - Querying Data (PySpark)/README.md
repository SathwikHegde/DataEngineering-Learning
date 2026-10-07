# Section 9: Apache Spark — Querying Data (PySpark)

This module breaks down programmatic data ingestion, storage abstraction, and distributed execution mechanics utilizing the PySpark DataFrame API. Serving as the primary orchestration layer for production Lakehouse Directed Acyclic Graphs (DAGs), PySpark achieves JVM-native processing speeds. It accomplishes this by passing logical query plans to the Catalyst Optimizer and executing vectorized, off-heap memory operations via the Project Tungsten engine.

Refer to `image_66d9bb.png` for the execution sequence and topological dependency graph.

---

## Section Overview

* **Total Duration:** 46 minutes
* **Total Modules:** 7
* **Primary Focus:** Spark Connect gRPC architecture, driver-executor topologies, explicit `StructType` schema bindings, and concurrent JDBC extraction.

---

## Curriculum Breakdown

### 60. Introduction to PySpark (3 min)

* **Spark Connect Architecture**: Modern Databricks environments separate the client REPL from the Spark driver using the Spark Connect gRPC protocol. DataFrame commands are compiled into lightweight, language-agnostic logical plans and sent to the remote driver. This bypasses local Java Virtual Machine (JVM) initialization and removes Py4J IPC serialization overhead.
* **Lazy Evaluation Semantics**: Pipeline execution is strictly divided into **Transformations** (building a logical DAG without triggering storage I/O) and **Actions** (invoking Catalyst compilation, generating Tungsten bytecode, and materializing the data to storage).
* **DataFrame Abstraction Layer**: Replacing legacy Resilient Distributed Datasets (RDDs), this architecture mandates strongly typed DataFrames. It uses Whole-Stage Code Generation (WSCG) to apply relational optimizations to the Abstract Syntax Tree (AST), regardless of the host programming language.

### 61. Extract Customers Data — Simple JSON (17 min)

* **Strict Schema Definition**: Production ingestion requires explicit `StructType` schemas, avoiding the dynamic `inferSchema` method. Hardcoding the schema prevents the high-latency, multi-pass I/O scans required for metadata inference and protects downstream dependencies from unexpected schema drift.
* **Code Implementation Pattern**:

```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

# Explicit schema definition to bypass inference I/O overhead
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

* **Semi-Structured Payload Parsing**: Using standard string indexing or regex on nested JSON causes significant JVM CPU degradation. Instead, casting raw string columns with a native `from_json()` schema expression parses attributes inline, avoiding expensive string manipulations and lowering garbage collection overhead.
* **Relational Normalization**: The `explode()` generator flattens nested arrays into individual rows. When combined with struct dot notation (`orders.items`), it normalizes complex hierarchical data into First Normal Form (1NF) relational structures.

### 63. Extract Memberships Data — Binary File (4 min)

* **Unstructured Asset Ingestion**: Loading raw binary files (such as PDFs, image vectors, and compressed archives) directly into executor memory via the `binaryFile` format reader.
* **Structural Metadata Extraction**: The reader maps binary assets to a fixed four-column metadata schema: `path` (`StringType`), `modificationTime` (`TimestampType`), `length` (`LongType`), and `content` (`BinaryType`).

### 64 & 65. Extract Addresses & Payments (TSV / CSV) (5 min + 8 min)

* **Delimiter & Format Configuration**: Parsing flat files by configuring low-level format options (`sep`, `header`, `quote`, `escape`) to accurately process escape sequences and malformed enclosures.
* **Data Corruption Handling Modes**:

| Parsing Mode | Execution Behavior | Data Integrity Impact |
| --- | --- | --- |
| **`PERMISSIVE`** *(Default)* | Converts unparseable values to `NULL` or routes invalid payloads to a specific `columnNameOfCorruptRecord` column. | Retains maximum readable data without crashing executor tasks. |
| **`DROPMALFORMED`** | Completely drops anomalous records during the initial I/O read phase. | Yields strictly compliant rows, but causes silent data loss for malformed entries. |
| **`FAILFAST`** | Instantly aborts the Spark job when a structural anomaly is detected. | Triggers a fatal runtime exception, failing the associated DAG task. |

### 66. Extract Refunds Data — SQL Table via JDBC (4 min)

* **Secret Management Integration**: Securing operational credentials using Databricks Secret Scopes (`dbutils.secrets.get()`). This ensures safe credential injection and prevents plaintext passwords from leaking into source control or Spark UI logs.
* **Concurrent Multi-Executor Reads**: By default, JDBC extractions route through a single network socket on one executor core, causing an immediate bottleneck. Configuring numeric partition boundaries establishes parallel socket connections across distributed workers, saturating available network bandwidth.
* **Code Implementation Pattern**:

```python
df_refunds = (spark.read
    .format("jdbc")
    .option("url", "jdbc:postgresql://rds-cluster-prod.internal:5432/finance")
    .option("dbtable", "transaction_refunds")
    # Concurrency boundaries for multi-partition extraction
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

* **Transformation Lineage Classification**: Narrow transformations (`select()`, `filter()`, `withColumn()`) run locally within partition boundaries without inter-node data transfers. Wide transformations (`groupBy()`, `join()`, `distinct()`, `repartition()`) require an `Exchange` physical operator, forcing a cluster-wide data shuffle that creates hard physical stage boundaries.
* **Broadcast Join Topologies**: When joining massive fact tables with small dimension tables (typically under 10 MB based on the `spark.sql.autoBroadcastJoinThreshold`), using a `broadcast(small_df)` hint replicates the dimension dataset across all executors. This turns a slow Sort-Merge Join (SMJ) into a highly efficient Broadcast Hash Join (BHJ), bypassing the network shuffle phase.
* **Partition Size Heuristics**: Over-partitioning causes object store metadata throttling and task scheduling delays, while under-partitioning leads to CPU starvation and Out-Of-Memory (`OOM`) exceptions. For production Delta Lake environments, target physical file blocks between 100 MB and 1 GB per Parquet file.

---

[← Back to Section 8: Apache Spark — Transforming Data (SQL)](https://www.google.com/search?q=./section08-readme.md) | [Next Section: Section 10: Advanced Transformations & Complex Data Structures →](https://www.google.com/search?q=./section10-readme.md)
