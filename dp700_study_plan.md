# DP-700 Study Plan — 45 Days to Exam

**Exam:** August 11, 2026 · 10:30 AM · GEF Alexandria  
**Starting level:** Basic SQL, Basic Python  
**Daily commitment:** 2-3 hours (weekdays), 4-5 hours (weekends)  
**Total estimated study hours:** ~130 hours

---

## Primary Study Resource

**Microsoft Learn — DP-700 Learning Path (FREE):**  
[https://learn.microsoft.com/en-us/credentials/certifications/fabric-data-engineer-associate/](https://learn.microsoft.com/en-us/credentials/certifications/fabric-data-engineer-associate/)

This page links to ALL the official modules. Work through them in the order below.

**Practice Assessment (FREE):**  
Microsoft provides a free practice test on the same page. Take it at the end of Week 4 and again in Week 6.

---

## Week 1: June 27 – July 3 — Fabric Foundations

**Goal:** Understand what Microsoft Fabric IS, how it's different from normal databases, and how all the pieces connect.

| Day | Focus | Time |
|---|---|---|
| Fri-Sat | Read the "Get started with Microsoft Fabric" module on Microsoft Learn | 3-4h |
| Sun | Read "Introduction to Lakehouses" and "Introduction to Data Warehouses in Fabric" | 4-5h |
| Mon-Thu | Hands-on: Create a free Fabric trial workspace. Create a Lakehouse. Upload a small CSV. Query it with SQL | 2-3h/day |

**By end of week you should know:**
- What OneLake, Lakehouse, and Warehouse are (and the difference)
- How to navigate the Fabric portal
- What the Medallion Architecture means (Bronze/Silver/Gold)

---

## Week 2: July 4 – July 10 — Data Ingestion (Pipelines & Dataflows)

**Goal:** Learn how data gets INTO Fabric.

| Day | Focus | Time |
|---|---|---|
| Fri-Sat | Read "Ingest data with Data Factory pipelines" and "Ingest data with Dataflows Gen2" modules | 3-4h |
| Sun | Hands-on: Build a simple pipeline that copies data from a public source into your Lakehouse | 4-5h |
| Mon-Thu | Read "Use Spark notebooks" module. Write a basic PySpark notebook that reads and cleans data | 2-3h/day |

**By end of week you should know:**
- How to build a Data Factory pipeline (copy activity)
- The difference between Pipelines and Dataflows Gen2
- How to write basic PySpark in a Fabric notebook

---

## Week 3: July 11 – July 17 — Data Transformation & Modeling

**Goal:** Learn how data gets TRANSFORMED inside Fabric.

| Day | Focus | Time |
|---|---|---|
| Fri-Sat | Read "Transform data with Spark" and "Use Delta Lake tables" modules | 3-4h |
| Sun | Hands-on: Transform your Week 2 data through Bronze → Silver → Gold layers | 4-5h |
| Mon-Thu | Read "Design a data warehouse" and "Load data into a warehouse" modules. Practice writing SQL queries in the warehouse | 2-3h/day |

**By end of week you should know:**
- Delta Lake tables and why they matter
- How to write SQL transformations in Fabric warehouses
- Star schema basics (fact tables vs. dimension tables)

---

## Week 4: July 18 – July 24 — Security, Monitoring & Optimization

**Goal:** Cover the exam topics most people skip (and lose points on).

| Day | Focus | Time |
|---|---|---|
| Fri-Sat | Read "Implement security in Fabric" and "Administer Fabric" modules | 3-4h |
| Sun | Read "Monitor Fabric" and "Optimize performance" modules | 4-5h |
| Mon-Wed | Review all modules completed so far. Fill gaps in notes | 2-3h/day |
| Thu | **Take the FREE Microsoft Practice Assessment for the first time.** Don't study beforehand — treat it as a diagnostic | 2h |

**By end of week you should know:**
- Workspace roles and row-level security (RLS)
- How to monitor pipeline runs and query performance
- Your weak areas (from the practice test results)

---

## Week 5: July 25 – July 31 — Weak Spot Drilling

**Goal:** Attack whatever the practice test exposed as weak.

| Day | Focus | Time |
|---|---|---|
| Fri-Sun | Re-study the modules where you scored lowest on the practice assessment | 4-5h/day |
| Mon-Thu | Hands-on practice in your Fabric trial: rebuild a complete pipeline end-to-end (ingest → transform → warehouse → query) | 2-3h/day |

**By end of week you should be able to:**
- Build a complete data pipeline in Fabric from memory
- Explain the difference between Lakehouse, Warehouse, and SQL Analytics Endpoint
- Answer security and monitoring questions without guessing

---

## Week 6: August 1 – August 10 — Final Review & Mock Exams

**Goal:** Simulate exam conditions. Lock in confidence.

| Day | Focus | Time |
|---|---|---|
| Fri | **Take the practice assessment again.** Target: 80%+ | 2h |
| Sat-Sun | Review any remaining weak spots. Re-read notes | 4h/day |
| Mon-Wed | Light review only. Read your own notes. Don't cram new material | 1-2h/day |
| Thu (Aug 7) | Take the practice assessment one final time | 2h |
| Fri-Sun | **REST.** Light review only. No heavy studying. Sleep well | 1h max |
| **Mon Aug 11** | **EXAM DAY — 10:30 AM — GEF Alexandria** | — |

---

## Daily Non-Negotiables (alongside studying)

These take 20 minutes total. Don't skip them.

- [ ] **2 Upwork proposals** (15 min)
- [ ] **1 GitHub commit** (push your study notes or Fabric screenshots — keeps the graph green)

---

## Exam Day Checklist

- [ ] Bring government-issued photo ID (national ID or passport)
- [ ] Arrive 30 minutes early (10:00 AM)
- [ ] No phones, notes, or smartwatches allowed inside
- [ ] The exam IS open-book: you can access Microsoft Learn during the exam
- [ ] You need **700/1000 to pass** (70%)
- [ ] Results are immediate — you'll know before you leave

---

## Key Reminders

> [!IMPORTANT]
> **The exam allows you to use Microsoft Learn during the test.** This means you don't need to memorize every detail — you need to understand concepts well enough to know WHERE to look. Focus on understanding, not memorization.

> [!WARNING]  
> **The Fabric free trial lasts 60 days.** Sign up NOW so it doesn't expire before your exam. You need hands-on practice, not just reading.

> [!TIP]
> **If you feel overwhelmed:** Remember that thousands of people pass this exam every month with similar backgrounds. The pass rate for well-prepared candidates is high. You have 45 days and a structured plan. That is enough.
