# Section 9: Apache Spark — Querying Data (PySpark)

This section focuses on data extraction, storage decoupling, and programmatic API interactions using the **PySpark DataFrame API**. In modern distributed environments, PySpark serves as the primary interface for production data engineering, delivering execution performance identical to Scala due to the optimizations of the **Catalyst Optimizer** and the **Tungsten** execution engine.

Refer to `image_66d9bb.png` for the lesson timeline and curriculum sequence.

---

## Section Overview

* **Total Duration:** 46 minutes
* **Total Lessons:** 7
* **Primary Focus:** Distributed driver-worker architecture, decoupled client sessions, structured file schema layout bindings, and parallel relational extraction.

---

## Curriculum Breakdown

### 60. Introduction to PySpark (3 min)

* **The Remote Connectivity Standard**: Modern environments lean heavily on the **Spark Connect** client-server protocol. This architecture decouples the client application layer from the Spark driver, executing commands as lightweight gRPC requests from a thin client environment without requiring local Java Virtual Machine (JVM) installations.
* **Lazy Evaluation Architecture**: Differentiating between structural **Transformations** (which append execution instructions to a logical DAG plan without pulling data into memory) and **Actions** (which compile, optimize, and compute the logical plan to return physical results or persist output to storage).

* **DataFrame API Paradigm**: Resilient Distributed Datasets (RDDs) serve as a legacy storage and execution abstraction layer. Production pipelines enforce the use of strict **DataFrames** to let the engine apply cross-language optimizations via the Catalyst execution planner.

### 61. Extract Customers Data — Simple JSON (17 min)

* **Schema Ingestion Standards**: While running `inferSchema` or relying on implicit JSON evaluation is acceptable for interactive prototyping, production pipelines enforce explicitly declared `StructType` definitions. Defining structural bounds ahead of time prevents expensive object storage preview scans and immunizes downstream schemas against sudden structural data drift.
* **Code Implementation Pattern**:
```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

# Enforcing explicit structure for production safety
customer_schema = StructType([
    StructField("customer_id", StringType(), False),
    StructField("profile_name", StringType(), True),
    StructField("signup_epoch", IntegerType(), True)
])

df_customers = (spark.read
    .schema(customer_schema)
    .json("abfss://raw-zone@storageaccount.dfs.core.windows.net/customers/*.json"))
### 62. Extract Orders Data — Complex JSON as Text (5 min)

* **Semi-Structured Optimization**: Parsing highly nested string JSON blobs via direct string indexing can trigger severe shuffle overhead. Utilizing `from_json()` bound to a strict layout schema allows the underlying engine to evaluate nested elements inline, minimizing memory footprints.
* **Relational Flattening**: Using the `explode()` transformation function to transpose arrays into vertical rows, combined with classic standard object dot notation (e.g., `orders.items`) to un-nest complex JSON maps.

### 63. Extract Memberships Data — Binary File (4 min)

* **Unstructured Ingestion Paths**: Reading raw binary streams (such as image blocks, geospatial maps, or raw PDF data fields) natively into a uniform DataFrame structure using the `binaryFile` format pointer.
* **Metadata Schema Layout**: Each ingested record automatically binds to a standard infrastructure schema containing the row elements: `path` (StringType), `modificationTime` (TimestampType), `length` (LongType), and `content` (BinaryType).

### 64 & 65. Extract Addresses & Payments (TSV / CSV) (5 min + 8 min)

* **Character-Separated Parsing**: Ingesting flat file objects by fine-tuning delimiter expectations (`sep`), text wrappers, and row headers.
* **Corruption Handling Modes**: Configuring structural reading fallback policies to establish robust data quality boundaries:
* `PERMISSIVE` (Default): Inserts corrupted rows into a designated null block or dumps malformed strings into a custom bad-record field while continuing execution.
* `DROPMALFORMED`: Silently ignores and filters out rows containing invalid elements during the initial read.
* `FAILFAST`: Immediately crashes execution and outputs a driver stack trace the moment a single structural schema anomaly is encountered.



### 66. Extract Refunds Data — SQL Table via JDBC (4 min)

* **Secure Metadata Ingestion**: Always encapsulate database connection properties inside isolated **Databricks Secrets** vaults, calling `dbutils.secrets.get()` to resolve credentials securely at runtime rather than exposing plain text keys inside repo configurations.
* **Parallelizing JDBC Bottlenecks**: Default JDBC extractions run over a single network socket connection to a single cluster executor, creating an acute data engineering bottleneck. Eliminate this by declaring explicit boundary parameters to force parallel multi-executor reads across connection threads.
* **Code Implementation Pattern**:
```python
df_refunds = (spark.read
    .format("jdbc")
    .option("url", "jdbc:postgresql://rds-cluster-prod.internal:5432/finance")
    .option("dbtable", "transaction_refunds")
    # Concurrent partitioning parameters
    .option("partitionColumn", "refund_date")
    .option("lowerBound", "2026-01-01")
    .option("upperBound", "2026-12-31")
    .option("numPartitions", "16")
    .option("user", dbutils.secrets.get(scope="jdbc-scope", key="db-user"))
    .option("password", dbutils.secrets.get(scope="jdbc-scope", key="db-pass"))
    .load())

```

## Important Exam Considerations

* **Narrow vs. Wide Execution Scopes**: Ensure you can classify internal operations smoothly for the exam. Narrow transformations (e.g., `select()`, `filter()`, `withColumn()`) execute completely within an isolated worker partition without requiring network data exchanges. Wide transformations (e.g., `groupBy()`, `join()`, `distinct()`) force data shuffles across worker nodes, drawing a boundary between execution stages.
* **Broadcast Join Optimization Bounds**: When executing relational joins combining a massive factual transactional DataFrame with a small lookup table ($\le 10\text{MB}$ by default), you can optimize performance by using an explicit broadcast hint: `broadcast(small_df)`. This action pushes a complete copy of the lookup table to every worker node, converting an expensive network shuffle join into a highly efficient local map-side join.
* **Partition Tuning Math**: Over-partitioning data introduces severe storage metadata scanning latencies, while under-partitioning causes poor CPU utilization and triggers out-of-memory errors. The target operational standard for production environments is to adjust partition volumes to maintain individual data file sizes on disk between **100MB and 1GB**.

---

[← Back to Section 8: Advanced Transformations & Delta Lake](https://www.google.com/search?q=./section08-readme.md) | [Next Section: Section 10: Apache Spark - Transformations & Complex Data Structures →](https://www.google.com/search?q=./section10-readme.md)
