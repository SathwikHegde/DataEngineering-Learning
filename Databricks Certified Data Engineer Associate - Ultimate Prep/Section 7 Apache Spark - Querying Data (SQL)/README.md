# Section 7: Apache Spark — Querying Data (SQL)

This section focuses on the **Extract** and **Transform** phases of the data engineering lifecycle using **Spark SQL**. You will master the techniques required to interact with diverse file formats (JSON, TSV, CSV, Binary) and learn how to structure your storage layer using Views, Managed Tables, and External Tables under modern governance frameworks.

Refer to `image_b9238b.png` (or your course dashboard) for the lesson sequence covered in this module.

---

## Section Overview

* **Total Duration:** 1 hour 25 minutes
* **Total Lessons:** 11
* **Primary Focus:** Native data extraction patterns, declarative schema mapping, file-level metadata tracking via `read_files()`, and Unity Catalog compliance.

---

## Curriculum Breakdown

| Lesson # | Title | Duration | Key Learning Outcome |
| --- | --- | --- | --- |
| **35** | **Simple JSON Extraction** | 13 min | Querying flat JSON datasets directly from object storage via raw paths. |
| **36/37** | **Views & Temporary Views** | 13 min | Creating reusable virtual transformations (Session vs. Global application scopes). |
| **38** | **Complex JSON as Text** | 8 min | Traversing deeply nested paths and extracting semi-structured attributes. |
| **39/40** | **Binary File Extraction** | 5 min | Ingesting non-textual raw streams and evaluating necessary cluster parameters. |
| **41** | **`read_files` Function (TSV)** | 9 min | Utilizing the native table-valued function to stream tab-separated records. |
| **42** | **CSV via External Table** | 15 min | Declaring explicit schemas and cloud pointer locations for unmanaged data storage. |
| **43** | **⚠️ Mandatory Security Update** | 1 min | Understanding strict Unity Catalog enforcement policies for modern workspaces. |
| **44** | **SQL Table via External Table** | 14 min | Orchestrating multi-layer data extractions for transactional database targets. |
| **45** | **Querying via PySpark** | 8 min | Bridging the gap between declarative SQL text and the DataFrame API. |

---

## Key Technical Concepts

### 1. The `read_files()` Table-Valued Function

A primary focus of the modern curriculum, `read_files()` replaces legacy path-based querying (e.g., `SELECT * FROM json.`path``). This native SQL table function acts as a high-performance scanner for cloud object storage (AWS S3, Azure ADLS, GCP GCS) directly within your queries.

* **Syntax Pattern:**
```sql
SELECT * FROM read_files(
  'abfss://raw-zone@storageaccount.dfs.core.windows.net/orders/',
  format => 'tsv',
  header => 'true'
);

```
* **Why it matters:** It automatically exposes hidden file-level metadata columns (such as `_metadata.file_name`, `_metadata.file_size`, and `_metadata.file_block_start`), allowing you to audit file ingestion sources directly in your select statements.

### 2. Table Objects: Managed vs. External

* **Managed Tables:** Unity Catalog completely controls both the metadata registration and the underlying physical Parquet data files inside your workspace's default storage root. If you execute a `DROP TABLE` command, **both the metadata and the actual physical data files are permanently deleted.**
* **External Tables:** You provision an explicit external storage path using the `LOCATION` clause. Databricks manages only the metadata catalog state. If you execute a `DROP TABLE` command, **only the catalog reference is removed; the physical underlying raw data files remain completely safe in your cloud bucket.** This is the architectural standard for Bronze-layer historical extraction.

### 3. Unity Catalog Enforcement Boundaries (Lesson 43)

Workspaces established under modern governance frameworks disable legacy, unmanaged patterns by default:

* **Hive Metastore Deprecation:** The legacy `hive_metastore` catalog is locked down. Every new table asset must reside inside a Unity Catalog metastore container.
* **DBFS Root Restriction:** Accessing the un-governed `dbfs:/` root is heavily restricted. Storage paths must use **Unity Catalog Volumes** or secure **External Locations**.
* **Compute Access Modes:** Traditional "No Isolation" clusters cannot interact with governed data assets. Workloads must execute on **Shared** or **Single User** compute instances to comply with data-level security boundaries.

---

## Pro-Tips for Data Engineers

* **Leverage the Colon Operator (`:`) for Semi-Structured JSON:** Avoid writing convoluted extraction regex string splits. When querying nested JSON blobs, use the inline path traversal operator to navigate keys cleanly:
```sql
SELECT payload:user:identity:email_address AS user_email FROM bronze.raw_events;

```


* **Scoping Virtual Assets Responsibly:** Use `CREATE OR REPLACE TEMPORARY VIEW` for intermediate transformation steps that only need to live for the duration of your active notebook session. If you need a view to persist across different user sessions or separate clusters, promote it to a standard permanent view inside a shared Unity Catalog schema.
* **Idempotent Table Initialization:** When setting up your multi-hop landing zones, prefer the stability of `CREATE TABLE IF NOT EXISTS` or `CREATE OR REPLACE TABLE` (CTAS) patterns over simple create clauses to keep automated job tasks from failing due to object naming collisions.

---

[← Back to Section 6: Data Objects in the Lakehouse](https://www.google.com/search?q=./section06-readme.md) | [Next Section: Section 8: Apache Spark — Transforming Data (SQL) →](https://www.google.com/search?q=./section08-readme.md)
