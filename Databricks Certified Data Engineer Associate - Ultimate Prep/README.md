# Section 7: Apache Spark - Querying Data (SQL)

This section focuses on the "Extract" and "Transform" phases of the ETL lifecycle using **Spark SQL**. You will learn how to interact with various file formats (JSON, TSV, CSV, Binary) and how to structure your data using Views and External Tables within the Databricks Lakehouse.

---

## 🏛️ Section Overview

* **Total Duration:** 1 hour 25 minutes
* **Total Lessons:** 11
* **Primary Focus:** Data extraction patterns, the `read_files` function, and modern Unity Catalog table management.

---

## 🛠️ Section Modules

| Lesson # | Title | Duration | Key Learning Outcome |
| --- | --- | --- | --- |
| 35 | **Simple JSON Extraction** | 13 min | Querying flat JSON files directly using SQL. |
| 36/37 | **Views & Temporary Views** | 13 min | Creating reusable virtual tables (Session vs. Global scope). |
| 38 | **Complex JSON as Text** | 8 min | Handling nested structures and raw string extraction. |
| 39/40 | **Binary File Extraction** | 5 min | Reading non-textual data and cluster compute requirements. |
| 41 | **`read_files` Function (TSV)** | 9 min | Using the modern SQL table function to read raw files. |
| 42 | **CSV via External Table** | 15 min | Defining schema and location for data stored outside the managed warehouse. |
| 43 | **⚠️ Important Note (Post-Dec 2025)** | 1 min | Mandatory Unity Catalog enforcement for new subscriptions. |
| 44 | **SQL Table via External Table** | 14 min | Orchestrating multi-layer data extraction for Refunds data. |
| 45 | **Querying via PySpark** | 8 min | Bridging the gap between SQL syntax and the DataFrame API. |

---

## 🚀 Key Technical Concepts

### 1. The `read_files()` Function

A standout feature in the 2026 curriculum, `read_files()` is a powerful SQL table function. It allows you to query cloud storage (S3/ADLS/GCS) directly without pre-defining a table.

* **Example:** `SELECT * FROM read_files('path/to/data', format => 'tsv')`
* **Why it matters:** It supports schema inference and provides metadata about the source files (like file name and size) directly in your query results.

### 2. External Tables vs. Managed Tables

* **Managed Tables:** Databricks manages both the metadata and the actual data files. If you drop the table, the data is deleted.
* **External Tables:** You provide a `LOCATION`. Databricks manages the metadata, but the data files remain in your cloud storage even if the table is dropped. This is the standard for "Bronze" layer ingestion.

### 3. ⚠️ The "December 18, 2025" Rule (Lesson 43)

If your Databricks subscription was created after **December 18, 2025**, legacy features are disabled by default.

* **No Hive Metastore:** You must use **Unity Catalog** for all table definitions.
* **No DBFS Root:** Accessing the legacy `dbfs:/` root is restricted; you must use **Volumes** or **External Locations** to interact with files.
* **Isolation Enforcement:** Clusters must use "Shared" or "Single User" access modes to comply with modern security standards.

---

## 💡 Pro-Tips for Data Engineers

1. **Prefer `read_files` over `json.` pathing:** For modern pipelines, `read_files` is more performant and offers better control over file-level metadata.
2. **Temporary Views for Logic, Tables for Storage:** Use `CREATE OR REPLACE TEMP VIEW` for intermediate transformation steps that don't need to persist after your cluster shuts down.
3. **Complex JSON Handling:** When dealing with highly nested JSON, use the `:` operator (e.g., `payload:user:id`) to traverse fields efficiently in SQL.

---

[Next Section: Advanced Transformations & Delta Lake →](https://www.google.com/search?q=./section8-readme.md)

What specific file format or data source are you planning to ingest first using these Spark SQL techniques?