---
title: "Schema"
linkTitle: "Schema"
weight: 120
description: >
  Creating and modifying databases, tables, indexes, and storage engines.
---

The *schema* is the structure of a database: its tables, columns, constraints, indexes, and relationships. You define and modify this structure with DDL (Data Definition Language) statements. Getting the schema right matters because changes to live tables with many rows can be slow and require careful planning.

## Databases

To create a new database, run the `CREATE DATABASE` statement:

```sql
CREATE DATABASE [IF NOT EXISTS] dbname;
```

To remove an existing database and all its tables:

```sql
DROP DATABASE [IF EXISTS] dbname;
```

To switch to a database and make it the current one, run the `USE` statement:

```sql
USE dbname;
```

## Tables

### Common column attributes

When defining columns, the following attributes control data validation and default behavior:

| Attribute | Description |
|---|---|
| `NOT NULL` | The column does not accept null values. If omitted, the column accepts nulls. |
| `UNIQUE` | Every value in the column must be unique. |
| `DEFAULT default_value` | Specifies a default value as a literal or expression. |
| `AUTO_INCREMENT` | Generates an integer value automatically when a new row is added. |

`NOT NULL` and `UNIQUE` are types of *constraints*.

### Creating a table

To create a table without constraints:

```sql
CREATE TABLE vendors
(
    vendor_id       INT,
    vendor_name     VARCHAR(50)
);
```

To create a table with column attributes applied:

```sql
CREATE TABLE vendors
(
    vendor_id       INT             NOT NULL        UNIQUE AUTO_INCREMENT,
    vendor_name     VARCHAR(50)     NOT NULL        UNIQUE
);
```

### Primary key constraints

When you designate a column as a primary key, MySQL automatically forces `NOT NULL` and `UNIQUE` on that column and creates an index for it.

Code a column-level primary key constraint as part of the column definition:

```sql
CREATE TABLE vendors
(
    vendor_id       INT             PRIMARY KEY     AUTO_INCREMENT,
    vendor_name     VARCHAR(50)     NOT NULL        UNIQUE
);
```

Code a table-level primary key constraint as a separate entry in the column list. This allows you to name the constraint and is required for composite primary keys:

```sql
CREATE TABLE vendors
(
    vendor_id           INT             AUTO_INCREMENT,
    vendor_name         VARCHAR(50)     NOT NULL,
    CONSTRAINT vendors_pk PRIMARY KEY (vendor_id),
    CONSTRAINT vendor_name_uq UNIQUE (vendor_name)
);
```

### Foreign key constraints

Also called a *reference constraint*, a foreign key constraint requires that values in one table match values in another. This enforces referential integrity.

To define a foreign key at the column level:

```sql
CREATE TABLE invoices
(
    invoice_id          INT             PRIMARY KEY,
    vendor_id           INT             REFERENCES vendors (vendor_id),
    invoice_number      VARCHAR(50)     NOT NULL        UNIQUE
);
```

To define a named foreign key at the table level (preferred, since the name aids in debugging and schema changes):

```sql
CREATE TABLE invoices
(
    invoice_id          INT             PRIMARY KEY,
    vendor_id           INT             NOT NULL,
    invoice_number      VARCHAR(50)     NOT NULL        UNIQUE,
    CONSTRAINT invoices_fk_vendors
       FOREIGN KEY (vendor_id) REFERENCES vendors (vendor_id)
);
```

## ALTER TABLE

Apply the `ALTER TABLE` statement to modify the structure of an existing table.

To add a column:

```sql
ALTER TABLE vendors
ADD last_transaction_date DATE;
```

To remove a column:

```sql
ALTER TABLE vendors
DROP COLUMN last_transaction_date;
```

To change the length of a column:

```sql
ALTER TABLE vendors
MODIFY vendor_name VARCHAR(100) NOT NULL;
```

To change the data type:

```sql
ALTER TABLE vendors
MODIFY vendor_name CHAR(100) NOT NULL;
```

To change the default value of a column:

```sql
ALTER TABLE vendors
MODIFY vendor_name VARCHAR(100) NOT NULL DEFAULT 'New Vendor';
```

To rename a column:

```sql
ALTER TABLE vendors
RENAME COLUMN vendor_name TO v_name;
```

To add a primary key constraint:

```sql
ALTER TABLE vendors
ADD PRIMARY KEY (vendor_id);
```

To add a named foreign key constraint:

```sql
ALTER TABLE invoices
ADD CONSTRAINT invoice_fk_vendors
   FOREIGN KEY (vendor_id) REFERENCES vendors (vendor_id);
```

To drop a primary key constraint:

```sql
ALTER TABLE vendors
DROP PRIMARY KEY;
```

To drop a foreign key constraint (you must know its name):

```sql
ALTER TABLE invoices
DROP FOREIGN KEY invoice_fk_vendors;
```

## Renaming, truncating, and dropping tables

Apply these statements with care. They cannot be reversed.

To rename a table:

```sql
RENAME TABLE vendors TO vendor;
```

To delete all rows from a table while keeping its structure:

```sql
TRUNCATE TABLE vendor;
```

To delete a table from the current database:

```sql
DROP TABLE vendor;
```

To qualify the table with its database name:

```sql
DROP TABLE ex.vendor;
```

## Indexes

An index provides efficient row access by allowing the database engine to go directly to a row rather than scanning the entire table. MySQL creates indexes automatically for primary keys, foreign keys, and unique keys. Create additional indexes on columns frequently applied in `WHERE` clauses or `JOIN` conditions.

