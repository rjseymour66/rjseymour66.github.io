---
title: "Subqueries"
linkTitle: "Subqueries"
weight: 90
description: >
  Subqueries in WHERE, HAVING, SELECT, and FROM, plus CTEs and recursive queries.
---

A *subquery* is a `SELECT` statement nested inside another SQL statement. Subqueries let you reference computed values and derived tables without intermediate storage. A subquery cannot include an `ORDER BY` clause.

## Where to introduce a subquery

A subquery can appear in four positions:

1. In a `WHERE` clause as a search condition
2. In a `HAVING` clause as a search condition
3. In the `FROM` clause as a table specification (called an *inline view*)
4. In the `SELECT` clause as a column specification

## Subqueries vs. JOINs

Most subqueries can be rewritten as joins, and vice versa. Choose the form that is clearest for the problem at hand:

Choose a join when:
- The `SELECT` clause needs columns from both tables
- The relationship is a natural primary key to foreign key match

Choose a subquery when:
- You need to pass an aggregate value to the main query
- The relationship is ad hoc rather than a defined key relationship
- The query logic is easier to read broken into stages

**As a join:**

```sql
SELECT invoice_number, invoice_date, invoice_total
FROM invoices JOIN vendors
   ON invoices.vendor_id = vendors.vendor_id
WHERE vendor_state = 'CA'
ORDER BY invoice_date;
```

**As a subquery:**

```sql
SELECT invoice_number, invoice_date, invoice_total
FROM invoices
WHERE vendor_id IN
   (SELECT vendor_id
   FROM vendors
   WHERE vendor_state = 'CA')
ORDER BY invoice_date;
```

## Subqueries in the WHERE clause

### IN operator

The `IN` operator tests whether a value is contained in the list returned by a subquery. The subquery must return a single column. The following returns all vendors that are not yet in the invoices table:

```sql
SELECT vendor_id, vendor_name, vendor_state
FROM vendors
WHERE vendor_id NOT IN
   (SELECT DISTINCT vendor_id
   FROM invoices)
ORDER BY vendor_id;
```

### Comparison operators

When a subquery uses a comparison operator, it must return a single value. Aggregate functions ensure the subquery collapses to one result. The following returns invoices with a balance below the average balance:

```sql
SELECT invoice_number, invoice_date,
   invoice_total - payment_total - credit_total AS balance_due
FROM invoices
WHERE invoice_total - payment_total - credit_total > 0
  AND invoice_total - payment_total - credit_total <
      (
         SELECT AVG(invoice_total - payment_total - credit_total)
         FROM invoices
         WHERE invoice_total - payment_total - credit_total > 0
      )
ORDER BY invoice_total DESC;
```

### ALL keyword

`ALL` modifies a comparison operator so the condition must be true for every value returned by the subquery. If the subquery returns no rows, the condition is always true.

The following equivalences illustrate how `ALL` behaves:

| Condition | Equivalent | Description |
|---|---|---|
| `x > ALL (1, 2)` | `x > 2` | True if x is greater than the maximum value |
| `x < ALL (1, 2)` | `x < 1` | True if x is less than the minimum value |
| `x = ALL (1, 2)` | `(x = 1) AND (x = 2)` | True if all subquery values equal x |
| `x <> ALL (1, 2)` | `x NOT IN (1, 2)` | True if x is not in the subquery result |

The following returns invoices larger than the largest invoice for vendor 34:

```sql
SELECT vendor_name, invoice_number, invoice_total
FROM invoices i JOIN vendors v ON i.vendor_id = v.vendor_id
WHERE invoice_total > ALL
   (SELECT invoice_total
    FROM invoices
    WHERE vendor_id = 34)
ORDER BY vendor_name;
```

### ANY and SOME keywords

`ANY` and `SOME` are interchangeable. The condition is true if it is true for at least one value returned by the subquery. Rewriting with `MAX` or `MIN` is usually clearer and performs better:

```sql
-- Using ANY
WHERE invoice_total < ANY
   (SELECT invoice_total
    FROM invoices
    WHERE vendor_id = 115)

-- Equivalent rewrite using MAX
WHERE invoice_total <
   (SELECT MAX(invoice_total)
    FROM invoices
    WHERE vendor_id = 115)
```

### Correlated subqueries

An *uncorrelated* subquery runs once for the entire query. A *correlated* subquery runs once for each row processed by the outer query, because it references a value from the outer query. This is similar to a loop.

The following returns invoices that exceed the average for their specific vendor. The subquery references `i.vendor_id` from the outer query, so it recalculates for each row:

```sql
SELECT vendor_id, invoice_number, invoice_total
FROM invoices i
WHERE invoice_total >
   (SELECT AVG(invoice_total)
    FROM invoices
    WHERE vendor_id = i.vendor_id)
ORDER BY vendor_id, invoice_total;
```

### EXISTS operator

`EXISTS` tests whether a subquery returns any rows. It is typically combined with a correlated subquery. The following returns all vendors that have no invoices:

```sql
SELECT vendor_id, vendor_name, vendor_state
FROM vendors
WHERE NOT EXISTS
   (SELECT *
    FROM invoices
    WHERE vendor_id = vendors.vendor_id)
```

## Subqueries in the FROM clause

Code a subquery in place of a table name to create an inline view. Assign an alias to the subquery and reference its column aliases in the outer query.

