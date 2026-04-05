# Ecommerce-Sales-Analysis
SQL analysis of 100K+ orders on the Olist Brazilian e-commerce dataset - revenue trends, delivery performance, customer retention, and cancellation risk
# E-Commerce Sales Analysis (Olist Dataset)

![SQL](https://img.shields.io/badge/SQL-PostgreSQL-blue?style=flat-square)
![Domain](https://img.shields.io/badge/Domain-E--Commerce-teal?style=flat-square)
![Dataset](https://img.shields.io/badge/Dataset-Olist%20Brazil-coral?style=flat-square)
![Type](https://img.shields.io/badge/Type-BA%20Portfolio%20Project-purple?style=flat-square)

---

## Business Problem

Olist is a Brazilian e-commerce marketplace connecting thousands of small merchants to customers nationwide. With over 100,000 orders across multiple product categories, states, and payment methods, the business faces a common challenge: **the data exists, but the insights do not**.

This project uses SQL to answer the core business questions that drive e-commerce strategy:
- Where is revenue growing and where is it declining?
- Which states, categories, and payment methods matter most?
- How reliably are we delivering — and what does failure cost us?
- Are customers coming back, and how valuable are those who do?
- How much revenue are we losing to cancellations?

---

## Key Findings

| # | Finding | Business Impact |
|---|---------|----------------|
| 1 | **Credit card is the dominant payment method** (57% of orders) with avg 6.5 instalments | High instalment exposure — collection risk should be monitored monthly |
| 2 | **São Paulo generates ~30% of total revenue** despite being one of many states | Heavy geographic concentration — growth requires expanding other states |
| 3 | **Computers and electronics have the highest AOV** but also the highest freight costs | Margin erosion from shipping — free-shipping threshold review needed |
| 4 | **Repeat buyers spend 79% more** per customer than one-time buyers | Retention investment has clear ROI — re-engagement campaigns are justified |
| 5 | **14% cancellation rate** with high-instalment credit card orders most at risk | Revenue reporting must exclude cancellations; fraud monitoring needed |
| 6 | **Late delivery rate is 8.3%** with northern states showing worst performance | Logistics SLA review needed for northern fulfilment partners |

---

## Recommendations

**1. Build a customer re-engagement programme**
Repeat buyers generate nearly double the revenue of one-time buyers (Query 09). An automated email sequence triggered 30 days after first delivery — with a personalised discount on the customer's purchased category — can meaningfully increase repeat purchase rate.

**2. Investigate freight costs in computers and electronics**
Query 07 shows freight cost represents ~5% of item total in high-value categories. At scale, this is a significant margin drag. Negotiating category-specific logistics rates or setting a free-shipping threshold (e.g. orders above R$500) could improve unit economics.

**3. Set a monthly revenue-at-risk dashboard**
Query 10 shows cancelled orders carry significant value. Finance should track net revenue (delivered only) separately from GMV (all orders), and alert when cancellation rate exceeds 15% in any payment method segment.

**4. Expand marketing investment beyond São Paulo**
Query 05 shows SP dominates revenue share. States like MG, RJ, and RS have growing urban populations with strong e-commerce adoption — targeted performance marketing in these states can diversify revenue concentration risk.

**5. Review logistics SLAs for northern states**
Query 08 shows delivery delays cluster in specific states. Escalating with logistics partners and offering customers in high-delay states slightly longer (but more accurate) delivery estimates reduces negative reviews.

---

## SQL Query Dictionary

### Query 04 — Monthly revenue trend
**Business question:** How is revenue trending month over month?
**Key technique:** `LAG()` window function for MoM growth calculation

```sql
SELECT
    order_month_label AS order_month,
    COUNT(DISTINCT order_id) AS total_orders,
    ROUND(SUM(order_value), 2) AS total_revenue,
    ROUND(
        (SUM(order_value) - LAG(SUM(order_value),1) OVER (ORDER BY order_month))
        * 100.0 / NULLIF(LAG(SUM(order_value),1) OVER (ORDER BY order_month), 0)
    , 2) AS mom_growth_pct
FROM v_orders_clean
WHERE order_status = 'delivered'
GROUP BY order_month, order_month_label
ORDER BY order_month;
```

---

### Query 05 — Revenue by state
**Business question:** Which states drive the most revenue?
**Key technique:** `RANK()` and `SUM() OVER()` window functions

---

### Query 06 — Payment method analysis
**Business question:** How do customers pay, and what does it mean for revenue and risk?
**Key technique:** Dual window functions for order share and revenue share in one query

---

### Query 07 — Product category performance
**Business question:** Which categories generate the most revenue, and what is the freight burden?
**Key technique:** Multi-metric aggregation with freight ratio calculation

---

### Query 08 — Delivery performance
**Business question:** How reliably do we deliver on time, and where do we fail?
**Key technique:** Pre-calculated `is_late` binary flag enabling `AVG(is_late)` = late rate

---

### Query 09 — Repeat customer analysis
**Business question:** How valuable are repeat customers vs one-time buyers?
**Key technique:** CTE to calculate per-customer metrics before aggregation into segments

```sql
WITH customer_orders AS (
    SELECT customer_id,
           COUNT(DISTINCT order_id) AS order_count,
           ROUND(SUM(order_value), 2) AS total_spent
    FROM v_orders_clean
    WHERE order_status = 'delivered'
    GROUP BY customer_id
)
SELECT
    CASE WHEN order_count > 1 THEN 'Repeat buyer' ELSE 'One-time buyer' END AS customer_type,
    COUNT(*) AS total_customers,
    ROUND(AVG(total_spent), 2) AS avg_revenue_per_customer
FROM customer_orders
GROUP BY customer_type;
```

---

### Query 10 — Cancellation analysis
**Business question:** How much revenue is lost to cancellations, and which payment methods are highest risk?
**Key technique:** Conditional aggregation (`SUM(CASE WHEN...)`) to pivot status into columns

---

## Dataset

| Property | Detail |
|----------|--------|
| Source | [Olist Brazilian E-Commerce — Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) |
| Orders | ~100,000 orders (2016–2018) |
| Tables | 5 relational tables (orders, customers, products, payments, order items) |
| Key fields | Order status, purchase timestamp, payment type, product category, delivery dates, customer state |

---

## Tools & Skills

| Area | Detail |
|------|--------|
| Language | SQL (PostgreSQL) |
| Techniques | Multi-table JOINs, CTEs, window functions (`LAG`, `RANK`, `SUM OVER`), conditional aggregation, date functions, cohort analysis |
| Platform | DB Fiddle / pgAdmin |

---

## SQL Concepts Demonstrated

This project goes beyond the basics. Key advanced techniques used:

- **Window functions** — `LAG()` for month-over-month growth, `RANK()` for category ranking, `SUM() OVER()` for share calculations without subqueries
- **CTEs** — multi-step analysis broken into readable, testable logical blocks
- **Multi-table JOINs** — 3-table joins across orders, customers, products, and payments
- **Conditional aggregation** — `SUM(CASE WHEN...)` pattern to pivot row-level flags into column metrics
- **Date functions** — `DATE_TRUNC`, `DATE_PART`, timestamp arithmetic for delivery delay calculation

---

## Project Structure

```
Ecommerce-Sales-Analysis/
├── queries/
│   ├── 01_create_tables.sql
│   ├── 02_insert_data.sql
│   ├── 03_create_views.sql
│   ├── 04_monthly_revenue_trend.sql
│   ├── 05_revenue_by_state.sql
│   ├── 06_payment_method_analysis.sql
│   ├── 07_category_performance.sql
│   ├── 08_delivery_performance.sql
│   ├── 09_repeat_customer_analysis.sql
│   └── 10_cancellation_analysis.sql
├── data/
│   └── olist_sample.csv
├── reports/
│   └── insights_summary.md
├── README.md
└── LICENSE
```

---

*Part of my Business Analyst portfolio — [LinkedIn](https://www.linkedin.com/in/ps-p-322700240/) · [GitHub Profile](https://github.com/gctian64-del)*
