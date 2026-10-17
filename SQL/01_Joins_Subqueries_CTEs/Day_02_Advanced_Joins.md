# Day 02 — Advanced Joins (In Progress)

**Started:** 2026-10-17  
**Database:** PostgreSQL  
**Topics:** Self JOIN, FULL OUTER JOIN, CROSS JOIN, Non-Equi JOIN; ON vs WHERE edge cases  
**Status:** Topic 1 (Self JOIN) passed; Topic 2 (FULL OUTER JOIN) next

## Day 2 plan
1. Self JOIN — employee/manager hierarchy.
2. FULL OUTER JOIN — reconcile transactions between two systems.
3. CROSS JOIN — build product/month reporting grids.
4. Non-Equi JOIN — match salaries to ranges.
5. Advanced join conditions — ON vs WHERE and NULL behavior.
6. One-at-a-time coding and interview assessment.

## Topic 1: Self JOIN

A self join uses two aliases of the same table to compare or relate different rows. For an employee-manager relationship, the employee's `manager_id` refers to another row's `employee_id`.

**Learner's concept response:** “because its connect throgh id not name to extract his name we need to join on manager id to employee id”

**Evaluation:** Correct. The manager name is not stored in the employee row; the relationship is through IDs. Interview wording improvement: “The manager_id references another row's employee_id in the same table, so we join Employees to itself with separate aliases to retrieve the manager's name.”

### Practice setup

```sql
DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
  employee_id INT PRIMARY KEY,
  employee_name VARCHAR(50) NOT NULL,
  manager_id INT REFERENCES employees(employee_id)
);
INSERT INTO employees (employee_id, employee_name, manager_id) VALUES
(1, 'Priya', NULL),
(2, 'Amit', 1),
(3, 'Neha', 1),
(4, 'Ravi', 2),
(5, 'Sara', 2),
(6, 'Dev', 3);
```

### Challenge 01 — Passed (2026-10-17)

Return every employee and their manager's name. Include Priya, whose manager_id is NULL. Choose the appropriate join type and use separate aliases.

| employee_name | manager_name |
|---|---|
| Priya | NULL |
| Amit | Priya |
| Neha | Priya |
| Ravi | Amit |
| Sara | Amit |
| Dev | Neha |

**Learner submitted and passed the employee–manager Self JOIN challenge.** The exact query and feedback are recorded below.

### Review criteria
- Correct relationship between employee.manager_id and manager.employee_id.
- Retains top-level manager with NULL manager_id.
- One row per employee, no unnecessary duplicate rows.
- Explain when to use LEFT vs INNER JOIN.

## Pending topics
- [x] Self JOIN challenge and feedback
- [ ] FULL OUTER JOIN
- [ ] CROSS JOIN
- [ ] Non-Equi JOIN
- [ ] Advanced ON vs WHERE scenarios
- [ ] Day 2 interview check and revision

Day 1 remains completed; Day 2 is not yet completed.


---

# Advanced teaching guide — Day 02

**Scope:** Deep understanding of Self, FULL OUTER, CROSS, and Non-Equi joins; PostgreSQL NULL semantics; ON vs WHERE; fanout; performance; business interview cases. This guide supplies worked examples for learning. The employee–manager challenge above has now been passed.

## A. Universal senior-level JOIN checklist

Before any join, answer:

1. **Output grain:** One row per what? Employee, order, customer-month, transaction?
2. **Join key uniqueness:** Is the key unique on the left, right, both, or neither?
3. **Required unmatched rows:** Must the output retain all left rows, all right rows, both, or only matches?
4. **NULL semantics:** Can keys be NULL? Should NULL match NULL?
5. **Measures:** Does joining multiply a fact value? Should a child table be pre-aggregated?
6. **Validation:** Check row counts, unmatched keys, duplicate matches and control totals.
7. **Performance:** Check filters, join indexes, table sizes, statistics and actual plan.

For each left row with key k, an inner join emits as many rows as there are matching right rows for k. If a key appears L(k) times on the left and R(k) times on the right, it contributes L(k) × R(k) joined rows. Uniqueness is what makes a join cardinality predictable.

## B. Self JOIN — deeper understanding

A self join joins **two logical instances of one physical table**. Aliases give each instance a different role. Common use cases: employee–manager, customer referrals, product pairs, matching events, predecessor/successor records, duplicate investigation.

**Employee hierarchy concept:** `employee.manager_id` points to `manager.employee_id` in the same table. A LEFT self join preserves the CEO or top-level employee whose manager_id is NULL. The employee–manager exercise earlier in this document is **still for the learner to solve**.

