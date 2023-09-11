# Case Study Questions

The following questions can be considered key business questions and metrics that the Balanced Tree team requires for their monthly reports.

Each question can be answered using a single query - but as you are writing the SQL to solve each individual problem, keep in mind how you would generate all of these metrics in a single SQL script which the Balanced Tree team can run each month.


# High Level Sales Analysis

1. What was the total quantity sold for all products?

```SQL
SELECT SUM(qty) AS 'total quantity sold for all products'
FROM balanced_tree.sales
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs1.png>)


2. What is the total generated revenue for all products before discounts?

```SQL
SELECT SUM(price * qty) 'total generated revenue for all products before discounts'
FROM balanced_tree.sales
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs2.png>)


3. What was the total discount amount for all products?

```SQL
SELECT ROUND(SUM(CAST(discount AS FLOAT)/CAST(100 AS FLOAT) * CAST((price * qty) AS FLOAT)), 2) AS 'total discount amount for all products'
FROM balanced_tree.sales
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs3.png>)



# Transaction Analysis

1. How many unique transactions were there?

```SQL
SELECT COUNT(DISTINCT txn_id) AS 'number_unique_transactions'
FROM balanced_tree.sales
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs4.png>)

2. What is the average unique products purchased in each transaction?

```SQL
WITH las AS
	(SELECT DISTINCT txn_id, COUNT(prod_id) OVER (PARTITION BY txn_id ORDER BY txn_id RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) 
										AS '(number of) unique products'
	FROM balanced_tree.sales)
SELECT AVG([(number of) unique products]) AS 'average (number of) unique products purchased in each transaction'
FROM las
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs5.png>)

3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?

```SQL
WITH las AS
	(SELECT  SUM(price * qty * (CAST(100 - discount AS FLOAT)/100)) AS 'revenue per transaction after discount'
	FROM balanced_tree.sales
	GROUP BY txn_id)
SELECT DISTINCT PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY [revenue per transaction after discount] ) OVER () AS '25th percentile',
       PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY [revenue per transaction after discount] ) OVER () AS '50th percentile',
       PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY [revenue per transaction after discount] ) OVER () AS '75th percentile'
FROM las
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs6.png>)

4. What is the average discount value per transaction?

```SQL
WITH las AS
	(SELECT  SUM(price * qty * (CAST(discount AS FLOAT)/100)) AS 'discount value per transaction '
	FROM balanced_tree.sales
	GROUP BY txn_id)
SELECT ROUND(AVG([discount value per transaction]), 2) AS 'average discount value per transaction'
FROM las
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs7.png>)

5. What is the percentage split of all transactions for members vs non-members?

```SQL
WITH las AS
	(SELECT	DISTINCT txn_id, member
	 FROM balanced_tree.sales)
SELECT	ROUND(CAST(SUM(CASE
                          WHEN member = 't' THEN 1
                          ELSE 0
                       END) AS FLOAT)/CAST(COUNT(member) AS FLOAT) * 100, 2) AS 'percentage of transactions by members',
        ROUND(CAST(SUM(CASE
                          WHEN member = 'f' THEN 1
                          ELSE 0
                       END) AS FLOAT)/CAST(COUNT(member) AS FLOAT) * 100, 2) AS 'percentage of transactions by non-members'
FROM las;
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs8.png>)


6. What is the average revenue for member transactions and non-member transactions?

```SQL
WITH las AS
	(SELECT txn_id, member, SUM(price * qty * (CAST(100 - discount AS FLOAT)/100)) AS 'revenue per transaction after discount'
	FROM balanced_tree.sales
	GROUP BY txn_id, member)
SELECT CASE
          WHEN member = 't' THEN 'member'
          ELSE 'non-member'
       END AS 'membership status', 
       ROUND(AVG([revenue per transaction after discount]), 2) AS 'average revenue for member transactions and non-member transactions'
FROM las 
GROUP BY CASE
            WHEN member = 't' THEN 'member'
            ELSE 'non-member'
         END;
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs10.png>)


# Product Analysis

1. What are the top 3 products by total revenue before discount?

```SQL
SELECT TOP 3 p.product_name AS 'top 3 products by total revenue before discount' , SUM(s.price * s.qty) AS 'total revenue before discount'
FROM balanced_tree.sales s
LEFT JOIN balanced_tree.product_details p
ON s.prod_id = p.product_id
GROUP BY p.product_name
ORDER BY SUM(s.price * s.qty) DESC
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs11.png>)


