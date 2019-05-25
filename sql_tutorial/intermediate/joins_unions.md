# SQL Joins

For this section we are taking a detour and doing the exercises based on the Mode SQL datasets. We will upload them in Big query so the interface remains the same.

#### Steps to upload MODE sql datasets in Big query

## The Anatomy of a Join

```sql
SELECT teams.conference AS conference,
       ROUND(AVG(players.weight),2) AS weight
  FROM `sandbox-bq.join_training.football_teams` AS teams
       JOIN `sandbox-bq.join_training.football_players` AS players
       ON teams.school_name = players.school_name
 GROUP BY teams.conference
 ORDER BY weight DESC 
```
* When performing joins, it’s easiest to give your table names aliases. 
* Once you’ve given a table an alias, you can refer to columns in that table in the `SELECT` clause using the alias name
* After the `FROM` statement, we have two new statements: `JOIN`, which is followed by a table name, and `ON`, which is followed by a couple column names separated by an equals sign.
* `ON` indicates how the two tables (the one after the `FROM` and the one after the `JOIN`) relate to each other. These relationships are sometimes called __mappings.__

Write a query that displays player names, school names and conferences for schools in the "FBS (Division I-A Teams)" division.
```sql
SELECT players.player_name AS name,
       players.school_name AS school_name,
       teams.conference AS conferences
  FROM `sandbox-bq.join_training.football_players` AS players
       INNER JOIN `sandbox-bq.join_training.football_teams` AS teams
       ON players.school_name = teams.school_name
 WHERE teams.division = 'FBS (Division I-A Teams)'
```

## LEFT JOIN
`LEFT JOIN` command tells Big Query to return all rows in the table in the `FROM` clause, regardless of whether or not they have matches in the table in the LEFT JOIN clause

```sql
SELECT companies.permalink AS comapanies_permalink,
       companies.name AS companies_name,
       acquisitions.company_permalink AS acquisition_permalink,
       acquisitions.company_name AS acquisition_name
  FROM `sandbox-bq.join_training.crunchbase_companies` AS companies
       LEFT JOIN `sandbox-bq.join_training.crunchbase_acqusitions` AS acquisitions
       ON companies.permalink = acquisitions.company_permalink
 ORDER BY companies_name
```

Write a query that performs an inner join between the tutorial.crunchbase_acquisitions table and the tutorial.crunchbase_companies table, but instead of listing individual rows, count the number of non-null rows in each table.

```sql
SELECT COUNT(companies.permalink) AS companies_count,
       COUNT(acquisitions.company_permalink) AS acquisitions_count
  FROM `sandbox-bq.join_training.crunchbase_companies` AS companies
       INNER JOIN `sandbox-bq.join_training.crunchbase_acqusitions` AS acquisitions
       ON companies.permalink = acquisitions.company_permalink
```
Modify the query above to be a `LEFT JOIN`. Note the difference in results.

```sql
SELECT COUNT(companies.permalink) AS companies_count,
       COUNT(acquisitions.company_permalink) AS acquisitions_count
  FROM `sandbox-bq.join_training.crunchbase_companies` AS companies
       LEFT JOIN `sandbox-bq.join_training.crunchbase_acqusitions` AS acquisitions
       ON companies.permalink = acquisitions.company_permalink
```

