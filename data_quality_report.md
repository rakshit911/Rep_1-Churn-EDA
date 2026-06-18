# Data Quality Report — D2C Customer Churn Dataset
**Prepared by:** Capstone Part 1  
**Snapshot date:** 2025-09-30  
**Files audited:** 7 (customers, orders, support_tickets, web_events_snapshot, churn_labels, rfm_modeling_snapshot, intervention_history)

---

## Summary Table

| # | Issue | File | Column(s) | Count | Severity | Recommended Treatment |
|---|---|---|---|---|---|---|
| 1 | Missing values — loyalty tier | customers.csv | `loyalty_tier` | 1,386 nulls (57.8%) | Medium | Fill with `"Not Enrolled"` category; treat as valid segment in modelling |
| 2 | Missing values — skin type | customers.csv | `skin_type` | 401 nulls (16.7%) | Low | Fill with `"Unknown"`; low expected feature importance |
| 3 | Missing values — order rating | orders.csv | `rating` | 80 nulls (0.8%) | Low | Impute with median rating per category before computing avg_rating features |
| 4 | Duplicate-like records | orders.csv | `order_id` | 12 rows with `_DUP` suffix | Medium | Drop `_DUP` rows before feature engineering; affects 11 customer_ids |
| 5 | Post-snapshot orders | orders.csv | `order_date` | 1,872 rows after 2025-09-30 | **Critical** | **Never use as features** , these fall in the churn target window (direct label leakage) |
| 6 | Outlier order values | orders.csv | `gross_amount` | 73 records above ₹2,396 fence; max ₹24,789 | Medium | Winsorise at 99th percentile (₹2,343) for monetary feature computation |
| 7 | Date consistency | All snapshot files | `snapshot_date` | All 4 files consistently show 2025-09-30 | None | No issue , verified consistent |
| 8 | Leakage-risk columns | churn_labels.csv, rfm_snapshot.csv | `churn_next_60d`, `split` | Present in 2 files | **Critical** | Exclude from feature matrix entirely; `churn_next_60d` is the target variable |

---

## Detailed Findings

### Issue 1 & 2 — Missing Values

```
File              Column           Nulls    Pct     
customers.csv     loyalty_tier     1,386    57.8%
customers.csv     skin_type          401    16.7%
orders.csv        rating              80     0.8%
rfm_snapshot.csv  loyalty_tier     1,386    57.8%   (mirrors customers.csv)
```

**loyalty_tier nulls** represent customers who have never enrolled in the loyalty programme, not data errors. When computing churn rates:
- Not enrolled: 48.3% churn rate
- Silver: 48.8% churn rate  
- Gold: 40.8% churn rate  
- Platinum: 37.1% churn rate  

This pattern is meaningful — null should not be imputed as one of the existing tiers.

**Treatment:** Replace `NaN` with `"Not Enrolled"` string before encoding. Use ordinal encoding: `Not Enrolled=0, Silver=1, Gold=2, Platinum=3`.

**skin_type nulls** appear to be genuinely missing (customers who did not fill in their profile). Given its low expected predictive value for churn, fill with `"Unknown"`.

**rating nulls** occur when a customer did not leave a rating. Before computing `avg_rating_180d`, impute with the median rating per product category (3.0–4.0 range).

---

### Issue 3 — Duplicate-Like Records

12 order rows have `order_id` values ending in `_DUP`:

```
order_id           customer_id  order_date    gross_amount
ORD008249_DUP      CUST00153    2025-11-04    321.31     
ORD002124_DUP      CUST00628    2025-03-18    410.04     
ORD002862_DUP      CUST00837    2025-07-12    952.02     
ORD002916_DUP      CUST00848    2025-09-26    547.18     
ORD002970_DUP      CUST00869    2024-12-22    818.64     
ORD008836_DUP      CUST00875    2025-10-23    711.20     (post-snapshot)
ORD003897_DUP      CUST01140    2025-04-14    769.96     
ORD004577_DUP      CUST01335    2025-02-12    533.07     
ORD005451_DUP      CUST01601    2024-11-07   1160.41     
ORD005529_DUP      CUST01621    2024-08-12    339.33     
ORD006220_DUP      CUST01820    2025-01-22    562.71     
ORD009518_DUP      CUST01820    2025-10-16    802.18     (post-snapshot)
```


