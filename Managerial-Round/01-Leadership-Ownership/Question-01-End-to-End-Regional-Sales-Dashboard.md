# Question 01 — End-to-End Ownership of Regional Sales Performance Dashboard
Date: 2026-10-13
Category: Leadership & Ownership
Status: Practiced — needs refinement

## Interview question
Tell me about a time when you took ownership of a project from start to finish. What challenges did you face, and how did you ensure its success?

## Original answer — faithful summary
For a technology brand, five regions had separate quarterly sales analyses. Leadership wanted a consolidated one-page view. We began with a POC using Excel/CSV data and built a Power BI dashboard, with Python for initial ETL. We defined metrics including leads, targeted leads, opportunities, revenue, conversion percentages, revenue per lead and customer value buckets. The funnel assessed eligibility and potential using geographical details, sales-rep assignment/capacity, targeting and lines of business. We built global and regional drilldowns by region, subregion, line of business and campaign type. After the POC became a full project, we obtained database access, migrated ETL logic to SQL functions, scheduled orchestration in Airflow for Mondays at 9 AM, and refreshed reporting. Leaders gained a consolidated view to compare regions and guide next-quarter planning.

## Feedback (provisional score 7/10)
Strengths: concrete business problem; credible POC-to-production journey; detailed funnel knowledge; Python, SQL, Airflow and Power BI; clear leadership use case.
Improve: open with your personal ownership, not 'we'; explain decisions and tradeoffs; simplify the lengthy funnel explanation; distinguish your contribution from the team's; add a verified outcome (time saved, adoption, refresh reliability, faster planning) if available. Avoid asserting unverified numerical impact.

## Interview-ready answer (draft; confirm personal responsibilities)
“One project that stands out was a quarterly sales-performance reporting solution for a technology brand operating across five regions. Each region was being analysed separately, which made it difficult for leadership to compare performance and plan at a global level.

Our objective was to bring those regional views into one consistent Power BI dashboard. I was involved from the proof-of-concept stage through production delivery. We started with Excel and CSV extracts, used Python to prepare the data, and built a POC in Power BI. An important part of the work was agreeing on common KPIs—such as targeted leads, opportunities created, conversion rates and revenue—so the five regions could be compared fairly.

We also designed a sales-funnel view to show how leads progressed toward opportunities and revenue, including filters for region, subregion, line of business and campaign type. Once the POC was accepted and we received database access, we moved the ETL logic into SQL functions and used Airflow to schedule the weekly pipeline, with the dashboard refreshed for Monday reporting.

The outcome was a consolidated view that allowed leaders to identify stronger and weaker regions, understand funnel drop-offs, and use the findings in next-quarter planning. For me, the key learning was that owning a BI solution means more than building visuals: it means aligning KPI definitions, proving the approach, productionising the pipeline and making sure the final report supports decisions.”

## Revision cues
Problem: five disconnected regional views.
Action: POC → shared KPIs → funnel dashboard → production SQL/Airflow.
Outcome: one consistent view for regional comparisons and quarterly planning.
Clarify later: exact personal ownership; measurable impact; one significant challenge and how you resolved it.
