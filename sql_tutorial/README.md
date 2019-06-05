# SQL Tutorial

## Exploring the GA 360 sample dataset
* How many tables are there in the dataset?
* What information is encapsulated in one row of a GA 360 table?
* What are the differences compared to an excel table? 
* What is a project, a datset and a table?


## Selecting columns

```sql
-- Get in a practice to comment your codes
-- Follow a styling format and stick to it
SELECT trafficSource.source AS source,
       device.deviceCategory AS device
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
 LIMIT 5
```

### Review  questions 
* What are the two required clauses in any SQL query and what is the order they should be in?
* How do you format the table name in the `FROM` clause?
* How do you separate multiple columns?
* How do you select every column in the table?
* How do you rename column names?
* How do you limit your results?



## Quering multiple tables, Filtering and Sorting the results

### Querying Multiple tables

#### Review Questions
* What are wildcard tables and when to use it?
* Are the tables in GA 360 big query dataset eligible for wildcard table operations? Why?
* What is the table wildcard symbol and how to use it?
* In which clause do you use the `_TABLE_SUFFIX` operator?
* How do you query all the tables?
* How do you query a range of tables?
* How do you query selected tables?
* True or False: Shorter or empty prefixes work better than the longer prefixes

### Review Questions
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
17. Which operator works similar to _does not contain_?
18. Which clause allows you to reorder your results based on the data in one or more columns?
19. How can you sort the results in descending order?
* How can you sort by multiple columns? 
* Can you substitue the column name by numbers in the `ORDER BY` clause? What do the numbers correspond to?



## Aggregate and Group BY

## Review questions
* Which aggregate function is used to count the number of rows in a particular column?
* How do you count all the rows in a given dataset?
* How do `NULL` values affect the count of individual columns?
* Can `COUNT` be used on non-numnerical columns?
* Which aggreagate function totals the values in a given column?
* How do `SUM` treats `NULL` values?
* Which aggregate functions return the highest and the loweset values in a particular column?
* Which aggregate function calculates the average of values in a given column? What is it limitation?
* Which clause allows you to separate data into groups which can be aggregated independently of one another?
* How do you use `GROUP BY` with multiple coulmns?
* As with the `ORDER BY` clause can you substitue the column names with numbers in the `GROUP BY` clause?
* If you want to control how the aggregations are grouped together, you need to use what?
* Can you use `WHERE` clause to filter on aggregated columns? How do you filter on aggregated columns?
* Which clause to use to look at only unqiue values in a column?
* How do you use `DISTINCT` with `COUNT`?


# Solutions

```sql
SELECT device.deviceCategory AS device_category,
       channelGrouping,
       trafficSource.source AS source,
       COUNT(fullVisitorId) AS users,
       SUM(totals.newVisits) AS new_users,
       SUM(totals.visits) AS sessions,
       SUM(totals.pageviews) AS pageviews,
       SUM(totals.totalTransactionRevenue)/1000000 AS revenue,
       SUM(totals.transactions) AS transactions
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
 WHERE device.deviceCategory = 'desktop'
   AND (channelGrouping IN ('Organic Search', 'Direct'))
   AND _TABLE_SUFFIX BETWEEN '0101' AND '0331'
 GROUP BY 1,2,3
HAVING transactions IS NOT NULL
   AND new_users > 1000
 ORDER BY transactions DESC
```
