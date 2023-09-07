# Case Study Questions

Using the following DDL schema details to create an ERD for all the Clique Bait datasets.

Click here to access the DB Diagram tool to create the ERD.

![Alt text](<clique bait pics/cb6.png>)

### Clique Bait ERD

![Alt text](<clique bait pics/cb erd.png>)



# 2. Digital Analysis

Using the available datasets - answer the following questions using a single query for each one:

1. How many users are there?

```SQL
SELECT COUNT(DISTINCT usser_id) AS 'number_of_users'
FROM clique_bait.users
```
![Alt text](<clique bait pics/cbs1.png>)


2. How many cookies does each user have on average?

```SQL
WITH las AS
	(SELECT usser_id, COUNT(cookie_id) AS 'number_of_cookies_per_user'
	FROM clique_bait.users
	GROUP BY usser_id)
SELECT ROUND(AVG(CAST(number_of_cookies_per_user AS FLOAT)), 0) AS 'average_number_of_cookies_per_user'
FROM las
```
![Alt text](<clique bait pics/cbs3.png>)


3. What is the unique number of visits by all users per month?

```SQL
SELECT  MONTH(e.event_time) AS 'event_month', COUNT(DISTINCT e.visit_id) AS 'unique number of visits by all users per month'
FROM clique_bait.eventss e
LEFT JOIN clique_bait.users u
ON e.cookie_id = u.cookie_id
GROUP BY MONTH(e.event_time) 
ORDER BY  MONTH(e.event_time)
```
![Alt text](<clique bait pics/cbs4.png>)


4. What is the number of events for each event type?

```SQL
SELECT e.event_type, i.event_name, COUNT(e.event_type) AS 'number_of_events_for_each_event_type'
FROM clique_bait.eventss e
LEFT JOIN clique_bait.event_identifier i
ON e.event_type = i.event_type
GROUP BY e.event_type, i.event_name
ORDER BY e.event_type, i.event_name
```
![Alt text](<clique bait pics/cbs5.png>)


5. What is the percentage of visits which have a purchase event?

```SQL
WITH las AS
	(SELECT COUNT(visit_id) OVER (PARTITION BY visit_id, cookie_id ORDER BY cookie_id) 'countery'
	FROM clique_bait.eventss
	WHERE event_type = 3)
SELECT ROUND(CAST(COUNT(countery) AS FLOAT)/CAST((SELECT COUNT(DISTINCT visit_id) FROM clique_bait.eventss) AS FLOAT) * 100 , 2) AS 'percentage of visits which have a purchase event'
FROM las
```
![Alt text](<clique bait pics/cbs7.png>)


6. What is the percentage of visits which view the checkout page but do not have a purchase event?

```SQL
WITH las AS
	(SELECT * 
	FROM clique_bait.eventss
	WHERE visit_id IN	(SELECT visit_id  
	                     FROM clique_bait.eventss
	                     WHERE page_id = 12
	                     GROUP BY visit_id, cookie_id) 
	),
	les AS
	(SELECT COUNT(visit_id) OVER (PARTITION BY visit_id, cookie_id ORDER BY visit_id) AS 'counting'
	FROM las
	WHERE event_type = 3)
SELECT ROUND(((CAST((SELECT COUNT(DISTINCT visit_id) FROM las) AS FLOAT) - CAST(COUNT(counting) AS FLOAT))/ 
					CAST((SELECT COUNT(DISTINCT visit_id) FROM las) AS FLOAT)) * 100, 2) AS 'percentage of visits which view the checkout page but do not have a purchase event'
FROM les
```
![Alt text](<clique bait pics/cbs8.png>)


7. What are the top 3 pages by number of views?

```SQL
SELECT TOP 3 page_id, COUNT(page_id) AS 'number of views of the page'
FROM clique_bait.eventss
WHERE event_type = 1
GROUP BY page_id
ORDER BY COUNT(page_id) DESC
```
![Alt text](<clique bait pics/cbs9.png>)


8. What is the number of views and cart adds for each product category?

```SQL
WITH las AS
	(SELECT e.*, p.page_name, p.product_category, p.product_id
	FROM clique_bait.eventss e
	LEFT JOIN clique_bait.page_hierarchy p
	ON e.page_id = p.page_id
	WHERE e.page_id IN (SELECT page_id FROM clique_bait.page_hierarchy WHERE product_category IS NOT NULL))
SELECT product_category, CASE 
	                         WHEN event_type = 1 THEN 'page view'
	                         ELSE 'cart add'
	                     END AS 'event type', COUNT(product_category) 'count of event'  
FROM las
WHERE event_type IN (1, 2)
GROUP BY product_category, CASE
	                          WHEN event_type = 1 THEN 'page view'
	                          ELSE 'cart add'
	                       END 
```
![Alt text](<clique bait pics/cbs10.png>)


