# Section 9: Apache Spark — Querying Data (PySpark)

This module examines the programmatic ingestion frameworks, storage plane decoupling, and distributed execution semantics inherent to the PySpark DataFrame API. Functioning as the primary orchestration interface for production Lakehouse Directed Acyclic Graphs (DAGs), PySpark attains JVM-native processing latency. This performance is realized by delegating logical query plans to the Catalyst Optimizer and executing vectorized, off-heap memory computations through the Project Tungsten physical engine.

Refer to `image_66d9bb.png` for the execution sequence and topological dependency graph.

---

## Section Overview

* **Total Duration:** 46 minutes
* **Total Modules:** 7
* **Primary Focus:** Spark Connect gRPC architectures, distributed driver-executor topologies, explicit `StructType` schema enforcements, and partitioned JDBC extraction concurrency.

---

## Curriculum Breakdown

### 60. Introduction to PySpark (3 min)

* **Spark Connect Architecture**: Contemporary Databricks architectures decouple the client REPL environment from the Spark driver utilizing the Spark Connect gRPC protocol. Programmatic DataFrame instructions compile into lightweight, language-agnostic logical plans and transmit to the remote driver, thereby circumventing local Java Virtual Machine (JVM) initialization and neutralizing Py4J IPC serialization overhead.
* **Lazy Evaluation Semantics**: Pipeline execution relies on a strict bifurcation: **Transformations** (which construct a logical DAG lineage without invoking storage I/O) and **Actions** (which trigger Catalyst compilation, generate physical Tungsten bytecode, and materialize state to the distributed object store).
* **DataFrame Abstraction Layer**: Superseding legacy Resilient Distributed Datasets (RDDs), this architecture mandates strongly typed DataFrames. It leverages Whole-Stage Code Generation (WSCG) to apply relational optimizations across the Abstract Syntax Tree (AST) regardless of the host programming language.

### 61. Extract Customers Data — Simple JSON (17 min)

* **Strict Schema Definition**: Production-grade ingestion topologies require explicitly declared `StructType` schemas, strictly deprecating the dynamic `inferSchema` evaluation. Explicit schema binding bypasses the high-latency, multi-pass storage I/O required for metadata inference and isolates downstream DAG components from unanticipated structural data drift.
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

* **Semi-Structured Payload Parsing**: Utilizing imperative string indexing or regular expressions on nested JSON payloads induces severe JVM CPU degradation. Casting raw string columns via a native `from_json()` schema expression parses nested attributes inline, circumventing costly string manipulations and mitigating JVM garbage collection pressure.
* **Relational Normalization**: Deploying the `explode()` generator unpacks nested arrays into discrete vertical rows. Synthesizing this with struct dot notation (`orders.items`) normalizes complex hierarchical payloads into First Normal Form (1NF) relational schemas.

### 63. Extract Memberships Data — Binary File (4 min)

* **Unstructured Asset Ingestion**: Ingesting raw binary payloads (e.g., PDFs, image vectors, and compressed archives) directly into distributed executor memory utilizing the `binaryFile` format reader.
* **Structural Metadata Extraction**: The reader automatically serializes binary assets into a deterministic four-column metadata schema: `path` (`StringType`), `modificationTime` (`TimestampType`), `length` (`LongType`), and `content` (`BinaryType`).

### 64 & 65. Extract Addresses & Payments (TSV / CSV) (5 min + 8 min)

* **Delimiter & Format Configuration**: Parsing character-separated flat files by calibrating low-level format options (`sep`, `header`, `quote`, `escape`) to reconcile escape sequences and malformed string enclosures.
* **Data Corruption Handling Modes**:

| Parsing Mode | Execution Behavior | Data Integrity Impact |
| --- | --- | --- |
| **`PERMISSIVE`** *(Default)* | Coerces unparseable values to `NULL` or isolates invalid payloads into a designated `columnNameOfCorruptRecord` attribute. | Preserves maximum readable data without failing executor tasks. |
| **`DROPMALFORMED`** | Discards anomalous records entirely during the initial I/O scan phase. | Emits strictly compliant rows, resulting in silent data truncation for malformed records. |
| **`FAILFAST`** | Immediately aborts the Spark job upon detecting a structural anomaly. | Throws a fatal runtime exception and fails the dependent DAG task. |

### 66. Extract Refunds Data — SQL Table via JDBC (4 min)

* **Secret Management Integration**: Abstracting operational credentials utilizing Databricks Secret Scopes (`dbutils.secrets.get()`). This enforces secure credential injection, preventing plaintext authentication strings from surfacing in source control repositories or Spark UI execution telemetry.
* **Concurrent Multi-Executor Reads**: Default JDBC extractions route through a single network socket on a lone executor core, inducing an immediate throughput bottleneck. Defining numeric partition boundaries forces parallel socket connections across distributed worker cores, fully saturating available network bandwidth.
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

* **Transformation Lineage Classification**: Narrow transformations (`select()`, `filter()`, `withColumn()`) execute locally within partition boundaries without triggering inter-node data transfers. Wide transformations (`groupBy()`, `join()`, `distinct()`, `repartition()`) necessitate an `Exchange` physical operator, forcing a cluster-wide data shuffle that establishes rigid physical stage boundaries.
* **Broadcast Join Topologies**: When evaluating joins between massive fact tables and diminutive dimension tables (governed by the `spark.sql.autoBroadcastJoinThreshold` of 10 MB), utilizing a `broadcast(small_df)` hint forces total replication of the dimension dataset across all executors. This converts a high-latency Sort-Merge Join (SMJ) into a highly performant Broadcast Hash Join (BHJ), entirely averting the network shuffle phase.
* **Partition Size Heuristics**: Over-partitioning triggers severe object store metadata throttling and task scheduling overhead, whereas under-partitioning induces CPU starvation and Out-Of-Memory (`OOM`) JVM exceptions. Optimal physical file blocks for partition tuning in production Delta Lake environments must be maintained between 100 MB and 1 GB per Parquet file.

---

[← Back to Section 8: Apache Spark — Transforming Data (SQL)](https://www.google.com/search?q=./section08-readme.md) | [Next Section: Section 10: Advanced Transformations & Complex Data Structures →](https://www.google.com/search?q=./section10-readme.md)
