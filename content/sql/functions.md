---
title: "Functions"
linkTitle: "Functions"
weight: 110
description: >
  String, numeric, date/time, conditional, and regular expression functions.
---

MySQL provides a large library of built-in functions for transforming and analyzing data within queries. These functions let you format output, perform arithmetic, manipulate strings, work with dates, and implement conditional logic without requiring application-level post-processing.

## String functions

### Parsing with SUBSTRING_INDEX

`SUBSTRING_INDEX(string, delimiter, count)` returns the part of a string before (`count > 0`) or after (`count < 0`) the nth occurrence of a delimiter. This is commonly applied to split delimited values stored in a single column:

```sql
SELECT emp_name,
   SUBSTRING_INDEX(emp_name, ' ', 1) AS first_name,
   SUBSTRING_INDEX(emp_name, ' ', -1) AS last_name
FROM string_sample;
```

The positive `1` returns everything before the first space. The negative `-1` returns everything after the last space.

### Finding character positions with LOCATE

`LOCATE(substring, string[, start])` returns the position of the first occurrence of a substring. Pass a start position to search from a specific character:

```sql
SELECT emp_name,
   LOCATE(' ', emp_name) AS first_space,
   LOCATE(' ', emp_name, LOCATE(' ', emp_name) + 1) AS second_space
FROM string_sample;
```

The second call nests `LOCATE` inside itself to find the second space by starting the search after the first space.

### Extracting substrings with SUBSTRING

`SUBSTRING(string, start[, length])` extracts part of a string starting at position `start`. Combine it with `LOCATE` to extract name components at dynamic positions:

```sql
SELECT emp_name,
   SUBSTRING(emp_name, 1, LOCATE(' ', emp_name) - 1) AS first_name,
   SUBSTRING(emp_name, LOCATE(' ', emp_name) + 1) AS last_name
FROM string_sample;
```

The first call extracts from the beginning to the character before the first space. The second call extracts from the character after the first space to the end.

### String joining with CONCAT

`CONCAT(string1[, string2]...)` joins two or more strings into one. Apply it to combine separate columns into a formatted output:

```sql
SELECT vendor_name,
   CONCAT(vendor_city, ', ', vendor_state, ' ', vendor_zip_code) AS address
FROM vendors;
```

### LEFT

`LEFT(string, n)` extracts the first `n` characters from the left of a string. Apply it to generate initials or truncated codes:

```sql
SELECT vendor_contact_first_name, vendor_contact_last_name,
   CONCAT(LEFT(vendor_contact_first_name, 1),
      LEFT(vendor_contact_last_name, 1)) AS initials
FROM vendors;
```

## Numeric functions

The following table summarizes common numeric functions with example results:

| Function | Result |
|---|---|
| `ROUND(12.49, 0)` | 12 |
| `ROUND(12.50, 0)` | 13 |
| `ROUND(12.49, 1)` | 12.5 |
| `TRUNCATE(12.51, 0)` | 12 |
| `TRUNCATE(12.49, 1)` | 12.4 |
| `CEILING(12.5)` | 13 |
| `CEILING(-12.5)` | -12 |
| `FLOOR(-12.5)` | -13 |
| `FLOOR(12.5)` | 12 |
| `ABS(-1.25)` | 1.25 |
| `ABS(1.25)` | 1.25 |
| `SIGN(-1.25)` | -1 |
| `SIGN(1.25)` | 1 |
| `SQRT(125.43)` | 11.1995... |
| `POWER(9, 2)` | 81 |
| `RAND()` | 0.2444... (random) |

### Searching floating-point columns

Because `FLOAT` and `DOUBLE` values are approximate, exact equality comparisons may not return the expected rows. Instead, search within a range:

```sql
SELECT *
FROM float_sample
WHERE float_value BETWEEN 0.99 AND 1.01;
```

Alternatively, filter on the rounded value:

```sql
SELECT *
FROM float_sample
WHERE ROUND(float_value, 2) = 1.00;
```

## Date and time functions

### Current date and time

The following functions return the current date or time based on the system clock:

| Function | Description |
|---|---|
| `NOW()`, `SYSDATE()`, `CURRENT_TIMESTAMP()` | Returns the current local date and time |
| `CURDATE()`, `CURRENT_DATE()` | Returns the current local date |
| `CURTIME()`, `CURRENT_TIME()` | Returns the current local time |
| `UTC_DATE()` | Returns the current date in UTC |
| `UTC_TIME()` | Returns the current time in UTC |

