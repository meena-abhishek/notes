```sql
SELECT *
FROM table
WHERE condition1 
  OR condition2
  AND condition3
  AND column_name BETWEEN val1 AND val2
  AND column IN (....);
  ORDER BY column1 ASC, column2 DESC;
  ORDER BY 1,3 DESC;
  OFFSET 10
  LIMIT 5;
```

|                                |                                                           |
| ------------------------------ | --------------------------------------------------------- |
| `WHERE first_name LIKE 'a%'`   | Finds any values that starts with "a"                     |
| `WHERE first_name LIKE '%a'`   | Finds any values that ends with "a"                       |
| `WHERE first_name LIKE '%ae%'` | Finds any values that have "ae" in the middle             |
| `WHERE first_name LIKE '_b%'`  | Finds any values with "b" in the second position          |
| `WHERE first_name LIKE 'a%o'`  | Finds any values that starts with "a" and ends with "o"   |
| `WHERE first_name LIKE 'a___'` | Finds any value that starts with "a" and has 3 characters |

Theoretically, rows in a relational database such as Postgres, MySQL, SQL Server, etc. aren't stored in any particular or guaranteed sort order. In fact, executing an identical `SELECT *` query twice on the same dataset doesn't ensure the results will come back in the same order.

```sql
SELECT 
	ticker, 
    EXTRACT(YEAR FROM date) AS year, 
    ROUND(AVG(open),2) AS avg_open
FROM stock_prices 
GROUP BY ticker, year
ORDER BY year DESC;
```

`GROUP BY`, as you saw earlier, is all about grouping your data into categories!
```sql
SELECT ticker, MIN(open) AS min_open
FROM stock_prices
GROUP BY ticker
ORDER BY min_open DESC;
```

|WHERE|HAVING|
|---|---|
|When It Filters|Values BEFORE Grouping|Values AFTER Grouping|
|Operates On Data From|Individual Rows|Aggregated Values from Groups of Rows|
|Example|SELECT username, followers FROM instagram_data WHERE followers > 1000;|SELECT country FROM instagram_data GROUP BY country HAVING AVG(followers) > 100;|

```sql
SELECT ticker, AVG(open), MIN(open)
FROM stock_prices
GROUP BY ticker
HAVING AVG(open) > 200 AND MIN(open) > 100;
```
```sql
SELECT candidate_id
FROM candidates
WHERE skill IN ('Python', 'Tableau', 'PostgreSQL')
GROUP BY candidate_id
HAVING COUNT(skill) = 3
```

```sql
SELECT COUNT(DISTINCT user_id)
SELECT DISTINCT user_id, age
```

|**Operator**|**Description**|**Example**|**Result**|
|---|---|---|---|
|+|Addition|15 + 5|20|
|-|Subtraction|15 - 5|10|
|*|Multiplication|15 * 5|75|
|/|Division|15 / 5|3|
|%|Modulus (Remainder of Division)|14 % 5|4|
|^|Exponentiation (Not standard in all DBMS)|15 ^ 2|225|
|- (as a prefix)|Negation|-15|-15|

- P: Parentheses first
- E: Exponents (ie. `^`)
- MD: Multiplication and Division (left-to-right)
- AS: Addition and Subtraction (left-to-right)

```sql
SELECT 
  CAST(10 AS DECIMAL)/4,
  CAST(10 AS FLOAT)/4,
  10/CAST(6 AS DECIMAL),
  10/CAST(6 AS FLOAT); 
```
```sql
SELECT 
  10/6,
  10*1.0/6,
  10/6*1.0,
  10/(6*1.0);
```
```sql
SELECT 
  10::DECIMAL/4,
  10::FLOAT/4,
  10/4::DECIMAL,
  10/4::FLOAT,
  10::DECIMAL/6,
  10::FLOAT/6;
```
```sql
SELECT (part_column / total_column) * 100 AS percentage
FROM table;
```

In SQL, `NULL` indicates the **absence of a value**. Unlike an empty string or zero, `NULL` doesn't represent a specific value, but rather, a missing or unknown piece of information.

Here's an overview of what you'll learn in this tutorial:

- `IS NULL` and `IS NOT NULL`: Used to identify null and non-null values.
- `COALESCE()`: Returns the first non-null value from a list of arguments.
- `IFNULL()`: Substitutes null value with a specified value specified.

`NULL` values serve as placeholders for uncertain or absent data.

```sql
COALESCE(column_name, 'expression')
```
If `column_name` is NULL, it returns the specified 'expression'. Otherwise, it returns the value of `column_name`.

```sql
SELECT 
  book_title, 
  IFNULL(book_rating, 0) AS rated_books
FROM goodreads;
```
Returns the first non-NULL value from a list of expressions.
```sql
SELECT COALESCE(NULL, NULL, 100, 200);
-- Output: 100
```
In SQL Server, returns a replacement value if the first expression is `NULL`.
```sql
SELECT ISNULL(NULL, 100);
-- Output: 100
```

```sql
SELECT 
  actor, 
  character,
  CASE 
    WHEN engagement_rate >= 8.0 THEN 'High Engagement'
    WHEN engagement_rate BETWEEN 6.0 AND 7.9 THEN 'Moderate Engagement'
    WHEN engagement_rate < 6.0 THEN 'Low Engagement'
  END AS engagement_category
FROM marvel_avengers
WHERE platform IN ('TikTok', 'Instagram');
```

