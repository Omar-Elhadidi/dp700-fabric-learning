# M25: Administer a Microsoft Fabric Environment

Learn how to configure, govern, and manage an enterprise Microsoft Fabric tenant—exploring admin hierarchies, tenant settings, domain structures, admin delegation, user licensing, capacity management, and platform health monitoring.

## Learning Objectives
In this module, you learn how to:
- Describe the **Fabric Admin Hierarchy** and select the appropriate admin role (Global Admin, Fabric Admin, Capacity Admin, Workspace Admin).
- Configure **Tenant Settings**, set up **Domains**, and delegate admin rights securely.
- Manage **User Licenses** (Per-user vs. Capacity-based) and govern content access and sharing.
- Monitor platform usage, track capacity health, and apply governance controls.

---

## 1. Introduction

* **Core Purpose:** Fabric administration provides centralized governance, capacity allocation, security configuration, and feature delegation across an enterprise organization.
* **Key Admin Pillars:**
  1. **Admin Hierarchy:** Microsoft Entra Roles (Global Admin / Fabric Admin) vs. Resource Roles (Capacity Admin, Domain Admin, Workspace Admin).
  2. **Tenant Settings Management:** Controlling tenant-wide feature switches (e.g., Copilot, Git integration, external sharing, OneLake access).
  3. **Domains & Delegation:** Grouping workspaces into logical business domains (Finance, Sales, HR) to delegate domain-specific admin rights.
  4. **Licensing & Governance:** Balancing Free / Pro / Premium Per User (PPU) licenses with F-SKU / P-SKU compute capacities.

---

## 2. Understand the Fabric Admin Model

Fabric administration relies on a 5-level resource hierarchy and a 4-tier admin delegation model.

### 🏛️ The 5-Level Fabric Architecture Hierarchy (HIGH YIELD)
1. **Tenant:** Topmost scope aligned with the organization's **Microsoft Entra ID** directory.
2. **Capacity:** Dedicated pool of compute & storage resources (F-SKUs / P-SKUs).
3. **Domain:** Logical grouping of workspaces representing business units (e.g., Finance, Risk, HR) for domain-level policy enforcement.
4. **Workspace:** Collaboration container holding individual data items.
5. **Item:** Granular data assets (Lakehouse, Warehouse, Notebook, Pipeline, Report).

### 👥 The 4-Tier Fabric Admin Delegation Model (CRITICAL EXAM MUST-KNOW)

| Admin Role | Scope | Key Administrative Responsibilities |
| :--- | :--- | :--- |
| **Fabric Admin** | **Entire Tenant** | Controls global tenant settings, manages capacities/domains, assigns admins. *(Listed in Microsoft Entra ID as **Power BI Administrator**).* |
| **Capacity Admin** | **Specific Capacity** | Manages workspaces assigned to that capacity, monitors CU consumption and performance health. |
| **Domain Admin** | **Specific Domain** | Manages workspaces in the domain, delegates domain workspace assignment, enforces domain policies. |
| **Workspace Admin** | **Specific Workspace** | Controls workspace membership, item sharing permissions, and item management. |

### 🛠️ Admin Management Tools
* **Fabric Admin Portal:** Web interface for tenant settings, capacity settings, domain creation, and audit logs.
* **Microsoft 365 Admin Center:** Assigns user licenses (Fabric Free, Power BI Pro, PPU).
* **Microsoft Entra ID:** User identity, security groups, and Entra role assignments (**Power BI Administrator**).
* **Automated Scripting:** Fabric REST APIs and PowerShell cmdlets.

---

## 3. Configure Tenant Settings & Delegate Admin Rights

Administering Fabric involves configuring global tenant policies, creating domain governance boundaries, delegating domain settings, and allocating workspaces to compute capacities.

### ⚙️ Tenant Settings & Propagation
* **Master Switch:** `"Users can create Fabric items"` controls whether non-admin users can create Fabric items (lakehouses, warehouses, etc.) tenant-wide.
* **Propagation Delay:** Changes to tenant settings can take **up to 15 minutes** to propagate across the tenant.
* ⚠️ **Governance UI vs Data Security (EXAM HIGH YIELD):** Tenant settings control **UI feature availability** (e.g., hiding the Export to Excel button), NOT underlying data security! Users with direct query access can still extract data via T-SQL or REST APIs. Data security requires RLS, CLS, and Item Permissions.

### 🏢 Domains (Governance Boundaries) (CRITICAL EXAM GOTCHA)
* **What Domains Are:** Logical groupings of workspaces that reflect organizational departments (Finance, Risk, HR).
* ‼️ **Domain Access Myth:** Domain assignment **does NOT restrict item access or grant permissions**. Workspace roles and item permissions still govern data access. **Domains organize governance policies, NOT data access.**
* **Domain Assignment Rules:** Workspaces can be assigned to a domain manually, by workspace name pattern matching, or automatically by **Workspace Admin**.

### 🤝 Delegated Settings to Domain Admins
Fabric Admins can delegate specific tenant settings to Domain Admins (viewable under **Delegated settings**):
* **Certification:** Allows domain admins to designate their own departmental Data Stewards to certify semantic models.
* **Default Sensitivity Labels:** Allows domain admins to enforce domain-specific Purview labels (e.g., Finance defaults to "Confidential", Marketing defaults to "Public").