**Treatment:** Filter `orders["order_id"].str.contains("_DUP") == False` before any analysis or feature engineering.

**Impact on pre-snapshot features:** 9 of the 12 DUP rows are pre-snapshot. If included, they would inflate frequency and monetary features for the 11 affected customers. Dropped before feature engineering.

---

### Issue 4 — Post-Snapshot Orders (Critical Leakage Risk)

```
Total orders in file   : 10,009
Pre-snapshot (safe)    :  8,137  (order_date ≤ 2025-09-30)
Post-snapshot (unsafe) :  1,872  (order_date > 2025-09-30)
```

The churn label (`churn_next_60d`) is defined as: did the customer place an order between 2025-10-01 and 2025-11-29? Therefore, **any feature derived from post-snapshot orders directly reveals the churn label**. Using these orders would produce:
- Artificially perfect training accuracy
- A model that is completely useless in production (it requires future data it will never have)

**Treatment:**
```python
SNAPSHOT = pd.Timestamp("2025-09-30")
pre_orders = orders[orders["order_date"] <= SNAPSHOT].copy()
# Use ONLY pre_orders for feature engineering
```

---

### Issue 5 — Gross Amount Outliers

```
Statistic              Value
Mean                   ₹743.90
Median                 ₹597.06
75th percentile        ₹907.43
IQR upper fence (Q3+3×IQR)  ₹2,396.15
99th percentile        ₹2,342.97
Records above fence    73
Maximum value          ₹24,789.38
```

The maximum value (₹24,789) is ~41× the median, suggesting either data entry errors or a small number of very high-value bulk orders. 6 records exceed ₹5,000.

**Treatment:** Winsorise `gross_amount` at the 99th percentile (₹2,343) before computing features. This prevents a handful of outlier orders from distorting the monetary signal for 1% of customers.

---

### Issue 6 — Date Consistency

All `snapshot_date` fields across all four snapshot files (`web_events_snapshot.csv`, `churn_labels.csv`, `rfm_modeling_snapshot.csv`, `intervention_history.csv`) contain a single consistent value: `2025-09-30`.

```
customers.signup_date range  : 2024-01-01 – 2025-09-15  (no post-snapshot signups)
tickets.ticket_date range    : 2024-01-13 – 2025-09-30  (no future tickets)
All snapshot_date fields     : 2025-09-30 only           
```

**No issues found.** Date fields are internally consistent.

---

### Issue 7 — Leakage-Risk Columns Reference

| Column | File | Notes |
|---|---|---|
| `order_date > 2025-09-30` | orders.csv |  Post-snapshot orders directly encode the churn label |
| `churn_next_60d` | churn_labels.csv | This is the target, not a feature |
| `churn_next_60d` | rfm_modeling_snapshot.csv | Critical | Same target embedded in RFM file |
| `split` | churn_labels.csv |  Metadata / assignment, not a feature |
| `snapshot_date` | All snapshot files | Constant field: zero predictive value, wastes a feature slot |
| `customer_id` | All files |  Identifier: should not be a numeric feature |

---

## Join Health Summary

| Side Table | Orphan Keys | Coverage |
|---|---|---|
| orders.csv | 0 | All order customer_ids match customers.csv  |
| support_tickets.csv | 0 | All ticket customer_ids match customers.csv |
| web_events_snapshot.csv | 0 | 1:1 match with customers.csv |
| churn_labels.csv | 0 | 1:1 match with customers.csv |
| rfm_modeling_snapshot.csv | 0 | 1:1 match with customers.csv |
| intervention_history.csv | 0 | 1:1 match with customers.csv  |

1,153 customers (48.1%) have no support tickets, this is **expected behaviour**, not a data error. These customers simply never contacted support.

---

## Pre-Processing Checklist for Modelling

1. Filtered `orders` to `order_date ≤ 2025-09-30` (remove leakage rows)
2. Dropped `_DUP` order rows (`order_id.str.contains("_DUP")`)
3. Filled `loyalty_tier` nulls → `"Not Enrolled"`
4. Filled `skin_type` nulls → `"Unknown"`
5. Imputed `rating` nulls → median per category
6. Winsorised `gross_amount` at 99th percentile (₹2,343)
7. Dropped `churn_next_60d` and `split` from feature matrix
8. Used `split` column from `churn_labels.csv` for train/val/test assignment
