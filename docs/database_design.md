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

| Feature     | OLTP                              | OLAP                          |
| ----------- | --------------------------------- | ----------------------------- |
| Purpose     | Daily operations                  | Data analysis                 |
| Operations  | INSERT / UPDATE / DELETE          | SELECT (complex aggregations) |
| Query type  | Write-intensive                   | Read-intensive                |
| Query speed | Milliseconds                      | Seconds to minutes            |
| Data state  | Current                           | Historical                    |
| Schema      | Normalized                        | Denormalized                  |
| Users       | Many (thousands)                  | Few (analysts)                |
| Data size   | GB                                | TB / PB                       |
| Priority    | Quick and safer insertion of data | Quick queries for analytics   |

---

> 💡 **Key Insight:** Most real-world systems use **both** — OLTP for operations, and data is periodically moved to an OLAP system (a **Data Warehouse**) for analysis.

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

---

## Database views

A **view** is a **virtual table** defined by a stored SQL query. It doesn't store data itself — it pulls from the underlying base tables when queried.

### Standard View

A standard view is just a saved query. It always reflects the latest data but re-runs the query every time it's accessed.

```sql
-- Create the view
CREATE VIEW high_performers AS
SELECT e.employee_id, e.name, e.department, pr.score
FROM employees e
JOIN performance_review pr ON e.employee_id = pr.employee_id
WHERE pr.score >= 90;

-- Query it like a regular table
SELECT * FROM high_performers;

-- Drop the view
DROP VIEW high_performers;
```

### Materialized View

A materialized view **physically stores the query result**. It's much faster to read but can become stale — you need to refresh it manually or on a schedule.

```sql
-- Create the materialized view
CREATE MATERIALIZED VIEW dept_avg_scores AS
SELECT department, AVG(score) AS avg_score, COUNT(*) AS total_employees
FROM employees e
JOIN performance_review pr ON e.employee_id = pr.employee_id
GROUP BY department;

-- Query it (reads from stored data, not live tables)
SELECT * FROM dept_avg_scores
ORDER BY avg_score DESC;

-- Refresh when underlying data changes
REFRESH MATERIALIZED VIEW dept_avg_scores;

-- Drop the materialized view
DROP MATERIALIZED VIEW dept_avg_scores;
```

### Quick Comparison

| Feature        | Standard View              | Materialized View            |
| -------------- | -------------------------- | ---------------------------- |
| Data storage   | No (virtual)               | Yes (physically cached)      |
| Query speed    | Slower (re-runs each time) | Faster (reads stored data)   |
| Data freshness | Always up-to-date          | Can be stale until refreshed |
| Use case       | OLTP, simple abstractions  | OLAP, expensive aggregations |

### Why use views?

| Benefit        | Explanation                                             |
| -------------- | ------------------------------------------------------- |
| Simplification | Hide complex JOINs or aggregations behind a simple name |
| Reusability    | Write query logic once, reuse it across many queries    |
| Security       | Expose only specific columns/rows to certain users      |
| Abstraction    | Decouple app logic from underlying table structure      |

## Database management

### Database roles

A **role** is a named set of permissions that can be granted to users (or other roles). Instead of assigning privileges to each user individually, you assign them to a role and then grant that role to users.

#### Creating and assigning roles

```sql
-- Create roles
CREATE ROLE read_only;
CREATE ROLE analyst;
CREATE ROLE app_user;
CREATE ROLE admin WITH CREATEDB CREATEROLE;

-- Grant privileges to roles
GRANT SELECT ON ALL TABLES IN SCHEMA public TO read_only;

GRANT SELECT ON employees, performance_review TO analyst;
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA public TO analyst;

GRANT SELECT, INSERT, UPDATE, DELETE ON orders TO app_user;

-- Assign roles to users
GRANT read_only TO alice;
GRANT analyst  TO bob;
GRANT app_user TO backend_service;

-- Revoke a role from a user
REVOKE analyst FROM bob;

-- Drop a role
DROP ROLE read_only;
```

#### Role inheritance

Roles can inherit permissions from other roles, building a hierarchy.

```sql
-- analyst inherits everything read_only can do, plus more
CREATE ROLE read_only;
CREATE ROLE analyst;

GRANT read_only TO analyst;   -- analyst now inherits read_only's privileges

GRANT SELECT ON reports TO analyst;   -- analyst-only privilege

-- Now any user granted the analyst role also gets read_only permissions
GRANT analyst TO bob;
```

