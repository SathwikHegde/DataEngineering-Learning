Section 2: Azure Subscription Setup & Environment Readiness

This section focuses on preparing the core Azure environment required for deploying and managing Azure Databricks. Because Azure Databricks is a fully managed, first-party Azure service, a properly configured Azure subscription, identity framework, and resource hierarchy must be established before implementing storage, compute, or governance components.

Refer to the course dashboard for the accompanying video demonstrations and guided walkthroughs.

Section Overview
Duration: 10 minutes
Lessons: 2
Core Topics: Azure tenant and subscription setup, cloud credits and entitlements, Azure Resource Manager (ARM) fundamentals, resource organization, and cost governance.
Lesson Breakdown
1. Creating an Azure Free Account (6 minutes)

This lesson walks through the process of setting up a dedicated Azure environment for learning and experimentation.

Key topics include:

Azure Sandbox Deployment

Create an Azure account and dedicated Azure Entra ID tenant.
Establish a subscription that serves as the foundation for all cloud resources used throughout the course.

$200 Free Credit

Receive a one-time 30-day credit allocation for exploring Azure services without financial risk.
Deploy and test services such as Databricks, storage accounts, and networking components within the available credit.

12-Month Free Services

Access selected infrastructure services with monthly usage limits for up to one year.
Includes compute instances, managed databases, and storage offerings.

Always-Free Azure Services

Continue using selected services with permanent free-tier quotas.
Examples include Azure Functions, Event Grid, and limited Cosmos DB resources.

Azure Identity Foundation

Understand the role of Azure Entra ID as the identity and access management layer.
Learn how authentication, authorization, RBAC, and service integrations are governed across Azure resources.
2. Azure Portal Navigation & Resource Organization (4 minutes)

This lesson introduces the Azure management experience and the organizational structure used to manage services effectively.

Key topics include:

Azure Resource Manager (ARM)
Learn how ARM acts as Azure's centralized management framework.
Create, modify, and govern resources consistently across subscriptions and regions.
Understand how Azure services are deployed and managed through the ARM control plane.
Resource Groups
Organize cloud resources into logical management containers.
Implement a structured naming convention, such as:
rg-databricks-dev-eastus-001

Use Resource Groups to:
Simplify lifecycle management
Enforce regional boundaries
Apply RBAC permissions
Manage environments such as development, testing, and production
Core Services for Data Engineering

Azure Databricks

Enable the Microsoft.Databricks resource provider.
Deploy Databricks workspaces through Azure Resource Manager.
Understand the relationship between Azure infrastructure and the Databricks control plane.

Azure Data Lake Storage Gen2 (ADLS Gen2)

Create storage accounts with Hierarchical Namespace (HNS) enabled.
Build the storage foundation required for:
Delta Lake tables
Lakehouse architectures
Unity Catalog external locations
Enterprise-scale analytics workloads
Cost Management & Budget Controls
Configure Azure Cost Management and Billing.
Establish spending thresholds and alert policies.
Create automated notifications at key consumption levels, such as:
50% of budget
75% of budget
90% of budget

These controls help prevent unexpected charges caused by long-running clusters, idle resources, or excessive compute consumption.

Essential Best Practices for Data Engineers
Manage Costs Proactively

Azure trial subscriptions include spending protections that prevent unintended charges. As you move toward production or a Pay-As-You-Go model, implement:

Budget alerts
Cost monitoring dashboards
Cluster auto-termination policies
Resource tagging standards

These measures help control both Azure infrastructure costs and Databricks Unit (DBU) consumption.

Keep Services in the Same Region

For optimal performance and cost efficiency, provision related services within the same Azure region.

Examples include:

Azure Databricks Workspace
ADLS Gen2 Storage
Unity Catalog Access Connectors
Supporting networking resources

Co-locating these resources minimizes latency, reduces data transfer overhead, and avoids unnecessary cross-region egress charges.

Use Resource Groups for Lifecycle Management

Place all project resources inside dedicated Resource Groups whenever possible.

Benefits include:

Simplified administration
Consistent security management
Easier environment cleanup
Prevention of orphaned resources

When a project is no longer needed, deleting the Resource Group removes all associated infrastructure in a single operation, helping conserve cloud credits and reduce operational overhead.

Previous: Section 1 - Course Overview
 Next: Section 3 - Introduction to Lakehouse Architecture →
