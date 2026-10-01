# Section 14: Lakeflow Declarative Pipelines (DLT) — Overview

This module introduces **Delta Live Tables (DLT)** within the unified Lakeflow architecture. DLT executes a paradigm shift from imperative pipeline engineering—which necessitates manual orchestration of Structured Streaming offsets, checkpoint states, and retry heuristics—to a highly resilient declarative framework. Data engineers define target end-states and data quality invariants, abstracting the underlying execution engine to autonomously orchestrate compute infrastructure, resolve Directed Acyclic Graph (DAG) dependencies, and enforce deterministic structural lineage.

Refer to `image_628c04.png` for the topological dependency graph and curriculum sequencing.

---

## Section Overview

* **Total Duration:** 20 minutes
* **Total Modules:** 3
* **Primary Focus:** Declarative ETL semantics, automated DAG synthesis, stateful stream processing, and polyglot DLT runtime syntax.

---

## Curriculum Breakdown

| Module # | Title | Duration | Core Architectural Outcome |
| --- | --- | --- | --- |
| **100** | **Introduction to Delta Live Tables** | 8 min | Analyzing the FinOps and operational efficiency of declarative engineering versus imperative Spark Structured Streaming pipelines. |
| **101** | **DLT Architecture** | 4 min | Examining how the Lakeflow engine parses abstract syntax trees (AST) to synthesize visual DAGs and provision elastic autoscaling compute. |
| **102** | **Programming with DLT** | 9 min | Defining foundational code topologies utilizing ANSI SQL and native PySpark declarative decorators. |

---

## Core Architectural Concepts

### 1. Declarative vs. Imperative Execution Semantics

In legacy imperative topologies, engineers must explicitly program micro-batch mechanics (`.readStream`, `.writeStream`, `.option("checkpointLocation", path)`) and manage transaction triggers. With Lakeflow Declarative Pipelines, this operational overhead is completely abstracted. The runtime compiler inspects the execution scripts, infers dataset topological relationships based on declarative references, and autonomously synthesizes an optimized end-to-end physical execution DAG.

### 2. Polyglot Runtime Syntax

DLT natively supports both declarative ANSI SQL and Python. While multi-language execution graphs are fully supported within a unified pipeline environment, strict file-level language isolation is mandated.

* **SQL Implementation Pattern:**

```sql
-- Declaring a stateful incremental ingestion table utilizing Auto Loader natively
CREATE OR REFRESH STREAMING LIVE TABLE bronze_customers
AS SELECT * FROM cloud_files("abfss://raw-zone@storageaccount.dfs.core.windows.net/customers", "json");

```

* **Python Implementation Pattern:**

```python
import dlt

# Decorating a Python function to register a managed pipeline node
@dlt.table(name="bronze_customers")
def bronze_customers():
    return (spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "json")
        .load("abfss://raw-zone@storageaccount.dfs.core.windows.net/customers"))

```

### 3. Object State Classifications

* **Streaming Live Tables (`STREAMING LIVE TABLE`)**: Optimized for stateful, append-only ingestion topologies. The engine tracks high-water marks within the transaction log to process exclusively novel micro-batches materialized since the preceding pipeline execution.
* **Materialized Views / Live Tables (`LIVE TABLE`)**: Traditional batch assets that execute full table scans to recompute complex aggregations, analytical window functions, and unified business logic from scratch during each pipeline invocation.

---

## Important Exam Considerations

* **Non-Interactive Compilation Constraint**: Certification scenarios rigorously test the execution boundaries of DLT. DLT scripts **cannot** be executed imperatively cell-by-cell within an interactive REPL workspace cluster. Attempting to do so yields a runtime exception. DLT code must be compiled and executed exclusively via an active Pipeline Deployment within the Lakeflow control plane.
* **Data Quality Guardrails (Expectations)**: DLT enforces data integrity via declarative Expectations evaluated natively during execution. You must understand the distinct routing behaviors:
* `ON VIOLATION ALLOW`: Logs constraint violations transparently to the event telemetry log while permitting anomalous records to propagate downstream.
* `ON VIOLATION DROP`: Silently filters anomalous records from the micro-batch prior to materialization in the target Delta table.
* `ON VIOLATION FAIL`: Instantly aborts the active pipeline DAG upon anomaly detection, preventing downstream data contamination.


* **Deterministic Lineage Generation**: By enforcing explicit dataset dependency mappings (via `LIVE.<table_name>` namespace references), the Lakeflow compiler natively captures and visualizes schema dependencies down to the granular column level without requiring manual instrumentation.

---

[← Back to Section 13: Delta Lake Architecture & Internal Mechanics](https://www.google.com/search?q=./section13-readme.md) | [Next Section: Section 15: Lakeflow Spark Declarative Pipelines Project →](https://www.google.com/search?q=./section15-readme.md)
