# bigquery_googleanalytics
A lot of digital analyst have worked on Google analytics or Adobe Analytics but they don't have enough experience with SQL or Big Query. I faced the same issue and while there were enough tutorial online, I didnt find one which gave instructions on how to effectively use SQL for the GA 360 dataset. This is my attempt to create this tutorial.

We go from knowing nothing about SQL to writing advanced scripts for finding user cohorts and purchase behavior.

## All exercises
#### Selecting columns

You should know- Basic SQL clauses and reading the GA 360 schema

* Select columns which has visitor ID, the visit number, session ID, session start time and date
* Select any two columns from the following records - totals, trafficSource, device, geoNetwork. Can you select columns from hits.eCommerceAction? What error do you get?

#### Filtering and Ordering

You should know- Conditional and logical operators, ordering columns and using table suffixes

* Write a query to get the name of countries that contains "United" in the month of Jan 2017
* Write a query to get the names of all the cities except for the following - Mumbai, Hong Kong, San Francisco and London for the dates of 15th Sept 2016, 29th Oct 2016 and 7th dec 2016.
* Write a query where the referral path is not null between the dates may 10th 2017 to may 24th 2017. 
* Write a query to get session ids where the time on site is more than 50 seconds but less than or equal to 500 seconds in the year 2016. Sort it in descending order. 
* Write a query where the channel is Social and total hits in a session is greater than 5 or the device is mobile and it is a returning user for the date 19th Feb 2017
* Write a query where total.visits is greater than 1 across all the tables. What is the output and what is the reason for it?

#### Aggregating and Grouping

You should know - Aggregation functions like `SUM`, `COUNT`, `DISTINCT` etc and using `GROUP BY`

* 1. Write a query to find the total users, new users, sessions, pageviews, revenue and transactions for the period of Jan'17 to March'17. Are the revenue numbers correct or do we need to treat it further?
  2. Group the above query by channel groupping, device category and source
  3. Modify the grouped query to filter for desktop and channel grouping containing either organic search or direct. Filter it further to include only those rows that have transactions and new users is greater than 1000. Order it by transactions in descending order
* Write a query to find unique values in the medium column

#### Conditional Functions

You should know - `CASE`, `IF`, `IFNULL` etc. How to use these functions with aggregate functions.

* Write a query to replace `NULL` values in the referral path column
* Write a query to find total sessions across device category for the year 2017. Include a column that replaces the values in the device category column with Computer, Phone and Tab for desktop, mobile and tablet respectively
* Write a query to get total users, revenue and transactions by channel for the year of 2017. Add an additional column which categorises the rows based on the following conditions
  1. If the users are greater than 58000 and the revenue is greater than 128600 and the transations is greater than 800 then categorise it as 'High Engagemnt - High Revenue'
  2. After the above condition is met then if only user are greater than 58000 then categorise is as 'High Engagement'
  3. Similarly if only the revenue is greater than 120000 then categorise it as 'High Revenue'


#### Working with Date and Time

You should know- differences between data types like `DATETIME`, `DATE`, `TIME` AND `TIMESTAMP`. Functions to format and parse dates, findind time and date intervals

* Write a query to get total pageviews and sessions for all the tables. Group them by year, quarter and month based on the values of the visitStartTime column. (Don't include the visitStartTime column in the output). Ensure the time zone is America Los Angeles.
* Write a query to get total users and transactions for the month of June 2017. Group them by week, day of the week, and hour based on the values of the date column. (Don't include the date column in the output). Ensure the time zone is America Los Angeles.
* Write a query to get total sessions for 23rd May 2017. Also get total sessions for the date 2 months prior to 23rd as well as get total sessions for the date which is 3 weeks after 23rd.

#### Working with Repeated fields - UNNEST

You should know - using unnest to flatten repeated fields, concept of joins in SQL

* Write a query to get page path, title and hostname along with pageviews and entrances for the period of April 2017
* Write a query to get event category and event action along with total and unique events for the period of Nov 2016
* Write a query to product, revenue and quantity for the period Jan-Mar 2017

#### Sub query

You should know - Using sub query in the `FROM` clause, in the `WHERE` clause and in the `SELECT` clause. Use of `WITH` clause

* Write a query to get sessions, pageviews, transactions, users, users/session, pages/session, bounce rate and ecommerce rate grouped by channel grouping for the period of May'17 to Jun'17



































