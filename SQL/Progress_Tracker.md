# SQL Learning Progress Tracker

**Program start:** 2026-10-14  
**Target:** 42 days of senior/lead SQL mastery  
**Primary database:** PostgreSQL  
**Last updated:** 2026-10-17

## Status summary
- Days completed: **1 / 42**
- Current lesson: **Day 02 — Advanced Joins (ON vs WHERE next)**
- Coding challenges passed: **2**
- Mock interviews completed: **0**
- Current next action: **Learn advanced LEFT JOIN ON vs WHERE predicates and NULL behavior; Non-Equi reference solution shared on request**

## Topic tracker
| Day | Topic | Status | Coding | Interview |
|---:|---|---|---|---|
| 01 | Joins, grain, cardinality, fanout | Completed (2026-10-14) | Passed (attempt 4) | Concept checks passed; concise explanation for revision |
| 02 | Advanced joins: self, full, cross, non-equi | In progress (2026-10-17) | Self JOIN passed; FULL OUTER JOIN reference provided (not independently passed); CROSS JOIN reference provided (not independently passed) | Self JOIN concept response correct |
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

| 2026-10-17 | 02 | Self JOIN attempt 1 uses table `employee` instead of `employees` and `b.employee` instead of `b.employee_id` | Use correct table and column names; LEFT self join is appropriate to retain the top-level manager | Corrected; passed |

## Day 02 lesson preparation
- Full advanced guide committed 2026-10-17: Self JOIN, FULL OUTER JOIN, CROSS JOIN, non-equi range joins, ON vs WHERE, NULL-safe comparisons, row-count and performance checks.
- Worked PostgreSQL examples and expected outputs available in `01_Joins_Subqueries_CTEs/Day_02_Advanced_Joins.md`.
- Learner submitted Self JOIN challenge 01 attempt 1. Relationship and LEFT JOIN direction are correct, but two identifier errors prevent execution. Corrected Self JOIN submitted and passed; FULL OUTER JOIN is next. Day 02 overall remains in progress.

## Day 02 interview concept feedback
- **2026-10-17 — Self JOIN purpose:** Correct. Learner explained that manager_id must be matched to employee_id in the same table to retrieve manager name. Refine interview wording to describe two aliases and a self-referencing relationship.
- **Day 02 hands-on:** Employee–manager Self JOIN passed on corrected attempt. Remaining advanced join exercises pending; do not mark Day 02 completed.

| 2026-10-17 | 02 | Self JOIN attempt used `employee` instead of created `employees` table and returned `manager` alias instead of `manager_name` | Match schema exactly and use required output column names/order; join condition and LEFT JOIN are correct | Corrected; passed |

## Day 02 Self JOIN — passing submission (2026-10-17)

```sql
SELECT b.employee_name AS manager_name,
       a.employee_name AS employee_name
FROM employees a
LEFT JOIN employees b
  ON a.manager_id = b.employee_id;
```

- **Passed:** Correct table, manager key, aliases, LEFT JOIN, and output names; top-level employee Priya remains with NULL manager.
- **Optional:** Return employee_name first to match requested column order; ORDER BY a.employee_id for deterministic display.
- **Next:** FULL OUTER JOIN reconciliation challenge; Day 02 remains in progress.

## Day 02 FULL OUTER JOIN — attempt 1 feedback (2026-10-17)

- Correct FULL OUTER JOIN on order_id and appropriate use of COALESCE in principle.
- Fix MySQL-style backticks: PostgreSQL identifiers must be unquoted or double-quoted.
- Determine row presence with order_id IS NULL, not amount IS NULL.
- Compare nullable amounts using IS DISTINCT FROM, with a defined policy for both amounts NULL.
- Use exact requested status labels. Reference solution provided on learner request; no independent corrected attempt, so challenge not marked passed.

## Day 02 FULL OUTER JOIN — reference solution shared (2026-10-17)

- Learner requested the correct answer after their first attempt.
- Reference query uses FULL OUTER JOIN, COALESCE, missing-row checks on order_id, and NULL-safe amount comparison via IS DISTINCT FROM.
- Expected classifications: 101 MATCH, 102 AMOUNT_MISMATCH, 103 ONLY_IN_SALES, 104 ONLY_IN_FINANCE.
- Explanation: NULL amount does not mean a missing order. Both NULL amounts currently classify as MATCH; business rules may require a missing-amount category.
- Coding challenge remains not independently passed; move to CROSS JOIN next.

## Day 02 CROSS JOIN — challenge assigned (2026-10-17)

- Practice setup and expected six-row output saved in Day_02_Advanced_Joins.md.
- Input: 3 products × 2 months; 6 sales records with multiple transactions for some product-months.
- Goal: CROSS JOIN reporting grid, LEFT JOIN sales, SUM and COALESCE; total 3400, including zero-sales combinations.
- Partial CTE attempt submitted and reviewed; learner requested the complete solution. Reference provided, not independently passed.

## Day 02 CROSS JOIN — reference solution shared (2026-10-17)

- Learner started monthly sales CTE but used nonexistent columns and incomplete EXTRACT syntax. DATE_TRUNC('month', sale_date)::date is the correct PostgreSQL month-start expression.
- Complete solution uses monthly sales CTE, products CROSS JOIN months, LEFT JOIN by product_id and month_start, and COALESCE for missing sales.
- Six expected rows; total revenue 3400. Reference solution shared on learner request, not independently passed.
- Next: Non-equi JOIN range-matching challenge.

## Day 02 Non-Equi JOIN — challenge assigned (2026-10-17)

- Practice setup: employee_pay with six employees including boundary salaries 60000, 100000, 200000 and one NULL salary; salary_bands with four half-open ranges.
- Goal: LEFT JOIN to salary bands using inclusive lower / exclusive upper boundaries; preserve NULL salary; expected six rows with no duplicates.
- Learner requested the correct answer without an attempt. Reference solution provided; not independently passed.

## Day 02 Non-Equi JOIN — reference solution shared (2026-10-17)

- Learner requested the correct answer directly; no independent coding attempt submitted.
- Reference uses LEFT JOIN employee_pay to salary_bands with salary >= min_salary and salary < max_salary; six expected rows including NULL salary.
- Correctly explains half-open interval boundaries and the need to validate overlapping ranges in production.
- Not counted as a passed challenge. Next topic: ON vs WHERE and NULL behavior.

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
