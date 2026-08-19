# Section 15: Lakeflow Spark Declarative Pipelines (SDP) — Project

This section delivers an end-to-end, production-grade implementation project using **Lakeflow Spark Declarative Pipelines (SDP)** (formerly Delta Live Tables / DLT). Over the course of 2 hours and 40 minutes, you will build a complete Medallion architecture—progressing from raw multi-source ingestion to conformed analytical consumption layers while enforcing data quality validation frameworks, Change Data Capture (CDC) synchronization, and multi-catalog Unity Catalog governance.

Refer to `image_08e047.png` for the complete sequence of project implementation tasks.

---

## Section Overview

* **Total Duration:** 2 Hours 40 Minutes
* **Total Modules:** 14
* **Core Objective:** Implement a multi-hop Medallion DAG using declarative SQL and Python APIs, enforce data governance constraints with Expectations, execute automated SCD Type 1 and Type 2 synchronization via `APPLY CHANGES INTO`, and orchestrate cross-catalog schema publication.

---

## Project Implementation Breakdown

| Module | Title | Duration | Language | Technical Focus Area |
| --- | --- | --- | --- | --- |
| **104** | **Project Overview** | 2 min | — | System topology, schema contracts, and architecture requirements. |
| **105** | **Cluster Configuration & Azure VM Quota** | 12 min | — | Sizing dedicated DLT cluster runtimes within cloud compute subscription limits. |
| **106** | **Project Environment Set-up** | 14 min | — | Storage provisioning, sample telemetry staging, and source path mapping. |
| **107** | **Intro to Streaming Tables** | 10 min | SQL | Incremental append ingestion using `CREATE OR REFRESH STREAMING TABLE`. |
| **108** | **Recent Changes to the UI** | 2 min | — | Telemetry monitoring, DAG visualization, and data quality metrics dashboards. |
| **109** | **Create CircuitBox Pipeline** | 25 min | — | Pipeline graph definition, target catalog binding, and compilation analysis. |
| **110** | **Intro to SDP Expectations** | 19 min | SQL | Declarative constraint enforcement (`ALLOW`, `DROP`, `FAIL`) via SQL. |
| **111** | **Intro to Apply Changes** | 17 min | SQL | CDC ingestion and state synchronization implementing **SCD Type 1**. |
| **112** | **Creating SDP Datasets** | 12 min | Python | Multi-language DAG extension leveraging the `dlt` PySpark module. |
| **113** | **Implementing SDP Expectations** | 11 min | Python | Programmatic data validation rules using `@dlt.expect` decorators. |
| **114** | **Implementing Slowly Changing Dimensions** | 10 min | Python | Historical lineage tracking via `dlt.apply_changes()` for **SCD Type 2**. |
| **115** | **Process Orders Data — Assignment** | 10 min | Polyglot | Independent module building the relational Silver-tier order processing stage. |
| **116** | **Intro to Materialized Views** | 16 min | Polyglot | Pre-computed, incrementally refreshed Gold aggregations using Materialized Views. |
| **117** | **Publish to Multiple Catalogs/Schemas** | 1 min | — | Centralized multi-namespace deployment within the Unity Catalog hierarchy. |

---

## Core Architectural Implementation Concepts

### 1. Declarative Data Quality Enforcement (Expectations)

Expectations integrate data quality validation directly into the pipeline execution graph without interrupting the underlying streaming query plan. Validation policies are evaluated at the row level with three distinct enforcement actions:

* **`ON VIOLATION ALLOW` (Retain & Track)**: Logs constraint violation telemetry to the internal event log while allowing invalid rows to flow downstream.
```sql
CONSTRAINT valid_timestamp EXPECT (event_timestamp <= current_timestamp()) ON VIOLATION ALLOW

```


* **`ON VIOLATION DROP` (Filter Invalid)**: Atomically drops rows that fail the boolean predicate before they reach target storage, preventing downstream pipeline contamination.
```sql
CONSTRAINT positive_price EXPECT (unit_price > 0) ON VIOLATION DROP

```


* **`ON VIOLATION FAIL` (Halt Execution)**: Immediately aborts pipeline execution and throws a runtime exception if any row violates critical business constraints.
```sql
CONSTRAINT valid_id EXPECT (order_id IS NOT NULL) ON VIOLATION FAIL

```



### 2. Change Data Capture via `APPLY CHANGES INTO`

The declarative `APPLY CHANGES INTO` API abstracts manual, high-overhead `MERGE INTO` operations, handling out-of-order records, late-arriving updates, and schema synchronization automatically:

* **SCD Type 1 (Deterministic Overwrite)**: Maintains current state by overwriting existing records matching the declared `KEYS` with the latest record based on `SEQUENCE BY` ordering.
```sql
APPLY CHANGES INTO live.dim_customers
FROM stream(live.stage_customers)
KEYS (customer_id)
APPLY AS DELETE WHEN operation = 'DELETE'
SEQUENCE BY event_timestamp
COLUMNS * EXCEPT (operation);

```


* **SCD Type 2 (Historical Version Tracking)**: Preserves historical state by maintaining row-level validity intervals using automated metadata columns (`__START_AT`, `__END_AT`), setting `__END_AT = NULL` for the active version record.
```python
dlt.apply_changes(
    target="dim_customers_scd2",
    source="stage_customers",
    keys=["customer_id"],
    sequence_by="event_timestamp",
    apply_as_deletes="operation = 'DELETE'",
    stored_as_scd_type="2",
)

```



### 3. Streaming Tables vs. Materialized Views

* **Streaming Tables (`STREAMING TABLE`)**: Backed by Spark Structured Streaming engines and checkpoint directories. They ingest append-only, high-throughput source data incrementally, processing each file or transaction log record exactly once.
* **Materialized Views (`MATERIALIZED VIEW`)**: Compute deterministic, pre-aggregated query states over historical or upstream datasets. Incremental refresh engines automatically compute only the net changes from the source tables, optimizing read-side query performance for downstream consumers and business intelligence dashboards.

---

## Important Exam Considerations

* **Polyglot Pipeline Isolation Boundaries**: A single Lakeflow Declarative Pipeline DAG can execute both SQL and Python source artifacts concurrently. However, **individual source files must be homogeneous**—mixing SQL and Python code within the same notebook is strictly prohibited.
* **Unified Namespace Deployment**: Pipelines managed by Unity Catalog support writing outputs to specific 3-tier namespaces (`catalog.schema.table`), allowing Bronze, Silver, and Gold targets to be published to distinct governance catalogs within a single pipeline configuration.
* **DLT Event Log Telemetry**: System metrics, execution lineage, and data quality expectation statistics are automatically recorded in an internal Delta table (the DLT Event Log). You can query this event log directly using Spark SQL to audit historical pipeline run durations, expectation drop ratios, and failure root causes.

---

[← Back to Section 14: Lakeflow Declarative Pipelines Overview](https://www.google.com/search?q=./section14-readme.md) | [Next Section: Section 16: Lakeflow Jobs & Workflow Orchestration →](https://www.google.com/search?q=./section16-readme.md)
