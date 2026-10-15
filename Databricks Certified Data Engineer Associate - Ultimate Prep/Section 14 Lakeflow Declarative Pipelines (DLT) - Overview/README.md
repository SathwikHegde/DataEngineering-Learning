# Section 14: Lakeflow Declarative Pipelines (DLT) — Overview

This module introduces Delta Live Tables (DLT) as the declarative execution engine within the Databricks Lakeflow architecture. It signifies a fundamental shift from imperative pipeline engineering—which demands manual orchestration of Structured Streaming offsets, stateful checkpointing, and retry heuristics—to a fully declarative paradigm. Data engineers define target end-states and data quality invariants, abstracting the execution plane to autonomously provision elastic compute, resolve Directed Acyclic Graph (DAG) topologies, and materialize deterministic schema lineage.

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
| **100** | **Introduction to Delta Live Tables** | 8 min | Evaluating the FinOps unit economics and operational efficiency of declarative pipelines versus imperative Spark Structured Streaming DAGs. |
| **101** | **DLT Architecture** | 4 min | Analyzing Lakeflow Abstract Syntax Tree (AST) compilation to synthesize visual DAGs and provision elastic compute infrastructure. |
| **102** | **Programming with DLT** | 9 min | Engineering foundational pipeline topologies utilizing ANSI SQL and native PySpark declarative decorators. |

---

## Core Architectural Concepts

### 1. Declarative vs. Imperative Execution Semantics

Legacy imperative architectures require explicit programming of micro-batch triggers and storage checkpoints (e.g., `.readStream`, `.writeStream`, `.option("checkpointLocation", path)`). Lakeflow Declarative Pipelines neutralize this operational overhead. The execution compiler parses source scripts, infers topological dataset dependencies via declarative namespace references, and autonomously generates an optimized physical DAG.

### 2. Polyglot Runtime Syntax

DLT natively supports declarative ANSI SQL and Python. Multi-language execution graphs are fully supported within a unified pipeline runtime, though strict language isolation is enforced at the individual file level.

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

* **Streaming Live Tables (`STREAMING LIVE TABLE`)**: Optimized for stateful, append-only ingestion topologies. The engine evaluates high-water marks within the transaction log, strictly processing novel micro-batches materialized since the preceding pipeline execution.
* **Materialized Views / Live Tables (`LIVE TABLE`)**: Batch-oriented assets that execute full table scans to recompute complex analytical aggregations, window functions, and business logic from scratch upon each pipeline trigger.

---

## Important Exam Considerations

* **Non-Interactive Compilation Constraint**: Certification parameters strictly test DLT execution boundaries. DLT scripts cannot be executed imperatively cell-by-cell within an interactive REPL workspace cluster; doing so triggers a runtime exception. DLT code must be compiled and orchestrated exclusively via an active Pipeline Deployment within the Lakeflow control plane.
* **Data Quality Guardrails (Expectations)**: DLT enforces data integrity via declarative Expectations evaluated natively at runtime. The execution routing behaviors are absolute:
* `ON VIOLATION ALLOW`: Logs constraint violations transparently to the pipeline telemetry event log while permitting anomalous records to propagate downstream.
* `ON VIOLATION DROP`: Silently filters non-compliant records from the micro-batch prior to materialization in the target Delta table.
* `ON VIOLATION FAIL`: Instantly aborts the active pipeline DAG upon anomaly detection, preventing all downstream data contamination.


* **Deterministic Lineage Generation**: By enforcing explicit dataset dependency mappings (via `LIVE.<table_name>` namespace bindings), the Lakeflow compiler natively captures and visualizes schema dependencies down to the granular column level without manual code instrumentation.

---

[← Back to Section 13: Delta Lake Architecture & Internal Mechanics](https://www.google.com/search?q=./section13-readme.md) | [Next Section: Section 15: Lakeflow Spark Declarative Pipelines Project →](https://www.google.com/search?q=./section15-readme.md)