The following returns the largest invoice total per state by first computing totals per vendor, then finding the maximum per state:

```sql
SELECT vendor_state, MAX(sum_of_invoices) AS max_sum_of_invoices
FROM
(
   SELECT vendor_state, vendor_name,
      SUM(invoice_total) AS sum_of_invoices
   FROM vendors v JOIN invoices i
      ON v.vendor_id = i.vendor_id
   GROUP BY vendor_state, vendor_name
) t
GROUP BY vendor_state
ORDER BY vendor_state;
```

## Subqueries in the SELECT clause

Replace a column specification with a correlated subquery when the calculation requires data from another table. The subquery must return exactly one value per row. This can usually be rewritten as a join:

```sql
-- As a correlated subquery in SELECT
SELECT vendor_name,
   (SELECT MAX(invoice_date) FROM invoices
    WHERE vendor_id = vendors.vendor_id) AS latest_inv
FROM vendors
ORDER BY latest_inv DESC;
```

```sql
-- Equivalent rewrite as a LEFT JOIN
SELECT vendor_name, MAX(invoice_date) AS latest_inv
FROM vendors v
   LEFT JOIN invoices i ON v.vendor_id = i.vendor_id
GROUP BY vendor_name
ORDER BY latest_inv DESC;
```

## Complex queries with subqueries

### Procedure for building complex queries

Use this four-step approach to build queries that involve multiple subqueries or layers of aggregation:

1. State the problem in plain English. For example: "Which vendor in each state has the largest invoice total?"
2. Write pseudocode to outline the structure:

```sql
SELECT vendor_state, vendor_name, sum_of_invoices
FROM (subquery returning vendor_state, vendor_name, sum_of_invoices)
JOIN (subquery returning vendor_state, largest_sum_of_invoices)
   ON vendor_state AND sum_of_invoices
ORDER BY vendor_state;
```

3. Code and test each subquery independently.
4. Assemble and test the final query.

The following retrieves the vendor from each state with the largest invoice total:

```sql
SELECT t1.vendor_state, vendor_name, t1.sum_of_invoices
FROM
   (
      SELECT vendor_state, vendor_name,
         SUM(invoice_total) AS sum_of_invoices
      FROM vendors v JOIN invoices i
         ON v.vendor_id = i.vendor_id
      GROUP BY vendor_state, vendor_name
   ) t1
   JOIN
      (
         SELECT vendor_state,
            MAX(sum_of_invoices) AS sum_of_invoices
         FROM
         (
            SELECT vendor_state, vendor_name,
               SUM(invoice_total) AS sum_of_invoices
            FROM vendors v JOIN invoices i
               ON v.vendor_id = i.vendor_id
            GROUP BY vendor_state, vendor_name
         ) t2
         GROUP BY vendor_state
      ) t3
   ON t1.vendor_state = t3.vendor_state AND
      t1.sum_of_invoices = t3.sum_of_invoices
ORDER BY vendor_state;
```

## Common table expressions (CTEs)

A *CTE* creates one or more named temporary result sets that exist for the duration of the query. CTEs begin with the `WITH` keyword. Separate multiple CTEs with commas. CTEs can be applied with `SELECT`, `INSERT`, `UPDATE`, and `DELETE`, but most commonly appear with `SELECT`.

CTEs make complex, multi-subquery queries significantly easier to read and maintain. The following refactors the complex query above:

```sql
WITH summary AS
(
   SELECT vendor_state, vendor_name, SUM(invoice_total) AS sum_of_invoices
   FROM vendors v JOIN invoices i
      ON v.vendor_id = i.vendor_id
   GROUP BY vendor_state, vendor_name
),
top_in_state AS
(
   SELECT vendor_state, MAX(sum_of_invoices) AS sum_of_invoices
   FROM summary
   GROUP BY vendor_state
)
SELECT summary.vendor_state, summary.vendor_name,
      top_in_state.sum_of_invoices
FROM summary JOIN top_in_state
   ON summary.vendor_state = top_in_state.vendor_state AND
      summary.sum_of_invoices = top_in_state.sum_of_invoices
ORDER BY summary.vendor_state;
```

## Recursive CTEs

A *recursive CTE* loops through a result set and processes it iteratively to produce a final result. Recursive CTEs are commonly applied to hierarchical data, such as organizational charts where each employee row references a manager.

A recursive CTE consists of two parts joined by `UNION ALL`:
1. A non-recursive anchor query that returns the starting rows
2. A recursive query that joins the CTE back to itself

The following builds an employee hierarchy by traversing a `manager_id` relationship:

```sql
WITH RECURSIVE employees_cte AS
(
   -- Non-recursive anchor: top-level employees with no manager
   SELECT employee_id,
      CONCAT(first_name, ' ', last_name) AS employee_name,
      1 AS ranking
   FROM employees
   WHERE manager_id IS NULL
UNION ALL
   -- Recursive part: employees reporting to rows already in the CTE
   SELECT employees.employee_id,
      CONCAT(first_name, ' ', last_name),
      ranking + 1
   FROM employees
      JOIN employees_cte
      ON employees.manager_id = employees_cte.employee_id
)
SELECT *
FROM employees_cte
ORDER BY ranking, employee_id;
```

Each recursive iteration adds the direct reports of employees already in the CTE until no new rows are found.
