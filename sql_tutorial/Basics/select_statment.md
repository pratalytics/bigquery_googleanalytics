# SELECT and FROM

There are two required ingredients in any SQL query: `SELECT` and `FROM` —and they have to be in that order. `SELECT` indicates which columns you’d like to view, and `FROM` identifies the table that they live in.

```sql
-- Get in a practice to comment your codes
-- Follow a styling format and stick to it
SELECT trafficSource.source
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
```


Whenever you select multiple columns, they must be separated by commas, but you should not include a comma after the last column name.

```sql
-- Select multiple columns
SELECT trafficSource.source,
       device.deviceCategory
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
```

If you want to select every column in a table, you can use `*` instead of the column names. Though not recommended to do so in Big query.\
If you’d like your results to look a bit more presentable, you can use aliases
```sql
-- Using aliases
SELECT trafficSource.source AS source,
       device.deviceCategory AS device_category
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
```
