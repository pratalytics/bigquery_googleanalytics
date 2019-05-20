# WHERE clause
Once you know how to view some data using `SELECT` and `FROM`, the next step is filtering the data using the `WHERE` clause. Here’s what it looks like:
```sql
-- Using where clause
SELECT trafficSource.source AS source,
       device.deviceCategory AS device_category
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 WHERE device.deviceCategory = 'mobile'
```
_Note_: the clauses always need to be in this order: `SELECT`, `FROM`, `WHERE`.

### How does WHERE clause works?
If you write a `WHERE` clause that filters based on values in one column, you’ll limit the results in all columns to rows that satisfy the condition. The idea is that each row is one data point or observation, and all the information contained in that row belongs together.

## Comparison Operators
You can filter your results in a number of ways using comparison and logical operators.
### Comparison operators on numerical data
The most basic way to filter data is using comparison operators. The easiest way to understand them is to start by looking at a list of them:

Comparison Operator | Symbol
--------------------|--------
Equal to | `=`
Not Equal to | `<>` or `!=`
Less Than | `<`
Less Than or Equal to | `<=`
Greater Than | `>`
Greaterh Than or Equal to | `>=`

These comparison operators make the most sense when applied to numerical columns. For example, let’s use `>` to return only the rows where the pageviews are greater than 100

```sql
-- Comparison Operator
SELECT trafficSource.source AS source,
       device.deviceCategory AS device_category,
       totals.pageviews	AS pageviews
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 WHERE totals.pageviews	> 100
```

### Comparison operators on non-numerical data
All of the above operators work on non-numerical data as well. `=` and `!=` make perfect sense—they allow you to select rows that match or don’t match any value, respectively.
```sql
-- Comparison Operator on String columns
SELECT trafficSource.source AS source,
       device.deviceCategory AS device_category,
       totals.pageviews	AS pageviews
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 WHERE device.deviceCategory != 'desktop'
```

## Arithmetic in SQL
You can perform arithmetic in SQL using the same operators you would in Excel: `+`, `-`, `*`, `/`. However, in SQL you can only perform arithmetic across columns on values in a given row. To clarify, you can only add values in multiple columns from the same row together using `+`—if you want to add values across multiple rows, you’ll need to use __aggregate functions__
```sql
-- Sum of visits and bounces columns
SELECT trafficSource.source AS traffic_source,
       trafficSource.medium	AS medium,
       device.deviceCategory AS device_category,
       totals.visits + totals.bounces AS visits_bounces
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
```

## SQL Logical operators
You’ll likely also want to filter data using several conditions. Logical operators allow you to use multiple comparison operators in one query.

`LIKE` allows you to match similar values, instead of exact values.\
`IN` allows you to specify a list of values you’d like to include.\
`BETWEEN` allows you to select only rows within a certain range.\
`IS NULL` allows you to select rows that contain no data in a given column.\
`AND` allows you to select only rows that satisfy two conditions.\
`OR` allows you to select rows that satisfy either of two conditions.\
`NOT` allows you to select rows that do not match a certain condition.

### LIKE operator

`LIKE` is a logical operator in SQL that allows you to match on similar values rather than exact ones.
```sql
-- LIKE operator
SELECT trafficSource.source AS traffic_source,
       trafficSource.medium	AS medium,
       device.deviceCategory AS device_category,
       channelGrouping
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 WHERE  trafficSource.source LIKE '%google%'
```

The `%` used above represents any character or set of characters. In this case, `%` is referred to as a “wildcard.” You can also use `_` to substitute for an individual character.

### IN operator

`IN` is a logical operator that allows you to specify a list of values that you’d like to include in the results.

```sql
-- IN operator
SELECT trafficSource.source AS traffic_source,
       trafficSource.medium	AS medium,
       device.deviceCategory AS device_category,
       channelGrouping
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 WHERE  device.deviceCategory IN ('desktop', 'tablet')
```

### BETWEEN operator

`BETWEEN` is a logical operator that allows you to select only rows that are within a specific range. It has to be paired with the `AND` operator. `BETWEEN` includes the range bounds that you specify in the query, in addition to the values between them.

```sql
-- BETWEEN operator
SELECT trafficSource.source AS traffic_source,
       trafficSource.medium	AS medium,
       device.deviceCategory AS device_category,
       totals.pageviews AS pageviews
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 WHERE  totals.pageviews BETWEEN 10 AND 20
```

### IS NULL operator
`IS NULL` is a logical operator that allows you to exclude rows with missing data from your results.
```sql
-- IS NULL operator
SELECT trafficSource.source AS traffic_source,
       trafficSource.medium	AS medium,
       device.deviceCategory AS device_category,
       totals.bounces AS bounces
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 WHERE totals.bounces IS NULL
```

### AND Operator
`AND` is a logical operator that allows you to select only rows that satisfy two conditions. You can use SQL’s `AND` operator with additional `AND` statements or any other comparison operator, as many times as you want.

```sql
-- AND operator
SELECT trafficSource.source AS traffic_source,
       trafficSource.medium	AS medium,
       device.deviceCategory AS device_category,
       totals.pageviews AS pageviews
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 WHERE trafficSource.medium = 'referral'
   AND device.deviceCategory IN ('tablet', 'mobile')
```

### OR operator
`OR` is a logical operator that allows you to select rows that satisfy either of two conditions. It works the same way as `AND`, which selects the rows that satisfy both of two conditions. You can combine `AND` with `OR` using parenthesis.

```sql
-- OR operator
SELECT trafficSource.source AS traffic_source,
       trafficSource.medium	AS medium,
       device.deviceCategory AS device_category,
       totals.pageviews AS pageviews
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 WHERE trafficSource.medium = 'referral'
    OR device.deviceCategory IN ('tablet', 'mobile')
```

_Note_: Did you notice the difference between the two queries?

### NOT operator
`NOT` is a logical operator that you can put before any conditional statement to select rows for which that statement is false. `NOT` is commonly used with `LIKE`. `NOT` is also frequently used to identify non-null rows, but the syntax is somewhat special—you need to include `IS` beforehand.

```sql
-- NOT operator
SELECT trafficSource.source AS traffic_source,
       trafficSource.medium	AS medium,
       device.deviceCategory AS device_category,
       totals.bounces AS bounces
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 WHERE totals.bounces IS NOT NULL
   AND device.deviceCategory NOT IN ('tablet', 'mobile')
```

### ORDER BY operator
Once you’ve learned how to filter data, it’s time to learn how to sort data. The `ORDER BY` clause allows you to reorder your results based on the data in one or more columns. If you’d like your results in the opposite order (referred to as descending order), you need to add the `DESC` operator
