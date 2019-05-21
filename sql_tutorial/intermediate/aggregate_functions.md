# Aggregate Functions

You will use aggregate functions all the time, so it’s important to get comfortable with them. The functions themselves are the same ones you will find in Excel or any other analytics program. The Basic SQL Tutorial also pointed out that arithmetic operators only perform operations across rows. Aggregate functions are used to perform operations across entire columns (which could include millions of rows of data or more). \
Here’s a quick preview:

`COUNT` counts how many rows are in a particular column.\
`SUM` adds together all the values in a particular column.\
`MIN` and `MAX` return the lowest and highest values in a particular column, respectively.\
`AVG` calculates the average of a group of selected values.

## COUNT Function
### Counting all rows
`COUNT` is an aggregate function for counting the number of rows in a particular column. `COUNT` is the easiest aggregate function to begin with because verifying your results is extremely simple.

```sql
-- COUNT All Rows
SELECT COUNT(*) AS count_all_rows
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
```
_Note: Typing COUNT(1) has the same effect as COUNT(*). Which one you use is a matter of personal preference. In Big Query you can verify it by looking at Rows per page at the bottom. Secondly, always use a alias with any aggreagate functions_

### Counting individual columns
Things start to get a little bit tricky when you want to count individual columns. The following code will provide a count of all of rows in which the `totals.bounces` column is __not null__. You can also use `COUNT` on non-numerical columns

```sql
-- COUNT Column with null Rows
SELECT COUNT(totals.bounces) AS count_bounces
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
```
You’ll notice that this result is lower than what you got with `COUNT(*)`. That’s because totals.bounces has nulls (For a bounced session, the value is 1, otherwise it is null.)

## SUM Function
`SUM` is an aggregate function that totals the values in a given column. Unlike `COUNT`, you can only use `SUM` on columns containing numerical values. An important thing to remember: __aggregators only aggregate vertically__. If you want to perform a calculation across rows, you would do this with __simple arithmetic__.\
You don’t need to worry as much about the presence of nulls with `SUM` as you would with `COUNT`, as `SUM` treats nulls as 0.

```sql
-- SUM Function
SELECT SUM(totals.pageviews) AS pageviews
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
```

## MIN/MAX Function
`MIN` and `MAX` are aggregation functions that return the lowest and highest values in a particular column.

```sql
-- MIN/MAX Function
SELECT MIN(totals.pageviews) AS min_pageview,
       MAX(totals.pageviews) AS max_pageview
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
```

## AVG Function
`AVG` is an aggregate function that calculates the average of a selected group of values. It’s very useful, but has some limitations. First, it can only be used on numerical columns. Second, it ignores nulls completely.

```sql
-- AVG Function
SELECT AVG(totals.pageviews) AS avg_pageview_per_session_per_day,
       AVG(totals.bounces) AS avg_bounces_per_session_per_day -- Incorrect way of calculating
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
```
_Note: There are some cases in which you’ll want to treat null values as 0. For these cases, you’ll want to write a statement that changes the nulls to 0_

## GROUP BY Clause
Aggregate functions like `COUNT`, `AVG`, and `SUM` have something in common: they all aggregate across the entire table. But what if you want to aggregate only part of a table? For example, you might want to a total pageviews for each device.

In situations like this, you’d need to use the `GROUP BY` clause. `GROUP BY` allows you to separate data into groups, which can be aggregated independently of one another. You can group by multiple columns, but you have to separate column names with commas.

### Using GROUP BY with ORDER BY
The order of column names in your `GROUP BY` clause doesn’t matter—the results will be the same regardless. If you want to control how the aggregations are grouped together, use `ORDER BY`. 

```sql
-- GROUP BY with ORDER BY clause
SELECT channelGrouping,
       device.deviceCategory AS device_category,
       SUM(totals.pageviews) AS pageviews
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 GROUP BY channelGrouping, device.deviceCategory
 ORDER BY channelGrouping, pageviews DESC
```
