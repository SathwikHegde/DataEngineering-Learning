# Section 17: Data Governance

This section centers entirely on **Data Governance**, a core pillar of the Databricks Certified Data Engineer Associate exam. You will dive deep into how security, compliance, and asset discovery are enforced at scale using **Unity Catalog**, contrasting modern fine-grained access control models with legacy paradigms.

Refer to **image_eeab04.png** for the lesson timeline and curriculum breakdown.

---

## Section Overview

* **Total Duration:** 40 minutes
* **Total Lessons:** 5
* **Primary Focus:** Centralized data cataloging, line-of-business security, data lineage tracing, and data access control lists (ACLs).

---

## Curriculum Breakdown

### 122. Introduction to Data Governance (6 min)

* **The Challenge**: Managing access silos across multiple cloud object storage accounts, workspaces, and disparate business units without performance bottlenecks.
* **The Modern Mandate**: Ensuring data reliability, row-and-column-level confidentiality, regulatory compliance (GDPR/CCPA), and comprehensive operational audit trails across the entire enterprise Lakehouse.

### 123. Data Governance using Unity Catalog (7 min)

* **Centralized Governance**: Unity Catalog acts as a single, cross-workspace cataloging interface that sits above individual Databricks workspaces.
* **Object Hierarchy Recap**: Administering permissions across the defined 3-tier namespace structure:

$$\text{Catalog} \longrightarrow \text{Schema (Database)} \longrightarrow \text{Table / View / Volume}$$


* **Securable Objects**: Granting or revoking privileges on non-tabular assets like functional models, storage external locations, and shares.

### 124. Data Discovery, Audit & Lineage Demo (11 min)

* **Data Discovery**: Leveraging the integrated Catalog Explorer search engine to discover verified, business-ready datasets and read descriptions or documentation tags.
* **Automated Data Lineage**: A visual interface showing exactly how data flows from Bronze to Gold tiers. Unity Catalog captures run-time dependencies down to the individual column level without requiring code decoration.
* **Audit Logs**: Viewing the historical timeline of who read, modified, or altered permissions on a specific asset for comprehensive compliance tracking.

### 125. Data Access Control & Security (15 min)

* **SQL Standard Privileges**: Managing system identities using familiar, declarative SQL syntax:
```sql
-- Granting read access on a clean analytical table
GRANT SELECT ON TABLE market_intelligence.gold.monthly_summaries TO `finance-consumers`;

-- Granting full schema development ownership
GRANT CREATE TABLE ON SCHEMA ecom_analytics.silver TO `data-engineers`;

```


* **Dynamic Masking**: Implementing dynamic row-level filtering and column-level masking strings based on current user session contexts (`is_account_group_member()`) to protect Sensitive Personal Information (SPI) natively.

### 126. Legacy Privilege Model (1 min)

* **The Legacy Contrast**: Briefly contrasting modern identity federation with workspace-local table access control lists (ACLs) and the old Hive Metastore structure.
* **Deprecation Notice**: Why migrating old workspace-bound assets into isolated Unity Catalog containers is a critical priority for production architectures.

---

## Important Exam Considerations

* **Lineage Requirements**: For automated lineage tracking to function, queries must be executed on compute clusters configured with **Unity Catalog-compatible access modes** (Shared or Single User). Lineage is not captured on legacy, non-UC clusters.
* **Privilege Inheritance**: Permissions flow downward automatically through the object hierarchy. If a user group is granted `USAGE` and `SELECT` at the **Catalog** level, they implicitly retain read privileges on every current and future table within all schemas inside that catalog.
* **Principal Identification**: Unity Catalog utilizes a unified, account-level identity management sync. Ensure you grant permissions to synchronized account groups rather than localized, workspace-specific groups.

---

[Back to Course Introduction & Overview →](https://www.google.com/search?q=./README.md)

