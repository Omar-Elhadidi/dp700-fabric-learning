# M20: Secure a Microsoft Fabric Data Warehouse

Learn how to protect sensitive data in a Microsoft Fabric warehouse using Workspace roles, Item permissions, Granular SQL security (GRANT/DENY), Dynamic Data Masking (DDM), Row-Level Security (RLS), and Column-Level Security (CLS).

## Learning Objectives
In this module, you learn how to:
- Identify the multi-layered security model available in a Microsoft Fabric warehouse.
- Describe how **Dynamic Data Masking (DDM)** obscures sensitive column values.
- Explain how **Row-Level Security (RLS)** restricts row access based on user context.
- Describe how **Column-Level Security (CLS)** controls access to specific table columns.
- Explain how **SQL Granular Permissions** (`GRANT`, `DENY`, `REVOKE`) control access to warehouse objects.

---

## 1. Introduction

* **Core Purpose:** Data warehouses centralize enterprise data, making security and governance critical to protect sensitive business information (PII, financial data, operational metrics).
* **Multi-Layered Security Architecture:**
  1. **Workspace Roles & Item Permissions:** Broad access control (Workspace Admin/Member/Contributor/Viewer and Warehouse Read/ReadData/ReadAll).
  2. **Object-Level Permissions:** T-SQL `GRANT`/`DENY` on tables, views, schema, and stored procedures.
  3. **Row-Level Security (RLS):** Filter rows dynamically based on user identity (`USER_NAME()`).
  4. **Column-Level Security (CLS):** Restrict access to specific sensitive columns (`GRANT SELECT ON Table(Col1, Col2)`).
  5. **Dynamic Data Masking (DDM):** Obfuscate column data (e.g., credit card numbers, emails) without altering underlying storage.

---

## 2. Explore Dynamic Data Masking (DDM)

**Dynamic Data Masking (DDM)** obscures sensitive column values (PII, credit cards, emails) in query results **at runtime** without modifying physical data stored in OneLake.

### 🎭 The 4 DDM Masking Functions (HIGH YIELD FOR EXAM)
| Masking Function | Result Behavior | T-SQL Example Syntax |
| :--- | :--- | :--- |
| **`default()`** | Replaces value based on data type (`0` for numbers, `XXXX` for strings, `1900-01-01` for dates). | `ADD MASKED WITH (FUNCTION = 'default()')` |
| **`email()`** | Exposes 1st letter + `*****@domain.com` (e.g., `j*****@contoso.com`). | `ADD MASKED WITH (FUNCTION = 'email()')` |
| **`partial(prefix, padding, suffix)`** | Exposes specified prefix/suffix chars with custom padding string. | `ADD MASKED WITH (FUNCTION = 'partial(0,"XXXX-XXXX-XXXX-",4)')` *(shows last 4 digits)* |
| **`random(low, high)`** | Replaces numeric/binary values with a random number within range. | `ADD MASKED WITH (FUNCTION = 'random(1, 100)')` |

### 🛠️ T-SQL Configuration & Permissions

```sql
-- Apply masking functions to columns
ALTER TABLE dbo.Customers ALTER COLUMN Email ADD MASKED WITH (FUNCTION = 'email()');
ALTER TABLE dbo.Customers ALTER COLUMN PhoneNumber ADD MASKED WITH (FUNCTION = 'partial(0,"XXX-XXX-",4)');

-- Remove a mask
ALTER TABLE dbo.Customers ALTER COLUMN Email DROP MASKED;

-- Grant UNMASK permission to a specific user/role (to view real data)
GRANT UNMASK ON dbo.Customers TO [user@contoso.com];

-- Grant permission to manage masking rules without granting Admin rights
GRANT ALTER ANY MASK TO [engineer@contoso.com];
```

### ⚠️ Exam Security Considerations
* **Unmask Defaults:** Workspace Admins, Members, and Contributors implicitly hold `CONTROL` permission and **always see unmasked data**.
* **Inference Risk:** DDM conceals output values, but **does not restrict filtering or computation**. A user can infer masked values using conditional queries (e.g., divide-by-zero side-channel attacks). Combine DDM with RLS/CLS!

---

## 3. Implement Row-Level Security (RLS)

**Row-Level Security (RLS)** dynamically restricts row visibility based on user context (`USER_NAME()`) at the database engine level for `SELECT`, `UPDATE`, and `DELETE` queries. *(Note: `INSERT` operations are NOT restricted by filter predicates).*

### 🛠️ The 2 Components of RLS (EXAM HIGH YIELD)
1. **Filter Predicate:** An Inline Table-Valued Function (iTVF) defined `WITH SCHEMABINDING` returning `1` for allowed rows.
2. **Security Policy:** Binds the TVF filter predicate to target table(s) with `STATE = ON`.

