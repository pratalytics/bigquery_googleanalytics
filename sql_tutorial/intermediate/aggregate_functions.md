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

### HAVING Clause
If you want to filter on an aggregate field, `WHERE` clause doesn't work and you have to use a `HAVING` clause.

```sql
-- HAVING clause
SELECT channelGrouping,
       device.deviceCategory AS device_category,
       SUM(totals.pageviews) AS pageviews
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 GROUP BY channelGrouping, device.deviceCategory
HAVING SUM(totals.pageviews) > 50 
 ORDER BY channelGrouping, pageviews DESC  
```

#### Query clause order
The order in which you write the clauses is important. Here’s the order for everything you’ve learned so far:
1. `SELECT`
2. `FROM`
3. `WHERE`
4. `GROUP BY`
5. `HAVING`
6. `ORDER BY`

## DISTINCT Clause
You’ll occasionally want to look at only the unique values in a particular column. You can do this using `SELECT DISTINCT` syntax. If you include two (or more) columns in a `SELECT DISTINCT` clause, your results will contain all of the unique pairs of those two columns.

```sql
-- DISTINCT clause
SELECT DISTINCT channelGrouping, device.deviceCategory  
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 ORDER BY channelGrouping, device.deviceCategory 
```

### Using DISTINCT in aggregations
You can use `DISTINCT` when performing an aggregation. You’ll probably use it most commonly with the `COUNT` function.

```sql
-- DISTINCT with COUNT
SELECT COUNT(fullVisitorId) AS total_users,
       COUNT(DISTINCT fullVisitorId) AS unique_users
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
```
### CASE statement
The `CASE` statement is SQL’s way of handling if/then logic. The `CASE` statement is followed by at least one pair of `WHEN` and `THEN` statements. Every `CASE` statement must end with the `END` statement. The `ELSE` statement is optional, and provides a way to capture values not specified in the `WHEN/THEN` statements.

```sql
-- CASE statement
SELECT DISTINCT device.deviceCategory, 
       CASE device.deviceCategory
       WHEN 'desktop' THEN 'computer'
       WHEN 'mobile' THEN 'phone'
       ELSE 'others'
       END AS alt_device_names
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
```

### Adding multiple conditions to a CASE statement
You can also string together multiple conditional statements with `AND` and `OR` the same way you might in a `WHERE` clause:

```sql
-- CASE statement with multiple conditions
SELECT trafficSource.source,
       SUM(totals.pageviews) AS pageviews,
       COUNT(fullVisitorId) AS users,
       CASE 
       WHEN SUM(totals.pageviews) > 100 
       AND COUNT(fullVisitorId) > 100 THEN 'high_engagement'
       WHEN SUM(totals.pageviews) > 50
       AND COUNT(fullVisitorId) > 50 THEN 'medium_engagement'
       ELSE 'low_engagement'
       END AS engagement_category
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 GROUP BY trafficSource.source
 ORDER BY pageviews DESC 
```

### Using CASE with aggregate functions
`CASE`’s slightly more complicated and substantially more useful functionality comes from pairing it with aggregate functions. Using the `WHERE` clause only allows you to count one condition. The below query is an excellent place to use numbers instead of columns in the `GROUP BY` clause because repeating the `CASE` statement in the `GROUP BY` clause would make the query obnoxiously long. Alternatively, you can use the column’s alias in the `GROUP BY` clause

```sql
-- CASE statement with aggregate functions
SELECT CASE 
       WHEN channelGrouping = 'Organic Search'
       AND device.deviceCategory = 'desktop' THEN 'organic_desktop'
       WHEN channelGrouping = 'Organic Search'
       AND device.deviceCategory = 'mobile' THEN 'organic_mobile'
       WHEN channelGrouping = 'Direct'
       AND device.deviceCategory = 'desktop' THEN 'direct_desktop'
       WHEN channelGrouping = 'Direct'
       AND device.deviceCategory = 'mobile' THEN 'direct_mobile' 
       ELSE 'others'
       END AS segments,
       COUNT(fullVisitorId) AS users,
       SUM(totals.pageviews) AS pageviews
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 GROUP BY 1
 ORDER BY pageviews DESC
```

### Using CASE inside of aggregate functions
In the previous examples, data was displayed vertically, but in some instances, you might want to show data horizontally. This is known as _pivoting_

```sql
-- CASE statement inside of aggregate function
SELECT COUNT(CASE 
             WHEN channelGrouping = 'Organic Search'
             AND device.deviceCategory = 'desktop' THEN 1
             ELSE NULL
             END) AS organic_desktop,
       COUNT(CASE
             WHEN channelGrouping = 'Organic Search'
             AND device.deviceCategory = 'mobile' THEN 1
             ELSE NULL
             END) AS organic_mobile,
       COUNT(CASE
             WHEN channelGrouping = 'Direct'
             AND device.deviceCategory = 'desktop' THEN 1
             ELSE NULL
             END) AS direct_desktop,
       COUNT(CASE
             WHEN channelGrouping = 'Direct'
             AND device.deviceCategory = 'mobile' THEN 1
             ELSE NULL
             END) AS direct_mobile,
       COUNT(CASE
             WHEN (channelGrouping = 'Organic Search' AND device.deviceCategory = 'desktop')
             OR   (channelGrouping = 'Organic Search' AND device.deviceCategory = 'mobile')
             OR   (channelGrouping = 'Direct'AND device.deviceCategory = 'desktop')
             OR   (channelGrouping = 'Direct'AND device.deviceCategory = 'mobile') THEN NULL
             ELSE 1
             END) AS others
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
```

_Note: The above query is substantially bigger and complicated the queries we have written till now. I will add a In plain english section where I will try to expalin the query in a simple language._

###### Plain English
In excel, we will compute as below using a if else formula for each of the caclulated columns.

channelgrouping|devicecategory|organic_desktop|organic_mobile|direct_desktop|direct_mobile|others
---------------|--------------|---------------|--------------|--------------|-------------|-------
Organic Search |desktop       |1              | NULL         |NULL          |NULL         |NULL
Organic Search |mobile        |NULL           |1             |NULL          |NULL         |NULL
Direct         |desktop       |NULL           |NULL          |1             |NULL         |NULL
