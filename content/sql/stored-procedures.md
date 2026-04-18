---
title: "Stored procedures"
linkTitle: "Stored procedures"
weight: 140
description: >
  Writing stored procedures with variables, flow control, cursors, and error handling.
---

*Stored programs* are database objects that contain procedural code, allowing you to run logic inside the database rather than in application code. They reduce round trips between the application and the database and can enforce business rules at the data layer.

MySQL supports four types of stored programs:

| Type | Description |
|---|---|
| Stored procedure | Called explicitly from an application or SQL client with `CALL` |
| Stored function | Called from a SQL statement like a built-in function (for example, `CONCAT`) |
| Trigger | Executed automatically in response to an `INSERT`, `UPDATE`, or `DELETE` on a specified table |
| Event | Executed at a scheduled time |

This page focuses on stored procedures.

## Creating a stored procedure

Because procedure bodies contain semicolons, you must change the statement delimiter before defining a procedure. The `DELIMITER` command changes the delimiter from `;` to a custom string (typically `//`). Restore the original delimiter after the procedure definition.

The following procedure calculates the outstanding balance for a vendor and displays a message:

```sql
USE ap;

DROP PROCEDURE IF EXISTS test;

DELIMITER //

CREATE PROCEDURE test()
BEGIN
   DECLARE sum_balance_due_var DECIMAL(9, 2);

   SELECT SUM(invoice_total - payment_total - credit_total)
   INTO sum_balance_due_var
   FROM invoices
   WHERE vendor_id = 95;

   IF sum_balance_due_var > 0 THEN
      SELECT CONCAT('Balance due: $', sum_balance_due_var) AS message;
   ELSE
      SELECT 'Balance paid in full' AS message;
   END IF;
END//

DELIMITER ;

CALL test();
```

## Flow control keywords

The following flow control keywords are available in stored programs:

| Keyword | Description |
|---|---|
| `IF...ELSEIF...ELSE` | Executes a block based on a condition |
| `CASE...WHEN...ELSE` | Executes a block based on matching a value |
| `WHILE...DO...END WHILE` | Repeats a block while a condition is true |
| `REPEAT...UNTIL...END REPEAT` | Repeats a block until a condition becomes true |
| `DECLARE...CURSOR FOR` | Defines a result set that a loop can process row by row |
| `DECLARE...HANDLER` | Defines a handler that executes when the procedure encounters an error |

## Declaring and setting variables

Declare variables at the beginning of the `BEGIN...END` block with `DECLARE`. Set a variable to a literal or expression with `SET`. Assign a value from a query result with `SELECT ... INTO`.

The following procedure declares multiple variables, populates them from a query, and displays a summary:

```sql
USE ap;

DROP PROCEDURE IF EXISTS test;

DELIMITER //

CREATE PROCEDURE test()
BEGIN
  DECLARE max_invoice_total  DECIMAL(9,2);
  DECLARE min_invoice_total  DECIMAL(9,2);
  DECLARE percent_difference DECIMAL(9,4);
  DECLARE count_invoice_id   INT;
  DECLARE vendor_id_var      INT;

  SET vendor_id_var = 95;

  SELECT MAX(invoice_total), MIN(invoice_total), COUNT(invoice_id)
  INTO max_invoice_total, min_invoice_total, count_invoice_id
  FROM invoices WHERE vendor_id = vendor_id_var;

  SET percent_difference = (max_invoice_total - min_invoice_total) /
                         min_invoice_total * 100;

  SELECT CONCAT('$', max_invoice_total) AS 'Maximum invoice',
         CONCAT('$', min_invoice_total) AS 'Minimum invoice',
         CONCAT('%', ROUND(percent_difference, 2)) AS 'Percent difference',
         count_invoice_id AS 'Number of invoices';
END//

DELIMITER ;

CALL test();
```

## IF statements

The `IF...ELSEIF...ELSE` statement executes different blocks based on conditions. The following procedure checks whether the earliest outstanding invoice is overdue:

```sql
USE ap;

DROP PROCEDURE IF EXISTS test;

DELIMITER //

CREATE PROCEDURE test()
BEGIN
   DECLARE first_invoice_due_date DATE;

   SELECT MIN(invoice_due_date)
   INTO first_invoice_due_date
   FROM invoices
   WHERE invoice_total - payment_total - credit_total > 0;

   IF first_invoice_due_date < NOW() THEN
     SELECT 'Outstanding invoices are overdue!';
   ELSEIF first_invoice_due_date = SYSDATE() THEN
     SELECT 'Outstanding invoices are due today!';
   ELSE
     SELECT 'No invoices are overdue.';
   END IF;
END//

DELIMITER ;

CALL test();
```

## CASE statements

The `CASE` statement selects a branch by comparing a variable against a list of values. This is similar to a switch statement:

```sql
USE ap;

DROP PROCEDURE IF EXISTS test;

DELIMITER //

CREATE PROCEDURE test()
BEGIN
  DECLARE terms_id_var INT;

  SELECT terms_id INTO terms_id_var
  FROM invoices WHERE invoice_id = 4;

  CASE terms_id_var
    WHEN 1 THEN
      SELECT 'Net due 10 days' AS Terms;
    WHEN 2 THEN
      SELECT 'Net due 20 days' AS Terms;
    WHEN 3 THEN
      SELECT 'Net due 30 days' AS Terms;
    ELSE
      SELECT 'Net due more than 30 days' AS Terms;
  END CASE;
END//

DELIMITER ;
```

