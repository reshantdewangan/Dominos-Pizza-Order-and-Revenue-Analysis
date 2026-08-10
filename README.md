# 🍕 Domino’s Pizza Store Sales & Operational Analysis (SQL)

An end-to-end relational database analysis project using PostgreSQL and SQL in VS Code to analyze **$817.8K+** in sales, peak ordering patterns, menu category performance, and customer ordering trends across **21,350 transactions**.

---

## 2. Project Overview

This project analyzes a full year of store transaction data (**21,350 orders** and **49,574 pizzas sold**) for a Domino’s Pizza outlet in 2015. The objective was to evaluate financial performance, understand peak operational hours, identify top and bottom-performing menu items, and optimize store operations.

The analysis was conducted using **PostgreSQL** by designing a 5-table relational database schema, enforcing referential integrity constraints, executing data cleaning scripts, and writing advanced SQL queries utilizing **CTEs**, **Window Functions**, **multi-table JOINs**, and **aggregations**. The final output provides actionable insights to improve store profitability, staffing allocation, and inventory management.

---

## 3. Business Problem

To support store managers and franchise owners in data-driven decision-making, this project addresses the following key business questions:

- **Revenue Optimization:** Which pizza categories and individual SKUs generate the highest revenue?
- **Operational Efficiency:** What are the peak ordering hours of the day and busiest days of the week for staffing?
- **Menu Engineering:** Which pizza sizes and types underperform and contribute minimally to sales?
- **Customer Demand:** What is the store's Average Order Value (AOV) and average pizzas per order?
- **Sales Trends:** How does revenue accumulate over time, and what are the monthly order growth patterns?

---

## 4. Dataset & Data Architecture

The dataset consists of **5 relational CSV files** covering store transactions from **January 1, 2015 to December 31, 2015** (358 active sales days).

### Dataset Summary

| Table Name | Record Count | Description | Key Fields |
|---|---|---|---|
| `orders` | 21,350 | Order transaction headers | `order_id`, `custid`, `order_date`, `order_time`, `status` |
| `order_details` | 48,620 | Line-item details per order | `order_details_id`, `order_id`, `pizza_id`, `quantity` |
| `pizzas` | 96 | Pizza SKU size and pricing schedule | `pizza_id`, `pizza_type_id`, `size`, `price` |
| `pizza_types` | 32 | Pizza metadata, categories & ingredients | `pizza_type_id`, `name`, `category`, `ingredients` |
| `customers` | 10 | Customer master profiles | `custid`, `first_name`, `last_name`, `email`, `phone`, `address` |

### Database Entity-Relationship (ER) Schema

- **Primary Keys:** `orders(order_id)`, `customers(custid)`, `order_details(order_details_id)`, `pizza_types(pizza_type_id)`, `pizzas(pizza_id)`
- **Foreign Keys:**
  - `orders.custid` ➔ `customers.custid`
  - `order_details.order_id` ➔ `orders.order_id`
  - `order_details.pizza_id` ➔ `pizzas.pizza_id`
  - `pizzas.pizza_type_id` ➔ `pizza_types.pizza_type_id`

---

## 5. Tools & Technologies

| Category | Tool / Technology |
|---|---|
| **Database Engine** | PostgreSQL |
| **Querying & Analysis** | SQL (Structured Query Language) |
| **Development Environment** | Visual Studio Code (VS Code) |
| **Database Management** | pgAdmin / PostgreSQL CLI |
| **Version Control** | Git & GitHub |
| **Data Storage Format** | CSV (Comma-Separated Values) |

---

## 6. Project Workflow

```
Raw CSV Datasets
       ↓
Relational Database Setup (PostgreSQL)
       ↓
Primary & Foreign Key Constraints
       ↓
Data Cleaning & Format Validation (Regex / Window Functions)
       ↓
Exploratory Data Analysis (EDA)
       ↓
Advanced SQL Business Analytics (CTEs, Window Functions, JOINs)
       ↓
KPI Computation & Business Insights
       ↓
Strategic Recommendations
```