### Common role patterns

| Role        | Typical Privileges                             | Assigned to              |
| ----------- | ---------------------------------------------- | ------------------------ |
| `read_only` | `SELECT` on all tables                         | Reporting users, interns |
| `analyst`   | `SELECT` on specific tables + functions        | Data analysts            |
| `app_user`  | `SELECT, INSERT, UPDATE, DELETE` on app tables | Backend services         |
| `db_admin`  | Full privileges, `CREATE`, `DROP`, etc.        | DBAs only                |

> 💡 **Best practice:** Follow the **principle of least privilege** — grant each role only the permissions it actually needs, nothing more.

### Database partitioning

**Partitioning** is splitting a large table into smaller, more manageable pieces called **partitions**, while still appearing as a single table to queries. Each partition holds a subset of the data based on a defined rule.

#### Types of partitioning

**Horizontal partitioning** — splits by rows. Each partition has the same columns but a different subset of rows. Range, list, hash are all horizontal partitioning.

**Vertical partitioning** — splits by columns. Each partition holds a subset of columns for the same rows, linked by a shared key. Useful when some columns are large (e.g. blobs, JSON) or rarely accessed. Vertical partitioning is typically done manually through table design rather than a built-in PARTITION BY clause.

**1. Range Partitioning** — splits rows by a range of values (most common for dates)

```sql
CREATE TABLE orders (
    order_id   INT,
    order_date DATE,
    amount     NUMERIC
) PARTITION BY RANGE (order_date);

CREATE TABLE orders_2024 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

CREATE TABLE orders_2025 PARTITION OF orders
    FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
```

**2. List Partitioning** — splits rows by explicit discrete values

```sql
CREATE TABLE employees (
    employee_id INT,
    name        TEXT,
    region      TEXT
) PARTITION BY LIST (region);

CREATE TABLE employees_us PARTITION OF employees
    FOR VALUES IN ('US', 'CA');

CREATE TABLE employees_eu PARTITION OF employees
    FOR VALUES IN ('UK', 'DE', 'FR');
```

**3. Hash Partitioning** — distributes rows evenly using a hash function; useful when no natural range or category exists

```sql
CREATE TABLE sessions (
    session_id UUID,
    user_id    INT,
    data       TEXT
) PARTITION BY HASH (user_id);

CREATE TABLE sessions_0 PARTITION OF sessions
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE sessions_1 PARTITION OF sessions
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);

CREATE TABLE sessions_2 PARTITION OF sessions
    FOR VALUES WITH (MODULUS 4, REMAINDER 2);

CREATE TABLE sessions_3 PARTITION OF sessions
    FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

#### Sharding vs. partitioning

Sharding is horizontal partitioning across multiple physical database servers (nodes). Each shard is an independent database holding a slice of the data.

Where regular partitioning keeps all data on one server, sharding spreads it across many:

Users table (100M rows)
├── Shard 1 (server A) → user_id 1 – 25,000,000
├── Shard 2 (server B) → user_id 25M+1 – 50,000,000
├── Shard 3 (server C) → user_id 50M+1 – 75,000,000
└── Shard 4 (server D) → user_id 75M+1 – 100,000,000

|                | Partitioning          | Sharding                             |
| -------------- | --------------------- | ------------------------------------ |
| **Location**   | Single server         | Multiple servers                     |
| **Scales**     | Storage + query speed | Storage + throughput + compute       |
| **Complexity** | Low (built into DB)   | High (application/middleware level)  |
| **Joins**      | Easy (same DB)        | Hard (data is on different machines) |

#### Why partition?

| Benefit               | Explanation                                                        |
| --------------------- | ------------------------------------------------------------------ |
| **Query performance** | The planner scans only relevant partitions (**partition pruning**) |
| **Maintenance**       | Archive or drop old data by dropping a partition (instant)         |
| **Bulk loads**        | Load data into a partition, then attach it to the main table       |
| **Index efficiency**  | Smaller per-partition indexes fit better in memory                 |

> 💡 **Partition pruning:** When you query `WHERE order_date >= '2025-01-01'`, the database skips all other partitions entirely and only scans `orders_2025` — even though you queried the parent `orders` table.
