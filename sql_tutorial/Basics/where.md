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

