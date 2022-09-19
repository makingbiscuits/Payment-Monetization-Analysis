# Payments/Monetization-Analysis


✨ ✨ Solution for Payments/Monetization Analyst task for Turing College✨ ✨ 

The PowerPoint slides. It explains the main insights of the analysis.

:rocket: SQL (BigQuery), Google Sheets, PowerPoint

### The task:

Your manager is very happy with the data you provided him the week before, but now he is interested in how you can handle analysis on your own. He does not give you concrete questions to be answered, just asks you to use any data in the company databases and come up with a useful analysis on your own. It can be from different aspects of the business, but he expects you to provide him with at least 3-5 insights about business.

You should use tables from olist_db to come up with an analysis on your own. You will have to write SQL queries to extract needed data, then analyze and visualize this data either with Google Sheets or other data visualization tools. After all this, you will have to come up with a presentation oriented to executives from your company. Have in mind your audience for the presentation, these will be business oriented people who are not data analysts and may not have studied statistics before.

- Write SQL queries to extract needed data, these queries should be well documented, with indentation and provided in separate file
- Provide your analysis in Google Sheets or any other data visualization tool, have in mind that this file should contain all the things you tried to look at even if they did not bring any useful insights
- Provide presentation with around 10 slides which would be business focused and would present only the most interesting 3-5 insights
- See whether you can apply at least 2 techniques learned in this or previous sprints
- Explore the data. See whether there are interesting data points that can give more insights to your presentation.
- Provide analytical insights, what are the drawbacks of this analysis, what further analysis could you recommend?

### Main SQL Queries:

PARETO CHARTS:
``` SQL 
--pareto chart for products-revenue
WITH category_sales AS 
(
 SELECT
   string_field_1 AS product_category_name,
   SUM(price + freight_value) AS revenue
 FROM `tc-da-1.olist_db.olist_order_items_dataset` items
 INNER JOIN `tc-da-1.olist_db.olist_products_dataset` AS products
 ON items.product_id = products.product_id
 INNER JOIN `tc-da-1.olist_db.product_category_name_translation` AS translations
 ON products.product_category_name = translations.string_field_0
 GROUP BY 1
 ORDER BY revenue DESC
),

t2 AS
(
   SELECT
   product_category_name,
   revenue,
   SUM(revenue) OVER (ORDER BY revenue DESC) AS running_total,
   SUM(revenue) OVER() AS total
 FROM
   category_sales
)

SELECT
 product_category_name,
 revenue,
 running_total / total AS percent_of_total
FROM t2
ORDER BY revenue DESC;



--pareto chart for seller states-revenue
WITH seller_states_sales AS 
(
 SELECT
   seller_state,
   SUM(price + freight_value) AS revenue
 FROM `tc-da-1.olist_db.olist_order_items_dataset` items
 INNER JOIN `tc-da-1.olist_db.olist_sellers_dataset` sellers
 ON items.seller_id = sellers.seller_id
 GROUP BY 1
 ORDER BY revenue DESC
),

t2 AS
(
   SELECT
   seller_state,
   revenue,
   SUM(revenue) OVER (ORDER BY revenue DESC) AS running_total,
   SUM(revenue) OVER() AS total
 FROM
   seller_states_sales
)

SELECT
 seller_state,
 revenue,
 running_total / total AS percent_of_total
FROM t2
ORDER BY revenue DESC;



--pareto chart for customer states-revenue
WITH customer_states_sales AS 
(
 SELECT
   customer_state,
   SUM(price + freight_value) AS revenue
 FROM `tc-da-1.olist_db.olist_order_items_dataset` AS items
 INNER JOIN `tc-da-1.olist_db.olist_orders_dataset` AS orders 
 ON items.order_id = orders.order_id
 INNER JOIN `tc-da-1.olist_db.olist_customesr_dataset` AS customers
 ON orders.customer_id = customers.customer_id
 GROUP BY 1
 ORDER BY revenue DESC
),

t2 AS
(
   SELECT
   customer_state,
   revenue,
   SUM(revenue) OVER (ORDER BY revenue DESC) AS running_total,
   SUM(revenue) OVER() AS total
 FROM
   customer_states_sales
)

SELECT
 customer_state,
 revenue,
 running_total / total AS percent_of_total
FROM t2
ORDER BY revenue DESC;
```

AVERAGE REVIEW SCORE:
``` SQL 
 -- average review score for all states/customers: 4.04
 
 SELECT
   AVG(review_score) AS avg_review_score
 FROM `tc-da-1.olist_db.olist_order_items_dataset` AS items
 INNER JOIN `tc-da-1.olist_db.olist_orders_dataset` AS orders 
 ON items.order_id = orders.order_id
 INNER JOIN `tc-da-1.olist_db.olist_customesr_dataset` AS customers
 ON orders.customer_id = customers.customer_id
 INNER JOIN `tc-da-1.olist_db.olist_order_reviews_dataset` AS reviews
 ON orders.order_id = reviews.order_id
 INNER JOIN `tc-da-1.olist_db.olist_products_dataset` AS products
 ON items.product_id = products.product_id
 INNER JOIN `tc-da-1.olist_db.product_category_name_translation` AS translations
 ON products.product_category_name = translations.string_field_0
 ORDER BY 1 DESC
 ```

