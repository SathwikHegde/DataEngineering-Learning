# Section 17: Data Governance with Unity Catalog

This section centers entirely on **Data Governance**, a core pillar of the Databricks Certified Data Engineer Associate exam. You will dive deep into how security, compliance, and asset discovery are enforced at scale using **Unity Catalog**, contrasting modern fine-grained access control models with legacy, workspace-local paradigms.

Refer to **image_eeab04.png** for the lesson timeline and curriculum breakdown.

---

## Section Overview

* **Total Duration:** 40 minutes
* **Total Lessons:** 5
* **Primary Focus:** Centralized data cataloging, object inheritance hierarchies, automated column-level lineage tracing, and data access control lists (ACLs).

---

## Curriculum Breakdown

### 122. Introduction to Data Governance (6 min)

* **The Enterprise Challenge**: Historically, managing separate data access silos across multiple cloud object storage accounts, business units, and global cloud regions created severe operational bottlenecks and security vulnerabilities.
* **The Compliance Mandate**: Modern engineering demands centralized control over data reliability, row-and-column-level confidentiality, regulatory audits (GDPR, CCPA), and comprehensive, immutable operational audit trails across the entire enterprise Lakehouse.

### 123. Data Governance using Unity Catalog (7 min)

* **Centralized Control Plane**: Unity Catalog acts as a single, cross-workspace governance layer that sits completely above individual Databricks workspaces, unifying identity management and security privileges globally.
* **Object Hierarchy Mechanics**: Administering granular permissions across the standard 3-tier namespace structure:

$$\text{Catalog} \longrightarrow \text{Schema (Database)} \longrightarrow \text{Table / View / Volume}$$


* **Securable Objects**: Managing access permissions past standard tables to abstract assets—including registered ML models, storage External Locations, storage credentials, and clean-room Shares.

### 124. Data Discovery, Audit & Lineage Demo (11 min)

* **Data Discovery UI**: Leveraging the integrated Catalog Explorer search interface to discover business-ready datasets, trace markdown schema documentation, and audit tag compliance.
* **Automated Data Lineage Tracking**: Utilizing a fully interactive visual graph interface that charts data dependency paths from raw ingestion up to the presentation tiers. Unity Catalog captures run-time dependencies down to the individual column level automatically without requiring code decoration or manual annotation.
* **Audit Log Inspections**: Querying system logs to compile historical timelines showing exactly who read, modified, or altered permissions on any specific metadata asset.

### 125. Data Access Control & Security (15 min)

* **SQL Standard Privileges**: Provisioning group and user permissions natively using familiar ANSI-compliant SQL syntax:
```sql
-- Granting read access on a clean analytical gold table
GRANT USAGE ON CATALOG market_intelligence TO `finance-consumers`;
GRANT USAGE ON SCHEMA market_intelligence.gold TO `finance-consumers`;
GRANT SELECT ON TABLE market_intelligence.gold.monthly_summaries TO `finance-consumers`;

-- Granting full schema development ownership to an engineering team
GRANT CREATE TABLE, CREATE VIEW ON SCHEMA ecom_analytics.silver TO `data-engineers`;

```


* **Dynamic Security Masks**: Implementing real-time row-level filtering and column-level masking policies based on current user session contexts (using built-in functions like `is_account_group_member()`) to mask sensitive data elements (PII/SPI) on the fly.

### 126. Legacy Privilege Model (1 min)

* **The Legacy Contrast**: Contrasting modern account-level identity federation with workspace-local table access control lists (ACLs) and the old, un-governed Hive Metastore structure.
* **Deprecation Pathways**: Why migrating legacy workspace-bound assets into structured Unity Catalog containers is a critical priority for robust production architectures.

---

## Important Exam Considerations

* **Lineage Requirements**: For automated lineage tracking to function, queries must be executed on compute clusters configured with **Unity Catalog-compatible access modes** (**Shared** or **Single User**). Lineage is not captured on legacy, non-UC "No Isolation" clusters.
* **Privilege Inheritance Logic**: Permissions flow downward automatically through the object hierarchy. If a user group is granted `USAGE` and `SELECT` at the **Catalog** level, they implicitly retain read privileges on every current and future table within all schemas inside that catalog.
* **Unified Principal Identification**: Unity Catalog utilizes a unified, account-level identity management sync. Ensure your test answers prioritize granting permissions to synchronized account groups rather than localized, workspace-specific users.

---

[← Back to Section 16: Lakeflow Jobs & Workflow Orchestration](https://www.google.com/search?q=./section16-readme.md) | [Next Section: Section 18: Advanced Security - Row Filters & Column Masks →](https://www.google.com/search?q=./section18-readme.md)
