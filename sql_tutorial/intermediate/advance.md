# Advance stuff

## Changing a column’s data type
```sql
CAST(column_name AS integer)
```

## Date Format
Assuming you’ve got some dates properly stored as a `date` or `time` data type, you can do some pretty powerful things. Maybe you’d like to calculate a field of dates a week after an existing field. Or maybe you’d like to create a field that indicates how many days apart the values in two other date fields are. These are trivially simple, but it’s important to keep in mind that the data type of your results will depend on exactly what you are doing to the dates.

### Big Query Datetime functions

```sql
SELECT CURRENT_DATETIME('+05:30') as now,
       DATETIME_ADD(CURRENT_DATETIME('+05:30'), INTERVAL 7 DAY) AS days_7_from_now,
       DATETIME_SUB(CURRENT_DATETIME('+05:30'), INTERVAL 1 MONTH) AS previous_month,
       DATETIME_DIFF(DATETIME(2019,06,15,0,0,0), CURRENT_DATETIME('+05:30'), DAY) AS no_of_days_remaning,
       DATETIME_TRUNC(CURRENT_DATETIME('+05:30'), YEAR) AS truncated_date,
       FORMAT_DATETIME('%y', CURRENT_DATETIME('+05:30')) AS formatted_date,
       PARSE_DATETIME('%d/%m/%Y', '31/07/1987') AS parsed_date
```

### Big query Timestamp functions

`CURRENT_TIMESTAMP()` produces a `TIMESTAMP` value that is continuous, non-ambiguous, has exactly 60 seconds per minute and does not repeat values over the leap second.

```sql
SELECT CURRENT_DATETIME('+05:30') AS now,
       CURRENT_TIMESTAMP() AS current_time,
       EXTRACT(DATE FROM CURRENT_TIMESTAMP() AT TIME ZONE '+05:30') AS extracted_time,
       EXTRACT(DATE FROM CURRENT_DATETIME()) AS extracted_date,
       STRING(CURRENT_TIMESTAMP(), '+05:30') AS string_time,
       TIMESTAMP(DATE '2020-05-12', '+05:30') AS timestamp_from_date
       -- Rest of the functions similar to DATETIME
```

#### Exercises

```sql
SELECT permalink,
       companies.name AS company_name,
       acquisitions.acquired_at_cleaned,
       companies.founded_at_clean,
       DATE_DIFF(EXTRACT(DATE FROM acquisitions.acquired_at_cleaned), PARSE_DATE('%F', companies.founded_at_clean), YEAR)
       
  FROM `sandbox-bq.join_training.companies_clean_date` AS companies
       INNER JOIN `sandbox-bq.join_training.acquisition_clean_date` AS acquisitions
       ON companies.permalink = acquisitions.company_permalink 
 WHERE founded_at_clean IS NOT NULL
 ORDER BY acquisitions.acquired_at_cleaned DESC
```


Write a query that counts the number of companies acquired within 3 years, 5 years, and 10 years of being founded (in 3 separate columns and it is non cummulative). Include a column for total companies acquired as well. Group by category and limit to only rows with a founding date.

```sql
SELECT companies.category_code AS category,
       COUNT(CASE 
             WHEN DATE_DIFF(EXTRACT(DATE FROM acquisitions.acquired_at_cleaned), PARSE_DATE('%F', companies.founded_at_clean), YEAR) IS NOT NULL THEN 1
             ELSE NULL END) AS acquired_companies_count,
       COUNT(CASE
             WHEN DATE_DIFF(EXTRACT(DATE FROM acquisitions.acquired_at_cleaned), PARSE_DATE('%F', companies.founded_at_clean), YEAR) <= 3 THEN 1
             ELSE NULL END) AS acquired_companies_3years,
       COUNT(CASE
             WHEN DATE_DIFF(EXTRACT(DATE FROM acquisitions.acquired_at_cleaned), PARSE_DATE('%F', companies.founded_at_clean), YEAR) <= 5
             AND DATE_DIFF(EXTRACT(DATE FROM acquisitions.acquired_at_cleaned), PARSE_DATE('%F', companies.founded_at_clean), YEAR) >= 4 THEN 1
             ELSE NULL END) AS acquired_companies_5years,
       COUNT(CASE
             WHEN DATE_DIFF(EXTRACT(DATE FROM acquisitions.acquired_at_cleaned), PARSE_DATE('%F', companies.founded_at_clean), YEAR) <= 10
             AND DATE_DIFF(EXTRACT(DATE FROM acquisitions.acquired_at_cleaned), PARSE_DATE('%F', companies.founded_at_clean), YEAR) >= 6 THEN 1
             ELSE NULL END) AS acquired_companies_10years,
       COUNT(CASE
             WHEN DATE_DIFF(EXTRACT(DATE FROM acquisitions.acquired_at_cleaned), PARSE_DATE('%F', companies.founded_at_clean), YEAR) >= 11 THEN 1
             ELSE NULL END) AS acquired_companies_10plusyears
       
  FROM `sandbox-bq.join_training.companies_clean_date` AS companies
       LEFT JOIN `sandbox-bq.join_training.acquisition_clean_date` AS acquisitions
       ON companies.permalink = acquisitions.company_permalink
       AND companies.founded_at_clean IS NOT NULL
    
 GROUP BY category
 ORDER BY acquired_companies_count DESC 

```