### Worked example (different from the challenge): pairs of colleagues sharing a manager

Use the `employees` table from the existing setup. We want each **unordered pair** of employees who share a non-NULL manager, without a row pairing an employee with themself and without mirrored pairs.

```sql
SELECT
    e1.employee_name AS colleague_1,
    e2.employee_name AS colleague_2,
    e1.manager_id
FROM employees AS e1
JOIN employees AS e2
  ON e1.manager_id = e2.manager_id
 AND e1.employee_id < e2.employee_id
WHERE e1.manager_id IS NOT NULL
ORDER BY e1.employee_id, e2.employee_id;
```

Expected result:

| colleague_1 | colleague_2 | manager_id |
|---|---|---:|
| Amit | Neha | 1 |
| Ravi | Sara | 2 |

**Why `<` instead of `<>`?** The less-than predicate keeps each unordered pair once. `<>` would return both Amit–Neha and Neha–Amit. An equality-only self join would also pair each employee with themself.

**Advanced considerations:**
- For the manager hierarchy, one self join finds **one level**. Multiple hierarchy levels require repeated joins or a recursive CTE (Day 06).
- Joining on nonunique business names is unsafe; join on stable IDs.
- For the hierarchy lookup, a primary-key index on manager.employee_id supports efficient matches. A separate index on employees.manager_id may help reverse lookups (all direct reports).
- An inner self join would omit employees without a matching manager; a left self join retains them.
- Aliases are roles, not copies of the table stored on disk.

## C. FULL OUTER JOIN — two-system reconciliation

A FULL OUTER JOIN returns matched rows **plus unmatched rows from either side**. It is useful for Finance vs CRM reconciliation, source vs warehouse QA, and migration audits.

### Sample input

```sql
DROP TABLE IF EXISTS day2_crm_sales;
DROP TABLE IF EXISTS day2_finance_sales;

CREATE TABLE day2_crm_sales (
  order_id INT PRIMARY KEY,
  amount NUMERIC(12,2) NOT NULL
);
CREATE TABLE day2_finance_sales (
  order_id INT PRIMARY KEY,
  amount NUMERIC(12,2) NOT NULL
);

INSERT INTO day2_crm_sales VALUES
(1001,1000), (1002,200), (1004,500);
INSERT INTO day2_finance_sales VALUES
(1001,1000), (1003,700), (1004,450);
```

### Worked query

```sql
SELECT
    COALESCE(c.order_id, f.order_id) AS order_id,
    c.amount AS crm_amount,
    f.amount AS finance_amount,
    CASE
      WHEN c.order_id IS NULL THEN 'ONLY_IN_FINANCE'
      WHEN f.order_id IS NULL THEN 'ONLY_IN_CRM'
      WHEN c.amount <> f.amount THEN 'AMOUNT_MISMATCH'
      ELSE 'MATCH'
    END AS reconciliation_status
FROM day2_crm_sales AS c
FULL OUTER JOIN day2_finance_sales AS f
  ON c.order_id = f.order_id
ORDER BY COALESCE(c.order_id, f.order_id);
```

Expected result:

| order_id | crm_amount | finance_amount | reconciliation_status |
|---:|---:|---:|---|
| 1001 | 1000 | 1000 | MATCH |
| 1002 | 200 | NULL | ONLY_IN_CRM |
| 1003 | NULL | 700 | ONLY_IN_FINANCE |
| 1004 | 500 | 450 | AMOUNT_MISMATCH |

**Senior-level caveats:**
- A FULL JOIN is not a cure for duplicate source keys. If order 1001 appears twice on each side, it produces four matched rows. Check uniqueness or pre-aggregate to the agreed reconciliation grain first.
- `COALESCE(c.order_id, f.order_id)` produces the common reporting key for unmatched records.
- With nullable amounts, `c.amount <> f.amount` yields UNKNOWN if one amount is NULL. For NULL-sensitive comparisons use `c.amount IS DISTINCT FROM f.amount` (when that NULL behavior is desired).
- A `WHERE c.order_id IS NOT NULL` after the join **drops finance-only rows**. For period-scoped reconciliation, filter each source in a CTE before FULL JOIN.
- Compare like-for-like: currency, tax inclusion, time zone, business cutoffs, transaction status and grain.

### Reconciliation validation

```sql
SELECT COUNT(*) AS crm_rows, SUM(amount) AS crm_total
FROM day2_crm_sales;

SELECT COUNT(*) AS finance_rows, SUM(amount) AS finance_total
FROM day2_finance_sales;
```

Do not infer financial equality from equal row counts. Investigate unmatched keys and mismatched values separately.

## D. CROSS JOIN — intentional Cartesian product

