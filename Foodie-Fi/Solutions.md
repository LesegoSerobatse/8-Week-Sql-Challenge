### A. Customer Journey

Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

[To be completed ]


### B. Data Analysis Questions

1. How many customers has Foodie-Fi ever had?

```SQL
SELECT COUNT(DISTINCT customer_id) AS 'No. of customers Foodie-Fi ever had'
FROM subscriptions
```
![Alt text](<Foodie-Fi pics/ffs1.png>)


2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

```SQL
SELECT DATEPART(MONTH, start_date) AS 'Month number', '1st'+ ' ' + DATENAME(MONTH, start_date) AS 'Month', COUNT(DATENAME(MONTH, start_date)) AS 'Number of new subscriptions on trial'
FROM subscriptions
WHERE plan_id = 0 
GROUP BY DATENAME(MONTH, start_date), DATEPART(MONTH, start_date)
ORDER BY DATEPART(MONTH, start_date)
```
![Alt text](<Foodie-Fi pics/ffs2.png>)


3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

```SQL
SELECT s.plan_id, p.plan_name, COUNT(p.plan_name) AS 'Count per plan after the year 2020'
FROM subscriptions s
LEFT JOIN plans p
ON s.plan_id = p.plan_id
WHERE DATEPART(YEAR, s.start_date) > 2020 
GROUP BY s.plan_id, p.plan_name
ORDER BY s.plan_id
```
![Alt text](<Foodie-Fi pics/ffs3.png>)


4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

```SQL
SELECT COUNT(s.customer_id) AS 'Customer count', ROUND(((CAST(COUNT(s.customer_id) AS FLOAT)/ (SELECT COUNT(DISTINCT customer_id) FROM subscriptions)) * 100), 1)  AS 'percentage of customers'
FROM subscriptions s
LEFT JOIN plans p
ON s.plan_id = p.plan_id
WHERE p.plan_name = 'churn'
```
![Alt text](<Foodie-Fi pics/ffs4.png>)


5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

```SQL
WITH las AS
	(SELECT customer_id, STRING_AGG(CAST(plan_id AS VARCHAR(20)), ',') AS 'customers_churned_after_trial'
	FROM subscriptions
	GROUP BY customer_id
	HAVING STRING_AGG(CAST(plan_id AS VARCHAR(20)), ',') = '0,4')
SELECT COUNT(customers_churned_after_trial) AS 'Number_of_customers_churned_after_trial', 
	ROUND((CAST((COUNT(customers_churned_after_trial)) AS FLOAT)/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions)) * 100, 0) AS 'Percentage_of_customers_churned_after_trial'
FROM las;
```
![Alt text](<Foodie-Fi pics/ffs5.png>)


6. What is the number and percentage of customer plans after their initial free trial?

```SQL
WITH kas AS
	(SELECT customer_id, SUBSTRING((STRING_AGG(CAST(plan_id AS VARCHAR(20)), ',')), 1, 3) AS 'customers_plans_after_trial_B'
	FROM subscriptions
	GROUP BY customer_id)
SELECT CASE
		 WHEN customers_plans_after_trial_B = '0,1' THEN 'Upgraded to basic monthly plan'
		 WHEN customers_plans_after_trial_B = '0,2' THEN 'Upgraded to pro monthly plan'
		 WHEN customers_plans_after_trial_B = '0,3' THEN 'Upgraded to pro annual plan'
		 WHEN customers_plans_after_trial_B = '0,4' THEN 'degraded to churn plan'
	   END AS 'customers_plans_after_trial_period',customers_plans_after_trial_B,
	ROUND((CAST((COUNT(customers_plans_after_trial_B)) AS FLOAT)/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions)) * 100, 2) AS 'Percentage_of_customers_after_trial'
FROM kas
GROUP BY customers_plans_after_trial_B
```
![Alt text](<Foodie-Fi pics/ffs6.png>)


7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