2. What is the total quantity, revenue and discount for each segment?

```SQL
SELECT p.segment_name, SUM(s.qty) AS 'total quantity', 
       ROUND(SUM(s.price * s.qty * (CAST(100 - s.discount AS FLOAT)/100)), 2) AS 'total revenue', 
       ROUND(SUM(s.price * s.qty * (CAST(s.discount AS FLOAT)/100)), 2) AS 'total discount'
FROM balanced_tree.sales s
LEFT JOIN balanced_tree.product_details p
ON s.prod_id = p.product_id
GROUP BY p.segment_name
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs12.png>)


3. What is the top selling product for each segment?

```SQL
WITH las AS
	(SELECT p.segment_name, p.product_name, SUM(s.qty) AS 'top selling product for each segment'
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	GROUP BY p.segment_name, p.product_name),
	les AS
	(SELECT product_name,segment_name, [top selling product for each segment] ,
		   RANK() OVER (PARTITION BY segment_name ORDER BY [top selling product for each segment] DESC ) AS 'record'
	FROM las)
SELECT product_name,segment_name, [top selling product for each segment]
FROM les
WHERE record = 1
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs13.png>)


4. What is the total quantity, revenue and discount for each category?

```SQL
SELECT p.category_name, SUM(s.qty) AS 'total quantity', 
       ROUND(SUM(s.price * s.qty * (CAST(100 - s.discount AS FLOAT)/100)), 2) AS 'total revenue', 
       ROUND(SUM(s.price * s.qty * (CAST(s.discount AS FLOAT)/100)), 2) AS 'total discount'
FROM balanced_tree.sales s
LEFT JOIN balanced_tree.product_details p
ON s.prod_id = p.product_id
GROUP BY p.category_name
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs15.png>)


5. What is the top selling product for each category?

```SQL
WITH las AS
	(SELECT p.category_name, p.product_name, SUM(s.qty) AS 'top selling product for each category'
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	GROUP BY p.category_name, p.product_name),
	les AS
	(SELECT product_name,category_name, [top selling product for each category] ,
		   RANK() OVER (PARTITION BY category_name ORDER BY [top selling product for each category] DESC ) AS 'record'
	FROM las)
SELECT product_name,category_name, [top selling product for each category]
FROM les
WHERE record = 1
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs16.png>)


6. What is the percentage split of revenue by product for each segment?

```SQL
WITH las AS
	(SELECT p.product_name, p.segment_name, ROUND(SUM(s.price * s.qty * (CAST(100 - s.discount AS FLOAT)/100)), 2) AS 'revenue by product for each segment' 
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	GROUP BY p.product_name, p.segment_name)
SELECT product_name, segment_name, ROUND([revenue by product for each segment]/ 
                                   SUM([revenue by product for each segment]) OVER (PARTITION BY segment_name ORDER BY [revenue by product for each segment]
                                   RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) * 100 , 2) 
                                   AS 'percentage split of revenue by product for each segment'
FROM las
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs17.png>)


7. What is the percentage split of revenue by segment for each category?

```SQL
WITH las AS
	(SELECT p.category_name, p.segment_name, ROUND(SUM(s.price * s.qty * (CAST(100 - s.discount AS FLOAT)/100)), 2) AS 'revenue by segment for each category' 
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	GROUP BY p.category_name, p.segment_name)
SELECT category_name, segment_name, ROUND([revenue by segment for each category]/ 
                                    SUM([revenue by segment for each category]) OVER (PARTITION BY category_name ORDER BY [revenue by segment for each category]
                                    RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) * 100 , 2) 
                                    AS 'percentage split of revenue by segment for each category'
