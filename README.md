# sql-data-analysis-ecommerce
This project contains a collection of SQL queries intended to get insights from the data. Using BigQuery, I analyzed two primary datasets: a Shopping Cart Database and the public TheLook E-Commerce dataset. The goal was to extract actionable insights regarding sales performance, inventory management, and logistics efficiency.

SQL Techniques Applied:
1. Complex Joins: Merging 3+ tables (Sales, Products, Orders, etc.) to create a unified view of the business.
2. Common Table Expressions (CTE): Utilizing WITH clauses to break down complex logic into readable logistics data.
3. Aggregation & Ranking: Using SUM(), COUNT(), and ORDER BY to identify top/bottom performing assets.
4. Date Manipulation: Using EXTRACT() and DATE() functions to perform time-series analysis (monthly and yearly trends).
5. Filtering & Logic: Implementing WHERE clauses to isolate "Completed" transactions and specific date ranges.

Case studies and queries:

-- In terms of the supply of item, which product category that have the most variety of SKU that being input to the system in the year 2019?
SELECT product_category, COUNT(DISTINCT product_sku) total_sku_variety
FROM `sql-project-376612.thelook_ecommerce.inventory_items`
WHERE date(created_at) BETWEEN '2019-01-01' AND '2019-12-31'
GROUP BY 1
ORDER BY 2 DESC;

-- Considering completed orders and focusing on the month of shipment, which month in the year 2021 had the lowest total order performance for the Jeans category?
SELECT EXTRACT(MONTH FROM o.shipped_at) as month, COUNT(i.product_category) product_category, i.product_category
FROM `sql-project-376612.thelook_ecommerce.order_items` o
JOIN `sql-project-376612.thelook_ecommerce.inventory_items` i
ON o.inventory_item_id = i.id
WHERE EXTRACT(YEAR FROM o.shipped_at) = 2021
AND product_category = 'Jeans'
AND status = 'Complete'
GROUP BY 1,3
ORDER BY 2;

-- To retrieve the location with the highest number of buyers (use unique user) who made purchases on our platform during the year 2022, which of the following SQL scripts is correct?
SELECT DISTINCT t2.country, COUNT(DISTINCT t1.user_id) total_user
FROM `sql-project-376612.thelook_ecommerce.orders` t1
LEFT JOIN `sql-project-376612.thelook_ecommerce.users` t2
ON t2.id=t1.user_id
WHERE t1.status = 'Complete'
AND DATE(t1.shipped_at) >= '2022-01-01'
AND DATE(t1.shipped_at) < '2023-01-01'
GROUP BY 1
ORDER BY 2 DESC;

-- Considering the completed orders that were shipped in the year 2022, which distribution center to which country destination had the highest total number of items sold?
WITH distribution as 
(
  SELECT a.id, a.name, b.id
  FROM `sql-project-376612.thelook_ecommerce.distribution_centers` a
  LEFT JOIN `sql-project-376612.thelook_ecommerce.products` b
  ON a.id = b.distribution_center_id
),
productss as 
(
  SELECT b.id, c.product_id, c.order_id
  FROM `sql-project-376612.thelook_ecommerce.products` b
  LEFT JOIN `sql-project-376612.thelook_ecommerce.order_items` c
  ON b.id = c.product_id
),
orderss as 
(
  SELECT c.order_id, d.order_id, d.user_id,d.status, d.shipped_at, d.num_of_item
  FROM `sql-project-376612.thelook_ecommerce.order_items` c
  LEFT JOIN `sql-project-376612.thelook_ecommerce.orders` d
  ON c.order_id = d.order_id
),
userss AS 
(
  SELECT d.user_id, e.country
  FROM `sql-project-376612.thelook_ecommerce.orders` d
  LEFT JOIN `sql-project-376612.thelook_ecommerce.users` e
  ON d.user_id = e.id
)
SELECT
distribution.name as distribution_center,
userss.country as country_destination,
SUM(orderss.num_of_item)
FROM distribution, productss, orderss, userss
WHERE status = 'Complete'
AND DATE(shipped_at) BETWEEN '2022-01-01' AND '2022-12-31'
GROUP BY 1,2
ORDER BY 3 DESC

-- Which product that gives highest revenue in the past 10 months?
SELECT b.product_name, SUM(a.total_price) total_revenue
FROM `carbon-quanta-390304.shopping_cart_database.sales` a
JOIN `carbon-quanta-390304.shopping_cart_database.products` b ON a.product_id = b.product_ID
JOIN `carbon-quanta-390304.shopping_cart_database.orders` c ON a.order_id = c.order_id
GROUP BY 1
ORDER BY 2 DESC;

-- Which product that gives lowest revenue in the past 10 months?
SELECT b.product_ID, b.product_name, b.size, b.colour, SUM(a.total_price) total_revenue
FROM `carbon-quanta-390304.shopping_cart_database.sales` a
JOIN `carbon-quanta-390304.shopping_cart_database.products` b ON a.product_id = b.product_ID
JOIN `carbon-quanta-390304.shopping_cart_database.orders` c ON a.order_id = c.order_id
GROUP BY 1,2,3,4
ORDER BY 5
LIMIT 10;

-- Highest quantity sold
SELECT b.product_ID, b.product_name, b.size, b.colour, SUM(a.quantity) total_quantity
FROM `carbon-quanta-390304.shopping_cart_database.sales` a
JOIN `carbon-quanta-390304.shopping_cart_database.products` b ON a.product_id = b.product_ID
JOIN `carbon-quanta-390304.shopping_cart_database.orders` c ON a.order_id = c.order_id
GROUP BY 1,2,3,4
ORDER BY 5 DESC
LIMIT 10;

-- Lowest quantity sold
SELECT b.product_ID, b.product_name, b.size, b.colour, SUM(a.quantity) total_quantity
FROM `carbon-quanta-390304.shopping_cart_database.sales` a
JOIN `carbon-quanta-390304.shopping_cart_database.products` b ON a.product_id = b.product_ID
JOIN `carbon-quanta-390304.shopping_cart_database.orders` c ON a.order_id = c.order_id
GROUP BY 1,2,3,4
ORDER BY 5
LIMIT 10;


-- Which product type that gives the highest revenue in the past 10 months?
SELECT b.product_type, SUM(a.total_price) total_revenue
FROM `carbon-quanta-390304.shopping_cart_database.sales` a
JOIN `carbon-quanta-390304.shopping_cart_database.products` b ON a.product_id = b.product_ID
JOIN `carbon-quanta-390304.shopping_cart_database.orders` c ON a.order_id = c.order_id
GROUP BY 1
ORDER BY 2 DESC;


-- Which product type that has highest quantity sold
SELECT b.product_type, SUM(a.quantity) total_quantity
FROM `carbon-quanta-390304.shopping_cart_database.sales` a
JOIN `carbon-quanta-390304.shopping_cart_database.products` b ON a.product_id = b.product_ID
JOIN `carbon-quanta-390304.shopping_cart_database.orders` c ON a.order_id = c.order_id
GROUP BY 1
ORDER BY 2 DESC;

-- highest to lowest revenue product type per month
SELECT EXTRACT(MONTH FROM c.order_date) as month, b.product_type, SUM(a.total_price) total_revenue
FROM `carbon-quanta-390304.shopping_cart_database.sales` a
JOIN `carbon-quanta-390304.shopping_cart_database.products` b ON a.product_id = b.product_ID
JOIN `carbon-quanta-390304.shopping_cart_database.orders` c ON a.order_id = c.order_id
GROUP BY 1, 2
ORDER BY 3 DESC;
