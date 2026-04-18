---
title: "Joins"
linkTitle: "Joins"
weight: 50
description: >
  Combining data from multiple tables with JOIN, UNION, and related syntax.
---

Joins retrieve data from two or more tables and combine it into a single result set. You will rely on joins constantly when working with normalized databases, where data is split across related tables to avoid duplication. For example, an invoices table stores vendor IDs rather than repeating vendor names on every row. A join reconnects that data at query time.

## INNER JOIN

An *inner join* returns only the rows where the join condition is satisfied in both tables. This is the most common type of join.

Specify the two tables in the `FROM` clause with the `JOIN` keyword and an `ON` phrase that names the matching columns. Prefix column names with the table name when the same column name appears in both tables. These are called *qualified columns*.

```sql
SELECT invoice_number, vendor_name
FROM vendors INNER JOIN invoices
   ON vendors.vendor_id = invoices.vendor_id
ORDER BY invoice_number;
```

The `INNER` keyword is optional. `JOIN` alone produces an inner join.

### Table aliases

A table alias is a short alternative name assigned to a table in the `FROM` clause. Aliases reduce repetition when qualifying column names:

```sql
SELECT invoice_number, vendor_name, invoice_due_date,
   invoice_total - payment_total - credit_total AS balance_due
FROM vendors v JOIN invoices i
   ON v.vendor_id = i.vendor_id
WHERE invoice_total - payment_total - credit_total > 0
ORDER BY invoice_due_date DESC;
```

### Implicit syntax

An older syntax places table names in the `FROM` clause separated by commas and moves the join condition into `WHERE`. Prefer the explicit `JOIN...ON` syntax for readability:

```sql
SELECT invoice_number, vendor_name
FROM vendors v, invoices i
WHERE v.vendor_id = i.vendor_id
ORDER BY invoice_number;
```

### Joining a table in another database

To join a table from a different database, qualify the table name with the database name:

```sql
SELECT vendor_name, customer_last_name, customer_first_name,
   vendor_state AS state, vendor_city AS city
FROM vendors v
   JOIN om.customers c
   ON v.vendor_zip_code = c.customer_zip
ORDER BY state, city;
```

Here `om` is the database name, `customers` is the table name, and `c` is the alias.

### Compound join

Create two or more comparisons in a join condition with `AND` or `OR`. This is useful when no single column uniquely identifies a match between two tables:

```sql
SELECT customer_first_name, customer_last_name
FROM customers c JOIN employees e
   ON c.customer_first_name = e.first_name
   AND c.customer_last_name = e.last_name;
```

This returns customers whose first and last names both appear in the employees table.

### Self join

A self join connects a table to itself. This is useful for hierarchical data, such as finding all other vendors in the same city, or building an org chart from a single employees table:

```sql
SELECT DISTINCT v1.vendor_name, v1.vendor_city, v1.vendor_state
FROM vendors v1 JOIN vendors v2
   ON v1.vendor_city = v2.vendor_city
   AND v1.vendor_state = v2.vendor_state
   AND v1.vendor_name != v2.vendor_name
ORDER BY v1.vendor_state, v1.vendor_city;
```

Aliases distinguish one instance of the table from the other.

### Joining more than two tables

Chain multiple joins to combine three or more tables. Join each table on the column it shares with the previous table in the chain:

```sql
SELECT vendor_name, invoice_number, invoice_date,
   line_item_amount, account_description
FROM vendors v
   JOIN invoices i
      ON v.vendor_id = i.vendor_id
   JOIN invoice_line_items li
      ON i.invoice_id = li.invoice_id
   JOIN general_ledger_accounts gl
      ON li.account_number = gl.account_number
WHERE invoice_total - payment_total - credit_total > 0
ORDER BY vendor_name, line_item_amount DESC;
```

## OUTER JOIN

An *outer join* returns all rows from one table, plus any matching rows from the other. Where no match exists, null values fill the unmatched columns. This is useful when you want to include all records from one side of a relationship regardless of whether a match exists on the other side — for example, listing all vendors, including those that have not yet generated any invoices.

### LEFT and RIGHT outer joins

`LEFT JOIN` returns all rows from the first (left) table. `RIGHT JOIN` returns all rows from the second (right) table.

The following returns all vendors, including those with no matching invoices:

