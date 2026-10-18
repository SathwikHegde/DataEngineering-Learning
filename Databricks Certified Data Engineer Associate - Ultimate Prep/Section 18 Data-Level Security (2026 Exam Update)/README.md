# Section 18: Data-Level Security

This module defines the implementation of fine-grained access control (FGAC) within the Databricks Lakehouse architecture. Aligned with the Data Engineer Associate blueprint, this section covers the deployment of dynamic data masking, row-level predicate filtering, and Attribute-Based Access Control (ABAC) via Unity Catalog. These mechanisms enforce global regulatory compliance (e.g., GDPR, CCPA, HIPAA) dynamically at the metadata tier, eliminating the operational overhead of duplicating physical Parquet or Delta objects across isolated storage containers.

Refer to `image_5e6b09.png` for the topological dependency graph and execution sequencing.

---

## Section Overview

* **Total Duration:** 54 minutes
* **Total Modules:** 4
* **Primary Focus:** Dynamic multi-tenant view abstractions, deterministic Row Filters, UDF-based Column Masks, and Unity Catalog governed tags for ABAC orchestration.

---

## Curriculum Breakdown

### 127. Data-Level Security Overview (6 min)

* **The Physical Storage Bottleneck**: Legacy access control paradigms required the physical duplication of restricted datasets across isolated storage environments to enforce tenant separation. This architecture introduced massive ETL pipeline latency, data redundancy, and synchronization overhead.
* **The Unified Metadata Solution**: Contemporary FGAC decouples security logic from the physical storage substrate. Unity Catalog utilizes runtime evaluation during the query compilation phase to autonomously obfuscate, filter, or project records based on the invoking principal's identity and active authorization grants.

### 128. Data-Level Security Using Dynamic Views (14 min)

* **Runtime Context Evaluation**: Embedding native SQL context functions within standard view definitions to evaluate principal authorization dynamically during Catalyst query compilation.
* **Core Execution Functions**:
* `is_account_group_member()`: Evaluates a boolean condition against the executing principal's group assignments, securely synchronized at the Databricks account tier.
* `current_user()`: Extracts the deterministic string representation of the active principal invoking the Spark session.


* **Implementation Pattern**:

```sql
CREATE OR REPLACE VIEW production.silver.secure_salaries AS
SELECT
  employee_id,
  department,
  CASE 
    WHEN is_account_group_member('hr-managers') THEN raw_salary
    ELSE NULL 
  END AS annual_salary
FROM production.silver.base_payroll;

```

### 129. Data-Level Security Using Row Filters and Column Masks (13 min)

* **Policy Decoupling via Unity Catalog**: Binding deterministic security policies directly to table metadata within Unity Catalog. This architecture guarantees that access restrictions are universally enforced across all compute planes (interactive notebooks, SQL Warehouses, REST API endpoints) prior to payload materialization.
* **Row Filters**: Enforcing horizontal, predicate-based restrictions. The Catalyst Optimizer intercepts the logical query plan and injects the filter condition, restricting row-level exposure based on predefined column vectors.
* **Column Masks**: Executing vertical obfuscation via scalar User-Defined Functions (UDFs). The underlying physical Parquet files remain immutable, while masked payloads are dynamically computed during the query projection phase.
* **Implementation Pattern**:

```sql
-- 1. Define the deterministic security function
CREATE OR REPLACE FUNCTION governance.policies.mask_ssn(ssn STRING)
RETURN CASE 
  WHEN is_account_group_member('payroll-admins') THEN ssn
  ELSE CONCAT('XXX-XX-', RIGHT(ssn, 4))
END;

-- 2. Bind the function to the target schema attribute
ALTER TABLE production.silver.customer_profiles 
ALTER COLUMN social_security_number SET MASK governance.policies.mask_ssn;

```

### 130. Data-Level Security Using ABAC Policies and Governed Tags (21 min)

* **Attribute-Based Access Control (ABAC)**: Transitioning from rigid, matrix-driven Role-Based Access Control (RBAC) to highly scalable, metadata-driven policy execution.
* **Governed Tags**: Assigning standardized key-value metadata pairs (e.g., `Classification = PII` or `Confidentiality = High`) to securable objects across the Catalog Explorer namespace (catalogs, schemas, tables, or columns).
* **Tag-Based Policies**: Abstracting security logic into generalized routines that evaluate object tags dynamically at runtime. Instead of defining discrete column masks per table, a unified policy can declare that any column tagged `Classification = PII` is automatically obfuscated unless the invoking principal holds verified membership in the Compliance group.

---

## Important Exam Considerations

* **Catalyst Optimizer & Predicate Pushdown**: Row filters and column masks are evaluated by the Catalyst Optimizer during the logical plan generation phase on serverless or interactive compute clusters. Catalyst aggressively pushes these implicit filter predicates down to the cloud object storage layer, leveraging Delta/Parquet min/max statistics to guarantee that FGAC enforcement does not induce network I/O bottlenecks.
* **Execution Context Privileges**: The explicitly defined Owner of a securable table (the principal identity or group mapped to the object's creation) inherently bypasses localized row filters and column masks unless the policy logic explicitly captures and restricts the Owner role.
* **UDF Nesting Constraints**: Unity Catalog explicitly prohibits stacking multiple independent Column Masks onto a single target column. Complex conditional logic or multi-branch evaluations must be completely consolidated into a single, unified SQL UDF framework prior to metadata binding.

---

[← Back to Section 17: Data Governance with Unity Catalog](https://www.google.com/search?q=./section17-readme.md) | [Next Section: Section 19: Delta Sharing & Lakehouse Federation →](https://www.google.com/search?q=./section19-readme.md)
