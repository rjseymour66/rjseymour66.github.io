---
title: "Concepts"
linkTitle: "Concepts"
weight: 30
description: >
  Core relational database concepts and terminology.
---

A *relational database* organizes data into tables. Each table represents a single entity — for example, customers or invoices. Tables are made of rows and columns, sometimes called records and fields.

Understanding these terms makes it easier to read SQL documentation and reason about schema design decisions.

## Terminology

The following terms appear throughout SQL documentation and query output:

*Cell*
: The intersection of a row and a column.

*Column*
: Represents an attribute of an entity, such as the amount of an invoice. Each column has a data type that constrains what values it can hold.

*Row*
: Contains a set of values for a single instance of the entity.

*Constraint*
: A rule that restricts the type of data a column can store. For example, `NOT NULL` prevents empty values and `UNIQUE` prevents duplicate values.

*Data type*
: Determines what kind of information a column stores. Choosing the smallest appropriate data type reduces storage overhead and improves query performance. Common data types include `CHAR`, `VARCHAR`, `INT`, `DECIMAL`, `FLOAT`, and `DATE`.

*Primary key*
: Uniquely identifies each row in a table. Usually a single column, but can span multiple columns.

*Composite primary key*
: A primary key that uses two or more columns together.

*Non-primary key*
: Also called a *unique key*. Uniquely identifies each row but is not designated as the primary key.

*Foreign key*
: One or more columns in a table that reference the primary key of another table. Foreign keys express a one-to-many relationship between tables.

*Referential integrity*
: The guarantee that changes to data do not create invalid relationships between tables. Enforced through foreign key constraints.

*Index*
: Provides efficient access to rows based on the values in specific columns. MySQL creates indexes automatically for primary keys, foreign keys, and unique keys.

*Null*
: A value that is unknown, unavailable, or not applicable. Null is not the same as zero or an empty string.

*Default value*
: A value assigned to a column automatically when no other value is provided.

*Auto-increment column*
: A column whose value the database generates automatically when a new row is inserted.

*Entity-relationship (ER) diagram*
: A visual representation of how the tables in a database are defined and related.

*Result set*
: The data returned by a query. Also called a result table.

*Scalar function*
: A function that operates on a single value and returns a single value.

*Aggregate function*
: A function that operates on a series of values and returns a single summary value. Also called a *column function* because it typically operates on column values.

*Summary query*
: A query that contains one or more aggregate functions.

*Database object*
: Any defined object in a database used to store or reference data. Everything created with a `CREATE` statement is a database object, including tables, views, sequences, indexes, and synonyms.

## DML and DDL statements

SQL statements fall into two categories. *DML* (Data Manipulation Language) statements work with data. *DDL* (Data Definition Language) statements create and modify the structure of a database.

DML statements include:

- `SELECT`: retrieves data from one or more tables
- `INSERT`: adds new rows to a table
- `UPDATE`: modifies existing rows in a table
- `DELETE`: removes rows from a table

DDL statements include:

- `CREATE DATABASE`: creates a new database on the server
- `CREATE TABLE`: creates a new table in a database
- `CREATE INDEX`: creates a new index for a table
- `ALTER TABLE`: changes the definition of an existing table
- `ALTER INDEX`: changes the structure of an existing index
- `DROP DATABASE`: deletes an existing database and all its tables
- `DROP TABLE`: deletes an existing table
- `DROP INDEX`: deletes an existing index

## Building queries

Build queries one clause at a time. Start with a `SELECT` and `FROM` clause, verify the result, then add `WHERE`, `ORDER BY`, and other clauses incrementally. This approach makes it easier to isolate the source of unexpected results.
