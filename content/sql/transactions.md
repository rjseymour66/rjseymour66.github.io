---
title: "Transactions"
linkTitle: "Transactions"
weight: 150
description: >
  Transactions, savepoints, isolation levels, and row locking.
---

A *transaction* groups two or more SQL statements into a single logical unit of work. If any statement in the group fails, you can roll back all the changes as if none of them ran. Transactions are essential whenever a business operation requires multiple table updates that must succeed or fail together — for example, recording an invoice and its line items simultaneously.

Note: only storage engines that support transactions (such as InnoDB) can apply the statements on this page. MyISAM does not support transactions.

## Basic transaction

Start a transaction with `START TRANSACTION`. This disables autocommit for the statements that follow. Confirm all changes with `COMMIT`. Undo all changes with `ROLLBACK`.

The following procedure records an invoice and its line items as a single transaction. If any statement fails, the error handler sets `sql_error` to `TRUE` and the procedure rolls back all changes:

```sql
USE ap;

DROP PROCEDURE IF EXISTS test;

DELIMITER //

CREATE PROCEDURE test()
BEGIN
  DECLARE sql_error INT DEFAULT FALSE;

  DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
    SET sql_error = TRUE;

  START TRANSACTION;

  INSERT INTO invoices
  VALUES (115, 34, 'ZXA-080', '2014-06-30',
          14092.59, 0, 0, 3, '2014-09-30', NULL);

  INSERT INTO invoice_line_items
  VALUES (115, 1, 160, 4447.23, 'HW upgrade');

  INSERT INTO invoice_line_items
  VALUES (115, 2, 167, 9645.36, 'OS upgrade');

  IF sql_error = FALSE THEN
    COMMIT;
    SELECT 'The transaction was committed.';
  ELSE
    ROLLBACK;
    SELECT 'The transaction was rolled back.';
  END IF;
END//

DELIMITER ;

CALL test();
```

## Savepoints

A *savepoint* marks a position within a transaction. Roll back to a savepoint to undo only the statements that ran after it, rather than rolling back the entire transaction.

The following demonstrates rolling back to progressively earlier savepoints:

```sql
USE ap;

START TRANSACTION;

SAVEPOINT before_invoice;

INSERT INTO invoices
VALUES (115, 34, 'ZXA-080', '2015-01-18',
        14092.59, 0, 0, 3, '2015-04-18', NULL);

SAVEPOINT before_line_item1;

INSERT INTO invoice_line_items
VALUES (115, 1, 160, 4447.23, 'HW upgrade');

SAVEPOINT before_line_item2;

INSERT INTO invoice_line_items
VALUES (115, 2, 167, 9645.36, 'OS upgrade');

ROLLBACK TO SAVEPOINT before_line_item2;
ROLLBACK TO SAVEPOINT before_line_item1;
ROLLBACK TO SAVEPOINT before_invoice;

COMMIT;
```

Each `ROLLBACK TO SAVEPOINT` undoes everything after the named point without ending the transaction. A final `COMMIT` or `ROLLBACK` ends the transaction.

## Concurrency and locking

*Concurrency* occurs when two or more users access and modify the same data at the same time. Concurrent updates can produce inconsistent results when one transaction reads data that another transaction is in the process of changing. Transactions with `START TRANSACTION` and `COMMIT` lock the affected rows and prevent other transactions from modifying them until the work is committed.

### Isolation levels

The *transaction isolation level* controls the degree to which transactions are isolated from one another. More restrictive levels eliminate more concurrency problems but reduce throughput. The four levels, from least to most restrictive, are:

1. `READ UNCOMMITTED`
2. `READ COMMITTED`
3. `REPEATABLE READ` (the MySQL default)
4. `SERIALIZABLE`

To set the isolation level for a transaction:

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### Row locking

When the default isolation level is insufficient, lock specific rows explicitly. The following locking options are available:

`SELECT ... FOR SHARE`
: Locks the selected rows so other transactions can read them but cannot modify them until the current transaction commits. If the rows are already locked for update by another transaction, this statement waits for that transaction to commit.

`SELECT ... FOR UPDATE`
: Locks the selected rows and their associated indexes so other transactions cannot read or modify them. If the rows are locked by another transaction, this statement waits.

`SELECT ... FOR UPDATE NOWAIT`
: Same as `FOR UPDATE`, but returns an error immediately if the rows are already locked rather than waiting.

`SELECT ... FOR UPDATE SKIP LOCKED`
: Same as `FOR UPDATE`, but skips rows that are locked and returns only the rows that are available.

The following demonstrates all three `FOR UPDATE` variants across separate transactions:

```sql
-- Transaction B: waits for locks
START TRANSACTION;
SELECT * FROM sales_reps WHERE rep_id < 5 FOR UPDATE;
COMMIT;

-- Transaction C: fails immediately if rows are locked
START TRANSACTION;
SELECT * FROM sales_reps WHERE rep_id < 5 FOR UPDATE NOWAIT;
COMMIT;

-- Transaction D: skips locked rows and returns the rest
START TRANSACTION;
SELECT * FROM sales_reps WHERE rep_id < 5 FOR UPDATE SKIP LOCKED;
COMMIT;
```

### Deadlocks

A *deadlock* occurs when two transactions each hold a lock that the other transaction needs, so neither can proceed. InnoDB detects deadlocks automatically and rolls back one of the transactions to break the cycle. The rolled-back transaction receives an error and must be retried by the application.

To reduce deadlock frequency, always acquire locks in the same order across transactions and keep transactions short.
