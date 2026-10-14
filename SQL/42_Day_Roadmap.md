# Senior SQL Mastery — 42-Day Roadmap

**Start:** 2026-10-14  
**Database:** PostgreSQL  
**Level:** Senior / Lead Data Analyst, BI Analyst, Analytics Engineer  
**Daily time:** 60–90 minutes  
**Learning method:** One concept at a time → worked example → coding challenge → interview explanation → feedback → GitHub notes.

## Learning outcomes
- Write correct SQL under complex join, date, windowing, and aggregation requirements.
- Explain grain, cardinality, data correctness, edge cases, and performance trade-offs.
- Design PostgreSQL data models and SQL-based data engineering workflows.
- Handle senior-level business cases in marketing analytics, ARR/churn, and retail.
- Communicate solutions clearly in interviews.

## Week 1 — Joins, Subqueries & CTEs
| Day | Topic |
|---|---|
| 01 | Join fundamentals, grain, cardinality and fanout |
| 02 | Advanced joins: self, full, cross and non-equi |
| 03 | Semi-joins, anti-joins, EXISTS and NOT EXISTS |
| 04 | Many-to-many joins and duplicate prevention |
| 05 | Scalar, correlated and nested subqueries |
| 06 | CTEs, recursive CTEs and materialization |
| 07 | Revision and scenario-based mock interview |

## Week 2 — Advanced Window Functions
| Day | Topic |
|---|---|
| 08 | OVER, PARTITION BY and ORDER BY |
| 09 | ROW_NUMBER, RANK, DENSE_RANK and NTILE |
| 10 | LAG, LEAD and period comparisons |
| 11 | Running totals and moving averages |
| 12 | Window frames, FIRST_VALUE and LAST_VALUE |
| 13 | Gaps and islands, consecutive activity |
| 14 | Window-function interview assessment |

## Week 3 — Date Logic & Business Analytics
| Day | Topic |
|---|---|
| 15 | Date arithmetic, intervals and boundaries |
| 16 | Dormant customers and as-of-date analysis |
| 17 | Cohort retention and churn analysis |
| 18 | Customer reactivation and win-back logic |
| 19 | Marketing funnel and conversion analysis |
| 20 | Subscription ARR movement and reconciliation |
| 21 | Business case-study mock interview |

## Week 4 — PostgreSQL & Performance
| Day | Topic |
|---|---|
| 22 | PostgreSQL data types, NULLs and casting |
| 23 | Transactions, constraints and isolation |
| 24 | Stored procedures, functions and PL/pgSQL |
| 25 | Dynamic SQL, exception handling and temp tables |
| 26 | B-tree, composite, partial and expression indexes |
| 27 | EXPLAIN ANALYZE and query optimization |
| 28 | Query-tuning interview assessment |

## Week 5 — Data Modeling & Engineering SQL
| Day | Topic |
|---|---|
| 29 | Normalization, keys and data grain |
| 30 | Star schema, fact/dimension tables and SCD |
| 31 | Partitioning, materialized views and maintenance |
| 32 | Data quality, deduplication and reconciliation |
| 33 | Incremental loads, UPSERT and CDC patterns |
| 34 | SQL security, permissions and row-level security |
| 35 | Data modeling and pipeline mock interview |

## Week 6 — Senior/Lead SQL Scenarios
| Day | Topic |
|---|---|
| 36 | Large-table joins, statistics and tuning trade-offs |
| 37 | Concurrency, locks and deadlocks |
| 38 | End-to-end marketing analytics SQL case |
| 39 | SaaS revenue and customer lifecycle case |
| 40 | Retail transactions and dormant-customer case |
| 41 | End-to-end analytical database design |
| 42 | Final senior/lead SQL interview assessment |

## Daily routine
1. **Concept (15 min):** Definition, intuition, when to use, and pitfalls.
2. **Walkthrough (20 min):** Reproducible PostgreSQL data, complete examples, expected outputs.
3. **Coding challenge (20 min):** Learner submits an independent query before receiving an ideal solution.
4. **Interview practice (15 min):** One question at a time, with feedback on clarity and senior-level depth.
5. **Revision and notes (10 min):** Record mistakes, edge cases, optimizations and follow-up exercises.

## Repository workflow
- Lessons are saved in topic folders beneath `SQL/`.
- `Progress_Tracker.md` records topic status, mistakes, and revision work.
- Exercises remain **unsolved** in notes until the learner submits an attempt.
- When a solution is reviewed, add the corrected query and explanations to the relevant lesson.
- Never mark a topic complete without finishing the exercise and review.
