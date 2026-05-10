# Section 5: Introduction to Unity Catalog

This section serves as the definitive guide to **Unity Catalog (UC)**, the unified governance layer for the Databricks Data Intelligence Platform. Understanding UC is critical for the Associate certification, as it marks the shift from workspace-local data management to a centralized, cross-workspace governance model.

---

## Section Overview

* **Total Duration:** 50 minutes
* **Lessons:** 6
* **Primary Focus:** Transitioning from the legacy Hive Metastore to the 3-tier namespace and securing cloud storage.

---

## Section Modules

| Lesson # | Title | Duration | Key Learning Outcome |
| --- | --- | --- | --- |
| 25 | **Introduction to Unity Catalog** | 6 min | Overview of unified governance for data and AI. |
| 26 | **UC / Hive Metastore Object Model** | 6 min | Understanding the transition from 2-tier to 3-tier naming. |
| 27 | **Create Unity Catalog Metastore** | 14 min | Step-by-step walkthrough of creating the regional metastore. |
| 28 | **Cluster Configurations for UC** | 4 min | Configuring "Shared" or "Single User" access modes. |
| 29 | **Configure Access to Cloud Storage** | 5 min | Lecture on Storage Credentials and External Locations. |
| 30 | **Configure Access to Cloud Storage** | 14 min | Demo of connecting Databricks to Azure Data Lake Storage. |

---

## Core Architectural Shifts

### **The 3-Tier Namespace**

In this section, you will learn how Unity Catalog organizes data assets using a **three-tier hierarchy**:

1. **Catalog:** The highest level of the container (e.g., `production`).
2. **Schema (Database):** A logical grouping within a catalog.
3. **Table / View / Volume:** The actual data object.

### **Unified Governance**

Unlike the legacy **Hive Metastore**, which is often siloed within a single workspace, a **Unity Catalog Metastore** is a top-level container that can be assigned to multiple workspaces in the same region. This allows for centralized auditing, lineage tracking, and permission management.

### **Securing the "Plumbing"**

Lessons 29 and 30 focus on the secure abstraction of your underlying cloud infrastructure:

* **Storage Credentials:** Securely store the identity (Managed Identity or Service Principal) used to talk to Azure Storage.
* **External Locations:** Define specific cloud storage paths that Unity Catalog governs, removing the need for users to manage individual storage keys or SAS tokens.

---

## Key Takeaways for the Exam

* **One Metastore per Region:** You generally only create one Unity Catalog metastore per region within your account.
* **Compute Compatibility:** To access UC-governed tables, you must use a cluster configured with **Unity Catalog-compatible access modes** (Shared or Single User).
* **Governance Hierarchy:** Permissions (GRANT/REVOKE) are inherited from the Catalog down to the individual Table.

---

[Next Section: Data Objects in the Lakehouse →](https://www.google.com/search?q=./section6-readme.md)