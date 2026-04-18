---
title: "Querying"
linkTitle: "Querying"
weight: 40
description: >
  SELECT, filtering, sorting, and limiting query results.
---

The `SELECT` statement retrieves data from one or more tables. Most queries you write day-to-day combine `SELECT` with `WHERE` for filtering and `ORDER BY` for sorting. Understanding the clauses available lets you express most data retrieval tasks without resorting to application-level filtering.

## SELECT syntax

The full syntax for a `SELECT` statement is as follows:

```sql
SELECT select_list
[FROM table_source]
[WHERE search_condition]
[ORDER BY order_by_list]
[LIMIT row_limit]
```

### Column specifications

The `SELECT` list names the columns to retrieve. You can specify:

- `*`: all columns
- A column name: `column_name`
- An arithmetic expression: `column_1 - column_2`
- A function result: `CONCAT(first_name, ' ', last_name)`
- A column alias: `column_name AS alias`

When an alias contains a space, enclose it in quotes. For example, `'Total Due'`.

### DISTINCT

Apply `DISTINCT` to eliminate duplicate rows from a result set. This is useful when querying a column that contains repeated values.

```sql
SELECT DISTINCT vendor_state
FROM vendors;
```

### Testing expressions and functions

Run `SELECT` without a `FROM` clause to test expressions and functions interactively:

```sql
SELECT CURRENT_DATE,
   DATE_FORMAT(CURRENT_DATE, '%m/%d/%y') AS 'MM/DD/YY',
   DATE_FORMAT(CURRENT_DATE, '%e-%b-%Y') AS 'DD-Mon-YYYY';
```

```sql
SELECT 12345.6789 AS value,
   ROUND(12345.6789) AS nearest_dollar,
   ROUND(12345.6789, 1) AS nearest_dime;
```

## Arithmetic operators

The following arithmetic operators are available in expressions:

| Operator | Operation |
|---|---|
| `*` | Multiplication |
| `/` | Division |
| `DIV` | Integer division |
| `%` | Modulo |
| `+` | Addition |
| `-` | Subtraction |

## WHERE clause

The `WHERE` clause filters rows by a boolean expression. Only rows where the condition evaluates to true are returned. If you omit `WHERE`, the query returns all rows.

`SELECT` filters columns. `WHERE` filters rows.

### Comparison operators

The following query returns all invoices with a positive balance:

```sql
SELECT invoice_number, invoice_total
FROM invoices
WHERE invoice_total - payment_total - credit_total > 0;
```

### IN operator

The `IN` operator compares a value against a list of expressions. Include `NOT` to invert the match:

```sql
WHERE terms_id IN (1, 2, 3);
```

```sql
WHERE vendor_state NOT IN ('CA', 'NV', 'OR');
```

### BETWEEN operator

The `BETWEEN` operator tests whether a value falls within a range (inclusive). Include `NOT` to exclude the range:

```sql
WHERE invoice_date BETWEEN '2018-06-01' AND '2018-06-30';
```

```sql
WHERE vendor_zip_code NOT BETWEEN 93600 AND 93799;
```

### LIKE operator

The `LIKE` operator matches string patterns using wildcards. This is useful for searching partial strings when you do not know the exact value. The following wildcards are available:

| Character | Meaning | Example | Returns |
|---|---|---|---|
| `%` | Any string of zero or more characters | `WHERE vendor_city LIKE 'SAN%'` | San Diego, Santa Ana |
| `_` | Any single character | `WHERE vendor_name LIKE 'COMPU_ER'` | Compuserve, Computerworld |

### REGEXP operator

The `REGEXP` operator matches patterns using regular expressions. This provides more precise control than `LIKE` for complex string matching. The following special characters and constructs are available:

| Character | Meaning | Example | Returns |
|---|---|---|---|
| `^` | Matches at the beginning | `WHERE vendor_city REGEXP '^SA'` | Sacramento, Santa Ana |
| `$` | Matches at the end | `WHERE vendor_city REGEXP 'NA$'` | Gardena, Pasadena |
| `.` | Any single character | | |
| `[charlist]` | Any single character in the list | `WHERE vendor_city REGEXP 'N[CV]'` | NC, NV |
| `[char1-char2]` | Any single character in the range | `WHERE vendor_city REGEXP 'N[A-J]'` | NA through NJ |
| `\|` | Either of two patterns | `WHERE vendor_city REGEXP 'RS\|SN'` | Traverse City, Fresno |

For example, the following returns any city that ends with any letter, then a vowel, then the letter `n`:

```sql
WHERE vendor_city REGEXP '[A-Z][AEIOU]N$'
```

### IS NULL

To test for null values, apply `IS NULL` or `IS NOT NULL`:

```sql
SELECT * FROM invoices
WHERE invoice_total IS NOT NULL;
```

### Logical operators

Combine two or more conditions with `AND` and `OR`. Apply `NOT` to negate a condition. The order of precedence is: `NOT`, then `AND`, then `OR`.

```sql
WHERE vendor_state = 'NJ' AND vendor_city = 'Springfield';
```

```sql
WHERE vendor_state = 'NJ' OR vendor_city = 'Springfield';
```

```sql
WHERE NOT vendor_state = 'CA';
```

## ORDER BY clause

The `ORDER BY` clause sorts rows in the result set. The default sort order is ascending (`ASC`). Specify `DESC` to sort in descending order. You can sort by column names, expressions, aliases, or column positions. To sort by more than one column, list the columns in the order you want them applied. This is called a *nested sort*.

```sql
SELECT vendor_name,
   CONCAT(vendor_city, ', ', vendor_state, ' ', vendor_zip_code) AS address
FROM vendors
ORDER BY vendor_name DESC;
```

```sql
SELECT vendor_name,
   CONCAT(vendor_city, ', ', vendor_state, ' ', vendor_zip_code) AS address
FROM vendors
ORDER BY CONCAT(vendor_contact_last_name, vendor_contact_first_name);
```

## LIMIT clause

The `LIMIT` clause restricts the number of rows returned. Pair it with `ORDER BY` to retrieve top-N results. You can also specify an offset to skip rows. The offset is zero-indexed.

The syntax is:

```sql
LIMIT [offset,] row_count
```

For example, the following returns 3 rows starting from the row at index 2 (the third row):

```sql
SELECT invoice_id, vendor_id, invoice_total
FROM invoices
ORDER BY invoice_id
LIMIT 2, 3;
```
