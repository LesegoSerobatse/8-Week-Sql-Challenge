##### Data Cleaning

Before you start writing your SQL queries however - you might want to investigate the data, you may want to do something with some of those null values and data types in the customer_orders and runner_orders tables!

**Customers Orders table cleaning**
```SQL
UPDATE customer_orders
SET exclusions = ''
WHERE exclusions IS NULL OR exclusions LIKE '%null%';

UPDATE customer_orders
SET extras = ''
WHERE extras IS NULL OR extras LIKE '%null%';

SELECT *
FROM customer_orders;

SELECT COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'customer_orders';
```

![Alt text](<Pizza Runner pics/prs1.png>)

![Alt text](<Pizza Runner pics/prs2.png>)


**Runner's Orders table cleaning**

```SQL
UPDATE runner_orders
SET pickup_time = 
		CASE
			WHEN pickup_time IS NULL THEN ''
			WHEN pickup_time LIKE '%null%' THEN ''
			ELSE pickup_time
		END,
	cancellation =
		CASE
			WHEN cancellation IS NULL THEN ''
			WHEN cancellation LIKE '%null%' THEN ''
			ELSE cancellation
		END,
	distance = 
		CASE
			WHEN distance IS NULL THEN ''
			WHEN distance LIKE '%null%' THEN ''
			WHEN distance LIKE '%km' THEN TRIM(REPLACE(distance, 'km', ''))
			ELSE distance
		END,
	duration = 
		CASE
			WHEN duration IS NULL THEN ''
			WHEN duration LIKE '%null%' THEN ''
			WHEN duration LIKE '%minutes' THEN TRIM(REPLACE(duration, 'minutes', ''))
			WHEN duration LIKE '%minute' THEN TRIM(REPLACE(duration, 'minute', ''))
			WHEN duration LIKE '%mins' THEN TRIM(REPLACE(duration, 'mins', ''))
			ELSE duration
		END;

ALTER TABLE runner_orders
ALTER COLUMN pickup_time DATETIME;

ALTER TABLE runner_orders
ALTER COLUMN distance FLOAT;

ALTER TABLE runner_orders
ALTER COLUMN duration INT;

SELECT COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'runner_orders';			

SELECT *
FROM runner_orders;
```

![Alt text](<Pizza Runner pics/prs3.png>)

![Alt text](<Pizza Runner pics/prs4.png>)



### A. Pizza Metrics

1. How many pizzas were ordered?

```SQL
SELECT COUNT(*) AS 'Number of Pizzas ordered'
FROM customer_orders;
```

![Alt text](<Pizza Runner pics/prm1.png>)


2. How many unique customer orders were made?

```SQL
SELECT COUNT(DISTINCT order_id) AS 'Number of Orders'
FROM customer_orders;
```
![Alt text](<Pizza Runner pics/prm2.png>)


3. How many successful orders were delivered by each runner?

```SQL
SELECT runner_id, COUNT(order_id)-SUM 
			(CASE
				WHEN cancellation LIKE '%cancellation%' THEN 1
				ELSE 0
			END) AS 'Order completed by runners'
FROM runner_orders
GROUP BY runner_id;
```
![Alt text](<Pizza Runner pics/prm3.png>)


4. How many of each type of pizza was delivered?

```SQL
SELECT customer_orders.pizza_id, COUNT(customer_orders.pizza_id) - SUM(CASE
																			WHEN runner_orders.cancellation LIKE '%cancellation%' THEN 1
																			ELSE 0
																		END) AS 'Number of delivered pizzas'
FROM customer_orders
JOIN runner_orders
ON customer_orders.order_id = runner_orders.order_id
GROUP BY customer_orders.pizza_id
```
![Alt text](<Pizza Runner pics/prm4.png>)


5. How many Vegetarian and Meatlovers were ordered by each customer?

```SQL
WITH les AS
	(
	SELECT c.customer_id, p.pizza_id, COUNT(p.pizza_id) AS 'Pizzas Ordered'
	FROM customer_orders c
	JOIN pizza_names p
	ON c.pizza_id = p.pizza_id
	GROUP BY c.customer_id, p.pizza_id
	)
SELECT l.customer_id, p.pizza_name, l.[Pizzas Ordered]
FROM les l
LEFT JOIN pizza_names p
ON l.pizza_id = p.pizza_id
```
![Alt text](<Pizza Runner pics/prm5.png>)


6. What was the maximum number of pizzas delivered in a single order?

```SQL
WITH Ngomi AS
		(SELECT cus.customer_id, cus.order_id, COUNT(cus.order_id) - SUM(CASE 
																			WHEN run.cancellation LIKE '%cancellation%' THEN 1
																			ELSE 0
																		 END) AS 'Delivered Orders'
											
		FROM customer_orders cus
		LEFT JOIN runner_orders run
		ON cus.order_id = run.order_id
		GROUP BY cus.order_id, cus.customer_id)
SELECT MAX([Delivered Orders]) AS 'Largest Order Delivered'
FROM Ngomi N
```
![Alt text](<Pizza Runner pics/prm6.png>)


