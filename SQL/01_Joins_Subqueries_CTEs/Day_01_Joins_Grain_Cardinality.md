# Day 01 — SQL Joins, Data Grain & Cardinality

**Date:** 2026-10-14  
**Status:** In progress — hands-on challenge pending  
**Database:** PostgreSQL  
**Target level:** Senior Data Analyst / BI Analyst

## Learning objectives
1. Identify the grain of every table before joining.
2. Recognize one-to-one, one-to-many and many-to-many relationships.
3. Explain the behavior of INNER JOIN and LEFT JOIN.
4. Prevent incorrect metrics caused by join fanout.
5. Explain why `COUNT(DISTINCT ...)` can fix counts but not a duplicated `SUM`.

## 1. Key concepts

**Data grain** is what one row represents:
- `sql_lab_customers`: one row per customer.
- `sql_lab_orders`: one row per order.
- `sql_lab_payments`: one row per payment transaction.

**Cardinality** is how rows relate between tables:
- One customer can have many orders.
- One order can have many payments.
- A customer can have zero orders.

**Join fanout** occurs when a row on one side matches multiple rows on the other side. If an order matches two payment rows, its order amount appears twice after joining. Aggregating order amounts *after* that join can inflate revenue.

## 2. Reproducible practice dataset

```sql
DROP TABLE IF EXISTS sql_lab_payments;
DROP TABLE IF EXISTS sql_lab_orders;
DROP TABLE IF EXISTS sql_lab_customers;

CREATE TABLE sql_lab_customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(50)
);

CREATE TABLE sql_lab_orders (
    order_id INT PRIMARY KEY,
    customer_id INT REFERENCES sql_lab_customers(customer_id),
    order_date DATE,
    amount NUMERIC(12,2)
);

CREATE TABLE sql_lab_payments (
    payment_id INT PRIMARY KEY,
    order_id INT REFERENCES sql_lab_orders(order_id),
    paid_amount NUMERIC(12,2)
);

INSERT INTO sql_lab_customers VALUES
(101, 'Asha'),
(102, 'Bharat'),
(103, 'Charu'),
(104, 'Dev');

INSERT INTO sql_lab_orders VALUES
(1001, 101, '2026-09-01', 1200),
(1002, 101, '2026-09-10', 800),
(1003, 102, '2026-09-12', 1500),
(1004, 104, '2026-09-20', 500);

INSERT INTO sql_lab_payments VALUES
(5001, 1001, 600),
(5002, 1001, 600),
(5003, 1002, 800),
(5004, 1003, 1000),
(5005, 1004, 300),
(5006, 1004, 100);
```

### Source data

**Customers**

| customer_id | customer_name |
|---:|---|
| 101 | Asha |
| 102 | Bharat |
| 103 | Charu |
| 104 | Dev |

**Orders**

| order_id | customer_id | amount |
|---:|---:|---:|
| 1001 | 101 | 1200 |
| 1002 | 101 | 800 |
| 1003 | 102 | 1500 |
| 1004 | 104 | 500 |

**Payments**

| payment_id | order_id | paid_amount |
|---:|---:|---:|
| 5001 | 1001 | 600 |
| 5002 | 1001 | 600 |
| 5003 | 1002 | 800 |
| 5004 | 1003 | 1000 |
| 5005 | 1004 | 300 |
| 5006 | 1004 | 100 |

## 3. Example: INNER JOIN

**Question:** Show customers with orders, one row per order.

```sql
SELECT c.customer_id, c.customer_name, o.order_id, o.amount
FROM sql_lab_customers AS c
INNER JOIN sql_lab_orders AS o
  ON c.customer_id = o.customer_id
ORDER BY c.customer_id, o.order_id;
```

**Expected output**

| customer_id | customer_name | order_id | amount |
|---:|---|---:|---:|
| 101 | Asha | 1001 | 1200 |
| 101 | Asha | 1002 | 800 |
| 102 | Bharat | 1003 | 1500 |
| 104 | Dev | 1004 | 500 |

**Why?** Charu has no orders, so she does not appear. Asha appears twice because she has two orders.

## 4. Example: LEFT JOIN with aggregation

**Question:** Include all customers and report order count and order revenue.

```sql
SELECT
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS total_orders,
    COALESCE(SUM(o.amount), 0) AS total_revenue
FROM sql_lab_customers AS c
LEFT JOIN sql_lab_orders AS o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
ORDER BY c.customer_id;
```

**Expected output**

| customer_id | customer_name | total_orders | total_revenue |
|---:|---|---:|---:|
| 101 | Asha | 2 | 2000 |
| 102 | Bharat | 1 | 1500 |
| 103 | Charu | 0 | 0 |
| 104 | Dev | 1 | 500 |

**Senior-level notes**
- `COUNT(o.order_id)` returns 0 for the unmatched customer; `COUNT(*)` would count the preserved LEFT JOIN row and return 1.
- `COALESCE(SUM(...),0)` returns 0 when no matching order amounts exist.
- Aggregate at the grain of the metric. A customer-level summary is not permission to join multiple lower-grain tables and sum blindly.

