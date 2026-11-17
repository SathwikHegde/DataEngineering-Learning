
Section 18: Data-Level Security

This section outlines how to set up fine-grained access control (FGAC) in the Databricks Lakehouse. Aligned with the Data Engineer Associate exam blueprint, it covers dynamic data masking, row-level filtering, and Attribute-Based Access Control (ABAC) using Unity Catalog. These features help you meet global privacy laws (like GDPR, CCPA, and HIPAA) directly at the metadata layer, removing the need to copy physical Parquet or Delta files into separate storage buckets.
Check image_5e6b09.png for the dependency graph and lesson order.

Section Overview

• Total Duration: 54 minutes
• Total Modules: 4
• Core Topics: Multi-tenant dynamic views, deterministic row filters, UDF-based column masks, and Unity Catalog governed tags for ABAC policies.

Curriculum Breakdown


127. Data-Level Security Overview (6 min)

• The Storage Duplication Problem: Older access control setups forced teams to copy restricted data into separate storage buckets for different user groups. This caused pipeline delays, wasted storage, and sync headaches.
• The Metadata-Driven Solution: Modern FGAC separates security rules from physical files. Unity Catalog evaluates who is running a query during compilation and automatically masks, filters, or hides records based on their permissions.

128. Data-Level Security Using Dynamic Views (14 min)

• Checking User Identity at Runtime: Use built-in SQL context functions inside standard views so Catalyst can evaluate permissions when a query runs.
• Key Context Functions:
	• is_account_group_member(): Returns TRUE or FALSE depending on whether the current user belongs to a specific account-level group.
	• current_user(): Returns the exact username or email of the person running the query.
• Example Implementation:
sql
CREATE OR REPLACE VIEW production.silver.secure_salaries AS
SELECT
  employee_id,
  department,
  CASE 
    WHEN is_account_group_member('hr-managers') THEN raw_salary
    ELSE NULL 
  END AS annual_salary
FROM production.silver.base_payroll;
Use code with caution.

129. Data-Level Security Using Row Filters and Column Masks (13 min)

• Table-Level Policy Binding: Attach security rules directly to table metadata in Unity Catalog. This ensures rules apply consistently across notebooks, SQL Warehouses, and APIs before any data is returned.
• Row Filters: Hide specific rows based on a condition. The Catalyst Optimizer automatically adds this filter to the query plan so unauthorized users never see restricted rows.
• Column Masks: Hide or scramble sensitive values inside a column using a scalar User-Defined Function (UDF). The underlying Parquet files stay untouched while masked values are generated on the fly.
• Example Implementation:
sql
-- 1. Create the masking function
CREATE OR REPLACE FUNCTION governance.policies.mask_ssn(ssn STRING)
RETURN CASE 
  WHEN is_account_group_member('payroll-admins') THEN ssn
  ELSE CONCAT('XXX-XX-', RIGHT(ssn, 4))
END;

-- 2. Apply the mask to a table column
ALTER TABLE production.silver.customer_profiles 
ALTER COLUMN social_security_number SET MASK governance.policies.mask_ssn;
Use code with caution.

130. Data-Level Security Using ABAC Policies and Governed Tags (21 min)

• Attribute-Based Access Control (ABAC): Move beyond rigid Role-Based Access Control (RBAC) matrices to scalable, metadata-driven security rules.
• Governed Tags: Attach standardized key-value tags (such as Classification = PII or Confidentiality = High) to catalogs, schemas, tables, or columns in Catalog Explorer.
• Tag-Based Policies: Write broad security rules that check object tags at runtime. Instead of attaching masks one table at a time, a single policy can automatically mask every column tagged Classification = PII unless the user belongs to the Compliance group.

Key Exam Takeaways

• Catalyst Predicate Pushdown: Catalyst evaluates row filters and column masks during logical plan generation. It pushes these filters down to cloud storage and uses Delta/Parquet min/max stats to avoid unnecessary data scans and network bottlenecks.
• Table Owner Bypass: A table's owner (the user or group that created it) bypasses row filters and column masks by default unless the security function explicitly restricts the owner.
• One Mask Per Column: Unity Catalog allows only one column mask per column. If you need multiple conditions or branches, combine all logic into a single SQL UDF before attaching it.
← Back to Section 17: Data Governance with Unity Catalog | Next Section: Section 19: Delta Sharing & Lakehouse Federation →
Would you like to adjust the tone (e.g., more technical or even shorter), or paste another section README to rewrite?
