# SQL Project: Data Analysis for Zomato – A Food Delivery Company

## Overview
This project demonstrates my SQL problem-solving skills through the analysis of data for Zomato, a popular food delivery company in India. The project involves setting up the database, importing data, handling null values, and solving a variety of business problems using complex SQL queries.

![ERD](https://github.com/nitesh2930/zomato_sql_project/blob/main/erd.png.jpg)

## Project Structure
- **Database Setup:** Creation of the `zomato_db` database and the required tables.
- **Data Import:** Inserting sample data into the tables.
- **Data Cleaning:** Handling null values and ensuring data integrity.
- **Business Problems:** Solving 20 specific business problems using SQL queries.

## Database Setup

```sql
CREATE DATABASE zomato_db;'''

-- Zomato Database & Schema Setup

DROP TABLE IF EXISTS deliveries CASCADE;
DROP TABLE IF EXISTS orders CASCADE;
DROP TABLE IF EXISTS restaurants CASCADE;
DROP TABLE IF EXISTS customers CASCADE;
DROP TABLE IF EXISTS riders CASCADE;

-- 1. Customers Table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(50) NOT NULL,
    reg_date DATE
);

-- 2. Restaurants Table
CREATE TABLE restaurants (
    restaurant_id INT PRIMARY KEY,
    restaurant_name VARCHAR(70) NOT NULL,
    city VARCHAR(50),
    opening_hours VARCHAR(50)
);

-- 3. Riders Table
CREATE TABLE riders (
    rider_id INT PRIMARY KEY,
    rider_name VARCHAR(50) NOT NULL,
    sign_up DATE
);

-- 4. Orders Table
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    restaurant_id INT,
    order_item VARCHAR(100),
    order_date DATE,
    order_time TIME,
    order_status VARCHAR(55),
    total_amount NUMERIC(10, 2)
);

-- 5. Deliveries Table
CREATE TABLE deliveries (
    delivery_id INT PRIMARY KEY,
    order_id INT,
    delivery_status VARCHAR(35),
    delivery_time TIME,
    rider_id INT
);

-- Foreign Key Constraints

ALTER TABLE orders
ADD CONSTRAINT fk_orders_customers
FOREIGN KEY (customer_id)
REFERENCES customers(customer_id);

ALTER TABLE orders
ADD CONSTRAINT fk_orders_restaurants
FOREIGN KEY (restaurant_id)
REFERENCES restaurants(restaurant_id);

ALTER TABLE deliveries
ADD CONSTRAINT fk_deliveries_orders
FOREIGN KEY (order_id)
REFERENCES orders(order_id);

ALTER TABLE deliveries
ADD CONSTRAINT fk_deliveries_riders
FOREIGN KEY (rider_id)
REFERENCES riders(rider_id);

------Data Import--------
-- 1. Import Customers
\copy customers(customer_id, customer_name, reg_date) FROM 'data/customers.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',');

-- 2. Import Restaurants
\copy restaurants(restaurant_id, restaurant_name, city, opening_hours) FROM 'data/restaurants.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',');

-- 3. Import Riders
\copy riders(rider_id, rider_name, sign_up) FROM 'data/riders.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',');

-- 4. Import Orders
\copy orders(order_id, customer_id, restaurant_id, order_item, order_date, order_time, order_status, total_amount) FROM 'data/orders.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',');

-- 5. Import Deliveries
\copy deliveries(delivery_id, order_id, delivery_status, delivery_time, rider_id) FROM 'data/deliveries.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',');


-- ----------------------------------------------------------------------------------------------------
-- DATA CLEANING & INTEGRITY CHECKS (NULL VALUES)
-- ----------------------------------------------------------------------------------------------------

-- 1. Check NULL values in CUSTOMERS table
SELECT COUNT(*) AS null_count_customers
FROM customers
WHERE customer_name IS NULL
   OR reg_date IS NULL;


-- 2. Check NULL values in RESTAURANTS table
SELECT COUNT(*) AS null_count_restaurants
FROM restaurants
WHERE restaurant_name IS NULL
   OR city IS NULL
   OR opening_hours IS NULL;


-- 3. Check NULL values in RIDERS table
SELECT COUNT(*) AS null_count_riders
FROM riders
WHERE rider_name IS NULL
   OR sign_up IS NULL;


-- 4. Check NULL values in ORDERS table
SELECT COUNT(*) AS null_count_orders
FROM orders
WHERE customer_id IS NULL
   OR restaurant_id IS NULL
   OR order_item IS NULL
   OR order_date IS NULL
   OR order_time IS NULL
   OR order_status IS NULL
   OR total_amount IS NULL;


-- 5. Check NULL values in DELIVERIES table
-- (delivery_time null ho sakta hai agar order deliver nahi hua, isliye rider_id aur status check karte hain)
SELECT COUNT(*) AS null_count_deliveries
FROM deliveries
WHERE order_id IS NULL
   OR delivery_status IS NULL
   OR rider_id IS NULL;

----------------------------------
--reports and analysis---
----------------------------------

--Q.1
--write a query to find the top 5 most frequently ordered dishes by customer called "Arjun Mehta" in the last one year.
--


---- join customers and order table
----filter for last one year
----filter arjun mehta
----group by costumer_id,customer_name,order item,count


SELECT 
customer_name,
dishes,
total_orders
SELECT
FROM --yaha table name aayega
(SELECT
    c.customer_id,
    c.customer_name,
    o.order_item AS dishes,
    COUNT(*) AS total_orders,
	DENSE_RANK() OVER (
    ORDER BY COUNT(*) DESC) AS dish_rank
FROM customers AS c
INNER JOIN orders AS o
    ON c.customer_id = o.customer_id
WHERE o.order_date >= (
    SELECT MAX(order_date) - INTERVAL '1 year'
    FROM orders
)
AND c.customer_name = 'Arjun Mehta'
GROUP BY
    c.customer_id,
    c.customer_name,
    o.order_item
ORDER BY 4 DESC
)as t1
WHERE dish_rank<=5;


-------------------------------------------------------------------------------------------------------------------------------------
--Q.2  popular time slots
---identify the time slots during which the most order are placed. based on two hours of intervals. 
--- mujhe es question ko tum kaise solve karoge mujhe steps batao
---METHOD 1

SELECT
  FLOOR(EXTRACT(HOUR FROM order_time)/2)*2 as start_time,
  FLOOR(EXTRACT(HOUR FROM order_time)/2)*2+2 as end_time,
  COUNT(*) as total_orders
FROM orders

GROUP BY 1,2
ORDER BY  3 DESC; 


------METHOD-2----------

SELECT
CASE
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 0  AND 1 THEN '00:00 - 02:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 2  AND 3 THEN '02:00 - 04:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 4  AND 5 THEN '04:00 - 06:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 6  AND 7 THEN '06:00 - 08:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 8  AND 9 THEN '08:00 - 10:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 10 AND 11 THEN '10:00 - 12:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 12 AND 13 THEN '12:00 - 14:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 14 AND 15 THEN '14:00 - 16:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 16 AND 17 THEN '16:00 - 18:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 18 AND 19 THEN '18:00 - 20:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 20 AND 21 THEN '20:00 - 22:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 22 AND 23 THEN '22:00 - 00:00'
END AS time_slots,
COUNT(*) AS total_orders
FROM orders
GROUP BY 1
ORDER BY 2 DESC;



------------------------------------------------------------------------------------------
-- order value analysis

--Q.3 find the average order value per customer who has placed more than 750 orders.
--return customer_name,and aov(average order value)



SELECT 
  c.customer_name,
  AVG(total_amount) AS aov,
  COUNT(*) total_orders
FROM customers as c
INNER JOIN orders as o
ON c.customer_id=o.customer_id
GROUP BY 1
HAVING COUNT(*) >750;


-------------------------------------------------------------------------------------------------------------------------------------

-- High value customers
---Q.4 List the customers who have spent more than 100k in total on food orders
--return customer name,customer_id



SELECT 
  c.customer_id,
  c.customer_name,
  SUM(o.total_amount) as total_spent
FROM customers as c
INNER JOIN orders as o
ON c.customer_id=o.customer_id
GROUP BY 1,2
HAVING SUM(total_amount) >100000;


----------------------------------------------------------------------------------------------------------------------------------------
-- order without delivery
--Q.5 write a query to find orders that were placed but not delivered.
--return each restaurent name,city and number of not delivered.



SELECT 
  r.restaurant_name,
  r.city,
  COUNT(o.order_id) as not_deliver_order
  
FROM orders as o
INNER JOIN restaurants as r
ON r.restaurant_id=o.restaurant_id
LEFT JOIN deleveries as d
ON o.order_id=d.order_id

WHERE delevery_status IS NULL

GROUP BY 1,2
ORDER BY 3 DESC;



--------------------------------------------------------------------------------------------------------------------------------
--Restaurant Revenue Ranking:
--Rank restaurants by their enutotal revenue from the last year ,including their name,
--total revenue and rank within their city

--NOTE ->isme mene alag se cte query use kiya h and rank me partition use kiya h
WITH ranking_table AS (
    SELECT
        r.restaurant_name,
        r.city,
        SUM(o.total_amount) AS revenue_of_restaurant,
        RANK() OVER (
            PARTITION BY r.city
            ORDER BY SUM(o.total_amount) DESC
        ) AS city_rank

    FROM restaurants AS r

    INNER JOIN orders AS o
        ON r.restaurant_id = o.restaurant_id

    GROUP BY 1,2
)

SELECT *
FROM ranking_table
WHERE city_rank <= 5;


-------------------------------------------------------------------------------------------------------------------
--Most poular dish by city:
--Q.7 identify the most popular dish in each city based on the no. of orders

WITH ranking_table AS
(
SELECT
      r.city,
      o.order_item,
	  COUNT(order_item) as popular_item,
	  RANK() OVER(PARTITION BY r.city ORDER BY COUNT(order_item)DESC) AS city_rank
FROM restaurants AS r
INNER JOIN orders AS o
 ON r.restaurant_id = o.restaurant_id
GROUP BY 1,2
)
SELECT *
FROM ranking_table
WHERE city_rank =1;


-----------------------------------------------------------------------------------------------------------
---CUSTOMER CHURN:
--Q.8 Find customer who have not placed an order in 2024 but did in 2023




SELECT DISTINCT customer_id FROM orders
   WHERE 
        EXTRACT(YEAR FROM order_date)=2023
    AND 
	  customer_id NOT IN
                    (SELECT DISTINCT customer_id FROM orders
					 WHERE 
                        EXTRACT(YEAR FROM order_date)=2024)

--NOTE:- iska output zero aayega kyuki isme mene check kiya jo 23 me order kiya h wo 24 me bhi kiya h


---------------------------------------------------------------------------------------------------------------

--Cancellation ratio comprasion:
--calculate and compare the order cancelation ratio for each restaurant between the
--current year and previous year.

WITH cancel_ratio_23 AS
(SELECT 
     o.restaurant_id,
     COUNT(o.order_id) as total_orders,
	 COUNT(CASE WHEN d.delevery_id IS NULL THEN 1 END) as not_deliver
FROM orders as o
LEFT JOIN deleveries as d
ON o.order_id=d.order_id
 WHERE EXTRACT(YEAR FROM order_date)=2023
GROUP BY 1
),

cancel_ratio_24 AS
(SELECT 
     o.restaurant_id,
     COUNT(o.order_id) as total_orders,
	 COUNT(CASE WHEN d.delevery_id IS NULL THEN 1 END) as not_deliver
FROM orders as o
LEFT JOIN deleveries as d
ON o.order_id=d.order_id
 WHERE EXTRACT(YEAR FROM order_date)=2024
GROUP BY 1
),
last_year_data AS (
SELECT 
      restaurant_id,
	  total_orders,
	  not_deliver,
	  ROUND((not_deliver::numeric/total_orders::numeric) * 100,0) as cancel_ratio 
	  FROM cancel_ratio_24
),

current_year_data AS (
SELECT 
      restaurant_id,
	  total_orders,
	  not_deliver,
	  ROUND((not_deliver::numeric/total_orders::numeric) * 100,0) as cancel_ratio 
	  FROM cancel_ratio_23
)

SELECT 
    c.restaurant_id AS restaurant_id,
    c.cancel_ratio AS current_year_cancel_ratio,
    l.cancel_ratio AS last_year_cancel_ratio
FROM current_year_data AS c
JOIN last_year_data AS l
    ON c.restaurant_id = l.restaurant_id;

--------------------------------------------------------------------------------------
--Q.10 Rider average delivery time:
--Determine each rider's average delivery time

SELECT 
   o.order_id,
	d.rider_id,
	o.order_time,
	d.delevery_time,
	d.delevery_time-o.order_time as time_difference,
	EXTRACT( EPOCH FROM (
        d.delevery_time - o.order_time + 
		CASE
            WHEN d.delevery_time < o.order_time
            THEN INTERVAL '1 day'
            ELSE INTERVAL '0 day'
          END
    )) / 60 AS time_difference_insec
FROM orders as o
INNER JOIN deleveries as d
ON o.order_id=d.order_id
WHERE d.delevery_status = 'Delivered';



---------------------------------------------------------------------------------------------------------------------------------
--Q.11 Monthly restaurant growth ratio:
--calculate each restaurant's growth ratio based on the total no. of delivered order since its joining

WITH growth_ratio AS (
    SELECT
        o.restaurant_id,
        TO_CHAR(o.order_date, 'MM-YY') AS month,

        COUNT(o.order_id) AS cr_month_orders,

        LAG(COUNT(o.order_id), 1) OVER (
            PARTITION BY o.restaurant_id
            ORDER BY DATE_TRUNC('month', o.order_date)
        ) AS prev_month_orders

    FROM orders AS o

    JOIN deleveries AS d
        ON o.order_id = d.order_id

    WHERE d.delevery_status = 'Delivered'

    GROUP BY
        o.restaurant_id,
        TO_CHAR(o.order_date, 'MM-YY'),
        DATE_TRUNC('month', o.order_date)
)

SELECT
    restaurant_id,
    month,
    prev_month_orders,
    cr_month_orders,

    ROUND(
        (
            (cr_month_orders::numeric - prev_month_orders::numeric)
            / NULLIF(prev_month_orders::numeric, 0)
        ) * 100,
        2
    ) AS growth_ratio

FROM growth_ratio


-----------------------------------------------------------------------------------
-- Q.12 Customer Segmentation:
-- Categorize customers into 'Gold' or 'Silver' groups based on their total spending
-- compared to the average order value (AOV). If a customer's total spending exceeds the AOV,
-- label them as 'Gold', otherwise, label them as 'Silver'. Write an SQL query to determine each segment's
-- total number of orders and total revenue


--customer total spend
--aov
--gold
--silver
--each category and total orders and total revenue

SELECT 
    cx_category,
    SUM(total_orders) AS total_orders,
    SUM(total_spent) AS total_revenue
FROM
(
    SELECT
        customer_id,
        SUM(total_amount) AS total_spent,
        COUNT(order_id) AS total_orders,
        CASE
            WHEN SUM(total_amount) > (SELECT AVG(total_amount) FROM orders) THEN 'Gold'
            ELSE 'Silver'
        END AS cx_category
    FROM orders
    GROUP BY 1
) AS t1
GROUP BY 1;

-------------------------------------------------------------------------------------
---- Q.13 Rider Monthly Earnings:
-- Calculate each rider's total monthly earnings, assuming they earn 8% of the order amount.

SELECT 
    d.rider_id,
    TO_CHAR(o.order_date, 'mm-yy') as month,
    SUM(total_amount) as revenue,
    SUM(total_amount)* 0.08 as riders_earning
FROM orders as o
JOIN deleveries as d
ON o.order_id = d.order_id
GROUP BY 1, 2
ORDER BY 1, 2

------------------------------------------------------------------------------------------

-- Q.14 Rider Ratings Analysis:
-- Find the number of 5-star, 4-star, and 3-star ratings each rider has.
-- riders receive this rating based on delivery time.
-- If orders are delivered less than 15 minutes of order received time the rider get 5 star rating,
-- if they deliver 15 and 20 minute they get 4 star rating
-- if they deliver after 20 minute they get 3 star rating.




SELECT 
    rider_id,
    stars,
    COUNT(*) as total_stars
FROM
(
    SELECT 
        rider_id,
        delevery_took_time,
        CASE 
            WHEN delevery_took_time < 15 THEN '5 star'
            WHEN delevery_took_time BETWEEN 15 AND 20 THEN '4 star'
            ELSE '3 star'
        END as stars
    FROM
    (
SELECT 
        o.order_id,
        o.order_time,
        d.delevery_time,
        EXTRACT(EPOCH FROM (d.delevery_time - o.order_time +
        CASE WHEN d.delevery_time < o.order_time THEN INTERVAL '1 day'
        ELSE INTERVAL '0 day' END
        ))/60 as delevery_took_time,
        d.rider_id
    FROM orders as o
    JOIN deleveries as d
    ON o.order_id = d.order_id
    WHERE delevery_status = 'Delivered'
  ) as t1
) as t2
GROUP BY 1, 2
ORDER BY 1, 3 DESC


-------------------------------------------------------------------------------------------------------------------------
---- Q.15 Order Frequency by Day:
-- Analyze order frequency per day of the week and identify the peak day for each restaurant.

SELECT * FROM
(
  SELECT
        r.restaurant_name,
        -- o.order_date,
        TO_CHAR(o.order_date, 'Day') as day,
        COUNT(o.order_id) as total_orders,
        RANK() OVER(PARTITION BY r.restaurant_name ORDER BY COUNT(o.order_id) DESC) as rank
    FROM orders as o
    JOIN
    restaurants as r
    ON o.restaurant_id = r.restaurant_id
    GROUP BY 1, 2
    ORDER BY 1, 3 DESC
    ) as t1
WHERE rank = 1


----------------------------------------------------------------------------------------------------------------------------------
-- Q.16 Customer Lifetime Value (CLV):
-- Calculate the total revenue generated by each customer over all their orders.

SELECT 
    o.customer_id,
    c.customer_name,
    SUM(o.total_amount) as CLV
FROM orders as o
JOIN customers as c
ON o.customer_id = c.customer_id
GROUP BY 1, 2

---------------------------------------------------------------------------------------------------------------------------
-- Q.17 Monthly Sales Trends:
-- Identify sales trends by comparing each month's total sales to the previous month.


SELECT 
    EXTRACT(YEAR FROM order_date) as year,
    EXTRACT(MONTH FROM order_date) as month,
    SUM(total_amount) as total_sale,
    LAG(SUM(total_amount), 1) OVER(ORDER BY EXTRACT(YEAR FROM order_date), EXTRACT(MONTH FROM order_date)) as prev_month_sale
FROM orders
GROUP BY 1, 2


-----------------------------------------------------------------------------------------------------------------------------------
-- Q.18 Rider Efficiency:
-- Evaluate rider efficiency by determining average delivery times and identifying those with the lowest and highest averages.


WITH new_table
AS
(
    SELECT 
        *,
        d.rider_id as riders_id,
        EXTRACT(EPOCH FROM (d.delevery_time - o.order_time +
        CASE WHEN d.delevery_time < o.order_time THEN INTERVAL '1 day' ELSE
        INTERVAL '0 day' END))/60 as time_deliver
    FROM orders as o
    JOIN deleveries as d
    ON o.order_id = d.order_id
    WHERE d.delevery_status = 'Delevered'
),
riders_time
AS
(
    SELECT 
        riders_id,
        AVG(time_deliver) avg_time
    FROM new_table
    GROUP BY 1
)
SELECT 
    MIN(avg_time),
    MAX(avg_time)
FROM riders_time


----------------------------------------------------------------------------------------------------------------------------------
-- Q.19 Order Item Popularity:
-- Track the popularity of specific order items over time and identify seasonal demand spikes.

SELECT 
    order_item,
    seasons,
    COUNT(order_id) as total_orders
FROM
(
SELECT 
    *,
    EXTRACT(MONTH FROM order_date) as month,
    CASE 
        WHEN EXTRACT(MONTH FROM order_date) BETWEEN 4 AND 6 THEN 'Spring'
        WHEN EXTRACT(MONTH FROM order_date) > 6 AND
        EXTRACT(MONTH FROM order_date) < 9 THEN 'Summer'
        ELSE 'Winter'
    END as seasons
FROM orders
) as t1
GROUP BY 1, 2
ORDER BY 1, 3 DESC





-------------------------------------------------------------------------------------------------------------------------------------
-- Q.20 Monthly Restaurant Growth Ratio:
-- Calculate each restaurant's growth ratio based on the total number of delivered orders since its joining

SELECT 
    r.city,
    SUM(total_amount) as total_revenue,
    RANK() OVER(ORDER BY SUM(total_amount) DESC) as city_rank
FROM orders as o
JOIN
restaurants as r
ON o.restaurant_id = r.restaurant_id
GROUP BY 1




-----------------------------------------------END OF REPORTS------------------------------------------------------------





























