Avoid creating indexes on columns that are updated frequently. Indexes slow down `INSERT`, `UPDATE`, and `DELETE` operations because the index must be updated with every row change.

The standard naming convention for an index is:

```
table_column_ix
```

For example: `invoices_invoice_date_ix`.

To create an index on a single column:

```sql
CREATE INDEX invoices_invoice_date_ix
   ON invoices (invoice_date);
```

To create an index on two columns:

```sql
CREATE INDEX invoices_vendor_id_invoice_number_ix
   ON invoices (invoice_id, invoice_number);
```

To create a unique index (prevents duplicate values in the indexed column):

```sql
CREATE UNIQUE INDEX vendors_vendor_phone_ix
   ON vendors (vendor_phone);
```

To create an index with descending sort order:

```sql
CREATE INDEX invoices_invoice_total_ix
   ON invoices (invoice_total DESC);
```

To remove an index:

```sql
DROP INDEX vendors_vendor_phone_ix ON vendors;
```

## Storage engines

A *storage engine* determines how MySQL stores data and which features are available. The most commonly applied engines are:

`InnoDB`
: The default engine since MySQL 5.5. Supports transactions, foreign keys, and row-level locking. Apply InnoDB for any application that requires data integrity.

`MyISAM`
: An older engine without transaction support. Faster for read-heavy workloads without concurrent writes.

To view all available storage engines on the server:

```sql
SHOW ENGINES;
```

To view the current default storage engine:

```sql
SHOW VARIABLES LIKE 'default_storage_engine';
```

To view the storage engine for all tables in a database:

```sql
SELECT table_name, engine
FROM information_schema.tables
WHERE table_schema = 'ap';
```

To specify an engine when creating a table:

```sql
CREATE TABLE product_descriptions
(
    product_id              INT             PRIMARY KEY,
    product_description     VARCHAR(200)
)
ENGINE = MyISAM;
```

To change the engine for an existing table:

```sql
ALTER TABLE product_descriptions ENGINE = InnoDB;
```

To change the default engine for the current session:

```sql
SET SESSION default_storage_engine = InnoDB;
```

## Full database creation script

A script is a file containing one or more SQL statements. When creating a full database, create tables without foreign keys first so that other tables can reference them. The following script creates a complete accounts payable schema:

```sql
-- Create the database
DROP DATABASE IF EXISTS ap;
CREATE DATABASE ap;

-- Select the database
USE ap;

-- Create tables without foreign keys first
CREATE TABLE general_ledger_accounts
(
  account_number        INT            PRIMARY KEY,
  account_description   VARCHAR(50)    UNIQUE
);

CREATE TABLE terms
(
  terms_id              INT            PRIMARY KEY,
  terms_description     VARCHAR(50)    NOT NULL,
  terms_due_days        INT            NOT NULL
);

CREATE TABLE vendors
(
  vendor_id                     INT            PRIMARY KEY   AUTO_INCREMENT,
  vendor_name                   VARCHAR(50)    NOT NULL      UNIQUE,
  vendor_address1               VARCHAR(50),
  vendor_address2               VARCHAR(50),
  vendor_city                   VARCHAR(50)    NOT NULL,
  vendor_state                  CHAR(2)        NOT NULL,
  vendor_zip_code               VARCHAR(20)    NOT NULL,
  vendor_phone                  VARCHAR(50),
  vendor_contact_last_name      VARCHAR(50),
  vendor_contact_first_name     VARCHAR(50),
  default_terms_id              INT            NOT NULL,
  default_account_number        INT            NOT NULL,
  CONSTRAINT vendors_fk_terms
    FOREIGN KEY (default_terms_id)
    REFERENCES terms (terms_id),
  CONSTRAINT vendors_fk_accounts
    FOREIGN KEY (default_account_number)
    REFERENCES general_ledger_accounts (account_number)
);

CREATE TABLE invoices
(
  invoice_id            INT            PRIMARY KEY   AUTO_INCREMENT,
  vendor_id             INT            NOT NULL,
  invoice_number        VARCHAR(50)    NOT NULL,
  invoice_date          DATE           NOT NULL,
  invoice_total         DECIMAL(9,2)   NOT NULL,
  payment_total         DECIMAL(9,2)   NOT NULL      DEFAULT 0,
  credit_total          DECIMAL(9,2)   NOT NULL      DEFAULT 0,
  terms_id              INT            NOT NULL,
  invoice_due_date      DATE           NOT NULL,
  payment_date          DATE,
  CONSTRAINT invoices_fk_vendors
    FOREIGN KEY (vendor_id)
    REFERENCES vendors (vendor_id),
  CONSTRAINT invoices_fk_terms
    FOREIGN KEY (terms_id)
    REFERENCES terms (terms_id)
);

CREATE TABLE invoice_line_items
(
  invoice_id              INT            NOT NULL,
  invoice_sequence        INT            NOT NULL,
  account_number          INT            NOT NULL,
  line_item_amount        DECIMAL(9,2)   NOT NULL,
  line_item_description   VARCHAR(100)   NOT NULL,
  CONSTRAINT line_items_pk
    PRIMARY KEY (invoice_id, invoice_sequence),
  CONSTRAINT line_items_fk_invoices
    FOREIGN KEY (invoice_id)
    REFERENCES invoices (invoice_id),
  CONSTRAINT line_items_fk_accounts
    FOREIGN KEY (account_number)
    REFERENCES general_ledger_accounts (account_number)
);

-- Create indexes
CREATE INDEX invoices_invoice_date_ix
  ON invoices (invoice_date DESC);
```