## Subqueries

```sql
SELECT *
  FROM (SELECT *
          FROM `sandbox-bq.join_training.sf_crime_2014`
          WHERE day_of_week = 'Friday') AS sub
 WHERE sub.resolution = 'NONE'
```
Let’s break down what happens when you run the above query:

First, the BQ engine runs the “inner query”—the part between the parentheses. If you were to run this on its own, it would produce a result set like any other query. It might sound like a no-brainer, but it’s important: your inner query must actually run on its own, as the database will treat it as an independent query. Once the inner query runs, the outer query will run _using the results from the inner query as its underlying table_

### Using subqueries to aggregate in multiple stages

```sql
SELECT FORMAT_DATE('%m',PARSE_DATE('%m/%d/%Y',SUBSTR(sub_1.date,0,10))) AS month,
       sub_1.day_of_week,
       AVG(sub_1.incidents) AS avg_no_incidents
  FROM (SELECT day_of_week,
               date,
               COUNT(incidnt_num) AS incidents
         FROM `sandbox-bq.join_training.sf_crime_2014`
        GROUP BY day_of_week, date) AS sub_1
  GROUP BY 1,2
  ORDER BY 1, 3
```

### Subqueries in conditional logic
You can use subqueries in conditional logic (in conjunction with `WHERE`, `JOIN/ON`, or `CASE`

```sql
SELECT date
  FROM `sandbox-bq.join_training.sf_crime_2014`
 WHERE date = (SELECT MIN(date)
                 FROM `sandbox-bq.join_training.sf_crime_2014`)
```

The above query works because the result of the subquery is only one cell. Most conditional logic will work with subqueries containing one-cell results. However, `IN` is the only type of conditional logic that will work when the inner query contains multiple results

```sql
SELECT date,
       category
  FROM `sandbox-bq.join_training.sf_crime_2014`
 WHERE date IN (SELECT date
                 FROM `sandbox-bq.join_training.sf_crime_2014`
                ORDER BY date
                LIMIT 5)
```

_Note that you should not include an alias when you write a subquery in a conditional statement. This is because the subquery is treated as an individual value (or set of values in the `IN` case) rather than as a table._

### Joining subqueries
It’s fairly common to join a subquery that hits the same table as the outer query rather than filtering in the `WHERE` clause. The following query produces the same results as the previous example

```sql
SELECT incidents.date,
       incidents.category
  FROM `sandbox-bq.join_training.sf_crime_2014` incidents
       INNER JOIN (SELECT date
                     FROM `sandbox-bq.join_training.sf_crime_2014`
                    ORDER BY date
                    LIMIT 5) AS sub
       ON incidents.date = sub.date
```

This can be particularly useful when combined with aggregations. When you join, the requirements for your subquery output aren’t as stringent as when you use the `WHERE` clause. For example, your inner query can output multiple results. 

Imagine you’d like to aggregate all of the companies receiving investment and companies acquired each month. The right way.

```sql
SELECT COALESCE(acquisitions.month, investments.month) AS month,
       acquisitions.companies_acquired,
       investments.companies_rec_investments 
  FROM      
       (
         SELECT acquired_month AS month,
                COUNT(DISTINCT company_permalink) AS companies_acquired
           FROM `sandbox-bq.join_training.crunchbase_acqusitions`
          GROUP BY month
        ) AS acquisitions
         
        FULL JOIN
        
        (
         SELECT funded_month AS month,
                COUNT(DISTINCT company_permalink) AS companies_rec_investments
           FROM `sandbox-bq.join_training.crunchbase_investments` 
          GROUP BY month 
        ) AS investments
        
       ON acquisitions.month = investments.month
 ORDER BY month DESC
```

Write a query that counts the number of companies founded and acquired by quarter starting in Q1 2012. Create the aggregations in two separate queries, then join them.

```sql
SELECT COALESCE(companies.quarter_year, acquisitions.quarter_year) AS quarter_year,
       companies.companies_founded,
       acquisitions.companies_acquired 
  FROM (
        SELECT founded_quarter AS quarter_year,
               COUNT(permalink) AS companies_founded
          FROM `sandbox-bq.join_training.crunchbase_companies`
         WHERE founded_year >= 2012
         GROUP BY quarter_year
       ) AS companies
       
       FULL JOIN
       
       (
        SELECT acquired_quarter AS quarter_year,
               COUNT(DISTINCT company_permalink) AS companies_acquired
          FROM `sandbox-bq.join_training.crunchbase_acqusitions`
         WHERE acquired_year >= 2012
         GROUP BY quarter_year 
       ) AS acquisitions
       
       ON companies.quarter_year = acquisitions.quarter_year
 ORDER BY quarter_year
```

## Window functions

A window function performs a calculation across a set of table rows that are somehow related to the current row. This is comparable to the type of calculation that can be done with an aggregate function. But unlike regular aggregate functions, use of a window function does not cause rows to become grouped into a single output row — the rows retain their separate identities. Behind the scenes, the window function is able to access more than just the current row of the query result.

```sql
SELECT duration_sec,
       SUM(duration_sec) OVER (ORDER BY start_date) AS running_total
  FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
```

You can see that the above query creates an aggregation (`running_total`) without using `GROUP BY`. Let’s break down the syntax and see how it works.

### Basic windowing syntax

The first part of the above aggregation, `SUM(duration_seconds)`, looks a lot like any other aggregation. Adding `OVER` designates it as a window function. You could read the above aggregation as “take the sum of `duration_seconds` _over_ the entire result set, in order by `start_time`.”

If you’d like to narrow the window from the entire dataset to individual groups within the dataset, you can use `PARTITION BY` to do so:

```sql
SELECT start_station_name,
       duration_sec,
       SUM(duration_sec) OVER 
                              (PARTITION BY start_station_name ORDER BY start_date) AS running_total
  FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
 WHERE start_date < '2017-10-20'
```

The above query groups and orders the query by `start_station_name`. Within each value of `start_station_name`, it is ordered by `start_date`, and __the running total sums across the current row and all previous rows of duration_seconds__. Scroll down until the `start_station_name` value changes and you will notice that running_total starts over. That’s what happens when you group using `PARTITION BY`. In case you’re still stumped by `ORDER BY`, it simply orders by the designated column(s) the same way the ORDER BY clause would, except that it treats every partition as separate. It also creates the running total—without ORDER BY, each value will simply be a sum of all the duration_seconds values in its respective start_station_name.

```sql
SELECT start_station_name,
       duration_sec,
       SUM(duration_sec) OVER 
                         (PARTITION BY start_station_name) AS running_total
  FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
 WHERE start_date < '2017-10-20'
```

The `ORDER and PARTITION` define what is referred to as the “window”—the ordered subset of data over which calculations are made.

_Note: You can’t use window functions and standard aggregations in the same query. More specifically, you can’t include window functions in a GROUP BY clause._


Write a query modification of the above example query that shows the duration of each ride as a percentage of the total time accrued by riders from each start_terminal

```sql
SELECT start_station_name,
       duration_sec,
       SUM(duration_sec) OVER 
                         (PARTITION BY start_station_name) AS partition_total,
       ROUND(duration_sec/ SUM(duration_sec) OVER 
                         (PARTITION BY start_station_name) * 100,2) AS pct_of_total                 
  FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
 WHERE start_date < '2017-10-20'
 ORDER BY start_station_name, pct_of_total DESC
```

### SUM, COUNT, and AVG with Window functions

When using window functions, you can apply the same aggregates that you would under normal circumstances— `SUM`, `COUNT`, and `AVG`

```sql
SELECT start_station_name,
       duration_sec,
       SUM(duration_sec) OVER
                         (PARTITION BY start_station_name) AS partition_total,
       SUM(duration_sec) OVER
                         (PARTITION BY start_station_name ORDER BY start_date ) AS running_total,
       COUNT(duration_sec) OVER
                           (PARTITION BY start_station_name) AS partition_count,
       COUNT(duration_sec) OVER
                           (PARTITION BY start_station_name ORDER BY start_date) AS running_count,
       AVG(duration_sec) OVER
                         (PARTITION BY start_station_name) AS partition_avg,
       AVG(duration_sec) OVER
                         (PARTITION BY start_station_name ORDER BY start_date) AS running_avg
  FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
```

### ROW_NUMBER()
`ROW_NUMBER()` does just what it sounds like—displays the number of a given row. It starts are 1 and numbers the rows according to the ORDER BY part of the window statement. `ROW_NUMBER()` does not require you to specify a variable within the parentheses. Using the `PARTITION BY` clause will allow you to begin counting 1 again in each partition.

```sql
SELECT start_station_name,
       duration_sec,
       ROW_NUMBER() OVER
                    (PARTITION BY start_station_name ORDER BY start_date) AS row_num
  FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
```

### RANK() and DENSE_RANK()

`RANK()` is slightly different from `ROW_NUMBER()`. If you order by start_time, for example, it might be the case that some terminals have rides with two identical start times. In this case, they are given the same rank, whereas `ROW_NUMBER()` gives them different numbers.

You can also use `DENSE_RANK()` instead of `RANK()` depending on your application. Imagine a situation in which three entries have the same value. Using either command, they will all get the same rank. For the sake of this example, let’s say it’s “2.” Here’s how the two commands would evaluate the next results differently:

`RANK()` would give the identical rows a rank of 2, then skip ranks 3 and 4, so the next result would be 5
`DENSE_RANK()` would still give all the identical rows a rank of 2, but the following row would be 3—no ranks would be skipped.

```sql
SELECT start_station_name,
       start_date,
       ROW_NUMBER() OVER
                    (PARTITION BY start_station_name ORDER BY start_date) AS row_num,
       RANK() OVER
              (PARTITION BY start_station_name ORDER BY start_date) rank_num,
       DENSE_RANK() OVER
                    (PARTITION BY start_station_name ORDER BY start_date) dense_rank_num
  FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
```

### NTILE
You can use window functions to identify what percentile (or quartile, or any other subdivision) a given row falls into. The syntax is `NTILE(*# of buckets*)`. In this case, `ORDER BY` determines which column to use to determine the quartiles (or whatever number of ‘tiles you specify).\
If you’re working with very small windows, keep this in mind and consider using quartiles or similarly small bands.

```sql
SELECT start_station_name,
       duration_sec,
       NTILE(4) OVER
                (PARTITION BY start_station_name ORDER BY duration_sec) AS quartile,
       NTILE(5) OVER
                (PARTITION BY start_station_name ORDER BY duration_sec) AS quintile,
       NTILE(100) OVER
                (PARTITION BY start_station_name ORDER BY duration_sec) AS percentile
  FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
 WHERE start_date > '2014-08-19'
   AND start_station_name != '10th Ave at E 15th St'
```

### LAG and LEAD
It can often be useful to compare rows to preceding or following rows, especially if you’ve got the data in an order that makes sense. You can use `LAG` or `LEAD` to create columns that pull values from other rows—all you need to do is enter which column to pull from and how many rows away you’d like to do the pull. `LAG` pulls from previous rows and `LEAD` pulls from following rows

```sql
SELECT start_station_name,
       duration_sec,
       LAG(duration_sec, 1) OVER
                            (PARTITION BY start_station_name ORDER BY duration_sec) AS lag_duration_sec,
       LEAD(duration_sec, 1) OVER
                             (PARTITION BY start_station_name ORDER BY duration_sec) AS lead_duration_sec,
       duration_sec - LAG(duration_sec, 1) OVER
                                           (PARTITION BY start_station_name ORDER BY duration_sec) AS lag_diff
  FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
```

### Defining a window alias
If you’re planning to write several window functions in to the same query, using the same window, you can create an alias. The `WINDOW` clause, if included, should always come after the `WHERE` clause.

```sql
SELECT start_station_name,
       duration_sec,
       NTILE(4) OVER ntile_window AS quartile,
       NTILE(5) OVER ntile_window AS quintile,
       NTILE(100) OVER ntile_window AS percentile
  FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
 WHERE start_date > '2014-08-19'
   AND start_station_name != '10th Ave at E 15th St'
WINDOW ntile_window AS (PARTITION BY start_station_name ORDER BY duration_sec)
```


### Pivoting In Big Query

```sql
SELECT teams.conference,
       players.year,
       COUNT(*) AS player_count
  FROM `sandbox-bq.join_training.football_players` AS players
       INNER JOIN `sandbox-bq.join_training.football_teams` AS teams
       ON players.school_name = teams.school_name
 GROUP BY teams.conference, players.year
 ORDER BY 1, 2
```

```sql
WITH  sub AS
            (
             SELECT teams.conference,
                    players.year,
                    COUNT(*) AS player_count
               FROM `sandbox-bq.join_training.football_players` AS players
                    INNER JOIN `sandbox-bq.join_training.football_teams` AS teams
                    ON players.school_name = teams.school_name
              GROUP BY teams.conference, players.year
              ORDER BY 1, 2
              )

SELECT conference,
       SUM(player_count) AS total_players,
       SUM(CASE
           WHEN year = 'FR' THEN player_count
           ELSE NULL END) AS fr,
       SUM(CASE
           WHEN year = 'SO' THEN player_count
           ELSE NULL END) AS so,
       SUM(CASE
           WHEN year = 'JR' THEN player_count
           ELSE NULL END) AS jr,
       SUM(CASE
           WHEN year = 'SR' THEN player_count
           ELSE NULL END) AS sr
  FROM sub
 GROUP BY 1
 ORDER BY 2 DESC
```









































