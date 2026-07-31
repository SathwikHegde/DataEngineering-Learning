# Section 2: Azure Subscription Setup & Environment Preparation

This section guides you through building your core cloud infrastructure. Because Azure Databricks runs directly within your managed cloud tenant, configuring a fresh Azure subscription is the essential first step before deploying compute workspaces, data lakes, and secure storage layers.

*Review your course dashboard to access the video lectures and time-coded walkthroughs for this section.*

---

## Section Overview

* **Total Duration:** 10 minutes
* **Total Lessons:** 2
* **Primary Focus:** Azure account creation, cloud credit management, portal interface navigation, and resource organization.

---

## Curriculum Breakdown

### 1. Registering an Azure Free Account (6 min)

* **Setting Up Your Sandbox**: A step-by-step walkthrough for creating a dedicated, safe learning environment. Your new account includes:
* **$200 Trial Credit**: A $200 credit valid for 30 days to test and deploy any Azure resources.
* **12 Months of Select Free Services**: Complimentary monthly allowances for key operational tools, including virtual machines and relational databases.
* **Always-Free Tier**: Permanent access to foundational serverless functions, lightweight databases, and messaging hubs—even after your initial credits expire.



### 2. Navigating the Azure Portal (4 min)

* **Exploring the Management Dashboard**: A guided tour of the Azure Portal, focused on the tools you'll use as a data engineer:
* **Resource Groups**: Learn to create logical boundaries (such as `rg-databricks-prep-01`) to isolate and manage permissions for all your Databricks and data lake assets.
* **Global Search & Discovery**: Quickly locate essential resources, specifically **Azure Databricks Workspaces** and **Azure Data Lake Storage Gen2 (ADLS Gen2)** accounts.
* **Azure Marketplace**: Provision official managed services and pre-configured software templates directly into your environment.
* **Cost Management & Budgets**: Set up automated spending alerts to monitor your remaining free credits and prevent unexpected charges.



---

## Key Takeaways for Data Engineers

* **Credit Card Verification**: You must provide a credit card during signup solely to confirm your identity. Azure places a protective spending limit on free accounts, so you won't be charged unless you explicitly choose to upgrade to a **Pay-As-You-Go** subscription.
* **Selecting the Right Region**: When creating Resource Groups and storage, pick an Azure region geographically near you. This reduces network latency and ensures reliable access to compute capacity.
* **Organized Cleanup**: Keep all lab resources inside dedicated, isolated Resource Groups. This makes teardown quick and simple when you complete a project, preventing idle clusters or stray storage accounts from draining your credits.

---

[← Back to Section 1: Course Overview](https://www.google.com/search?q=./section01-readme.md) | [Next Section: Section 3: Introduction to Lakehouse Architecture →](https://www.google.com/search?q=./section03-readme.md)
