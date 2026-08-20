# Section 2: Azure Subscription Setup & Environment Preparation

This section establishes the foundational cloud infrastructure required to host enterprise-grade Databricks workspaces. Because Azure Databricks operates as a first-party, managed Software-as-a-Service (SaaS) and Platform-as-a-Service (PaaS) offering deeply integrated with Microsoft Azure, configuring a compliant subscription, identity perimeter, and resource hierarchy is the critical prerequisite prior to provisioning compute planes, Delta storage lakes, and Unity Catalog metastores.

Review your course dashboard to access the video lectures and synchronized walkthroughs for this section.

---

## Section Overview

* **Total Duration:** 10 minutes
* **Total Lessons:** 2
* **Primary Focus:** Azure Tenant and Subscription architecture, cloud entitlement and credit allocations, Azure Resource Manager (ARM) hierarchy, and proactive cost guardrails.

---

## Curriculum Breakdown

### 1. Registering an Azure Free Account (6 min)

* **Cloud Sandbox Provisioning**: Step-by-step walkthrough of creating an isolated Azure Entra ID (formerly Azure Active Directory) tenant and associated root subscription. The sandbox includes:
* **$200 Evaluation Credit**: A 30-day initial credit allocation enabling risk-free provisioning of compute clusters, storage accounts, and networking infrastructure.
* **12 Months of Tier-1 Services**: Extended allowances for core infrastructure components, including Linux/Windows compute instances, managed SQL databases, and standard object storage tiers.
* **Always-Free Tier Capacities**: Permanent baseline access to serverless components, such as Azure Functions, Event Grid subscriptions, and Cosmos DB throughput tiers.


* **Identity and Tenant Hierarchy**: Understanding how an Azure Subscription sits within a root Entra ID tenant, establishing the security perimeter where Role-Based Access Control (RBAC) and OAuth authorization tokens operate during workspace and storage integration.

### 2. Navigating the Azure Portal & Resource Organization (4 min)

* **Azure Resource Manager (ARM) Architecture**: Navigating the unified management layer used to create, update, and delete cloud resources declaratively and imperatively across regions.
* **Resource Group Boundary Isolation**:
* Implementing structured Resource Groups (e.g., `rg-databricks-dev-eastus-001`) to serve as logical management containers for all project-related assets.
* Leveraging Resource Groups to enforce unified lifecycle policies, regional data residency boundaries, and localized RBAC role assignments (`Contributor`, `Owner`, `Reader`).


* **Core Data Engineering Service Discovery**:
* **Azure Databricks Workspaces**: Locating and registering the `Microsoft.Databricks` resource provider to enable control plane and data plane deployment.
* **Azure Data Lake Storage Gen2 (ADLS Gen2)**: Identifying hierarchical namespace (HNS) enabled storage accounts (`Microsoft.Storage`) necessary for hosting Delta Lake table structures and Unity Catalog external storage roots.


* **Cost Management, Budgets, and Automated Alerts**:
* Configuring proactive cost boundaries within **Azure Cost Management + Billing**.
* Creating metric-driven budget alert thresholds (e.g., 50%, 75%, and 90% of total allocated credit) with automated email and webhook notifications to prevent runaway compute costs or unmonitored cluster idle times.



---

## Key Architectural Principles for Data Engineers

* **Spending Caps and Transition Economics**: Azure automatically enforces a $0 spending limit on trial subscriptions to prevent inadvertent billing. When evaluating production workloads or transitioning to a **Pay-As-You-Go** subscription, strict budget monitoring and cluster auto-termination rules must be configured to control Databricks Unit (DBU) and virtual machine core consumption.
* **Geographic Co-Location and Latency Mitigation**: All interconnected assets—including Resource Groups, Azure Databricks Workspaces, ADLS Gen2 storage accounts, and Unity Catalog access connectors—must be provisioned within the same Azure geographic region (e.g., `eastus` or `westus2`). Cross-region network traffic introduces severe network latency penalties and incurs billable egress bandwidth costs during large-scale Spark shuffles and data extraction tasks.
* **Resource Lifecycle Encapsulation**: Placing all sandbox infrastructure components inside dedicated, project-specific Resource Groups enables atomic teardown. Deleting a single Resource Group automatically cascades and terminates all encapsulated storage containers, virtual networks, and managed compute nodes, ensuring no orphan resources consume cloud credits.

---

[← Back to Section 1: Course Overview](https://www.google.com/search?q=./section01-readme.md) | [Next Section: Section 3: Introduction to Lakehouse Architecture →](https://www.google.com/search?q=./section03-readme.md)
