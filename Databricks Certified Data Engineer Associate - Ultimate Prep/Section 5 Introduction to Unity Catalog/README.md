# Section 15: Lakeflow Spark Declarative Pipelines (SDP) — Project

This section provides an end-to-end technical implementation guide for Lakeflow Spark Declarative Pipelines (SDP), formerly Delta Live Tables (DLT). The 2-hour and 40-minute module focuses on architecting an enterprise-grade Medallion pipeline, progressing from raw multi-protocol ingestion to conformed Gold layers while enforcing row-level data quality, Change Data Capture (CDC) synchronization, and Unity Catalog multi-namespace governance.

Refer to `image_08e047.png` for the detailed project implementation flow and task dependencies.

---

## Section Overview

* **Total Duration:** 2 Hours 40 Minutes
* **Total Modules:** 14
* **Core Objective:** Architect a polyglot Medallion Directed Acyclic Graph (DAG) using declarative SQL and Python APIs, implement constraint enforcement frameworks, automate Slowly Changing Dimension (SCD Type 1 and Type 2) synchronization via `APPLY CHANGES INTO`, and deploy centralized cross-catalog schema definitions.

---

## Project Implementation Breakdown

| Module | Title | Duration | Language | Technical Focus Area |
| --- | --- | --- | --- | --- |
| **104** | **Project Overview** | 2 min | — | System topology, schema contracts, and downstream latency SLAs. |
| **105** | **Cluster Configuration & Azure VM Quota** | 12 min | — | Sizing dedicated DLT cluster runtimes, worker core allocations, and VM quota limits. |
| **106** | **Project Environment Set-up** | 14 min | — | Cloud storage staging, seeding source datasets, and mounting checkpoint paths. |
| **107** | **Intro to Streaming Tables** | 10 min | SQL | Declarative append-only ingestion using `CREATE OR REFRESH STREAMING TABLE`. |
| **108** | **Recent Changes to the UI** | 2 min | — | Pipeline graph visualizer, real-time compute metrics, and event log exploration. |
| **109** | **Create CircuitBox Pipeline** | 25 min | — | Pipeline compilation, setting target schemas, and topological DAG dependency analysis. |
| **110** | **Intro to SDP Expectations** | 19 min | SQL | SQL-based declarative data quality constraints (`ALLOW`, `DROP`, `FAIL`). |
| **111** | **Intro to Apply Changes** | 17 min | SQL | Declarative CDC ingestion and state reconciliation implementing **SCD Type 1**. |
| **112** | **Creating SDP Datasets** | 12 min | Python | Multi-language DAG extension leveraging the `@dlt.table` and `@dlt.view` decorators. |
| **113** | **Implementing SDP Expectations** | 11 min | Python | Programmatic data validation rules using `@dlt.expect_*` Python decorators. |
| **114** | **Implementing Slowly Changing Dimensions** | 10 min | Python | Historical lineage tracking via `dlt.apply_changes()` targeting **SCD Type 2**. |
| **115** | **Process Orders Data — Assignment** | 10 min | Polyglot | Independent milestone building the relational Silver-tier order normalization stage. |
| **116** | **Intro to Materialized Views** | 16 min | Polyglot | Incremental computation of aggregated Gold consumption layers via Materialized Views. |
| **117** | **Publish to Multiple Catalogs/Schemas** | 1 min | — | Multi-namespace publication across development, staging, and production Unity Catalogs. |

---

## Core Architectural Implementation Concepts

### 1. Declarative Data Quality Enforcement (Expectations)

Expectations integrate data quality validation directly into the pipeline execution graph without breaking the underlying Spark Structured Streaming query plan. Constraints evaluate row-by-row with three operational enforcement actions:

* **`ON VIOLATION ALLOW` (Audit & Retain):** Logs constraint violation metrics directly to the system event log while allowing non-compliant records to proceed downstream for auditing.
```sql
CONSTRAINT valid_timestamp EXPECT (event_timestamp <= current_timestamp()) ON VIOLATION ALLOW

```


* **`ON VIOLATION DROP` (Filter & Discard):** Atomically drops rows that fail the boolean predicate before they are committed to destination Delta storage, preventing downstream dataset corruption.
```sql
CONSTRAINT positive_price EXPECT (unit_price > 0) ON VIOLATION DROP

```


* **`ON VIOLATION FAIL` (Abort Transaction):** Halts pipeline execution immediately and returns a critical execution exception if any row violates the defined business rule.
```sql
CONSTRAINT valid_id EXPECT (order_id IS NOT NULL) ON VIOLATION FAIL

```



### 2. Change Data Capture via `APPLY CHANGES INTO`

The declarative `APPLY CHANGES INTO` abstraction eliminates manual `MERGE INTO` tuning, automatically managing out-of-order records, late-arriving events, and schema synchronization:

* **SCD Type 1 (Deterministic Overwrite):** Maintains the current record state by overwriting existing target rows matching the specified `KEYS` using `SEQUENCE BY` column ordering.
```sql
APPLY CHANGES INTO live.dim_customers
FROM stream(live.stage_customers)
KEYS (customer_id)
APPLY AS DELETE WHEN operation = 'DELETE'
SEQUENCE BY event_timestamp
COLUMNS * EXCEPT (operation);

```


* **SCD Type 2 (Historical Dimension Tracking):** Preserves historical state intervals across changes, automatically generating metadata tracking columns (`__START_AT`, `__END_AT`) and populating `__END_AT = NULL` for the active record version.
```python
import dlt

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

* **Streaming Tables (`STREAMING TABLE`):** Backed by the Spark Structured Streaming engine and dedicated state checkpoint directories. These process append-only, high-throughput source streams incrementally, ensuring each record or raw storage object is evaluated exactly once.
* **Materialized Views (`MATERIALIZED VIEW`):** Maintain pre-computed query states over upstream transactional tables or historical data stores. The incremental computation engine processes only the net delta changes from source tables during each refresh cycle, reducing latency and query compute costs for downstream BI consumers.

---

## Important Exam Considerations

* **Polyglot Pipeline Homogeneity Rule:** While a single Lakeflow Declarative Pipeline DAG can coordinate tasks across both SQL and Python source files, individual source files must be homogeneous. Embedding Python cells within a SQL notebook (or `%sql` within Python modules) inside a declarative pipeline triggers a compilation failure.
* **Target Catalog Routing:** Pipelines governed by Unity Catalog support direct output routing to explicit 3-tier namespaces (`catalog.schema.table`), allowing developers to deploy Bronze, Silver, and Gold objects to separate physical storage containers and access tiers from a single pipeline definition.
* **DLT Event Log Telemetry:** Platform metrics, runtime graph dependencies, cluster configurations, and data quality pass/drop metrics write continuously to an internal Delta table (the DLT Event Log). This log can be queried directly via Spark SQL to audit historical pipeline run durations, expectation drop ratios, and failure root causes.

---

[← Back to Section 14: Lakeflow Declarative Pipelines Overview](https://www.google.com/search?q=./section14-readme.md) | [Next Section: Section 16: Lakeflow Jobs & Workflow Orchestration →](https://www.google.com/search?q=./section16-readme.md)
](https://www.faymyers.com/)