## 5. Failure mode: revenue inflation after joining payments

**Incorrect SQL — intentionally included for debugging**

```sql
SELECT
    c.customer_id,
    COUNT(DISTINCT o.order_id) AS total_orders,
    COALESCE(SUM(o.amount), 0) AS total_revenue
FROM sql_lab_customers AS c
LEFT JOIN sql_lab_orders AS o
    ON c.customer_id = o.customer_id
LEFT JOIN sql_lab_payments AS p
    ON o.order_id = p.order_id
GROUP BY c.customer_id
ORDER BY c.customer_id;
```

| Customer | Correct order revenue | Incorrect query result |
|---|---:|---:|
| Asha | 2000 | 3200 |
| Bharat | 1500 | 1500 |
| Charu | 0 | 0 |
| Dev | 500 | 1000 |

For Asha, order 1001 has two payments. The joined rows include `1200, 1200, 800`; summing produces `3200` instead of `2000`.

**Important pitfalls**
- `COUNT(DISTINCT o.order_id)` fixes the order count, **not** the repeated order amount.
- `SUM(DISTINCT o.amount)` is unsafe: separate orders can have identical amounts.
- Think about pre-aggregating each measure at the appropriate grain before combining results.
- Check source totals against post-join totals and inspect join keys for uniqueness.

## 6. Coding challenge 01 — Passed (attempt 4, 2026-10-14)

**Business requirement:** Return one row per customer containing:
- customer ID and name
- total number of orders
- total order value
- total payment amount received

Include customers with zero orders. Do not inflate order value due to multiple payments. Do not use `SUM(DISTINCT order_amount)` as a workaround.

**Expected output**

| Customer | Orders | Order value | Paid |
|---|---:|---:|---:|
| Asha | 2 | 2000 | 2000 |
| Bharat | 1 | 1500 | 1000 |
| Charu | 0 | 0 | 0 |
| Dev | 1 | 500 | 400 |

**Status:** Passed after four attempts. Full corrected learner solution and feedback: [Day 1 challenge review](Day_01_Challenge_01_Attempt_Review.md).

## 7. Interview discussion (ask one at a time)
1. Why can a JOIN return more rows than either input table?
2. Why is `SUM(DISTINCT amount)` not a safe solution for duplicate revenue?
3. How would you validate row counts and totals in a large production dataset?
4. How do grain and join cardinality affect Power BI measures and star-schema design?

## 8. Revision checklist
- [x] Can explain the grain of each table.
- [x] Can predict row counts for INNER and LEFT joins (review uniqueness assumptions).
- [x] Can explain COUNT(*) vs COUNT(column) in LEFT JOINs.
- [x] Can detect join fanout and inflated SUM.
- [x] Can independently solve Coding Challenge 01.
- [ ] Can explain the solution in 60 seconds without rambling.

## 9. Feedback log
- **Submitted SQL:** Passed in attempt 4; see linked challenge review.
- **Mistakes identified:** Initial join fanout, aliases, payment column references, COALESCE spelling; all corrected. Interview wording on SUM(DISTINCT) remains a revision item.
- **Performance observations:** Pre-aggregation avoids multiplicative rows; no query-plan benchmark performed.
- **Next action:** Day 2 advanced joins; short review of SUM(DISTINCT) and concise interview phrasing.

## 10. Day 01 interview feedback — 2026-10-14

**Question:** Customers has 100 rows, Orders has 500 rows. Can an INNER JOIN on `customer_id` return more than 500 rows?

**Learner response:** “Might be more than 500 based on how much rows we are getting against one customer.”

**Assessment:** Partially correct. The answer recognizes that match multiplicity matters, but must explicitly consider whether `Customers.customer_id` is unique.

**Correct reasoning:** If `customer_id` is unique in Customers, every Orders row matches at most one customer, so the result cannot exceed 500 rows. If Customers contains duplicate `customer_id` values, a single order can match multiple customer records and produce more than 500 joined rows.

**Revision point:** Before predicting JOIN row counts, state the grain and uniqueness of both join keys. Do not confuse many orders per customer with duplicated customer records.

**Status:** Follow-up resolved and coding challenge passed.


## 12. Day 01 final interview check — passed (2026-10-14)

**Follow-up:** Three distinct orders have amounts 500, 500 and 800. Compare SUM(amount) and SUM(DISTINCT amount).

**Learner answer:** SUM(amount) = 1800; SUM(DISTINCT amount) = 1300. **Both correct.** Since both 500-valued orders are legitimate, 1800 is the correct order revenue. DISTINCT eliminates duplicate *values*, not duplicate order identities.

**Suggested interview answer:** “I aggregated payments by order_id before joining because an order can have multiple payment rows. This prevents multiplying the order value. SUM(DISTINCT amount) is unsafe because two different orders may have the same amount.”

**Overall Day 01 status:** Completed. SQL challenge passed; concept check passed. Revision: practice delivering the explanation concisely without prompting.
