
Section 15: Lakeflow Spark Declarative Pipelines (SDP) — Project

This module provides a detailed implementation guide for Lakeflow Spark Declarative Pipelines (SDP), formally known as Delta Live Tables (DLT). This 160-minute curriculum focuses on building an enterprise-grade Medallion architecture, moving from multi-protocol ingestion to conformed Gold presentation layers while applying row-level data quality rules, Change Data Capture (CDC) syncing, and Unity Catalog data governance.
Refer to the execution sequencing and topological dependencies in image_08e047.png.

Section Overview

• Total Duration: 2 Hours 40 Minutes
• Total Modules: 14
• Primary Focus: Building polyglot Medallion Directed Acyclic Graphs (DAGs) using declarative SQL and Python APIs, enforcing data validation rules, automating Slowly Changing Dimension (SCD Type 1 and Type 2) tracking via APPLY CHANGES INTO, and orchestrating multi-catalog schema deployments.

Project Implementation Breakdown


Module	Title	Duration	Language	Technical Scope
104	Project Overview	2 min	—	System topology, schema contracts, and downstream latency SLAs.
105	Cluster Configuration & Azure VM Quota	12 min	—	Sizing dedicated SDP compute runtimes, worker core allocations, and Azure VM quota limits.
106	Project Environment Set-up	14 min	—	Cloud storage staging, seeding source payloads, and mounting stateful checkpoint paths.
107	Intro to Streaming Tables	10 min	SQL	Declarative append-only ingestion using CREATE OR REFRESH STREAMING TABLE.
108	Recent Changes to the UI	2 min	—	Pipeline DAG visualization, real-time compute telemetry, and event log auditing.
109	Create CircuitBox Pipeline	25 min	—	Pipeline compilation, target schema binding, and topological DAG dependency resolution.
110	Intro to SDP Expectations	19 min	SQL	Declarative data quality constraints via SQL (ALLOW, DROP, FAIL).
111	Intro to Apply Changes	17 min	SQL	Declarative CDC ingestion and state reconciliation for SCD Type 1 architectures.
112	Creating SDP Datasets	12 min	Python	Multi-language DAG extensions using the @dlt.table and @dlt.view decorators.
113	Implementing SDP Expectations	11 min	Python	Programmatic data validation rules using @dlt.expect_* decorators.
114	Implementing Slowly Changing Dimensions	10 min	Python	Historical lineage tracking via dlt.apply_changes() targeting SCD Type 2 topologies.
115	Process Orders Data — Assignment	10 min	Polyglot	Independent execution of relational Silver-tier order normalization DAGs.
116	Intro to Materialized Views	16 min	Polyglot	Incremental computation of aggregated Gold consumption layers using Materialized Views.
117	Publish to Multiple Catalogs/Schemas	1 min	—	Multi-namespace deployments across development, staging, and production Unity Catalog environments.


Core Architectural Implementation Concepts


1. Declarative Data Quality Enforcement (Expectations)

Expectations integrate data quality validation checks directly into the execution DAG without modifying the underlying Spark Structured Streaming logical plan. Rules evaluate row-by-row against three distinct behaviors:
• ON VIOLATION ALLOW (Audit & Retain): Records constraint violations inside the event telemetry log but allows the invalid rows to pass downstream for further audit.
sql
CONSTRAINT valid_timestamp EXPECT (event_timestamp <= current_timestamp()) ON VIOLATION ALLOW
Use code with caution.
• ON VIOLATION DROP (Filter & Discard): Drops invalid records before they are saved to the target Delta table, preventing downstream pollution.
sql
CONSTRAINT positive_price EXPECT (unit_price > 0) ON VIOLATION DROP
Use code with caution.
• ON VIOLATION FAIL (Abort Transaction): Halts the pipeline DAG immediately upon finding an invalid record, throwing a fatal runtime exception.
sql
CONSTRAINT valid_id EXPECT (order_id IS NOT NULL) ON VIOLATION FAIL
Use code with caution.

2. Change Data Capture via APPLY CHANGES INTO

The declarative APPLY CHANGES INTO engine simplifies manual MERGE INTO queries by automatically handling out-of-order records, late-arriving events, and schema synchronization:
• SCD Type 1 (Latest State Overwrite): Retains only the current state by overwriting destination rows matching specified KEYS, sorted sequentially by a time vector column.
sql
APPLY CHANGES INTO live.dim_customers
FROM stream(live.stage_customers)
KEYS (customer_id)
APPLY AS DELETE WHEN operation = 'DELETE'
SEQUENCE BY event_timestamp
COLUMNS * EXCEPT (operation);
Use code with caution.
• SCD Type 2 (Historical Tracking): Retains a history of changes across records over time. It automatically tracks historical validity windows using __START_AT and __END_AT columns, closing the active current version with __END_AT = NULL.
python
import dlt

dlt.apply_changes(
    target="dim_customers_scd2",
    source="stage_customers",
    keys=["customer_id"],
    sequence_by="event_timestamp",
    apply_as_deletes="operation = 'DELETE'",
    stored_as_scd_type="2",
)
Use code with caution.

3. Streaming Tables vs. Materialized Views

• Streaming Tables (STREAMING TABLE): Managed by Spark Structured Streaming and structured state checkpointing. Built for append-only, high-volume data ingestion to guarantee exactly-once processing across incoming sequential micro-batches.
• Materialized Views (MATERIALIZED VIEW): Cache pre-computed results on top of upstream transactional sources. The incremental engine processes only net-new changes during table refreshes, cutting down pipeline latency and compute overhead for data analytics and BI applications.

Important Exam Considerations

• Polyglot Pipeline Homogeneity Rule: Even though an SDP pipeline DAG can mix separate SQL and Python source files into one unified graph, individual file assets must stay entirely homogeneous. Trying to run Python commands within a SQL notebook (or vice versa) results in a compilation failure.
• Unity Catalog Target Routing: Pipelines governed by Unity Catalog fully support explicit 3-tier namespace routing (catalog.schema.table). This allows developers to publish Bronze, Silver, and Gold data assets to isolated physical storage layers and separate access groups using a single pipeline configurations file.
• Event Log Telemetry: Engine operational metrics, Catalyst DAG dependencies, cluster scaling details, and data validation logs are continuously written to an internal DLT Event Log. This Delta table can be queried directly using Spark SQL to inspect historical latency patterns, expectation drop rates, and error traces.
← Back to Section 14: Lakeflow Declarative Pipelines Overview | Next Section: Section 16: Lakeflow Jobs & Workflow Orchestration →
Would you like me to construct an interactive practice test on these three expectation policies and SCD behaviors, or should we deep-dive into how Streaming Tables maintain their checkpoints?
