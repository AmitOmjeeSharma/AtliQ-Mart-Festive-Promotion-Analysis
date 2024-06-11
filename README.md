# *AtliQ-Mart-Festive-Promotion-Analysis*
This project analyzes the effectiveness of Diwali 2023 and Sankranti 2024 promotional campaigns across 50 AtliQ Mart supermarkets. Utilizing Power BI, SQL &amp; Excel. The analysis provides actionable insights to optimize future promotional strategies.

## Introduction

In the retail sector, promotional campaigns are essential for boosting sales and attracting customers during the festive season. The purpose of this study is to evaluate the effectiveness of AtliQ Mart's advertising strategies for Sankranti 2024 and Diwali 2023. We seek to understand the impact of these efforts and offer suggestions for improving upcoming marketing campaigns by utilizing data analytics.

## Data Sources

The analysis is based on data obtained from AtliQ Mart's internal databases. The main datasets used include fact_events, dim_products, dim_stores, and sales_summary. These datasets contain information about product sales, store locations, promotional events, and campaign revenues.

## Project Overview:

1. Analyzed data from AtliQ Mart's internal databases.
2. Performed SQL queries to fulfill business ad-hoc-requests.
3. The insights aim to guide future promotional strategies and optimize resource allocation.
4. A dashboard has been created to provide recommended insights to sales & business directors.

## Business Requests

### 1. High-Value Products in 'BOGOF' Promotion

**Objective:** Identify high-value products featured in the 'BOGOF' (Buy One Get One Free) promotion.

```sql
SELECT
      DISTINCT product_name, base_price 
FROM
    fact_events fe
JOIN
    dim_products p ON p.product_code = fe.product_code
WHERE
     base_price > 500 AND promo_type = 'BOGOF';
```

### 2. Store Presence Overview

**Objective:** Provide an overview of the number of stores in each city.

```sql
SELECT
      city , COUNT(store_id) AS Store_Count 
FROM
    dim_stores
GROUP BY
        city
ORDER BY
        store_count DESC;
```

### 3. Promotional Campaign Revenue Analysis
**Objective:** Display total revenue generated before and after each promotional campaign.

```sql
SELECT
      campaign_name, CONCAT(ROUND(SUM(base_price * `quantity_sold(before_promo)`)/1000000,2),'M') as Revenue_Before,
      CONCAT(ROUND( SUM(CASE
                            WHEN promo_type = "25% OFF" THEN base_price * 0.75 * `quantity_sold(after_promo)`
                            WHEN promo_type = "33% OFF" THEN base_price * 0.67 * `quantity_sold(after_promo)`
                            WHEN promo_type = "50% OFF" THEN base_price * 0.5 * `quantity_sold(after_promo)`
                            WHEN promo_type = "BOGOF" THEN base_price * 0.5 * `quantity_sold(after_promo)`
                            WHEN promo_type = "500 Cashback" THEN (base_price - 500) * `quantity_sold(after_promo)`
                        END)/1000000,2),'M') as Revenue_After 
FROM
    fact_events fe
JOIN
    dim_campaigns c ON c.campaign_id = fe.campaign_id
GROUP BY
        campaign_name;

```

### 4. Incremental Sold Quantity Analysis during Diwali Campaign
**Objective:** Calculate Incremental Sold Quantity (ISU%) for each category during the Diwali campaign.

```sql
WITH CTE as
          (SELECT category,
          Round(SUM((`quantity_sold(after_promo)`- `quantity_sold(before_promo)`)*100)/ SUM(`quantity_sold(before_promo)`),2) as `ISU%`
FROM
    fact_events fe
JOIN
    dim_products p ON fe.product_code = p.product_code
JOIN
    dim_campaigns c ON fe.campaign_id = c.campaign_id
WHERE
     campaign_name = "Diwali" 
GROUP BY
        category)

SELECT
      category, `ISU%`, row_number() OVER(ORDER BY `ISU%` DESC) as rank_order
FROM CTE;
```

### 5. Top 5 Products by Incremental Revenue Percentage
**Objective:** Identify the top 5 products ranked by Incremental Revenue Percentage (IR%) across all campaigns.

```sql
WITH CTE as 
        	(SELECT product_name, category,
            ROUND(((ROUND( SUM(CASE
                        			      WHEN promo_type = "25% OFF" THEN base_price * 0.75 * `quantity_sold(after_promo)`
                                    WHEN promo_type = "33% OFF" THEN base_price * 0.67 * `quantity_sold(after_promo)`
                                    WHEN promo_type = "50% OFF" THEN base_price * 0.5 * `quantity_sold(after_promo)`
                                    WHEN promo_type = "BOGOF" THEN base_price * 0.5 * `quantity_sold(after_promo)`
                                    WHEN promo_type = "500 Cashback" THEN (base_price - 500) * `quantity_sold(after_promo)`
                                END)/1000000,2) - ROUND(SUM(base_price * `quantity_sold(before_promo)`)/1000000,2)) /
                                    ROUND(SUM(base_price * `quantity_sold(before_promo)`)/1000000,2))*100,2) as `IR%`
FROM
    fact_events fe
JOIN
    dim_products p ON fe.product_code = p.product_code
JOIN
    dim_campaigns c ON fe.campaign_id = c.campaign_id  
GROUP BY
        product_name,category )
SELECT
      product_name, category, `IR%`
FROM CTE
ORDER BY
        `IR%` DESC LIMIT 5 ;
```
