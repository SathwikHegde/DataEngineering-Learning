
Section 7: Apache Spark — Querying Data (SQL)

This section covers the Extract and Transform stages of the data engineering lifecycle using Spark SQL. You will learn how to query common file formats (JSON, TSV, CSV, and Binary) and organize your storage layer with Views, Managed Tables, and External Tables under modern governance rules.
Check image_b9238b.png (or your course dashboard) for the full lesson order in this module.

Section Overview

• Total Duration: 1 hour 25 minutes
• Total Lessons: 11
• Core Focus: Direct file querying, schema definitions, file metadata tracking with read_files(), and Unity Catalog rules.

Curriculum Breakdown


Lesson #	Title	Duration	Key Learning Goal
35	Simple JSON Extraction	13 min	Query flat JSON files directly from cloud storage using file paths.
36/37	Views & Temporary Views	13 min	Build reusable virtual tables across session-level and global scopes.
38	Complex JSON as Text	8 min	Navigate nested JSON structures and pull out semi-structured fields.
39/40	Binary File Extraction	5 min	Read raw binary files and configure the required cluster settings.
41	read_files Function (TSV)	9 min	Use the built-in table-valued function to ingest tab-separated data.
42	CSV via External Table	15 min	Define explicit schemas and storage locations for external CSV files.
43	Mandatory Security Update	1 min	Review required Unity Catalog security defaults in modern workspaces.
44	SQL Table via External Table	14 min	Set up multi-step data extractions from external relational sources.
45	Querying via PySpark	8 min	Combine SQL queries with the PySpark DataFrame API.


Key Technical Concepts


1. The read_files() Table-Valued Function

A core part of the updated curriculum, read_files() replaces older path-based syntax (such as SELECT * FROM json.`path` ). This built-in SQL function reads files directly from cloud object storage (AWS S3, Azure ADLS, or GCP GCS) inside standard queries.
• Example Syntax:
sql
SELECT * FROM read_files(
  'abfss://raw-zone@storageaccount.dfs.core.windows.net/orders/',
  format => 'tsv',
  header => 'true'
);
Use code with caution.
• Why It Matters: It automatically includes hidden file-level metadata fields (such as _metadata.file_name, _metadata.file_size, and _metadata.file_block_start), letting you track and audit source files directly inside your SELECT queries.

2. Table Objects: Managed vs. External

• Managed Tables: Unity Catalog manages both the table metadata and the underlying Parquet data files inside your workspace's default storage location. Running DROP TABLE permanently deletes both the metadata and the physical data files.
• External Tables: You specify an external cloud path using the LOCATION keyword, so Databricks tracks only the table metadata. Running DROP TABLE removes only the catalog entry while leaving the underlying data files intact in cloud storage—making External Tables ideal for raw Bronze-layer data.

3. Unity Catalog Enforcement Rules (Lesson 43)

Modern workspaces enforce strict governance boundaries by disabling legacy, unmanaged workflows by default:
• Hive Metastore Disabled: The legacy hive_metastore catalog is restricted. All new tables must be created inside a Unity Catalog metastore.
• DBFS Root Locked Down: Direct access to the un-governed dbfs:/ root is blocked. Storage paths must point to Unity Catalog Volumes or governed External Locations.
• Cluster Access Modes: Legacy "No Isolation" clusters cannot access governed data. Compute clusters must run in Shared or Single User access mode to enforce data security.

Practical Tips for Data Engineers

• Use the Colon Operator (:) for Nested JSON: Skip complex regex parsing when querying JSON strings. Use inline colon notation to extract nested keys cleanly:
sql
SELECT payload:user:identity:email_address AS user_email FROM bronze.raw_events;
Use code with caution.
• Choose the Right View Scope: Use CREATE OR REPLACE TEMPORARY VIEW for intermediate transformations needed only within your current notebook session. If a view must persist across users or clusters, create a permanent view inside a shared Unity Catalog schema.
• Write Idempotent Table Creation Scripts: When building landing pipelines, use CREATE TABLE IF NOT EXISTS or CREATE OR REPLACE TABLE (CTAS) rather than plain CREATE TABLE statements to prevent automated jobs from failing on existing table names.
← Back to Section 6: Data Objects in the Lakehouse | Next Section: Section 8: Apache Spark — Transforming Data (SQL) →
Would you like me to adjust the tone (e.g., shorter or more technical), or paste another section README to rewrite?
