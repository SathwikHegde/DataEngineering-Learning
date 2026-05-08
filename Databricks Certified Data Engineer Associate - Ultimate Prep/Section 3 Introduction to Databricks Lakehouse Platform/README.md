# Section 4: Databricks Workspace Components

This section is the "engine room" of the course. You will move from architectural theory to hands-on proficiency with the specific tools and interfaces used by Data Engineers every day. By the end of this module, you will be able to configure high-performance compute, leverage advanced notebook features, and implement professional version control workflows.

---

## Section Modules

### 14. Databricks Architecture Overview (8 min)

* **Plane Separation:** Understanding the distinction between the **Control Plane** (hosted by Databricks) and the **Data Plane** (hosted in your Azure subscription).
* **The 2026 Shift:** Introduction to **Serverless Workspaces**, which are now Generally Available (GA), offering a fully managed SaaS experience where Databricks handles the infrastructure scaling and security patching.

### 15. Introduction to Databricks Compute (4 min)

* **Compute Types:** A breakdown of **All-Purpose Compute** (for interactive development), **Job Compute** (automated, cost-efficient for production), and **SQL Warehouses** (optimized for BI).
* **Serverless vs. Classic:** Why serverless is becoming the default for notebooks and Lakeflow jobs due to instant start times and zero-management overhead.

### 16. Databricks Cluster Configuration (8 min)

* **Sizing & Scaling:** Best practices for setting worker types, enabling **Autoscaling**, and configuring **Autotermination** (recommended 15–30 mins) to prevent runaway costs.
* **Policy & Governance:** Using **Compute Policies** to restrict cluster creation to specific VM sizes and enforce tagging for cost attribution.

### 17. Create Databricks Cluster (13 min)

* **Hands-on Lab:** Step-by-step creation of your first interactive cluster.
* **Runtime Selection:** Choosing the appropriate **Databricks Runtime (DBR)** (e.g., DBR 18.0+) to access the latest Spark features and security updates.

### 18. Troubleshooting Databricks Cluster Quota and VM Issues (8 min)

* **Azure Quotas:** Resolving the common "Quota Exceeded" error by requesting core increases in the Azure Portal or switching to lower-footprint VM types like `Standard_DS3_v2`.
* **Regional Availability:** How to identify if a specific VM family is unavailable in your chosen region.

### 19. Databricks Notebooks (15 min)

* **Collaborative Authoring:** Real-time co-authoring and commenting.
* **The Data Science Agent:** Using the new AI-powered assistant to generate EDA (Exploratory Data Analysis) code and visualize datasets with natural language prompts.
* **New in 2026:** Direct image pasting into Markdown and the "Tab Session Restore" feature for managing multiple active workflows.

### 20. Databricks Magic Commands (13 min)

* **Language Mixing:** Using `%sql`, `%python`, `%scala`, and `%r` within the same notebook.
* **System Commands:** `%sh` for shell scripts, `%fs` for file system exploration, and `%pip` for notebook-scoped library installation.
* **Profiling Magics:** Introduction to `%%profile` and `%%oprofile` (Runtime 17.2+) for deep-dive performance analysis of your Python code.

### 21. Databricks Utilities (9 min)

* **dbutils:** Mastering the core utility library.
* `dbutils.fs`: Managing the Databricks File System (DBFS).
* `dbutils.secrets`: Securely retrieving credentials from Azure Key Vault.
* `dbutils.widgets`: Creating parameterized notebooks for dynamic inputs.



### 22 & 23. Databricks Git Folders (Repos) & Demo (4 min + 16 min)

* **The Transition:** Understanding the shift from "Repos" to the more robust **Git folders** interface.
* **Full Git Lifecycle:** A live demo of cloning a repo, branch management, committing changes, and resolving merge conflicts directly in the UI.
* **Web Terminal:** Using the integrated terminal for advanced Git CLI operations like `git stash` and `git rebase`.

### 24. Debugging Databricks Notebooks (16 min)

* **Integrated Testing:** Utilizing the new **Tests Sidebar** for running pytest-based unit tests within your workspace.
* **Visual Debugging:** Using line numbering and the Spark UI to identify bottlenecks in wide transformations or skewed data.

---

## 💡 Pro-Tips for Section 4

1. **Use Serverless for Labs:** To maximize your $200 Azure credit, use **Serverless compute** whenever possible—it starts in seconds and you only pay for the exact duration your code runs.
2. **Tag Everything:** Always add a "Project" or "Owner" tag in your cluster configuration. This makes it significantly easier to track spending in the **Lakeflow System Tables**.
3. **Git-First Workflow:** Never write code in a standalone workspace folder. Always create a **Git folder** first to ensure your work is versioned and safe from accidental deletion.

---

[Next Section: Data Ingestion with Auto Loader & Spark →](https://www.google.com/search?q=./section5-readme.md)