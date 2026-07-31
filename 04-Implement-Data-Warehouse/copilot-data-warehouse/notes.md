# Get Started with Copilot in Microsoft Fabric for Data Warehouse

Accelerate warehouse analytics with Copilot in Fabric—use AI chat, inline completions, and quick actions to generate, refine, explain, and fix T-SQL while applying best-practice prompting and governance.

## Learning Objectives
In this module, you learn how to:
- Apply **Copilot** across Fabric Data Warehouse experiences to accelerate querying, authoring, and troubleshooting.
- Generate, refine, explain, and fix T-SQL using **Chat**, **Inline code completions**, and **Quick actions**.
- Incorporate effective **prompting**, **naming**, and **schema design** practices that improve Copilot accuracy.
- Use Copilot capabilities responsibly to enhance productivity while maintaining data quality and governance.

---

## 1. Introduction

* **Core Purpose:** Copilot for Fabric Data Warehouse integrates generative AI directly into the SQL query editor to assist data engineers and analysts with T-SQL authoring, query optimization, and troubleshooting.
* **Key Interaction Modes:**
  1. **Inline Code Completions:** Real-time T-SQL code suggestions as you type.
  2. **Copilot Chat Pane:** Interactive natural language conversation to generate, translate, and refine SQL queries.
  3. **Quick Actions:** One-click ribbon tools (**Explain query**, **Fix query errors**).
* **Prerequisites & SKU Requirement:** Copilot requires an **F2 or P1 capacity or higher** (not available on trial SKUs).

---

## 2. Use Copilot Code Completion

Copilot provides schema-aware, real-time T-SQL code suggestions directly inside the SQL Query Editor.

### ⚙️ Setup & Enablement
* **Settings Toggle:** Warehouse Settings → Copilot pane → **Show Copilot completions** (status also visible on query editor status bar).
* **SKU Requirement:** **F2 or P SKU minimum** (Trial SKUs NOT supported).

### ⌨️ Interaction & Keyboard Shortcuts (EXAM HIGH YIELD)
Suggestions display as **dimmed ghost text** while typing.

| Action / Shortcut | Function |
| :--- | :--- |
| **`Tab`** | Accept the **entire** ghost text suggestion. |
| **`Ctrl + Right Arrow`** (*`Cmd + Right`*) | Accept suggestion **word-by-word** (partial accept). |
| **Hover Over Suggestion** | Cycle through and preview **alternative** SQL suggestions. |
| **`Esc` / Continue Typing** | Dismiss the suggestion. |

### 💬 Comment-Driven Generation (`--`)
Write single-line SQL comments (`-- prompt text`) to instruct Copilot to generate T-SQL queries directly beneath:

```sql
-- Calculate top 10 customers by total sales amount in 2024
SELECT TOP 10 CustomerID, SUM(SalesAmount) AS TotalSales
FROM dbo.FactSales
WHERE YEAR(OrderDate) = 2024
GROUP BY CustomerID
ORDER BY TotalSales DESC;
```

---

## 3. Use Copilot Chat Pane

The **Copilot Chat Pane** (opened via the ribbon `Copilot` button) provides an interactive, conversational natural language experience for query generation, concept learning, and multi-turn refactoring.

### ⚡ Session Context & Multi-Turn History
* Retains session context across prompts. Follow-up messages build on previous outputs (e.g., generating a query first, then asking to refine or fix it) without needing to re-paste the full query or context.

### 🛠️ Copilot Slash Commands (`/`) (HIGH YIELD FOR EXAM)
Special slash commands executed at the start of a chat prompt to direct Copilot actions:

| Slash Command | Primary Purpose | Example Syntax |
| :--- | :--- | :--- |
| **`/generate-sql`** | Generates a T-SQL statement from a natural language request. | `/generate-sql select top 10 products by sales` |
| **`/explain`** | Explains the active query in the current SQL editor tab line-by-line. | `/explain` |
| **`/fix`** | Fixes syntax/logic errors or refactors the active query tab. | `/fix using CTAS instead of ALTER TABLE` |
| **`/question`** | Answers general architectural or conceptual questions. | `/question what security types are supported in Fabric DW?` |
| **`/help`** | Displays guidance and documentation for using Copilot. | `/help` |

---

## 4. Use Copilot Quick Actions

**Quick Actions** are ribbon toolbar buttons located directly near the **`Run`** button in the SQL Query Editor ribbon.

### 🔍 1. `Explain` (Explain Query)
* **Usage:** Highlight the specific query block (or full script), then click **Explain**.
* **Behavior:** Generates a top-level summary comment block + inline `--` comments next to individual T-SQL statements detailing joins, aggregations, and filter conditions.

### 🔧 2. `Fix` (Fix Query Errors) (EXAM HIGH YIELD)
* **Activation Behavior:** The **`Fix`** button is **grayed out (disabled) by default**. It activates automatically **ONLY AFTER a query run fails with an error**.
* **Context Capture:** Automatically feeds the exact engine error traceback into Copilot—no manual copy-pasting of error messages needed.
* **Output:** Generates corrected T-SQL syntax resolving column references, syntax typos, or invalid join logic.

---

## 5. Copilot Best Practices & Optimization

To maximize Copilot accuracy and reliability when generating T-SQL:

### 📐 Schema & Data Model Best Practices (EXAM HIGH YIELD)
* **Descriptive Naming:** Use business-friendly, expressive table and column names (e.g., `CustomerRegion` vs `CustRgn`). Copilot relies heavily on names to understand schema context.
* **Defined Model Relationships:** Establish explicit Primary Key (PK) / Foreign Key (FK) relationships in the **Model View**. Copilot uses these relationships to generate accurate T-SQL `JOIN` syntax.

### ✍️ Prompt Engineering Guidelines
* **Be Explicit:** Explicitly state the requested columns, aggregations (`SUM`, `AVG`), and filter criteria in your prompt.
* **Use In-Line Comments:** Add `--` comments at the top or inside complex scripts to guide multi-table logic.
* **Language Support:** Prompts **must be in English** (English-to-T-SQL translation).

---

## 6. Module Summary

* **Copilot DW Overview:** AI assistant for T-SQL query generation, completion, explanation, and error fixing integrated into Fabric Data Warehouse. Requires **F2 or P SKU minimum** (trial SKUs not supported).
* **Code Completion:** Inline ghost text suggestions. Press **`Tab`** to accept full suggestion, **`Ctrl + Right Arrow`** to accept word-by-word. Supports `--` comment-driven prompting.
* **Chat Pane & Slash Commands (`/`):** Interactive conversational AI retaining multi-turn session history.
  * `/generate-sql` (generate T-SQL), `/explain` (explain query tab), `/fix` (fix tab errors), `/question` (conceptual answers), `/help` (docs).
* **Quick Actions:** Ribbon buttons near `Run`:
  * **`Explain`:** Generates summary + inline `--` comments explaining code.
  * **`Fix`:** Disabled by default; **activates automatically after query run returns an error**, capturing runtime error context automatically.
* **Best Practices:** Model View PK/FK relationships are critical for accurate `JOIN` generation; use descriptive column/table names; write explicit prompts in **English**.
