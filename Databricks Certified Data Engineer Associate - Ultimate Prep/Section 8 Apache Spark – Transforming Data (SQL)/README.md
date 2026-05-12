# Section 8: Apache Spark – Transforming Data (SQL)

This section shifts from environment setup to the **core logic of data engineering**: applying transformations to turn raw datasets into high-value business entities. This module focuses on using **Spark SQL** to clean, restructure, and enrich e-commerce data.

---

## Module Breakdown

### **46. Data Profiling in Databricks** (11 min)

Before transforming data, you must understand its distribution and quality.

* **Summary Statistics**: Using `describe()` and `summary()` to view counts, means, and standard deviations.
* **Databricks Visual Profiler**: Utilizing the built-in "Data Profile" tab in notebooks to identify null values and skewed distributions at a glance.

### **47–51. Transform Business Entities** (47 min total)

These units focus on the "Silver" layer of the Medallion architecture—cleaning specific domain data:

* **Customers & Memberships**: Handling de-duplication, case normalization (Upper/Lower), and type casting.
* **Payments & Refunds**: Standardizing currency formats and date/time strings for financial consistency.
* **Addresses**: Normalizing geographical fields and preparing data for relational joins.

### **52–54. Handling Complex Orders Data (JSON)** (26 min)

Modern data often arrives as semi-structured JSON strings.

* **Querying Strings**: Using `get_json_object` to extract specific fields without parsing the whole string.
* **Conversion**: Using `from_json` with a defined schema to convert raw strings into structured **StructType** or **MapType** columns.
* **Exploding Arrays**: Using the `explode()` function to "flatten" order line items from a nested array into individual rows for granular analysis.

### **55–56. Relational Operations & Aggregations** (15 min)

Combining datasets to create business-ready "Gold" tables.

* **JOINS**: Performing **Inner** and **Left** joins to link customers with their normalized address data.
* **Monthly Summaries**: Using `date_trunc('month', ...)` or `month()` combined with `GROUP BY` to calculate total revenue and order volume per month.

### **57–59. Advanced Spark SQL Functions** (29 min)

Extending Spark's capabilities for logic that goes beyond standard SQL.

* **User Defined Functions (UDFs)**: Creating custom Python or SQL functions for specialized logic, while being mindful of performance overhead.
* **Higher Order Functions**: Using `transform()`, `filter()`, and `exists()` to manipulate complex array and map data directly within SQL queries without exploding the rows.

---

## Key Technical Skills

| Feature | Functionality |
| --- | --- |
| **JSON Processing** | `get_json_object`, `from_json`, `to_json` |
| **Data Cleaning** | `coalesce`, `distinct`, `regexp_replace` |
| **Complex Types** | `explode`, `array_contains`, `struct` |
| **Functional SQL** | Lambda expressions within Higher Order Functions |

---

## Transformation Checklist

* [ ] Did you validate your **JSON Schema** before using `from_json`?
* [ ] Have you checked for **Data Skew** during your aggregations?
* [ ] Can your UDF logic be replaced by a **Native Spark Function** for better performance?

---

**Since you're working through these data transformations, would you like me to generate a "SQL vs. Higher-Order Functions" cheat sheet to help you decide when to use each for array manipulation?**