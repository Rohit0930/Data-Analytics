# Day 02 — Advanced Joins (In Progress)

**Started:** 2026-10-17  
**Database:** PostgreSQL  
**Topics:** Self JOIN, FULL OUTER JOIN, CROSS JOIN, Non-Equi JOIN; ON vs WHERE edge cases  
**Status:** Topic 1 (Self JOIN), coding challenge pending

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

### Challenge 01 — Awaiting learner solution

Return every employee and their manager's name. Include Priya, whose manager_id is NULL. Choose the appropriate join type and use separate aliases.

| employee_name | manager_name |
|---|---|
| Priya | NULL |
| Amit | Priya |
| Neha | Priya |
| Ravi | Amit |
| Sara | Amit |
| Dev | Neha |

**Do not reveal the solution until the learner attempts it.**

### Review criteria
- Correct relationship between employee.manager_id and manager.employee_id.
- Retains top-level manager with NULL manager_id.
- One row per employee, no unnecessary duplicate rows.
- Explain when to use LEFT vs INNER JOIN.

## Pending topics
- [ ] Self JOIN challenge and feedback
- [ ] FULL OUTER JOIN
- [ ] CROSS JOIN
- [ ] Non-Equi JOIN
- [ ] Advanced ON vs WHERE scenarios
- [ ] Day 2 interview check and revision

Day 1 remains completed; Day 2 is not yet completed.