```sql
-- 1. Create dedicated security schema
CREATE SCHEMA Sec;
GO

-- 2. Define the Inline Table-Valued Function (TVF)
CREATE FUNCTION Sec.tvf_SecurityPredicateBySalesPerson(@SalesPerson AS NVARCHAR(50))
    RETURNS TABLE
WITH SCHEMABINDING
AS
    RETURN SELECT 1 AS result
           WHERE @SalesPerson = USER_NAME()
              OR USER_NAME() = 'salesadmin@contoso.com';
GO

-- 3. Bind the security policy to the table
CREATE SECURITY POLICY Sec.SalesPolicy
ADD FILTER PREDICATE Sec.tvf_SecurityPredicateBySalesPerson(SalesPerson) ON dbo.Sales
WITH (STATE = ON);
GO

-- Disable policy temporarily
ALTER SECURITY POLICY Sec.SalesPolicy WITH (STATE = OFF);
```

### ⚠️ Workspace Admin & Role Behaviors (CRITICAL EXAM GOTCHA)
* **RLS Applies to EVERYONE:** Unlike table permissions, RLS predicates apply to **ALL users—including Workspace Admins, Members, and Contributors!**
* **Admin Exemption Rule:** If your TVF predicate does NOT explicitly include an admin check (`OR USER_NAME() = 'admin@contoso.com'`), workspace admins will be filtered out and won't see those rows when querying!
* **Management Rights:** Administering security policies requires `ALTER ANY SECURITY POLICY` permission.

---

## 4. Implement Column-Level Security (CLS)

**Column-Level Security (CLS)** restricts user access to specific sensitive columns (`MedicalHistory`, `Salary`) without modifying underlying table schemas.

### 🛠️ T-SQL Configuration Syntax
CLS is implemented using standard T-SQL `GRANT` and `DENY` statements with column-level specifications:

```sql
-- 1. Create database roles
CREATE ROLE Doctor AUTHORIZATION dbo;
CREATE ROLE Receptionist AUTHORIZATION dbo;

-- 2. Grant table-level SELECT access
GRANT SELECT ON dbo.Patients TO Doctor;
GRANT SELECT ON dbo.Patients TO Receptionist;

-- 3. Explicitly DENY SELECT on sensitive column for restricted roles
DENY SELECT ON dbo.Patients (MedicalHistory) TO Receptionist;
```

### ⚡ Power BI Direct Lake Fallback Behavior (CRITICAL EXAM GOTCHA)
* **Direct Lake Fallback to DirectQuery:** When Power BI connects to a Fabric Warehouse table that has **Column-Level Security (CLS)** or **Row-Level Security (RLS)** applied, Power BI queries automatically **fall back from Direct Lake mode to DirectQuery mode**.
* **Security & Performance Trade-off:** Security rules are strictly enforced at all times, but performance operates under **DirectQuery** mechanics rather than OneLake Direct Lake in-memory speed.

---

## 5. Configure SQL Granular Permissions (`GRANT`, `DENY`, `REVOKE`)

Granular SQL permissions provide fine-grained object-level access control over tables, views, schemas, functions, and stored procedures.

### ⚖️ Permission Precedence Rule (EXAM HIGH YIELD)
* **`DENY` ALWAYS OVERRIDES `GRANT`:** If a user is granted `SELECT` via one role but denied `SELECT` via another role or direct assignment, the **`DENY` statement wins**.

### 📋 Object Permission Types
| Category | Permission | What It Allows |
| :--- | :--- | :--- |
| **Tables & Views** | `SELECT`, `INSERT`, `UPDATE`, `DELETE` | Standard DML data read and manipulation. |
| **Procedures & Functions** | `EXECUTE` | Run the stored procedure or user-defined function. |
| **Procedures & Functions** | `ALTER` | Modify the underlying T-SQL definition. |
| **All Objects** | `CONTROL` | Full ownership-level control over the object. |

### 🔒 Security Pattern: Stored Procedure Encapsulation
A common enterprise security pattern is to **grant `EXECUTE` on specific stored procedures** while **denying direct `SELECT`/`INSERT` on underlying base tables**:

```sql
-- Allow user to execute the sales report procedure
GRANT EXECUTE ON dbo.usp_GetSalesData TO [bob@contoso.com];

-- Prevent direct querying of underlying sales tables
DENY SELECT ON dbo.Sales TO [bob@contoso.com];
```

---

## 6. Module Summary

* **Dynamic Data Masking (DDM):** Obscures column values at runtime (`default()`, `email()`, `partial()`, `random()`) without altering underlying OneLake Parquet data. Privileged users need `GRANT UNMASK`.
* **Row-Level Security (RLS):** Restricts row access dynamically via an Inline TVF + Security Policy. Applies to **ALL users (including Workspace Admins)**—TVF must explicitly include admin exceptions.
* **Column-Level Security (CLS):** Restricts column access using `DENY SELECT ON Table(Col) TO Role`. **Power BI Direct Lake mode automatically falls back to DirectQuery mode** when CLS or RLS is present.
* **SQL Granular Permissions:** Fine-grained object control (`GRANT`, `DENY`, `REVOKE`, `EXECUTE`). `DENY` always overrides `GRANT`. Enforce principle of least privilege (e.g. `EXECUTE` on procedures instead of direct table `SELECT`).