REVIEW SCORE, REVENUE, FOR RIO, BAHIA, OR ALL STATES:
``` SQL
 -- all customer states, their revenue, avg review and their count

SELECT
  customer_state,
  SUM(price + freight_value) AS revenue,
  COUNT(review_score) AS review_count,
  AVG(review_score) AS avg_review_score
FROM `tc-da-1.olist_db.olist_order_items_dataset` AS items
INNER JOIN `tc-da-1.olist_db.olist_orders_dataset` AS orders 
ON items.order_id = orders.order_id
INNER JOIN `tc-da-1.olist_db.olist_customesr_dataset` AS customers
ON orders.customer_id = customers.customer_id
INNER JOIN `tc-da-1.olist_db.olist_order_reviews_dataset` AS reviews
ON orders.order_id = reviews.order_id
GROUP BY 1
ORDER BY revenue DESC

  --products, review, two states
 
SELECT
    string_field_1 AS product_name,
   SUM(price + freight_value) AS revenue,
   COUNT(review_score) AS review_count,
   AVG(review_score) AS avg_review_score
FROM `tc-da-1.olist_db.olist_order_items_dataset` AS items
INNER JOIN `tc-da-1.olist_db.olist_orders_dataset` AS orders 
ON items.order_id = orders.order_id
INNER JOIN `tc-da-1.olist_db.olist_customesr_dataset` AS customers
ON orders.customer_id = customers.customer_id
INNER JOIN `tc-da-1.olist_db.olist_order_reviews_dataset` AS reviews
ON orders.order_id = reviews.order_id
INNER JOIN `tc-da-1.olist_db.olist_products_dataset` AS products
ON items.product_id = products.product_id
INNER JOIN `tc-da-1.olist_db.product_category_name_translation` AS translations
ON products.product_category_name = translations.string_field_0
WHERE 
customer_state = 'RJ' 
  -- OR customer_state = 'BA'
GROUP BY 1
ORDER BY 4 DESC

--all states, prodcuts, review 
 
SELECT
  string_field_1 AS prodcut_name,
  SUM(price + freight_value) AS revenue,
  COUNT(review_score) AS review_count,
  AVG(review_score) AS avg_review_score
FROM `tc-da-1.olist_db.olist_order_items_dataset` AS items
INNER JOIN `tc-da-1.olist_db.olist_orders_dataset` AS orders 
ON items.order_id = orders.order_id
INNER JOIN `tc-da-1.olist_db.olist_customesr_dataset` AS customers
ON orders.customer_id = customers.customer_id
INNER JOIN `tc-da-1.olist_db.olist_order_reviews_dataset` AS reviews
ON orders.order_id = reviews.order_id
INNER JOIN `tc-da-1.olist_db.olist_products_dataset` AS products
ON items.product_id = products.product_id
INNER JOIN `tc-da-1.olist_db.product_category_name_translation` AS translations
ON products.product_category_name = translations.string_field_0
GROUP BY 1
ORDER BY 4 DESC
```

AVERAGE revenue, review in RIO:
``` SQL
SELECT
  string_field_1 AS prodcut_name,
  AVG(price + freight_value) AS avg_revenue,
  COUNT(review_score) AS review_count,
  AVG(review_score) AS avg_review_score
FROM `tc-da-1.olist_db.olist_order_items_dataset` AS items
INNER JOIN `tc-da-1.olist_db.olist_orders_dataset` AS orders 
ON items.order_id = orders.order_id
INNER JOIN `tc-da-1.olist_db.olist_customesr_dataset` AS customers
ON orders.customer_id = customers.customer_id
INNER JOIN `tc-da-1.olist_db.olist_order_reviews_dataset` AS reviews
ON orders.order_id = reviews.order_id
INNER JOIN `tc-da-1.olist_db.olist_products_dataset` AS products
ON items.product_id = products.product_id
INNER JOIN `tc-da-1.olist_db.product_category_name_translation` AS translations
ON products.product_category_name = translations.string_field_0
WHERE 
customer_state = 'RJ' 
GROUP BY 1
ORDER BY 4 DESC
```