### Parsing date and time components

The following functions extract individual components from a date or time value:

| Function | Result |
|---|---|
| `DAYOFMONTH('2018-12-03')` | 3 |
| `MONTH('2018-12-03')` | 12 |
| `YEAR('2018-12-03')` | 2018 |
| `HOUR('11:35:00')` | 11 |
| `MINUTE('11:35:00')` | 35 |
| `SECOND('11:35:00')` | 0 |
| `DAYOFWEEK('2018-12-03')` | 2 |
| `QUARTER('2018-12-03')` | 4 |
| `DAYOFYEAR('2018-12-03')` | 337 |
| `WEEK('2018-12-03')` | 48 |
| `LAST_DAY('2018-12-03')` | 31 |
| `DAYNAME('2018-12-03')` | Monday |
| `MONTHNAME('2018-12-03')` | December |

### EXTRACT function

`EXTRACT(unit FROM datetime)` extracts a specified unit from a date or time value and returns an integer. The following examples show available units:

| Function | Result |
|---|---|
| `EXTRACT(SECOND FROM '2018-12-03 11:35:00')` | 0 |
| `EXTRACT(MINUTE FROM '2018-12-03 11:35:00')` | 35 |
| `EXTRACT(HOUR FROM '2018-12-03 11:35:00')` | 11 |
| `EXTRACT(DAY FROM '2018-12-03 11:35:00')` | 3 |
| `EXTRACT(MONTH FROM '2018-12-03 11:35:00')` | 12 |
| `EXTRACT(YEAR FROM '2018-12-03 11:35:00')` | 2018 |
| `EXTRACT(HOUR_MINUTE FROM '2018-12-03 11:35:00')` | 1135 |
| `EXTRACT(YEAR_MONTH FROM '2018-12-03 11:35:00')` | 201812 |

### DATE_FORMAT

`DATE_FORMAT(date, format)` formats a date value using a format string. Prefix each format code with `%`. The following format codes are available:

| Code | Result |
|---|---|
| `%m` | Month, numeric (01...12) |
| `%c` | Month, numeric (1...12) |
| `%M` | Month name (January...December) |
| `%b` | Abbreviated month name (Jan...Dec) |
| `%d` | Day of month, numeric (00...31) |
| `%e` | Day of month, numeric (0...31) |
| `%D` | Day of month with suffix (1st, 2nd, 3rd) |
| `%y` | Year, 2 digits |
| `%Y` | Year, 4 digits |
| `%W` | Weekday name (Sunday...Saturday) |
| `%a` | Abbreviated weekday name (Sun...Sat) |
| `%H` | Hour (00...23) |
| `%h` | Hour (01...12) |
| `%i` | Minutes (00...59) |
| `%r` | Time, 12-hour (hh:mm:ss AM or PM) |
| `%T` | Time, 24-hour (hh:mm:ss) |
| `%S` | Seconds (00...59) |
| `%p` | AM or PM |

The following examples demonstrate `DATE_FORMAT` output:

| Function call | Result |
|---|---|
| `DATE_FORMAT('2018-12-03', '%m/%d/%y')` | 12/03/18 |
| `DATE_FORMAT('2018-12-03', '%W, %M, %D, %Y')` | Monday, December 3rd, 2018 |
| `DATE_FORMAT('2018-12-03', '%e-%b-%y')` | 3-Dec-18 |
| `DATE_FORMAT('2018-12-03 16:45', '%r')` | 04:45:00 PM |
| `DATE_FORMAT('16:45', '%l:%i %p')` | 4:45 PM |

### Date and time arithmetic

`DATE_ADD` and `DATE_SUB` add or subtract an interval from a date. `DATEDIFF` returns the number of days between two dates:

| Function call | Result |
|---|---|
| `DATE_ADD('2018-12-31', INTERVAL 1 DAY)` | 2019-01-01 |
| `DATE_ADD('2018-12-31', INTERVAL 3 MONTH)` | 2019-03-31 |
| `DATE_ADD('2018-12-31 23:59:59', INTERVAL 1 SECOND)` | 2019-01-01 00:00:00 |
| `DATE_ADD('2019-01-01', INTERVAL -1 DAY)` | 2018-12-31 |
| `DATE_SUB('2018-12-31', INTERVAL 1 DAY)` | 2018-12-30 |
| `DATEDIFF('2018-12-30', '2018-12-03')` | 27 |

### Searching for dates and times

To retrieve rows within a date range:

```sql
SELECT *
FROM date_sample
WHERE start_date >= '2018-02-28' AND start_date < '2018-03-01';
```

To retrieve rows for a specific month, day, and year using extraction functions:

```sql
SELECT *
FROM date_sample
WHERE MONTH(start_date) = 2
   AND DAYOFMONTH(start_date) = 28
   AND YEAR(start_date) = 2018;
```

To retrieve rows matching a formatted date string:

```sql
SELECT *
FROM date_sample
WHERE DATE_FORMAT(start_date, '%m-%d-%Y') = '02-28-2018';
```

To retrieve rows matching an exact time:

```sql
SELECT * FROM date_sample
WHERE DATE_FORMAT(start_date, '%T') = '10:00:00';
```

To retrieve rows in a range of times:

```sql
SELECT * FROM date_sample
WHERE EXTRACT(HOUR_MINUTE FROM start_date) BETWEEN 900 AND 1200;
```

## Conditional functions

### CASE

`CASE` returns a value based on a set of conditions. When MySQL finds a `WHEN` expression that matches the test expression, it returns the corresponding `THEN` value. This is the SQL equivalent of an if-else chain and is commonly applied to categorize or label data in the result set.

The following classifies invoices by how overdue they are:

```sql
SELECT invoice_number, invoice_total, invoice_date, invoice_due_date,
   CASE
      WHEN DATEDIFF(NOW(), invoice_due_date) > 30
         THEN 'Over 30 days past due'
      WHEN DATEDIFF(NOW(), invoice_due_date) > 0
         THEN '1 to 30 days past due'
      ELSE 'Current'
   END AS invoice_status
FROM invoices
WHERE invoice_total - payment_total - credit_total > 0;
```

### IF

`IF(condition, true_value, false_value)` returns one value if the condition is true and another if it is false:

```sql
SELECT vendor_name,
   IF(vendor_city = 'Fresno', 'Yes', 'No') AS is_city_fresno
FROM vendors;
```

### IFNULL

`IFNULL(expression, replacement)` returns the expression if it is not null, and the replacement value if it is. Apply this to substitute a displayable value for null in reports:

```sql
SELECT payment_date,
   IFNULL(payment_date, 'No Payment') AS new_date
FROM invoices;
```

### COALESCE

`COALESCE(expr1, expr2, ...)` returns the first non-null expression in the list. It is more flexible than `IFNULL` when multiple fallback values are needed:

```sql
SELECT payment_date,
   COALESCE(payment_date, 'No Payment') AS new_date
FROM invoices;
```

## Regular expression functions

MySQL provides dedicated functions for regular expression matching, searching, and replacement. These are more powerful than the `REGEXP` operator used in `WHERE` clauses and can appear anywhere in the `SELECT` list.

| Function | Example | Result | Description |
|---|---|---|---|
| `REGEXP_LIKE(expr, pattern)` | `REGEXP_LIKE('abc123', '123')` | 1 | Returns 1 if the expression matches the pattern, 0 otherwise |
| `REGEXP_INSTR(expr, pattern[, start])` | `REGEXP_INSTR('abc123', '123')` | 4 | Returns the index of the first matching substring. Returns 0 if not found. |
| `REGEXP_SUBSTR(expr, pattern[, start])` | `REGEXP_SUBSTR('abc123', '[A-Z][1-9]*$')` | c123 | Returns the first matching substring, or null if not found |
| `REGEXP_REPLACE(expr, pattern, replace[, start])` | `REGEXP_REPLACE('abc123', '1\|2', '3')` | abc333 | Replaces all occurrences of the pattern with the replacement string |

The following finds all vendor cities that contain a space:

```sql
SELECT DISTINCT vendor_city, REGEXP_INSTR(vendor_city, ' ') AS space_index
FROM vendors
WHERE REGEXP_INSTR(vendor_city, ' ') > 0
ORDER BY vendor_city;
```

The following returns cities starting with `SAN` or `LOS`:

```sql
SELECT vendor_city, REGEXP_SUBSTR(vendor_city, '^SAN|LOS') AS city_match
FROM vendors
WHERE REGEXP_SUBSTR(vendor_city, '^SAN|LOS') IS NOT NULL;
```

The following replaces the word `STREET` with `St` in address data:

```sql
SELECT vendor_name, vendor_address1,
   REGEXP_REPLACE(vendor_address1, 'STREET', 'St') AS new_address1
FROM vendors
WHERE REGEXP_LIKE(vendor_address1, 'STREET');
```