9.  What are the top 3 products by purchases?

```SQL
WITH las AS
	(SELECT * 
	FROM clique_bait.eventss
	WHERE visit_id IN	(SELECT visit_id  
	                     FROM clique_bait.eventss
	                     WHERE page_id = 13 AND event_type = 3
	                     GROUP BY visit_id, cookie_id)),
	les AS
	(SELECT l.*, p.page_name, p.product_category, p.product_id
	FROM las l
	LEFT JOIN clique_bait.page_hierarchy p
	ON l.page_id = p.page_id
	WHERE l.page_id IN (SELECT page_id FROM clique_bait.page_hierarchy WHERE product_category IS NOT NULL) AND event_type = 2)
SELECT TOP 3 page_name, COUNT(page_name) AS 'purchased products'
FROM les
GROUP BY page_name
ORDER BY COUNT(page_name) DESC
```
![Alt text](<clique bait pics/cbs11.png>)



# 3. Product Funnel Analysis

Using a single SQL query - create a new output table which has the following details:

   - How many times was each product viewed?
   - How many times was each product added to cart?
   - How many times was each product added to a cart but not purchased (abandoned)?
   - How many times was each product purchased?

```SQL
SELECT  p.page_name, 
			SUM(CASE
			       WHEN e.event_type = 1 THEN 1
			       ELSE 0
			    END) AS 'product views', 
			SUM(CASE
			       WHEN e.event_type = 2 THEN 1
			       ELSE 0
			    END) AS 'product cart add',
			SUM(CASE
			       WHEN e.event_type = 2 AND e.lasst_value <> 3
			          THEN 1
			       ELSE 0
			    END) AS 'product abandoned',
			SUM(CASE
			       WHEN e.event_type = 2 AND e.lasst_value = 3
			          THEN 1
			       ELSE 0
			    END) AS 'product purchased'	 
	INTO clique_bait.temp_product
FROM (SELECT *  , LAST_VALUE(event_type) OVER (PARTITION BY visit_id, cookie_id ORDER BY visit_id, cookie_id, sequence_number RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'lasst_value'
	  FROM clique_bait.eventss) e
LEFT JOIN clique_bait.page_hierarchy p
ON e.page_id = p.page_id
WHERE e.page_id IN (SELECT page_id FROM clique_bait.page_hierarchy WHERE product_category IS NOT NULL)
GROUP BY p.page_name;

SELECT *
FROM clique_bait.temp_product;
```
![Alt text](<clique bait pics/cbs14.png>)
  
Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

```SQL
SELECT  p.product_category, 
		SUM(CASE
		       WHEN e.event_type = 1 THEN 1
		       ELSE 0
		    END) AS 'product_category_views', 
		SUM(CASE
		       WHEN e.event_type = 2 THEN 1
		       ELSE 0
		    END) AS 'product_category_cart_add',
		SUM(CASE
		       WHEN e.event_type = 2 AND e.lasst_value <> 3
		          THEN 1
		       ELSE 0
		    END) AS 'product_category_abandoned',
		SUM(CASE
		       WHEN e.event_type = 2 AND e.lasst_value = 3
		          THEN 1
		       ELSE 0
		    END) AS 'product_category_purchased'	 
	INTO clique_bait.temp_product_category
FROM (SELECT *  , LAST_VALUE(event_type) OVER (PARTITION BY visit_id, cookie_id ORDER BY visit_id, cookie_id, sequence_number RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'lasst_value'
	  FROM clique_bait.eventss) e
LEFT JOIN clique_bait.page_hierarchy p
ON e.page_id = p.page_id
WHERE e.page_id IN (SELECT page_id FROM clique_bait.page_hierarchy WHERE product_category IS NOT NULL)
GROUP BY p.product_category;

SELECT *
FROM clique_bait.temp_product_category;
```
![Alt text](<clique bait pics/cbs15.png>)


Use your 2 new output tables - answer the following questions:

1. Which product had the most views, cart adds and purchases?

```SQL
SELECT *
FROM
	(SELECT TOP 1 page_name, [product views], 'product had the most views' AS 'event'
	FROM clique_bait.temp_product
	ORDER BY [product views] DESC) a
UNION ALL
SELECT *
FROM
	(SELECT TOP 1 page_name, [product cart add], 'product had the most cart adds' AS 'event'
	FROM clique_bait.temp_product
	ORDER BY [product cart add] DESC) b
UNION ALL
SELECT *
FROM
	(SELECT TOP 1 page_name, [product purchased], 'product had the most purchases' AS 'event'
	FROM clique_bait.temp_product
	ORDER BY [product purchased] DESC) c
```
![Alt text](<clique bait pics/cbs16.png>)


