# Section 5: Introduction to Unity Catalog

This section serves as the definitive guide to **Unity Catalog (UC)**, the unified governance layer for the Databricks Data Intelligence Platform. Understanding UC is critical for the Associate certification, as it marks the shift from workspace-local data management to a centralized, cross-workspace governance model.

---

## Section Overview

* **Total Duration:** 50 minutes
* **Total Lessons:** 6
* **Primary Focus:** Transitioning from the legacy Hive Metastore to the 3-tier namespace and securing cloud storage.

---

## Curriculum Breakdown

| Lesson # | Title | Duration | Key Learning Outcome |
| --- | --- | --- | --- |
| **25** | **Introduction to Unity Catalog** | 6 min | Overview of unified governance for data and AI. |
| **26** | **UC / Hive Metastore Object Model** | 6 min | Understanding the transition from 2-tier to 3-tier naming. |
| **27** | **Create Unity Catalog Metastore** | 14 min | Step-by-step walkthrough of creating the regional metastore. |
| **28** | **Cluster Configurations for UC** | 4 min | Configuring "Shared" or "Single User" access modes. |
| **29** | **Configure Access to Cloud Storage (Lecture)** | 5 min | Overview of Storage Credentials and External Locations. |
| **30** | **Configure Access to Cloud Storage (Demo)** | 14 min | Demo of connecting Databricks to Azure Data Lake Storage. |

---

## Core Architectural Shifts

### 1. The 3-Tier Namespace

Unity Catalog organizes data assets using a **three-tier hierarchy**:

1. **Catalog:** The highest level of the container (e.g., `production`).
2. **Schema (Database):** A logical grouping within a catalog.
3. **Table / View / Volume:** The actual data object (e.g., `production.silver.customers`).

### 2. Unified Governance

Unlike the legacy **Hive Metastore**, which is often siloed within a single workspace, a **Unity Catalog Metastore** is a top-level container that can be assigned to multiple workspaces in the same region. This allows for centralized auditing, lineage tracking, and permission management across the entire enterprise.

### 3. Securing Cloud Infrastructure

* **Storage Credentials:** Securely encapsulate identity management (Service Principal or Managed Identity) used to authenticate with cloud storage buckets.
* **External Locations:** Define specific cloud storage paths governed by Unity Catalog, eliminating the need for users to manage raw storage keys or SAS tokens directly.

---

## Important Exam Considerations

* **One Metastore per Region**: A single Unity Catalog metastore is typically deployed per cloud region per account and mapped across multiple workspaces.
* **Compute Access Modes**: To query UC-governed assets, compute clusters must be configured with supported UC access modes (**Shared** or **Single User**). Legacy "No Isolation" modes cannot access UC tables.
* **Governance Inheritance**: Permissions granted via `GRANT` / `REVOKE` cascade downwards through the object hierarchy from Catalog down to Schema and individual Tables/Views.

---

[← Back to Section 4: Databricks Workspace Architecture & Developer Tools](https://www.google.com/search?q=./section04-readme.md) | [Next Section: Section 6: Data Objects in the Lakehouse →](https://www.google.com/search?q=./section06-readme.md)