1. **Database Modeling & DDL Setup:** Created tables with appropriate data types (`INTEGER`, `VARCHAR`, `NUMERIC`, `DATE`, `BIGINT`) and defined Primary/Foreign Key constraints.
2. **Data Cleaning & Validation:** Deduplicated customer profiles using `ROW_NUMBER()`, imputed missing phone numbers, and validated date formats using Regular Expressions (`!~`).
3. **Exploratory Data Analysis:** Analyzed total sales volume, revenue by category, pricing tiers, and temporal order distributions.
4. **Advanced Analytical Querying:** Used CTEs and Window Functions (`LAG`, `RANK`, `SUM() OVER()`) for cumulative sales, MoM growth, and category-level rankings.
5. **Insights & Recommendations:** Translated raw SQL outputs into structured business recommendations for management.

---

## 7. Data Cleaning & Preparation

Before conducting analysis, extensive data quality checks and cleaning routines were performed in PostgreSQL:

- **Referential Integrity Enforcement:** Added foreign key constraints to prevent orphan records across `orders`, `order_details`, `pizzas`, and `customers`.
- **Deduplication via Window Functions:** Removed duplicate customer records using `ROW_NUMBER() OVER (PARTITION BY email ORDER BY custid)`.
- **Null Value Imputation:** Handled missing contact records using `UPDATE customers SET phone = 0 WHERE phone IS NULL` and `UPDATE customers SET first_name = '-' WHERE phone IS NULL`.
- **Date Format Validation:** Applied Regex validation (`order_date !~ '^(19|20)\d\d-(0[1-9]|1[0-2])-(0[1-9]|[12][0-9]|3[01])$'`) to ensure standard ISO date formatting.
- **Quantity Validation:** Checked for zero or negative values in order details to preserve computational accuracy.

---

## 8. Data Analysis & SQL Techniques

### SQL Analytical Techniques Applied

#### 1. Multi-Table JOINs & Revenue Aggregation
Calculated total store revenue by joining order line items with pricing tables:
```sql
SELECT SUM(od.quantity * p.price) AS total_revenue
FROM order_details od
JOIN pizzas p ON p.pizza_id = od.pizza_id;
```

#### 2. Window Functions — Category Revenue Rankings
Ranked top 3 pizzas by revenue within each category using `RANK() OVER (PARTITION BY ...)`:
```sql
WITH pizza_revenue AS (
    SELECT 
        pt.name, 
        pt.category, 
        SUM(od.quantity * p.price) AS revenue,
        RANK() OVER(PARTITION BY pt.category ORDER BY SUM(od.quantity * p.price) DESC) AS rnk
    FROM order_details od
    JOIN pizzas p ON p.pizza_id = od.pizza_id
    JOIN pizza_types pt ON pt.pizza_type_id = p.pizza_type_id
    GROUP BY pt.name, pt.category
)
SELECT name, category, revenue, rnk
FROM pizza_revenue
WHERE rnk <= 3;
```

#### 3. Common Table Expressions (CTEs) & Lag Analysis
Calculated Month-over-Month (MoM) order growth using CTEs and `LAG()`:
```sql
WITH monthly_orders AS (
    SELECT 
        DATE_TRUNC('month', order_date) AS month,
        COUNT(order_id) AS order_count
    FROM orders 
    GROUP BY DATE_TRUNC('month', order_date)
)
SELECT 
    month,
    order_count,
    LAG(order_count) OVER (ORDER BY month) AS prev_month,
    ROUND(100.0 * (order_count - LAG(order_count) OVER (ORDER BY month)) / NULLIF(LAG(order_count) OVER (ORDER BY month), 0), 2) AS mom_growth_pct
FROM monthly_orders
ORDER BY month;
```

#### 4. Conditional Customer Segmentation (`CASE` Statements)
Segmented customers by purchasing tier:
```sql
WITH cust_spent AS (
    SELECT 
        c.custid, 
        SUM(od.quantity * p.price) AS total_spent
    FROM customers c
    JOIN orders o ON o.custid = c.custid
    JOIN order_details od ON od.order_id = o.order_id
    JOIN pizzas p ON p.pizza_id = od.pizza_id
    GROUP BY c.custid
)
SELECT 
    CASE WHEN total_spent > 50000 THEN 'High Value' ELSE 'Regular' END AS segment,
    COUNT(*) AS customer_count
FROM cust_spent
GROUP BY segment;
```

