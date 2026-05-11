# Customer 360 Analytics Pipeline

### Identity Resolution · RFM Segmentation · CLTV Modeling · Python → MySQL → Tableau

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?logo=mysql&logoColor=white)
![Tableau](https://img.shields.io/badge/Tableau-Public-E87722?logo=tableau&logoColor=white)
![Scikit-learn](https://img.shields.io/badge/Scikit--learn-1.3-F7931E?logo=scikitlearn&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-2.0-150458?logo=pandas&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-1D9E75)

> **Portfolio case study** targeting Data Analyst roles.
> Built a production-structured end-to-end analytics pipeline on the Customer360Insights dataset (Kaggle).

---

## Live Dashboard

> 📊 **[View on Tableau Public → add your link here]**

![Dashboard Preview](screenshots/dashboard_overview.png)

---

## Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [Solution & Architecture](#2-solution--architecture)
3. [Tech Stack](#3-tech-stack)
4. [Project Structure](#4-project-structure)
5. [Dataset](#5-dataset)
6. [Pipeline Walkthrough](#6-pipeline-walkthrough)
7. [Key SQL Queries](#7-key-sql-queries)
8. [Results](#8-results)
9. [Business Insights](#9-business-insights)
10. [Tableau Dashboard](#10-tableau-dashboard)
11. [How to Run](#11-how-to-run)
12. [Screenshots](#12-screenshots)

---

## 1. Problem Statement

Large enterprises operating across web, mobile, in-store, and CRM channels accumulate the same customer as multiple fragmented records — each with slightly different names, email formats, or phone numbers. This creates three critical business problems:

**Inflated customer counts** — Marketing campaigns reach the same person multiple times, wasting budget.

**Incomplete profiles** — Without a unified view, CLTV, churn risk, and purchase behaviour cannot be measured accurately per customer.

**No actionable segmentation** — CRM and product teams cannot personalise offers or build cross-sell pipelines without clean, consolidated data.

> *In a dataset of 2,000 sessions spanning 1,200 customers, 44 near-duplicate pairs were identified — a 3.7% near-dupe rate that would distort every downstream metric if unresolved.*

---

## 2. Solution & Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    PIPELINE ARCHITECTURE                         │
│                                                                  │
│  ┌─────────────────┐    ┌──────────────────┐    ┌────────────┐  │
│  │   PYTHON        │    │   MYSQL          │    │  TABLEAU   │  │
│  │                 │    │                  │    │            │  │
│  │ Phase 1: Load   │    │ 6 raw tables     │    │ 6 sheets   │  │
│  │ Phase 2: Dedup  │───▶│ 7 analytical     │───▶│ Live MySQL │  │
│  │ Phase 3: KPIs   │    │   views          │    │ connection │  │
│  │ Phase 4: Seg.   │    │ Window fns + CTEs│    │ Filters    │  │
│  │ Phase 5: Export │    │ Stored procs     │    │ Actions    │  │
│  └─────────────────┘    └──────────────────┘    └────────────┘  │
│                                                                  │
│  Customer360Insights.csv → customer360 DB → Live Dashboard      │
└──────────────────────────────────────────────────────────────────┘
```

**Data flow:**
```
2,000 raw sessions (25 columns)
        │
        ▼ Strip whitespace · Parse datetimes · Fix nulls · Engineer features
        │
1,200 customer-level profiles
        │
        ▼ Blocking (last_name[:3] + city[:4]) → Fuzzy matching (name + city + age)
        │
44 near-duplicate pairs resolved → 1,197 golden records
        │
        ▼ RFM · CLTV · Churn flag · Engagement score · K-Means (k=5)
        │
6 MySQL tables → 7 Tableau views → Interactive dashboard
```

---

## 3. Tech Stack

| Layer | Tool | Purpose |
|-------|------|---------|
| Data engineering | Python 3.10, Pandas, NumPy | Load, clean, transform |
| Identity resolution | rapidfuzz | Fuzzy name/city matching |
| Machine learning | Scikit-learn (KMeans, StandardScaler) | Customer segmentation |
| Database | MySQL 8.0, SQLAlchemy, PyMySQL | Storage + analytical views |
| Visualisation | Tableau Desktop / Tableau Public | Live interactive dashboard |

---

## 4. Project Structure

```
customer360-analytics-pipeline/
│
├── Customer360-Analytics.ipynb   # Full Python pipeline (5 phases)
├── customer360.sql               # MySQL schema, views, queries
├── Customer360dashboard.twb      # Tableau workbook
│
├── screenshots/
│   ├── dashboard_overview.png
│   ├── rfm_bubble_map.png
│   ├── revenue_by_segment.png
│   ├── cohort_heatmap.png
│   ├── crosssell_funnel.png
│   ├── channel_mix.png
│   └── mysql_tables.png
│
└── README.md
```

---

## 5. Dataset

**Customer360Insights.csv** — [Kaggle](https://www.kaggle.com/code/davedarshan/synthetic-saga-the-art-of-crafting-customer360in)

| Property | Value |
|----------|-------|
| Rows | 2,000 sessions |
| Columns | 25 |
| Unique customers | 1,200 |
| Date range | 2019-01-01 → 2023-12-31 |

**Key columns used:**

| Column | Renamed to | Used for |
|--------|-----------|---------|
| `CustomerID` | `customer_id` | Identity key |
| `FullName` | `full_name` | Fuzzy matching |
| `CampaignSchema` | `channel` | Channel attribution |
| `SessionStart` | `last_txn_date` | Recency calculation |
| `Price × Quantity` | `txn_revenue` | Monetary value |
| `CreditScore` | `credit_score` | Engagement scoring |
| `MonthlyIncome` | `monthly_income` | Income band |
| `OrderReturn` | `order_return_rate` | Return behaviour |

**Nulls handled:**

| Column | Nulls | Fix |
|--------|-------|-----|
| `OrderConfirmationTime` | 300 | Filled with `CartAdditionTime + 15 min` |
| `PaymentMethod` | 300 | Filled with `'Unknown'` |
| `ReturnReason` | 1,764 | Filled with `'No Return'` (expected — no return occurred) |

---

## 6. Pipeline Walkthrough

### Phase 1 — Load & Clean

```python
df = pd.read_csv('Customer360Insights.csv')
df.columns = df.columns.str.strip()          # fix trailing whitespace in 'CampaignSchema '

# channel consolidation: 5 ad types → 3 clean channels
channel_map = {
    'Instagram-ads': 'Social', 'Facebook-ads': 'Social', 'Twitter-ads': 'Social',
    'Google-ads': 'Search',    'Billboard-QR code': 'Offline',
}
df['channel'] = df['channel'].map(channel_map).fillna('Other')

# derived: revenue per session, session duration, income band
df['txn_revenue']  = (df['price'] * df['quantity']).round(2)
df['session_mins'] = ((df['session_end'] - df['session_start'])
                      .dt.total_seconds() / 60).clip(0, 300)
```

---

### Phase 2 — Identity Resolution

The most technically complex phase. Resolves near-duplicate customer records from multiple channels into single golden profiles.

**Step 1 — Blocking:** Reduces 1,200² = 1.44M candidate pairs to a manageable set using a composite key: `last_name[:3] + city[:4]`.

**Step 2 — Fuzzy scoring:** Each candidate pair scored on 3 signals:

```python
composite = (
    0.50 * fuzz.token_sort_ratio(name_a, name_b) / 100 +  # name similarity
    0.30 * fuzz.ratio(city_a, city_b)           / 100 +  # city match
    0.20 * (1.0 if abs(age_a - age_b) <= 2 else 0.0)    # age proximity
)
# score ≥ 0.82 → confirmed match → merged via Union-Find
# score 0.55–0.82 → review queue
```

**Step 3 — Union-Find:** Matched IDs are merged into clusters. Each cluster produces one golden record (most recent session takes priority).

```
Near-duplicate pairs evaluated : 44
Confirmed matches               : resolved into golden records
Golden records produced         : 1,197
```

---

### Phase 3 — KPI Engineering

35 customer-level features built on the 1,197 golden records:

```python
SNAPSHOT = golden['last_txn_date'].max() + pd.Timedelta(days=1)  # dataset-driven

# RFM
g['recency_days']    = (SNAPSHOT - g['last_txn_date']).dt.days
g['frequency']       = g['txn_count']
g['monetary']        = g['total_spend']

# CLTV (24-month proxy)
g['avg_order_value'] = g['monetary'] / g['frequency'].replace(0, 1)
g['freq_monthly']    = g['frequency'] / g['tenure_months']
g['cltv']            = (g['avg_order_value'] * g['freq_monthly'] * 24).round(2)

# Churn risk: inactive customers (top 60% recency) + low frequency (bottom 40%)
recency_threshold   = g['recency_days'].quantile(0.60)
frequency_threshold = g['frequency'].quantile(0.40)
g['churn_risk'] = (
    (g['recency_days'] >= recency_threshold) &
    (g['frequency']    <= frequency_threshold)
).astype(int)

# Engagement score (0–100): session duration + frequency + credit score
g['engagement_score'] = (
    g['avg_session_mins'].clip(0, 60) / 60 * 40 +
    g['frequency'].clip(0, 10)        / 10 * 40 +
    (g['credit_score'].clip(300, 900) - 300) / 600 * 20
).round(1)
```

---

### Phase 4 — Segmentation

**RFM scoring** using rank percentiles (avoids `pd.qcut` duplicate bin error):

```python
def safe_score(series, reverse=False):
    pct = series.rank(method='first', pct=True)
    if reverse:
        pct = 1 - pct
    return pd.cut(pct, bins=[-0.01, 0.20, 0.40, 0.60, 0.80, 1.01],
                  labels=[1, 2, 3, 4, 5]).astype(int)

g['R_score'] = safe_score(g['recency_days'], reverse=True)
g['F_score'] = safe_score(g['frequency'])
g['M_score'] = safe_score(g['monetary'])
```

**K-Means clustering** (k=5, justified by elbow + silhouette analysis):

```python
X_scaled = StandardScaler().fit_transform(
    g[['recency_days', 'frequency', 'monetary']])
km = KMeans(n_clusters=5, random_state=42, n_init=10)
g['cluster'] = km.fit_predict(X_scaled)
```

**Segment → CRM action mapping:**

| Segment | Customers | Avg CLTV | CRM Action |
|---------|-----------|---------|------------|
| Champions | 112 | $4,382 | Referral program + premium card upsell |
| Promising | 148 | $760 | Cross-sell — travel rewards / premium product |
| At-Risk | 351 | $523 | Win-back campaign within 7 days |
| Hibernating | 306 | $359 | Reactivation email series |
| Lost | 280 | $187 | Suppress — quarterly review |

---

### Phase 5 — MySQL Export

6 tables written via SQLAlchemy with duplicate column detection:

```python
eng = create_engine("mysql+pymysql://user:pass@localhost:3306/customer360")

tables = {
    'customer_kpis'   : clean_df(kpi_df),      # 1,197 rows · 35 KPI columns
    'golden_records'  : clean_df(golden),       # 1,197 deduplicated profiles
    'identity_matches': identity_matches_df,    # 44 match pairs with scores
    'segment_summary' : seg_summary,            # 5 rows · segment aggregates
    'channel_mix'     : channel_mix,            # 20 rows · channel breakdown
    'country_mix'     : country_mix,            # 45 rows · country breakdown
}
```

---

## 7. Key SQL Queries

### Executive KPI summary

```sql
SELECT
    COUNT(DISTINCT customer_id)                            AS total_customers,
    ROUND(AVG(cltv), 2)                                    AS avg_cltv,
    SUM(churn_risk)                                        AS churn_risk_customers,
    ROUND(SUM(churn_risk) / COUNT(*) * 100, 2)            AS churn_risk_pct,
    ROUND(SUM(CASE WHEN churn_risk = 1 THEN cltv END), 2) AS cltv_at_risk
FROM customer_kpis;
```

---

### Segment performance with window function

```sql
SELECT
    segment,
    COUNT(*)                                                AS customers,
    ROUND(SUM(monetary) * 100.0 / SUM(SUM(monetary))
          OVER (), 2)                                       AS revenue_share_pct,
    ROUND(AVG(cltv), 2)                                    AS avg_cltv,
    ROUND(SUM(CASE WHEN churn_risk=1 THEN cltv END), 2)   AS cltv_at_risk
FROM customer_kpis
GROUP BY segment
ORDER BY avg_cltv DESC;
```

---

### CLTV ranking — RANK, NTILE, PERCENT_RANK 

```sql
SELECT
    customer_id, segment, monetary, cltv,
    RANK()         OVER (ORDER BY cltv DESC)           AS cltv_rank,
    NTILE(10)      OVER (ORDER BY cltv DESC)           AS cltv_decile,
    PERCENT_RANK() OVER (ORDER BY cltv DESC)           AS cltv_pct_rank,
    RANK()         OVER (PARTITION BY segment
                         ORDER BY cltv DESC)           AS rank_within_segment,
    AVG(cltv)      OVER (PARTITION BY segment)         AS segment_avg_cltv,
    ROUND(cltv - AVG(cltv) OVER (), 2)                AS cltv_vs_overall_avg
FROM customer_kpis
ORDER BY cltv DESC
LIMIT 100;
```

---

### Pareto / 80-20 analysis — CTE with running total

```sql
WITH revenue_ranked AS (
    SELECT
        customer_id, segment, monetary,
        SUM(monetary) OVER (
            ORDER BY monetary DESC
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        )                                              AS cumulative_revenue,
        SUM(monetary) OVER ()                          AS total_revenue,
        ROW_NUMBER() OVER (ORDER BY monetary DESC)    AS rank_num,
        COUNT(*) OVER ()                               AS total_customers
    FROM customer_kpis
)
SELECT
    rank_num, segment,
    ROUND(cumulative_revenue / total_revenue * 100, 2) AS cumulative_rev_pct,
    ROUND(rank_num * 100.0 / total_customers, 2)       AS cumulative_cust_pct
FROM revenue_ranked
WHERE rank_num <= 200;
```

---

### Churn risk deep-dive — multi-step CTE

```sql
WITH churn_base AS (
    SELECT segment, cltv, recency_days, monetary
    FROM customer_kpis
    WHERE churn_risk = 1
),
churn_summary AS (
    SELECT
        segment,
        COUNT(*)                    AS churned_customers,
        ROUND(SUM(cltv), 2)        AS total_cltv_at_risk,
        ROUND(AVG(recency_days), 0) AS avg_recency_days
    FROM churn_base
    GROUP BY segment
)
SELECT
    cs.*,
    ROUND(cs.total_cltv_at_risk * 100.0 /
          SUM(cs.total_cltv_at_risk) OVER (), 2)  AS cltv_share_pct,
    RANK() OVER (ORDER BY total_cltv_at_risk DESC) AS priority_rank
FROM churn_summary cs
ORDER BY total_cltv_at_risk DESC;
```

---

### Cohort retention heatmap — CTE + partition window (Amex loyalty use case)

```sql
WITH monthly_activity AS (
    SELECT
        customer_id,
        DATE_FORMAT(signup_date,   '%Y-%m') AS signup_month,
        DATE_FORMAT(last_txn_date, '%Y-%m') AS last_active_month
    FROM customer_kpis
),
cohort_counts AS (
    SELECT signup_month, last_active_month, COUNT(*) AS customers
    FROM monthly_activity
    GROUP BY signup_month, last_active_month
)
SELECT
    signup_month,
    last_active_month,
    customers,
    SUM(customers) OVER (PARTITION BY signup_month) AS cohort_size,
    ROUND(customers * 100.0 /
          SUM(customers) OVER (PARTITION BY signup_month), 2) AS retention_pct
FROM cohort_counts
ORDER BY signup_month, last_active_month;
```

---

### Cross-sell funnel — Amex upsell pipeline

```sql
WITH funnel AS (
    SELECT 'Total base'         AS stage, COUNT(*) AS customers,
           ROUND(AVG(cltv),2)  AS avg_cltv, 1 AS stage_order FROM customer_kpis
    UNION ALL
    SELECT 'Active (no churn)', COUNT(*), ROUND(AVG(cltv),2), 2
    FROM customer_kpis WHERE churn_risk = 0
    UNION ALL
    SELECT 'High CLTV (top 35%)', COUNT(*), ROUND(AVG(cltv),2), 3
    FROM customer_kpis WHERE cltv_decile >= 6
    UNION ALL
    SELECT 'Promising segment',  COUNT(*), ROUND(AVG(cltv),2), 4
    FROM customer_kpis WHERE segment = 'Promising'
    UNION ALL
    SELECT 'Champions only',     COUNT(*), ROUND(AVG(cltv),2), 5
    FROM customer_kpis WHERE segment = 'Champions'
)
SELECT stage, customers, avg_cltv,
       ROUND(customers * 100.0 /
             FIRST_VALUE(customers) OVER (ORDER BY stage_order), 2) AS funnel_pct
FROM funnel ORDER BY stage_order;
```

---

### Stored procedure — dynamic churn recalculation

```sql
CALL sp_refresh_churn(730, 1);   -- flag customers absent 2+ years with 1 purchase
CALL sp_refresh_churn(365, 2);   -- stricter: 1 year + max 2 purchases
```

---

## 8. Results

```
┌─────────────────────────────────────────────────────────────┐
│  PIPELINE OUTPUT                                            │
├──────────────────────────────┬──────────────────────────────┤
│  Raw sessions loaded         │  2,000                       │
│  Unique customers identified │  1,200                       │
│  Near-duplicate pairs found  │  44                          │
│  Golden records              │  1,197                       │
│  KPI features per customer   │  35                          │
│  Segments created            │  5                           │
│  Churn risk customers        │  479  (40%)                  │
│  Avg CLTV                    │  $1,242                      │
│  Champions avg CLTV          │  $4,382                      │
│  Total revenue (2-yr proj.)  │  $1,521K                     │
│  MySQL tables written        │  6                           │
│  MySQL analytical views      │  7                           │
│  Tableau sheets              │  6                           │
└──────────────────────────────┴──────────────────────────────┘
```

---

## 9. Business Insights

**1. Revenue is highly concentrated**
Champions represent 9.4% of customers but drive **50.9% of total revenue** at an avg CLTV of $4,382. Classic Pareto concentration — validates the focus on high-value customer retention.

**2. $247K CLTV is recoverable**
407 customers are flagged as churn risk with $247K in projected 24-month value at stake. The At-Risk segment (351 customers, $523 avg CLTV) represents the highest ROI win-back opportunity.

**3. Cross-sell funnel narrows to 112 Champions**
Out of 1,197 customers → 718 active → 479 high-CLTV → 148 Promising → **112 Champions**. These 112 are the priority targets for an Amex-style premium card upsell at $4,382 avg CLTV.

**4. Social channel acquires most customers but Search acquires highest-value ones**
Channel attribution in `vw_channel_mix` shows Social dominates volume while Search-acquired customers show higher avg CLTV — informing budget allocation decisions.

**5. Cohort retention varies significantly by signup month**
The retention heatmap identifies which acquisition cohorts retain longest — 2020-08 and 2021-02 cohorts show above-average retention, signalling effective acquisition campaigns in those periods.

---

## 10. Tableau Dashboard

**6 sheets connected live to MySQL:**

| Sheet | Chart type | MySQL view |
|-------|-----------|------------|
| KPI tiles | 4 BAN numbers + filter | vw_segment_performance |
| Revenue by segment | Horizontal bar | vw_segment_performance |
| RFM Bubble Map | Scatter/bubble | vw_rfm_scatter |
| Channel mix | 100% stacked bar | vw_channel_mix |
| Cohort Retention Heatmap | Square heatmap | vw_cohort_retention |
| Cross-Sell Funnel | Horizontal funnel | vw_crosssell_funnel |

**Interactive features:**
- Segment filter (All / Champions / At-Risk / Promising / Hibernating / Lost)
- Highlight action: clicking any segment highlights it across all 6 charts simultaneously
- Custom tooltips on each chart showing segment, CLTV, and CRM action recommendation

---

## 11. How to Run

### Prerequisites

```bash
pip install pandas numpy scikit-learn rapidfuzz sqlalchemy pymysql matplotlib seaborn
```

MySQL 8.0 running locally.

### Step 1 — MySQL setup

```sql
CREATE DATABASE customer360;
CREATE USER 'your_user'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON customer360.* TO 'your_user'@'localhost';
FLUSH PRIVILEGES;
```

### Step 2 — Update credentials in notebook

In `Customer360-Analytics.ipynb`, update the connection cell:

```python
engine = create_engine("mysql+pymysql://YOUR_USER:YOUR_PASSWORD@localhost:3306/customer360")
```

### Step 3 — Run the notebook

```bash
jupyter notebook Customer360-Analytics.ipynb
```

Run all cells top to bottom. Expected runtime: ~3–5 minutes.

### Step 4 — Run SQL file

Open `customer360.sql` in MySQL Workbench → run entire file (`Ctrl+Shift+Enter`).
This creates 7 analytical views, 8 interview-ready queries, and 2 stored procedures.

### Step 5 — Connect Tableau

```
Tableau Desktop → Connect → MySQL
Server   : localhost
Port     : 3306
Database : customer360
Username : your_user
Password : your_password
```

Connect to views: `vw_segment_performance`, `vw_rfm_scatter`, `vw_channel_mix`,
`vw_cohort_retention`, `vw_crosssell_funnel`.

Or open `Customer360dashboard.twb` directly (update the data source credentials).

---

## 12. Screenshots

> Add these screenshots to a `screenshots/` folder in your repo.

### Required screenshots (7 total):

| File | What to capture | How |
|------|----------------|-----|
| `dashboard_overview.png` | Full dashboard — all 6 charts visible | In Tableau: press F7 (presentation mode) → screenshot |
| `rfm_bubble_map.png` | RFM bubble map sheet only | Click the sheet tab → screenshot |
| `revenue_by_segment.png` | Revenue bar chart sheet only | Click the sheet tab → screenshot |
| `cohort_heatmap.png` | Cohort retention heatmap sheet | Click the sheet tab → screenshot |
| `crosssell_funnel.png` | Cross-sell funnel sheet | Click the sheet tab → screenshot |
| `channel_mix.png` | Channel mix stacked bar sheet | Click the sheet tab → screenshot |
| `mysql_tables.png` | MySQL Workbench showing all 6 tables + 7 views in the left panel | Open Workbench → expand customer360 schema → screenshot |

**Recommended:** also add `pipeline_architecture.png` — a simple diagram showing Python → MySQL → Tableau flow. You can draw this in draw.io (free) and export as PNG.

---

## Author

Built as a portfolio case study targeting Data Analyst roles.

**Skills demonstrated:** Python · Pandas · Scikit-learn · rapidfuzz · SQLAlchemy · MySQL · Window functions · CTEs · Stored procedures · Tableau · RFM analysis · K-Means clustering · Identity resolution · CLTV modeling · Cohort analysis

##  Contact

**Sandeep Singh**

- LinkedIn: [(https://www.linkedin.com/in/sandeep-singh-aaa1271b7/)]

- Email: [sandeepsinghss9871@gmail.com]

- GitHub: [(https://github.com/sandeepsingh9871)]

---

*Dataset: Customer360Insights.csv — synthetic data from Kaggle. All customer records are fictional.*
