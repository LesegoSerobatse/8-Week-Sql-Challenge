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
SELECT runner_id, 
		COUNT(order_id)-
		SUM (CASE 
				WHEN cancellation LIKE '%cancellation%' THEN 1
				ELSE 0
			END) AS 'Order completed by runners'
FROM runner_orders
GROUP BY runner_id;
```
![Alt text](<Pizza Runner pics/prm3.png>)


4. How many of each type of pizza was delivered?

```SQL
SELECT customer_orders.pizza_id, 
		COUNT(customer_orders.pizza_id) - 
		SUM(CASE 
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
		(SELECT cus.customer_id, cus.order_id, 
				COUNT(cus.order_id) - 
				SUM(CASE 
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
		(SELECT  [Pizza Number], 
					SUM(CASE
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




### C. Ingredient Optimisation


1. What are the standard ingredients for each pizza?

```SQL
WITH tt AS
	(SELECT pizza_id, CAST(VALUE AS INT) AS 'topping_id'
	FROM pizza_recipes
	CROSS APPLY string_split(toppings, ',')),
	nn AS
	(SELECT tt.pizza_id, tt.topping_id, p.topping_name 
	FROM tt
	LEFT JOIN pizza_toppings p
	ON tt.topping_id = p.topping_id),
	temp AS
	(SELECT pizza_id, STUFF( (SELECT DISTINCT ','+ topping_name FROM nn WHERE pizza_id = t.pizza_id FOR XML PATH('')), 1, 1, '') AS 'Pizza_standard_ingredients'
	FROM nn t
	GROUP BY t.pizza_id)
SELECT piz.pizza_name, te.Pizza_standard_ingredients
FROM temp te
LEFT JOIN pizza_names piz
ON piz.pizza_id = te.pizza_id
```
![Alt text](<Pizza Runner pics/pro.png>)


2. What was the most commonly added extra?

```SQL
WITH tem AS
	(SELECT order_id, CAST(VALUE AS INT) AS 'Extra'
	FROM customer_orders
	CROSS APPLY STRING_SPLIT(extras, ',')),
	 tep AS
	 (SELECT p.topping_name, t.Extra, COUNT(t.Extra) AS 'Extra Frequency'
	 FROM tem t
	 LEFT JOIN pizza_toppings p
	 ON t.Extra = p.topping_id
	 WHERE Extra != 0 
	 GROUP BY t.Extra, p.topping_name)
SELECT topping_name, Extra, [Extra Frequency]
FROM tep
WHERE [Extra Frequency] = (SELECT MAX([Extra Frequency] FROM tep)
```
![Alt text](<Pizza Runner pics/pro1.png>)


3. What was the most common exclusion?

```SQL
WITH tem AS
	(SELECT order_id, CAST(VALUE AS INT) AS 'Exclusion'
	FROM customer_orders
	CROSS APPLY STRING_SPLIT(exclusions, ',')),
	 tep AS
	 (SELECT p.topping_name, t.Exclusion, COUNT(t.Exclusion) AS 'Exclusion Frequency'
	 FROM tem t
	 LEFT JOIN pizza_toppings p
	 ON t.Exclusion = p.topping_id
	 WHERE Exclusion != 0 
	 GROUP BY t.Exclusion, p.topping_name)
SELECT topping_name, Exclusion, [Exclusion Frequency] 
FROM tep
WHERE [Exclusion Frequency] = (SELECT MAX([Exclusion Frequency]) FROM tep)
```
![Alt text](<Pizza Runner pics/pro2.png>)


4. Generate an order item for each record in the customers_orders table in the format of one of the following:
   - Meat Lovers
   - Meat Lovers - Exclude Beef
   - Meat Lovers - Extra Bacon
   - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

```SQL
WITH LS AS
	(SELECT order_id, exclusions, ROW_NUMBER() OVER (ORDER BY order_id) AS record
	FROM customer_orders),
	LT AS
	(SELECT order_id, record, CAST(VALUE AS INT) AS exclus
	FROM LS
	CROSS APPLY STRING_SPLIT(exclusions, ',')),
	LU AS
	(SELECT L.order_id, L.record, T.topping_name
	FROM LT L
	LEFT JOIN pizza_toppings T
	ON L.exclus = T.topping_id),
	LX AS
	(SELECT order_id, record, STRING_AGG(topping_name, ',') AS toppings_names
	FROM LU
	GROUP BY order_id, record)
SELECT * INTO Exclusions
FROM LX;

SELECT *
FROM Exclusions;


WITH TS AS
	(SELECT order_id, extras, ROW_NUMBER() OVER (ORDER BY order_id) AS record
	FROM customer_orders),
	TT AS
	(SELECT order_id, record, CAST(VALUE AS INT) AS extra
	FROM TS
	CROSS APPLY STRING_SPLIT(extras, ',')),
	TU AS
	(SELECT L.order_id, L.record, T.topping_name
	FROM TT L
	LEFT JOIN pizza_toppings T
	ON L.extra = T.topping_id),
	TX AS
	(SELECT order_id, record, STRING_AGG(topping_name, ',') AS toppingZ_nameZ
	FROM TU
	GROUP BY order_id, record)
SELECT * INTO Extras
FROM TX;

WITH Cre AS
	(SELECT order_id, customer_id, pizza_id, exclusions, extras, order_time, ROW_NUMBER() OVER (ORDER BY order_id) AS record
	FROM customer_orders)
SELECT * INTO Customa_Ordas
FROM Cre;

WITH Leaf AS
		(SELECT c.order_id, c.customer_id, c.pizza_id, c.exclusions, c.extras, c.order_time, p.pizza_name, 
				CASE
					WHEN e.toppings_names IS NULL THEN ''
					ELSE 'Exclude ' + e.toppings_names
				END AS Exc, 
				CASE
					WHEN ex.toppingZ_nameZ IS NULL THEN ''
					ELSE 'Extra ' + ex.toppingZ_nameZ
				END AS Ext
	FROM Customa_Ordas c
	LEFT JOIN Exclusions e
	ON c.record = e.record
	LEFT JOIN Extras ex
	ON c.record = ex.record
	LEFT JOIN pizza_names p
	ON c.pizza_id = p.pizza_id)

SELECT * INTO New_Customa_Ordaz
FROM Leaf;


SELECT order_id, customer_id, pizza_id, exclusions, extras, order_time, 
		CASE
			WHEN Exc LIKE '' AND Ext LIKE '' THEN ''
			WHEN LEN(Exc) != 0 AND LEN(Ext) = 0 THEN CAST(pizza_name AS VARCHAR(20)) + ' - ' + Exc
			WHEN LEN(Exc) = 0 AND LEN(Ext) != 0 THEN CAST(pizza_name AS VARCHAR(20)) + ' - ' + Ext
			WHEN LEN(Exc) != 0 AND LEN(Ext) != 0 THEN CAST(pizza_name AS VARCHAR(20)) + ' - '+ Exc + ' - ' + Ext
		END AS 'Pizza Type - Exclusions - Extras'
FROM New_Customa_Ordaz;
```
![Alt text](<Pizza Runner pics/pro4.png>)


5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
    - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

```SQL
WITH las AS
	(SELECT c.order_id,c.customer_id, c.pizza_id, c.exclusions, c.extras, c.order_time, ROW_NUMBER() OVER (ORDER BY c.order_id) AS record, p.toppings
	FROM customer_orders c
	LEFT JOIN pizza_recipes p
	ON c.pizza_id = p.pizza_id),
	les AS
	(SELECT record, exclusions, VALUE AS 'matching'
	FROM las
	CROSS APPLY STRING_SPLIT(toppings, ',')),
	lis AS
	(SELECT record, exclusions, matching, 
			CASE
				WHEN LEN(exclusions) = 0 THEN matching
				WHEN LEN(exclusions) = 1 AND TRIM(matching) != (exclusions) THEN matching
				WHEN LEN(exclusions) = 1 AND TRIM(matching) = (exclusions) THEN ''
				ELSE TRIM(matching FROM (exclusions))
			END AS Tried
	FROM les),
	los AS
	(SELECT record, exclusions, matching, 
			CASE
				WHEN Tried LIKE ',%' OR Tried LIKE '%,' THEN ''
				WHEN Tried LIKE '%,%' AND Tried != matching THEN matching
				ELSE Tried
			END AS Fin_Tried
	FROM lis),
	lus AS
	(SELECT record, STRING_AGG(Fin_Tried, ',') AS excluded
	FROM los
	GROUP BY record),
	mas AS
	(SELECT a.order_id, a.customer_id, a.pizza_id, a.exclusions, a.extras, a.order_time, a.record, a.toppings, u.excluded + ',' + a.extras AS final_recipe
	FROM las a
	LEFT JOIN lus u
	ON a.record = u.record),
	mes AS
	(SELECT record, VALUE AS ingredienta
	FROM mas
	CROSS APPLY STRING_SPLIT(final_recipe, ',')),
	mis AS
	(SELECT record, ingredienta, COUNT(ingredienta) AS countings
	FROM mes
	WHERE ingredienta != ''
	GROUP BY record, ingredienta),
	mos AS
	(SELECT m.record, m.ingredienta, m.countings, p.topping_name
	FROM mis m
	LEFT JOIN pizza_toppings p
	ON m.ingredienta = p.topping_id),
	mus AS
	(SELECT record, ingredienta, 
			CASE
				WHEN countings = 1 THEN topping_name
				ELSE CAST(countings AS VARCHAR(2))+'x'+ topping_name
			END AS frequency_count
	FROM mos),
	nas AS
	(SELECT record, STRING_AGG(frequency_count, ',') WITHIN GROUP (ORDER BY frequency_count ) AS pizza_recipe_ingredients
	FROM mus
	GROUP BY record)
SELECT l.order_id, l.customer_id, l.pizza_id, l.exclusions, l.extras, l.order_time, n.pizza_recipe_ingredients
FROM las l
LEFT JOIN nas n
ON l.record = n.record
```
![Alt text](<Pizza Runner pics/pro5.png>)


6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

```SQL
WITH fas AS
	(SELECT c.order_id, c.pizza_id, c.exclusions, c.extras, p.toppings, ROW_NUMBER() OVER (ORDER BY c.order_id) AS record
	FROM customer_orders c
	LEFT JOIN pizza_recipes p
	ON c.pizza_id = p.pizza_id
	LEFT JOIN runner_orders r
	ON c.order_id = r.order_id
	WHERE r.distance != 0),
	fes AS 
	(SELECT record, exclusions, VALUE AS ingred
	FROM fas
	CROSS APPLY STRING_SPLIT(toppings, ',')),
	fis AS
	(SELECT record, exclusions, ingred, 
			CASE
				WHEN LEN(REPLACE((exclusions), TRIM(ingred), '')) < LEN((exclusions)) THEN ''
				ELSE ingred
			END AS ingred_2
	FROM fes),
	fos AS
	(SELECT record, STRING_AGG(ingred_2, ',') AS ingrediets
	FROM fis
	GROUP BY record),
	fus AS
	(SELECT fas.order_id, fas.pizza_id, fas.exclusions, fas.extras, fas.toppings, fos.record, fos.ingrediets
	FROM fas
	LEFT JOIN fos
	ON fas.record = fos.record),
	gas AS
	(SELECT record, ingrediets + ',' + extras AS ingredients
	FROM fus),
	ges AS
	(SELECT record, VALUE AS ingridy
	FROM gas
	CROSS APPLY STRING_SPLIT(ingredients, ',')),
	gis AS
	(SELECT TRIM(ingridy) AS ingridy, COUNT(TRIM(ingridy)) AS ingredients_count
	FROM ges
	WHERE ingridy != ''
	GROUP BY TRIM(ingridy))
SELECT p.topping_name, g.ingredients_count AS 'Total Quantity of each Ingredient'
FROM pizza_toppings p
LEFT JOIN gis g
ON p.topping_id = g.ingridy
ORDER BY g.ingredients_count DESC
```
![Alt text](<Pizza Runner pics/pro6.png>)



### D. Pricing and Ratings

1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

```SQL
SELECT  SUM (IIF(c.pizza_id = 1, 12, 10)) AS 'Total Revenue for Pizza Runner in $'
	FROM customer_orders c
	LEFT JOIN runner_orders r
	ON c.order_id = r.order_id
	LEFT JOIN pizza_names p
	ON c.pizza_id = p.pizza_id
	WHERE distance > 0
```
![Alt text](<Pizza Runner pics/prr1.png>)


2. What if there was an additional $1 charge for any pizza extras?
    - Add cheese is $1 extra

```SQL
WITH tas AS	
	(SELECT  c.pizza_id, p.pizza_name, c.extras, IIF(c.pizza_id = 1, 12, 10) AS 'pizza_cost_in_$', ROW_NUMBER() OVER (ORDER BY c.order_id) AS 'record'
	FROM customer_orders c
	LEFT JOIN runner_orders r
	ON c.order_id = r.order_id
	LEFT JOIN pizza_names p
	ON c.pizza_id = p.pizza_id
	WHERE distance > 0),
	tes AS
	(SELECT record, VALUE AS 'extra_split'
	FROM tas
	CROSS APPLY STRING_SPLIT(extras, ',')),
	tis AS
	(SELECT record, extra_split, IIF(extra_split = '', 0, 1) AS 'extra_cost'
	FROM tes),
	tos AS
	(SELECT record, SUM(extra_cost) AS 'extra_costs'
	FROM tis
	GROUP BY record),
	tus AS
	(SELECT a.*, o.extra_costs
	FROM tas a
	LEFT JOIN tos o
	ON a.record = o.record)
SELECT SUM(pizza_cost_in_$ + extra_costs) AS 'Total Revenue(inclusive of extras revenue) for Pizza Runner in $'
FROM tus
```
![Alt text](<Pizza Runner pics/prr2.png>)


3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

```SQL
--0.order_id (track)
--1.On-time delivery (customer input)
--2.Delivery success (customer input)
--3.Order accuracy (customer input)
--4.Professionalism (customer input)
--5.Communication skills (customer input)
--6.Friendliness (customer input)
--7. Overall rating (customer input)

--Table schema generation
IF OBJECT_ID('customer_ratings', 'U') IS NOT NULL
DROP TABLE customer_ratings;
CREATE TABLE customer_ratings(
		runner_id INT,
		customer_id INT, 
		order_id INT,
		delivery_success BIT,
		on_time_delivery BIT,
		order_accuracy BIT,
		professionalism INT CHECK (professionalism < 11),
		communication_skills INT CHECK (communication_skills < 11),
		friendliness INT CHECK (friendliness < 11),
		overall_rating INT CHECK (overall_rating < 11) );

INSERT INTO customer_ratings
VALUES 
	  (1, 101, 1, 1, 0, 1, 6, 6, 9, 6),
	  (1, 101, 2, 1, 1, 1, 7, 6, 9, 7),
	  (1, 102, 3, 1, 1, 1, 8, 7, 10, 8),
	  (2, 103, 4, 1, 0, 1, 7, 5, 4, 5),
	  (3, 104, 5, 1, 1, 1, 6, 6, 8, 7);

SELECT *
FROM customer_ratings;
```
![Alt text](<Pizza Runner pics/prr4.png>)


4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
   - customer_id
   - order_id
   - runner_id
   - rating
   - order_time
   - pickup_time
   - Time between order and pickup
   - Delivery duration
   - Average speed
   - Total number of pizzas

```SQL
SELECT DISTINCT c.customer_id, c.order_id, r.runner_id, cr.overall_rating, c.order_time, r.pickup_time, 
				DATEDIFF(MINUTE, c.order_time, r.pickup_time) AS 'Time between order and pickup in minutes',
				r.duration, ROUND(CAST(r.distance AS FLOAT)/(CAST(r.duration AS FLOAT)/CAST('60' AS FLOAT)), 2) AS 'Speed in km/h',
				COUNT(c.order_id) OVER (PARTITION BY c.order_id ORDER BY c.order_id) AS 'Total number of pizzas per order'
FROM customer_orders c
LEFT JOIN runner_orders r
ON c.order_id = r.order_id
LEFT JOIN customer_ratings cr
ON c.order_id = cr.order_id
WHERE c.order_id < 6
```
![Alt text](<Pizza Runner pics/prr5.png>)


5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

```SQL
SELECT SUM(IIF(c.pizza_id = 1, 12, 10)) -   ((SELECT SUM(CAST(distance AS FLOAT)) FROM runner_orders WHERE order_id < 6) * 0.3 ) AS 'Total_money_left_for_Pizza_Runner_in_$'
FROM customer_orders c
LEFT JOIN runner_orders r
ON c.order_id = r.order_id
LEFT JOIN pizza_names p
ON c.pizza_id = p.pizza_id
WHERE c.order_id < 6 
```
![Alt text](<Pizza Runner pics/prr6.png>)



### E. Bonus Questions

If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

**IN PROGRESS**

