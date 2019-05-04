# SQL Style Guide (EXPERIMENTAL)

<!--*
# Document freshness: { owner: '[AUTHOR_NAME]' reviewed: '2019-05-03' }
*-->

[TOC]

## General editing guidelines

These guidelines are intended to make the scripts more readable and maintainable, and allow easier transfer of ownership.

* Use 2 spaces to represent 'one indent'.
* Keep line length within 120 characters.
* Multiple query blocks in one script should be separated with two blank lines.
* Don't use blank lines within query blocks (except for UNION clause).
* Table names should be all lowercase with underscores as spaces. Same pattern applies to field names and aliases.
* Script file names should be lowercase using snake style where possible.
* Script files should have proper `.sql` extension.


## Correct use of comments

### Script header

Put a comment block at the top of the script containing the standard header.

```sql
/* 
  Description: 
    Script description.
    New lines should be indented like this. 
 
  Notes: 
    Specific details the user of the script to be aware of. 
 
  Links: 
    Design doc - https://...
    Implementation plan - https://... 
 
  POC: 
    Full Name1 (ldap1) 
    Full Name2 (ldap2) 
 
  PK Columns: 
    column_a 
    column_b
*/
```

Sections are separated with a single blank line. **Notes** and **Links** sections are optional and could be skipped if empty. When added, keep the order of the sections in accordance with the example above.

If multiple notes to be added, bullet list should be indented like this:

```sql
/* 
  ...
 
  Notes: 
    - Specific details the user of the script to be aware of which are large enough to fill
      more than one line.
    - Some more notes added here as well.
    
  ...

*/
```

Remove legacy code when updating scripts checked into git rather than commenting it out (this information is already available in the history of commits and can be reverted when needed). Don't include a history of changes in the comment header.

Please add a single blank line after the comment header before writing the code.

```sql
/* 
  Description: 
    Script description.
 
  POC: 
    Full Name1 (ldap1)  
 
  PK Columns: 
    column_a 
*/

CREATE TABLE ...
...
```

Though there are no physical primary keys in most of the engines, please fill the **PK Columns** sections with the list of columns that have to guarantee uniqueness of a row in the output dataset. Even when there's a `GROUP BY` clause including 50 fields, usually there is a shorter list of columns that generate unique combination. 


### Comments in code