Count the number of unique companies (don't double-count companies) and unique acquired companies by state. Do not include results for which there is no state data, and order by the number of acquired companies from highest to lowest.

```sql
SELECT companies.state_code AS state_code,
       COUNT(DISTINCT companies.permalink) AS unique_companies_count,
       COUNT(DISTINCT acquisitions.company_permalink) AS unique_acquired_companies
  FROM `sandbox-bq.join_training.crunchbase_companies` AS companies
       LEFT JOIN `sandbox-bq.join_training.crunchbase_acqusitions` AS acquisitions
       ON companies.permalink = acquisitions.company_permalink
 WHERE companies.state_code IS NOT NULL
 GROUP BY state_code
 ORDER BY unique_acquired_companies DESC
```

## Filtering on the `ON` clause
Normally, filtering is processed in the `WHERE` clause once the two tables have already been joined. It’s possible, though that you might want to filter one or both of the tables _before_ joining them. For example, you only want to create matches between the tables under certain circumstances.

```sql
SELECT companies.permalink AS companies_permalink,
       companies.name AS comapanies_name,
       acquisitions.company_permalink AS acquisition_permalink,
       acquisitions.acquired_at AS acquired_date
  FROM `sandbox-bq.join_training.crunchbase_companies` AS companies
       LEFT JOIN `sandbox-bq.join_training.crunchbase_acqusitions` AS acquisitions
       ON companies.permalink = acquisitions.company_permalink
       AND acquisitions.company_permalink != '/company/1000memories'
 ORDER BY companies_permalink
```

What’s happening above is that the conditional statement `AND`... is evaluated before the join occurs. You can think of it as a `WHERE` clause that only applies to one of the tables. 

### Filtering in the WHERE clause
If you move the same filter to the `WHERE` clause, you will notice that the filter happens after the tables are joined. The result is that the 1000memories row is joined onto the original table, but then it is filtered out entirely (in both tables) in the `WHERE` clause before displaying results.

```sql
SELECT companies.permalink AS companies_permalink,
       companies.name AS comapanies_name,
       acquisitions.company_permalink AS acquisition_permalink,
       acquisitions.acquired_at AS acquired_date
  FROM `sandbox-bq.join_training.crunchbase_companies` AS companies
       LEFT JOIN `sandbox-bq.join_training.crunchbase_acqusitions` AS acquisitions
       ON companies.permalink = acquisitions.company_permalink
 WHERE acquisitions.company_permalink != '/company/1000memories'
 ORDER BY companies_permalink
```

Write a query that shows a company's name, "status" (found in the Companies table), and the number of unique investors in that company. Order by the number of investors from most to fewest. Limit to only companies in the state of New York.

```sql
SELECT companies.name AS companies_name,
       companies.status AS companies_status,
       COUNT(DISTINCT investments.investor_permalink) AS unique_investors
  FROM `sandbox-bq.join_training.crunchbase_companies` AS companies
       LEFT JOIN `sandbox-bq.join_training.crunchbase_investments` AS investments
       ON companies.permalink = investments.company_permalink
          AND companies.state_code = 'NY'
 GROUP BY companies_name, companies_status
 ORDER BY unique_investors DESC 
```

Write a query that lists investors based on the number of companies in which they are invested. Include a row for companies with no investor, and order from most companies to least.

```sql
SELECT CASE 
       WHEN investments.investor_name IS NULL THEN 'No Investor'
       ELSE investments.investor_name END AS investor_name,
       COUNT(DISTINCT companies.permalink) AS unique_companies
  FROM `sandbox-bq.join_training.crunchbase_companies` AS companies
       LEFT JOIN `sandbox-bq.join_training.crunchbase_investments` AS investments
       ON companies.permalink = investments.company_permalink
 GROUP BY investor_name
 ORDER BY unique_companies DESC
```

## Full Outer Join
`FULL JOIN` returns unmatched rows from both tables. It is commonly used in conjunction with aggregations to understand the amount of overlap between two tables.

```sql
SELECT COUNT(CASE
             WHEN companies.permalink IS NOT NULL AND acquisitions.company_permalink IS NULL THEN companies.permalink
             ELSE NULL END) AS companies_only,
       COUNT(CASE
             WHEN acquisitions.company_permalink IS NOT NULL AND companies.permalink IS NULL THEN acquisitions.company_permalink
             ELSE NULL END) AS acquisitions_only,
       COUNT(CASE
             WHEN companies.permalink IS NOT NULL AND acquisitions.company_permalink IS NOT NULL THEN companies.permalink
             ELSE NULL END) AS both_tables
  FROM `sandbox-bq.join_training.crunchbase_companies` AS companies
       FULL JOIN `sandbox-bq.join_training.crunchbase_acqusitions` AS acquisitions
       ON companies.permalink = acquisitions.company_permalink 
```

## SQL Unions
SQL joins allow you to combine two datasets side-by-side, but `UNION` allows you to stack one dataset on top of the other. Put differently, `UNION` allows you to write two separate `SELECT` statements, and to have the results of one statement display in the same table as the results from the other statement.

Note that `UNION` only appends distinct values. More specifically, when you use `UNION`, the dataset is appended, and any rows in the appended table that are exactly identical to rows in the first table are dropped. If you’d like to append all the values from the second table, use `UNION ALL`. You’ll likely use `UNION ALL` far more often than `UNION`. 

SQL has strict rules for appending data:

1. Both tables must have the same number of columns
2. The columns must have the same data types in the same order as the first table

## Using comparison operators with joins

```sql
-- Comparison operators with Join
SELECT companies.permalink AS companies_permalink,
       companies.name AS companies_name,
       companies.status AS companies_status,
       COUNT(investments.investor_permalink) AS investors,
       COUNT(DISTINCT investments.investor_permalink) AS unique_investors
  FROM `sandbox-bq.join_training.crunchbase_companies` AS companies
       LEFT JOIN `sandbox-bq.join_training.crunchbase_investments` AS investments
       ON companies.permalink = investments.company_permalink
          AND investments.funded_year > companies.founded_year + 5
 GROUP BY 1,2,3
 ORDER BY investors DESC
```
This technique is especially useful for creating date ranges as shown above. It’s important to note that this produces a different result than the following query because it only joins rows that fit the investments.funded_year > companies.founded_year + 5 condition rather than joining all rows and then filtering:


## Self Joins

```sql
--SELF Joins
SELECT jp_inv.company_name AS company_name
  FROM `sandbox-bq.join_training.crunchbase_investments` AS jp_inv
       INNER JOIN `sandbox-bq.join_training.crunchbase_investments` AS gb_inv
       ON jp_inv.company_name = gb_inv.company_name
          AND gb_inv.investor_country_code = 'GBR'
          AND gb_inv.funded_at > jp_inv.funded_at
 WHERE jp_inv.investor_country_code = 'JPN'
 ORDER BY 1
```