### ⚡ Assigning Workspaces to Capacities
* Workspaces assigned to an F-SKU or P-SKU capacity run on dedicated compute resources.
* **Architecture Best Practice:** Keep **Production workspaces** on dedicated F/P capacity, and place **Development workspaces** on separate test/trial capacities so experimental dev queries don't throttle production CUs.

---

## 4. Manage User Access & Licensing

Fabric combines **Capacity Licenses** (shared F/P compute) with **Per-User Licenses** (individual creation/viewing rights).

### 💳 Per-User License Types
1. **Fabric Free:** Auto-assigned on first sign-in. Enables creating and sharing non-Power BI Fabric items (lakehouses, warehouses, notebooks, pipelines) on F-capacities.
2. **Power BI Pro:** Required for creating, publishing, and sharing Power BI reports/dashboards in shared workspaces.
3. **Power BI Premium Per User (PPU):** Grants Power BI PPU features per user. *(Does NOT provide F-capacity for non-Power BI items).*

### 🎯 The F64 Licensing Threshold Rule (CRITICAL EXAM MUST-KNOW)

| Capacity Size | Non-Power BI Creator | Power BI Report Creator | Power BI Report Viewer |
| :--- | :---: | :---: | :---: |
| **F64 or Larger** *(F64+, P1+)* | Free License | Pro / PPU | **Free License** *(No Pro required!)* |
| **Smaller than F64** *(F2 to F32)* | Free License | Pro / PPU | **Pro or PPU License Required** |

### 🛠️ License Management & Distribution Patterns
* **Where Licenses are Assigned:** Managed in the **Microsoft 365 Admin Center** (Billing ➔ Licenses) using **Entra ID Group-based licensing** (not in Fabric Admin Portal).
* **Workspace Apps vs. Workspace Roles:**
  * **Workspace Apps:** Recommended for large consumer audiences. Provides clean, read-only access to published reports while hiding work-in-progress.
  * **Workspace Roles:** Best for active developer/analyst collaboration. Exposes all workspace items including work-in-progress.
* **External Sharing (B2B):** Controlled via tenant settings; external recipients require appropriate licensing or Entra ID B2B guest access.

---

## 5. Monitor & Govern Your Fabric Environment

Platform governance requires a combination of operational job troubleshooting, strategic usage analysis, capacity scaling, audit logging, and content endorsement.

### 📊 Strategic Adoption vs. Operational Monitoring (EXAM HIGH YIELD)
* **Monitoring Hub:** Operational tool to track live/recent job statuses (`Failed`, `In Progress`, `Succeeded`) and troubleshoot specific errors.
* **Admin Monitoring Workspace:** Strategic workspace containing the **Feature Usage and Adoption Report**. Enables Fabric Admins to analyze long-term feature adoption trends, active users, and platform usage across departments.

### ⚡ Capacity Management: Scaling & Pausing
* **Scale Up / Scale Down:** Dynamically increase or decrease capacity SKU size (e.g., scale F64 ➔ F128 during month-end closing, then scale down) to handle temporary compute spikes without permanent licensing costs.
* **Pause Capacity:** Non-production capacities (Dev/Test) can be **paused** when idle (e.g., overnight/weekends) to halt billing. ⚠️ *Pausing renders all workspace items on that capacity temporarily unavailable.*

### 🛡️ Governance, Audit Logs & Endorsements
* **Audit Logs:** Captured in Admin Portal / Microsoft Purview to track user actions (e.g., who exported data to Excel, modified permissions, or shared content externally).
* **Item Endorsements:**
  * **Promoted:** Applied by workspace Contributors/Admins to mark good-quality content.
  * **Certified:** Formal seal of trust applied **only by designated certifiers** (managed via the *Certification* tenant setting).
* **OneLake Catalog (Govern Tab):** Provides Fabric Admins with a governance snapshot of tenant-wide sensitivity label coverage and endorsement metrics.

---

## 6. Module Summary

* **Admin Hierarchy & Roles:** 5-level hierarchy (Tenant ➔ Capacity ➔ Domain ➔ Workspace ➔ Item). 4-tier admin delegation (Fabric Admin [Power BI Admin in Entra ID], Capacity Admin, Domain Admin, Workspace Admin).
* **Tenant Settings & Domains:** Tenant settings control UI feature switches (15-min propagation delay). Domains organize governance boundaries and delegated policies—they do **NOT** restrict data access.
* **Licensing & F64 Threshold:** Capacity (F/P SKUs) + Per-User (Free, Pro, PPU). On **F64+ capacities**, report viewers need only a **Free license**. On **< F64 capacities**, viewers require **Pro or PPU**. Licenses assigned in Microsoft 365 Admin Center.
* **Monitoring & Governance:** Monitoring Hub (operational job status), Admin Monitoring Workspace (strategic Feature Usage report), Capacity Metrics App (CU health, scale up/down, pause), Audit logs, Promoted vs. Certified endorsements.

---
