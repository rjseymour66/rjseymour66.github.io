---
title: "Window functions"
linkTitle: "Window functions"
weight: 80
description: >
  OVER, PARTITION BY, frames, ranking, and analytic functions.
---

*Window functions* apply an aggregate or ranking calculation across a set of rows related to the current row, without collapsing those rows into a single output row. Unlike `GROUP BY`, which returns one row per group, window functions preserve every row in the result set while adding a calculated column alongside it. This makes them ideal for running totals, ranking within categories, comparing a value to the previous period, and percentile calculations.

## Aggregate window functions

Add the `OVER` clause to any aggregate function to turn it into a window function. The `OVER` clause defines the *window*: the set of rows the function sees for each output row.

`OVER()` with empty parentheses applies the function across all rows in the result set. Every row in the result receives the same value:

```sql
SELECT vendor_id, invoice_date, invoice_total,
   SUM(invoice_total) OVER() AS total_invoices,
   SUM(invoice_total) OVER(PARTITION BY vendor_id) AS vendor_total
FROM invoices
WHERE invoice_total > 5000;
```

`OVER()` calculates the sum of all rows. `OVER(PARTITION BY vendor_id)` calculates a separate sum for each vendor. Both columns appear on every row alongside the original data.

### ORDER BY within OVER

Adding `ORDER BY` inside `OVER(PARTITION BY ...)` changes the calculation from a full-group total to a *cumulative running total*. Each row accumulates values from the first row of the partition through the current row:

```sql
SELECT vendor_id, invoice_date, invoice_total,
   SUM(invoice_total) OVER() AS total_invoices,
   SUM(invoice_total) OVER(PARTITION BY vendor_id
      ORDER BY invoice_total) AS vendor_total
FROM invoices
WHERE invoice_total > 5000;
```

## Frames

A *frame* defines a subset of the current partition relative to the current row. Frames let you calculate rolling windows, such as a 3-month moving average. Because a frame is relative to the current row, it shifts as the query moves through the partition.

Specify a frame with `ROWS` (by row count) or `RANGE` (by value distance). The following frame values are available:

| Value | Description |
|---|---|
| `CURRENT ROW` | The frame starts or ends with the current row |
| `UNBOUNDED PRECEDING` | The frame starts or ends with the first row in the partition |
| `UNBOUNDED FOLLOWING` | The frame starts or ends with the last row in the partition |
| `expr PRECEDING` | The frame starts `expr` rows or values before the current row |
| `expr FOLLOWING` | The frame starts `expr` rows or values after the current row |

To define both a start and end point, apply `BETWEEN`. The starting point must not come after the ending point.

### ROWS frame

The following calculates a running total from the start of each vendor's partition through the current row:

```sql
SELECT vendor_id, invoice_date, invoice_total,
   SUM(invoice_total) OVER() AS total_invoices,
   SUM(invoice_total) OVER(PARTITION BY vendor_id ORDER BY invoice_date
      ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS vendor_total
FROM invoices
WHERE invoice_date BETWEEN '2018-04-01' AND '2018-04-30';
```

### RANGE frame

`RANGE` includes *peers*: rows that share the same value in the sort column. The following applies a range-based frame that includes all preceding rows and peers of the current row:

```sql
SELECT vendor_id, invoice_date, invoice_total,
   SUM(invoice_total) OVER() AS total_invoices,
   SUM(invoice_total) OVER(PARTITION BY vendor_id ORDER BY invoice_date
      RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS vendor_total
FROM invoices
WHERE invoice_date BETWEEN '2018-04-01' AND '2018-04-30';
```

### 3-month moving average

The following calculates a 3-month rolling average of monthly invoice totals. The `1 PRECEDING AND 1 FOLLOWING` frame covers the month before, the current month, and the month after. If a preceding or subsequent month does not exist, the average is calculated over the available rows:

```sql
SELECT MONTH(invoice_date) AS month, SUM(invoice_total) AS total_invoices,
   ROUND(AVG(SUM(invoice_total)) OVER(ORDER BY MONTH(invoice_date)
      RANGE BETWEEN 1 PRECEDING AND 1 FOLLOWING), 2) AS 3_month_avg
FROM invoices
GROUP BY MONTH(invoice_date);
```

## Named windows

When two or more functions in the same query share the same window definition, define a named window with the `WINDOW` clause. Place `WINDOW` after `HAVING` and before `ORDER BY`. Named windows work like column aliases.