2. Which product was most likely to be abandoned?

```SQL
SELECT TOP 1 page_name, [product abandoned], 'product was most likely to be abandoned' AS 'event'
FROM clique_bait.temp_product
ORDER BY [product abandoned] DESC
```
![Alt text](<clique bait pics/cbs17.png>)

3. Which product had the highest view to purchase percentage?

```SQL
SELECT TOP 1 page_name, ROUND(CAST([product purchased] AS FLOAT)/CAST([product views] AS FLOAT) * 100, 2) AS 'view to purchase percentage'
FROM clique_bait.temp_product
ORDER BY ROUND(CAST([product purchased] AS FLOAT)/CAST([product views] AS FLOAT) * 100, 2) DESC
```
![Alt text](<clique bait pics/cbs18.png>)

4. What is the average conversion rate from view to cart add?

```SQL
SELECT ROUND(AVG(CAST([product cart add] AS FLOAT)/CAST([product views] AS FLOAT) * 100) , 2) AS 'average conversion rate from view to cart add'
FROM clique_bait.temp_product
```
![Alt text](<clique bait pics/cbs23.png>)

5. What is the average conversion rate from cart add to purchase?

```SQL
SELECT ROUND(AVG(CAST([product purchased] AS FLOAT)/CAST([product cart add] AS FLOAT) * 100), 2) AS 'average conversion rate from cart add to purchase'
FROM clique_bait.temp_product
```
![Alt text](<clique bait pics/cbs20.png>)



# 3. Campaigns Analysis

Generate a table that has 1 single row for every unique visit_id record and has the following columns:

   - user_id
   - visit_id
   - visit_start_time: the earliest event_time for each visit
   - page_views: count of page views for each visit
   - cart_adds: count of product cart add events for each visit
   - purchase: 1/0 flag if a purchase event exists for each visit
   - campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
    impression: count of ad impressions for each visit
   - click: count of ad clicks for each visit
   - (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

```SQL
WITH las AS
	(SELECT e.*, p.page_name, u.usser_id,
	        FIRST_VALUE(e.event_time) OVER (PARTITION BY e.visit_id, e.cookie_id ORDER BY e.visit_id, e.cookie_id, e.sequence_number RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)  AS 'start_time',
	        CASE
	           WHEN LAST_VALUE(e.event_type) OVER (PARTITION BY e.visit_id, e.cookie_id ORDER BY e.visit_id, e.cookie_id, e.sequence_number RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) = 3
	                THEN 1 
	           ELSE 0
	        END AS 'purchase'
	FROM clique_bait.eventss e
	LEFT JOIN clique_bait.page_hierarchy p
	ON e.page_id = p.page_id
	LEFT JOIN clique_bait.users u
	ON e.cookie_id = u.cookie_id)
SELECT l.usser_id, l.visit_id, l.start_time, 
       SUM(CASE
              WHEN l.event_type = 1 THEN 1
              ELSE 0
           END) AS 'page_views',
       SUM(CASE
              WHEN l.event_type = 2 THEN 1
              ELSE 0
           END) AS 'cart_adds',
       l.purchase, c.campaign_name,
       SUM(CASE
              WHEN l.event_type = 4 THEN 1
              ELSE 0
           END) AS 'impression',
       SUM(CASE
              WHEN l.event_type = 5 THEN 1
              ELSE 0
           END) AS 'click',
       STRING_AGG(CASE
                     WHEN l.event_type = 2 THEN l.page_name
                  END, ',') WITHIN GROUP (ORDER BY l.sequence_number ASC) AS 'cart_products'								
FROM las l
LEFT JOIN clique_bait.campaign_identifier c
ON l.start_time >= c.start_date AND l.start_time <= c.end_date
GROUP BY l.usser_id, l.visit_id, l.start_time, l.purchase, c.campaign_name
ORDER BY l.usser_id
```
![Alt text](<clique bait pics/cbs21.png>)

Use the subsequent dataset to generate at least 5 insights for the Clique Bait team - bonus: prepare a single A4 infographic that the team can use for their management reporting sessions, be sure to emphasise the most important points from your findings.

Some ideas you might want to investigate further include:

   - Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event
   - Does clicking on an impression lead to higher purchase rates?
   - What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?
   - What metrics can you use to quantify the success or failure of each campaign compared to eachother?

**[TO BE COMPLETED]**