```sql
SELECT vendor_name, invoice_number, invoice_total
FROM vendors LEFT JOIN invoices
   ON vendors.vendor_id = invoices.vendor_id
ORDER BY vendor_name;
```

You can combine inner and outer joins in a single query:

```sql
SELECT department_name, last_name, project_number
FROM departments d
   JOIN employees e
      ON d.department_number = e.department_number
   LEFT JOIN projects p
      ON e.employee_id = p.employee_id
ORDER BY department_name, last_name;
```

## USING clause

Replace the `ON` clause with `USING` when the join columns have the same name in both tables. This is called an *equijoin*. To join on multiple columns, list them inside the parentheses:

```sql
SELECT invoice_number, vendor_name
FROM vendors
   JOIN invoices USING (vendor_id)
ORDER BY invoice_number;
```

```sql
SELECT department_name, last_name, project_number
FROM departments
   JOIN employees USING (department_number)
   LEFT JOIN projects USING (employee_id)
ORDER BY department_name;
```

## NATURAL JOIN

`NATURAL JOIN` automatically joins two tables on all columns that share the same name. Avoid this in production code since schema changes can silently alter join behavior:

```sql
SELECT department_name AS dept_name, last_name, project_number
FROM departments
   NATURAL JOIN employees
   LEFT JOIN projects USING (employee_id)
ORDER BY department_name;
```

## CROSS JOIN

A *cross join* returns the Cartesian product: every row from the first table paired with every row from the second table. The result set grows quickly (rows in table A multiplied by rows in table B). Cross joins are occasionally useful for generating test data or all possible combinations:

```sql
SELECT departments.department_number, department_name, employee_id, last_name
FROM departments CROSS JOIN employees
ORDER BY departments.department_number;
```

## UNION

`UNION` combines the result sets of two or more `SELECT` statements into a single result set. Each `SELECT` must return the same number of columns, and corresponding columns must have compatible data types. Column names in the final result set come from the first `SELECT`.

By default, `UNION` eliminates duplicate rows. Add `ALL` to include duplicates. Add `ORDER BY` after the last `SELECT` to sort the combined results.

A common real-world application is categorizing records by status and combining the categories into one output. The following classifies invoices by payment tier:

```sql
SELECT invoice_number, vendor_name, '33% Payment' AS payment_type,
   invoice_total AS total, invoice_total * 0.333 AS payment
FROM invoices JOIN vendors
   ON invoices.vendor_id = vendors.vendor_id
WHERE invoice_total > 10000
UNION
SELECT invoice_number, vendor_name, '50% Payment' AS payment_type,
   invoice_total AS total, invoice_total * 0.5 AS payment
FROM invoices JOIN vendors
   ON invoices.vendor_id = vendors.vendor_id
WHERE invoice_total BETWEEN 500 AND 10000
UNION
SELECT invoice_number, vendor_name, 'Full amount' AS payment_type,
   invoice_total AS total, invoice_total AS payment
FROM invoices JOIN vendors
   ON invoices.vendor_id = vendors.vendor_id
WHERE invoice_total < 500
ORDER BY payment_type, vendor_name, invoice_number;
```

You can also union results from the same table to add a categorical label to each row:

```sql
SELECT 'Active' AS source, invoice_number, invoice_date, invoice_total
FROM invoices
WHERE invoice_total - payment_total - credit_total > 0
UNION
SELECT 'Paid' AS source, invoice_number, invoice_date, invoice_total
FROM invoices
WHERE invoice_total - payment_total - credit_total <= 0
ORDER BY invoice_total DESC;
```

### Simulate a FULL OUTER JOIN with UNION

MySQL does not have a `FULL OUTER JOIN` keyword. Simulate one by combining a left outer join and a right outer join with `UNION`:

```sql
SELECT department_name AS dept_name, d.department_number AS d_dept_no,
   e.department_number AS e_dept_no, last_name
FROM departments d
   LEFT JOIN employees e
   ON d.department_number = e.department_number
UNION
SELECT department_name AS dept_name, d.department_number AS d_dept_no,
   e.department_number AS e_dept_no, last_name
FROM departments d
   RIGHT JOIN employees e
   ON d.department_number = e.department_number
ORDER BY dept_name;
```

Rows with no match on either side display `NULL` in the unmatched columns.
