# REVENUE-ANALYSIS-PROJECT
## I. Introduction

## II. Dataset

## III. Analysis
### **Query 01: Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)**
```sql
SELECT
 EXTRACT(month FROM PARSE_DATE('%Y%m%d', date)) AS month,
 SUM(totals.visits) AS visits,
 SUM(totals.pageviews) AS pageviews,
 SUM(totals.transactions) AS transactions 
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY month
ORDER BY month;
```
| month  | visits | pageviews | transactions |
|--------|--------|-----------|-------------|
| 201701 | 64694  | 257708    | 713         |
| 201702 | 62192  | 233373    | 733         |
| 201703 | 69931  | 259522    | 993         |

**Key insights:**
The data from Q1 2017 shows relatively stable website traffic, with visits and pageviews maintaining a consistent trend across January, February, and March. However, March stands out with a notable increase in transactions (993) compared to the previous months (713 in January and 733 in February).

This spike in transactions suggests an improvement in conversion rates, potentially driven by seasonal trends, marketing efforts, or enhanced user engagement. Further analysis could explore factors contributing to this increase, such as campaign performance, user behavior, or external events influencing purchasing decisions.

### **Query 02: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)**
```sql
SELECT
 trafficSource.source,
 SUM(totals.visits) AS total_visits,  
 SUM(totals.bounces) AS total_no_of_bounces,
 SUM(totals.bounces)/SUM(totals.visits)*100 AS bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
GROUP BY trafficSource.source
ORDER BY total_visits DESC;
```
| Row | source                 | total_visits | total_no_of_bounces | bounce_rate |
|-----|------------------------|-------------|----------------------|----------------|
| 1   | google                 | 38,400      | 19,798               | 51.56          |
| 2   | (direct)               | 19,891      | 8,606                | 43.27          |
| 3   | youtube.com            | 6,351       | 4,238                | 66.73          |
| 4   | analytics.google.com   | 1,972       | 1,064                | 53.96          |
| 5   | Partners               | 1,788       | 936                  | 52.35          |
| 6   | m.facebook.com         | 669         | 430                  | 64.28          |
| 7   | google.com             | 368         | 183                  | 49.73          |
| 8   | dfa                    | 302         | 124                  | 41.06          |
| 9   | sites.google.com       | 230         | 97                   | 42.17          |
| 10  | facebook.com           | 191         | 102                  | 53.40          |

**Key insights:**
- **Google** is the top traffic source with **38,400 visits**, but it also has a relatively high bounce rate of **51.56%**.
- **Direct traffic** contributes significantly (**19,891 visits**) with a lower bounce rate (**43.27%**), indicating engaged users.
- **YouTube referrals** have the highest bounce rate (**66.73%**), suggesting visitors from YouTube might be less engaged.
- **Facebook (m.facebook.com & facebook.com)** shows mixed performance, with **moderate visit counts** but a bounce rate above **50%**.

### **Query 3: Revenue by traffic source by week, by month in June 2017**
```sql
WITH month_data AS (
  SELECT 
    'Month' AS time_type,
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS time,
    trafficSource.source AS source,
    FORMAT('%.4f',SUM(product.productRevenue)/1000000) AS revenue
  FROM 
    `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    UNNEST(hits) AS hit,
    UNNEST(hit.product) AS product
  WHERE 
    product.productRevenue IS NOT NULL
  GROUP BY 
    time, 
    source
),

week_data AS (
  SELECT 
    'Week' AS time_type,
    FORMAT_DATE('%Y%W', PARSE_DATE('%Y%m%d', date)) AS time,
    trafficSource.source AS source,
    FORMAT('%.4f',SUM(product.productRevenue)/1000000) AS revenue
  FROM 
    `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    UNNEST(hits) AS hits,
    UNNEST(hits.product) AS product 
  WHERE 
    product.productRevenue IS NOT NULL
  GROUP BY 
    time, 
    source)

SELECT * FROM month_data
UNION ALL
SELECT * FROM week_data
ORDER BY source, time;
```
| Row | time_type | time   | source           | revenue  |
|-----|----------|--------|------------------|-------------|
| 1   | Month    | 201706 | (direct)         | 97,333.62   |
| 2   | Week     | 201722 | (direct)         | 6,888.90    |
| 3   | Week     | 201723 | (direct)         | 17,325.68   |
| 4   | Week     | 201724 | (direct)         | 30,908.91   |
| 5   | Week     | 201725 | (direct)         | 27,295.32   |
| 6   | Week     | 201726 | (direct)         | 14,914.81   |
| 7   | Month    | 201706 | bing             | 13.98       |
| 8   | Week     | 201724 | bing             | 13.98       |
| 9   | Month    | 201706 | chat.google.com  | 74.03       |
| 10  | Week     | 201723 | chat.google.com  | 74.03       |

**Key insights:**
- **Direct traffic** dominates revenue generation, particularly for **June 2017 ($97,333.62)**.
- **Weekly fluctuations** in revenue from direct traffic are observed, with a peak in **Week 24 ($30,908.91)**.
- **Bing contributes minimally** to revenue, generating only **$13.98** in both monthly and weekly records.
- **chat.google.com has a small revenue share**, consistently generating **$74.03** in the recorded periods.