---

## 9. Key Business KPIs

| Key Performance Indicator | Exact Result |
|---|---|
| **Total Revenue** | **$817,860.05** |
| **Total Orders Placed** | **21,350** |
| **Total Pizzas Sold** | **49,574** |
| **Average Order Value (AOV)** | **$38.31** |
| **Average Pizzas Per Order** | **2.32 pizzas** |
| **Average Daily Pizza Sales** | **138.47 pizzas/day** |
| **Top Category by Revenue** | **Classic ($220,053.10 / 26.91%)** |
| **Top Revenue Pizza** | **The Thai Chicken Pizza ($43,434.25)** |
| **Most Popular Pizza Size** | **Large (L) (45.89% revenue / 18,956 units)** |
| **Busiest Day of Week** | **Friday (3,538 orders / $136,073.90)** |
| **Peak Ordering Hours** | **12:00 PM – 1:00 PM & 5:00 PM – 6:00 PM** |

---

## 10. Key Insights

### Insight 1 — Large Size (L) Dominates Store Revenue
- **Finding:** Large size pizzas generate nearly half of all store revenue.
- **Evidence:** Large (L) pizzas produced **$375,318.70 (45.89%)** of total revenue with **18,956 units** sold, followed by Medium (**$249,382.25, 30.49%**) and Small (**$178,076.50, 21.77%**). Extra Large (XL) and Double Extra Large (XXL) combined represent under 2% (**$15,082.60**).
- **Business Meaning:** Customer purchasing behavior is heavily centered on Large and Medium options. Carrying XL and XXL inventory creates storage complexity for minimal sales return.

### Insight 2 — Dual Daily Peak Windows During Lunch & Dinner
- **Finding:** Daily orders spike sharply twice per day during lunch and dinner hours.
- **Evidence:** The lunch rush peaks at **12:00 PM – 1:00 PM (2,520 and 2,455 orders)**, while the dinner rush peaks at **5:00 PM – 6:00 PM (2,336 and 2,399 orders)**. Orders drop significantly after 9:00 PM.
- **Business Meaning:** Kitchen prep, dough proofing, and shift scheduling must be pre-aligned for 11:30 AM and 4:30 PM to avoid service bottlenecks.

### Insight 3 — Friday & Thursday Drive Maximum Weekly Volume
- **Finding:** Order volume builds progressively through the week, peaking on Friday.
- **Evidence:** Friday produced the highest revenue (**$136,073.90 across 3,538 orders**), followed by Thursday (**$123,528.50, 3,239 orders**) and Saturday (**$123,182.40, 3,158 orders**). Sunday recorded the lowest volume (**$99,203.50, 2,624 orders**).
- **Business Meaning:** Delivery driver coverage and ingredient stocking must be scaled to maximum capacity from Thursday through Saturday.

### Insight 4 — Specialty Chicken Pizzas Generate Top Individual Revenue
- **Finding:** Specialty chicken offerings dominate the top revenue-generating menu items.
- **Evidence:** **The Thai Chicken Pizza** was the #1 revenue contributor (**$43,434.25, 5.31%**), followed by **The Barbecue Chicken Pizza** (**$42,768.00, 5.23%**) and **The California Chicken Pizza** (**$41,409.50, 5.06%**).
- **Business Meaning:** Premium chicken pizzas command higher price points ($20.75+ for L) and deliver stronger revenue yields per order.

### Insight 5 — Category Sales Distribution is Balanced
- **Finding:** Revenue is well-balanced across all 4 pizza categories, with Classic slightly in the lead.
- **Evidence:** Classic generated **$220,053.10 (26.91%)**, Supreme generated **$208,197.00 (25.46%)**, Chicken generated **$195,919.50 (23.96%)**, and Veggie generated **$193,690.45 (23.68%)**.
- **Business Meaning:** A balanced menu portfolio mitigates category risk, though Classic remains the volume leader (14,888 pizzas).