```SQL
WITH las AS
	(SELECT s.customer_id, STRING_AGG(CAST(s.plan_id AS VARCHAR(20)), ',') AS 'plans_by_year_end_2020'
	FROM subscriptions s
	LEFT JOIN plans p
	ON s.plan_id = p.plan_id
	WHERE s.start_date <= '2020-12-31'
	GROUP BY s.customer_id),
	les AS
	(SELECT customer_id, SUBSTRING(plans_by_year_end_2020, LEN(plans_by_year_end_2020), 1) AS 'year_end_plans'
	FROM las),
	lis AS
	(SELECT year_end_plans, COUNT(year_end_plans) AS 'plans_breakdoown_by_customer_number'
	FROM les
	GROUP BY year_end_plans)
SELECT p.plan_name, l.year_end_plans, l.plans_breakdoown_by_customer_number,
(CAST(l.plans_breakdoown_by_customer_number AS FLOAT)/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions)) * 100 AS 'Percentage breakdown of customers'
FROM lis l
LEFT JOIN plans p
ON l.year_end_plans = p.plan_id
GROUP BY p.plan_name, l.year_end_plans, l.plans_breakdoown_by_customer_number
ORDER BY l.year_end_plans
```
![Alt text](<Foodie-Fi pics/ffs7.png>)


8. How many customers have upgraded to an annual plan in 2020?

```SQL
WITH las AS
	(SELECT s.customer_id, STRING_AGG(CAST(s.plan_id AS VARCHAR(20)), ',') AS 'plans_by_year_end_2020'
	FROM subscriptions s
	LEFT JOIN plans p
	ON s.plan_id = p.plan_id
	WHERE s.start_date <= '2020-12-31'
	GROUP BY s.customer_id)
SELECT COUNT(plans_by_year_end_2020) AS 'Number_of_customers_upgraded_to_annual_plan'
FROM las
WHERE plans_by_year_end_2020 LIKE '%3%'
```
![Alt text](<Foodie-Fi pics/ffs8.png>)


9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

```SQL
WITH tas AS
	(SELECT s.*, p.plan_name, p.price
	FROM subscriptions s
	LEFT JOIN plans p
	ON s.plan_id = p.plan_id
	WHERE s.plan_id IN (0,3)),
	tes AS
	(SELECT customer_id, STRING_AGG(CAST(plan_id AS VARCHAR(20)), ',') AS 'plans_by_year_', STRING_AGG(CAST(start_date AS VARCHAR(100)), ',') AS 'Dates'
	FROM tas
	GROUP BY customer_id),
	tis AS
	(SELECT customer_id, plans_by_year_, Dates
	FROM tes
	WHERE plans_by_year_ LIKE '%3%'),
	tos AS
	(SELECT customer_id, plans_by_year_, Dates, SUBSTRING(Dates, 1, CHARINDEX(',', Dates) - 1) AS date_start, SUBSTRING(Dates, CHARINDEX(',', Dates) + 1, LEN(Dates)) AS date_end
	FROM tis),
	tus AS
	(SELECT customer_id, Dates, DATEDIFF(DAY, CAST(date_start AS DATE), CAST(date_end AS DATE)) AS 'Number_of_days_to_annual_plan_from_trial'
	FROM tos)
SELECT AVG(CAST(Number_of_days_to_annual_plan_from_trial AS FLOAT)) AS 'Average_number_of_days_to_annual_plan_from_trial'
FROM tus
```
![Alt text](<Foodie-Fi pics/ffs9.png>)


10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

