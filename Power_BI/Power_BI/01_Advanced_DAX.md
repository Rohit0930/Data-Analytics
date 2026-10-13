# 01 — Advanced DAX: Mastery Notes (7 Years Experience)

> Living interview-preparation notes. Work through **one lesson at a time**, test every example in Power BI, and add your own project scenarios.

## Learning roadmap

1. Row context vs filter context (current lesson)
2. Context transition and CALCULATE
3. ALL, ALLSELECTED, REMOVEFILTERS, ALLEXCEPT, KEEPFILTERS
4. Iterators: SUMX, AVERAGEX, RANKX
5. Virtual tables: FILTER, ADDCOLUMNS, SUMMARIZE, SUMMARIZECOLUMNS
6. Relationships in DAX: USERELATIONSHIP, CROSSFILTER, TREATAS
7. Time intelligence: fiscal periods, rolling windows, prior periods
8. Calculation groups, field parameters and dynamic format strings
9. Semi-additive measures, ARR bridges, churn and cohorts
10. DAX Studio, server timings, query plans, debugging wrong totals

---

## Lesson 1 — Row context vs filter context

### 1. Example data

Assume a table called `Sales`:

| OrderID | Region | Product | Amount |
|---|---|---|---:|
| 1 | North | A | 100 |
| 2 | North | B | 200 |
| 3 | South | A | 150 |
| 4 | South | B | 50 |

Grand total: **500**. North: **300**. South: **200**.

### 2. Row context: "Which row am I on?"

Row context means an expression can refer to values of the current row. It is created by a **calculated column** or by an **iterator** such as `SUMX`.

Calculated column (create using **New column** in the Sales table):

```dax
Commission = Sales[Amount] * 0.10
```

Output for the four rows: 10, 20, 15, 5.

Iterator measure (create using **New measure**):

```dax
Commission Total =
SUMX(
    Sales,
    Sales[Amount] * 0.10
)
```

The iterator processes each visible row and sums the result. With no slicers, it returns **50**; with Region = North, it returns **30**.

**Important:** Row context by itself does **not** automatically filter the entire Sales table or a related fact table.

### 3. Filter context: "Which rows are currently included?"

Filter context comes from slicers, visual rows/columns, page/report filters, relationships, and explicit DAX filters such as `CALCULATE`.

Measure:

```dax
Total Sales = SUM(Sales[Amount])
```

- No filter: **500**
- Region = North: **300**
- Region = South: **200**
- Region = North and Product = A: **100**

A measure is evaluated under the filter context of the visual cell. A measure does not inherently create a row context.

### 4. Row context + filter context can exist together

```dax
Commission Total =
SUMX(
    Sales,
    Sales[Amount] * 0.10
)
```

If a North slicer is selected, the **filter context** restricts Sales to the two North rows; `SUMX` then creates a **row context** for each of those rows.

### 5. Why a calculated column can produce a surprising grand total

Suppose `Customers` has one row per customer and a one-to-many active relationship to `Sales` through `CustomerID`.

A calculated column on Customers:

```dax
Incorrect Customer Total = SUM(Sales[Amount])
```

This expression has a row context over Customers but does not turn that current customer into a filter on Sales; it may return the same overall total for every customer.

Instead:

```dax
Customer Total =
CALCULATE(
    SUM(Sales[Amount])
)
```

Inside the calculated column, `CALCULATE` performs **context transition**: it converts the current Customers row context into filter context, which can then propagate through the relationship to Sales. We will examine the details and exceptions in Lesson 2.

### 6. Frequent senior-interview mistakes

- Saying "row context means the current row in a table visual" — a visual row normally contributes **filter context** to a measure.
- Assuming `SUMX` is always slower than `SUM` — performance depends on the expression and query plan.
- Assuming row context automatically filters related tables.
- Confusing a calculated column (computed at refresh) with a measure (evaluated at query time).
- Ignoring the active relationships and current filters when explaining a measure.

### 7. Interview-ready answer (about 40 seconds)

> Row context tells DAX which row is currently being evaluated. We typically get it in calculated columns and iterator functions like SUMX. Filter context determines which rows are included in a calculation, based on slicers, visual selections, relationships and DAX filters. For example, a Total Sales measure returns 500 across all regions but 300 when North is selected. A SUMX calculation can have both contexts at once: the slicer filters the rows, and SUMX iterates through them. CALCULATE is important because it can modify filter context and perform context transition when row context exists.

### 8. Practice questions

**Question A.** Using the four Sales rows above, what does `Total Sales = SUM(Sales[Amount])` return in a matrix with Region on rows? Explain why.

**Question B.** Why does a calculated column on Customers containing `SUM(Sales[Amount])` not automatically return that customer's sales, even when a relationship exists?

**Question C.** With Region = North, what does `SUMX(Sales, Sales[Amount] * 0.10)` return? Which two contexts are involved?

### 9. Hands-on task

1. In Power BI Desktop, use **Enter data** to create the Sales table above.
2. Create the calculated column `Commission`.
3. Create the measures `Total Sales` and `Commission Total`.
4. Build a matrix with Region on rows and the two measures as values.
5. Add a Product slicer and predict the values **before** changing it.

**Expected matrix without slicers:**

| Region | Total Sales | Commission Total |
|---|---:|---:|
| North | 300 | 30 |
| South | 200 | 20 |
| Total | 500 | 50 |

---

## Next lesson

**Lesson 2: CALCULATE and context transition** — including nested filters, `FILTER`, `KEEPFILTERS`, and debugging incorrect totals.

## Reference

- [Microsoft Learn — DAX overview](https://learn.microsoft.com/en-us/dax/dax-overview)
- [Microsoft Learn — CALCULATE](https://learn.microsoft.com/en-us/dax/calculate-function-dax)
- [Microsoft Learn — SUMX](https://learn.microsoft.com/en-us/dax/sumx-function-dax)
