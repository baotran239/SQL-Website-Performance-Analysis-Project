## I. Introduction
This project leverages Google BigQuery to analyze customer purchase behavior on an e-commerce platform. By applying advanced SQL techniques such as CTEs, Aggregate Functions, and Table Functions, it processes key metrics like page views, transactions, and bounce rates, etc. to uncover trends and insights. The findings help businesses optimize strategies, enhance user experience, and drive conversions through data-driven decisions.

## II. Dataset
This is a dataset available on Google BigQuery named "ga_session".

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

The data from Q1 2017 shows relatively stable website traffic, with visits and pageviews maintaining a consistent trend across January, February, and March. However, March stands out with a notable increase in transactions (993) compared to the previous months (713 in January and 733 in February).

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
ORDER BY CAST(revenue AS FLOAT64) DESC;
```
| Row | time_type |   time   |   source   |   revenue   |
|-----|----------|----------|------------|-------------|
|  1  |  Month   |  201706  |  (direct)  |  97,333.62  |
|  2  |  Week    |  201724  |  (direct)  |  30,908.91  |
|  3  |  Week    |  201725  |  (direct)  |  27,295.32  |
|  4  |  Month   |  201706  |   google   |  18,757.18  |
|  5  |  Week    |  201723  |  (direct)  |  17,325.68  |
|  6  |  Week    |  201726  |  (direct)  |  14,914.81  |
|  7  |  Week    |  201724  |   google   |   9,217.17  |
|  8  |  Month   |  201706  |    dfa     |   8,862.23  |
|  9  |  Week    |  201722  |  (direct)  |   6,888.90  |
| 10  |  Week    |  201726  |   google   |   5,330.57  |

Both monthly and weekly data show that **direct traffic** drives the highest revenue, followed by **Google**, with other sources contributing less.

### **Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017**
```sql
WITH 
purchaser_data AS(
  SELECT
      FORMAT_DATE("%Y%m",PARSE_DATE("%Y%m%d",DATE)) AS month,
      (SUM(totals.pageviews)/COUNT(DISTINCT fullvisitorid)) AS avg_pageviews_purchase,
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
    ,UNNEST(hits) hits
    ,UNNEST(product) product
  WHERE _table_suffix BETWEEN '0601' AND '0731'
  AND totals.transactions>=1
  AND product.productRevenue IS NOT NULL
  GROUP BY month
),

non_purchaser_data AS(
  SELECT
      FORMAT_DATE("%Y%m",PARSE_DATE("%Y%m%d",DATE)) AS month,
      SUM(totals.pageviews)/COUNT(DISTINCT fullvisitorid) AS avg_pageviews_non_purchase,
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
      ,UNNEST(hits) hits
    ,UNNEST(product) product
  WHERE _table_suffix BETWEEN '0601' AND '0731'
  AND totals.transactions IS NULL
  AND product.productRevenue IS NULL
  GROUP BY month
)

SELECT
    pd.*,
    avg_pageviews_non_purchase
FROM purchaser_data pd
FULL JOIN non_purchaser_data USING(month)
ORDER BY pd.month;
```
| Row |  month  | avg_pageviews_purch | avg_pageviews_nonpurch |
|-----|---------|---------------------|------------------------|
|  1  | 201706  | 94.02               | 316.87                 |
|  2  | 201707  | 124.24              | 334.06                 |

Non-purchasing users have significantly higher average pageviews compared to purchasing users, but both groups show an increasing trend in July 2017.

### **Query 05: Average number of transactions per user that made a purchase in July 2017**
```sql
SELECT
    FORMAT_DATE("%Y%m",PARSE_DATE("%Y%m%d",DATE)) AS month,
    SUM(totals.transactions)/COUNT(DISTINCT fullvisitorid) AS Avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    ,UNNEST (hits) hits,
    UNNEST(product) product
