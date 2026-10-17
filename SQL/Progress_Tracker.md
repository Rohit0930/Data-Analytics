# SQL Learning Progress Tracker

**Program start:** 2026-10-14  
**Target:** 42 days of senior/lead SQL mastery  
**Primary database:** PostgreSQL  
**Last updated:** 2026-10-17

## Status summary
- Days completed: **1 / 42**
- Current lesson: **Day 02 — Advanced Joins (Self JOIN in progress)**
- Coding challenges passed: **1**
- Mock interviews completed: **0**
- Current next action: **Resubmit Day 02 employee–manager Self JOIN with correct table name and manager key**

## Topic tracker
| Day | Topic | Status | Coding | Interview |
|---:|---|---|---|---|
| 01 | Joins, grain, cardinality, fanout | Completed (2026-10-14) | Passed (attempt 4) | Concept checks passed; concise explanation for revision |
| 02 | Advanced joins: self, full, cross, non-equi | In progress (2026-10-17) | Self JOIN attempt 1: correct logic, identifiers to fix | Self JOIN concept response correct |
| 03–07 | Semi/anti joins, subqueries, CTEs, mock | Not started | — | — |
| 08–14 | Window functions and mock | Not started | — | — |
| 15–21 | Date logic and business analytics | Not started | — | — |
| 22–28 | PostgreSQL and performance | Not started | — | — |
| 29–35 | Data modeling and engineering SQL | Not started | — | — |
| 36–42 | Senior scenarios and final assessment | Not started | — | — |

## Mistake and revision log
| Date | Day | Mistake / learning gap | Correct principle | Revision status |
|---|---|---|---|---|
| 2026-10-14 | 01 | First join-cardinality answer did not explicitly check uniqueness of the Customers join key | With unique customer_id in Customers, each Orders row matches at most one customer; duplicate customer keys can multiply matches | Reviewed |
| 2026-10-14 | 01 | ROW_NUMBER duplicate-ID query returns one row per excess occurrence; ORDER BY customer_id is non-deterministic within a customer partition | Use GROUP BY/HAVING for one row per duplicate ID; use ROW_NUMBER with a stable tie-breaker for record-level deduplication | Review later |

| 2026-10-14 | 01 | First coding attempt: missing table aliases; invalid payments join key; alias with spaces; COUNT and SUM inflated by multiple payments; NULL totals for customers without orders | Fix aliases and join on orders.order_id = payments.order_id; pre-aggregate payment records by order before customer-level totals; COALESCE zero-order totals | Corrected; reviewed |
| 2026-10-14 | 01 | Attempt 2: SUM(amount_paid) references nonexistent column; COALESCE misspelled collese; alias AS amount paid invalid | Use SUM(paid_amount) AS amount_paid in CTE; COALESCE; AS total_paid | Corrected in attempt 4 |
| 2026-10-14 | 01 | Attempt 3: source payments table has paid_amount, not amount_paid; payment CTE outputs amount_paid, not paid_amount | In CTE use SUM(paid_amount) AS amount_paid; in final SELECT use SUM(c.amount_paid) | Corrected in attempt 4 |

| 2026-10-14 | 01 | Interview explanation for SUM(DISTINCT amount) was incomplete; conflated duplicate join rows with duplicate monetary values | SUM(DISTINCT amount) deduplicates values, not orders: two valid orders with identical amounts must both count. Pre-aggregate payments per order_id to prevent fanout | Concept check passed; wording for revision |

| 2026-10-17 | 02 | Self JOIN attempt 1 uses table `employee` instead of `employees` and `b.employee` instead of `b.employee_id` | Use correct table and column names; LEFT self join is appropriate to retain the top-level manager | Revision pending |

## Day 02 lesson preparation
- Full advanced guide committed 2026-10-17: Self JOIN, FULL OUTER JOIN, CROSS JOIN, non-equi range joins, ON vs WHERE, NULL-safe comparisons, row-count and performance checks.
- Worked PostgreSQL examples and expected outputs available in `01_Joins_Subqueries_CTEs/Day_02_Advanced_Joins.md`.
- Learner submitted Self JOIN challenge 01 attempt 1. Relationship and LEFT JOIN direction are correct, but two identifier errors prevent execution. Awaiting corrected query; Day 02 challenge not yet passed.

## Day 02 interview concept feedback
- **2026-10-17 — Self JOIN purpose:** Correct. Learner explained that manager_id must be matched to employee_id in the same table to retrieve manager name. Refine interview wording to describe two aliases and a self-referencing relationship.
- **Day 02 hands-on:** Not yet attempted. Do not mark completed.

| 2026-10-17 | 02 | Self JOIN attempt used `employee` instead of created `employees` table and returned `manager` alias instead of `manager_name` | Match schema exactly and use required output column names/order; join condition and LEFT JOIN are correct | Resubmission pending |

## Assessment rules
- **Correctness:** Query matches the requirement and expected result.
- **Edge cases:** Handles zero matches, duplicate keys, NULLs and boundary dates when relevant.
- **Performance:** Avoids unnecessary work and explains the query plan where relevant.
- **Explanation:** States the grain, approach, risks and validation steps.
- **Completion:** A lesson is completed only after hands-on review and interview feedback.

## Notes
- First conceptual interview response: **partially correct**. User recognized match multiplicity; uniqueness/fanout was clarified.
- Second interview response (detecting duplicate customer IDs): **correct**, using both ROW_NUMBER CTE and GROUP BY/HAVING. Improvement points: ROW_NUMBER output can repeat IDs when there are 3+ occurrences; use stable ordering for deterministic deduplication; COUNT(*) includes NULL-group rows.
- First Day 01 customer/order/payment SQL attempt submitted. Review: query does not run as written (missing aliases, invalid join, unquoted column alias), and naive multi-payment join duplicates order counts and amounts. Learner asked to revise; challenge not passed.
- Second Day 01 coding attempt correctly pre-aggregated payments per order and fixed join keys. Three SQL syntax issues remain. Coding challenge not yet passed; learner asked to resubmit corrected runnable query.
- Third coding attempt fixed COALESCE and the output alias. Payment CTE source column and downstream CTE column reference remain reversed. Fourth attempt is valid PostgreSQL and produces correct totals. Coding challenge passed.
- Fourth coding attempt passed: correctly uses SUM(paid_amount) AS paid_amount in CTE, then SUM(c.paid_amount) in outer query, with correct LEFT JOINs, COUNT, COALESCE, and GROUP BY. Sample expected output: Asha 2/2000/2000, Bharat 1/1500/1000, Charu 0/0/0, Dev 1/500/400.
- Final Day 1 interview question attempted: user correctly connected the risk to joins but did not explain why SUM(DISTINCT amount) is unsafe when separate orders share an amount. Follow-up example answered correctly: SUM(500,500,800)=1800; SUM(DISTINCT)=1300. Day 1 completed; concise interview explanation is a revision item.
- Day 01 completion recorded 2026-10-14. Correctly computed both SUM and SUM(DISTINCT) for duplicate-valued orders, demonstrating why DISTINCT is unsafe for order revenue. Interview phrasing still worth practicing.
- Add actual mistakes only after observing an attempt; do not assume weaknesses.
- Keep lesson notes and tracker synchronized after each completed topic.