FROM las
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs18.png>)


8. What is the percentage split of total revenue by category?

```SQL
WITH las AS
	(SELECT p.category_name, ROUND(SUM(s.price * s.qty * (CAST(100 - s.discount AS FLOAT)/100)), 2) AS 'total revenue by category' 
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	GROUP BY p.category_name)
SELECT category_name, ROUND([total revenue by category] / SUM([total revenue by category]) OVER () * 100 , 2) AS 'percentage split of total revenue by category'
FROM las
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs19.png>)


9.  What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)

```SQL
WITH las AS
	(SELECT p.product_name, s.txn_id , COUNT(p.product_name) OVER (PARTITION BY s.txn_id, p.product_name ORDER BY s.txn_id) AS 'penetration'
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id)
SELECT DISTINCT product_name, CAST(SUM(penetration) OVER (PARTITION BY product_name ORDER BY txn_id RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS FLOAT)/
                              CAST((SELECT COUNT(DISTINCT txn_id) FROM las) AS FLOAT) * 100 AS 'penetration rate'
FROM las
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs20.png>)


10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

```SQL
WITH las AS
	(SELECT p.product_name, s.txn_id 
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id),
	les AS
	(SELECT DISTINCT product_name
	FROM las),
	lis AS
	(SELECT l.product_name AS product_name_1 , e.product_name AS product_name_2, s.product_name AS product_name_3,
	        ROW_NUMBER() OVER (ORDER BY l.product_name, e.product_name, s.product_name) AS 'record'
	FROM les l
	CROSS JOIN les e
	CROSS JOIN les s
	WHERE l.product_name < e.product_name  AND l.product_name < s.product_name AND e.product_name < s.product_name),
	los AS
	(SELECT *
	FROM lis
	CROSS JOIN las),
	lus AS
	(SELECT txn_id, record, CASE
	                           WHEN product_name = product_name_1 OR product_name = product_name_2 OR product_name = product_name_3 THEN 1
	                           ELSE 0
	                        END AS 'combination counter'
	FROM los),
	kas AS
	(SELECT *, SUM([combination counter]) OVER (PARTITION BY txn_id, record ORDER BY txn_id RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'decider'
	FROM lus),
	kes AS
	(SELECT DISTINCT txn_id, record, decider
	FROM kas),
	kis AS
	(SELECT record, SUM(CASE
	                       WHEN decider = 3 THEN 1
	                       ELSE 0
	                    END) AS 'combination frequency'
	FROM kes
	GROUP BY record)
SELECT TOP 1 l.product_name_1, l.product_name_2, l.product_name_3, l.record, k.[combination frequency] AS 'most common combination of at least 1 quantity of any 3 products in a 1 single transaction'
FROM lis l
LEFT JOIN kis k
ON l.record = k.record
ORDER BY k.[combination frequency] DESC
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs21.png>)



# Reporting Challenge

Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.

Imagine that the Chief Financial Officer (which is also Danny) has asked for all of these questions at the end of every month.

He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the samne analysis for February without many changes (if at all).

Feel free to split up your final outputs into as many tables as you need - but be sure to explicitly reference which table outputs relate to which question for full marks.

```SQL
CREATE PROCEDURE balanced_tree.Month_Report @Month_Number INT
AS
BEGIN
	SELECT	SUM(qty) AS 'total quantity sold for all products',
			SUM(price * qty) 'total generated revenue for all products before discounts',
			ROUND(SUM(CAST(discount AS FLOAT)/CAST(100 AS FLOAT) * CAST((price * qty) AS FLOAT)), 2) AS 'total discount amount for all products'	   
	FROM balanced_tree.sales
	WHERE MONTH(start_txn_time) =  @Month_Number;

	SELECT COUNT(DISTINCT txn_id) AS 'number_unique_transactions'
	FROM balanced_tree.sales
	WHERE MONTH(start_txn_time) =  @Month_Number;

	WITH las AS
	(SELECT DISTINCT txn_id, COUNT(prod_id) OVER (PARTITION BY txn_id ORDER BY txn_id RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) 
										AS '(number of) unique products'
	FROM balanced_tree.sales
	WHERE MONTH(start_txn_time) =  @Month_Number)
	SELECT AVG([(number of) unique products]) AS 'average (number of) unique products purchased in each transaction'
	FROM las;

	WITH les AS
	(SELECT  SUM(price * qty * (CAST(100 - discount AS FLOAT)/100)) AS 'revenue per transaction after discount'
	FROM balanced_tree.sales
	WHERE MONTH(start_txn_time) =  @Month_Number
	GROUP BY txn_id)
	SELECT DISTINCT PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY [revenue per transaction after discount] ) OVER () AS '25th percentile',
		   PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY [revenue per transaction after discount] ) OVER () AS '50th percentile',
		   PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY [revenue per transaction after discount] ) OVER () AS '75th percentile'
	FROM les;

	WITH lis AS
	(SELECT  SUM(price * qty * (CAST(discount AS FLOAT)/100)) AS 'discount value per transaction '
	FROM balanced_tree.sales
	WHERE MONTH(start_txn_time) =  @Month_Number
	GROUP BY txn_id)
	SELECT ROUND(AVG([discount value per transaction]), 2) AS 'average discount value per transaction'
	FROM lis;

	WITH los AS
	(SELECT	DISTINCT txn_id, member
	 FROM balanced_tree.sales
	 WHERE MONTH(start_txn_time) =  @Month_Number)
	 SELECT	ROUND(CAST(SUM(CASE
						WHEN member = 't' THEN 1
						ELSE 0
					 END) AS FLOAT)/CAST(COUNT(member) AS FLOAT) * 100, 2) AS 'percentage of transactions by members',
			ROUND(CAST(SUM(CASE
						WHEN member = 'f' THEN 1
						ELSE 0
					 END) AS FLOAT)/CAST(COUNT(member) AS FLOAT) * 100, 2) AS 'percentage of transactions by non-members'
	 FROM los;

	 WITH lus AS
	(SELECT txn_id, member, SUM(price * qty * (CAST(100 - discount AS FLOAT)/100)) AS 'revenue per transaction after discount'
	FROM balanced_tree.sales
	WHERE MONTH(start_txn_time) =  @Month_Number
	GROUP BY txn_id, member)
	SELECT CASE
				WHEN member = 't' THEN 'member'
				ELSE 'non-member'
			END AS 'membership status', 
			ROUND(AVG([revenue per transaction after discount]), 2) AS 'average revenue for member transactions and non-member transactions'
	FROM lus 
	GROUP BY CASE
				WHEN member = 't' THEN 'member'
				ELSE 'non-member'
			END;

	SELECT TOP 3 p.product_name AS 'top 3 products by total revenue before discount', SUM(s.price * s.qty) AS 'total revenue before discount'
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	WHERE MONTH(start_txn_time) =  @Month_Number
	GROUP BY p.product_name
	ORDER BY SUM(s.price * s.qty) DESC;

	SELECT p.segment_name, SUM(s.qty) AS 'total quantity', 
		ROUND(SUM(s.price * s.qty * (CAST(100 - s.discount AS FLOAT)/100)), 2) AS 'total revenue', 
		ROUND(SUM(s.price * s.qty * (CAST(s.discount AS FLOAT)/100)), 2) AS 'total discount'
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	WHERE MONTH(start_txn_time) =  @Month_Number
	GROUP BY p.segment_name;

	WITH kas AS
	(SELECT p.segment_name, p.product_name, SUM(s.qty) AS 'top selling product for each segment'
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	WHERE MONTH(start_txn_time) =  @Month_Number
	GROUP BY p.segment_name, p.product_name),
	kes AS
	(SELECT product_name,segment_name, [top selling product for each segment] ,
		   RANK() OVER (PARTITION BY segment_name ORDER BY [top selling product for each segment] DESC ) AS 'record'
	FROM kas)
	SELECT product_name,segment_name, [top selling product for each segment]
	FROM kes
	WHERE record = 1;

	SELECT p.category_name, SUM(s.qty) AS 'total quantity', 
		ROUND(SUM(s.price * s.qty * (CAST(100 - s.discount AS FLOAT)/100)), 2) AS 'total revenue', 
		ROUND(SUM(s.price * s.qty * (CAST(s.discount AS FLOAT)/100)), 2) AS 'total discount'
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	WHERE MONTH(start_txn_time) =  @Month_Number
	GROUP BY p.category_name;

	WITH kis AS
	(SELECT p.category_name, p.product_name, SUM(s.qty) AS 'top selling product for each category'
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	WHERE MONTH(start_txn_time) =  @Month_Number
	GROUP BY p.category_name, p.product_name),
	kos AS
	(SELECT product_name,category_name, [top selling product for each category] ,
		   RANK() OVER (PARTITION BY category_name ORDER BY [top selling product for each category] DESC ) AS 'record'
	FROM kis)
	SELECT product_name,category_name, [top selling product for each category]
	FROM kos
	WHERE record = 1;

	WITH mas AS
	(SELECT p.product_name, p.segment_name, ROUND(SUM(s.price * s.qty * (CAST(100 - s.discount AS FLOAT)/100)), 2) AS 'revenue by product for each segment' 
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	WHERE MONTH(start_txn_time) =  @Month_Number
	GROUP BY p.product_name, p.segment_name)
	SELECT product_name, segment_name, ROUND([revenue by product for each segment]/ 
										SUM([revenue by product for each segment]) OVER (PARTITION BY segment_name ORDER BY [revenue by product for each segment]
										RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) * 100 , 2) 
										AS 'percentage split of revenue by product for each segment'
	FROM mas;

	WITH mes AS
	(SELECT p.category_name, p.segment_name, ROUND(SUM(s.price * s.qty * (CAST(100 - s.discount AS FLOAT)/100)), 2) AS 'revenue by segment for each category' 
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	WHERE MONTH(start_txn_time) =  @Month_Number
	GROUP BY p.category_name, p.segment_name)
	SELECT category_name, segment_name, ROUND([revenue by segment for each category]/ 
										SUM([revenue by segment for each category]) OVER (PARTITION BY category_name ORDER BY [revenue by segment for each category]
										RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) * 100 , 2) 
										AS 'percentage split of revenue by segment for each category'
	FROM mes;

	WITH mis AS
	(SELECT p.category_name, ROUND(SUM(s.price * s.qty * (CAST(100 - s.discount AS FLOAT)/100)), 2) AS 'total revenue by category' 
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	WHERE MONTH(start_txn_time) =  @Month_Number
	GROUP BY p.category_name)
	SELECT category_name, ROUND([total revenue by category] / SUM([total revenue by category]) OVER () * 100 , 2) 
										AS 'percentage split of total revenue by category'
	FROM mis;

	WITH mos AS
	(SELECT p.product_name, s.txn_id , COUNT(p.product_name) OVER (PARTITION BY s.txn_id, p.product_name ORDER BY s.txn_id) AS 'penetration'
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	WHERE MONTH(start_txn_time) =  @Month_Number)
	SELECT DISTINCT product_name, CAST(SUM(penetration) OVER (PARTITION BY product_name ORDER BY txn_id
														 RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS FLOAT)/
								  CAST((SELECT COUNT(DISTINCT txn_id) FROM mos) AS FLOAT) * 100 AS 'penetration rate'
	FROM mos;

	WITH nas AS
	(SELECT p.product_name, s.txn_id 
	FROM balanced_tree.sales s
	LEFT JOIN balanced_tree.product_details p
	ON s.prod_id = p.product_id
	WHERE MONTH(start_txn_time) =  @Month_Number),
	nes AS
	(SELECT DISTINCT product_name
	FROM nas),
	nis AS
	(SELECT l.product_name AS product_name_1 , e.product_name AS product_name_2, s.product_name AS product_name_3,
				  ROW_NUMBER() OVER (ORDER BY l.product_name, e.product_name, s.product_name) AS 'record'
	FROM nes l
	CROSS JOIN nes e
	CROSS JOIN nes s
	WHERE l.product_name < e.product_name  AND l.product_name < s.product_name AND e.product_name < s.product_name),
	nos AS
	(SELECT *
	FROM nis
	CROSS JOIN nas),
	nus AS
	(SELECT txn_id, record, CASE
								WHEN product_name = product_name_1 OR product_name = product_name_2 OR product_name = product_name_3 THEN 1
								ELSE 0
							END AS 'combination counter'
	FROM nos),
	pas AS
	(SELECT *, SUM([combination counter]) OVER (PARTITION BY txn_id, record ORDER BY txn_id RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'decider'
	FROM nus),
	pes AS
	(SELECT DISTINCT txn_id, record, decider
	FROM pas),
	pis AS
	(SELECT record, SUM(CASE
						  WHEN decider = 3 THEN 1
						  ELSE 0
					   END) AS 'combination frequency'
	FROM pes
	GROUP BY record)
	SELECT TOP 1 l.product_name_1, l.product_name_2, l.product_name_3, k.[combination frequency] AS 'most common combination of at least 1 quantity of any 3 products in a 1 single transaction frequency count'
	FROM nis l
	LEFT JOIN pis k
	ON l.record = k.record
	ORDER BY k.[combination frequency] DESC;

END;
```
When running a report for a certain month, one just needs to just enter the month number upon executing this single line 
of code:

```SQL
EXEC balanced_tree.Month_Report @Month_Number = 1;
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs24.png>)
![Alt text](<Balanced Tree Clothing Co. pics/btcs25.png>)



# Bonus Challenge

Use a single SQL query to transform the product_hierarchy and product_prices datasets to the product_details table.

Hint: you may want to consider using a recursive CTE to solve this problem!


```SQL
WITH product_remake AS
		(	
			SELECT CAST('-' AS VARCHAR(50)) AS 'product_name', id AS 'category_id', id AS 'segment_id',
			       CAST(NULL AS INT) 'style_id', level_text AS 'category_name', CAST('' AS VARCHAR(10)) AS 'segment_name',
			       CAST('' AS VARCHAR(30)) AS 'style_name', id AS 'id' 
			FROM balanced_tree.product_hierarchy 
			WHERE parent_id IS NULL

			UNION ALL

			SELECT CAST(CASE 
			               WHEN r.product_name = '-' THEN h.level_text + ' ' + r.product_name + ' ' + r.category_name
			               ELSE h.level_text + ' ' + r.product_name
			            END AS VARCHAR(50)) AS 'product_name',
			       r.category_id AS 'category_id', h.parent_id 'segment_id', h.id AS 'style_id',
			       r.category_name AS 'category_name', 
				   CAST( CASE
			                WHEN r.segment_name = '' THEN h.level_text
			                ELSE r.segment_name
			             END AS VARCHAR(10)) AS 'segment_name', 
			       CAST(h.level_text AS VARCHAR(30)) AS 'style_name', h.id AS 'id'
			FROM balanced_tree.product_hierarchy h
			INNER JOIN product_remake r
			ON h.parent_id = r.id)
SELECT p.product_id, p.price, r.product_name, r.category_id, r.segment_id, r.style_id, r.category_name, r.segment_name, r.style_name
FROM product_remake r
INNER JOIN balanced_tree.product_prices p
ON r.style_id = p.id
```
![Alt text](<Balanced Tree Clothing Co. pics/btcs23.png>)