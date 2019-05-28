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