* Use `-- ` when commenting (add one space symbol between '--' and the actual comment).
* Avoid commenting inline for any non-short comment.
* When commenting, indent comments accordingly to the line to which they are referring.
* Use `/* */` when commenting out blocks of script during development (don't leave `/* */` style comments in production code).
* Do not use lines of dashes around comment blocks (avoid `'------------------------------'` style comments).

For example:

```sql
-- Comment example with multiple lines which could not fit into 120 character limit should be added accordingly to the
-- referring line.
SELECT
  column_1,
  column_2,
  column_3,
  -- This comment is referenced to the column_4...
  -- Please add multiple lines like this.
  column_4
FROM
  -- And this comment describes something applicable to table_a 
  table_a
...
```

Use full length where possible. Instead of this:

```sql
-- Comment example with multiple
-- lines which could not fit into 120 character limit
-- should not be written
-- like this.
SELECT
  column_1,
  column_2
FROM
  table_a
...
```

Please do this:

```sql
-- Comment example with multiple lines which could not fit into 120 character limit should be added accordingly to the
-- referring line.
SELECT
  column_1,
  column_2
FROM
  table_a
...
```

## Indentation

The general indentation style is:

```sql
SELECT
  column_1,
  column_2,
  column_3,
  column_4
FROM
  table_a
WHERE
  ...
  AND ...
GROUP BY
  ..., ...
ORDER BY
  ..., ...
```

Indent clauses that are part of another clause. For example, `JOIN`s are part of the `FROM` clause, so they should be indented after the `FROM` line:

```sql
SELECT
  alias_a.column_a,
  alias_a.column_b,
  alias_b.column_c
FROM (
  SELECT
    column_a,
    column_b
  FROM
    table_a
) AS alias_a
LEFT JOIN (
  SELECT
    column_a,
    column_c
  FROM
    table_b
) AS alias_b
ON alias_a.column_a = alias_b.column_a
WHERE
  ...
GROUP BY
  ...
ORDER BY
  ...
```

For UNION [ALL|DISTINCT], do:

```sql
SELECT
  column_a,
  column_b
FROM (
  SELECT
    column_a,
    column_b
  FROM
    table_a

  UNION ALL

  SELECT
    column_a,
    column_b
  FROM
    table_b
)
```

This also includes `ON` clauses, so for multiple `JOIN` conditions, do this:

```sql
...
ON table_a.key = table_b.key
  AND table_a.day = table_b.day
```

Keep the order of the `ON` statements in join clauses consistent. For example:

```sql
...
  FROM
    table_1
  INNER JOIN
    table_2
  ON table_1.field_1 = table_2.field_1
...
```

Is easier to understand than:

```sql
...
FROM
  table_1
INNER JOIN
  table_3
ON table_3.field_2 = table_1.field_2
...
```

Also indent sub-queries, but do not place the parentheses on a line by themselves. Vertically align the opening and closing parentheses to ease visual parsing. Use the following style:

```sql
SELECT
  alias_a.column_a,
  alias_a.column_b,
  alias_b.column_c
FROM (
  SELECT
    alias_x.column_a,
    alias_x.column_b,
    alias_y.column_c
  FROM (
    SELECT
      column_a,
      column_b
    FROM
      table_a
  ) AS alias_x
  LEFT JOIN (
    SELECT
      column_a,
      column_c
    FROM
      table_b
  ) AS alias_y
  ON alias_x.column_a = alias_y.column_a
) AS alias_a
LEFT JOIN (
  SELECT
    column_a,
    column_b,
    column_c
  FROM
    table_c
) AS alias_b
ON alias_a.column_a = alias_b.column_a
  AND alias_a.column_b = alias_b.column_b
```

Even if there is only one item following the clause, break the line to make things easier to read.

Instead of:

```sql
SELECT column
FROM table
WHERE condition
```

Use:

```sql
SELECT
  column
FROM
  table
WHERE
  condition
```

Indentation of CASE statement:

```sql
SELECT
  ...
  CASE
    WHEN ...
      THEN ...
    WHEN ...
      AND ...
      THEN ...
    ELSE ...
  END AS alias_name
FROM
  ...
WHERE
  ...
```

* Format `WITH` clauses like so:

```sql
WITH
  alias_a AS (
    SELECT
      'A' AS value
  ),
  alias_b AS (
    SELECT
      something AS value
    FROM
      some_table
  )
SELECT
  ...
FROM
  alias_a

UNION DISTINCT

SELECT
  ...
FROM  
  alias_b
```

Commas are not operators, and should go at the end of the line (without preceding space).

For example, don't do this:

```sql
SELECT
  column_1
 ,column_2
 ,column_3
...
```

## Spacing

* Put a space before and after arithmetic operators like `+` and comparison operators like `=`. Please do this:

```sql
IF(column_1 != 'some value', column_2 + 3.14, CAST(NULL AS DOUBLE))
```

Don't do this:

```sql
IF(column_1!='some value',column_2+3.14,CAST(NULL AS DOUBLE))
```

* Put a space after a comma to separate arguments/fields/indexes. For example, do:

```sql
expr IN (expr1, expr2, ...)
```

Don't do this:

```sql
expr IN (expr1,expr2)
```

For `GROUP BY` use this pattern:

```sql
GROUP BY
  1, 2, 3, 4, 5, 6
```

Don't do this:

```sql
GROUP BY
  1,2,3,4,5,6
```


## Line lengths

Keep the length of lines at 120 characters or less.

Applying a limit of 120 characters to line lengths reduces the amount of horizontal scrolling. If you have to break a line, do
so before operators (logical operators, such as AND, or mathematical operators like +). Indent each broken line one level from the first line of the broken clause. For example, don't do this on one line:

```sql
IF(column_a = 'value','dhaksudKDKFSUDKFsdfuhkseuhsuf3762878234','GFufyidfus673ieKJHFSKDFkfusudfydsif7DSHGJFjfefJ') AS alias
```

Use:

```sql
IF(column_a = 'value', 'dhaksudKDKFSUDKFsdfuhkseuhsuf3762878234',
  'GFufyidfus673ieKJHFSKDFkfusudfydsif7DSHGJFjfefJ') AS alias
```


## Uppercase or lowercase

* Use UPPER CASE for SQL keywords (`SELECT, FROM, WHERE, IS NULL, AND, OR, SUM` etc.)
* Use lowercase for field names, aliases and user-created tables (using underscores to separate words e.g. `my_field_name`, `v_view_name`). Except where case-sensitivity is required.
* Use snake case for external functions.
* Casting functions to change data type should be uppercase (`INT32(expr), STRING(expr), DOUBLE(expr)`, `CAST(column_a AS VARCHAR)`, etc.)


## Use the full syntax

Always use the `AS` keyword in aliases. It highlights the fact that something is being renamed. It applies to both column and table aliases. For example, instead of this:

```sql
SELECT
  column_1 col_a,
  column_2 col_b,
  column_3 col_c
FROM
  table_1 tbl_a
LEFT JOIN
  ...
...
```

Please do this:

```sql
SELECT
  column_1 AS col_a,
  column_2 AS col_b,
  column_3 AS col_c
FROM
  table_1 AS tbl_a
LEFT JOIN
  ...
...
```

Avoid using fully-qualify tables in joins. Use aliases where possible. Whenever joins are being used, all fields have to be explicitly declared with aliases (even if there are no ambiguous fields). Table alias names should describe their use rather than
using `a` or other shortened nicknames.

Please do this:

```sql
SELECT
  alias_a.column_a,
  alias_a.column_b,
  alias_b.column_c
FROM (
  SELECT
    alias_x.column_a,
    alias_x.column_b,
    alias_y.column_c
  FROM (
    SELECT
      column_a,
      column_b
    FROM
      table_a
  ) AS alias_x
  LEFT JOIN (
    SELECT
      column_a,
      column_c
    FROM
      table_b
  ) AS alias_y
  ON alias_x.column_a = alias_y.column_a
) AS alias_a
LEFT JOIN (
  SELECT
    column_a,
    column_b,
    column_c
  FROM
    table_c
) AS alias_b
ON alias_a.column_a = alias_b.column_a
  AND alias_a.column_b = alias_b.column_b
```

Not this:

```sql
SELECT
  column_a,
  column_b,
  column_3
FROM (
  SELECT
    x.column_a,
    x.column_b,
    column_c
  FROM (
    SELECT
      column_a,
      column_b
    FROM
      table_a
  ) AS x
  LEFT JOIN (
    SELECT
      column_a,
      column_c
    FROM
      table_b
  ) AS y
  ON x.column_a = y.column_a
) AS aaa
LEFT JOIN (
  SELECT
    column_1,
    column_2,
    column_3
  FROM
    table_c
) AS b
ON column_a = column_1
  AND column_b = column_2
```

If no joins are used, do not use table aliasing. For example, rather than this:

```sql
SELECT
  tbl_a.column_1,
  tbl_a.column_2,
  tbl_a.column_3
FROM
  table_1 AS tbl_a
WHERE
  ...
```

Keep it simple:

```sql
SELECT
  column_1,
  column_2,
  column_3
FROM
  table_1
WHERE
  ...
```

Use `ON` instead of `USING` when joining tables, as it is more specific, avoids ambiguity, and behaves in a standard way
across database platforms.


## Grouping and ordering

SQL allows referring to columns by number, and that's the preferred way. Instead of this:

```sql
SELECT
  column_1,
  (UNIX_TIMESTAMP(date_column, 'yyyy-MM-dd') + 82800) * 1000 AS time_column,
  IF(column_3 = 'fhfdusSYDTEU&&7383473762', 'some_value', CONCAT(column_4, ' -> ', column_5) AS expression_column,
  COUNT(*) AS records
FROM
  table_1
GROUP BY
  column_1,
  (UNIX_TIMESTAMP(date_column, 'yyyy-MM-dd') + 82800) * 1000,
  IF(column_3 = 'fhfdusSYDTEU&&7383473762', 'some_value', CONCAT(column_4, ' -> ', column_5)
ORDER BY,
  column_1,
  time_column,
  expression_column
```

Please do this:

```sql
SELECT
  column_1,
  (UNIX_TIMESTAMP(date_column, 'yyyy-MM-dd') + 82800) * 1000 AS time_column,
  IF(column_3 = 'fhfdusSYDTEU&&7383473762', 'some_value', CONCAT(column_4, ' -> ', column_5) AS expression_column,
  COUNT(*) AS records
FROM
  table_1
GROUP BY
  1, 2, 3
ORDER BY,
  1, 2, 3
```

To avoid confusion in `GROUP BY` indexing, please move all aggregations to the bottom of the `SELECT` clause. Please do this:

```sql
SELECT
  column_1,
  (UNIX_TIMESTAMP(date_column, 'yyyy-MM-dd') + 82800) * 1000 AS time_column,
  IF(column_3 = 'fhfdusSYDTEU&&7383473762', 'some_value', CONCAT(column_4, ' -> ', column_5) AS expression_column,
  COUNT(*) AS records,
  SUM(column_6) AS amount
FROM
  table_1
GROUP BY
  1, 2, 3
```

Not this:

```sql
SELECT
  column_1,
  COUNT(*) AS records,
  (UNIX_TIMESTAMP(date_column, 'yyyy-MM-dd') + 82800) * 1000 AS time_column,
  SUM(column_6) AS amount,
  IF(column_3 = 'fhfdusSYDTEU&&7383473762', 'some_value', CONCAT(column_4, ' -> ', column_5) AS expression_column 
FROM
  table_1
GROUP BY
  1, 3, 5
```

It's recommended to break the line for every 10 grouping/ordering indexes:

```sql
SELECT
  ...
FROM
  table_1
GROUP BY
  1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
  11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
  21, 22, 23, 24, 25
```

For grouping with no aggregations, please use `SELECT DISTINCT` where possible:

```sql
SELECT DISTINCT
  column_1,
  column_2
FROM
  table_1
```

Don't do this:

```sql
SELECT
  column_1,
  column_2
FROM
  table_1
GROUP BY
  1, 2
```