A CROSS JOIN creates **every combination** of rows. With 2 products and 2 months, the result has 2 × 2 = 4 rows. This is useful for building a complete reporting grid so months with no sales appear as zero, but can explode row counts.

### Worked example: complete product–month revenue grid

```sql
WITH products(product_id, product_name) AS (
  VALUES (1,'Basic'), (2,'Pro')
),
months(month_start) AS (
  VALUES (DATE '2026-09-01'), (DATE '2026-10-01')
),
sales(product_id, month_start, revenue) AS (
  VALUES
    (1, DATE '2026-09-01', 100::numeric),
    (2, DATE '2026-09-01', 200::numeric),
    (2, DATE '2026-10-01', 300::numeric)
)
SELECT
    p.product_name,
    m.month_start,
    COALESCE(s.revenue, 0) AS revenue
FROM products AS p
CROSS JOIN months AS m
LEFT JOIN sales AS s
  ON s.product_id = p.product_id
 AND s.month_start = m.month_start
ORDER BY p.product_id, m.month_start;
```

Expected result:

| product_name | month_start | revenue |
|---|---|---:|
| Basic | 2026-09-01 | 100 |
| Basic | 2026-10-01 | 0 |
| Pro | 2026-09-01 | 200 |
| Pro | 2026-10-01 | 300 |

**Critical:** The sales input in this example is already unique at product–month grain. If actual sales contain multiple transactions, aggregate them to product–month **before** joining to the grid.

**PostgreSQL extension:** `generate_series(DATE '2026-01-01', DATE '2026-12-01', INTERVAL '1 month')` can generate monthly dates; cast the resulting timestamps to date as needed.

**Performance:** 10,000 products × 365 dates = 3,650,000 rows *before* any additional joins. Restrict dimensions and time periods early. A missing ON clause in an ordinary INNER JOIN can accidentally create a Cartesian product.

## E. Non-equi JOIN — matching by inequality or range

An equi join uses equality. A non-equi join uses predicates such as `<`, `>=`, `BETWEEN`, or overlapping date ranges. Examples: salary bands, SCD Type 2 effective-date lookup, tiered pricing and interval overlap.

### Worked example: salary band assignment

```sql
WITH staff(employee_name, salary) AS (
  VALUES ('Asha',50000), ('Bharat',85000), ('Charu',120000)
),
bands(band_name, min_salary, max_salary) AS (
  VALUES
    ('Junior',0,60000),
    ('Mid',60000,100000),
    ('Senior',100000,200000)
)
SELECT s.employee_name, s.salary, b.band_name
FROM staff AS s
LEFT JOIN bands AS b
  ON s.salary >= b.min_salary
 AND s.salary < b.max_salary
ORDER BY s.salary;
```

Expected result:

| employee_name | salary | band_name |
|---|---:|---|
| Asha | 50000 | Junior |
| Bharat | 85000 | Mid |
| Charu | 120000 | Senior |

**Why `>= lower AND < upper`?** Half-open intervals [lower, upper) ensure salary 60000 belongs to **Mid only**. `BETWEEN` includes both ends, which can cause a boundary salary to match two adjacent ranges.

**Edge cases:** Overlapping bands create duplicate matches; gaps create unmatched salaries; salaries beyond the highest band produce NULL on LEFT JOIN. Define and test boundary rules before deployment.

### Effective-dated dimension / SCD Type 2 pattern

```sql
SELECT
    f.order_id,
    f.customer_id,
    f.order_date,
    d.customer_segment
FROM fact_orders AS f
LEFT JOIN dim_customer_history AS d
  ON f.customer_id = d.customer_id
 AND f.order_date >= d.effective_from
 AND f.order_date <  d.effective_to;
```

