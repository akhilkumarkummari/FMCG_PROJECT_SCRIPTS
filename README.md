# FMCG_PROJECT_SCRIPTS

Query -1 :-

Select a.Product_code,a.product_name, f.base_price,f.promo_type
From retail_events_db.dim_products as a
Left Join retail_events_db.fact_events as f ON
a.product_code = f.product_code
where f.promo_type = 'BOGOF' AND f.base_price > 500;

Query -2 :-

select count(store_id) as Store_Count, City
from retail_events_db.dim_stores
Group by(City)
order by Store_Count Desc;

Query -3 :-

WITH total_revenue_beforepromo AS (
    SELECT
        n.campaign_name,
        ROUND(SUM(k.base_price * k.`quantity_sold(before_promo)`) / 1000000, 2) AS Total_Revenue_Before_Promo
    FROM
        retail_events_db.fact_events AS k
    LEFT JOIN
        retail_events_db.dim_campaigns AS n ON k.campaign_id = n.campaign_id
    GROUP BY
        n.campaign_name
),
total_revenue_afterpromo AS (
    SELECT
        n.campaign_name,
        SUM(
            CASE
                WHEN k.promo_type = '25% OFF' THEN (k.base_price * k.`quantity_sold(after_promo)`) * 0.75
                WHEN k.promo_type = '50% OFF' THEN (k.base_price * k.`quantity_sold(after_promo)`) * 0.5
                WHEN k.promo_type = '33% OFF' THEN (k.base_price * k.`quantity_sold(after_promo)`) * 0.67
                WHEN k.promo_type = 'BOGOF' THEN (k.base_price * k.`quantity_sold(after_promo)`) + (k.base_price * k.`quantity_sold(after_promo)` / 2)
                WHEN k.promo_type = '500 Cashback' THEN (k.base_price * k.`quantity_sold(after_promo)`) - 500
                ELSE k.base_price * k.`quantity_sold(after_promo)`
            END
        ) / 1000000 AS Total_Revenue_After_Promo
    FROM
        retail_events_db.fact_events AS k
    LEFT JOIN
        retail_events_db.dim_campaigns AS n ON k.campaign_id = n.campaign_id
    GROUP BY
        n.campaign_name
)
SELECT
    z.campaign_name,
    z.Total_Revenue_Before_Promo,
    j.Total_Revenue_After_Promo
FROM
    total_revenue_beforepromo AS z
LEFT JOIN
    total_revenue_afterpromo AS j ON j.campaign_name = z.campaign_name;

Query -4 :-

WITH ISU_percent AS (
    SELECT 
        Category,
        ROUND(
            SUM(
                (quantity_sold_after_promo - quantity_sold_before_promo) / quantity_sold_before_promo
            ) * 100, 2
        ) AS ISU_percentile
    FROM (
        SELECT 
            Category,
            CASE
                WHEN k.Promo_type = 'BOGOF' THEN k.`quantity_sold(after_promo)` / 2
                ELSE k.`quantity_sold(after_promo)`
            END AS quantity_sold_after_promo,
            k.`quantity_sold(before_promo)` AS quantity_sold_before_promo
        FROM 
            retail_events_db.fact_events AS k
        LEFT JOIN 
            retail_events_db.dim_products AS n ON k.product_code = n.product_code
        LEFT JOIN 
            retail_events_db.dim_campaigns AS l ON l.campaign_id = k.campaign_id
            where campaign_name = 'Diwali'
            
    ) AS subquery
    
    GROUP BY 
        category
    
)

SELECT 
    *,
    RANK() OVER (ORDER BY ISU_percentile DESC) AS ISU_Rank
FROM 
    ISU_percent;


Query - 5:-

WITH IR AS (
    SELECT 
        product_name,
        category,
        ROUND(
            (SUM((k.`quantity_sold(after_promo)` * k.base_price) - (k.`quantity_sold(before_promo)` * k.base_price)) / 1000000), 2
        ) as IR_Percentile
    FROM 
        retail_events_db.fact_events AS k
    LEFT JOIN 
        retail_events_db.dim_campaigns AS n ON k.campaign_id = n.campaign_id
    LEFT JOIN 
        retail_events_db.dim_products AS j ON j.product_code = k.product_code
	Left JOIN
		retail_events_db.dim_stores AS l ON l.store_id = k.store_id
    GROUP BY 
        product_name, category, campaign_name
)

SELECT 
    *,
    RANK() OVER (ORDER BY IR_percentile DESC) as TopProducts
FROM 
    IR
LIMIT 
    5;


Query 6:-