### Insight 6 — Underperforming Menu Items (Brie Carré & Green Garden)
- **Finding:** Select specialty pizzas generate disproportionately low revenue.
- **Evidence:** **The Brie Carré Pizza** generated only **$11,588.50 (1.42% share)** and **The Green Garden Pizza** generated **$13,955.75 (1.71% share)**.
- **Business Meaning:** Stocking unique ingredients for low-velocity pizzas increases food spoilage risks and inventory holding costs.

---

## 11. Business Recommendations

### Recommendation 1 — Optimize Peak Hour Shift Staffing
- **Action:** Schedule maximum kitchen prep staff and delivery drivers during peak windows: **11:30 AM – 2:00 PM** and **4:30 PM – 8:00 PM**, particularly Thursday through Saturday.
- **Reason:** Peak hour analysis shows over 45% of daily transactions occur within these two windows, led by Friday's 3,538 orders.
- **Expected Impact:** Could help reduce order fulfillment times, increase kitchen throughput, and improve delivery SLA compliance.

### Recommendation 2 — Rationalize Low-Velocity Menu Items & XXL Sizes
- **Action:** Evaluate replacing bottom-tier items like **The Brie Carré Pizza** ($11.5K revenue) and phase out XXL size SKUs.
- **Reason:** XXL sizes accounted for only 28 total sales (0.12% revenue) across the entire year, yet require dedicated storage boxes and dough sizing.
- **Expected Impact:** May reduce ingredient waste, lower carrying costs, and simplify kitchen assembly lines.

### Recommendation 3 — Promote Combo Bundles to Elevate Average Order Value (AOV)
- **Action:** Introduce promotional combos (e.g., "Large Specialty Chicken Pizza + Side + Beverage") structured above the **$38.31 AOV** baseline.
- **Reason:** Average order size is currently 2.32 pizzas per order, with Large pizzas driving 45.89% of revenue.
- **Expected Impact:** Could support raising the Average Order Value from $38.31 toward $45.00+ by encouraging complementary add-on purchases.

