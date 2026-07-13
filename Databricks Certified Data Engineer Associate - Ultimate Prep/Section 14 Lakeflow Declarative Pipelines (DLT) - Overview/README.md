# Section 14: Lakeflow Declarative Pipelines (DLT) — Overview

This section introduces **Delta Live Tables (DLT)** within the unified **Lakeflow** framework. DLT completely shifts the ETL development paradigm from imperative coding (where you manually manage Spark streams, storage checkpoints, and state recovery retries) to a highly reliable **declarative framework**. Developers simply define the desired data states and data quality bounds, leaving the underlying execution engine to automatically orchestrate the compute infrastructure, manage complex dataset dependencies, and enforce structural lineage.

Refer to `image_628c04.png` for the lesson timeline and structure of this introductory module.

---

## Section Overview

* **Total Duration:** 20 minutes
* **Total Lessons:** 3
* **Primary Focus:** Declarative ETL foundations, automated DAG graph synthesis, and basic multi-language DLT syntax patterns.

---

## Curriculum Breakdown

| Lesson # | Title | Duration | Core Learning Outcome |
| --- | --- | --- | --- |
| **100** | **Introduction to Delta Live Tables** | 8 min | Understanding the cost and operational benefits of declarative engineering over manual Spark streaming pipelines. |
| **101** | **DLT Architecture** | 4 min | How Databricks parses source files to build visual execution graphs and handle elastic autoscaling profiles. |
| **102** | **Programming with DLT** | 9 min | Introductory code structures using both SQL text and PySpark decorative wrappers. |

---

## Core Architectural Concepts

### 1. Declarative vs. Imperative Execution

In a legacy, imperative PySpark streaming job, you must write explicit, operational micro-batch mechanics for parameters like `.readStream`, `.writeStream`, `.option("checkpointLocation", path)`, and transaction triggers.

With **Lakeflow Declarative Pipelines (DLT)**, you shift focus entirely to the end state. The underlying compiler inspects your complete code notebooks, infers the relationships between separate datasets based on queries, and auto-synthesizes an end-to-end processing execution graph (DAG).

### 2. Polyglot DLT Programming Syntax

DLT supports both ANSI SQL and Python natively. While you can connect SQL and Python files within the same overall pipeline execution graph, you must isolate the languages to distinct, dedicated files.

* **SQL Pattern:**
```sql
-- Declaring an incremental ingest table using Auto Loader natively
CREATE OR REFRESH STREAMING LIVE TABLE bronze_customers
AS SELECT * FROM cloud_files("abfss://raw-zone@storageaccount.dfs.core.windows.net/customers", "json");

```
* **Python Pattern:**
```python
import dlt

# Decorating a Python function to register a managed pipeline table
@dlt.table(name="bronze_customers")
def bronze_customers():
    return (spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "json")
        .load("abfss://raw-zone@storageaccount.dfs.core.windows.net/customers"))

```
### 3. Object Classifications Inside the Graph

* **Streaming Live Tables (`STREAMING LIVE TABLE`)**: Optimized for append-only, high-velocity streams. They statefully check historical logs to process *only* files or messages that have arrived since the previous pipeline refresh.
* **Materialized Views / Live Tables (`LIVE TABLE`)**: Traditional batch datasets that re-compute complicated queries, analytical window functions, and business aggregations completely from scratch during each execution step.

---

## Important Exam Considerations

* **The Non-Interactive Execution Constraint**: For the certification exam, remember that you **cannot** execute DLT notebooks cell-by-cell inside an interactive workspace cluster. Attempting to run a cell containing DLT code will throw an error. The source code must be linked directly to an active **Pipeline Deployment** inside the Workflows persona interface.
* **The Syntax of Data Quality Guardrails**: Pay close attention to the structural definition of *Expectations* for data quality enforcement. DLT evaluates records using clean constraints:
* `ON VIOLATION ALLOW`: Logs data exceptions transparently in the background telemetry but passes rows onward.
* `ON VIOLATION DROP`: Filters bad rows out of the stream silently before they land in target storage.
* `ON VIOLATION FAIL`: Crashes the entire active pipeline run immediately when a critical data anomaly is detected.


* **Automatic Lineage Generation**: Because the Lakeflow compilation layer evaluates all tables via explicit relationship mappings (`LIVE.<table_name>`), schema lineage dependencies are tracked natively down to individual table and column levels and rendered directly in the pipeline UI.

---

[← Back to Section 13: Delta Lake Architecture & Internal Mechanics](https://www.google.com/search?q=./section13-readme.md) | [Next Section: Section 15: Lakeflow Spark Declarative Pipelines Project →](https://www.google.com/search?q=./section15-readme.md)
