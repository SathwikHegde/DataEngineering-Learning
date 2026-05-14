# Section 9: Apache Spark - Querying Data (PySpark)

This section of the **Databricks Certified Data Engineer Associate** course focuses on data extraction and interaction using the **PySpark DataFrame API**. As of 2026, PySpark has become the primary interface for production data engineering, offering near-identical performance to Scala thanks to the **Catalyst Optimizer** and **Tungsten** execution engine.

Refer to **image_66d9bb.png** for the lesson sequence covered in this module.

---

## 60. Introduction to PySpark

* **The 2026 Standard**: In Spark 4.0+, **Spark Connect** is the default client protocol, allowing you to connect to a remote Spark cluster as a thin client without a local JVM.
* **Architecture**: Understand the difference between **Transformations** (lazy evaluation) and **Actions** (triggering computation).
* **RDD Deprecation**: Resilient Distributed Datasets (RDDs) are now legacy; all modern development should use the **DataFrame API** for better performance.

---

## Data Extraction Patterns

### 61. Extract Customers Data - Simple JSON

* **Schema Inference**: Learn to use `inferSchema` for quick exploration, but understand that defining an explicit `StructType` is preferred for production stability.
* **Code Example**:
```python
df_customers = spark.read.json("path/to/customers.json")

```



### 62. Extract Orders Data - Complex JSON as Text

* **Handling Nested Data**: In 2026, the "Pro" way to handle complex JSON is using `map_from_arrays` or `from_json` with a predefined schema to avoid costly shuffles and improve performance by up to 35%.
* **Transformation**: Use `explode()` to flatten arrays and dot notation (e.g., `orders.items`) to access nested fields.

### 63. Extract Memberships Data - Binary File

* **Native Reader**: Use the `binaryFile` format to read images, PDFs, or other non-text files as a DataFrame where each row contains the file's raw bytes and metadata (path, modification time, length).
* **Code Example**:

```python
  df_binary = spark.read.format("binaryFile").load("path/to/files/")

```

### 64. Extract Addresses Data - TSV

* **Tab-Separated Values**: Since TSVs are a variant of CSV, use the CSV reader with a custom delimiter.
* **Code Example**:

```python
  df_addresses = spark.read.option("sep", "\t").option("header", "true").csv("path/to/addresses.tsv")

```

### 65. Extract Payments Data - CSV

* **Standard Ingestion**: Mastering the common flags: `header`, `inferSchema`, `mode` (PERMISSIVE, DROPMALFORMED, or FAILFAST), and `nullValue`.

### 66. Extract Refunds Data - SQL Table via JDBC

* **Secure Connections**: Always use **Databricks Secrets** (`dbutils.secrets.get`) to retrieve database credentials rather than hardcoding them.
* **JDBC Partitioning**: To avoid single-executor bottlenecks, specify partitioning parameters like `partitionColumn`, `lowerBound`, `upperBound`, and `numPartitions` to enable parallel reads from the relational database.
* **Code Example**:

```python
  df_refunds = (spark.read
    .format("jdbc")
    .option("url", jdbc_url)
    .option("dbtable", "refunds_table")
    .option("user", dbutils.secrets.get("scope", "user"))
    .option("password", dbutils.secrets.get("scope", "password"))
    .load())

```

---

## Performance Checklist for 2026

* **Broadcast Joins**: Manually hint small DataFrames to be broadcast to avoid expensive shuffles across the cluster.
* **Partition Management**: Use `repartition()` for increasing parallelism or `coalesce()` for reducing partitions efficiently before writing data.
* **Unity Catalog Integration**: Ensure your clusters are using **Shared** or **Single User** access modes to comply with modern governance standards for all PySpark workloads.

```

```