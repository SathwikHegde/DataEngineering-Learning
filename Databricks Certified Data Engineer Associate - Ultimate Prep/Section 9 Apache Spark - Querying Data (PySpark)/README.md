# Section 9: Apache Spark — Querying Data (PySpark)

This module details the programmatic ingestion patterns, storage plane abstraction, and distributed execution semantics natively exposed by the PySpark DataFrame API. Functioning as the primary declarative interface for production ETL Directed Acyclic Graphs (DAGs) within Lakehouse architectures, PySpark achieves JVM-native execution latency. This performance is realized by compiling logical query plans via the Catalyst Optimizer and delegating vectorized, off-heap memory operations to the Project Tungsten physical execution engine.

Refer to `image_66d9bb.png` for the topological dependency graph and execution sequencing.

---

## Section Overview

* **Total Duration:** 46 minutes
* **Total Modules:** 7
* **Primary Focus:** Distributed driver-executor topologies, Spark Connect gRPC decoupling, deterministic `StructType` schema bindings, and concurrent JDBC extraction partition tuning.

---

## Curriculum Breakdown

### 60. Introduction to PySpark (3 min)

* **Spark Connect Architecture**: Contemporary Databricks compute planes decouple client-side REPL environments from the Spark driver utilizing the Spark Connect gRPC protocol. Programmatic DataFrame invocations compile into lightweight, language-agnostic unresolved logical plans and transmit to the remote driver, thereby neutralizing local Java Virtual Machine (JVM) dependencies and mitigating Py4J IPC serialization overhead.
* **Lazy Evaluation Semantics**: Execution is strictly bifurcated into **Transformations** (which construct a logical DAG lineage without initiating storage I/O) and **Actions** (which trigger Catalyst compilation, generate physical Tungsten bytecode, and materialize output states to the distributed storage substrate).
* **DataFrame Abstraction Layer**: Deprecates legacy Resilient Distributed Datasets (RDDs) in favor of strictly typed DataFrames. This paradigm enforces Whole-Stage Code Generation (WSCG), empowering the engine to apply relational optimizations across the Abstract Syntax Tree (AST) regardless of the invoked host language.

### 61. Extract Customers Data — Simple JSON (17 min)

* **Strict Schema Definition**: Enterprise ingestion topologies mandate explicitly defined `StructType` schemas, deprecating dynamic `inferSchema` evaluation. Explicit schema binding circumvents the high-latency, multi-pass storage I/O required for metadata inference and isolates downstream DAG components from structural data drift.
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

* **Semi-Structured Payload Parsing**: Applying imperative string indexing or regular expressions against nested JSON payloads introduces severe JVM CPU degradation. Binding raw string columns to a native `from_json()` schema expression evaluates nested attributes inline, bypassing expensive string manipulation and minimizing the JVM garbage collection footprint.
* **Relational Normalization**: Invoking the `explode()` generator transposes nested arrays into discrete vertical records. Combining this generator with struct dot notation (`orders.items`) flattens complex hierarchical maps into normalized, First Normal Form (1NF) tabular schemas.

### 63. Extract Memberships Data — Binary File (4 min)

* **Unstructured Asset Ingestion**: Loading raw binary assets (e.g., compressed byte streams, PDFs, image vectors) directly into distributed executor memory utilizing the `binaryFile` format reader.
* **Structural Metadata Extraction**: The reader automatically marshals binary payloads into a deterministic four-column metadata struct: `path` (`StringType`), `modificationTime` (`TimestampType`), `length` (`LongType`), and `content` (`BinaryType`).

### 64 & 65. Extract Addresses & Payments (TSV / CSV) (5 min + 8 min)

* **Delimiter & Format Configuration**: Ingesting character-separated flat files by tuning low-level parser configurations (`sep`, `header`, `quote`, `escape`) to reconcile malformed string enclosures and escape sequences.
* **Data Corruption Handling Modes**:

| Parsing Mode | Execution Behavior | Data Integrity Impact |
| --- | --- | --- |
| **`PERMISSIVE`** *(Default)* | Coerces malformed values to `NULL` or routes invalid payloads into a designated `columnNameOfCorruptRecord` field. | Preserves maximum readable data without interrupting executor tasks. |
| **`DROPMALFORMED`** | Drops unparseable records entirely during the initial I/O read phase. | Emits strictly compliant rows, resulting in silent data loss for malformed records. |
| **`FAILFAST`** | Instantly aborts the Spark job upon encountering a structural anomaly. | Throws a fatal runtime exception and fails the associated DAG task. |

### 66. Extract Refunds Data — SQL Table via JDBC (4 min)

* **Secret Management Integration**: Isolating operational credentials via Databricks Secret Scopes (`dbutils.secrets.get()`). This practice prevents plaintext authentication strings from leaking into source control repositories or Spark UI execution telemetry.
* **Concurrent Multi-Executor Reads**: Default JDBC reads route through a single network socket on a single executor core, creating an immediate throughput bottleneck. Establishing numeric partition boundaries forces parallel socket connections across multiple worker cores, saturating available network bandwidth.
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

* **Transformation Lineage Classification**: Narrow transformations (`select()`, `filter()`, `withColumn()`) compute locally within a partition boundary without initiating inter-node data exchanges. Wide transformations (`groupBy()`, `join()`, `distinct()`, `repartition()`) force an `Exchange` physical operator, triggering a cluster-wide data shuffle that establishes hard physical boundaries between execution stages.
* **Broadcast Join Topologies**: When executing joins between massive fact tables and small dimension tables (bound by the $\le 10\text{ MB}$ `spark.sql.autoBroadcastJoinThreshold`), injecting a `broadcast(small_df)` hint forces a complete replication of the dimension data to all executors. This mechanism converts a high-latency Sort-Merge Join (SMJ) into a highly performant Broadcast Hash Join (BHJ), entirely bypassing the network shuffle phase.
* **Partition Size Heuristics**: Over-partitioning induces severe object store metadata throttling and task scheduling overhead, whereas under-partitioning triggers CPU starvation and Out-Of-Memory (`OOM`) JVM exceptions. Optimal physical file sizes for partition tuning in production Delta Lake environments must be constrained between 100 MB and 1 GB per Parquet file block.

---

[← Back to Section 8: Apache Spark — Transforming Data (SQL)](https://www.google.com/search?q=./section08-readme.md&utm_source=gemini) | [Next Section: Section 10: Advanced Transformations & Complex Data Structures →](https://www.google.com/search?q=./section10-readme.md&utm_source=gemini)
