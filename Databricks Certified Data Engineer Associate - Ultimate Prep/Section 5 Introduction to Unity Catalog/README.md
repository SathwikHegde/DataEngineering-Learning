# Section 15: Lakeflow Spark Declarative Pipelines (SDP) — Project

This module provides a comprehensive implementation specification for Lakeflow Spark Declarative Pipelines (SDP), formerly Delta Live Tables (DLT). Spanning 160 minutes, the curriculum focuses on architecting an enterprise-grade Medallion topology, progressing from multi-protocol ingestion to conformed Gold presentation layers while enforcing row-level data quality invariants, Change Data Capture (CDC) state synchronization, and Unity Catalog multi-namespace governance.

Refer to `image_08e047.png` for the topological dependency graph and execution sequencing.

---

## Section Overview

* **Total Duration:** 2 Hours 40 Minutes
* **Total Modules:** 14
* **Primary Focus:** Architecting a polyglot Medallion Directed Acyclic Graph (DAG) utilizing declarative SQL and Python APIs, implementing deterministic constraint enforcement, automating Slowly Changing Dimension (SCD Type 1 and Type 2) materialization via `APPLY CHANGES INTO`, and orchestrating cross-catalog schema deployment.

---

## Project Implementation Breakdown

| Module | Title | Duration | Language | Technical Scope |
| --- | --- | --- | --- | --- |
| **104** | **Project Overview** | 2 min | — | System topology, schema contracts, and downstream latency SLAs. |
| **105** | **Cluster Configuration & Azure VM Quota** | 12 min | — | Sizing dedicated SDP compute runtimes, worker core allocations, and Azure VM quota constraints. |
| **106** | **Project Environment Set-up** | 14 min | — | Cloud storage staging, seeding source payloads, and mounting stateful checkpoint paths. |
| **107** | **Intro to Streaming Tables** | 10 min | SQL | Declarative append-only ingestion utilizing `CREATE OR REFRESH STREAMING TABLE`. |
| **108** | **Recent Changes to the UI** | 2 min | — | Pipeline DAG visualization, real-time compute telemetry, and event log interrogation. |
| **109** | **Create CircuitBox Pipeline** | 25 min | — | Pipeline compilation, target schema binding, and topological DAG dependency resolution. |
| **110** | **Intro to SDP Expectations** | 19 min | SQL | Declarative data quality constraints via SQL (`ALLOW`, `DROP`, `FAIL`). |
| **111** | **Intro to Apply Changes** | 17 min | SQL | Declarative CDC ingestion and state reconciliation for SCD Type 1 architectures. |
| **112** | **Creating SDP Datasets** | 12 min | Python | Multi-language DAG extension utilizing `@dlt.table` and `@dlt.view` decorators. |
| **113** | **Implementing SDP Expectations** | 11 min | Python | Programmatic data validation rules utilizing `@dlt.expect_*` decorators. |
| **114** | **Implementing Slowly Changing Dimensions** | 10 min | Python | Historical lineage tracking via `dlt.apply_changes()` targeting SCD Type 2 topologies. |
| **115** | **Process Orders Data — Assignment** | 10 min | Polyglot | Independent execution of relational Silver-tier order normalization DAGs. |
| **116** | **Intro to Materialized Views** | 16 min | Polyglot | Incremental computation of aggregated Gold consumption layers utilizing Materialized Views. |
| **117** | **Publish to Multiple Catalogs/Schemas** | 1 min | — | Multi-namespace deployment across development, staging, and production Unity Catalog environments. |

---

## Core Architectural Implementation Concepts

### 1. Declarative Data Quality Enforcement (Expectations)

Expectations integrate deterministic data quality validation natively into the execution DAG without mutating the underlying Spark Structured Streaming logical plan. Constraints evaluate row-by-row against three operational routing heuristics:

* **`ON VIOLATION ALLOW` (Audit & Retain):** Logs constraint violations transparently to the event telemetry log while permitting anomalous records to propagate downstream for auditing.

```sql
CONSTRAINT valid_timestamp EXPECT (event_timestamp <= current_timestamp()) ON VIOLATION ALLOW

```

* **`ON VIOLATION DROP` (Filter & Discard):** Atomically drops anomalous records prior to target Delta storage materialization, preventing downstream schema contamination.

```sql
CONSTRAINT positive_price EXPECT (unit_price > 0) ON VIOLATION DROP

```

* **`ON VIOLATION FAIL` (Abort Transaction):** Instantly aborts the active pipeline DAG upon anomaly detection, throwing a fatal runtime exception.

```sql
CONSTRAINT valid_id EXPECT (order_id IS NOT NULL) ON VIOLATION FAIL

```

### 2. Change Data Capture via `APPLY CHANGES INTO`

The declarative `APPLY CHANGES INTO` protocol abstracts manual `MERGE INTO` tuning, natively resolving out-of-order records, late-arriving events, and target schema synchronization:

* **SCD Type 1 (Deterministic Overwrite):** Maintains current state representations by overwriting target rows mapping to specified `KEYS`, sequenced by a temporal column vector.

```sql
APPLY CHANGES INTO live.dim_customers
FROM stream(live.stage_customers)
KEYS (customer_id)
APPLY AS DELETE WHEN operation = 'DELETE'
SEQUENCE BY event_timestamp
COLUMNS * EXCEPT (operation);

```

* **SCD Type 2 (Historical Dimension Tracking):** Preserves temporal state intervals across mutations, autonomously generating metadata tracking boundaries (`__START_AT`, `__END_AT`) and terminating the active record version with `__END_AT = NULL`.

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

* **Streaming Tables (`STREAMING TABLE`):** Governed by the Spark Structured Streaming engine and dedicated state checkpoint metadata. Engineered for append-only, high-throughput ingestion, ensuring exactly-once processing semantics for sequential micro-batches.
* **Materialized Views (`MATERIALIZED VIEW`):** Persist pre-computed query states atop upstream transactional entities. The incremental computation engine evaluates exclusively net delta modifications during refresh cycles, minimizing DAG execution latency and compute overhead for downstream BI consumers.

---

## Important Exam Considerations

* **Polyglot Pipeline Homogeneity Rule:** While Lakeflow Declarative Pipeline DAGs support discrete SQL and Python source files within a unified graph, individual file assets must remain strictly homogeneous. Embedding Python directives within a SQL notebook (or vice versa) triggers fatal compilation exceptions.
* **Unity Catalog Target Routing:** Unity Catalog-governed pipelines natively support explicit 3-tier namespace routing (`catalog.schema.table`). This permits the deployment of Bronze, Silver, and Gold assets to isolated physical storage containers and access tiers from a unified pipeline manifest.
* **Event Log Telemetry:** Platform metrics, Catalyst DAG dependencies, compute auto-scaling events, and data quality routing statistics write continuously to the internal DLT Event Log. This Delta table can be queried directly via Spark SQL to audit historical pipeline latency, expectation drop rates, and failure exception traces.

---

[← Back to Section 14: Lakeflow Declarative Pipelines Overview](https://www.google.com/search?q=./section14-readme.md) | [Next Section: Section 16: Lakeflow Jobs & Workflow Orchestration →](https://www.google.com/search?q=./section16-readme.md)