## Loops

### WHILE loop

The `WHILE` loop executes a block as long as a condition is true. The following also shows the equivalent `REPEAT` and named `LOOP` syntax as comments:

```sql
USE ap;

DROP PROCEDURE IF EXISTS test;

DELIMITER //

CREATE PROCEDURE test()
BEGIN
  DECLARE i INT DEFAULT 1;
  DECLARE s VARCHAR(400) DEFAULT '';

  WHILE i < 4 DO
    SET s = CONCAT(s, 'i=', i, ' | ');
    SET i = i + 1;
  END WHILE;

  -- REPEAT loop equivalent:
  -- REPEAT
  --   SET s = CONCAT(s, 'i=', i, ' | ');
  --   SET i = i + 1;
  -- UNTIL i = 4
  -- END REPEAT;

  SELECT s AS message;
END//

DELIMITER ;

CALL test();
```

## Cursors

Apply a cursor when you need to process a result set one row at a time. Declare the cursor, open it, fetch rows in a loop, then close it. A `CONTINUE HANDLER FOR NOT FOUND` detects when no more rows remain and exits the loop.

The following procedure applies a 10% credit to every invoice over $1000:

```sql
USE ap;

DROP PROCEDURE IF EXISTS test;

DELIMITER //

CREATE PROCEDURE test()
BEGIN
  DECLARE invoice_id_var    INT;
  DECLARE invoice_total_var DECIMAL(9,2);
  DECLARE row_not_found     TINYINT DEFAULT FALSE;
  DECLARE update_count      INT DEFAULT 0;

  -- Declare the cursor
  DECLARE invoices_cursor CURSOR FOR
    SELECT invoice_id, invoice_total FROM invoices
    WHERE invoice_total - payment_total - credit_total > 0;

  -- Set row_not_found to TRUE when no more rows exist
  DECLARE CONTINUE HANDLER FOR NOT FOUND
    SET row_not_found = TRUE;

  OPEN invoices_cursor;

  WHILE row_not_found = FALSE DO
    FETCH invoices_cursor INTO invoice_id_var, invoice_total_var;

    IF invoice_total_var > 1000 THEN
      UPDATE invoices
      SET credit_total = credit_total + (invoice_total * .1)
      WHERE invoice_id = invoice_id_var;

      SET update_count = update_count + 1;
    END IF;
  END WHILE;

  CLOSE invoices_cursor;

  SELECT CONCAT(update_count, ' row(s) updated.');
END//

DELIMITER ;

CALL test();
```

## Error handling

MySQL assigns each error a numeric code and a five-character SQLSTATE code. The most common error codes are:

| Error code | SQLSTATE | Description |
|---|---|---|
| 1329 | 02000 | Attempt to fetch from a row that does not exist |
| 1062 | 23000 | Attempt to store a duplicate value in a unique column |
| 1048 | 23000 | Attempt to insert a NULL into a column that does not accept nulls |
| 1216 | 23000 | Attempt to add or update a child row that violates a foreign key constraint |
| 1217 | 23000 | Attempt to delete or update a parent row that violates a foreign key constraint |

### CONTINUE handler

A `CONTINUE` handler lets the procedure resume execution after the error:

```sql
DECLARE CONTINUE HANDLER FOR 1062
  SET duplicate_entry_for_key = TRUE;
```

### EXIT handler

An `EXIT` handler stops execution of the current `BEGIN...END` block as soon as the error occurs:

```sql
DECLARE EXIT HANDLER FOR 1062
  SET duplicate_entry_for_key = TRUE;
```

### Named condition handler (SQLEXCEPTION)

To catch any SQL exception regardless of the specific error code, apply `SQLEXCEPTION`. This acts like a catch-all exception handler:

```sql
DECLARE EXIT HANDLER FOR SQLEXCEPTION
  SET sql_error = TRUE;
```

### Multiple condition handlers

When you declare multiple handlers, MySQL executes the most specific one first. Declare them from most specific to least specific:

```sql
USE ap;

DROP PROCEDURE IF EXISTS test;

DELIMITER //

CREATE PROCEDURE test()
BEGIN
  DECLARE duplicate_entry_for_key INT DEFAULT FALSE;
  DECLARE column_cannot_be_null   INT DEFAULT FALSE;
  DECLARE sql_exception           INT DEFAULT FALSE;

  BEGIN
    DECLARE EXIT HANDLER FOR 1062
      SET duplicate_entry_for_key = TRUE;
    DECLARE EXIT HANDLER FOR 1048
      SET column_cannot_be_null = TRUE;
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
      SET sql_exception = TRUE;

    INSERT INTO general_ledger_accounts VALUES (NULL, 'Test');

    SELECT '1 row was inserted.' AS message;
  END;

  IF duplicate_entry_for_key = TRUE THEN
    SELECT 'Row was not inserted - duplicate key encountered.' AS message;
  ELSEIF column_cannot_be_null = TRUE THEN
    SELECT 'Row was not inserted - column cannot be null.' AS message;
  ELSEIF sql_exception = TRUE THEN
    SELECT 'Row was not inserted - SQL exception encountered.' AS message;
  END IF;
END//

DELIMITER ;

CALL test();
```