7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```SQL
WITH Leg AS
	(
	SELECT cu.customer_id, cu.order_id, cu.pizza_id, cu.exclusions, cu.extras, ru.cancellation, ROW_NUMBER() OVER (ORDER BY cu.customer_id) AS 'Pizza_Number'
	FROM customer_orders cu
	LEFT JOIN runner_orders ru
	ON cu.order_id = ru.order_id
	),
	 Arm AS
		(SELECT Pizza_Number, customer_id, 
							SUM(CASE
									WHEN LEN(l.exclusions) = 0 AND LEN(l.extras) = 0 THEN 1
									ELSE 0
								 END) AS No_Change_1,
							SUM(CASE
									WHEN LEN(l.exclusions) != 0 OR LEN(l.extras) != 0 THEN 1
									ELSE 0
								 END) AS Change_1,
							SUM(CASE
									WHEN l.cancellation LIKE '%cancellation%' THEN 1
									ELSE 0
								END) AS Chancellation
		FROM Leg l
		GROUP BY Pizza_Number, customer_id)

SELECT customer_id,	SUM(CASE
							WHEN Chancellation = 0 THEN No_Change_1
							ELSE 0
						END) AS No_Changes,
					SUM(CASE
							WHEN Chancellation = 0 THEN Change_1
							ELSE 0
						END) AS Changed
FROM Arm
GROUP BY customer_id
```
![Alt text](<Pizza Runner pics/prm7.png>)


8. How many pizzas were delivered that had both exclusions and extras?

```SQL
WITH lese AS
		(SELECT c.customer_id, c.order_id, c.exclusions, c.extras, r.cancellation, ROW_NUMBER() OVER (ORDER BY c.order_id) AS 'Pizza Number'
		FROM customer_orders c
		LEFT JOIN runner_orders r
		ON c.order_id = r.order_id),
	 tese AS
		(SELECT  [Pizza Number], SUM(CASE
										WHEN LEN(exclusions) != 0 AND LEN(extras) != 0 AND LEN(cancellation) = 0 THEN 1
										ELSE 0
									END) AS 'Both Changes'
		FROM lese
		GROUP BY [Pizza Number])
SELECT SUM([Both Changes]) AS 'Number of Pizzas with both changes'
FROM tese
```
![Alt text](<Pizza Runner pics/prm8.png>)


9. What was the total volume of pizzas ordered for each hour of the day?

```SQL
SELECT DATEPART(hour, order_time) AS 'Hour Order time', COUNT(DATEPART(hour, order_time)) AS 'Total volume of pizzas ordered for each hour of the day'
FROM customer_orders
GROUP BY DATEPART(hour, order_time)
ORDER BY DATEPART(hour, order_time)
```
![Alt text](<Pizza Runner pics/prm9.png>)


10. What was the volume of orders for each day of the week?

```SQL
SELECT DATEPART(WEEKDAY, order_time) AS 'Day of the week', COUNT(DATEPART(WEEKDAY, order_time)) AS 'Total volume of orders for each day of the week'
FROM customer_orders
GROUP BY DATEPART(WEEKDAY, order_time)
ORDER BY DATEPART(WEEKDAY, order_time)
```
![Alt text](<Pizza Runner pics/prm10.png>)




### B. Runner and Customer Experience


1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```SQL
SELECT COUNT(runner_id) AS 'Runners signed up for each 1 week period', DATEPART(WEEK, registration_date) AS 'Week number of the year'
FROM runners
GROUP BY DATEPART(WEEK, registration_date)
```
![Alt text](<Pizza Runner pics/prc1.png>)


2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```SQL
SELECT r.runner_id, AVG(DATEDIFF(MINUTE, c.order_time, r.pickup_time)) AS 'Average time taken for pickup in minutes'
FROM customer_orders c
LEFT JOIN runner_orders r
ON c.order_id = r.order_id
WHERE r.distance != 0
GROUP BY r.runner_id
```
![Alt text](<Pizza Runner pics/prc2.png>)


3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

```SQL
WITH Correlation AS
	(
	SELECT c.customer_id, c.order_id, COUNT(c.order_id) AS 'Number of pizzas per order', AVG(DATEDIFF(MINUTE, c.order_time, r.pickup_time)) AS 'Order preparation time'
	FROM customer_orders c
	LEFT JOIN runner_orders r
	ON c.order_id = r.order_id
	WHERE r.distance != 0
	GROUP BY c.customer_id, c.order_id
	)
SELECT [Number of pizzas per order], AVG([Order preparation time]) AS 'Average preparation time per number of pizzas'
FROM Correlation
GROUP BY [Number of pizzas per order]
```
![Alt text](<Pizza Runner pics/prc3.png>)


4. What was the average distance travelled for each customer?

```SQL
SELECT c.customer_id, AVG(r.distance) AS 'Average distance travelled for each customer'
FROM customer_orders c
LEFT JOIN runner_orders r
ON c.order_id = r.order_id
WHERE r.distance != 0
GROUP BY c.customer_id
```
![Alt text](<Pizza Runner pics/prc4.png>)


5. What was the difference between the longest and shortest delivery times for all orders?

```SQL
SELECT MAX(r.duration) - MIN(r.duration) AS 'difference between the longest and shortest delivery times for all orders'
FROM runner_orders r
WHERE r.duration != 0
```
![Alt text](<Pizza Runner pics/prc5.png>)


6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

```SQL
SELECT order_id, runner_id, ROUND(AVG((distance)/(CAST((CAST(duration AS FLOAT))/(CAST(60 AS FLOAT)) AS FLOAT))), 1)  AS 'Average speed for each runner for each delivery in km/h'
FROM runner_orders
WHERE distance != 0
GROUP BY order_id, runner_id
ORDER BY runner_id
```
![Alt text](<Pizza Runner pics/prc6.png>)


7. What is the successful delivery percentage for each runner?

This query is ambiguous as the KPI for successful delivery is not known.





