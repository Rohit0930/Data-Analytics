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
