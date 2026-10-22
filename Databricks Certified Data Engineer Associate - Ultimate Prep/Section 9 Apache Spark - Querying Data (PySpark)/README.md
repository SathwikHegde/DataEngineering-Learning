# Section 9: Apache Spark — Querying Data (PySpark)

This module defines programmatic ingestion, storage abstraction, and distributed execution topologies within the PySpark DataFrame API. PySpark achieves JVM-native execution latency by compiling logical query plans via the Catalyst Optimizer and delegating memory operations to the Project Tungsten physical engine. Refer to `image_66d9bb.png` for the topological dependency graph.

## Section Overview

**Total Duration:** 46 minutes
**Total Modules:** 7
**Primary Focus:** Spark Connect gRPC architectures, driver-executor topologies, `StructType` schemas, and concurrent JDBC extraction.

## Curriculum Breakdown

### 60. Introduction to PySpark (3 min)

* Spark Connect architecture decouples the client REPL from the driver via gRPC, neutralizing Py4J IPC serialization overhead.
* Lazy evaluation strictly bifurcates execution into logical Transformations and storage-materializing Actions.
* DataFrames enforce strong typing and utilize Whole-Stage Code Generation (WSCG) for AST relational optimizations.

### 61. Extract Customers Data (17 min)

* Enterprise topologies mandate explicit `StructType` declarations to bypass high-latency inference I/O and isolate downstream DAGs.

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

### 62. Extract Orders Data (5 min)

* Binding raw strings to `from_json()` parses nested attributes inline, minimizing JVM garbage collection footprints.
* The `explode()` generator flattens nested arrays into First Normal Form (1NF) relational schemas.

### 63. Extract Memberships Data (4 min)

* The `binaryFile` reader ingests unstructured payloads (e.g., PDFs) directly into distributed executor memory.
* Binary assets serialize into a deterministic metadata struct: `path`, `modificationTime`, `length`, and `content`.

### 64 & 65. Extract Addresses & Payments (13 min)

* Character-separated file ingestion requires tuning format options like `sep` and `escape` to reconcile malformed enclosures.

| Parsing Mode | Execution Behavior | Data Integrity Impact |
| --- | --- | --- |
| **PERMISSIVE** | Coerces malformed values to NULL or routes payloads to a specific error column. | Preserves maximum readable data without interrupting executor tasks. |
| **DROPMALFORMED** | Drops unparseable records entirely during the initial I/O read phase. | Emits strictly compliant rows, resulting in silent data loss for anomalies. |
| **FAILFAST** | Instantly aborts the Spark job upon encountering a structural anomaly. | Throws a fatal runtime exception and fails the associated DAG task. |

### 66. Extract Refunds Data via JDBC (4 min)

* Databricks Secret Scopes isolate credentials to prevent plaintext leaks in source control or execution telemetry.
* Numeric partition boundaries force parallel socket connections across worker cores to saturate network bandwidth.

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

## Important Exam Considerations

* Narrow transformations compute locally; wide transformations force a cluster-wide Exchange shuffle establishing physical boundaries.
* Injecting a `broadcast()` hint replicates dimension data under 10 MB to all executors, converting an SMJ to a performant BHJ.
* Optimal physical file sizes for Delta Lake partition tuning must remain between 100 MB and 1 GB per block to prevent OOM errors.

[← Back to Section 8: Apache Spark — Transforming Data (SQL)](https://www.google.com/search?q=./section08-readme.md) | [Next Section: Section 10: Advanced Transformations & Complex Data Structures →](https://www.google.com/search?q=./section10-readme.md)
