---
title: "Data types"
linkTitle: "Data types"
weight: 100
description: >
  MySQL data types for strings, numbers, dates, enumerations, and large objects.
---

A *data type* specifies the kind of information a column is intended to store and determines what operations can be performed on it. Choosing the smallest appropriate data type reduces storage requirements and improves query performance, since smaller rows fit more efficiently in memory and disk cache.

## Data type categories

MySQL organizes data types into the following categories:

| Category | Description |
|---|---|
| Character | Strings of character data |
| Numeric | Integers and real numbers |
| Date and time | Dates, times, or both |
| Large Object (LOB) | Large strings of character or binary data |
| Spatial | Geographical values |
| JSON | JSON documents |

## Character types

The two most common character types are `CHAR` and `VARCHAR`:

| Type | Bytes | Description |
|---|---|---|
| `CHAR(M)` | M x 4 | Fixed-length string where M is the number of characters (0 to 255). With `utf8mb4`, MySQL reserves four bytes per character. |
| `VARCHAR(M)` | L + 1 | Variable-length string where M is the maximum number of characters (0 to 255). Stores only as many bytes as the string requires, plus one byte for the length. |

### CHAR

`CHAR` stores fixed-length strings. All values stored in a `CHAR(M)` column occupy the same number of bytes regardless of actual length. Trailing spaces pad shorter values. Apply `CHAR` for columns with a consistent length, such as a two-character state code (`CHAR(2)`).

### VARCHAR

`VARCHAR` stores variable-length strings. Storage equals the actual string length plus one byte to record the length. Apply `VARCHAR` for columns whose values vary in length from row to row.

### Storage comparison with utf8mb4

The `utf8mb4` character set uses up to four bytes per character. This affects `CHAR` more than `VARCHAR`:

| Data type | Original value | Value stored | Bytes used |
|---|---|---|---|
| `CHAR(2)` | 'CA' | 'CA' | 8 |
| `CHAR(10)` | 'CA' | 'CA        ' | 40 |
| `VARCHAR(10)` | 'CA' | 'CA' | 3 |
| `VARCHAR(20)` | 'California' | 'California' | 11 |
| `VARCHAR(20)` | 'New York' | 'New York' | 9 |

## Integer types

Integer types store positive or negative numbers without a decimal point. The following types are available, ordered by storage size:

| Type | Bytes |
|---|---|
| `BIGINT` | 8 |
| `INT` | 4 |
| `MEDIUMINT` | 3 |
| `SMALLINT` | 2 |
| `TINYINT` | 1 |

`INT` is the most common choice because it accommodates a wide range of values with only 4 bytes of storage.

Two optional attributes modify integer behavior:

`UNSIGNED`
: Prevents negative values from being stored, which doubles the positive range.

`ZEROFILL`
: Sets `UNSIGNED` automatically and pads the displayed value with leading zeros up to the maximum display width. For example, `INT` has a maximum display width of 10.

`BOOLEAN` and `BOOL` are synonyms for `TINYINT(1)`. The value `0` represents false and `1` represents true.

## Fixed-point and floating-point types

### DECIMAL

`DECIMAL(M, D)` stores exact numeric values. `M` specifies the total number of digits (precision, 1 to 65) and `D` specifies the digits to the right of the decimal point (scale, 0 to 30, cannot exceed M). Apply `DECIMAL` for monetary values and any calculation that requires exact precision.

### Floating-point types

| Type | Bytes | Description |
|---|---|---|
| `FLOAT` | 4 | Single-precision floating-point numbers |
| `DOUBLE` | 8 | Double-precision floating-point numbers |

Floating-point numbers are approximate. Do not compare them with exact equality operators. See [functions.md](../functions/) for how to search floating-point columns correctly.

## Date and time types

Apply `TIMESTAMP` to track when a row was inserted or last updated. The following date and time types are available:

| Type | Bytes | Description |
|---|---|---|
| `DATE` | 3 | Dates from January 1, 1000 to December 31, 9999. Default format: `'yyyy-mm-dd'`. |
| `TIME` | 3 | Times in the range -839:59:59 to 839:59:59. Default format: `'hh:mm:ss'`. |
| `DATETIME` | 8 | Combined date and time from January 1, 1000 to December 31, 9999. Default format: `'yyyy-mm-dd hh:mm:ss'`. |
| `TIMESTAMP` | 4 | Combined date and time from midnight January 1, 1970 to the year 2037. Default format: `'yyyy-mm-dd hh:mm:ss'`. |
| `YEAR[(4)]` | 1 | Years in four-digit format (1901 to 2155). One- and two-digit entries are accepted and converted automatically. |

## ENUM and SET types

Both types restrict column values to a predefined list of strings:

| Type | Bytes | Description |
|---|---|---|
| `ENUM` | 1-2 | Stores exactly one value from the list of acceptable values |
| `SET` | 1-8 | Stores zero or more values from the list of acceptable values (up to 64) |

Apply `ENUM` when a column can hold exactly one value from a fixed set, such as a status field. Apply `SET` when a column can hold multiple values simultaneously, such as a set of permissions.

## Large object types

Large object types store large amounts of character or binary data:

`BLOB` types
: Store binary large objects such as images, audio, and video.

`TEXT` types
: Store character large objects, sometimes called CLOBs. Apply these for long text content such as article bodies.

## Data conversion

MySQL performs *implicit conversions* automatically when comparing or combining values of different types. To convert explicitly, apply the `CAST` or `CONVERT` functions.

### CAST function

`CAST(expression AS type)` converts an expression to the specified type. The following cast types are available:

| Cast type | Description |
|---|---|
| `CHAR[(N)]` | A string of up to N characters |
| `DATE` | A DATE value |
| `DATETIME` | A DATETIME value |
| `TIME` | A TIME value |
| `SIGNED [INTEGER]` | A signed integer value |
| `UNSIGNED [INTEGER]` | An unsigned integer value |
| `DECIMAL[(M[,D])]` | A DECIMAL value with the specified precision and scale |

The following converts a date column to a string and a decimal column to an integer:

```sql
SELECT invoice_id, invoice_date, invoice_total,
   CAST(invoice_date AS CHAR(10)) AS char_date,
   CAST(invoice_total AS SIGNED) AS integer_total
FROM invoices;
```

To sort a string column that contains numeric values as if it were an integer column:

```sql
SELECT *
FROM string_example
ORDER BY CAST(emp_id AS SIGNED);
```

Alternatively, you can trigger an implicit conversion by adding zero:

```sql
SELECT *
FROM string_example
ORDER BY emp_id + 0;
```

### CONVERT function

`CONVERT` provides the same functionality as `CAST` with an alternative syntax:

```sql
SELECT invoice_id, invoice_date, invoice_total,
   CONVERT(invoice_date, CHAR(10)) AS char_date,
   CONVERT(invoice_total, SIGNED) AS integer_total
FROM invoices;
```

### FORMAT and CHAR functions

The following functions handle specialized conversion scenarios:

| Function | Description |
|---|---|
| `FORMAT(number, decimal)` | Converts a number to a string with comma-separated digit groups, rounded to the specified number of decimal places. If `decimal` is zero, the decimal point is omitted. |
| `CHAR(value1[, value2]...)` | Converts one or more integers (0 to 255) to a binary string. Typically applied to output ASCII control characters that cannot be typed directly. |
