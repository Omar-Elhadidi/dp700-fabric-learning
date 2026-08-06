# M23: Implement Continuous Integration and Continuous Delivery (CI/CD) in Microsoft Fabric

Learn how to manage the end-to-end Application Lifecycle Management (ALM) in Microsoft Fabric using Source Control Integration (Git), Fabric Deployment Pipelines, and Fabric REST APIs for CI/CD automation.

## Learning Objectives
In this module, you learn how to:
- Work with **Source Control Integration** (Git) in Fabric using Azure DevOps and GitHub.
- Configure and use **Fabric Deployment Pipelines** for multi-stage (Dev -> Test -> Prod) release management.
- Automate deployment workflows using **Fabric REST APIs**.

---

## 1. Introduction

* **Lifecycle Management in Fabric:** Provides structured mechanisms to incrementally release, integrate, and test changes across different Fabric workspaces (Development, Test, Production).
* **Two Core Pillars of Fabric CI/CD:**
  1. **Source Control Integration (Git):** Connects Fabric workspaces directly to Azure DevOps Repos or GitHub repositories for version control, branching, and PR code reviews.
  2. **Deployment Pipelines:** A visual tool within Fabric to promote content through Development, Test, and Production stages with parameter rules.
* **Automation:** Leveraging Fabric REST APIs for automated build and release pipelines (CI/CD).

---

## 2. Understand Continuous Integration & Continuous Delivery (CI/CD)

CI/CD establishes structured practices for multi-developer collaboration, code versioning, automated testing, and safe release deployment.

### 🔄 CI/CD Fundamentals
* **Continuous Integration (CI):** Developers frequently commit code changes to shared branches. Automated processes build, test, and validate changes early to detect conflicts before merging.
* **Continuous Delivery (CD):** Automatically prepares merged code in staging/testing environments for rapid deployment.
* **Continuous Deployment:** Fully automated deployment directly to Production once all automated tests pass.

### 🛠️ The 3 Tools of Fabric ALM (EXAM HIGH YIELD)
Fabric splits Application Lifecycle Management (ALM) across three complementary tools:

| Tool | Role in Fabric ALM | Primary Mechanism |
| :--- | :--- | :--- |
| **Git Integration** | **Integration (CI)** | Connects workspace to Azure DevOps or GitHub for version control, branching, and PR code reviews. |
| **Deployment Pipelines** | **Deployment (CD)** | Visual multi-stage promotion across dedicated workspaces (**Development ➔ Test ➔ Production**). |
| **Fabric REST APIs** | **Automation** | Enables programmatic triggering of deployments, Git syncs, and ALM tasks in CI/CD scripts. |

---

## 3. Implement Version Control & Git Integration

Git integration enables version control, branch management, and collaborative code reviews across Fabric items.

### 🌐 Supported Providers & Scope
* **Supported Git Providers:** **Azure DevOps Repos** and **GitHub**.
* **Configuration Scope:** Configured at the **Workspace Level** (Workspace Settings → Git integration).

### 🔄 Workspace & Git Syncing
* **Git Status Column:** Displays real-time sync state per item (`Conflict`, `Uncommitted`, `In sync`).
* **Source Control Window:**
  * **`Changes` Tab:** Commits local workspace edits up to the remote Git branch.
  * **`Updates` Tab:** Pulls remote Git commits down into the Fabric workspace.

### 🌿 Branching & Isolation Best Practices (EXAM HIGH YIELD)
* **Golden Rule:** **Never develop directly in a shared live workspace.** Workspace edits immediately impact all users.
* **"Branch out to new workspace" Pattern:**
  1. From a shared dev workspace connected to `main`, open Source Control → select **Branch out to new workspace**.
  2. Creates a new **feature branch** in Git AND a **dedicated isolated workspace** linked to that branch.
  3. Developer works safely in isolation, committing changes to their feature branch.
  4. Submit a **Pull Request (PR)** in Azure DevOps/GitHub to merge feature branch into `main`.
  5. Once merged, the shared dev workspace alerts users to **Synchronize** the latest code from `main`.

---

## 4. Implement Deployment Pipelines

**Deployment Pipelines** provide a visual, multi-stage release mechanism to promote Fabric items safely across dedicated workspaces.

### 🏭 Pipeline Stages & Assignment
* **Default Stages:** **Development ➔ Test ➔ Production** (custom stages can also be defined).
* **Workspace Assignment:** Each pipeline stage is assigned to its own **distinct, dedicated Fabric workspace**.
* **Deploying Items:** Copies/clones selected items from source stage to target stage (e.g., promoting a modified pipeline from Dev to Test).

### 🔀 Combining Git Integration & Deployment Pipelines (EXAM HIGH YIELD)
What is the recommended best practice architecture for combining Git with Deployment Pipelines?

* **Recommended Strategy:** **Connect ONLY the Development workspace to Git.**
* **Workflow:**
  1. Developers use **Git integration** in the **Development workspace** for version control, branching, and PR code reviews.
  2. Once merged to `main`, use the **Deployment Pipeline** to promote items from Development to **Test** and **Production** workspaces.
* **Why?** Avoids complex multi-branch Git sync conflicts across Test/Prod workspaces while maintaining strict, automated promotion.

---

## 5. Automate CI/CD Using Fabric REST APIs

Fabric REST APIs allow engineers to programmatically automate Application Lifecycle Management (ALM) inside Azure DevOps Release Pipelines or GitHub Actions.

### 🤖 Fabric CI/CD REST API Capabilities (HIGH YIELD)

1. **Git Integration REST APIs:**
   * **Commit to Git:** Programmatically commits local workspace changes to the connected remote branch.
   * **Update Workspace:** Programmatically pulls and syncs new Git commits into the Fabric workspace.
   * **Get Git Status:** Checks for uncommitted workspace changes or pending incoming Git commits.

2. **Deployment Pipelines REST APIs:**
   * **List Stage Items:** Returns supported items assigned to a specific deployment pipeline stage.
   * **Deploy Stage Content:** Programmatically triggers the promotion/deployment of items from one stage to another (e.g., automated nightly deployment from Dev ➔ Test).

---

## 6. Module Summary

* **Fabric ALM Triad:**
  * **Git Integration:** Source control, branching, PR code reviews, and workspace item tracking (Azure DevOps Repos & GitHub). Configured at Workspace level.
  * **Deployment Pipelines:** Visual multi-stage promotion across dedicated workspaces (**Development ➔ Test ➔ Production**).
  * **Fabric REST APIs:** Programmatic automation for committing to Git, syncing workspace updates, listing stage items, and deploying content across pipeline stages.
* **Best Practice Workflow:** Connect **ONLY the Development workspace to Git**. Developers use Git for branching & PR reviews in Dev, and use **Deployment Pipelines** to promote tested content to Test and Production workspaces (preventing multi-branch Git sync conflicts).
* **"Branch out to new workspace":** Instantly creates a new Git feature branch AND a dedicated isolated workspace linked to that feature branch.
