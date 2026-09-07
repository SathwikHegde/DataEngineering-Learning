# Section 9: Apache Spark — Querying Data (PySpark)

This module examines programmatic data extraction, storage plane decoupling, and distributed execution mechanics via the **PySpark DataFrame API**. Within distributed Lakehouse architectures, PySpark functions as the primary orchestration interface for production ETL DAGs. It achieves Scala-equivalent execution latency by pushing logical query plans to the **Catalyst Optimizer** and leveraging the **Project Tungsten** physical execution engine for vectorized, off-heap memory management.

Refer to `image_66d9bb.png` for the execution timeline and dependency sequence.

---

## Section Overview

* **Total Duration:** 46 minutes
* **Total Lessons:** 7
* **Primary Focus:** Distributed driver-worker node topologies, Spark Connect gRPC decoupling, explicit struct schema bindings, and partitioned JDBC extraction parallelism.

---

## Curriculum Breakdown

### 60. Introduction to PySpark (3 min)

* **Spark Connect Architecture**: Modern runtimes decouple client sessions from the Spark driver via the **Spark Connect** client-server protocol. Programmatic commands compile into lightweight, language-agnostic DataFrame unresolved logical plans and transmit via gRPC to the remote driver, eliminating local Java Virtual Machine (JVM) dependencies and Py4J serialization overhead.
* **Lazy Evaluation**: Separates pipeline execution into **Transformations** (which construct a logical Directed Acyclic Graph without initiating data I/O) and **Actions** (which trigger Catalyst optimization, compile physical Tungsten bytecode, and materialize output states to storage).
* **DataFrame Abstraction**: Deprecates legacy Resilient Distributed Datasets (RDDs) in favor of strictly typed **DataFrames**. This enables Whole-Stage Code Generation (WSCG) and allows the engine to apply relational optimizations regardless of the host language.

### 61. Extract Customers Data — Simple JSON (17 min)

* **Strict Schema Definition**: Production workloads mandate explicitly defined `StructType` models over dynamic schema inference (`inferSchema`). Declaring types explicitly bypasses the expensive two-pass I/O scan required to infer metadata across cloud object storage and immunizes downstream processing from structural data drift.
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

* **Semi-Structured Parsing**: Direct string indexing and regex operations on nested JSON payloads introduce severe JVM CPU overhead. Binding raw string columns to an explicit `from_json()` schema expression evaluates nested elements inline, minimizing garbage collection footprint.
* **Relational Normalization**: Utilizing the `explode()` generator to transpose nested arrays into independent vertical records, combined with struct dot notation (`orders.items`) to un-nest complex hierarchical maps into normalized tabular schemas.

### 63. Extract Memberships Data — Binary File (4 min)

* **Unstructured Ingestion**: Loading raw binary assets (e.g., images, PDFs, compressed byte streams) directly into distributed memory using the `binaryFile` format reader.
* **Structural Metadata Fields**: The reader maps binary payloads into a strict four-column metadata schema: `path` (`StringType`), `modificationTime` (`TimestampType`), `length` (`LongType`), and `content` (`BinaryType`).

### 64 & 65. Extract Addresses & Payments (TSV / CSV) (5 min + 8 min)

* **Delimiter & Format Options**: Ingesting character-separated flat files by tuning parser configurations (`sep`, `header`, `quote`, `escape`) to handle malformed string enclosures.
* **Data Corruption Handling Modes**:
* `PERMISSIVE` (Default): Inserts corrupted rows by forcing malformed values to `NULL` or routing invalid payloads into a dedicated `columnNameOfCorruptRecord` field without halting executor tasks.
* `DROPMALFORMED`: Discards unparseable records entirely during the I/O read phase, emitting strictly compliant rows.
* `FAILFAST`: Aborts the Spark job immediately upon encountering a structural anomaly, throwing a runtime exception and failing the associated DAG task.



### 66. Extract Refunds Data — SQL Table via JDBC (4 min)

* **Secret Management Integration**: Isolates operational credentials utilizing **Databricks Secret Scopes** via `dbutils.secrets.get()`, preventing raw authentication strings from leaking into source code or Spark UI execution logs.
* **Parallel Multi-Executor Reads**: Default JDBC reads route through a single network socket on one executor, severely bottlenecking throughput. Defining numeric partition boundaries forces parallel socket connections across multiple worker cores to saturate available network bandwidth.
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

* **Transformation Classification**: Narrow transformations (`select()`, `filter()`, `withColumn()`) compute locally within a partition boundary without inter-node data transfers. Wide transformations (`groupBy()`, `join()`, `distinct()`, `repartition()`) force an `Exchange` operation, triggering a cluster-wide data shuffle that delineates physical boundaries between execution stages.
* **Broadcast Join Mechanics**: When joining massive fact tables against small dimension tables (governed by the $\le 10\text{ MB}$ `spark.sql.autoBroadcastJoinThreshold`), utilizing a `broadcast(small_df)` hint forces a complete copy of the dimension data to all executors. This converts a highly expensive Sort-Merge Join (SMJ) into a low-latency Broadcast Hash Join (BHJ).
* **Partition Size Targets**: Over-partitioning induces object store metadata throttling and scheduler overhead, while under-partitioning causes CPU starvation and Out-Of-Memory (`OOM`) exceptions. Target physical file sizes for partition tuning in production Delta Lake environments must remain between **100 MB and 1 GB** per file block.

---

[← Back to Section 8: Apache Spark — Transforming Data (SQL)](https://www.google.com/search?q=./section08-readme.md) | [Next Section: Section 10: Advanced Transformations & Complex Data Structures →](https://www.google.com/search?q=./section10-readme.md)