```sql
SELECT 
  actor, 
  character, 
  platform
FROM marvel_avengers
WHERE 
  CASE 
    WHEN platform = 'Instagram' THEN followers >= 500000
    WHEN platform = 'Twitter' THEN followers >= 200000
    ELSE followers >= 100000
  END;
```
```sql
SELECT
  platform,
  COUNT(CASE 
    WHEN followers >= 500000 THEN 1
    ELSE NULL
  END) AS popular_actor_count,
  COUNT(CASE 
    WHEN followers < 500000 THEN 1
    ELSE NULL
  END) AS less_popular_actor_count
FROM marvel_avengers
GROUP BY platform;
```
```sql
SELECT
  platform,
  SUM(CASE 
    WHEN engagement_rate >= 8.0 THEN followers
    ELSE 0
  END) AS high_engagement_followers_sum,
  SUM(CASE 
    WHEN engagement_rate < 8.0 THEN followers
    ELSE 0
  END) AS low_engagement_followers_sum
FROM marvel_avengers
GROUP BY platform;
```
same for AVG, just use NULL for else

```sql
SELECT *
FROM artists
JOIN songs
  ON artists.artist_id = songs.artist_id;
```

|Type of Joins|Description|
|---|---|
|INNER JOIN|Returns only the **rows with matching values** from both tables.|
|LEFT JOIN|Returns **all the rows from the left table** and the **matching rows from the right table**.|
|RIGHT JOIN|Returns **all the rows from the right table** and the **matching rows from the left table**.|
|FULL OUTER JOIN|Returns all rows when there is a **match in either the left or the right table**. If there is no match, `NULL` values are returned for columns from the table without a match.|

If there is no matching data in the right table, the result will still include the left table's data with `NULL` values in the columns from the right table.

**A little tip:** Be careful with `FULL OUTER JOIN` as it can introduce duplicate rows to your results without you realizing!

```sql
SELECT 
  g.book_title, 
  g.book_rating, 
  o.order_date, 
  d.delivery_status
FROM goodreads g
JOIN orders o 
  ON g.book_id = o.book_id
    AND g.book_rating > 4.0
INNER JOIN deliveries d 
  ON o.order_id = d.order_id
    AND d.delivery_status = 'Delivered';
```
- `CURRENT_DATE`, `CURRENT_TIME` and `CURRENT TIMESTAMP` to return current date, time and timestamp.
- Comparison operators `<`, `>`, `=`, `<=`, and `>=` to compare dates
- `EXTRACT()` and `DATE_PART()` to extract specific components of date.
- `DATE_TRUNC()` to round down date or timestamp into specific level of precision.
- `INTERVAL` to add or subtract time intervals in calculations.
- `TO_CHAR()` to convert date or timestamp into strings.
- `::DATE`, `TO_DATE()`, `::TIMESTAMP`, and `TO_TIMESTAMP()` to convert strings into date or timestamp.

- `CURRENT_DATE`: Returns today's date.
- `CURRENT_TIME`: Returns the current time without the date.
- `CURRENT_TIMESTAMP` (or `NOW()`): Returns the current date and time.
```sql
EXTRACT(YEAR FROM sent_date) AS extracted_year,
DATE_PART('year', sent_date) AS part_year,
```

The `EXTRACT()` function extracts a specific component (i.e. year, month, day, hour, or minute) from a date or timestamp. You can also use `DATE_PART()` for the same purpose.

The `DATE_TRUNC()` rounds down a date or timestamp to a specified unit of time. In other words, it trims the finer details and retains the specified unit.

```sql
SELECT 
  message_id,
  sent_date,
  DATE_TRUNC('month', sent_date) AS truncated_to_month,
  DATE_TRUNC('day', sent_date) AS truncated_to_day,
  DATE_TRUNC('hour', sent_date) AS truncated_to_hour  
FROM messages
LIMIT 3;
```

Here's what's happening in the results:

- `truncated_to_month`: It rounds down the date to the beginning of the month. For example, if a message was sent on August 3rd, 2022 at 16:43, it's snapped to August 1st, 2022, while retaining the year and month.
- `truncated_to_day`: It rounds down the date to the beginning of the day. The same August 3rd message becomes August 3rd, 2022, with the hour, minute, and seconds set to zero.
- `truncated_to_hour`: It rounds down the time to the beginning of the hour. The message becomes August 3rd, 2022 at 16:00, with the minutes and seconds set to zero.

The `INTERVAL` function is used to handle date and time gaps by adding and subtracting intervals such as "3 days", "5 hours", "45 minutes".
```sql
SELECT 
  message_id,
  sent_date,
  sent_date + INTERVAL '2 days' AS add_2days,
  sent_date - INTERVAL '3 days' AS minus_3days,
  sent_date + INTERVAL '2 hours' AS add_2hours,
  sent_date - INTERVAL '10 minutes' AS minus_10mins
FROM messages
LIMIT 3;
```

```sql
SELECT 
  message_id,
  sent_date,
  TO_CHAR(sent_date, 'YYYY-MM-DD HH:MI:SS') AS formatted_iso8601,
  TO_CHAR(sent_date, 'YYYY-MM-DD HH:MI:SS AM') AS formatted_12hr,
  TO_CHAR(sent_date, 'Month DDth, YYYY') AS formatted_longmonth,
  TO_CHAR(sent_date, 'Mon DD, YYYY') AS formatted_shortmonth,
  TO_CHAR(sent_date, 'DD Month YYYY') AS formatted_daymonthyear,
  TO_CHAR(sent_date, 'Month') AS formatted_dayofmonth,
  TO_CHAR(sent_date, 'Day') AS formatted_dayofweek
FROM messages
LIMIT 3;
```

- `::DATE` or `TO_DATE()`: Convert strings into dates.
- `::TIMESTAMP` or `TO_TIMESTAMP()`: Convert strings into timestamps.
