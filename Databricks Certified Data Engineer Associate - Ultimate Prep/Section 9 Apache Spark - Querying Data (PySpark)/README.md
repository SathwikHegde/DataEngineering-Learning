# Section 9: Apache Spark — Querying Data (PySpark)

This section analyzes programmatic data extraction, storage layer decoupling, and distributed execution topologies utilizing the PySpark DataFrame API. As the primary orchestration interface for production ETL Directed Acyclic Graphs (DAGs) within Lakehouse architectures, PySpark delivers JVM-native execution latency. This is achieved by compiling logical query plans through the Catalyst Optimizer and executing vectorized, off-heap memory operations via the Project Tungsten engine.

Refer to `image_66d9bb.png` for the execution timeline and topological dependency sequence.

---

## Section Overview

* **Total Duration:** 46 minutes
* **Total Lessons:** 7
* **Primary Focus:** Distributed driver-executor node topologies, Spark Connect gRPC abstraction, explicit struct schema bindings, and partitioned JDBC extraction parallelism.

---

## Curriculum Breakdown

### 60. Introduction to PySpark (3 min)

* **Spark Connect Architecture**: Modern Databricks runtimes abstract client REPL sessions from the Spark driver via the Spark Connect client-server protocol. Programmatic DataFrame instructions compile into lightweight, language-agnostic unresolved logical plans and transmit via gRPC to the remote driver, eliminating local Java Virtual Machine (JVM) dependencies and Py4J serialization overhead.
* **Lazy Evaluation**: Pipeline execution is strictly bifurcated into **Transformations** (constructing a logical DAG lineage without initiating storage I/O) and **Actions** (triggering Catalyst optimization, compiling physical Tungsten bytecode, and materializing output states to the distributed storage plane).
* **DataFrame Abstraction**: Deprecates legacy Resilient Distributed Datasets (RDDs) in favor of strictly typed DataFrames. This architecture enforces Whole-Stage Code Generation (WSCG), allowing the engine to apply relational optimizations across the Abstract Syntax Tree (AST) irrespective of the host programming language.

### 61. Extract Customers Data — Simple JSON (17 min)

* **Strict Schema Definition**: Production pipelines require explicit `StructType` declarations to supersede dynamic `inferSchema` evaluations. Explicit declaration circumvents the high-latency, multi-pass I/O scans required to infer metadata across cloud object storage and immunizes downstream processing against structural data drift.
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

* **Semi-Structured Parsing**: Executing direct string indexing or regex on nested JSON payloads incurs substantial JVM CPU penalties. Binding raw string columns to an explicit `from_json()` schema expression evaluates nested elements inline, circumventing expensive string manipulation and minimizing the garbage collection footprint.
* **Relational Normalization**: Utilizing the `explode()` generator transposes nested arrays into independent vertical records. Combining this with struct dot notation (`orders.items`) un-nests complex hierarchical maps into normalized, First Normal Form (1NF) tabular schemas.

### 63. Extract Memberships Data — Binary File (4 min)

* **Unstructured Ingestion**: Loading raw binary assets (e.g., images, PDFs, compressed byte streams) directly into distributed memory via the `binaryFile` format reader.
* **Structural Metadata Fields**: The reader automatically maps binary payloads into a strict four-column metadata struct: `path` (`StringType`), `modificationTime` (`TimestampType`), `length` (`LongType`), and `content` (`BinaryType`).

### 64 & 65. Extract Addresses & Payments (TSV / CSV) (5 min + 8 min)

* **Delimiter & Format Options**: Ingesting character-separated flat files by tuning parser configurations (`sep`, `header`, `quote`, `escape`) to manage malformed string enclosures and escape sequences.
* **Data Corruption Handling Modes**:

| Parsing Mode | Execution Behavior | Data Integrity Impact |
| --- | --- | --- |
| **`PERMISSIVE`** *(Default)* | Forces malformed values to `NULL` or routes invalid payloads into a dedicated `columnNameOfCorruptRecord` field. | Executor tasks continue uninterrupted; preserves maximum readable data. |
| **`DROPMALFORMED`** | Discards unparseable records entirely during the I/O read phase. | Emits strictly compliant rows; silent data loss for malformed rows. |
| **`FAILFAST`** | Aborts the Spark job immediately upon encountering a structural anomaly. | Throws a runtime exception, failing the associated DAG task immediately. |

### 66. Extract Refunds Data — SQL Table via JDBC (4 min)

* **Secret Management Integration**: Isolates operational credentials utilizing Databricks Secret Scopes via `dbutils.secrets.get()`, preventing raw authentication strings from leaking into source code or Spark UI execution logs.
* **Parallel Multi-Executor Reads**: Default JDBC reads route through a single network socket on a single executor core, creating a severe throughput bottleneck. Defining numeric partition boundaries forces parallel socket connections across multiple worker cores to saturate available network bandwidth.
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

* **Transformation Classification**: Narrow transformations (`select()`, `filter()`, `withColumn()`) compute locally within a partition boundary without inter-node data exchanges. Wide transformations (`groupBy()`, `join()`, `distinct()`, `repartition()`) force an `Exchange` operation, triggering a cluster-wide data shuffle that delineates physical boundaries between execution stages.
* **Broadcast Join Mechanics**: When joining massive fact tables against small dimension tables (governed by the $\le 10\text{ MB}$ `spark.sql.autoBroadcastJoinThreshold`), utilizing a `broadcast(small_df)` hint forces a complete copy of the dimension data to all executors. This converts a high-latency Sort-Merge Join (SMJ) into a highly efficient Broadcast Hash Join (BHJ), bypassing the shuffle phase entirely.
* **Partition Size Targets**: Over-partitioning induces object store metadata throttling and scheduler overhead, while under-partitioning causes CPU starvation and Out-Of-Memory (`OOM`) exceptions. Target physical file sizes for partition tuning in production Delta Lake environments must remain between 100 MB and 1 GB per Parquet file block.

---

[← Back to Section 8: Apache Spark — Transforming Data (SQL)](https://www.google.com/search?q=./section08-readme.md&utm_source=gemini) | [Next Section: Section 10: Advanced Transformations & Complex Data Structures →](https://www.google.com/search?q=./section10-readme.md&utm_source=gemini)
