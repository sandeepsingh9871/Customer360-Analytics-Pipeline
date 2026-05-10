# Customer 360 Analytics Pipeline

### Identity Resolution · RFM Segmentation · CLTV Modeling · Python → MySQL → Tableau

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-orange?logo=mysql&logoColor=white)
![Tableau](https://img.shields.io/badge/Tableau-Public-E87722?logo=tableau&logoColor=white)
![Scikit-learn](https://img.shields.io/badge/Scikit--learn-1.3-F7931E?logo=scikitlearn&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-1D9E75)

---

## Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [Solution Overview](#2-solution-overview)
3. [Pipeline Architecture](#3-pipeline-architecture)
4. [Tech Stack](#4-tech-stack)
5. [Project Structure](#5-project-structure)
6. [Step-by-Step Walkthrough](#6-step-by-step-walkthrough)
   - [Layer 1 — Python](#layer-1--python)
   - [Layer 2 — MySQL](#layer-2--mysql)
   - [Layer 3 — Tableau](#layer-3--tableau)
7. [Key Results](#7-key-results)
8. [Business Insights](#8-business-insights)
9. [Skills Gained](#9-skills-gained)
10. [How to Run](#10-how-to-run)
11. [Relevance to Deloitte & Amex](#11-relevance-to-deloitte--amex)

---

## 1. Problem Statement

Large enterprises operating across multiple customer touchpoints — web, mobile, in-store, and CRM — accumulate fragmented, siloed customer data. The same real customer appears as multiple records across systems, each with slightly different names, email formats, or phone numbers. This creates three critical business problems:

**1. Inflated customer count** — Marketing sends duplicate campaigns to the same person, wasting budget and damaging customer experience.

**2. Incomplete customer profiles** — Without a unified view, it is impossible to accurately measure lifetime value, purchase behaviour, or churn risk per customer.

**3. No actionable segmentation** — CRM and product teams cannot personalise offers, identify high-value customers, or build cross-sell pipelines without clean, consolidated data.

> **Real-world scale of this problem:** Industry research shows that enterprise CRM systems carry an average duplicate rate of 10–30%. For a firm like American Express processing millions of cardholders, a 13% dupe rate means hundreds of thousands of ghost records distorting every downstream analytics decision.

---

## 2. Solution Overview

This project builds a complete, production-structured **Customer 360 analytics pipeline** that:

1. **Resolves identity** — merges duplicate customer records from 4 channels into single golden profiles using fuzzy matching and probabilistic scoring
2. **Engineers KPIs** — computes RFM (Recency, Frequency, Monetary) scores, CLTV, churn risk flags, and engagement metrics per unified customer
3. **Segments customers** — groups 80,000 customers into 5 actionable business segments using both rule-based RFM scoring and K-Means clustering
4. **Delivers insights** — surfaces findings through an interactive Tableau dashboard connected live to a MySQL analytical layer

The pipeline mirrors real consulting deliverables at Deloitte's Customer Strategy practice and American Express's data analytics teams.

---

## 3. Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    PIPELINE OVERVIEW                            │
│                                                                 │
│   ┌──────────────┐    ┌──────────────┐    ┌─────────────────┐  │
│   │              │    │              │    │                 │  │
│   │   PYTHON     │───▶│    MYSQL     │───▶│    TABLEAU      │  │
│   │              │    │              │    │                 │  │
│   │ • Data gen   │    │ • 6 tables   │    │ • 6 sheets      │  │
│   │ • EDA        │    │ • 7 views    │    │ • KPI tiles     │  │
│   │ • Identity   │    │ • Window fns │    │ • Bubble chart  │  │
│   │   resolution │    │ • CTEs       │    │ • Heatmap       │  │
│   │ • KPI eng.   │    │ • Stored     │    │ • Funnel        │  │
│   │ • K-Means    │    │   procedures │    │ • Filters       │  │
│   │              │    │              │    │ • Parameters    │  │
│   └──────────────┘    └──────────────┘    └─────────────────┘  │
│                                                                 │
│   01_generate_and_load.py  →  02_mysql_views.sql  →  Tableau   │
└─────────────────────────────────────────────────────────────────┘
```

**Data flow:**

```
Raw multi-channel data (92,000 records)
        │
        ▼
[Python] Blocking + Fuzzy Matching + Union-Find
        │
        ▼
Golden Records (80,000 unique customers, 13% dupes removed)
        │
        ▼
[Python] RFM calculation + CLTV + Churn flag + K-Means
        │
        ▼
[MySQL] 6 production tables + 7 analytical views
        │
        ▼
[Tableau] Live dashboard with filters + parameter controls
```

---

## 4. Tech Stack

| Layer | Tool | Purpose |
|-------|------|---------|
| Data engineering | Python 3.10, Pandas, NumPy | Data generation, cleaning, transformation |
| Identity resolution | rapidfuzz, recordlinkage | Fuzzy name/email matching, blocking |
| Machine learning | Scikit-learn (KMeans, StandardScaler) | Segmentation clustering |
| Database | MySQL 8.0, SQLAlchemy, PyMySQL | Storage, analytical views, window functions |
| Visualisation | Tableau Desktop / Tableau Public | Interactive dashboard |
| Version control | Git, GitHub | Project documentation and sharing |

---

## 5. Project Structure

```
customer360-analytics-pipeline/
│
├── 01_python_generate_and_load.py   # Layer 1: Full Python pipeline
├── 02_mysql_views.sql               # Layer 2: MySQL schema, views, queries
├── 03_tableau_guide.py              # Layer 3: Tableau connection reference
│
├── notebooks/
│   └── customer360_eda_charts.py    # Exploratory analysis + 7 charts
│
├── outputs/
│   ├── charts/
│   │   ├── 01_eda_overview.png
│   │   ├── 02_correlation_heatmap.png
│   │   ├── 03_match_score_dist.png
│   │   ├── 04_kpi_distributions.png
│   │   ├── 05_kmeans_selection.png
│   │   ├── 06_segmentation_results.png
│   │   └── 07_rfm_scatter.png
│   └── tableau_exports/
│       ├── segment_summary.csv
│       ├── channel_mix.csv
│       └── customer_kpis.csv
│
├── requirements.txt
└── README.md
```

---

## 6. Step-by-Step Walkthrough

### Layer 1 — Python

#### Phase 1 · Data Setup & EDA (Day 1–2)

The pipeline ingests a multi-channel Customer 360 dataset with 92,000 records across 4 acquisition channels: web (38%), mobile (29%), in-store (21%), and CRM (12%). Included in those 92,000 records are 12,000 intentionally injected duplicates — simulating the real-world dupe rate found in enterprise CRMs.

EDA covers:
- Univariate distributions: age, income band, region, spend
- Bivariate analysis: correlation heatmap across all numeric features
- Channel-level data quality audit: null rates, duplicate counts, schema validation

```python
df.isnull().sum()                         # null audit
sns.heatmap(df.corr(), annot=True)        # correlation matrix
df["channel"].value_counts().plot("bar")  # channel distribution
```

**Key EDA finding:** `total_spend` and `txn_count` have the highest correlation (0.71), confirming that frequency and monetary value move together — a key assumption for RFM analysis.

---

#### Phase 2 · Identity Resolution (Day 3–4)

This is the most technically challenging phase and the most directly relevant to enterprise consulting work.

**The challenge:** The same customer appears across web, mobile, in-store, and CRM with slight variations — "Jon Smith" vs "John Smith", `jon.smith@gmail.com` vs `jon_smith@gmail.com`.

**Step 1 — Blocking:** Rather than comparing all 92,000 × 92,000 pairs (8.4 billion comparisons), a blocking key on `last_name[:3] + zip_code` reduces candidates to 13,046 pairs — a 99.98% reduction in compute.

**Step 2 — Fuzzy matching:** Each candidate pair is scored on 4 dimensions:

```python
composite_score = (
    0.35 * fuzz.token_sort_ratio(name_a, name_b) / 100 +
    0.30 * fuzz.ratio(email_a, email_b)           / 100 +
    0.20 * (1.0 if phone_a == phone_b else 0.0)  +
    0.15 * (1.0 if zip_a   == zip_b   else 0.0)
)
```

**Step 3 — Thresholds:**
- Score ≥ 0.82 → confirmed match → merged into one golden record
- Score 0.55–0.82 → review queue → flagged for manual inspection
- Score < 0.55 → not a match

**Step 4 — Golden record creation:** Matched IDs are merged using Union-Find. When field values conflict (e.g. two different emails), the record from the most authoritative source wins: CRM > web > in-store > mobile.

```
Records before deduplication :  92,000
Golden records after merging :  80,000
Duplicates removed           :  12,000  (13.04% dupe rate)
Match pairs evaluated        :  13,046
```

---

#### Phase 3 · KPI Engineering (Day 5–6)

A customer-level KPI table is built on top of the 80,000 golden records. Every metric is calculated as of snapshot date 2024-01-01.

| KPI | Formula | Business use |
|-----|---------|-------------|
| Recency | Days since last transaction | Churn detection |
| Frequency | Total transaction count | Engagement measure |
| Monetary | Total spend | Revenue contribution |
| CLTV | `avg_order_value × freq_monthly × 24` | Lifetime value proxy |
| Churn risk flag | `recency > 90 days AND freq < median` | Win-back targeting |
| High engagement | `login_freq > median` | Upsell readiness |
| CLTV decile | `pd.qcut(cltv, 10)` | Ranking for prioritisation |

```python
g["recency_days"]  = (SNAPSHOT - g["last_txn_date"]).dt.days
g["cltv"]          = g["avg_order_value"] * g["freq_monthly"] * 24
g["churn_risk"]    = ((g["recency_days"] > 90) & (g["frequency"] < median)).astype(int)
```

---

#### Phase 4 · Customer Segmentation (Day 7–8)

Two segmentation approaches are used in parallel — rule-based RFM scoring for interpretability, and K-Means clustering for statistical rigour.

**RFM quintile scoring:**
Each of R, F, M is scored 1–5 using `pd.qcut`. A customer scoring 5-5-5 is a Champion; 1-1-1 is Lost.

**K-Means clustering:**
- Features: recency_days, frequency, monetary (StandardScaler normalised)
- Optimal k: determined by elbow method + silhouette score → k = 5
- Clusters are profiled and named by business meaning

**5 segments with CRM action mapping:**

| Segment | Customers | Avg CLTV | CRM Action |
|---------|-----------|---------|------------|
| Champions | 256 | $279,112 | Referral program + Amex Platinum upsell |
| Promising | 3,906 | $58,860 | Cross-sell Amex Gold / travel rewards |
| At-Risk | 16,162 | $8,092 | Win-back campaign within 7 days |
| Hibernating | 29,781 | $7,691 | Reactivation email series |
| Lost | 29,895 | $7,714 | Suppress — quarterly review |

---

### Layer 2 — MySQL

The Python output writes 6 tables to MySQL via SQLAlchemy:

```
raw_customers      — 92,000 original records across all channels
golden_records     — 80,000 deduplicated master profiles
identity_matches   — 13,046 candidate pairs with composite scores
customer_kpis      — 80,000 rows, 35 KPI columns per customer
segment_summary    — 5 rows aggregated segment statistics
channel_mix        — 20 rows channel breakdown per segment
```

On top of these tables, `02_mysql_views.sql` creates 7 analytical views using advanced SQL:

**Interview-ready SQL patterns used:**

```sql
-- Window functions: RFM ranking
RANK()         OVER (ORDER BY cltv DESC)           AS cltv_rank,
NTILE(10)      OVER (ORDER BY cltv DESC)           AS cltv_decile,
PERCENT_RANK() OVER (ORDER BY cltv DESC)           AS cltv_percentile,
SUM(monetary)  OVER (PARTITION BY segment)         AS segment_total_revenue

-- CTE: cohort retention
WITH monthly_signups AS (
    SELECT DATE_FORMAT(signup_date, '%Y-%m') AS signup_month, ...
),
cohort_counts AS (
    SELECT signup_month, last_active_month, COUNT(*) AS customer_count ...
)
SELECT retention_pct FROM cohort_counts ...

-- Pareto / 80-20 analysis
SUM(monetary) OVER (
    ORDER BY monetary DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
) AS cumulative_revenue
```

**Stored procedures** allow dynamic threshold recalculation:

```sql
-- Recalculate churn flag with new thresholds — no Python needed
CALL sp_refresh_churn_flag(60, 3.0);   -- stricter
CALL sp_refresh_churn_flag(120, 5.0);  -- looser
```

---

### Layer 3 — Tableau

Tableau Desktop connects live to MySQL using the 7 views as data sources. No CSV exports needed for the production version.

**6 sheets built:**

| Sheet | Chart type | MySQL view | Key insight shown |
|-------|-----------|------------|------------------|
| Sheet 1 | BAN tiles | v_kpi_summary | 80K customers, $11K avg CLTV |
| Sheet 2 | Horizontal bar | v_segment_performance | Champions drive revenue concentration |
| Sheet 3 | Bubble chart | v_rfm_scatter | RFM map — Recency vs Monetary |
| Sheet 4 | Stacked bar | v_channel_attribution | Channel mix per segment |
| Sheet 5 | Heatmap | v_cohort_retention | Cohort retention over time |
| Sheet 6 | Funnel | v_crosssell_funnel | Amex cross-sell pipeline |

**4 interactive parameters:**

| Parameter | Range | Effect |
|-----------|-------|--------|
| Churn recency threshold | 30–365 days | Churn KPI tile updates live |
| Churn frequency threshold | 1–20 | Adjusts churn segment size |
| Min CLTV filter | $0–$10,000 | Filters entire dashboard |
| Top N segments | 1–5 | Shows top N segments only |

**3 dashboard actions:**
- Highlight action: click any segment → highlights across all 6 charts simultaneously
- Filter drill-down: click a bubble → opens customer detail sheet for that segment
- Parameter action: click a bar → sets CLTV filter to that segment's average

---

## 7. Key Results

```
┌─────────────────────────────────────────────────────────────┐
│  PIPELINE OUTPUT SUMMARY                                    │
├──────────────────────────────┬──────────────────────────────┤
│  Raw records ingested        │  92,000                      │
│  Golden records (post-dedup) │  80,000                      │
│  Duplicates resolved         │  12,000  (13.04% dupe rate)  │
│  Match pairs evaluated       │  13,046                      │
│  Customer KPI features built │  35 columns per customer     │
│  Segments created            │  5                           │
│  Churn risk customers        │  14,611  (18.3%)             │
│  CLTV at risk                │  $170M                       │
│  Avg CLTV across all         │  $11,147                     │
│  Champions avg CLTV          │  $279,112                    │
│  MySQL tables written        │  6                           │
│  MySQL analytical views      │  7                           │
│  Tableau sheets              │  6                           │
│  Tableau parameters          │  4                           │
└──────────────────────────────┴──────────────────────────────┘
```

---

## 8. Business Insights

**Insight 1 — Identity resolution is a revenue issue, not just a data quality issue**

12,000 duplicate records at a 13.04% dupe rate means every metric was inflated before deduplication. Marketing campaigns were reaching the same person multiple times, and CLTV calculations were understated because spend was split across ghost records.

**Insight 2 — Revenue is highly concentrated (Pareto principle confirmed)**

Champions represent only 0.32% of customers (256 out of 80,000) but drive 6.5% of total revenue at an average CLTV of $279,112. The Promising segment (3,906 customers, avg CLTV $58,860) represents the highest-opportunity cross-sell pipeline — directly applicable to Amex's card upsell strategy.

**Insight 3 — $170M CLTV is at churn risk**

14,611 customers (18.3%) are flagged as high churn risk with $170M in projected 24-month CLTV at stake. A targeted win-back campaign on the At-Risk segment (16,162 customers, avg CLTV $8,092) represents the highest ROI retention action.

**Insight 4 — Channel acquisition predicts customer quality**

CRM-acquired customers have the highest average CLTV, followed by web. Mobile acquisition drives volume but lower average spend. This informs budget allocation across acquisition channels.

**Insight 5 — 40.6% of customers are high-engagement**

Customers with above-median login frequency (40.6% of the base) show significantly higher CLTV, validating engagement as a leading indicator for loyalty programme investment.

---

## 9. Skills Gained

**Technical skills:**

| Skill | Applied in |
|-------|-----------|
| Fuzzy string matching | Identity resolution — rapidfuzz token_sort_ratio |
| Union-Find algorithm | Cluster assignment for matched record groups |
| Feature engineering | 35-column KPI table from raw transactional data |
| K-Means clustering | Customer segmentation with elbow + silhouette selection |
| SQL window functions | RANK, NTILE, PERCENT_RANK, LAG, SUM OVER PARTITION |
| CTEs | Cohort retention query with multi-step aggregation |
| Stored procedures | Dynamic churn threshold recalculation in MySQL |
| SQLAlchemy ORM | Python → MySQL write pipeline with chunked inserts |
| Tableau parameters | Live dashboard controls wired to calculated fields |
| Tableau actions | Highlight, filter drill-down, parameter change |
| Data quality auditing | Null rates, dupe detection, schema validation |

**Business & analytical skills:**

| Skill | Applied in |
|-------|-----------|
| RFM analysis | Customer scoring and segment labelling |
| CLTV modeling | 24-month projected value per customer |
| Churn prediction | Rule-based flag with dynamic threshold |
| Customer profiling | Golden record creation with source priority logic |
| Pareto analysis | Revenue concentration by segment |
| Cohort analysis | Retention heatmap by signup month |
| Cross-sell strategy | Amex-style funnel: 80K → 4K priority targets |
| CRM action mapping | Business recommendation per segment |

---

## 10. How to Run

#### Prerequisites

```bash
pip install pandas numpy scikit-learn rapidfuzz sqlalchemy pymysql matplotlib seaborn
```

MySQL 8.0 installed and running locally.

#### Step 1 — MySQL setup (run once)

```sql
CREATE DATABASE customer360;
CREATE USER 'c360_user'@'localhost' IDENTIFIED BY 'C360_pass!';
GRANT ALL PRIVILEGES ON customer360.* TO 'c360_user'@'localhost';
```

#### Step 2 — Run Python layer

```bash
python 01_python_generate_and_load.py
```

Expected output: 6 tables written to MySQL, ~2–3 minutes runtime for 92K records.

To use the real Kaggle dataset instead of synthetic data, replace Phase 1 with:

```python
df = pd.read_csv('your_kaggle_customer360.csv')
```

#### Step 3 — Run MySQL layer

Open `02_mysql_views.sql` in MySQL Workbench and run the entire file (Ctrl+Shift+Enter). This creates 7 views and 2 stored procedures.

#### Step 4 — Connect Tableau

```
Tableau Desktop → Connect → MySQL
Host     : localhost
Port     : 3306
Database : customer360
Username : User
Password : pass!
```

Connect to views: `v_segment_performance`, `v_channel_attribution`, `v_cohort_retention`, `v_crosssell_funnel`, `v_rfm_scatter`, `v_kpi_summary`.

#### Tableau Public workaround (no local MySQL)

Export views as CSV from MySQL Workbench and use the files in `outputs/tableau_exports/` as Text File sources in Tableau Public. The dashboard structure and all calculations remain identical.
---

**Tools:** Python · MySQL · Tableau · Pandas · Scikit-learn · rapidfuzz · SQLAlchemy
---

## Author - Sandeep Singh

## 📧 Contact

**Sandeep Singh**
- LinkedIn: [(https://www.linkedin.com/in/sandeep-singh-aaa1271b7/)]
- Email: [sandeepsinghss9871@gmail.com]
- GitHub: [(https://github.com/sandeepsingh9871)]

---

*Dataset: Synthetic data generated to mirror Customer 360 Kaggle dataset structure. All customer records are fictional.*
