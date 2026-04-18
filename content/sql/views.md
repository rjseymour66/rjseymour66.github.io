---
title: "Views"
linkTitle: "Views"
weight: 130
description: >
  Creating, querying, updating, and managing views.
---

A *view* is a `SELECT` statement stored in the database as a named object. Think of it as a virtual table that consists only of the rows and columns its `CREATE VIEW` statement defines. Unlike query scripts saved to files, views are stored as part of the database itself, so any application or SQL client with database access can query them by name.

Views are commonly applied to simplify complex joins, restrict which columns a user can see, and present a stable interface to application code even as the underlying schema evolves.

## Creating a view

To create a basic view that exposes a subset of vendor columns:

```sql
CREATE VIEW vendors_min AS
   SELECT vendor_name, vendor_state, vendor_phone
   FROM vendors;
```

To create a view filtered by region:

```sql
CREATE VIEW vendors_sw AS
SELECT *
FROM vendors
WHERE vendor_state IN ('CA', 'AZ', 'NV', 'NM');
```

## Querying a view

Query a view the same way you query a table. You can apply `WHERE`, `ORDER BY`, and other clauses:

```sql
SELECT * FROM vendors_min
WHERE vendor_state = 'CA'
ORDER BY vendor_name;
```

## Updating through a view

To be *updatable*, a view must meet all of the following requirements:

- The `SELECT` list cannot include the `DISTINCT` keyword or an aggregate function
- The `SELECT` statement cannot include a `GROUP BY` or `HAVING` clause
- The result cannot be formed by a `UNION` operation

If a view is updatable, you can run `INSERT`, `UPDATE`, and `DELETE` against it. Changes pass through to the underlying table. To `INSERT` rows through a view, the `INSERT` statement must supply values for all columns that the view does not provide defaults for.

The following updates a phone number through the `vendors_min` view:

```sql
UPDATE vendors_min
SET vendor_phone = '(800) 555-3941'
WHERE vendor_name = 'Register of Copyrights';
```

## Replacing a view

To overwrite an existing view without dropping it first, include `OR REPLACE`. This is safer than a separate `DROP` and `CREATE` because it is atomic:

```sql
CREATE OR REPLACE VIEW vendor_invoices AS
   SELECT vendor_name, invoice_number, invoice_date, invoice_total
   FROM vendors
      JOIN invoices ON vendors.vendor_id = invoices.vendor_id;
```

## WITH CHECK OPTION

The `WITH CHECK OPTION` clause prevents modifications through a view that would cause the modified row to no longer satisfy the view's `WHERE` condition. Without it, you could update a row so that it disappears from the view, which is often unintentional.

The following view shows unpaid invoices. The `WITH CHECK OPTION` prevents any update that would change the balance to zero or below, which would remove the row from the view:

```sql
CREATE OR REPLACE VIEW vendor_payment AS
   SELECT vendor_name, invoice_number, invoice_date, payment_date,
      invoice_total, credit_total, payment_total
   FROM vendors JOIN invoices ON vendors.vendor_id = invoices.vendor_id
   WHERE invoice_total - payment_total - credit_total >= 0
WITH CHECK OPTION;
```

## Dropping a view

To remove a view from the database:

```sql
DROP VIEW vendors_min;
```

Dropping a view does not affect the underlying tables or data.
