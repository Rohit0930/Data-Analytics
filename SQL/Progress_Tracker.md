# SQL Learning Progress Tracker

**Program start:** 2026-10-14  
**Target:** 42 days of senior/lead SQL mastery  
**Primary database:** PostgreSQL  
**Last updated:** 2026-10-14

## Status summary
- Days completed: **0 / 42**
- Current lesson: **Day 01 — Joins, Grain & Cardinality**
- Coding challenges passed: **0**
- Mock interviews completed: **0**
- Current next action: **Explain why INNER JOIN output can exceed the number of Orders rows when the customer key is duplicated; then submit Coding Challenge 01**

## Topic tracker
| Day | Topic | Status | Coding | Interview |
|---:|---|---|---|---|
| 01 | Joins, grain, cardinality, fanout | In progress | Pending | Pending |
| 02–07 | Advanced joins, subqueries, CTEs, mock | Not started | — | — |
| 08–14 | Window functions and mock | Not started | — | — |
| 15–21 | Date logic and business analytics | Not started | — | — |
| 22–28 | PostgreSQL and performance | Not started | — | — |
| 29–35 | Data modeling and engineering SQL | Not started | — | — |
| 36–42 | Senior scenarios and final assessment | Not started | — | — |

## Mistake and revision log
| Date | Day | Mistake / learning gap | Correct principle | Revision status |
|---|---|---|---|---|
| 2026-10-14 | 01 | In first join-cardinality answer, recognized matching rows matter but did not explicitly check uniqueness of the Customers join key | With unique customer_id in Customers, each Orders row matches at most one customer and INNER JOIN output is <= Orders row count; duplicate customer keys can multiply matches | Needs reinforcement |

## Assessment rules
- **Correctness:** Query matches the requirement and expected result.
- **Edge cases:** Handles zero matches, duplicate keys, NULLs and boundary dates when relevant.
- **Performance:** Avoids unnecessary work and explains the query plan where relevant.
- **Explanation:** States the grain, approach, risks and validation steps.
- **Completion:** A lesson is completed only after hands-on review and interview feedback.

## Notes
- First conceptual interview response received: **partially correct**. User reasoned that row counts depend on matches per customer; feedback clarified uniqueness of the customer-side join key and fanout.\n- The learner has not yet submitted the Day 01 coding solution.
- Add actual mistakes only after observing an attempt; do not assume weaknesses.
- Keep lesson notes and tracker synchronized after each completed topic.