This is a **pattern** (the two business tables are not part of today's runnable sample setup). It assumes `effective_to` is populated with an exclusive end date; if NULL means current, the predicate must explicitly account for NULL. Overlapping history intervals can duplicate facts and inflate revenue. Production designs often use a stable surrogate dimension key and enforce non-overlap in ETL/validation.

## F. Advanced ON vs WHERE — essential outer-join behavior

The placement of a filter changes results with an OUTER JOIN.

Consider customers 1 and 2; only customer 1 has a recent order. We need **all customers**, with their October orders if present.

### Correct: condition in ON

```sql
WITH customers(customer_id) AS (
  VALUES (1),(2)
),
orders(order_id,customer_id,order_date) AS (
  VALUES (101,1,DATE '2026-10-10'),
         (102,1,DATE '2026-09-10')
)
SELECT c.customer_id, o.order_id
FROM customers AS c
LEFT JOIN orders AS o
  ON c.customer_id = o.customer_id
 AND o.order_date >= DATE '2026-10-01'
ORDER BY c.customer_id, o.order_id;
```

Result:

| customer_id | order_id |
|---:|---:|
| 1 | 101 |
| 2 | NULL |

### Incorrect for that requirement: condition in WHERE

```sql
WITH customers(customer_id) AS (VALUES (1),(2)),
orders(order_id,customer_id,order_date) AS (
  VALUES (101,1,DATE '2026-10-10'),
         (102,1,DATE '2026-09-10')
)
SELECT c.customer_id, o.order_id
FROM customers AS c
LEFT JOIN orders AS o
  ON c.customer_id = o.customer_id
WHERE o.order_date >= DATE '2026-10-01'
ORDER BY c.customer_id, o.order_id;
```

Result: only `1 | 101`. The WHERE predicate removes the NULL-extended row for customer 2, effectively losing the outer-join preservation.

**Nuance:** `WHERE o.order_date >= ... OR o.order_id IS NULL` is not a universal substitute for filtering in ON. A customer with *only older orders* would still be dropped, because the LEFT JOIN found rows and never created a NULL-extended row.

### Other advanced traps

- **NULL equality:** `NULL = NULL` is UNKNOWN, not TRUE. PostgreSQL's `IS NOT DISTINCT FROM` treats two NULLs as equal, but matching NULL keys can create unwanted many-to-many joins.
- **COUNT:** After a LEFT JOIN, `COUNT(*)` counts the preserved customer row even when no order exists. `COUNT(o.order_id)` counts only matched non-NULL order IDs.
- **RIGHT JOIN:** Usually rewrite as LEFT JOIN with table order swapped for readability.
- **FULL JOIN with filters:** A post-join WHERE on either source may eliminate the opposite source's unmatched rows.
- **Deduplication:** `DISTINCT` can hide symptoms of a wrong join but cannot repair incorrect aggregation or business grain.
- **PostgreSQL performance:** Inspect `EXPLAIN (ANALYZE, BUFFERS)` on a safe test workload, verify estimated vs actual row counts, consider indexes on equality keys and suitable indexes for range predicates. Nested-loop, hash and merge joins are alternative execution strategies; no single one is always best.
- **SQL order of operations:** FROM/JOIN/ON form joined rows, WHERE filters, GROUP BY aggregates, HAVING filters groups, SELECT projects, ORDER BY sorts (simplified logical view).

## G. Senior interview readiness — questions to master

1. When does a self join need LEFT JOIN instead of INNER JOIN?
2. How do you reconcile unmatched and mismatched records without multiplying amounts?
3. How can CROSS JOIN intentionally fix missing reporting periods, and when is it dangerous?
4. How do half-open intervals prevent double matches at effective-date boundaries?
5. Why does moving a right-table filter from ON to WHERE alter LEFT JOIN output?
6. How would you diagnose a join that unexpectedly returns 10× more rows?
7. What does a PostgreSQL execution plan reveal about poor join cardinality estimates?

Answer **one at a time**, after the corresponding hands-on attempt.

## H. Day 2 practice sequence and progress

- [x] Self JOIN purpose: learner answered concept question correctly.
- [ ] **Challenge 01 (active):** Employee–manager self join, preserving top-level employee (see earlier section; no solution provided).
- [ ] Challenge 02: FULL OUTER JOIN reconciliation with unmatched and mismatched records.
- [ ] Challenge 03: CROSS JOIN product–month grid, including zero-sales months.
- [ ] Challenge 04: Non-equi range join with boundary conditions.
- [ ] Challenge 05: ON vs WHERE troubleshooting and final interview explanation.

**Status:** Day 2 remains in progress. Do not mark exercises complete or publish solutions before learner attempts.


## I. Self JOIN challenge — attempt 1 (2026-10-17)

Learner's SQL:

```sql
SELECT b.employee_name AS manager, a.employee_name AS employee
FROM employee a
LEFT JOIN employee b
  ON a.manager_id = b.employee;
```

**Assessment:** Join logic correct; query cannot execute against the provided practice schema. Good use of LEFT JOIN and correct alias roles (a=employee, b=manager). Two identifiers to correct: table is `employees`, not `employee`; join target column is `b.employee_id`, not `b.employee`. Output alias order may be aligned to requested `employee_name, manager_name`. No reference solution revealed. Awaiting revised SQL.



## Self JOIN Challenge 01 — Learner attempt and feedback (2026-10-17)

**Submitted SQL:**

```sql
select b.employee_name as manager, a.employee_name as employee_name
from employee a
left join employee b on a.manager_id=b.employee_id;
```

**Assessment: Correct join logic; small schema/output corrections pending.**

- **Correct:** Joins the employee alias `a` to the manager alias `b` on `a.manager_id = b.employee_id`.
- **Correct:** LEFT JOIN retains top-level manager Priya with NULL manager name.
- **Correction 1:** The lesson setup creates table `employees` (plural), not `employee`.
- **Correction 2:** The requested output is `employee_name`, `manager_name`; use `manager_name` instead of `manager` and put employee first in SELECT.
- **Optional:** Add `ORDER BY a.employee_id` for deterministic presentation; SQL results are unordered without ORDER BY.
- **Performance/grain:** Since `employee_id` is a primary key, each employee matches at most one manager; no fanout occurs from the manager side.

**Status:** Concept passed. Coding challenge awaits resubmission using the actual practice table and requested output aliases. Reference corrected query intentionally withheld until learner revises.


## Day 02 learner submission and feedback — Self JOIN PASSED (2026-10-17)

**Submitted SQL:**

```sql
SELECT b.employee_name AS manager_name,
       a.employee_name AS employee_name
FROM employees a
LEFT JOIN employees b
  ON a.manager_id = b.employee_id;
```

**Evaluation: Passed.** Both references use the correct `employees` table, and the join matches `a.manager_id` to the unique `b.employee_id`. The LEFT JOIN retains Priya, whose manager_id is NULL. Both output columns are correctly named; the order of the selected columns differs from the example but does not affect correctness. An optional `ORDER BY a.employee_id` makes display order predictable.

**Expected logical result (shown in requested column order):**

| employee_name | manager_name |
|---|---|
| Priya | NULL |
| Amit | Priya |
| Neha | Priya |
| Ravi | Amit |
| Sara | Amit |
| Dev | Neha |

**Interview explanation:** “I join the employees table to itself using two aliases because manager_id points to another employee's employee_id. LEFT JOIN keeps employees who have no manager, such as the CEO.”

**Next:** FULL OUTER JOIN reconciliation exercise. Overall Day 02 remains in progress.


## FULL OUTER JOIN — attempt 1 feedback (2026-10-17)

Learner submitted a FULL OUTER JOIN between sales_orders and finance_orders on order_id, with COALESCE for the output ID and CASE for reconciliation status.

Assessment: PARTIALLY CORRECT; revise before marking passed.

- Join type and matching key are correct.
- Remove MySQL-style backtick quoting: PostgreSQL uses unquoted identifiers or double quotes, and COALESCE is a function.
- Use the joined order IDs to identify missing rows, not nullable monetary amounts.
- Handle nullable amount comparison explicitly (PostgreSQL IS DISTINCT FROM).
- Match required uppercase output statuses exactly.
- Example edge case: order 105 appears in both tables; Sales amount NULL and Finance amount 300. This is not finance-only.

Learner to resubmit corrected SQL. Reference solution was subsequently provided at learner request; see below.


## FULL OUTER JOIN — reference solution supplied on learner request (2026-10-17)

The learner explicitly requested the correct answer after the first attempt. The reference solution is now disclosed; this is **not** an independently passed coding challenge.

```sql
SELECT
    COALESCE(a.order_id, b.order_id) AS order_id,
    a.amount AS sales_amount,
    b.amount AS finance_amount,
    CASE
        WHEN a.order_id IS NULL THEN 'ONLY_IN_FINANCE'
        WHEN b.order_id IS NULL THEN 'ONLY_IN_SALES'
        WHEN a.amount IS DISTINCT FROM b.amount THEN 'AMOUNT_MISMATCH'
        ELSE 'MATCH'
    END AS status
FROM sales_orders a
FULL OUTER JOIN finance_orders b
    ON a.order_id = b.order_id
ORDER BY COALESCE(a.order_id, b.order_id);
```

Expected output:

| order_id | sales_amount | finance_amount | status |
|---:|---:|---:|---|
| 101 | 500 | 500 | MATCH |
| 102 | 700 | 650 | AMOUNT_MISMATCH |
| 103 | 900 | NULL | ONLY_IN_SALES |
| 104 | NULL | 300 | ONLY_IN_FINANCE |

**Explanation:** Identify missing rows by their IDs first. Only after both IDs are present should the amount comparison run. PostgreSQL `IS DISTINCT FROM` treats one NULL amount and one non-NULL amount as different. If both present amounts are NULL, the reference query labels the row MATCH; whether to instead flag missing amounts is a business decision.

**Next:** Learner should explain the NULL-amount edge case in their own words, then proceed to CROSS JOIN. Day 2 overall remains in progress.
