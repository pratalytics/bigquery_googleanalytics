# bigquery_googleanalytics
A lot of digital analyst have worked on Google analytics or Adobe Analytics but they don't have enough experience with SQL or Big Query. I faced the same issue and while there were enough tutorial online, I didnt find one which gave instructions on how to effectively use SQL for the GA 360 dataset. This is my attempt to create this tutorial.

We go from knowing nothing about SQL to writing advanced scripts for finding user cohorts and purchase behavior.


## Selecting columns

```sql
-- Get in a practice to comment your codes
-- Follow a styling format and stick to it
SELECT trafficSource.source AS source,
       device.deviceCategory AS device
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 LIMIT 5
```

### Conceptual  questions 
1. What are the two required clauses in any SQL query and what is the order they should be in?
2. How do you separate multiple columns?
3. How do you select every column in the table?
4. How do you you format column names?
5. How do you limit your results?

### Exercises
1. Select columns which has client ID, the visit number, session ID, session start time and date
2. Select any two columns from the following records - totals, trafficSource, device, geoNetwork. Can you select columns from hits.eCommerceAction? What error do you get?

## Filtering the results

### Conceptual Questions
1. Which clause to use to filter the results?
2. What is the order of the SQL clauses?
3. What are the different types of comparision operators? How do they work on non-numerical data?
4. How can you perform aritmethic operations across columns on values in a given row?
5. Which logical operator will you use to match on similar values rather than exact one?
6. Which symbol is used to represent a wildcard and it is used with which logical operator?
7. Is the `LIKE` operator case sensitive?
8. Which logical operator allows you to specifiy a list of values that you want in the results?
9. Can the `IN` operator be used on non-numerical values?
10. Which logical operator allows you to select rows that are within a specific range?
11. Does the `BETWEEN` operator include the boundry values?
12. Is a `NULL` value same as zero? Is it same as a blank string? Can you perform aritmethic on `NULL` values?
13. Which operator allows you to filter for `NULL` values in a column?
14. Which logical operator allows you to select only rows that must satisfy all the conditions (Minimum 2)? 
15. Which logical operator allows you to select only rows that satisfy either/any of the conditions (Minimum 2)?
16. How can we combine multiple `AND` and `OR` operators?

### Exercises
1. 





























