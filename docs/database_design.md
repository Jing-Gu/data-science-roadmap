# Database design

## Data processing

2 approaches to data processing: OLTP and OLAP

## 1. OLTP — Online Transaction Processing

OLTP systems are designed for **day-to-day operational tasks** — inserting, updating, and deleting records in real time.

### Characteristics

- Manages a **large number of short, fast queries**
- Optimized for **INSERT, UPDATE, DELETE** operations
- Data is **current and up-to-date**
- **Highly normalized** schema (3NF) to reduce redundancy
- Many concurrent users performing small transactions

### Examples

- Banking systems
- E-commerce order placement
- Hotel booking systems

### Example Query

```sql
INSERT INTO orders (user_id, product_id, quantity)
VALUES (42, 101, 2);
```

---

## 2. OLAP — Online Analytical Processing

OLAP systems are designed for **analysis, reporting, and decision-making** — reading large volumes of historical data.

### Characteristics

- Handles **few but complex, long-running queries**
- Optimized for **SELECT / aggregation** operations
- Data is **historical** (days, months, years old)
- **Denormalized** schema (Star/Snowflake schema) for query speed
- Few concurrent users, typically analysts or BI tools

### Examples

- Sales trend analysis
- Financial forecasting
- Business intelligence dashboards

### Example Query

```sql
SELECT region, SUM(sales_amount)
FROM fact_sales
WHERE year = 2025
GROUP BY region;
```

---

## Quick Comparison

| Feature     | OLTP                               | OLAP                          |
| ----------- | ---------------------------------- | ----------------------------- |
| Purpose     | Daily operations                   | Data analysis                 |
| Operations  | INSERT / UPDATE / DELETE           | SELECT (complex aggregations) |
| Query type  | Write-intensive                    | Read-intensive                |
| Query speed | Milliseconds                       | Seconds to minutes            |
| Data state  | Current                            | Historical                    |
| Schema      | Normalized                         | Denormalized                  |
| Users       | Many (thousands)                   | Few (analysts)                |
| Data size   | GB                                 | TB / PB                       |
| Priority    | Quick and safer inseration of data | quick queries for analytics   |

---

> 💡 **Key Insight:** Most real-world systems use **both** — OLTP for operations,
> and data is periodically moved to an OLAP system (a **Data Warehouse**) for analysis.

---

## Data schema and Normalization

### Normalization

Normalization is the process of organizing a database to reduce redundancy and improve data integrity by splitting data into related tables and defining relationships between them.

It follows a series of rules called Normal Forms (1NF → 2NF → 3NF → BCNF...).

#### OLTP → Normalize (3NF or higher)

Because data is constantly being written and modified. Consistency and integrity is important. A redundant column in an OLTP system is a bug waiting to happen.

#### OLAP → Denormalize (Star / Snowflake Schema)

Because analysts run massive aggregation queries across millions of rows. JOINing 10 normalized tables on every query is slow. Denormalized flat tables (or star schemas with a central fact table) are much faster to query. Data integrity matters less here because OLAP data is usually read-only, loaded from OLTP via an ETL pipeline.

### Star Schema vs Snowflake Schema

Both are used in **OLAP / Data Warehouses**. They organize data around a central **Fact Table** surrounded by **Dimension Tables**.

| Term                | Description                                                              |
| ------------------- | ------------------------------------------------------------------------ |
| **Fact Table**      | Contains measurable, quantitative data (sales amount, quantity, revenue) |
| **Dimension Table** | Contains descriptive context (who, what, where, when)                    |

#### Star Schema ⭐

The fact table sits in the center, directly connected to **denormalized** dimension tables.
Dimension tables are **flat** — no further splitting.

#### Snowflake Schema ❄️

An extension of Star Schema where dimension tables are further **normalized** into sub-dimensions.
Looks like a snowflake because of the branching structure.

> 💡 **In practice**, Star Schema is more commonly used in data warehouses because **query speed is the top priority** in OLAP systems.
