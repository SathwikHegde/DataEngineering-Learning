
Section 14: Lakeflow Declarative Pipelines (DLT) — Overview

This section introduces Delta Live Tables (DLT) as the declarative execution engine in the Databricks Lakeflow platform. It covers the shift from traditional, code-heavy pipeline scripts—where engineers manually manage Structured Streaming offsets, checkpoints, and retry logic—to a declarative model. Instead of writing step-by-step execution code, you define your target tables and data quality rules, letting Databricks automatically scale compute, build the dependency graph (DAG), and track data lineage.
Check image_628c04.png for the dependency graph and lesson order.

Section Overview

• Total Duration: 20 minutes
• Total Modules: 3
• Core Focus: Declarative ETL design, automatic DAG generation, stateful streaming, and writing DLT pipelines in SQL and Python.

Curriculum Breakdown


Module #	Title	Duration	Key Learning Goal
100	Introduction to Delta Live Tables	8 min	Compare the cost efficiency and operational benefits of declarative pipelines against traditional Spark Structured Streaming jobs.
101	DLT Architecture	4 min	Understand how the Lakeflow compiler parses pipeline code to build visual DAGs and auto-scale compute resources.
102	Programming with DLT	9 min	Build basic data pipelines using ANSI SQL statements and PySpark DLT decorators.


Core Architectural Concepts


1. Declarative vs. Imperative Execution

Traditional pipelines require you to manually configure streaming readers, triggers, and checkpoint folders (such as .readStream, .writeStream, and .option("checkpointLocation", path)). Lakeflow Declarative Pipelines remove this boilerplate. The compiler reads your pipeline files, detects table dependencies automatically, and builds an optimized execution graph (DAG).

2. Multi-Language Support (SQL & Python)

DLT supports both declarative SQL and Python. You can combine SQL and Python notebooks within the same pipeline, though each individual file must stick to a single language.
• SQL Example:
sql
-- Create a streaming table using Auto Loader natively in SQL
CREATE OR REFRESH STREAMING LIVE TABLE bronze_customers
AS SELECT * FROM cloud_files("abfss://raw-zone@storageaccount.dfs.core.windows.net/customers", "json");
Use code with caution.
• Python Example:
python
import dlt

# Use the @dlt.table decorator to define a managed pipeline table
@dlt.table(name="bronze_customers")
def bronze_customers():
    return (spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "json")
        .load("abfss://raw-zone@storageaccount.dfs.core.windows.net/customers"))
Use code with caution.

3. Table Types & State Behavior

• Streaming Live Tables (STREAMING LIVE TABLE): Built for incremental, append-only ingestion. DLT tracks progress in the transaction log and processes only new data that arrived since the last pipeline run.
• Materialized Views / Live Tables (LIVE TABLE): Batch-style tables that recompute full aggregations, joins, and business logic whenever the pipeline runs.

Key Exam Takeaways

• Cannot Run Interactively in Notebooks: Exam questions frequently test how DLT code runs. You cannot run DLT cells interactively one-by-one on a standard notebook cluster—doing so raises an error. DLT code must be attached to and executed through a Lakeflow Pipeline deployment.
• Data Quality Rules (Expectations): DLT enforces data quality rules directly during execution using three violation actions:
	• ON VIOLATION ALLOW: Logs bad records to the pipeline event log while still allowing them to pass into the target table.
	• ON VIOLATION DROP: Drops invalid records silently before writing data to the target table.
	• ON VIOLATION FAIL: Immediately stops the entire pipeline run when invalid data is found so bad records never reach downstream tables.
• Automatic Column-Level Lineage: When you reference upstream tables using the LIVE.<table_name> syntax, the Lakeflow compiler automatically builds and displays end-to-end column-level lineage without extra logging code.
← Back to Section 13: Delta Lake Architecture & Internal Mechanics | Next Section: Section 15: Lakeflow Spark Declarative Pipelines Project →
Would you like to adjust the tone (e.g., even shorter or more technical), or paste another section README to rewrite?