WHERE  totals.transactions>=1
AND product.productRevenue is not null
GROUP BY month;
```
| Row | Month  | Avg_total_transaction |
|-----|--------|----------------------|
| 1   | 201707 | 4.1639               |

In July 2017, the average total transactions per user were 4.16, indicating a solid level of engagement in e-commerce insudtry context.

### **Query 06: Average amount of money spent per session. Only include purchaser data in July 2017**
```sql
SELECT
 --EXTRACT(month FROM PARSE_DATE('%Y%m%d', date)) AS month,
 format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
 SUM(productRevenue)/SUM(totals.visits)/1000000 AS Avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST(hits) AS hits,
UNNEST(hits.product) AS product
WHERE totals.transactions IS NOT NULL AND product.productRevenue IS NOT NULL
GROUP BY month;
```
| Row |  month  | avg_total_transaction |
|-----|---------|-----------------------|
|  1  | 201707  | 43.86                 |

In July 2017, the average total transactions per user reached 43.86, reflecting a high level of user engagement in e-commerce insudtry context. This significant number suggests that users were highly active.

### **Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.**
```sql
with buyer_list as(
    SELECT
        distinct fullVisitorId
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    , UNNEST(hits) AS hits
    , UNNEST(hits.product) as product
    WHERE product.v2ProductName = "YouTube Men's Vintage Henley"
    AND totals.transactions>=1
    AND product.productRevenue is not null
)

SELECT
  product.v2ProductName AS other_purchased_products,
  SUM(product.productQuantity) AS quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
, UNNEST(hits) AS hits
, UNNEST(hits.product) as product
JOIN buyer_list using(fullVisitorId)
WHERE product.v2ProductName != "YouTube Men's Vintage Henley"
 and product.productRevenue is not null
GROUP BY other_purchased_products
ORDER BY quantity DESC;
```
| Row | other_purchased_products            | quantity |
|-----|-------------------------------------|----------|
|  1  | Google Sunglasses                  | 20       |
|  2  | Google Women's Vintage Hero        | 7        |
|  3  | SPF-15 Slim & Slender Lip Balm     | 6        |
|  4  | Google Women's Short Sleeve        | 4        |
|  5  | YouTube Men's Fleece Hoodie        | 3        |
|  6  | Google Men's Short Sleeve Bad...

The most frequently purchased additional product was Google Sunglasses (20 orders), followed by Google Women's Vintage Hero (7 orders) and SPF-15 Slim & Slender Lip Balm (6 orders). This insight can help in cross-selling strategies and product recommendations.

### **Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. For example, 100% product view then 40% add_to_cart and 10% purchase**
```sql
WITH product_data AS(
SELECT
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d',DATE)) AS month,
    COUNT(CASE WHEN eCommerceAction.action_type = '2' THEN product.v2ProductName END) AS num_product_view,
    COUNT(CASE WHEN eCommerceAction.action_type = '3' THEN product.v2ProductName END) AS num_add_to_cart,
    COUNT(CASE WHEN eCommerceAction.action_type = '6' AND product.productRevenue IS NOT NULL THEN product.v2ProductName END) AS num_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
,UNNEST(hits) AS hits
,UNNEST (hits.product) AS product
WHERE _table_suffix BETWEEN '20170101' AND '20170331'
AND eCommerceAction.action_type in ('2','3','6')
GROUP BY month
ORDER BY month
)

SELECT
    *,
    ROUND(num_add_to_cart/num_product_view * 100, 2) AS add_to_cart_rate,
    ROUND(num_purchase/num_product_view * 100, 2) AS purchase_rate
FROM product_data;
```
| Row |  month  | num_product_view | num_add_to_cart | num_purchase | add_to_cart_rate | purchase_rate |
|-----|---------|-----------------|----------------|--------------|------------------|--------------|
|  1  | 201701  | 25787           | 7342           | 2143         | 28.47            | 8.31         |
|  2  | 201702  | 21489           | 7360           | 2060         | 34.25            | 9.59         |
|  3  | 201703  | 23549           | 8782           | 2977         | 37.29            | 12.64        |

- The add-to-cart rate shows a steady increase from 28.47% in January 2017 to 37.29% in March 2017, indicating an improvement in customer interest in purchasing products.
- The purchase rate also rises from 8.31% to 12.64%, suggesting better conversion efficiency.
- The highest engagement and purchase activity are observed in March 2017, showing an increasing trend in customer purchasing behavior over time.

## IV. Conclusions