```SQL
WITH tas AS
	(SELECT s.*, p.plan_name, p.price
	FROM subscriptions s
	LEFT JOIN plans p
	ON s.plan_id = p.plan_id
	WHERE s.plan_id IN (0,3)),
	tes AS
	(SELECT customer_id, STRING_AGG(CAST(plan_id AS VARCHAR(20)), ',') AS 'plans_by_year_', STRING_AGG(CAST(start_date AS VARCHAR(100)), ',') AS 'Dates'
	FROM tas
	GROUP BY customer_id),
	tis AS
	(SELECT customer_id, plans_by_year_, Dates
	FROM tes
	WHERE plans_by_year_ LIKE '%3%'),
	tos AS
	(SELECT customer_id, plans_by_year_, Dates, SUBSTRING(Dates, 1, CHARINDEX(',', Dates) - 1) AS date_start, SUBSTRING(Dates, CHARINDEX(',', Dates) + 1, LEN(Dates)) AS date_end
	FROM tis),
	tus AS
	(SELECT customer_id, Dates, DATEDIFF(DAY, CAST(date_start AS DATE), CAST(date_end AS DATE)) AS 'Number_of_days_to_annual_plan_from_trial'
	FROM tos),
	tuus AS
	(SELECT Number_of_days_to_annual_plan_from_trial, 
		CASE
			WHEN Number_of_days_to_annual_plan_from_trial <= 30 THEN '0-30 days'
			WHEN Number_of_days_to_annual_plan_from_trial >= 31 AND Number_of_days_to_annual_plan_from_trial <= 60 THEN '31-60 days'
			WHEN Number_of_days_to_annual_plan_from_trial >= 61 AND Number_of_days_to_annual_plan_from_trial <= 90 THEN '61-90 days'
			WHEN Number_of_days_to_annual_plan_from_trial >= 91 AND Number_of_days_to_annual_plan_from_trial <= 120 THEN '91-120 days'
			WHEN Number_of_days_to_annual_plan_from_trial >= 121 AND Number_of_days_to_annual_plan_from_trial <= 150 THEN '121-150 days'
			WHEN Number_of_days_to_annual_plan_from_trial >= 151 AND Number_of_days_to_annual_plan_from_trial <= 180 THEN '151-180 days'
			WHEN Number_of_days_to_annual_plan_from_trial >= 181 AND Number_of_days_to_annual_plan_from_trial <= 210 THEN '181-210 days'
			WHEN Number_of_days_to_annual_plan_from_trial >= 211 AND Number_of_days_to_annual_plan_from_trial <= 240 THEN '211-240 days'
			WHEN Number_of_days_to_annual_plan_from_trial >= 241 AND Number_of_days_to_annual_plan_from_trial <= 270 THEN '241-270 days'
			WHEN Number_of_days_to_annual_plan_from_trial >= 271 AND Number_of_days_to_annual_plan_from_trial <= 300 THEN '271-300 days'
			WHEN Number_of_days_to_annual_plan_from_trial >= 301 AND Number_of_days_to_annual_plan_from_trial <= 330 THEN '301-330 days'
			WHEN Number_of_days_to_annual_plan_from_trial >= 331 AND Number_of_days_to_annual_plan_from_trial <= 360 THEN '331-360 days'
		END AS 'days_periods'
												
	FROM tus)
SELECT days_periods, ROUND(AVG(CAST(Number_of_days_to_annual_plan_from_trial AS FLOAT)), 2) AS 'Average value into 30 day periods'
FROM tuus
GROUP BY days_periods
ORDER BY ROUND(AVG(CAST(Number_of_days_to_annual_plan_from_trial AS FLOAT)), 2)
```
![Alt text](<Foodie-Fi pics/ffs10.png>)


11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

```SQL
WITH las AS
	(SELECT customer_id, STRING_AGG(plan_id, ',') AS 'plans_string'
	FROM downgrades
	WHERE  start_date <= '2020-12-31'
	GROUP BY customer_id)
SELECT COUNT(plans_string) AS 'Number of customers downgraded from a pro monthly to a basic monthly plan in 2020'
FROM las
WHERE plans_string LIKE '%2,1%'
```
![Alt text](<Foodie-Fi pics/ffs11.png>)



## C. Challenge Payment Question

The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

   - monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
   - upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
   - upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
   - once a customer churns they will no longer make payments