### Recommendation 4 — Launch Targeted Sunday/Monday Promotions
- **Action:** Introduce "Sunday Family Specials" or "Monday Mid-Week Slice Deals".
- **Reason:** Sunday generated the lowest revenue of the week ($99,203.50 vs Friday's $136,073.90).
- **Expected Impact:** Could help smooth weekly demand, boosting store capacity utilization on slower days.

---

## 12. Dashboard & Visualization

The project analysis supports an interactive store analytics dashboard focusing on operational KPIs and product hierarchy:

```
+-----------------------------------------------------------------------+
|                        DOMINO'S PIZZA DASHBOARD                       |
+-------------------+-------------------+-------------------+-----------+
| Total Revenue     | Total Orders      | Avg Order Value   | Total Qty |
|   $817,860.05     |     21,350        |      $38.31       |  49,574   |
+-------------------+-------------------+-------------------+-----------+
| [Peak Hours Heatmap]                  | [Category Revenue Share]      |
| 12 PM - 1 PM (Lunch Peak)             | Classic (26.91%)  Supreme(25.5%)|
| 5 PM - 6 PM (Dinner Peak)             | Chicken (23.96%)  Veggie (23.7%)|
+---------------------------------------+-------------------------------+
| [Top 5 Revenue Pizzas]                | [Revenue by Pizza Size]       |
| 1. Thai Chicken      ($43.4K)         | Large (L)       45.89%        |
| 2. BBQ Chicken       ($42.8K)         | Medium (M)      30.49%        |
| 3. California Chicken($41.4K)         | Small (S)       21.77%        |
+---------------------------------------+-------------------------------+
```

![Dashboard Preview](images/dashboard.png)  
*(Note: Replace with actual dashboard screenshot path when deployed)*

---

## 13. Project Structure

```
Domino-s-Pizza-Store-Analysis-SQL-Project/
│
├── data/                               # Raw CSV Transaction Datasets
│   ├── customers.csv                   # Customer master records (10 rows)
│   ├── orders.csv                      # Transaction headers (21,350 rows)
│   ├── order_details.csv               # Line-item details (48,620 rows)
│   ├── pizza_types.csv                 # Pizza metadata & categories (32 rows)
│   └── pizzas.csv                      # Pizza SKUs & prices (96 rows)
│
├── sql/                                # Database Setup & Business Queries
│   ├── Create Table and Constraints.md # Schema DDL, PKs & Foreign Keys
│   └── SQL Queries in Visual Studio Code.md # Cleaning & analytical SQL queries
│
├── docs/                               # Conceptual Guides & Notes
│   └── Steps.sql                       # Data types, cleaning & EDA workflow guide
│
├── Dominos Extra/                      # Database Backup & Extra Notes
│   ├── dominos_db.sql                  # PostgreSQL database dump file
│
└── README.md                           # Main Project README Documentation
```

---

## 14. How to Run & Reproduce the Project

### Prerequisites
- PostgreSQL 12+ installed
- Visual Studio Code (with SQL Extension) or pgAdmin 4
- Git installed on your local machine

### Step-by-Step Execution

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/your-username/Domino-s-Pizza-Store-Analysis-SQL-Project.git
   cd Domino-s-Pizza-Store-Analysis-SQL-Project
   ```

2. **Create PostgreSQL Database:**
   ```sql
   CREATE DATABASE dominos_db;
   ```

3. **Execute Schema & Constraints Setup:**
   - Open `Create Table and Constraints.md` in VS Code or pgAdmin.
   - Execute table creation scripts and primary/foreign key constraint statements.

4. **Import Data:**
   - Import `customers.csv`, `orders.csv`, `order_details.csv`, `pizza_types.csv`, and `pizzas.csv` into their corresponding database tables.

5. **Run Data Cleaning Queries:**
   - Execute deduplication and format validation queries from `SQL Queries in Visual Studio Code.md`.

6. **Run Analytical Queries:**
   - Execute the 20 structured analytical queries in `SQL Queries in Visual Studio Code.md` to compute store KPIs and performance rankings.

---

## 15. Skills Demonstrated

### Technical Skills
- **Relational Database Design:** Schema modeling, primary keys, foreign key constraints, referential integrity.
- **Advanced SQL Querying:** Multi-table `JOIN`s, `GROUP BY`, `HAVING`, Subqueries, Nested Aggregations.
- **Window Functions:** `ROW_NUMBER()`, `RANK()`, `LAG()`, `SUM() OVER ()`.
- **Common Table Expressions (CTEs):** Modular query structuring using `WITH` clauses.
- **Data Validation & Cleaning:** Imputation, regular expression pattern matching (`!~`), deduplication.

### Analytical & Business Skills
- **Key Performance Indicator (KPI) Analysis:** Revenue, Average Order Value (AOV), throughput per day.
- **Product & Category Analysis:** Share of wallet, SKU velocity ranking, menu rationalization.
- **Operational & Temporal Analytics:** Hourly peak-load analysis, day-of-week distribution.
- **Business Strategy:** Translating SQL outputs into actionable operational recommendations.

---

## 16. Key Learnings

Completing this project provided several practical insights into real-world data analytics:

1. **Importance of Schema Constraints:** Defining strict foreign keys early prevents orphan records and ensures data consistency across multi-table JOINs.
2. **Analytical Power of Window Functions:** Using `LAG()` for MoM trends and `RANK() OVER (PARTITION BY ...)` enabled category-level comparative analytics without needing temporary tables.
3. **Operational Impact of Data Analytics:** Translating hourly order distributions into staffing recommendations demonstrated how data analytics directly drives store operational efficiency.
4. **Data Validation Realities:** Real-world datasets often require date format validation via regex and field cleaning before performing calculations.

---

## 17. Conclusion

This project demonstrates an end-to-end data analysis workflow, turning raw transaction CSVs into strategic business intelligence using PostgreSQL and SQL. By analyzing **$817.8K+ in total revenue** across **21,350 orders**, the analysis uncovered vital patterns—such as the revenue dominance of Large pizzas (45.89%), distinct lunch and dinner demand rushes, and top-performing specialty chicken offerings. The derived recommendations offer store management a clear, data-driven roadmap to enhance peak-hour fulfillment, streamline underperforming SKUs, and maximize overall store profitability.
