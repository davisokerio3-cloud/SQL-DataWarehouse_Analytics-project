# 📊 Data Warehouse Analytics — SQL Analysis Project

![SQL](https://img.shields.io/badge/SQL-Server-blue?style=flat&logo=microsoftsqlserver)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![License](https://img.shields.io/badge/License-MIT-yellow)

## 📖 Project Overview

This project is a comprehensive **SQL-based analytics solution** built on top of a gold layer data warehouse. Using three business-ready tables — `dim_customers`, `dim_products` and `fact_sales` — it demonstrates a complete suite of analytical techniques used by professional data analysts and business intelligence teams in real organisations.

The analysis covers everything from basic data exploration through to advanced segmentation, performance benchmarking and consolidated business reporting — all written in pure SQL using window functions, CTEs, aggregations and CASE-based logic.

---

## 🎯 What This Project Demonstrates

- Writing professional analytical SQL queries from scratch
- Applying window functions for ranking, running totals and period comparisons
- Building customer and product segmentation using RFM methodology
- Measuring business performance across time, geography and product categories
- Producing consolidated reports suitable for management dashboards

---

## 🗂️ Repository Structure

```
DataWarehouseAnalytics/
│
├── 📁 datasets/
│   ├── gold_dim_customers.csv       # Customer dimension table
│   ├── gold_dim_products.csv        # Product dimension table
│   └── gold_fact_sales.csv          # Sales fact table
│
├── 📁 analytics/
│   ├── 01_database_exploration.sql
│   ├── 02_dimension_exploration.sql
│   ├── 03_date_exploration.sql
│   ├── 04_magnitude_analysis.sql
│   ├── 05_ranking_analysis.sql
│   ├── 06_change_over_time.sql
│   ├── 07_cumulative_analysis.sql
│   ├── 08_performance_analysis.sql
│   ├── 09_part_to_whole_analysis.sql
│   ├── 10_data_segmentation.sql
│   └── 11_reporting.sql
│
└── README.md
```

---

## 📦 Dataset Overview

| Table | Rows | Description |
|---|---|---|
| gold_dim_customers | 18,484 | Customer profiles — name, country, gender, marital status, birthdate |
| gold_dim_products | 295 | Product catalogue — name, category, subcategory, product line, cost |
| gold_fact_sales | 60,398 | All sales transactions — order number, dates, revenue, quantity, price |

### Data Model — Star Schema

```
          dim_customers                    dim_products
         ┌───────────────┐               ┌───────────────┐
         │ customer_key  │               │ product_key   │
         │ customer_id   │               │ product_id    │
         │ customer_num  │               │ product_name  │
         │ first_name    │               │ category      │
         │ last_name     │               │ subcategory   │
         │ country       │               │ product_line  │
         │ gender        │               │ cost          │
         │ marital_status│               │ maintenance   │
         │ birthdate     │               │ start_date    │
         │ create_date   │               └───────┬───────┘
         └───────┬───────┘                       │
                 │                               │
                 └──────────────┬────────────────┘
                                │
                       ┌────────▼────────┐
                       │   fact_sales    │
                       │ order_number    │
                       │ product_key  FK │
                       │ customer_key FK │
                       │ order_date      │
                       │ shipping_date   │
                       │ due_date        │
                       │ sales_amount    │
                       │ quantity        │
                       │ price           │
                       └─────────────────┘
```

---

## 🔍 Analytics Covered

### 1. 🗄️ Database Exploration
Understanding what exists in the database before doing any analysis — exploring tables, columns, data types and row counts.

```sql
SELECT TABLE_SCHEMA, TABLE_NAME, TABLE_TYPE
FROM INFORMATION_SCHEMA.TABLES
ORDER BY TABLE_SCHEMA, TABLE_NAME
```

---

### 2. 📐 Dimension Exploration
Exploring the unique values across key dimensions to understand the full scope of the business.

```sql
SELECT DISTINCT country, COUNT(*) AS customer_count
FROM [dbo].[gold.dim_customers]
GROUP BY country
ORDER BY customer_count DESC
```

**Findings:** 6 countries · 3 product categories · 4 product lines

---

### 3. 📅 Date Exploration
Understanding the time range covered by the data and identifying peak trading periods.

```sql
SELECT
    MIN(order_date)                                     AS first_order_date,
    MAX(order_date)                                     AS last_order_date,
    DATEDIFF(YEAR, MIN(order_date), MAX(order_date))    AS years_of_data
FROM [dbo].[gold.fact_sales]
```

**Findings:** Data spans 2010 to 2014 · Peak year was 2013

---

### 4. 📏 Magnitude Analysis
The headline KPIs — total revenue, orders, customers and units that define the scale of the business.

```sql
SELECT
    SUM(sales_amount)                       AS total_revenue,
    COUNT(DISTINCT order_number)            AS total_orders,
    COUNT(DISTINCT customer_key)            AS total_customers,
    SUM(quantity)                           AS total_units_sold,
    ROUND(SUM(sales_amount) * 1.0
        / COUNT(DISTINCT order_number), 2)  AS avg_order_value
FROM [dbo].[gold.fact_sales]
```

| KPI | Value |
|---|---|
| Total Revenue | $29,356,250 |
| Total Orders | 27,659 |
| Total Customers | 18,484 |
| Total Units Sold | 60,423 |
| Average Order Value | $1,061 |

---

### 5. 🏆 Ranking Analysis
Using RANK() and ROW_NUMBER() to find top and bottom performers across products, customers and countries.

```sql
SELECT
    p.product_name,
    SUM(f.sales_amount)                             AS total_revenue,
    RANK() OVER (ORDER BY SUM(f.sales_amount) DESC) AS revenue_rank
FROM [dbo].[gold.fact_sales] f
JOIN [dbo].[gold.dim_products] p ON f.product_key = p.product_key
GROUP BY p.product_name
ORDER BY revenue_rank
```

**Top product:** Mountain-200 Black 46 at $1,373,454 · **Top customer:** Jordan Turner at $15,998

---

### 6. 📈 Change Over Time Analysis
Tracking how metrics evolve period by period using LAG() to compare against the previous year and calculate growth rates.

```sql
SELECT
    YEAR(order_date)                                AS yr,
    SUM(sales_amount)                               AS current_revenue,
    LAG(SUM(sales_amount)) OVER (
        ORDER BY YEAR(order_date)
    )                                               AS prev_year_revenue,
    ROUND(
        100.0 * (SUM(sales_amount) -
            LAG(SUM(sales_amount)) OVER (ORDER BY YEAR(order_date)))
        / NULLIF(LAG(SUM(sales_amount)) OVER (
            ORDER BY YEAR(order_date)), 0)
    , 1)                                            AS yoy_growth_pct
FROM [dbo].[gold.fact_sales]
GROUP BY YEAR(order_date)
ORDER BY yr
```

| Year | Revenue | YoY Growth |
|---|---|---|
| 2011 | $7,075,088 | +16,194% |
| 2012 | $5,842,231 | -17.4% |
| 2013 | $16,344,878 | +179.7% |

---

### 7. 📊 Cumulative Analysis
Building running totals using SUM() window functions to reveal how revenue accumulated from the very first transaction.

```sql
SELECT
    YEAR(order_date)                                AS yr,
    MONTH(order_date)                               AS mth,
    SUM(sales_amount)                               AS monthly_revenue,
    SUM(SUM(sales_amount)) OVER (
        ORDER BY YEAR(order_date), MONTH(order_date)
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    )                                               AS cumulative_revenue,
    SUM(SUM(sales_amount)) OVER (
        PARTITION BY YEAR(order_date)
        ORDER BY MONTH(order_date)
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    )                                               AS ytd_revenue
FROM [dbo].[gold.fact_sales]
GROUP BY YEAR(order_date), MONTH(order_date)
ORDER BY yr, mth
```

---

### 8. ⚡ Performance Analysis
Benchmarking every product and customer against the average to identify stars, underperformers and at-risk segments.

```sql
SELECT
    p.product_name,
    SUM(f.sales_amount)                             AS product_revenue,
    AVG(SUM(f.sales_amount)) OVER ()                AS avg_product_revenue,
    CASE
        WHEN SUM(f.sales_amount) >=
             AVG(SUM(f.sales_amount)) OVER () * 2   THEN 'Star'
        WHEN SUM(f.sales_amount) >=
             AVG(SUM(f.sales_amount)) OVER ()       THEN 'Above Average'
        WHEN SUM(f.sales_amount) >=
             AVG(SUM(f.sales_amount)) OVER () * 0.5 THEN 'Below Average'
        ELSE                                             'Underperformer'
    END                                             AS performance_segment
FROM [dbo].[gold.fact_sales] f
JOIN [dbo].[gold.dim_products] p ON f.product_key = p.product_key
GROUP BY p.product_name, p.category
ORDER BY product_revenue DESC
```

---

### 9. 🥧 Part-to-Whole Analysis
Calculating each segment's percentage share of the total to reveal where the business is concentrated.

```sql
SELECT
    p.category,
    SUM(f.sales_amount)                             AS category_revenue,
    ROUND(
        100.0 * SUM(f.sales_amount)
        / SUM(SUM(f.sales_amount)) OVER ()
    , 2)                                            AS pct_of_total,
    ROUND(
        100.0 * SUM(SUM(f.sales_amount)) OVER (
            ORDER BY SUM(f.sales_amount) DESC
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        )
        / SUM(SUM(f.sales_amount)) OVER ()
    , 1)                                            AS cumulative_pct
FROM [dbo].[gold.fact_sales] f
JOIN [dbo].[gold.dim_products] p ON f.product_key = p.product_key
GROUP BY p.category
ORDER BY category_revenue DESC
```

| Category | Revenue | % of Total | Cumulative % |
|---|---|---|---|
| Bikes | $28,316,272 | 96.46% | 96.46% |
| Accessories | $700,262 | 2.39% | 98.85% |
| Clothing | $339,716 | 1.16% | 100.00% |

---

### 10. 🎯 Data Segmentation
Grouping customers and products into meaningful tiers — from simple spending bands through to full RFM (Recency, Frequency, Monetary) segmentation.

```sql
WITH customer_spending AS (
    SELECT
        customer_key,
        SUM(sales_amount)            AS total_revenue,
        COUNT(DISTINCT order_number) AS total_orders,
        DATEDIFF(DAY, MAX(order_date),
            (SELECT MAX(order_date)
             FROM [dbo].[gold.fact_sales])) AS recency_days
    FROM [dbo].[gold.fact_sales]
    GROUP BY customer_key
)
SELECT
    c.first_name + ' ' + c.last_name   AS customer_name,
    cs.total_revenue,
    CASE
        WHEN cs.total_revenue >= 10000  THEN 'VIP'
        WHEN cs.total_revenue >= 5000   THEN 'High Value'
        WHEN cs.total_revenue >= 1000   THEN 'Mid Value'
        ELSE                                 'Low Value'
    END                                AS spending_segment,
    CASE
        WHEN cs.recency_days <= 90      THEN 'Active'
        WHEN cs.recency_days <= 365     THEN 'Lapsing'
        ELSE                                 'Inactive'
    END                                AS activity_status
FROM customer_spending cs
JOIN [dbo].[gold.dim_customers] c ON cs.customer_key = c.customer_key
ORDER BY cs.total_revenue DESC
```

| Segment | Customers | % of Customers | Revenue | % of Revenue |
|---|---|---|---|---|
| VIP | 102 | 0.6% | $1,421,832 | 4.8% |
| High Value | 489 | 2.6% | $3,182,291 | 10.8% |
| Mid Value | 3,842 | 20.8% | $8,931,044 | 30.4% |
| Low Value | 14,051 | 76.0% | $15,821,083 | 53.9% |

---

### 11. 📋 Reporting
Consolidated single-row-per-entity reports combining all key metrics — designed for business users, sales teams and management.

```sql
-- Customer Consolidated Report — one complete row per customer
WITH customer_orders AS (
    SELECT
        customer_key,
        SUM(sales_amount)            AS total_revenue,
        COUNT(DISTINCT order_number) AS total_orders,
        MIN(order_date)              AS first_order_date,
        MAX(order_date)              AS last_order_date,
        DATEDIFF(MONTH,
            MIN(order_date),
            MAX(order_date))         AS lifespan_months
    FROM [dbo].[gold.fact_sales]
    GROUP BY customer_key
)
SELECT
    c.first_name + ' ' + c.last_name               AS customer_name,
    c.country,
    c.gender,
    o.total_revenue,
    o.total_orders,
    o.lifespan_months,
    ROUND(o.total_revenue
        / NULLIF(o.total_orders, 0), 2)             AS avg_order_value,
    CASE
        WHEN o.total_revenue >= 10000 THEN 'VIP'
        WHEN o.total_revenue >= 5000  THEN 'High Value'
        WHEN o.total_revenue >= 1000  THEN 'Mid Value'
        ELSE                               'Low Value'
    END                                             AS customer_segment,
    RANK() OVER (
        ORDER BY o.total_revenue DESC
    )                                               AS revenue_rank
FROM customer_orders o
JOIN [dbo].[gold.dim_customers] c ON o.customer_key = c.customer_key
ORDER BY o.total_revenue DESC
```

---

## 💡 Key Business Insights

| Area | Insight |
|---|---|
| Revenue scale | $29.4M total revenue across 27,659 orders |
| Category dominance | Bikes drive 96.5% of all revenue |
| Top two markets | USA and Australia nearly tied — 62% of total revenue combined |
| Peak year | 2013 delivered $16.3M — 55.7% of all-time revenue |
| Best market efficiency | Australia earns $2,523 per customer vs $1,172 for USA |
| Top product | Mountain-200 Black 46 at $1.37M lifetime revenue |
| Top customer | Jordan Turner at $15,998 lifetime spend |
| Gender split | Virtually equal — Female 50.4% vs Male 49.6% |
| Unit economics | Road bikes earn $2,520 per unit vs $14 for accessories |
| VIP power | 0.6% of customers generate 4.8% of revenue |

---

## 🛠️ SQL Concepts Used

| Concept | Applied In |
|---|---|
| `SUM() OVER()` | Part-to-whole, Cumulative analysis |
| `AVG() OVER()` | Performance benchmarking |
| `RANK()` `ROW_NUMBER()` `DENSE_RANK()` | Ranking analysis |
| `LAG()` `LEAD()` | Change over time |
| `NTILE()` | RFM customer scoring |
| `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` | Running totals |
| `PARTITION BY` | Country ranking, YTD resets |
| CTEs | All analytics scripts |
| `CASE` statements | Segmentation and performance flags |
| `NULLIF()` | Safe division |
| `DATEDIFF()` | Recency and lifespan calculations |
| `GROUP BY` + `HAVING` | Aggregations and filters |
| `INFORMATION_SCHEMA` | Database exploration |

---

## ▶️ How to Run

### Prerequisites
- SQL Server 2019 or later
- SQL Server Management Studio (SSMS)

### Setup

```sql
-- Step 1: Create and select the database
CREATE DATABASE DataWarehouseAnalytics;
GO
USE DataWarehouseAnalytics;
GO
```

Import each CSV file into SQL Server using SSMS:
Right click database → Tasks → Import Flat File → follow the wizard for each of the three CSV files.

Then open any file from the `/analytics/` folder in SSMS and press **F5** to run it.

Run them in order 01 through 11 — each builds on the previous.

---

## 🚀 Future Enhancements

- [ ] Connect to Power BI for visual dashboards
- [ ] Add date dimension for richer time intelligence
- [ ] Build saved views on top of reporting queries
- [ ] Add profit margin analysis using product cost data
- [ ] Implement customer churn prediction scoring

---

## 👤 Author

**Davis Okerio**
BSc Information Science — Technical University of Kenya
CPA (K) | CIFA Level II Candidate
Data analyst and Data engineer


## 📄 License

This project is licensed under the MIT License.
