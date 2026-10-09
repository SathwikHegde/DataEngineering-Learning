## Section 12: Data Ingestion with Lakeflow Connect (2026 Exam Update)
This repository contains study notes, architecture diagrams, and pipeline configurations for Section 12: Data Ingestion with Lakeflow Connect. This section reflects the updated certification requirements, focusing on the transition from traditional, boilerplate ingestion code to Databricks' native, platform-integrated Lakeflow Connect framework alongside cloud-native Auto Loader workflows.
------------------------------
## Section Overview

* Total Duration: 1 Hour 12 Minutes
* Total Modules: 8
* Primary Focus: Native SaaS/Database connectors, Change Data Capture (CDC), and streaming ingestion from cloud files and event buses.

------------------------------
## Curriculum Breakdown

| Module | Topic | Duration | Core Concept / Delivery |
|---|---|---|---|
| 80 | Data Ingestion Overview | 7 min | Evolutionary shift from manual ETL plumbing to platform-native, serverless ingestion. |
| 81 | Introduction to Auto Loader | 13 min | Utilizing cloudFiles for automated, incremental ingestion from object storage. |
| 82 | Auto Loader - File Options | 9 min | Configuring directory listing vs. file notification modes, paths, and glob patterns. |
| 83 | Auto Loader - Schema Evolution | 12 min | Working with schemaEvolutionMode and handling unexpected variations using the Rescued Data Column. |
| 84 | Streaming Ingestion with Kafka & Event Hubs | 5 min | Capturing sub-second event streams natively using Spark Structured Streaming connectors. |
| 85 | Lakeflow Connect - SaaS Connector Architecture | 4 min | Understanding the fully managed SaaS gateway framework (Salesforce, Workday, Jira, etc.). |
| 86 | Lakeflow Connect - SaaS Connector Demo | 16 min | Step-by-step point-and-click implementation of a live SaaS data replication pipeline. |
| 87 | Lakeflow Connect - Database Connector Architecture | 7 min | Deconstructing incremental database ingestion via Change Data Capture (CDC) and Change Tracking (CT). |

------------------------------
## Key Technical Architecture Updates## 1. Auto Loader (cloudFiles)
Auto Loader remains the standard mechanism to incrementally ingest files landing in cloud object storage (AWS S3, Azure ADLS, GCP GCS) without manual state tracking.

* Directory Listing vs. File Notification: Select Directory Listing for standard workloads. Transition to File Notification (utilizing AWS SNS/SQS or Azure Event Grid) for high-volume buckets containing millions of files to eliminate costly storage API scanning operations.
* The Rescued Data Column: Automatically isolates malformed data or unexpected schema mismatches, preventing production pipeline failures caused by upstream schema mutations.

# Standard 2026 Auto Loader PySpark Ingestion Patterndf_bronze = (spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", "dbfs:/mnt/telecom/schemas/customers")
    .option("cloudFiles.schemaEvolutionMode", "addNewColumns")
    .load("abfss://raw-data@storageaccount.dfs.core.windows.net/customers/"))

## 2. Lakeflow Connect (New 2026 Ecosystem Core)
Lakeflow Connect shifts data engineering workloads from imperative code implementations to declarative configurations, replacing custom ingestion scripts with platform-native, serverless replication.

* Serverless Execution: Operates entirely on serverless compute infrastructure. It eliminates cluster sizing and infrastructure management by utilizing automated scaling and native fault recovery mechanisms.
* Unified Governance: All target tables instantiated via Lakeflow Connect automatically register with a 3-tier namespace within Unity Catalog, enforcing end-to-end data lineage, audit logging, and granular access controls.
* Database CDC & Change Tracking: Native database connectors (including Microsoft SQL Server, PostgreSQL, and MySQL) bind directly to Change Data Capture (CDC) or Change Tracking (CT) frameworks to stream transactional mutations (inserts, updates, and deletes) into the Lakehouse.
* SaaS Ingestion: Managed abstraction layers handle complex authentication routines and API rate-limiting thresholds for enterprise operational systems (Salesforce Sales Cloud, Workday, ServiceNow, Jira, Confluence).

------------------------------
## Important Exam Considerations

* The 2026 Ingestion Allowance: Active workspaces include a complimentary allocation of 100 DBUs per day specifically designated for managed SaaS and database connectors via the Lakeflow Connect Free Tier. This allows for the cost-effective onboarding of up to 100 million records daily without incurring volume-based premiums.
* Schema Evolution Settings: For examination purposes, memorize the supported modes: addNewColumns (default), failOnNewColumns, rescue, and none.
* Kafka Integration: Streaming ingestion from Kafka or Azure Event Hubs relies on tracking unique offsets. Distinct checkpoint locations must be configured to maintain exactly-once delivery guarantees.

If you want to prepare further for the exam, I can:

* Provide a practice quiz covering these 2026 ingestion concepts
* Detailed code examples for Kafka / Event Hubs streaming setups
* A deep dive into the Rescued Data Column mechanics

Let me know what topic you want to focus on next.

