# Day 01 — Coding Challenge 01: Attempt 1 Review

Date: 2026-10-14
Status: Submitted; revision requested (not passed)

## Learner's submitted SQL

```sql
WITH CTE AS (
  SELECT A.CUSTOMER_ID, A.CUSTOMER_NAME,
         Count(B.Order_id) AS Total_Order,
         sum(b.amount) as Order_value,
         sum(c.paid_amount) as amount paid
  from sql_lab_customers
  left join sql_lab_orders on a.customer_id=b.customer_id
  left join sql_lab_payments on a.order_id=b.order_id
  group by A.CUSTOMER_ID,A.CUSTOMER_NAME
)
select * from cte;
```

## Review

Good decisions:
- Uses LEFT JOIN to retain customers with no orders.
- Recognizes final output should be grouped at customer grain.
- Identifies the correct business measures: order count, order value, payments received.

Corrections:
1. Add aliases a, b, and c to the FROM/JOIN clauses.
2. Payment join key should be b.order_id = c.order_id; the customers table has no order_id.
3. Use a single valid alias such as total_paid instead of unquoted `amount paid`.
4. A customer/order/payment join repeats orders with multiple payments. Asha's order 1001 has two payments, so naive COUNT(order_id) returns 3 rather than 2, and SUM(order amount) returns 3200 rather than 2000.
5. COALESCE is needed to return zero rather than NULL for a customer with no orders/payments.

## Guided next step

Try first aggregating payments at order_id grain in a CTE, then LEFT JOIN that result to orders and customers. At the customer level, count orders and sum order amount and pre-aggregated paid amount. Explain why the intermediate payment result has at most one row per order.

Do not use SUM(DISTINCT order amount). Corrected solution intentionally withheld until next learner attempt.


## Attempt 2 — 2026-10-14

**Learner SQL:**

```sql
WITH amount_paid AS (
    SELECT order_id, SUM(amount_paid) AS amount_paid
    FROM sql_lab_payments
    GROUP BY order_id
)
SELECT a.customer_id, a.customer_name,
       collese(COUNT(b.order_id), 0) AS total_order,
       collese(SUM(b.amount), 0) AS order_value,
       collese(SUM(c.paid_amount), 0) AS amount paid
FROM sql_lab_customers a
LEFT JOIN sql_lab_orders b ON a.customer_id = b.customer_id
LEFT JOIN amount_paid c ON b.order_id = c.order_id
GROUP BY a.customer_id, a.customer_name;
```

**Assessment:** The join and aggregation strategy is now logically correct. Payments are grouped to one row per order before joining, preventing fanout of order value.

**Three remaining SQL errors:**
1. Source column is `paid_amount`, not `amount_paid`: use `SUM(paid_amount) AS amount_paid` in the CTE.
2. Function name is `COALESCE`, not `collese`.
3. Alias `AS amount paid` is invalid without quotes; use `AS total_paid`.

Also, COUNT(b.order_id) already returns 0 for customers without orders, so COALESCE around COUNT is optional. Challenge remains in progress until runnable query is submitted. Reference solution still withheld.


## Attempt 3 — 2026-10-14

**Learner SQL:**

```sql
WITH amount_paid AS (
    SELECT order_id, SUM(amount_paid) AS amount_paid
    FROM sql_lab_payments
    GROUP BY order_id
)
SELECT a.customer_id, a.customer_name,
       COUNT(b.order_id) AS total_order,
       COALESCE(SUM(b.amount), 0) AS order_value,
       COALESCE(SUM(c.paid_amount), 0) AS paid_amount
FROM sql_lab_customers a
LEFT JOIN sql_lab_orders b ON a.customer_id = b.customer_id
LEFT JOIN amount_paid c ON b.order_id = c.order_id
GROUP BY a.customer_id, a.customer_name;
```

**Assessment:** Correct join/aggregation logic; query cannot execute until two column references are fixed.

- The physical payment column is `sql_lab_payments.paid_amount`. Use `SUM(paid_amount) AS amount_paid` inside the CTE.
- The CTE exposes `amount_paid`, so the final SELECT should use `SUM(c.amount_paid)`, not `SUM(c.paid_amount)`.
- COUNT(b.order_id) correctly returns zero for customers without orders.
- COALESCE around the SUM expressions correctly handles zero-order customers.
- An ORDER BY may be added for deterministic display, but is not necessary for correctness.

**Next action:** Resubmit the query with the two corrected references. Do not mark coding challenge passed until runnable SQL is submitted.
