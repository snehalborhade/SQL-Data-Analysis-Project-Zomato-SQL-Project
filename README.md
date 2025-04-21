# SQL-Data-Analysis-Project-Zomato-SQL-Project

Project Overview
This project serves as a comprehensive demonstration of my proficiency in SQL and data-driven problem-solving by analyzing a dataset inspired by Zomato, a leading food delivery platform in India. The objective is to showcase the application of SQL in real-world business scenarios through the end-to-end management of a relational database system. The project spans the complete lifecycle—from setting up the database environment to formulating and executing complex queries that solve practical business challenges.
Project Structure

Project Structure
1. Database Setup
The project begins with the creation of a dedicated relational database named zonato_db. Within this database, multiple interrelated tables are defined to accurately represent different entities such as customers, restaurants, orders, menus, and delivery partners.

2. Data Import
Following the schema definition, sample data relevant to each table is inserted to simulate real-world operations. This step ensures that the dataset is sufficiently rich and diverse to support a variety of analytical use cases.

3. Data Cleaning
To maintain data integrity and ensure accurate analysis, null values and inconsistencies in the data are identified and addressed. Appropriate data cleaning techniques are applied to ensure that all tables are reliable and query-ready.

4. Business Problem Solving Using SQL
The core of the project involves addressing 16 distinct business problems that mimic real operational and strategic challenges faced by food delivery platforms. These problems range from identifying high-performing restaurants and analyzing customer behavior to optimizing delivery performance and revenue generation. Each problem is solved through the implementation of advanced SQL queries involving joins, subqueries, aggregations, window functions, and conditional logic.

##DATABASE SETUP:=
...
--zomato Data Analysis using SQL
CREATE TABLE customers(
				customer_id INT PRIMARY KEY, 
				customer_name VARCHAR(55), 
				reg_date DATE
				);
CREATE TABLE restaurants
			(
				restaurant_id INT PRIMARY KEY,
				restaurant_name VARCHAR(55),
				city VARCHAR(25),	
				opening_hours VARCHAR(55)
			);
CREATE TABLE orders
			(
				order_id INT PRIMARY KEY,	
				customer_id INT, -- FROM CUSTOMER TABLE 
				restaurant_id INT, -- FROM RESTAURANT TABLE
				order_item	VARCHAR(55),
				order_date	DATE,
				order_time	TIME,
				order_status VARCHAR(55),
				total_amount FLOAT
			);


--adding foreign key constraint
ALTER TABLE orders
ADD CONSTRAINT fk_customers
FOREIGN KEY (customer_id)
REFERENCES customers(customer_id);

ALTER TABLE orders
ADD CONSTRAINT fk_restaurants
FOREIGN KEY (restaurant_id)
REFERENCES restaurants(restaurant_id);
...
			
CREATE TABLE riders
				(
                 rider_id INT PRIMARY KEY,
				 rider_name	VARCHAR(55),
				 sign_up DATE

				);
CREATE TABLE deliveries
					(
                      delivery_id INT PRIMARY KEY, 
					  order_id	INT, -- FROM ORDERS TABLR
					  delivery_status VARCHAR(35),	
					  delivery_time	TIME,
					  rider_id INT -- FROM RIDERS TABLE
					);

ALTER TABLE deliveries
ADD CONSTRAINT fk_orders
FOREIGN KEY (order_id)
REFERENCES orders(order_id);

ALTER TABLE deliveries
ADD CONSTRAINT fk_riders
FOREIGN KEY (rider_id)
REFERENCES riders(rider_id);

--EDA
TRUNCATE TABLE orders, customers, deliveries, restaurants, riders;


SELECT * FROM customers;
SELECT * FROM restaurants;
SELECT * FROM orders;
SELECT * FROM riders;
SELECT * FROM deliveries;

--import datasets

SELECT COUNT(*) FROM customers
WHERE 
	customer_id IS NULL
	OR 
	customer_name IS NULL 
	OR
	reg_date  IS NULL 

SELECT COUNT(*) FROM restaurants
WHERE 
	restaurant_id IS NULL
	OR 
	restaurant_name IS NULL 
	OR
	opening_hours  IS NULL 
	OR
	city IS NULL

SELECT COUNT(*) FROM orders
WHERE 
	order_id	IS NULL
	OR
	customer_id	IS NULL
	OR
	restaurant_id	IS NULL
	OR
	order_item	IS NULL
	OR
	order_date	IS NULL
	OR
	order_time	IS NULL
	OR
	order_status	IS NULL
	OR
	total_amount IS NULL

-- Q.1) Write a query to find the top 5 most frequently ordered dishes by cutomer "Leo Martinez"

