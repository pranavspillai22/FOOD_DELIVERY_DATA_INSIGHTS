--Q1. Find the top outlet by cuisine type without using limit and top function
WITH cte AS (
SELECT 
	cuisine,
    restaurant_id,
    count(*) as no_of_orders
from orders
group by cuisine, restaurant_id
)
select * from
(SELECT *, row_number() over(partition by cuisine order by no_of_orders desc) as rn
FROM cte ) a
where rn = 1;

--Q2. Find the daily new customer count from the launch date (everyday how many new customers are we acquiring)

----Method 1

WITH cte AS
(SELECT customer_code, cast(placed_at as date) AS first_order_date
		, ROW_NUMBER() OVER(PARTITION BY customer_code ORDER BY placed_at) AS rn 
FROM orders )
SELECT first_order_date, COUNT(*) AS new_customers
FROM cte
WHERE rn = 1
GROUP BY first_order_date
ORDER BY first_order_date;

----Method 2

WITH cte AS(
SELECT 
	customer_code,
    CAST(MIN(placed_at) AS DATE) AS first_order_date
FROM orders
GROUP BY customer_code
)
SELECT 
	first_order_date,
    COUNT(*) AS new_customers
FROM cte
GROUP BY first_order_date
ORDER BY first_order_date;

--Q3. Find count of all the users who were acquired in Jan 2025 and only placed one order in Jan and didnot place any other order
WITH cte AS(
SELECT customer_code, COUNT(*) AS no_of_orders
FROM orders
WHERE placed_at LIKE '2025-01%' 
AND customer_code NOT IN (SELECT
	DISTINCT customer_code
FROM orders
WHERE NOT placed_at LIKE '2025-01%')
GROUP BY customer_code
HAVING no_of_orders = 1)
SELECT count(*) AS total_users from cte;

--Q4. List all the customers with no order in the last 7 days but were acquired one month ago with their first order on promo
WITH cte AS(
SELECT 
	customer_code, 
    MIN(placed_at) AS first_order_date, 
    MAX(placed_at) AS latest_order_date
FROM orders
GROUP BY customer_code
)
SELECT cte.*, orders.Promo_code_Name AS first_order_promo
FROM cte
JOIN orders ON cte.Customer_code = orders.Customer_code AND cte.first_order_date = orders.Placed_at
WHERE cte.latest_order_date < DATE_ADD('2025-03-31', INTERVAL -7 DAY)
AND cte.first_order_date < DATE_ADD('2025-03-31', INTERVAL -1 MONTH)
AND orders.Promo_code_Name IS NOT NULL;

--Q5.Growth team is planning to create a trigger that will target customers after their every third order
--with a personalized communication and they asked to create a query for this.
WITH cte AS(
SELECT
	*, ROW_NUMBER() OVER(PARTITION BY Customer_code ORDER BY placed_at) AS rn
FROM orders)
SELECT *
FROM cte
WHERE rn % 3 = 0 AND CAST(placed_at AS DATE) = CAST(NOW() AS DATE);

--Q6. List the customers who placed more than 1 order and all their orders on a promo only.
SELECT Customer_code, COUNT(*) AS total_orders, COUNT(Promo_code_Name) AS promo_orders
FROM orders
GROUP BY Customer_code
HAVING total_orders >1 and total_orders = promo_orders;

--Q7.What percent of customers were organically acquired in Jan 2025(placed their first order without promocode)
--METHOD 1
WITH cte AS(
SELECT *, DENSE_RANK() OVER(PARTITION BY customer_code ORDER BY placed_at) AS rn
FROM orders 
WHERE placed_at LIKE '2025-01%')
SELECT 
	COUNT(CASE WHEN rn=1 AND promo_code_name IS NULL THEN customer_code END)/COUNT(DISTINCT customer_code) *100.0 AS organic_customer_percent
FROM cte;

--METHOD 2
WITH cte AS(
SELECT *, DENSE_RANK() OVER(PARTITION BY customer_code ORDER BY placed_at) AS rn
FROM orders 
WHERE placed_at LIKE '2025-01%')
, cte2 AS(
SELECT 
	COUNT(*) AS organic_customers
FROM cte
WHERE rn = 1 AND promo_code_name IS NULL)
SELECT
	organic_customers/(SELECT COUNT(DISTINCT customer_code) FROM cte) * 100.0 AS organic_customer_percent
FROM cte2;
