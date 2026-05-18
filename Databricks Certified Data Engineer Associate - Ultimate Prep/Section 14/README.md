# Section 14: Lakeflow Declarative Pipelines (DLT) - Overview

This section covers **Delta Live Tables (DLT)** within the unified **Lakeflow** framework. DLT completely changes how ETL pipelines are built by shifting from imperative coding (where you manually manage Spark streams, checkpoints, and retries) to a **declarative framework**. You simply define the data transformations and quality bounds, and the execution engine automatically orchestrates the infrastructure, manages dependencies, and tracks lineage.

Refer to **image_628c04.png** for the structure of this introductory module.

---

## Section Overview

* **Total Duration:** 20 minutes
* **Total Lessons:** 3
* **Primary Focus:** Declarative ETL foundations, pipeline graph execution, and basic DLT syntax patterns.

---

## Curriculum Breakdown

| Lesson # | Title | Duration | Core Learning Outcome |
| --- | --- | --- | --- |
| **100** | **Introduction to Delta Live Tables** | 8 min | Understanding the value proposition of declarative engineering over manual Spark streaming pipelines. |
| **101** | **DLT Architecture** | 4 min | How Databricks processes the Directed Acyclic Graph (DAG) and handles automatic scaling. |
| **102** | **Programming with DLT** | 9 min | Introductory code structures using both SQL and PySpark syntax to declare tables and views. |

---

## Core Architectural Concepts

### 1. Declarative vs. Imperative Execution

In a classic PySpark streaming job, you write explicit handlers for `.readStream`, `.writeStream`, `.checkpointLocation`, and micro-batch triggers.

With **Lakeflow Declarative Pipelines (DLT)**, you only describe the target states. The underlying engine looks at your complete code package, discovers the relationships between datasets, and constructs an execution graph (DAG) automatically.

### 2. DLT Programming Syntax

DLT supports both SQL and Python natively. A single pipeline can contain source code files from both languages.

* **SQL Pattern:**
```sql
CREATE OR REFRESH STREAMING LIVE TABLE bronze_customers
AS SELECT * FROM cloud_files("/mnt/raw/customers", "json");

```


* **Python Pattern:**
```python
import dlt

@dlt.table(name="bronze_customers")
def bronze_customers():
    return spark.readStream.format("cloudFiles").option("cloudFiles.format", "json").load("/mnt/raw/customers")

```



### 3. Object Classifications inside DLT

* **Streaming Live Tables:** Statefully track incremental data ingestion. They only process new files or records since the last refresh.
* **Live Tables / Views:** Standard batch tables computed from scratch across the current state of upstream data during each pipeline run.

---

## Important Exam Considerations

* **No Interactive Execution:** You cannot execute DLT code cell-by-cell in a standard Databricks notebook. The code files must be attached to a **Pipeline Deployment** inside the Workflows tab.
* **Data Quality Expectations:** While covered deeper in upcoming sections, remember that DLT uses *Expectations* (`CONSTRAINT... ON VIOLATION`) to gracefully handle dirty data via `EXPECT`, `EXPECT OR DROP`, or `EXPECT OR FAIL`.
* **Lineage Tracking:** Because the engine controls the compilation, end-to-end dataset lineage is automatically discovered and rendered visually in the user interface.