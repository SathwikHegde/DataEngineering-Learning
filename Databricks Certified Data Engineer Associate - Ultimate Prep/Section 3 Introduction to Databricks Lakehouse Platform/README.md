# Section 4: Databricks Workspace Components

This section details the core infrastructure and operational interfaces of the Databricks Workspace. The curriculum transitions from theoretical architectures to the practical provisioning of distributed compute resources, polyglot development environments, and version-controlled Infrastructure-as-Code (IaC) workflows required for enterprise data engineering.

---

## Curriculum Breakdown

### 14. Databricks Architecture Overview (8 min)

* **Control vs. Data Plane Architecture**: Defining the structural separation between the Databricks-managed Control Plane (web application, cluster manager, job scheduler) and the customer-managed Data Plane (virtual network, compute nodes, and cloud object storage).
* **Serverless Compute GA**: The deployment of Generally Available (GA) Serverless Workspaces, which offload compute provisioning, autoscaling algorithms, and security patching directly to Databricks' backend SaaS infrastructure.

### 15. Introduction to Databricks Compute (4 min)

* **Compute Topologies**: Distinguishing between **All-Purpose Compute** (for interactive REPL development), **Job Compute** (ephemeral, automated execution for production pipelines), and **SQL Warehouses** (vectorized query engines optimized for BI workloads).
* **Provisioning Models**: Evaluating the transition from Classic (customer-hosted VMs) to Serverless compute, emphasizing zero-management overhead and sub-second startup latencies.

### 16. Databricks Cluster Configuration (8 min)

* **Topology & Auto-Scaling**: Defining worker node instance types, configuring autoscaling thresholds, and enforcing autotermination limits (recommended 15-30 minute idle intervals) to optimize cloud resource expenditures.
* **Compute Policies & Resource Governance**: Implementing Compute Policies to constrain VM selection bounds and enforce mandatory metadata tagging for downstream cost attribution.

### 17. Create Databricks Cluster (13 min)

* **Interactive Cluster Provisioning**: Step-by-step instantiation of a distributed cluster workspace within the Azure environment.
* **Databricks Runtime (DBR) Evaluation**: Selecting the optimal runtime version (e.g., DBR 18.0+) to ensure compatibility with modern Spark API features, Delta Lake protocols, and critical OS-level vulnerability patches.

### 18. Troubleshooting Databricks Cluster Quota and VM Issues (8 min)

* **Azure vCPU Quota Mitigation**: Diagnosing Azure "Quota Exceeded" exceptions and executing quota increase requests via the Azure Resource Manager (ARM), or pivoting to lower-footprint VM families (e.g., `Standard_DS3_v2`).
* **Geographic VM Availability**: Analyzing regional capacity limitations for specific hardware SKU deployments.

### 19. Databricks Notebooks (15 min)

* **Collaborative Development Environments (CDE)**: Synchronous co-authoring, version history tracking, and real-time state management.
* **AI-Assisted Code Generation**: Leveraging the integrated Data Science Assistant for natural language-to-code compilation and programmatic Exploratory Data Analysis (EDA).
* **Workspace Enhancements**: Utilizing Tab Session Restore for state persistence across multi-pipeline workflows and integrating markdown-native image embedding.

### 20. Databricks Magic Commands (13 min)

* **Polyglot Execution**: Overriding default cell environments via `%sql`, `%python`, `%scala`, and `%r` to execute multi-language Directed Acyclic Graphs (DAGs) within a unified interface.
* **OS & Filesystem Interfacing**: Interacting with the operating system layer via `%sh`, navigating the Databricks File System via `%fs`, and executing isolated package dependency management via `%pip`.
* **Runtime Profiling**: Implementing `%%profile` and `%%oprofile` (available in DBR 17.2+) for granular CPU and memory profiling of Python execution blocks.

### 21. Databricks Utilities (9 min)

* **The `dbutils` API**: Programmatic interfacing with workspace environmental variables and secrets.
* **Core Modules**: Utilizing `dbutils.fs` for distributed filesystem operations, `dbutils.secrets` for secure Azure Key Vault credential retrieval, and `dbutils.widgets` for dynamic parameter injection at runtime.

### 22 & 23. Databricks Git Folders (Repos) & Demo (4 min + 16 min)

* **Git Folder Architecture**: Transitioning from legacy Repos to the modernized Git Folders architecture for persistent, branch-based version control.
* **CI/CD Lifecycle Management**: Executing Git clone operations, branch checkout, commit tracking, and merge conflict resolution via the native workspace UI.
* **Integrated CLI Access**: Utilizing the workspace Web Terminal for advanced Git command-line operations (e.g., `git stash`, `git rebase`).

### 24. Debugging Databricks Notebooks (16 min)

* **Automated Unit Testing**: Executing `pytest` validation frameworks directly against notebook cells via the integrated Tests Sidebar.
* **Spark UI Profiling & Bottleneck Resolution**: Diagnosing execution bottlenecks, data skew, and shuffle latencies utilizing line-by-line metrics and the Catalyst Spark UI DAG visualizer.

---

## Technical Best Practices

1. **Serverless Unit Economics**: Prioritize Serverless compute configurations for interactive development to minimize Azure DBU credit consumption, leveraging instant-on capabilities and exact-duration billing cycles.
2. **Mandatory Cost Attribution**: Mandate "Project" or "Owner" tagging definitions within cluster policies to facilitate downstream cost allocation and auditing via Unity Catalog System Tables.
3. **Version-Controlled Workflows**: Deprecate standalone workspace folder development. Initialize all pipelines within Git Folders to ensure code persistence, auditability, and seamless integration into enterprise CI/CD runners.

---

[Back to Section 3: Introduction to Lakehouse Architecture](https://www.google.com/search?q=./section03-readme.md) | [Next Section: Section 5: Introduction to Unity Catalog Governance](https://www.google.com/search?q=./section05-readme.md)