--join customers adn orders
--FILTER NATE MORGAN
--group by customer id , dishes and count


SELECT 
	customer_name, 
	dishes,
	total_orders
FROM
(SELECT
	c.customer_id,
	c.customer_name,
	o.order_item as dishes,
	COUNT(*) as total_orders,
	DENSE_RANK() OVER(ORDER BY COUNT(*) DESC) as rank 
FROM orders as o 
JOIN 
customers as c 
ON c.customer_id = o.customer_id
WHERE c.customer_name = 'Leo Martinez'
GROUP BY 1,2,3
ORDER BY 1,4 DESC) as t1
WHERE rank <= 5



--Q2 Pupular time slots 
-- Identify the time slots during which most orders are placed 

SELECT 
	CASE 
	   WHEN EXTRACT(HOUR FROM order_time) BETWEEN 0 AND 1 THEN '00:00 -02:00'
	   WHEN EXTRACT(HOUR FROM order_time) BETWEEN 2 AND 3 THEN '02:00 -04:00'
	   WHEN EXTRACT(HOUR FROM order_time) BETWEEN 4 AND 5 THEN '04:00 -06:00'
	   WHEN EXTRACT(HOUR FROM order_time) BETWEEN 6 AND 7 THEN '06:00 -08:00'
	   WHEN EXTRACT(HOUR FROM order_time) BETWEEN 8 AND 9 THEN '08:00 -10:00'
	   WHEN EXTRACT(HOUR FROM order_time) BETWEEN 10 AND 11 THEN '10:00 -12:00'
	   WHEN EXTRACT(HOUR FROM order_time) BETWEEN 12 AND 13 THEN '12:00 -14:00'
	   WHEN EXTRACT(HOUR FROM order_time) BETWEEN 14 AND 15 THEN '14:00 -16:00'
	   WHEN EXTRACT(HOUR FROM order_time) BETWEEN 16 AND 17 THEN '16:00 -18:00'
	   WHEN EXTRACT(HOUR FROM order_time) BETWEEN 18 AND 19 THEN '18:00 -20:00'
	   WHEN EXTRACT(HOUR FROM order_time) BETWEEN 20 AND 21 THEN '20:00 -22:00'
	   WHEN EXTRACT(HOUR FROM order_time) BETWEEN 22 AND 23 THEN '22:00 -00:00' 
   END AS time_slot,
   COUNT(order_id) AS order_count
FROM orders
GROUP BY time_slot
ORDER BY order_count DESC;


-- Q3. ORDER VALUE ANALYSIS 
-- Find the average order value per customer 
-- return customer_name and aov(average order value)

SELECT
	c.customer_name,
	AVG(o.total_amount) as aov
FROM orders as o
     JOIN customers as c 
	 ON c.customer_id = o.customer_id
GROUP BY 1



--Q4. high vale customers
--list the customers eho have spent more than 1000 in total on food orders
--return customer_name, and customer_id


SELECT
	c.customer_id,
	c.customer_name,
	SUM(o.total_amount) as total_spent
FROM orders as o
     JOIN customers as c 
	 ON c.customer_id = o.customer_id
GROUP BY 1
HAVING SUM(o.total_amount) > 1000


--05. Restaurant Revenue Ranking 
--Rank restaurants by their total revenue from last year , including their name, 
--total revenue and rank within their city


SELECT
    r.city,
	r.restaurant_name,
   SUM(o.total_amount) as revenue,
   RANK() OVER(PARTITION BY r.city ORDER BY SUM(o.total_amount)DESC) as rank
FROM orders as o
JOIN
restaurants as r
ON r.restaurant_id = o.restaurant_id 
GROUP BY 1,2
ORDER BY 1,3 DESC


--Q.7
--Most popular dish by city
-- Identify the most popular dish in each city based on the number of orders

SELECT 
     r.city, 
	 o.order_item as dish,
	 COUNT(order_id) as total_orders,
	 RANK() OVER(PARTITION BY r.city ORDER BY COUNT(order_id)DESC) as rank
FROM orders as o
JOIN 
restaurants as r
ON r.restaurant_id = o.restaurant_id
GROUP BY 1,2


--Q.8 find customers who havent placed orders in 2024 but did in 2023

SELECT * FROM orders
WHERE 
    EXTRACT(YEAR FROM order_date) = 2023
    AND
	customer_id NOT IN 
	(SELECT DISTINCT customer_id FROM orders
	  WHERE EXTRACT(YEAR FROM order_date)=2024)


--Q.9 Cancellation Rate Comparison 
--Calculate and compare the order cancellation rate for each restaurant between the current year 
-- previous year