CUSTOMER SEGMENTATION in RIO:
``` SQL

WITH t1 AS
 (
    SELECT  
    customers.customer_id AS customer_id,
    MAX(order_purchase_timestamp) AS last_purchase_date,
    COUNT( orders.order_id) AS frequency,
    SUM(price) AS monetary,
    AVG(review_score) AS review_score, 
    FROM `tc-da-1.olist_db.olist_order_items_dataset` AS items
    INNER JOIN `tc-da-1.olist_db.olist_orders_dataset` AS orders 
    ON items.order_id = orders.order_id
    INNER JOIN `tc-da-1.olist_db.olist_customesr_dataset` AS customers
    ON orders.customer_id = customers.customer_id
    INNER JOIN `tc-da-1.olist_db.olist_order_reviews_dataset` AS reviews
    ON orders.order_id = reviews.order_id
    INNER JOIN `tc-da-1.olist_db.olist_products_dataset` AS products
    ON items.product_id = products.product_id
    INNER JOIN `tc-da-1.olist_db.product_category_name_translation` AS translations
    ON products.product_category_name = translations.string_field_0
    WHERE customer_state = 'RJ'
    GROUP BY 1
),

t2 AS 
(
    SELECT MAX(last_purchase_date) AS reference_date
    FROM t1
),

RFM AS 
(
  SELECT t1.customer_id,
    DATE_DIFF(reference_date, t1.last_purchase_date, DAY) + 1 AS recency,
    t1.frequency AS frequency,
    t1.monetary AS monetary,
    review_score
    FROM t1, t2
),

percentiles AS 
(
  SELECT 
    a.*,
    r.percentiles[offset(25)] AS recency_25, 
    r.percentiles[offset(50)] AS recency_50,
    r.percentiles[offset(75)] AS recency_75,   
    m.percentiles[offset(25)] AS monetary_25, 
    m.percentiles[offset(50)] AS monetary_50,
    m.percentiles[offset(75)] AS monetary_75,     
    f.percentiles[offset(25)] AS frequency_25, 
    f.percentiles[offset(50)] AS frequency_50,
    f.percentiles[offset(75)] AS frequency_75, 
FROM RFM AS a,
    (SELECT APPROX_QUANTILES(recency, 100) percentiles 
    FROM RFM) AS r,
    (SELECT APPROX_QUANTILES(monetary, 100) percentiles 
    FROM RFM) AS m,
    (SELECT APPROX_QUANTILES(frequency, 100) percentiles 
    FROM RFM) AS f
),

RFM_scores AS 
(
    SELECT *
FROM (
    SELECT *,
        CASE
            WHEN recency <= recency_25 THEN 4
            WHEN recency <= recency_50 AND recency > recency_25 THEN 3
            WHEN recency <= recency_75 AND recency > recency_50 THEN 2
            WHEN recency > recency_75 THEN 1
        END AS r_score,
        CASE
            WHEN frequency <= frequency_25 THEN 1
            WHEN frequency <= frequency_50 AND frequency > frequency_25 THEN 2
            WHEN frequency <= frequency_75 AND frequency > frequency_50 THEN 3
            WHEN frequency > frequency_75 THEN 4
        END AS f_score,
                CASE
            WHEN monetary <= monetary_25 THEN 1
            WHEN monetary <= monetary_50 AND monetary > monetary_25 THEN 2
            WHEN monetary <= monetary_75 AND monetary > monetary_50 THEN 3
            WHEN monetary > monetary_75 THEN 4
        END AS m_score
    FROM percentiles
)
),

fm_scores AS 
(   
    SELECT *,
    CAST(ROUND((f_score + m_score) / 2, 0) AS INT64) AS fm_score
    FROM RFM_scores
),

t7 as (
SELECT 
customer_id,
review_score,
    recency,
    frequency,
    monetary,
    r_score,
    f_score,
    m_score,
    fm_score,
    CONCAT(r_score, f_score, m_score) AS RFM_score,
        CASE WHEN (r_score = 4 AND fm_score = 4) THEN 'Champions'
        WHEN (r_score = 4 AND fm_score = 3) 
            OR (r_score = 3 AND fm_score = 4) THEN 'Loyal Customers'
        WHEN (r_score = 4 AND fm_score = 2) 
            OR (r_score = 3 AND fm_score = 3) THEN 'Potential Loyalists'
        WHEN r_score = 4 AND fm_score = 1 THEN 'Recent Customers'
        WHEN (r_score = 3 AND fm_score = 1) THEN 'Promising'
        WHEN (r_score = 3 AND fm_score = 2) 
            OR (r_score = 2 AND fm_score = 3)
            OR (r_score = 2 AND fm_score = 2) THEN 'Customers Needing Attention'
        WHEN r_score = 2 AND fm_score = 1 THEN 'About to Sleep'
        WHEN (r_score = 2 AND fm_score = 4) 
            OR (r_score = 1 AND fm_score = 3) THEN 'At Risk'
        WHEN (r_score = 1 AND fm_score = 4)THEN 'Cant Lose Them'
        WHEN r_score = 1 AND fm_score = 2 THEN 'Hibernating'
        WHEN r_score = 1 AND fm_score = 1 THEN 'Lost'
        END AS rfm_segment     
FROM fm_scores
)

SELECT
rfm_segment,
AVG(review_score),
COUNT(1) AS customers
FROM t7
GROUP BY 1
ORDER BY 2 DESC
```
