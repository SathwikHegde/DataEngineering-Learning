# Section 6: Apache Spark – Overview

This section transitions from theoretical foundations to the **technical implementation** of data pipelines. It focuses on configuring the modern "Lakehouse" architecture, where the speed of Apache Spark meets the governance of Unity Catalog and the scalability of a cloud Data Lake.

---

## Module Breakdown

### **31. ETL With Apache Spark – Overview** (3 min)

A high-level introduction to using Spark as a distributed processing engine. This unit establishes why Spark is the industry standard for ETL:

* **Extract:** Connecting to diverse sources (CSV, Parquet, Delta, APIs).
* **Transform:** Leveraging the **DataFrame API** for filtering, joining, and aggregating at scale.
* **Load:** Writing optimized data back to storage for downstream consumption.

### **32. ETL Project Overview** (5 min)

This unit sets the roadmap for the hands-on project. You will move through the "Medallion Architecture" lifecycle:

1. **Bronze (Raw):** Ingesting source files exactly as they are.
2. **Silver (Cleaned):** De-duplicating and cleaning the data.
3. **Gold (Business Ready):** Aggregating data for high-level reporting.

### **33. Set-up Data Lake Project Environment** (11 min)

A technical deep-dive into cloud storage connectivity. You will learn to:

* Configure cloud containers (e.g., AWS S3 or Azure ADLS Gen2).
* Manage **Service Principals** or IAM roles for secure, non-interactive access.
* Configure Spark session properties to interact directly with the storage layer.

### **34. Set-up Unity Catalog Project Environment** (17 min)

The most critical configuration unit, focusing on modern data governance:

* **The Metastore:** Understanding the top-level container for all catalogs.
* **Three-Level Namespace:** Organizing data via `catalog.schema.table`.
* **Compute Setup:** Creating clusters with the correct **Access Mode** (Shared or Single User) to enable Unity Catalog features like lineage and fine-grained access control.

---

## Key Technologies

| Component | Role |
| --- | --- |
| **Apache Spark** | The distributed execution engine. |
| **Data Lake** | The physical storage layer for vast amounts of raw data. |
| **Unity Catalog** | The governance layer for discovery, auditing, and security. |
| **Delta Lake** | The table format that brings ACID transactions to the Data Lake. |

---

## Setup Checklist

Before proceeding to the coding labs, ensure you have:

1. Verified access to your cloud storage account.
2. Ensured your Databricks workspace (or local environment) is **Unity Catalog-enabled**.
3. Generated the necessary API keys or tokens for external environment authentication.

---
