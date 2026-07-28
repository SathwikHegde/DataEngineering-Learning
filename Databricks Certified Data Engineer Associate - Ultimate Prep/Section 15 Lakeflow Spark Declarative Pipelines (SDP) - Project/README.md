# Section 15: Lakeflow Spark Declarative Pipelines (SDP) — Project

This section delivers a comprehensive, hands-on architectural project using **Lakeflow Spark Declarative Pipelines (SDP)** (traditionally known as Delta Live Tables). Over the course of 2 hours and 40 minutes, you will build an end-to-end production application following the Medallion architecture—moving seamlessly from raw data ingestion to consumption-ready analytical layers while integrating advanced data validation, governance, and Slowly Changing Dimensions (SCD).

Refer to `image_08e047.png` for the complete sequence of project tasks.

---

## Section Overview

* **Total Duration:** 2 Hours 40 Minutes
* **Total Modules:** 14
* **Core Goal:** Implement a multi-hop Medallion pipeline using modern declarative SQL and Python syntax, validating data structures with Expectations, executing CDC synchronization, and managing multi-catalog deployment.

---

## Project Implementation Breakdown

| Module | Title | Duration | Language | Focus Area |
| --- | --- | --- | --- | --- |
| **104** | **Project Overview** | 2 min | — | System topology and business goals of the CircuitBox case study. |
| **105** | **Cluster Configuration & Azure VM Quota** | 12 min | — | Provisioning specialized SDP cluster sizes within Azure limits. |
| **106** | **Project Environment Set-up** | 14 min | — | **[Resource]** Ingesting base datasets and seeding cloud target storage paths. |
| **107** | **Intro to Streaming Tables** | 10 min | SQL | Appending incremental data inputs efficiently via `STREAMING LIVE TABLE`. |
| **108** | **Recent Changes to the UI** | 2 min | — | Essential walkthrough of the updated pipeline monitoring interface. |
| **109** | **Create CircuitBox Pipeline** | 25 min | — | Initial build, compilation, and dependency graphing of the core pipeline. |
| **110** | **Intro to SDP Expectations** | 19 min | SQL | Declaring business constraints to enforce schema and quality validation. |
| **111** | **Intro to Apply Changes** | 17 min | SQL | Tracking master dataset state shifts via **SCD Type 1** topology. |
| **112** | **Creating SDP Datasets** | 12 min | Python | Multi-language orchestration using PySpark-driven ingestion modules. |
| **113** | **Implementing SDP Expectations** | 11 min | Python | Writing programmatic data quality validation policies in Python. |
| **114** | **Implementing Slowly Changing Dimensions** | 10 min | Python | Maintaining historic track states using **SCD Type 2** architecture. |
| **115** | **Process Orders Data - Assignment** | 10 min | Mix | Capstone milestone to individually construct the order enrichment stage. |
| **116** | **Intro to Materialized Views** | 16 min | Mix | Building low-latency aggregated layers for business reporting layers. |
| **117** | **Publish to Multiple Catalogs/Schemas** | 1 min | — | Enforcing cross-catalog deployment governance from a centralized pipeline. |

---

## Core Architectural Implementation Methods

### 1. Data Quality Guardrails (Expectations)

Instead of letting dirty data crash execution mid-transit, SDP introduces declarative rules to handle malformed information inline. You will implement three distinct validation layers across both SQL and Python:

* **`ON VIOLATION ALLOW`**: Records the failure in telemetry logging but permits the record to pass into the target table.
* **`ON VIOLATION DROP`**: Silently drops the non-compliant rows from the stream before they reach destination storage.
* **`ON VIOLATION FAIL`**: Immediately halts pipeline execution if a critical data anomaly occurs.

### 2. Change Data Capture via `APPLY CHANGES`

Managing data volatility is simplified using the native `APPLY CHANGES INTO` syntax, abstracting away complex, manual merge operations:

* **SCD Type 1 (Module 111)**: Overwrites existing target values directly when changes are detected. Perfect for keeping current, normalized states (e.g., Customer Master).
* **SCD Type 2 (Module 114)**: Tracks historical changes by automatically managing flags (`__start_at`, `__end_at`, `__is_current`) to capture every chronological iteration of a record (e.g., Address modifications).

### 3. Streaming Tables vs. Materialized Views

* **Streaming Tables (`STREAMING LIVE TABLE`)**: Optimized for append-only, high-velocity data. They leverage stateful checkpointing to ensure files or messages are evaluated exactly once.
* **Materialized Views (`LIVE TABLE`)**: Periodically pre-compute complicated aggregates and analytical queries based on up-to-date source changes. This drastically cuts queries down to milliseconds for downstream BI tools.

---

## Important Exam Considerations

* **Mixing Languages But Not Files**: While you can mix SQL and Python files inside a single pipeline graph to build a unified DAG, you **cannot** mix languages within the exact same notebook file. A single file must be 100% SQL or 100% Python.
* **Target Catalog Specifications**: Review the mechanics of target catalog specification. In modern workspaces, you can publish output components to distinct schemas and external environments directly from your declarative definitions under Unity Catalog governance.
* **The Event Log Dataset**: Familiarize yourself with querying the automated DLT Event Log. It stores JSON telemetry regarding execution graph steps, cluster initialization details, and exact expectation pass/fail/drop metrics.

---

[← Back to Section 14: Lakeflow Declarative Pipelines Overview](https://www.google.com/search?q=./section14-readme.md) | [Next Section: Section 16: Lakeflow Jobs →](https://www.google.com/search?q=./section16-readme.md)
