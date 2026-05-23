# Section 18: Data-Level Security (2026 Exam Update)

This section targets fine-grained access control mechanisms within Databricks, aligning with the updated **2026 Associate Certification** guidelines. You will master the technical tools required to isolate, mask, and filter data assets dynamically across complex corporate operational topologies, ensuring strict adherence to global privacy laws (such as GDPR, CCPA, and HIPAA) right at the storage tier.

Refer to **image_5e6b09.png** for the chronological lesson sequence covered in this security module.

---

## Section Overview

* **Total Duration:** 54 minutes
* **Total Lessons:** 4
* **Primary Focus:** Dynamic multi-tenant views, line-of-business masking rules, Row Filters, Column Masks, and Attribute-Based Access Control (ABAC).

---

## Curriculum Breakdown

### 127. Data-Level Security Overview (6 min)

* **The Traditional Bottleneck**: Historically, separating sensitive employee records or restricted regional metrics required building and maintaining entirely separate physical tables for distinct user groups.
* **The Unified Solution**: Modern data-level security abstracts security rules away from physical storage. It uses intelligent runtime evaluation to automatically hide or alter records based on who is querying the data.

### 128. Data-Level Security Using Dynamic Views (14 min)

* **Dynamic Scoping**: Using functional built-in context functions inside classic SQL view structures to evaluate user permissions during query execution.
* **Core Security Functions**:
* `is_account_group_member()`: Validates if the current user belongs to an identity group synchronized at the account tier.
* `current_user()`: Pulls the explicit email address string of the person executing the query cell.


* **Code Implementation Pattern**:
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

* **Decoupling Policies**: Moving beyond dynamic views by attaching security logic directly to the physical underlying Unity Catalog table itself. This ensures the protection persists regardless of how or where the table is accessed.
* **Row Filters**: Automatically restricting table records horizontally based on a conditional column characteristic (e.g., a field agent only sees rows matching their assigned `region_id`).
* **Column Masks**: Altering values vertically inside a specific column without modifying the actual underlying physical Parquet records on disk.
* **Code Implementation Pattern**:
```sql
-- 1. Create the reusable security function
CREATE OR REPLACE FUNCTION governance.policies.mask_ssn(ssn STRING)
RETURN CASE 
  WHEN is_account_group_member('payroll-admins') THEN ssn
  ELSE CONCAT('XXX-XX-', RIGHT(ssn, 4))
END;

-- 2. Bind the mask directly to the target column
ALTER TABLE production.silver.customer_profiles 
ALTER COLUMN social_security_number SET MASK governance.policies.mask_ssn;

```
### 130. Data-Level Security Using ABAC Policies and Governed Tags (21 min)

* **Attribute-Based Access Control (ABAC)**: Transitioning from rigid Role-Based policies (RBAC) to highly flexible, scalable identity-and-attribute tagging logic.
* **Governed Tags**: Assigning key-value metadata properties (such as `Classification = PII` or `Confidentiality = High`) to catalogs, schemas, tables, or individual columns inside the Catalog Explorer interface.
* **Tag-Based Policies**: Writing generalized security routines that automatically look for these tags. For example, you can write a single rule stating: *"If a column is tagged as `Classification = PII`, automatically mask the column for any user who is not a member of the Compliance group."* This removes the need to manually write hundreds of individual column masking functions across your enterprise.

---

## Important Exam Considerations

* **Performance Considerations**: Row filters and column masks are evaluated on serverless or interactive compute clusters at query compile time. They leverage the **Catalyst Optimizer** to push down filters into the storage layer, ensuring security rules don't cause significant query bottlenecks.
* **Owner Overrides**: Be aware that the explicit **Owner** of a table (the principal identity or group that created it) is typically immune to localized row filters and column masks unless explicitly specified within the policy logic.
* **Nesting Limitations**: You cannot apply multiple distinct Column Masks to the exact same column simultaneously. If complex conditional branches are needed, they must be contained within a single, unified SQL security function logic framework.

---

[Back to Course Introduction & Setup →](https://www.google.com/search?q=./README.md)