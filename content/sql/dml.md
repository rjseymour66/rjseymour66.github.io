---
title: "DML"
linkTitle: "DML"
weight: 60
description: >
  INSERT, UPDATE, and DELETE statements for modifying table data.
---

*DML* (Data Manipulation Language) statements add, modify, and remove rows in a table. In application development, these statements power the create, update, and delete operations behind any user-facing feature. Understanding how to combine them with subqueries lets you express complex data changes in a single statement rather than coordinating multiple round trips from application code.

## INSERT

The `INSERT` statement adds one or more rows to a table. Name the target table in the `INSERT INTO` clause, optionally list the columns to populate, then provide values in the `VALUES` clause.

The following adds a single invoice row:

```sql
INSERT INTO invoices
   (vendor_id, invoice_number, invoice_total, terms_id, invoice_date, invoice_due_date)
VALUES
   (97, '456789', 8344.50, 1, '2018-08-01', '2018-08-31');
```

Notes on the column list and values:

- If you omit the column list, you must provide values for every column in table-definition order.
- Specify `NULL` to insert a null value explicitly.
- Specify `DEFAULT` to insert the column's default value, or to trigger an auto-increment column.
- You can omit columns that have default values, accept nulls, or are auto-incremented.
- To insert multiple rows, specify multiple `VALUES` groups separated by commas.

### Subquery in INSERT

Replace the `VALUES` clause with a `SELECT` statement to insert rows copied from another table. This is useful for archiving or duplicating data. When the column list is present, the order must match the `SELECT` clause of the subquery:

```sql
INSERT INTO invoice_archive
   (invoice_id, vendor_id, invoice_number, invoice_total, credit_total,
    payment_total, terms_id, invoice_date, invoice_due_date)
SELECT
    invoice_id, vendor_id, invoice_number, invoice_total, credit_total,
    payment_total, terms_id, invoice_date, invoice_due_date
FROM invoices
WHERE invoice_total - payment_total - credit_total = 0;
```

### Copy a table

Create a copy of a table for testing with `CREATE TABLE ... AS SELECT`. MySQL copies column definitions and data, but not primary keys, foreign keys, or indexes:

```sql
CREATE TABLE invoices_copy AS
SELECT *
FROM invoices;
```

## UPDATE

The `UPDATE` statement modifies the data in one or more rows of a table. Name the table in the `UPDATE` clause, specify column assignments in the `SET` clause, and restrict which rows are affected with a `WHERE` clause.

The following updates two columns for a specific invoice:

```sql
UPDATE invoices
SET payment_date = '2018-09-21',
   payment_total = 19351.18
WHERE invoice_number = '97/522';
```

Best practices for `UPDATE`:

- The `WHERE` clause should reference a primary key or foreign key to avoid unintended updates.
- Before running an `UPDATE`, test the `WHERE` clause by running it as a `SELECT` first. If the `SELECT` returns the correct rows, copy the `WHERE` clause into the `UPDATE`.

### Subquery in UPDATE

Apply a subquery in the `WHERE` clause to identify rows based on data in another table. The following sets a payment term for all invoices belonging to a specific vendor, identified by name:

```sql
UPDATE invoices
SET terms_id = 1
WHERE vendor_id =
   (SELECT vendor_id
    FROM vendors
    WHERE vendor_name = 'Pacific Bell');
```

The subquery returns the vendor ID. The `UPDATE` then applies only to invoices matching that ID.

## DELETE

The `DELETE` statement removes one or more rows from a table. A foreign key constraint may prevent deletion if child rows in another table reference the row you want to remove. In that case, delete the child rows first.

The following removes invoice line items for invoice 12:

```sql
DELETE FROM invoice_line_items
WHERE invoice_id = 12;
```

The `WHERE` clause is optional but should almost always be included. Omitting it deletes every row in the table.

Best practices for `DELETE`:

- Before running a `DELETE`, test the `WHERE` clause as a `SELECT` first. If the `SELECT` returns the correct rows, copy the `WHERE` clause into the `DELETE`.

### Subquery in DELETE

To delete rows that depend on data in another table, apply a subquery in the `WHERE` clause. The following deletes all line items for a specific vendor by first selecting the relevant invoice IDs:

```sql
DELETE FROM invoice_line_items
WHERE invoice_id IN
   (SELECT invoice_id
   FROM invoices
   WHERE vendor_id = 115);
```

When deleting related rows across multiple tables, always start with the child tables (those holding foreign keys) before deleting from the parent tables.
