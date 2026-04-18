---
title: "Aggregates"
linkTitle: "Aggregates"
weight: 70
description: >
  Aggregate functions, GROUP BY, HAVING, ROLLUP, and GROUPING.
---

*Aggregate functions* operate on a series of values and return a single summary value. A query that contains one or more aggregate functions is called a *summary query*. Aggregates power reporting, dashboards, and any feature that needs counts, totals, or averages from a set of rows — for example, calculating total revenue per vendor or finding the average invoice amount for a given month.

## Aggregate functions

The following aggregate functions are available:

| Function | Result |
|---|---|
| `AVG([ALL \| DISTINCT] expression)` | Average of all non-null values in the expression |
| `SUM([ALL \| DISTINCT] expression)` | Total of all non-null values in the expression |
| `MIN([ALL \| DISTINCT] expression)` | Lowest non-null value in the expression |
| `MAX([ALL \| DISTINCT] expression)` | Highest non-null value in the expression |
| `COUNT([ALL \| DISTINCT] expression)` | Number of non-null values in the expression |
| `COUNT(*)` | Number of rows selected by the query |

Notes on behavior:

- `ALL` is the default and includes all values except nulls.
- `COUNT(*)` is the only form that includes null values.
- Apply `DISTINCT` to exclude duplicate values. It has no effect on `MIN` or `MAX`.

The expression in each function is typically a column name.

The following counts all invoices with a balance due and calculates the total amount owed:

```sql
SELECT COUNT(*) AS number_of_invoices,
   SUM(invoice_total - payment_total - credit_total) AS total_due
FROM invoices
WHERE invoice_total - payment_total - credit_total > 0;
```

The following counts invoices, calculates the average, and calculates the total, all filtered by date:

```sql
SELECT 'After 1/1/2018' AS selection_date,
   COUNT(*) AS number_of_invoices,
   ROUND(AVG(invoice_total), 2) AS avg_invoice_amt,
   SUM(invoice_total) AS total_invoice_amt
FROM invoices
WHERE invoice_date > '2018-01-01';
```

The following distinguishes unique vendors from total invoice counts by combining `COUNT(DISTINCT ...)` and `COUNT(...)`:

```sql
SELECT COUNT(DISTINCT vendor_id) AS number_of_vendors,
   COUNT(vendor_id) AS number_of_invoices,
   ROUND(AVG(invoice_total), 2) AS avg_invoice_amt,
   SUM(invoice_total) AS total_invoice_amt
FROM invoices
WHERE invoice_date > '2018-01-01';
```

## GROUP BY and HAVING

The `GROUP BY` clause groups rows by the values in one or more columns. When you add aggregate functions to the `SELECT` clause, each aggregate is calculated per group rather than for the entire result set. A single row is returned for each unique combination of grouped values.

The `HAVING` clause filters groups after aggregation, the same way `WHERE` filters individual rows before aggregation. Both clauses appear after `WHERE` and before `ORDER BY`.

The following groups invoices by vendor and returns only vendors whose average invoice total exceeds 2000:

```sql
SELECT vendor_id, ROUND(AVG(invoice_total), 2) AS average_invoice_amount
FROM invoices
GROUP BY vendor_id
HAVING AVG(invoice_total) > 2000
ORDER BY average_invoice_amount DESC;
```

The following joins vendors and invoices, then groups by vendor name:

```sql
SELECT vendor_name, vendor_state,
   ROUND(AVG(invoice_total), 2) AS average_invoice_amount
FROM vendors JOIN invoices ON vendors.vendor_id = invoices.vendor_id
GROUP BY vendor_name
HAVING AVG(invoice_total) > 2000
ORDER BY average_invoice_amount DESC;
```

The following groups by two columns to produce a city-level summary, then filters for groups with at least two invoices:

```sql
SELECT vendor_state, vendor_city, COUNT(*) AS invoice_qty,
   ROUND(AVG(invoice_total), 2) AS invoice_avg
FROM invoices JOIN vendors
   ON invoices.vendor_id = vendors.vendor_id
GROUP BY vendor_state, vendor_city
HAVING COUNT(*) >= 2
ORDER BY vendor_state, vendor_city;
```

### HAVING vs WHERE

`WHERE` applies to every individual row before grouping. `HAVING` applies to each group after aggregation. The following two queries illustrate the distinction:

The first applies `HAVING` to filter groups:

```sql
SELECT vendor_name,
   COUNT(*) AS invoice_qty,
   ROUND(AVG(invoice_total), 2) AS invoice_avg
FROM vendors JOIN invoices
   ON vendors.vendor_id = invoices.vendor_id
GROUP BY vendor_name
HAVING AVG(invoice_total) > 500
ORDER BY invoice_qty DESC;
```

The second applies `WHERE` to filter individual rows before they are grouped:

```sql
SELECT vendor_name,
   COUNT(*) AS invoice_qty,
   ROUND(AVG(invoice_total), 2) AS invoice_avg
FROM vendors JOIN invoices
   ON vendors.vendor_id = invoices.vendor_id
WHERE invoice_total > 500
GROUP BY vendor_name
ORDER BY invoice_qty;
```

You can apply non-aggregate filter conditions in either clause. For readability, place all filters that relate to groups in `HAVING`.

## WITH ROLLUP

Add `WITH ROLLUP` to the `GROUP BY` clause to append one or more summary rows to the result set. The summary row totals all the aggregate columns. Columns that cannot be summarized contain `NULL`.

```sql
SELECT vendor_id, COUNT(*) AS invoice_count,
   SUM(invoice_total) AS invoice_total
FROM invoices
GROUP BY vendor_id WITH ROLLUP;
```

## GROUPING function

When `WITH ROLLUP` generates summary rows, it fills non-grouped columns with `NULL`. This can be ambiguous if the data itself contains nulls. The `GROUPING` function returns `1` when a null was produced by the rollup (summary row) and `0` otherwise. Combine it with `IF` to replace rollup nulls with descriptive labels:

```sql
SELECT IF(GROUPING(invoice_date) = 1, 'Grand totals', invoice_date) AS invoice_date,
   IF(GROUPING(payment_date) = 1, 'Invoice date totals', payment_date) AS payment_date,
   SUM(invoice_total) AS invoice_total,
   SUM(invoice_total - credit_total - payment_total) AS balance_due
FROM invoices
WHERE invoice_date BETWEEN '2018-07-24' AND '2018-07-31'
GROUP BY invoice_date, payment_date WITH ROLLUP;
```

The first `IF` displays `'Grand totals'` in the `invoice_date` column for the final summary row. The second displays `'Invoice date totals'` in the `payment_date` column for subtotal rows.
