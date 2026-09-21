![Bank Operations TAT & Bottleneck Analysis Dashboard](https://raw.githubusercontent.com/tranquochung09062001-dev/Banking-Operation/main/Dashboard_Cover.png)

# 🏦 Bank Operations TAT Analysis Dashboard | Process Bottleneck & Root Cause Analysis | Python & Power BI

_Analyzing turnaround time (TAT) across 7 core banking operations units to identify process bottlenecks, data quality risks, and root causes limiting operational efficiency – Bank Data Operations | Python & Power BI_

**+ Business question:** Across the bank's 7 core operational units, which process steps are the primary bottlenecks driving turnaround time (TAT) — and are these bottlenecks caused by workflow design, resourcing, data quality issues, or cross-department dependencies?

**+ Domain:** Banking Operations / Process Mining & Analytics

---

## 📑 Table of Contents
1. [📌 Background & Overview](#-background--overview)
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)
3. [🧠 Design Thinking Process](#-design-thinking-process)
4. [🧹 Data Cleaning & Standardization](#-data-cleaning--standardization)
5. [⚒️ Main Process](#️-main-process)
6. [📊 Key Insights & Visualizations](#-key-insights--visualizations)
7. [🛠️ Skills & Tools Applied](#️-skills--tools-applied)
8. [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)

---

## 📌 Background & Overview

### 📖 What is this project about?

This project analyzes **process turnaround time (TAT) across 7 core banking operations units** using **Python** for data cleaning/standardization and **Power BI** for interactive root-cause dashboards. The objective is to:

✔️ Measure end-to-end and per-step TAT for every operational unit (Counter, Vault, Domestic Transfer, International Transfer, Card Processing, Hotline, Process Improvement).

✔️ Identify the specific step(s) inside each process that consume the largest share of total processing time — the true bottleneck, not just the slowest-looking step.

✔️ Separate **workflow-design bottlenecks** (too many steps, no approval tiering, one step handling multiple business lines) from **cross-department bottlenecks** (waiting on external units/documents) and **data-quality issues** (Start > End timestamp errors).

✔️ Flag and quantify data quality problems (e.g. negative/invalid TAT from timestamp entry errors) so they are corrected before being used for capacity decisions.

✔️ Deliver a dashboard structured so a Head-of-Division-level stakeholder can understand the top bottleneck and its root cause within 30 seconds, without needing to query raw data.

The analysis is structured into **7 unit-level views**, each following the same pattern: *volume & TAT overview → step-level breakdown → root cause*.

### 👤 Who is this project for?

✔️ Data analysts & business/process analysts

✔️ Operations managers of each unit (Counter, Vault, Transfer, Card, Hotline, Process Improvement)

✔️ Heads of Division and senior leadership reviewing operational efficiency

✔️ Anyone studying real-world messy operational data cleaning (duplicate rates up to 51%, mixed units, negative values, capitalization inconsistencies)

---

## 📂 Dataset Description & Data Structure

### 📌 Data Source
- **Source:** Raw CSV exports from 7 bank operational units (internal process-timing logs, Vietnamese banking operations)
- **Format:** `.csv`, UTF-8, exported per unit (some units split across multiple files)
- **Scope:** 7 units, **122,357 raw records** → **120,937 records** after cleaning

| # | Unit | Description | Raw Records | Clean Records | Duplicates Removed |
|---|---|---|---|---|---|
| 1 | `UNIT_01_COUNTER` | Teller transactions (deposit/withdrawal/transfer/change) | 5,356 | 5,352 | 4 (0.07%) |
| 2 | `UNIT_02_VAULT` | Cash vault & document custody transfer | 350 | 306 | 44 (12.57%) |
| 3 | `UNIT_03_TRF_DOM` | Domestic inter-branch transfer orders | 110,160 | 109,873 | 287 (0.26%) |
| 4 | `UNIT_04_TRF_INT` | International transfer (2 sub-processes) | 666 | 666 | 0 (0.00%) |
| 5 | `UNIT_05_CARD_PROCESS` | Card issuance/processing (4 files consolidated) | 2,110 | 1,025 | 1,085 (51.42%) |
| 6 | `UNIT_06_HOTLINE` | Multi-channel customer support hotline | 3,695 | 3,695 | 0 (0.00%) |
| 7 | `UNIT_07_IMPROV` | Process-improvement proposal intake & approval | 20 | 20 | 0 (0.00%) |

### 📊 Data Structure & Relationships

Each unit follows a common **fact-table pattern** — one row per process step per transaction:

| Column (typical) | Description |
|---|---|
| `Số Ref` / Order ID | Unique transaction identifier |
| `Nghiệp vụ` | Business line (e.g. deposit, withdrawal, transfer type) |
| `Tên bước` | Process step name (e.g. Tiếp nhận, Kiểm tra, Hạch toán, Thu, Chi) |
| `Tên bước đo lường` | Granular measurement sub-step |
| `Start` / `End` | Step start/end timestamp (`dd/mm/yyyy hh:mm:ss`) |
| `Gián đoạn Start` / `Gián đoạn End` | Interruption start/end timestamp (if the step was paused) |
| `Nguyên nhân gián đoạn` | Recorded interruption reason |
| `Có làm lại?` / rework fields | Whether the step had to be redone, and why |
| `TAT_Total (Giây)` | Total elapsed time for the step (seconds) |
| `TAT_Gián đoạn (Giây)` | Time attributable to interruption |
| `TAT (Giây)` | Net processing time = Total − Interruption |

<img width="700" alt="Data Model" src="https://raw.githubusercontent.com/tranquochung09062001-dev/Banking-Operation/main/2026-09-21_15-46-52.png" />

---

## 🧠 Design Thinking Process

Before building the dashboard, a **stakeholder requirement analysis** was conducted using the Design Thinking framework, so the 7 unit pages are built around actual decisions a stakeholder needs to make — not around whatever fields happened to be in the raw export.

**1️⃣ Empathize**

![Step 1 - Empathize](https://raw.githubusercontent.com/tranquochung09062001-dev/Banking-Operation/main/step1_design_thinking_1.png)

Identified the primary stakeholder — a **Head of Operations Division** who does not query raw data, reviews the report only during monthly operations meetings, and needs to compare bottleneck severity *across* all 7 units, not just drill into one.

**2️⃣ Define**

![Step 2 - Define](https://raw.githubusercontent.com/tranquochung09062001-dev/Banking-Operation/main/step2_design_thinking_1.png)

Problem statement: *"A Head of Operations Division needs a comparable, root-cause-level view of turnaround time across 7 units — to see not just which step is slow, but why (workflow design, cross-department wait, or data error) — so resourcing and process-fix decisions target the real cause."*

**3️⃣ Ideate**

![Step 3 - Ideate](https://raw.githubusercontent.com/tranquochung09062001-dev/Banking-Operation/main/design_thinking_step3_1.png)

Mapped the decision points the dashboard had to support: identify the true bottleneck step, separate workflow-design causes from cross-department dependency causes, flag data-quality risk before the numbers are trusted, rank units by severity, size the right fix (process redesign vs. SLA), and translate each into a concrete next step.

**4️⃣ Prototype & Review**

![Step 4 - Prototype and Review](https://raw.githubusercontent.com/tranquochung09062001-dev/Banking-Operation/main/design_thinking_step4_1.png)

Structured each unit page around the same flow — Overview → Unit Detail → Root Cause → Data Quality — so the report reads in about 30 seconds even as a static screenshot, matching how the stakeholder actually opens it (a monthly review, not a daily habit).

---

## 🧹 Data Cleaning & Standardization

Raw exports required substantial cleaning before analysis. All 7 units went through a common pipeline (Python / pandas), with unit-specific handling layered on top:

**Common steps (all units):**
- Trim leading/trailing whitespace on every text field
- Convert empty strings to proper `NULL`
- Parse all timestamps to a consistent `dd/mm/yyyy hh:mm:ss` format
- Convert numeric TAT fields to float, **preserving negative and zero values as-is** (they carry business meaning — rework, instant processing — and must never be deleted or clipped)
- Remove exact-duplicate rows (all columns identical)

**Unit-specific handling:**
- `UNIT_02_VAULT`: normalized inconsistent capitalization (e.g. "Lưu DOCS" vs "lưu DOCS") before deduplication — responsible for most of its 12.57% duplicate rate
- `UNIT_05_CARD_PROCESS`: consolidated 4 separate process files into one table; the 51.42% duplicate rate here is the single biggest data-quality finding of the whole cleaning phase and is flagged for the source-system team to investigate
- `UNIT_07_IMPROV`: source data recorded processing time in **days**; converted to **minutes** (`× 1440`) to make it comparable with all other units, which report in minutes/seconds

**Result:** 8 clean, deduplicated CSV outputs (UTF-8), ready for the Power BI model — see [`/data/clean`](./data/clean) folder.

---

## ⚒️ Main Process

1️⃣ **Data Cleaning & Preprocessing** (Python / pandas)
- Standardized text, datetime, and numeric types across all 7 units
- Applied unit-specific fixes (capitalization, file consolidation, unit conversion)
- Documented every transformation rule in a standardization guide for reproducibility

2️⃣ **Exploratory & Root-Cause Analysis**
- Ranked process steps within each unit by both **total time share** and **average time per occurrence** — these can point to two different bottleneck definitions and must be reported separately (e.g. Court: *Hạch toán* is the volume bottleneck, *Thu tiền mặt* is the per-transaction-time bottleneck)
- Cross-checked step-level interruption counts against interruption *reason* fields to distinguish workflow-design issues from cross-department waiting time
- Flagged rows with `Start > End` (impossible negative duration from data-entry error) as a data-quality issue requiring source-system correction, separate from genuine negative TAT (rework)

3️⃣ **Power BI Visualization**
- Built one dashboard page per unit (7 pages) plus a cross-unit summary
- DAX measures: median/average TAT per step, % of total time by step, % of total interruption count by step, Sankey flow (business line → step → interruption reason)
- Cross-filtering slicers by Year/Month, Business line, and Customer status (busy/normal/quiet)

---

## 📊 Key Insights & Visualizations

### 1️⃣ Counter (Teller Transactions)

![Counter Dashboard](https://raw.githubusercontent.com/tranquochung09062001-dev/Banking-Operation/main/Counter_Dashboard.png)

📌 **Analysis:**
- **Observation:** 508 transactions in the sampled view, median TAT 328 seconds. **Hạch toán (posting)** is the volume bottleneck (72,462 sec total, 1,459/5,356 occurrences — the single largest total-time step), while **Thu tiền mặt (cash collection)** is the per-transaction bottleneck (135.7 sec/occurrence average, the highest of any step). These two consecutive steps (Thu → Hạch toán) together account for 54–70% of total TAT and 81–95% of total interruptions within the deposit/payment flow, depending on business line.
- **Root cause:** Cash counting is an inherently manual, hard-to-compress task. Posting is overloaded because it serves 5 different business lines (deposit/withdrawal ×2/change) through a single processing step — a **workflow design** issue, not a staffing issue.
- **Recommendation:** Consider splitting the posting step by business-line volume, or adding a dedicated fast-lane for the highest-volume line.

### 2️⃣ Vault (Cash & Document Custody)

_No dashboard screenshot available yet for this unit — add one to `./Vault_Dashboard.png` and re-link it here once the page is exported from Power BI._

📌 **Analysis:**
- **Observation:** 29 transactions (per business scope), median TAT 31 minutes. Three steps are nearly tied at ~19% of total time each. The **HO → Branch** route is consistently the slowest (median 55 min, P90/median ratio of only 1.05 — meaning it's *uniformly* slow, not skewed by outliers).
- **Root cause:** The HO→Branch route passes through 10 steps end-to-end, versus only 6 steps for the internal Role01→Role02 route — a **structural/route-design** bottleneck, not a single underperforming step.
- **Recommendation:** Review whether all 10 steps in the HO→Branch route are necessary, or whether some can be merged/parallelized.

### 3️⃣ Domestic Transfer (TRF_DOM)

![Domestic Transfer Dashboard](https://raw.githubusercontent.com/tranquochung09062001-dev/Banking-Operation/main/TRF_DOM_Dashboard.png)

📌 **Analysis:**
- **Observation:** ~74K transactions, median TAT 20.52 minutes. The **Approval** step accounts for **43.68%** of total processing time across 38,903 transactions — the highest-volume and highest-impact step in the entire dataset.
- **Data quality flag:** 24.29% of rows in the "Order Processing" step show `Start > End` — a data-entry error affecting an estimated 8,000+ of 33,524 transactions. This must be corrected before using this step's TAT for capacity planning.
- **Root cause:** Manual approval volume (~39,000 transactions) is concentrated into a single step.
- **Recommendation:** Investigate approval-step automation or tiered auto-approval for low-risk transactions; fix the source-system timestamp logging issue in parallel.

### 4️⃣ International Transfer (TRF_INT)

![International Transfer Dashboard](https://raw.githubusercontent.com/tranquochung09062001-dev/Banking-Operation/main/TRF_INT_Dashboard.png)

📌 **Analysis:**
- **Observation:** 666 transactions across 2 sub-processes, 0% duplicate rate — the cleanest dataset in the project. TAT values can legitimately run into hours, reflecting the nature of cross-border transfers.
- **Root cause:** No single-step anomaly; TAT scale is inherent to the international transfer process itself.
- **Recommendation:** Use this unit as the baseline data-quality benchmark for the other units.

### 5️⃣ Card Processing

![Card Processing Dashboard](https://raw.githubusercontent.com/tranquochung09062001-dev/Banking-Operation/main/Card_Process_Dashboard.png)

📌 **Analysis:**
- **Observation:** 1,017 total occurrences across 4 sub-processes. The **HO Card intake & processing** step accounts for **70.25%** of total time despite representing only **2.95%** of volume (30/1,017) — median 1,794 minutes (~30 hours) per occurrence, with one case reaching 2,940 minutes (~2 days).
- **Data quality flag:** 8 rows show `Start > End` (negative TAT), including one outlier at −415 minutes in the Kiểm soát (control/oversight) step.
- **Root cause:** The delay is **waiting time for documents/logfiles from the business unit or a third party**, not actual processing effort — a **cross-department coordination** bottleneck, not a capacity issue at HO Card.
- **Recommendation:** Address the fix at the source — reduce document turnaround time from the business unit — rather than adding headcount at HO.

### 6️⃣ Hotline (Multi-channel Support)

![Hotline Dashboard](https://raw.githubusercontent.com/tranquochung09062001-dev/Banking-Operation/main/Hotline_Dashboard.png)

📌 **Analysis:**
- **Observation:** ~2,000 transactions, median TAT 4.70 hours. **"Interaction with external unit (if applicable)"** accounts for 64.34% of total time. Route 2 specifically (772 occurrences) shows an average TAT of 22.32 hours — 3.7× its own median (6.10 hours), a classic long-tail signature.
- **Root cause:** Waiting on responses from external units outside Hotline's direct control, with no SLA commitment in place — a **service-commitment gap**, not an internal-capacity issue.
- **Recommendation:** Establish a response-time SLA with the units Hotline depends on.

### 7️⃣ Process Improvement (IMPROV)

![Process Improvement Dashboard](https://raw.githubusercontent.com/tranquochung09062001-dev/Banking-Operation/main/Improv_Dashboard.png)

📌 **Analysis:**
- **Observation:** Small sample (n=5 proposal cycles, 20 step-level records). **Idea intake & evaluation** accounts for 67.02% of total time; median TAT dropped from 13.66 to 9.66 days after adjusting for weekend days.
- **Caveat:** With n=5, percentages are highly sensitive to single outliers (one 17.75-day case drives the "Approve list" step's 32.5% time share despite a median of 0 days for that step).
- **Root cause:** Intake/evaluation depends on a single evaluator/department with no committed turnaround deadline.
- **Recommendation:** Treat all findings from this unit as directional, not conclusive, until more cycles are logged; confirm the backup-approver hypothesis with the process owner.

---

## 🛠️ Skills & Tools Applied

| Category | Skills / Tools |
|---|---|
| **Data Cleaning & ETL** | Python (pandas): whitespace trimming, datetime parsing, deduplication, unit conversion, multi-file consolidation, handling negative/zero values without data loss |
| **Data Quality Auditing** | Detecting `Start > End` timestamp errors, quantifying duplicate rates per unit, distinguishing genuine negative TAT (rework) from data-entry errors |
| **DAX (Power BI)** | Median/average TAT measures (`MEDIANX`, `AVERAGEX`), % time-share and % interruption-share by step, grouped aggregation with `ADDCOLUMNS`/`CALCULATE`, Sankey flow measures |
| **Data Visualization** | Power BI report design across 7 unit pages + summary, KPI cards, ranked bar charts, Sankey diagrams, cross-filtering slicers |
| **Root Cause Analysis** | Separating workflow-design bottlenecks, cross-department dependency bottlenecks, and data-quality-driven distortions using step-level and route-level breakdowns |
| **Data Storytelling** | Observation → Root cause → Recommendation structure per unit, calibrating claim confidence to sample size and evidence strength |

---

## 🔎 Final Conclusion & Recommendations

### 1. A Small Number of Steps Drive Most of the Delay, Across Every Unit

**Insight:** In all 7 units, just 1–2 steps account for 55–70% of total processing time — the bottleneck is never evenly distributed.

**Recommendation:** Focus process-improvement effort on the specific bottleneck step identified per unit, not on across-the-board efficiency initiatives.

### 2. Several Bottlenecks Are Workflow-Design Problems, Not Staffing Problems

**Insight:** Counter's posting overload (5 business lines into 1 step) and Vault's 10-step HO→Branch route are structural — adding people would not fix them.

**Recommendation:** Redesign the workflow (split steps by volume, reduce route length) before considering headcount changes.

### 3. Card Processing and Hotline Share the Same Root Cause: No Cross-Department SLA

**Insight:** Both units' top bottleneck is waiting on an external party (business unit documents; external unit response) with no committed turnaround time.

**Recommendation:** Establish formal SLAs with the dependent departments — this is a governance fix, not a process-redesign fix.

### 4. Domestic Transfer and Card Processing Both Have Material Data Quality Issues

**Insight:** 24.29% of Domestic Transfer's "Order Processing" rows and 8 rows in Card Processing show `Start > End` (impossible negative duration from timestamp entry errors).

**Recommendation:** Fix source-system timestamp capture before using either step's TAT for capacity or SLA decisions — the current numbers likely understate true bottleneck severity.

### 5. Card Processing Has the Highest Duplicate Rate (51.42%) of Any Unit

**Insight:** Consolidating its 4 source files revealed that over half of raw rows were exact duplicates — far above any other unit (next highest: Vault at 12.57%).

**Recommendation:** Investigate the source-system logging process for Card Processing specifically; this duplicate rate suggests a systemic export or multi-logging issue, not random noise.

### 6. Process Improvement's Findings Should Be Treated as Directional, Not Final

**Insight:** With only 5 completed proposal cycles, single outliers swing percentages dramatically (e.g. one 17.75-day case drives a 32.5% time-share finding).

**Recommendation:** Revisit this unit's analysis once more cycles have been logged; avoid making resourcing decisions based on the current sample alone.
