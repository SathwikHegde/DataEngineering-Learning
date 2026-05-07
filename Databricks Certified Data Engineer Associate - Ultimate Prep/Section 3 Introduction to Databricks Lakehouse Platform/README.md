# Section 3: Introduction to Databricks Lakehouse Platform

This section provides the architectural foundation of the course. You will move beyond basic cloud setup to understand the core philosophy of the **Data Lakehouse**—the paradigm shift that combines the low-cost storage of a data lake with the performance and structure of a data warehouse.

---

## Section Modules

### 9. Data Lakehouse Overview (11 min)
* **The Problem:** Traditional "Two-Tier" architectures (Data Lake + Data Warehouse) lead to data silos, stale data, and complex ETL.
* **The Solution:** The Lakehouse Platform.
* **Key Features:** Support for ACID transactions via **Delta Lake**, independent scaling of compute and storage, and support for diverse workloads (BI, ML, and Streaming) on a single copy of data.

### 10. Introduction to Medallion Architecture (5 min)
A deep dive into the industry-standard "multi-hop" data design pattern:
* **Bronze (Raw):** Ingesting data exactly as-is from source systems.
* **Silver (Filtered/Cleaned):** Applying schema enforcement, cleaning nulls, and joining tables to create a "Single Source of Truth."
* **Gold (Business-Ready):** Final aggregations and feature tables optimized for dashboards and machine learning models.

### 11. Databricks Overview (7 min)
Understanding the Databricks ecosystem as a unified **Data Intelligence Platform**:
* **The Foundation:** Built on open-source technologies like Apache Spark™, Delta Lake, and MLflow.
* **Unified Governance:** Introduction to **Unity Catalog** for managing data and AI assets across your entire organization.
* **AI Integration:** How the "Data Intelligence" layer uses generative AI to simplify SQL writing and data discovery.

### 12. Creating Azure Databricks Service (5 min)
Step-by-step walkthrough in the Azure Portal:
* **Deployment:** Navigating the Azure Marketplace to create a workspace.
* **Pricing Tiers:** Choosing between **Standard** and **Premium** (Premium is required for Unity Catalog and advanced security).
* **Managed Resource Groups:** Understanding how Azure automatically manages the virtual network and storage behind your workspace.

### 13. Databricks User Interface Overview (7 min)
Navigating the 2026 Workspace interface:
* **Persona Switcher:** Toggling between Data Engineering, Databricks SQL, and Machine Learning views.
* **Sidebar Navigation:** Accessing the Catalog Explorer, Workflows (Jobs), and Compute management.
* **Global Search:** Using the AI-powered search to find tables, notebooks, and documentation instantly.

---

## Key Takeaways for Data Engineers
1.  **Serverless First:** In 2026, Databricks defaults to **Serverless Compute**. It removes the need for manual cluster configuration, allowing you to focus on code while the platform handles the infrastructure scaling.
2.  **Schema Evolution:** Remember that while Bronze is often schema-less, the transition to Silver is where you must define and enforce data quality.
3.  **Compute Awareness:** Always be mindful of your **DBU (Databricks Unit)** consumption when testing notebooks to stay within your Azure credits.

---

[Next Section: Managing Databricks Compute & Clusters →](./section4-readme.md)