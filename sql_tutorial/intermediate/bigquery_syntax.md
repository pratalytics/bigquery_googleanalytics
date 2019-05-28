# Big Query Syntax

## Querying multiple tables using a wildcard table
Wildcard tables enable you to query multiple tables using concise SQL statements. A wildcard table represents a union of all the tables that match the wildcard expression. For example, the following `FROM` clause uses the wildcard expression `ga_sessions_201707*` to match all tables in the `google_analytics_sample` dataset that begin with the string ga_sessions_201707.

```sql
*
Query Data from 2017-07-01 to 2017-07-10
*/
SELECT date,
       SUM(totals.visits) AS sessions
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
 WHERE _TABLE_SUFFIX BETWEEN '01' AND '10'
 GROUP BY date
 ORDER BY date
```

### Limitations

* The wildcard table functionality does not support views. If the wildcard table matches any view in the dataset, the query returns an error. This is true whether or not your query contains a `WHERE` clause on the `_TABLE_SUFFIX` pseudo column to filter out the view.
* Currently, cached results are not supported for queries against multiple tables using a wildcard even if the Use Cached Results option is checked. If you run the same wildcard query multiple times, you are billed for each query.
* Wildcard tables support native BigQuery storage only. You cannot use wildcards when querying an external table or a view.
Queries that contain Data Manipulation Language (DML) statements cannot use a wildcard table as the target of the query. For example, a wildcard table may be used in the FROM clause of an UPDATE query, but a wildcard table cannot be used as the target of the UPDATE operation.

## Unnesting and Repeating Fields
