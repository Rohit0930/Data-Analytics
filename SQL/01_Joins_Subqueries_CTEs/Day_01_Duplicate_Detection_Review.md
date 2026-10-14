# Day 1 — Duplicate Customer ID Detection

Date: 2026-10-14

## Question
How would you identify duplicate customer IDs?

## Learner's answer
Proposed two correct methods: a ROW_NUMBER() CTE partitioned by customer_id, and GROUP BY customer_id HAVING COUNT(customer_id) > 1.

## Assessment
**Correct.** Both approaches identify duplicate keys.

```sql
SELECT customer_id, COUNT(*) AS occurrence_count
FROM customer_table
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

## Revision notes
- GROUP BY / HAVING is the simpler choice for a unique list of duplicate IDs and their counts.
- A ROW_NUMBER query filtered to rn > 1 returns every excess occurrence. Use DISTINCT if only unique IDs are needed.
- ORDER BY customer_id is not a meaningful tie-breaker within PARTITION BY customer_id. For record-level deduplication, use a stable ordering such as created_at, record_id.
- COUNT(*) includes NULL rows; COUNT(customer_id) excludes them.
- A primary key prohibits duplicate customer IDs. Consider staging/unconstrained tables for this scenario.

Next: Submit Day 1 coding challenge for correct order and payment totals.