SELECT 
 	o.restaurant_id,
	COUNT(o.order_id) as total_orders,
	COUNT(CASE WHEN d.delivery_status = 'not delivered' THEN 1 END) AS not_delivered
FROM orders as o
LEFT JOIN
deliveries as d
ON o.order_id = d.order_id
GROUP BY 1


--Q10) average delivery time taken by rider

SELECT 
		o.order_id,
		o.order_time,
		d.delivery_time,
		d.rider_id,
		d.delivery_time - o.order_time AS time_difference,
		EXTRACT(EPOCH FROM(d.delivery_time - o.order_time + 
		CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1days' ELSE INTERVAL '0 day' END))/60 
		as time_difference_insec
FROM orders as o 
JOIN
deliveries as d 
ON 
o.order_id = d.order_id
WHERE d.delivery_status = 'Delivered'


--Q.11 Monthly Restaurant Growth Rate
-- Calculate each restaurant's growth ratio based on the total 
--number of delivered orders since its joining
WITH growth_ratio 
AS
(
SELECT
	restaurant_id,
	TO_CHAR(o.order_date, 'mm-yy') as month, 
	COUNT(o.order_id) as current_month_orders,
	LAG(COUNT(o.order_id), 1) OVER(PARTITION BY o.restaurant_id ORDER BY TO_CHAR(o.order_date, 'mm-yy'))as prev_month_orders
FROM orders as o
JOIN deliveries as d
ON 
o.order_id = d.order_id
WHERE d.delivery_status = 'Delivered'
GROUP BY 1,2
ORDER BY 1,2
)
SELECT 
	 restaurant_id,
	 month,
	 current_month_orders,
	 prev_month_orders,
	 ROUND((current_month_orders::numeric-prev_month_orders::numeric)/prev_month_orders::numeric * 100,2) as monthly_growth_ratio
FROM growth_ratio 



--Q12 Customer Segmentation 
--Customer segmentarion into gold and siler groups based on their total spending 
--compared to the average order value (AOV). If a customers total spending exceeds the AOV
--label them as 'gold' otherwise label them as 'silver' 

SELECT 
	cx_category,
	SUM(total_orders),
	SUM(total_spent)
FROM
(SELECT 
		customer_id,
		SUM(total_amount) as total_spent,
		COUNT (order_id) as total_orders,
		CASE 
		WHEN SUM(total_amount) > (SELECT AVG(total_amount) FROM orders) THEN 'gold' 
			ElSE 'Silver'
		END as cx_category
FROM orders
group by 1
) as t1
GROUP BY 1


SELECT AVG(total_amount) FROM orders ---156




--Q13 Riders monthly earnings
--calcualte each riders'total monthly earings , assuming they earn 8% of the order amount


SELECT
	d.rider_id,
	TO_CHAR(o.order_date, 'mm-yy')as month,
	SUM(total_amount) as revenue,
	SUM(total_amount)* 0.8 as riders_earnings
FROM orders as o
JOIN deliveries as d
ON o.order_id = d.order_id
GROUP BY 1,2
ORDER BY 1, 2


--Q.14 order frequency by day
-- analyse order frequency per day of the week and identify the peak day for each restaurant


SELECT * FROM 
(SELECT 
	r.restaurant_name,
	--o.order_date,
	TO_CHAR(o.order_date,'day') as day,
	COUNT(o.order_id) as total_orders,
	RANK() OVER(PARTITION BY r.restaurant_name ORDER BY COUNT(o.order_id)DESC) as rank
FROM orders as o
JOIN
restaurants as r
ON o.restaurant_id = r.restaurant_id
GROUP BY 1,2
ORDER  BY 1,3 DESC)

WHERE rank = 1


--Q.15 Customer lifetime value (CLV)
--Calculate the total revenue generated by each customer over all their orders

SELECT 
	o.customer_id,
	c.customer_name,
	SUM(total_amount) as CLV
FROM orders as o

JOIN customers as c
ON o.customer_id = c.customer_id
GROUP BY 1,2


--q.16 Monthly Sales Trends 
--Identify sales trends by comparing each month's total sales to the previous month

SELECT 
	EXTRACT(YEAR FROM order_date) as year,
	EXTRACT(MONTH FROM order_date) as month,
	SUM(total_amount) as total_sale,
	LAG(SUM(total_amount), 1) OVER(ORDER BY EXTRACT(YEAR FROM order_date), EXTRACT(MONTH FROM order_date)) as previous_month_sale
FROM orders
GROUP BY 1,2
ORDER BY 1,2

