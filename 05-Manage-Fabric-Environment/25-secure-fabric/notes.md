# M25: Secure Data Access in Microsoft Fabric

Learn how to implement multi-layered security in Microsoft Fabric using Workspace Roles, Item Permissions, SQL Granular Permissions, and OneLake Security Roles (data access control at the file/folder and table level).

## Learning Objectives
In this module, you learn how to:
- Describe the **Microsoft Fabric Security Model** and its distinct access control layers.
- Configure **Workspace Roles** (Admin, Member, Contributor, Viewer) and **Item Permissions** (Read, ReadData, ReadAll, Build, Share).
- Apply **Granular Permissions** using T-SQL (in Data Warehouse / SQL Analytics Endpoint) and **OneLake Security Roles** (in Lakehouses).

---

## 1. Introduction

* **Core Challenge:** Enforcing the Principle of Least Privilege across enterprise data assets (lakehouses, warehouses, reports) to ensure team members have sufficient access without over-privileging.
* **Security Layers in Fabric:**
  1. **Workspace-level:** Coarse-grained role-based access (Admin, Member, Contributor, Viewer).
  2. **Item-level:** Specific item permissions (Read, ReadData, ReadAll, Build, Share, Reshare).
  3. **Data-level (Engines):** Fine-grained T-SQL object/row/column permissions (in DW/SQL Endpoint) and **OneLake Security Roles** (folder/file-level access control in Lakehouse).

---

## 2. Understand the Fabric Security Model

Fabric enforces access control through a 3-step sequential evaluation pipeline:
1. **Microsoft Entra ID Authentication** ➔ Verifies identity.
2. **Fabric Tenant/Workspace Access** ➔ Checks workspace/portal authorization.
3. **Data Security Evaluation** ➔ Evaluates engine & data object permissions.

### 🛡️ The 4 Data Security Layers (HIGH YIELD FOR EXAM)

| Security Layer | Granularity Scope | Best Used For |
| :--- | :--- | :--- |
| **1. Workspace Roles** | **All items** in a workspace. | Broad team collaboration across multiple items (**Admin, Member, Contributor, Viewer**). |
| **2. Item Permissions** | **Single item** (e.g., one Lakehouse/Warehouse). | Granting external users access to a specific item without giving workspace access. |
| **3. Compute Permissions** | Specific query engine (SQL, T-SQL DML). | T-SQL object-level `GRANT`/`DENY`, `ReadData`, `ReadAll`. |
| **4. OneLake Security** | **Folder/Table level** inside an item. | Granular data access enforced **consistently across ALL engines** (Spark, SQL, Direct Lake, APIs). |

---

## 3. Configure Workspace & Item Permissions

Workspaces use **Workspace Roles** for broad access across all items, and **Item Permissions** for fine-grained sharing of single items.

### 🏢 The 4 Workspace Roles & Capabilities (EXAM HIGH YIELD)
* **Admin:** Full control + can manage permissions and assign roles.
* **Member:** Create, modify, and **share** content. (Cannot manage workspace permissions).
* **Contributor:** Create and modify content. (**Cannot share** or manage permissions).
* **Viewer:** Can view items in the workspace list.
  * ⚠️ **VIEWER DATA GOTCHA:** Viewers can see Fabric items in the UI, but have **NO access to underlying data stored in OneLake by default!** Access to raw data must be explicitly granted via OneLake Security Roles or Item Permissions.

### 🔗 Lakehouse Item Permissions (Sharing Options)

When sharing a specific Lakehouse (`...` → **Manage permissions**):

| Permission Name | Engine Scope | What It Allows |
| :--- | :--- | :--- |
| **`Read`** *(Base)* | Metadata only | View item metadata and reports. **No access to underlying data!** |
| **`ReadData`** | SQL Analytics Endpoint | Read all tables via T-SQL queries. |
| **`ReadAll`** | PySpark & OneLake APIs | Read all files/tables via Spark & OneLake. *(Automatically adds user to `DefaultReader` OneLake role).* |
| **`Build`** | Semantic Model | Create new Power BI reports on the default semantic model. |

---

## 4. Apply Granular Permissions

When item-level access is too broad, use **T-SQL Granular Permissions** (for SQL Endpoint) or **OneLake Security Roles** (for Lakehouse files/tables).

### 🛠️ OneLake Security Roles (RBAC) (HIGH YIELD FOR EXAM)

OneLake Security Roles define fine-grained access to specific tables/folders and apply **consistently across Spark, SQL Analytics Endpoint, Direct Lake, and OneLake APIs**.

* **Target Audience:** Used to grant specific table/folder access to **Viewers** or users with item-level `Read` permission (since Admins, Members, and Contributors already have full access).
* **Role Creation Rights:** Only workspace **Admins** and **Members** can create or modify OneLake security roles.
* **4 Role Components:**
  1. **Data:** Specific tables or folders.
  2. **Permission:** `Read` or `ReadWrite`.
  3. **Members:** Assigned users or Azure Entra security groups.
  4. **Constraints:** Optional row or column filters.

### ⚠️ The `DefaultReader` Role Trap (CRITICAL EXAM GOTCHA)
* Every Lakehouse has a built-in **`DefaultReader`** role granting access to all data.
* Granting `ReadAll` (`Read all Apache Spark`) automatically adds the recipient to `DefaultReader`.
* ‼️ **EXAM MUST-KNOW:** When restricting a user to a custom OneLake security role with specific table/folder access, you **MUST REMOVE THEM FROM `DefaultReader`**—otherwise, their `DefaultReader` membership will override the custom restriction and expose all data!

---

## 5. Module Summary

* **Layered Security Hierarchy:**
  1. **Workspace Roles (Admin, Member, Contributor, Viewer):** Apply to ALL items. Note: Viewers have **no OneLake data access by default**.
  2. **Item Permissions (Share):** `Read` (metadata only), `ReadData` (SQL Endpoint), `ReadAll` (Spark/OneLake APIs), `Build` (Semantic Model).
  3. **T-SQL Granular Permissions:** DCL (`GRANT`/`DENY`/`REVOKE`), RLS, CLS, and DDM enforced on SQL Analytics Endpoint.
  4. **OneLake Security Roles:** Item-internal RBAC enforcing table/folder access **consistently across Spark, SQL, Direct Lake, and OneLake APIs**. Must remove users from **`DefaultReader`** when applying custom restricted roles!

---
