# REVENUE-ANALYSIS-PROJECT
## I. Introduction

## II. Dataset

## III. Analysis
1. Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
SELECT
 EXTRACT(month FROM PARSE_DATE('%Y%m%d', date)) AS month,
 SUM(totals.visits) AS visits,
 SUM(totals.pageviews) AS pageviews,
 SUM(totals.transactions) AS transactions 
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY month
ORDER BY month;   