WITH ISU_percent AS (
    SELECT 
        city,
        ROUND(
            SUM(
                (quantity_sold_after_promo - quantity_sold_before_promo) / quantity_sold_before_promo
            ) * 100, 2
        ) AS ISU_percentile
    FROM (
        SELECT 
            j.city,
            CASE
                WHEN k.Promo_type = 'BOGOF' THEN k.`quantity_sold(after_promo)` / 2
                ELSE k.`quantity_sold(after_promo)`
            END AS quantity_sold_after_promo,
            k.`quantity_sold(before_promo)` AS quantity_sold_before_promo
        FROM 
            retail_events_db.fact_events AS k
        LEFT JOIN 
            retail_events_db.dim_products AS n ON k.product_code = n.product_code
        LEFT JOIN 
            retail_events_db.dim_campaigns AS l ON l.campaign_id = k.campaign_id
		Left JOIN
		retail_events_db.dim_stores AS j ON j.store_id = k.store_id
            
    ) AS subquery
    
    GROUP BY 
        j.city
    
)

SELECT 
    *,
    RANK() OVER (ORDER BY ISU_percentile ASC) AS ISU_Rank
FROM 
    ISU_percent
    limit 10;

Query 7 :-

WITH IR AS (
    SELECT 
        promo_type,
        ROUND(
            (SUM((k.`quantity_sold(after_promo)` * k.base_price) - (k.`quantity_sold(before_promo)` * k.base_price)) / 1000000), 2
        ) as IR_Percentile
    FROM 
        retail_events_db.fact_events AS k
    LEFT JOIN 
        retail_events_db.dim_campaigns AS n ON k.campaign_id = n.campaign_id
    LEFT JOIN 
        retail_events_db.dim_products AS j ON j.product_code = k.product_code
	Left JOIN
		retail_events_db.dim_stores AS l ON l.store_id = k.store_id
    GROUP BY 
        promo_type
)

SELECT 
    *,
    RANK() OVER (ORDER BY IR_percentile DESC) as TopProducts
FROM 
    IR
LIMIT 
    5;


Query 8:-

WITH ISU_percent AS (
    SELECT 
        Promo_type,
        ROUND(
            SUM(
                (quantity_sold_after_promo - quantity_sold_before_promo) / quantity_sold_before_promo
            ) * 100, 2
        ) AS ISU_percentile
    FROM (
        SELECT 
            Promo_type,
            CASE
                WHEN k.Promo_type = 'BOGOF' THEN k.`quantity_sold(after_promo)` / 2
                ELSE k.`quantity_sold(after_promo)`
            END AS quantity_sold_after_promo,
            k.`quantity_sold(before_promo)` AS quantity_sold_before_promo
        FROM 
            retail_events_db.fact_events AS k
        LEFT JOIN 
            retail_events_db.dim_campaigns AS l ON l.campaign_id = k.campaign_id
        
            
    ) AS subquery
    
    GROUP BY 
        Promo_type
    
)

SELECT 
    *,
    RANK() OVER (ORDER BY ISU_percentile) AS ISU_Rank
FROM 
    ISU_percent;


Query 9:-

WITH IR AS (
    SELECT 
        promo_type,city,
        ROUND(
            (SUM((k.`quantity_sold(after_promo)` * k.base_price) - (k.`quantity_sold(before_promo)` * k.base_price)) / 1000000), 2
        ) as IR_Percentile
    FROM 
        retail_events_db.fact_events AS k
    LEFT JOIN 
        retail_events_db.dim_campaigns AS n ON k.campaign_id = n.campaign_id
    LEFT JOIN 
        retail_events_db.dim_products AS j ON j.product_code = k.product_code
	Left JOIN
		retail_events_db.dim_stores AS l ON l.store_id = k.store_id
    GROUP BY 
        promo_type,city
)

SELECT 
    *,
    RANK() OVER (ORDER BY IR_percentile DESC) as TopProducts
FROM 
    IR
LIMIT 
    10;

Query 10 :-

WITH IR AS (
    SELECT 
        city,
        ROUND(
            (SUM((k.`quantity_sold(after_promo)` * k.base_price) - (k.`quantity_sold(before_promo)` * k.base_price)) / 1000000), 2
        ) as IR_Percentile
    FROM 
        retail_events_db.fact_events AS k
    LEFT JOIN 
        retail_events_db.dim_campaigns AS n ON k.campaign_id = n.campaign_id
    LEFT JOIN 
        retail_events_db.dim_products AS j ON j.product_code = k.product_code
	Left JOIN
		retail_events_db.dim_stores AS l ON l.store_id = k.store_id
    GROUP BY 
        city
)

SELECT 
    *,
    RANK() OVER (ORDER BY IR_percentile DESC) as TopProducts
FROM 
    IR
LIMIT 
    10;


