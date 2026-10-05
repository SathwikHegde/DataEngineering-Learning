# Section 14: Lakeflow Declarative Pipelines (DLT) — Overview

This section details Delta Live Tables (DLT) within the Lakeflow architecture, marking a transition from imperative pipeline construction—requiring manual management of Structured Streaming offsets, stateful checkpoints, and retry logic—to a declarative execution model. Engineers specify target table states and data quality constraints, while the underlying execution engine autonomously provisions compute resources, resolves Directed Acyclic Graph (DAG) dependencies, and enforces deterministic structural lineage.

Refer to `image_628c04.png` for the topological dependency graph and curriculum sequencing.

---

## Section Overview

* **Total Duration:** 20 minutes
* **Total Modules:** 3
* **Primary Focus:** Declarative ETL semantics, automated DAG compilation, stateful stream orchestration, and polyglot runtime syntax.

---

## Curriculum Breakdown

| Module # | Title | Duration | Core Architectural Outcome |
| --- | --- | --- | --- |
| **100** | **Introduction to Delta Live Tables** | 8 min | Evaluating the FinOps unit economics and operational efficiency of declarative engineering compared to imperative Spark Structured Streaming workflows. |
| **101** | **DLT Architecture** | 4 min | Analyzing Lakeflow engine Abstract Syntax Tree (AST) parsing to compile visual DAGs and allocate elastic, auto-scaling compute clusters. |
| **102** | **Programming with DLT** | 9 min | Implementing foundational code topologies utilizing ANSI SQL and native PySpark declarative decorators. |

---

## Core Architectural Concepts

### 1. Declarative vs. Imperative Execution Semantics

Legacy imperative topologies mandate explicit programming of micro-batch mechanics (`.readStream`, `.writeStream`, `.option("checkpointLocation", path)`) and trigger intervals. Lakeflow Declarative Pipelines abstract this operational overhead. The runtime compiler parses execution scripts, infers topological dataset relationships via declarative references, and autonomously generates an optimized physical execution DAG.

### 2. Polyglot Runtime Syntax

DLT supports declarative ANSI SQL and Python natively. Multi-language execution graphs are fully supported within a unified pipeline environment, but strict language isolation at the file level is required.

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

* **Streaming Live Tables (`STREAMING LIVE TABLE`)**: Engineered for stateful, append-only ingestion topologies. The execution engine tracks high-water marks within the transaction log, processing exclusively novel micro-batches appended since the preceding pipeline execution.
* **Materialized Views / Live Tables (`LIVE TABLE`)**: Batch-oriented assets that execute full table scans to recompute complex aggregations, analytical window functions, and business logic from scratch upon each pipeline invocation.

---

## Important Exam Considerations

* **Non-Interactive Compilation Constraint**: DLT scripts cannot be executed imperatively cell-by-cell within an interactive REPL workspace cluster; attempting to do so yields a runtime exception. DLT code must be compiled and executed exclusively via an active Pipeline Deployment within the Lakeflow control plane.
* **Data Quality Guardrails (Expectations)**: DLT enforces data integrity through declarative Expectations evaluated at runtime. The execution routing behaviors are strictly defined:
* `ON VIOLATION ALLOW`: Logs constraint violations transparently to the event telemetry log while permitting anomalous records to propagate downstream.
* `ON VIOLATION DROP`: Silently filters anomalous records from the micro-batch prior to materialization in the target Delta table.
* `ON VIOLATION FAIL`: Instantly aborts the active pipeline DAG upon anomaly detection, preventing downstream data contamination.


* **Deterministic Lineage Generation**: By enforcing explicit dataset dependency mappings via `LIVE.<table_name>` namespace references, the Lakeflow compiler natively captures and visualizes schema dependencies down to the granular column level without requiring manual instrumentation.

---

[← Back to Section 13: Delta Lake Architecture & Internal Mechanics](https://www.google.com/search?q=./section13-readme.md) | [Next Section: Section 15: Lakeflow Spark Declarative Pipelines Project →](https://www.google.com/search?q=./section15-readme.md)