```SQL
--CREATE A NEW TABLE FROM SUBSCRIPTIONS TABLE, WE JUST WANT THE COLUMN NAMES AND DATA TYPES
SELECT s.customer_id, s.plan_id, s.start_date, p.plan_name, p.price
INTO subscriptions_c
FROM subscriptions s
LEFT JOIN plans p
ON s.plan_id = p.plan_id
ORDER BY customer_id, start_date;

--EMPTY THE CONTENTS OF THE NEW TABLE
DELETE FROM subscriptions_c



--JOIN THE TWO TABLES, THEN CREATE HELPER COLUMNS THAT WILL MAKE IT EASY TO COMPARE VALUES LATER, THEN CREATE A TEMP TABLE FROM THE CTE's.
WITH las AS
	(SELECT s.customer_id, s.plan_id, p.plan_name, s.start_date, p.price,  CAST('2020-12-31' AS DATE) AS 'end_date', 
	LAG(customer_id) OVER (ORDER BY customer_id, start_date) AS 'pre_customer_id',
	LAG(start_date) OVER (ORDER BY customer_id, start_date) AS 'pre_date',
	LEAD(customer_id) OVER (ORDER BY customer_id, start_date) AS 'post_customer_id',
	LEAD(start_date) OVER (ORDER BY customer_id, start_date) AS 'post_date',
	ROW_NUMBER() OVER (ORDER BY customer_id, start_date) AS 'record',
	LAG(s.plan_id) OVER (ORDER BY customer_id, start_date) AS 'pre-plan',
	LAG(p.price) OVER (ORDER BY customer_id, start_date) AS 'pre-price'
	FROM subscriptions s
	LEFT JOIN plans p
	ON s.plan_id = p.plan_id
	WHERE s.plan_id != 0 AND start_date <= '2020-12-31'),
	les AS
	(SELECT *, CASE
				WHEN pre_customer_id IS NULL THEN DATEDIFF(DAY, start_date, end_date)/30 + 1
				WHEN plan_id = 3 THEN 1
				WHEN customer_id != pre_customer_id AND customer_id != post_customer_id THEN DATEDIFF(DAY, start_date, end_date)/30 + 1
				WHEN customer_id != pre_customer_id AND customer_id = post_customer_id THEN DATEDIFF(DAY,start_date, post_date)/30 + 1
				WHEN customer_id = pre_customer_id AND customer_id != post_customer_id THEN DATEDIFF(DAY, start_date, end_date)/30 + 1
				WHEN customer_id = pre_customer_id AND customer_id = post_customer_id THEN DATEDIFF(DAY, start_date, post_date)/30 + 1
			  END AS 'payment_period_plan'
	FROM las)
SELECT *
INTO temp_subs
FROM les


--USE A WHILE LOOP TO ITERATE THROUGH EACH ROW, AND CREATE DATA FOR THE TABLE WE CREATED EARLIER
DECLARE @Counter INT 
DECLARE @Pay INT
SET @Counter=1 
SET @Pay = 1

WHILE ( @Counter <= (SELECT MAX(record) FROM temp_subs)) 
BEGIN 
		WHILE ( @Pay <= (SELECT payment_period_plan FROM temp_subs WHERE record = @Counter)) 
		BEGIN 
			INSERT INTO subscriptions_c
				SELECT customer_id, plan_id,  
						CASE
							WHEN @Pay = 1 THEN start_date
							ELSE DATEADD(MONTH, @Pay - 1, start_date )
						END AS 'payment_date', 
						plan_name, price
				FROM temp_subs
				WHERE record = @Counter
			SET @Pay = @Pay + 1 
		END 
    SET @Counter = @Counter + 1 
	SET @Pay = 1
END


--CLEAN OUT THE DISCREPENCIES FROM OUR TABLE WITH NEW DATA
DELETE
FROM subscriptions_c
WHERE start_date > '2020-12-31'
ORDER BY customer_id;


-- CLEAN OUT DISCREPENCIES WITH THE USE OF HELPER COLUMNS AND INSERT NEW COLUMNS TO FINALISE OUR TABLE
WITH kas AS
	(SELECT *, LAG(customer_id) OVER (ORDER BY customer_id, plan_id, start_date) AS 'pre_customer',
			  LAG(plan_id) OVER (ORDER BY customer_id, plan_id, start_date) AS 'pre_plan',
			  LAG(price) OVER (ORDER BY customer_id, plan_id, start_date) AS 'pre_price',
			  LAG(start_date) OVER (ORDER BY customer_id, plan_id, start_date) AS 'pre_date',
			  ROW_NUMBER() OVER (ORDER BY customer_id, plan_id, start_date) AS 'record'
	FROM subscriptions_c),
	kes AS
	(SELECT *
	FROM kas 
	WHERE start_date = pre_date),
	kis AS
	(SELECT * 
	FROM kas
	WHERE record NOT IN (SELECT record - 1 FROM kes) )
SELECT customer_id, plan_id, plan_name, 
		CASE
			WHEN customer_id = pre_customer AND plan_id = 3 AND pre_plan = 2  AND start_date != pre_date THEN DATEADD(MONTH, 1, pre_date)
			ELSE start_date
		END AS 'payment_date', 
		CASE
			WHEN customer_id = pre_customer AND  plan_id IN (2,3) AND pre_plan = 1  THEN price - pre_price
			WHEN price IS NULL THEN 0
			ELSE price
		END AS 'amount',
		ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY customer_id, plan_id, start_date) AS 'payment_order'
FROM kis
```
![Alt text](<Foodie-Fi pics/ffs15.png>)



## D. Outside The Box Questions

[IN PROGRESS]




    