```sql
SELECT vendor_id, invoice_date, invoice_total,
   SUM(invoice_total) OVER vendor_window AS vendor_total,
   ROUND(AVG(invoice_total) OVER vendor_window, 2) AS vendor_avg,
   MAX(invoice_total) OVER vendor_window AS vendor_max,
   MIN(invoice_total) OVER vendor_window AS vendor_min
FROM invoices
WHERE invoice_total > 5000
WINDOW vendor_window AS (PARTITION BY vendor_id);
```

`WINDOW vendor_window AS (PARTITION BY vendor_id)` defines the window once. Each `OVER vendor_window` clause references it by name.

## Ranking functions

*Ranking functions* are non-aggregate window functions that assign a position to each row within its partition. They are useful for leaderboards, pagination by rank, and top-N-per-group queries.

### ROW_NUMBER

`ROW_NUMBER` assigns a unique sequential number to each row within a partition, starting at 1. The sequence is determined by the `ORDER BY` inside `OVER`:

```sql
SELECT ROW_NUMBER() OVER(PARTITION BY vendor_state
   ORDER BY vendor_name) AS row_number,
   vendor_name, vendor_state
FROM vendors;
```

### RANK and DENSE_RANK

Both functions return the rank of each row. When rows tie, both functions give them the same rank.

`RANK` skips rank numbers after a tie. For example, two rows tied at rank 2 are both ranked 2, and the next row is ranked 4. `DENSE_RANK` does not skip: the next row after a tie gets the next consecutive rank.

```sql
SELECT RANK() OVER(ORDER BY invoice_total) AS rank,
   DENSE_RANK() OVER(ORDER BY invoice_total) AS dense_rank,
   invoice_total, invoice_number
FROM invoices;
```

### NTILE

`NTILE(n)` divides the rows in a partition into `n` equal groups and returns the group number for each row. This is useful for bucketing values into percentile groups:

```sql
SELECT terms_description,
   NTILE(2) OVER(ORDER BY terms_id) AS tile2,
   NTILE(3) OVER(ORDER BY terms_id) AS tile3,
   NTILE(4) OVER(ORDER BY terms_id) AS tile4
FROM terms;
```

## Analytic functions

*Analytic functions* perform calculations on ordered sets of data. Unlike ranking functions, they return values from specific rows in the window rather than positional ranks.

### FIRST_VALUE, NTH_VALUE, and LAST_VALUE

These functions return the first, nth, and last values in an ordered set. When using `LAST_VALUE` or `NTH_VALUE` with `PARTITION BY`, include a `ROWS` or `RANGE` clause to define the full partition as the frame. Otherwise the default frame ends at the current row, and `LAST_VALUE` returns the current row's value instead of the partition's last row.

The following returns the top, second, and bottom sales rep per year:

```sql
SELECT sales_year,
   CONCAT(rep_first_name, ' ', rep_last_name) AS rep_name,
   sales_total,
   FIRST_VALUE(CONCAT(rep_first_name, ' ', rep_last_name))
       OVER(PARTITION BY sales_year ORDER BY sales_total DESC)
       AS highest_sales,
   NTH_VALUE(CONCAT(rep_first_name, ' ', rep_last_name), 2)
       OVER(PARTITION BY sales_year ORDER BY sales_total DESC
       RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
       AS second_highest_sales,
   LAST_VALUE(CONCAT(rep_first_name, ' ', rep_last_name))
       OVER(PARTITION BY sales_year ORDER BY sales_total DESC
       RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
       AS lowest_sales
FROM sales_totals JOIN sales_reps ON sales_totals.rep_id = sales_reps.rep_id;
```

### LAG

`LAG` returns a value from a row that precedes the current row by a specified offset. This is the standard way to compute period-over-period change:

```sql
SELECT rep_id, sales_year, sales_total AS current_sales,
   LAG(sales_total, 1, 0) OVER(PARTITION BY rep_id ORDER BY sales_year) AS last_sales,
   sales_total - LAG(sales_total, 1, 0)
      OVER(PARTITION BY rep_id ORDER BY sales_year) AS change
FROM sales_totals;
```

The `OVER` clause groups by `rep_id` and orders by `sales_year`. `LAG(sales_total, 1, 0)` returns the value from the previous row. The third argument (`0`) is the default returned when no preceding row exists.

### PERCENT_RANK and CUME_DIST

`PERCENT_RANK` calculates the relative rank of each row as a percentage. The first row in a partition always returns 0 and the last always returns 1.

`CUME_DIST` calculates what fraction of rows in the partition have a value less than or equal to the current row's value.

```sql
SELECT sales_year, rep_id, sales_total,
   PERCENT_RANK() OVER(PARTITION BY sales_year ORDER BY sales_total) AS pct_rank,
   CUME_DIST() OVER(PARTITION BY sales_year ORDER BY sales_total) AS cume_dist
FROM sales_totals;
```
