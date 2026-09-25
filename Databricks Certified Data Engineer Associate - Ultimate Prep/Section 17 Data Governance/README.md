# Section 17: Data Governance with Unity Catalog

This module examines the principles of Data Governance within the Databricks Lakehouse architecture. As a core competency for the Data Engineer Associate certification, this section details the enforcement of security, regulatory compliance, and asset discovery at scale utilizing Unity Catalog. It contrasts modern fine-grained access control (FGAC) architectures with legacy, workspace-local governance paradigms.

Refer to `image_eeab04.png` for the topological dependency graph and curriculum sequencing.

---

## Section Overview

* **Total Duration:** 40 minutes
* **Total Modules:** 5
* **Primary Focus:** Centralized metastore governance, object inheritance hierarchies, automated column-level lineage tracking, and Role-Based Access Control (RBAC) implementations.

---

## Curriculum Breakdown

### 122. Introduction to Data Governance (6 min)

* **Enterprise Governance Topologies**: Historically, managing fragmented data access across decentralized cloud object storage accounts and isolated workspaces induced severe operational latency and security vulnerabilities.
* **Regulatory Compliance Frameworks**: Modern data engineering necessitates centralized control over dataset reliability, row-and-column-level confidentiality, and immutable audit trails to satisfy stringent regulatory compliance mandates (e.g., GDPR, CCPA, HIPAA) across the enterprise Lakehouse.

### 123. Data Governance using Unity Catalog (7 min)

* **Centralized Control Plane**: Unity Catalog functions as a unified, cross-workspace governance layer operating above individual Databricks execution environments, centralizing identity federation and security privileges globally.
* **Object Inheritance Mechanics**: Administering granular access controls across the standardized three-tier Unity Catalog namespace:

$$\text{Catalog} \longrightarrow \text{Schema (Database)} \longrightarrow \text{Table / View / Volume}$$

* **Securable Assets**: Extending access control policies beyond relational tables to abstract infrastructural assets, including MLflow registered models, external Storage Credentials, External Locations, and Delta Sharing objects.

### 124. Data Discovery, Audit & Lineage Demo (11 min)

* **Asset Discovery**: Utilizing the integrated Catalog Explorer UI to discover governed datasets, parse schema metadata, and audit Attribute-Based Access Control (ABAC) tag compliance.
* **Automated Lineage Generation**: Leveraging the runtime execution graph to automatically chart data dependency paths from raw ingestion through to aggregated Gold layers. Unity Catalog captures runtime Catalyst dependencies down to the column level without requiring manual code instrumentation.
* **System Audit Telemetry**: Querying system tables and audit logs to compile deterministic historical timelines, detailing identity-based access, DML modifications, and privilege mutations on governed metadata assets.

### 125. Data Access Control & Security (15 min)

* **Standardized ANSI SQL Privileges**: Provisioning principal and group authorization natively utilizing standard Data Control Language (DCL) execution:

```sql
-- Granting read access on a conformed analytical Gold table
GRANT USAGE ON CATALOG market_intelligence TO `finance-consumers`;
GRANT USAGE ON SCHEMA market_intelligence.gold TO `finance-consumers`;
GRANT SELECT ON TABLE market_intelligence.gold.monthly_summaries TO `finance-consumers`;

-- Granting schema-level DDL execution privileges to an engineering principal
GRANT CREATE TABLE, CREATE VIEW ON SCHEMA ecom_analytics.silver TO `data-engineers`;

```

* **Dynamic Security Policies**: Engineering runtime Row-Level Security (RLS) filters and Column Masks. These policies utilize built-in context functions (e.g., `is_account_group_member()`) to obfuscate Sensitive Personal Information (SPI) or Personally Identifiable Information (PII) dynamically during the Catalyst query compilation phase.

### 126. Legacy Privilege Model (1 min)

* **Legacy Architecture Contrast**: Differentiating modern account-level identity federation from legacy workspace-local Access Control Lists (ACLs) and the deprecated, ungoverned Hive Metastore topology.
* **Migration Pathways**: Examining the architectural imperative of upgrading legacy workspace-bound assets into structured, governed Unity Catalog containers for production workloads.

---

## Important Exam Considerations

* **Lineage Execution Prerequisites**: Automated lineage tracking strictly requires query execution on compute clusters configured with **Unity Catalog-compatible access modes** (i.e., **Shared** or **Single User**). Catalyst lineage telemetry is not captured on legacy "No Isolation Shared" compute topologies.
* **Privilege Inheritance Semantics**: Access permissions automatically propagate downward through the Unity Catalog object hierarchy. Granting `USAGE` and `SELECT` at the **Catalog** level implicitly propagates read privileges to the principal for all current and future relational assets within all descendant schemas.
* **Identity Federation**: Unity Catalog mandates unified, account-level identity synchronization. Certification scenarios will prioritize granting privileges to synchronized Account Groups over localized, workspace-bound user identities.

---

[← Back to Section 16: Lakeflow Jobs & Workflow Orchestration](https://www.google.com/search?q=./section16-readme.md&utm_source=gemini) | [Next Section: Section 18: Advanced Security - Row Filters & Column Masks →](https://www.google.com/search?q=./section18-readme.md&utm_source=gemini)
