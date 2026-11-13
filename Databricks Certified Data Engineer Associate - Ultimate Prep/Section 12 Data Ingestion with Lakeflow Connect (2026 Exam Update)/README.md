
Section 12: Data Ingestion with Lakeflow Connect (2026 Technical Guide)

This guide outlines the architecture, ingestion workflows, and framework updates for Section 12: Data Ingestion with Lakeflow Connect. The curriculum covers how to modernize data pipelines by moving away from custom, code-heavy ETL scripts and adopting declarative ingestion tools powered by Databricks Lakeflow Connect and Auto Loader (cloudFiles).

Curriculum Overview

• Total Duration: 1 Hour 12 Minutes
• Total Modules: 8
• Core Focus Areas: Managed SaaS and database connectors, low-latency Change Data Capture (CDC), and fault-tolerant event streaming.

Module	Topic	Duration	Key Learning Goal
80	Data Ingestion Evolution	7 min	Move from disconnected, custom ETL tools to a unified, serverless ingestion platform.
81	Auto Loader Core Mechanics	13 min	Use cloudFiles for fast, incremental processing of files in cloud storage.
82	Storage Discovery Optimizations	9 min	Compare standard directory listing against event-driven file notifications (SNS/SQS, Event Grid).
83	Schema Evolution & Resolution	12 min	Manage schema changes reliably using schemaEvolutionMode settings and rescued data columns.
84	Pub/Sub Message Bus Ingestion	5 min	Process high-speed, sub-second data streams from Apache Kafka and Azure Event Hubs.
85	Lakeflow Connect: SaaS Gateway	4 min	Use managed connectors that automatically handle API rate limits, token refreshes, and pagination.
86	Lakeflow Connect: SaaS Integration	16 min	Set up, launch, and monitor live data pipelines from managed SaaS applications.
87	Lakeflow Connect: RDBMS CDC	7 min	Read database transaction logs with minimal overhead to keep data consistent at scale.


Core Architecture & Design Patterns


1. High-Scale File Ingestion: Auto Loader (cloudFiles)

Auto Loader acts as a lightweight virtual file system that discovers and processes new unstructured or semi-structured files from cloud storage (AWS S3, Azure ADLS Gen2, GCP GCS) as they arrive.
• File Discovery Methods: Use Directory Listing for small-to-medium file counts. For massive buckets containing millions of files, switch to File Notification mode. This connects directly to cloud messaging services (like AWS SQS or Azure Event Grid) to avoid slow, expensive storage API scans.
• Schema Protection: The _rescued_data column automatically catches mismatched data types, unexpected fields, or corrupt records. This isolates bad data without crashing your pipeline.
Production-Ready 2026 Auto Loader PySpark Example
python
df_bronze = (spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", "abfss://metadata@storage.dfs.core.windows.net/schemas/customers")
    .option("cloudFiles.schemaEvolutionMode", "addNewColumns")
    .load("abfss://raw-data@storage.dfs.core.windows.net/customers/"))
Use code with caution.

2. Enterprise Ingestion Plane: Lakeflow Connect

Lakeflow Connect replaces custom replication code with fully managed, declarative connectors that run on Serverless Compute.
• Serverless Execution: Removes the need to size clusters, tune JVMs, or manage infrastructure. It provides automatic scaling, state tracking, and built-in failure recovery.
• Built-In Governance (Unity Catalog): Destination tables are organized automatically using a 3-tier namespace (catalog.schema.table), giving you end-to-end data lineage, audit logs, and fine-grained access controls.
• Database Change Data Capture (CDC): Native connectors for SQL Server, PostgreSQL, and MySQL read transaction logs (WAL/Binlog) or Change Tracking tables directly, streaming reliable updates straight into Delta Lake.
• SaaS Connectors: Automatically manages authentication (OAuth, JWT), pagination, and API rate limits for business apps like Salesforce, Workday, ServiceNow, and Jira.

Exam Checklist & Key Edge Cases

• Free Tier Limits: Workspaces include 100 DBUs per day under the Lakeflow Connect Free Tier. This covers up to 100 million daily row changes at no extra platform cost.
• Schema Evolution Modes: You must know how Auto Loader reacts to schema changes under each of the four supported modes:
	• addNewColumns: Automatically adds newly discovered columns to the target schema.
	• failOnNewColumns: Stops the pipeline immediately when an unexpected column appears.
	• rescue: Keeps the existing schema unchanged and routes new or mismatched fields into the _rescued_data column.
	• none: Ignores schema changes entirely and drops any unrecognized fields.
• Exactly-Once Processing & Offsets: High-speed streams from Kafka or Event Hubs track their reading progress using offsets. You must configure a unique checkpointLocation for every stream to guarantee exactly-once processing across restarts.
Which exam focus area would you like to dive into next?
Code examples for setting up Kafka/Event Hubs exactly-once checkpointsA side-by-side comparison of Directory Listing vs. File Notification performancePractice exam questions testing schemaEvolutionMode error-handling scenarios
