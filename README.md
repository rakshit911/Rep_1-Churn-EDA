# Part 1 - D2C Customer Churn: Data Audit, EDA & Business Understanding

**Capstone Project | AI/ML Course | Part 1 of 4**

This repository contains the full data audit, exploratory data analysis, and business understanding for a D2C personal-care brand's customer churn problem. The snapshot date is **2025-09-30**, and the target is whether a customer will make at least one purchase in the following 60 days.

---

## Repository Contents

```
part1-eda/
├── README.md                    
├── requirements.txt             (Python dependencies)
├── eda_audit.ipynb              (Main code notebook)
├── data_quality_report.md       
├── business_memo.md             
└── data/
    ├── customers.csv
    ├── orders.csv
    ├── support_tickets.csv
    ├── web_events_snapshot.csv
    ├── churn_labels.csv
    ├── rfm_modeling_snapshot.csv
    └── intervention_history.csv
```

**Charts are generated and saved to a `charts/` folder when the notebook is run**.

---

## Setup

### 1. Clone the repository

git clone <your-repo-url>
cd part1-eda


### 2. Install dependencies

pip install -r requirements.txt


### 3. Launch the notebook

jupyter notebook eda_audit.ipynb


Or with JupyterLab:

jupyter lab eda_audit.ipynb


### 4. Run all cells
In Jupyter: **Kernel → Restart & Run All**

The notebook will load data from `data/`, generate all charts, and save them to `charts/`.

---

## What the Notebook Covers

| Section | Content |
|---|---|
| 1 | Load all 7 datasets, row counts, dtypes, schema |
| 2 | Join integrity, orphan key checks, 1:1 table verification |
| 3 | Data quality audit, missing values, duplicates, outliers, date consistency, leakage risks |
| 4 | Churn distribution, label balance, train/val/test split |
| 5 | Customer demographics, acquisition channel, age group, loyalty tier, city tier |
| 6 | Order behaviour ,  recency buckets, frequency, monetary, discount quintiles |
| 7 | Return/refund behaviour , return rate vs churn |
| 8 | Support ticket analysis , issue types, sentiment, ticket volume vs churn |
| 9 | Web/app activity , sessions, last visit, abandoned carts |
| 10 | Campaign/intervention history , campaign type, CRM priority vs actual churn |
| 11 | Feature correlation analysis , heatmap + churn correlation bar chart |
| 12 | Five churn-risk hypotheses , evidence-backed with supporting charts |

---

## Key Findings

- **Overall churn rate: 47.0%** (1,127 churned / 2,400 total), nearly balanced
- **Strongest predictor: `recency_days`** (r=+0.561) , customers inactive 90+ days churn at 78–91%
- **Second strongest: `last_visit_days_ago`** (r=+0.527) , web inactivity mirrors purchase inactivity
- **Loyalty gap:** Unenrolled customers (57.8% of base) churn at 48.3% vs Platinum at 37.1%
- **Channel quality:** Organic (39.8%) and Referral (42.2%) channels significantly outperform paid channels (~50%)
- **Leakage risk:** 1,872 post-snapshot orders in `orders.csv` must never be used as model features

---

## Data Quality Issues Found

| Issue | Impact | Treatment |
|---|---|---|
| 1,386 nulls in `loyalty_tier` (57.8%) | Feature encoding | Fill with `"Not Enrolled"` |
| 12 `_DUP` order records | Inflated frequency/monetary | Drop before feature engineering |
| 1,872 post-snapshot orders | **Critical leakage risk** | Filter to `order_date ≤ 2025-09-30` |
| 73 `gross_amount` outliers (max ₹24,789) | Distorted monetary features | Winsorise at 99th percentile (₹2,343) |
| 80 null `rating` values | Avg rating computation | Impute with median per category |

---

## Deliverables

| File | Description |
|---|---|
| `eda_audit.ipynb` | Full analysis notebook with 9 charts |
| `data_quality_report.md` | Structured audit with treatment recommendations |
| `business_memo.md` | 4-finding memo for the retention team |
| `charts/` | 9 saved chart PNGs (generated on run) |
