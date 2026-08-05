# M26: Secure Data Access in Microsoft Fabric

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

*(Waiting for next unit)*
