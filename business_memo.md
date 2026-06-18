# Business Memo: Customer Churn Risk Findings

**To:** Retention & CRM Team  
**From:** Data Analytics (Capstone Part 1)  
**Date:** 2025-09-30 (Snapshot Date)  
**Re:** Pre-Campaign Findings - D2C Churn Analysis

---

## Situation

As of 30 September 2025, **47.0% of our 2,400 active customers** are projected to churn in the next 60 days i.e. they will make no purchase between 1 October and 29 November. This is not a small tail risk. Nearly half our customer base is at risk of going silent.

Before launching any retention campaign, this memo outlines what the data reveals about *why* customers churn and who should be prioritised. since the budget is limited, it is crucial to find the customers who can be incited using promotions or discounts.

---

## Finding 1: Last Ordered

The last ordered date can tell a lot about a customer's churn status,

| Last order | Churn rate | n |
|---|---|---|
| Within 30 days | 11.9% | 671 |
| 31–60 days ago | 30.4% | 441 |
| 61–90 days ago | 45.7% | 361 |
| 91–180 days ago | 78.7% | 597 |
| 180+ days ago | 91.4% | 302 |

Customers who have not purchased in 3 months are nearly certain to churn (78.7%). Those who have not purchased in 6 months are effectively already lost (91.4%). We have 899 customers in the 91+ day buckets who have been left uncontacted. **This is the highest-priority group for intervention.**

A related finding: customers who have not visited our website in the past 26+ days churn at dramatically higher rates than those who visited within the last 10 days. Web inactivity and purchase inactivity are connected and both are early warning signals we can act on before an order is actually missed.

**Investigations:** Are our email and push campaigns reaching lapsed customers at all? The analysis shows that customers currently flagged by the CRM as "high priority" do genuinely churn at 74.7% . But 1,163 customers are in that bucket. We cannot intervene on all of them with the same message.

---

## Finding 2: The Loyalty Programme Is Under-Utilised

**57.8% of our customers (1,386 people) have never enrolled in the loyalty programme.**

These customers churn at 48.3%. Platinum members churn at only 37.1%. The 11.2 percentage point gap is large enough to be commercially significant.

This is the largest single addressable population we can act on without building a model. Sending loyalty enrolment offers to the 1,386 unenrolled customers, particularly those who are also showing signs of recency drift, could meaningfully reduce churn.

**Investigations:** Why have 58% of customers never enrolled? Is the programme hard to find? Is the Silver tier benefit insufficiently compelling to drive sign-up? A short survey of recently-churned unenrolled customers would answer this directly.

---

## Finding 3: Paid Acquisition Channels Bring Structurally Higher-Churn Customers

**Customers acquired through Google Search (50.4%) and Instagram (49.9%) churn approximately 8–10 percentage points more than those acquired through Referral (42.2%) or Organic (39.8%).**

Paid channels attract price-sensitive customers who respond to an ad and try the product once. Word-of-mouth and organic search attract customers who sought us out because of genuine brand interest. Those customers stay.

This has direct implications for budget allocation and churn forecasting: if we grow heavily through paid channels in Q4, our churn base will structurally increase even if our absolute customer count rises.

**Investigations:** Do paid-channel customers respond differently to retention campaigns? Are they reachable via the campaigns we currently run, or do they require different messaging? Segmenting campaign performance by acquisition channel would answer this.

---

## Finding 4: Discount Dependency Is a Slow-Burn Churn Risk

Customers in the top quintile of discount usage (those who consistently buy at the highest discount rates) churn at 50.7%, compared to 45.2% for customers who rarely use discounts, a consistent 4–5 percentage point premium across every quintile.

This matters because **discount-heavy campaigns, while effective short-term, may be training a segment of customers to only purchase at a discount.** When we run a campaign without a discount, or when a customer's promotional window expires, they leave.

**Investigations:** What percentage of our campaigns have a discount component? For the discount-heavy customer segment, would free shipping, early access, or loyalty points produce better long-term retention than another 20% off?

---

## Data Quality Issues That Could Affect Campaign Targeting

Two data issues need to be resolved before running campaigns:

1. **Post-snapshot orders in the orders file**: There are 1,872 orders dated after 30 September in the data. These should not be used to evaluate or attribute any campaigns launched October onwards. Ensure the CRM and analytics team are filtering to pre-snapshot data only.

2. **12 duplicate order records**: Twelve order rows carry a `_DUP` suffix in the order ID. Before computing customer-level frequency or spend features (which feed the CRM priority scoring), these must be removed. They could overstate the activity level of 11 affected customers.

---

## Recommended Next Steps (Priority Order)

| Priority | Action | Rationale |
|---|---|---|
| 1 | Contact the 899 customers with recency > 90 days with a targeted win-back offer | Highest churn probability; still within reach |
| 2 | Launch loyalty enrolment drive for the 1,386 unenrolled customers with recency 30–90 days | Large addressable pool; proven loyalty-churn relationship |
| 3 | Investigate why `new_launch` campaigns show the highest post-campaign churn rate (51.0%) | Product-market fit issue or wrong targeting |
| 4 | Shift Q4 acquisition budget to Referral and Organic channels | Structurally lower churn; better long-term ROI |
| 5 | Run a post-purchase survey for churned discount-heavy customers | Understand whether price was the only loyalty driver |
