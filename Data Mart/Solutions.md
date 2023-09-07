
# Case Study Questions

In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:

   - Convert the week_date to a DATE format

   - Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc

   - Add a month_number with the calendar month for each week_date value as the 3rd column

   - Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values

   - Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value

![Alt text](<Data Mart pics/dm2.png>)


  - Add a new demographic column using the following mapping for the first letter in the segment values:

![Alt text](<Data Mart pics/dm3.png>)


   - Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns

   - Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record

```SQL
SELECT CONVERT(DATE,  
                    CASE
                       WHEN CHARINDEX('/', STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0') )= 2
                         THEN STUFF(STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0'), 1, 0, '0') 
                       ELSE STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0')
                    END, 103) AS week_date,
                    CASE
                       WHEN YEAR(CONVERT(DATE, CASE
                                                  WHEN CHARINDEX('/', STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0') )= 2
                                                    THEN STUFF(STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0'), 1, 0, '0') 
                                                  ELSE STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0')
                                               END, 103)) = '2018'
                            THEN DATEPART(WEEK, DATEADD(DAY, -1, CONVERT(DATE, CASE
                                                                                  WHEN CHARINDEX('/', STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0') )= 2
                                                                                       THEN STUFF(STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0'), 1, 0, '0') 
                                                                                  ELSE STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0')
                                                                               END, 103)))
                       WHEN YEAR(CONVERT(DATE, CASE
                                                  WHEN CHARINDEX('/', STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0') )= 2
                                                       THEN STUFF(STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0'), 1, 0, '0') 
                                                  ELSE STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0')
                                               END, 103)) = '2019'
                            THEN DATEPART(WEEK, DATEADD(DAY, -2, CONVERT(DATE, CASE
                                                                                  WHEN CHARINDEX('/', STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0') )= 2
                                                                                       THEN STUFF(STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0'), 1, 0, '0') 
                                                                                  ELSE STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0')
                                                                               END, 103)))
                       WHEN YEAR(CONVERT(DATE, CASE
                                                  WHEN CHARINDEX('/', STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0') )= 2
                                                       THEN STUFF(STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0'), 1, 0, '0') 
                                                  ELSE STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0')
                                               END, 103)) = '2020'
                            THEN DATEPART(WEEK, DATEADD(DAY, -3, CONVERT(DATE, CASE
                                                                                  WHEN CHARINDEX('/', STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0') )= 2
                                                                                       THEN STUFF(STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0'), 1, 0, '0') 
                                                                                  ELSE STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0')
                                                                               END, 103)))
					  END AS 'week_number',
					  MONTH(CONVERT(DATE, CASE
                                              WHEN CHARINDEX('/', STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0') )= 2
                                                   THEN STUFF(STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0'), 1, 0, '0') 
                                              ELSE STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0')
                                          END, 103)) AS 'month_number',
					   YEAR(CONVERT(DATE, CASE
                                             WHEN CHARINDEX('/', STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0') )= 2
                                                  THEN STUFF(STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0'), 1, 0, '0') 
                                             ELSE STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0')
                                          END, 103)) AS 'calender_year',
						CASE
							WHEN segment = 'null' THEN 'unknown'
							WHEN RIGHT(segment, 1) = '1' THEN 'Young Adults'
							WHEN RIGHT(segment, 1) = '2' THEN 'Middle Aged'
							WHEN RIGHT(segment, 1) IN ('3', '4') THEN 'Retirees'
						END AS 'age_band',
						CASE
							WHEN segment = 'null' THEN 'unknown'
							WHEN LEFT(segment, 1) = 'C' THEN 'Couples'
							WHEN LEFT(segment, 1) = 'F' THEN 'Families'
						END AS 'demographic',
						ROUND(CAST(sales AS FLOAT)/CAST(transactions AS FLOAT), 2) AS 'avg_transaction', 
						region, platformm, customer_type, transactions, sales
INTO clean_weekly_sales_1					 					 
FROM weekly_sales
ORDER BY CONVERT(DATE,  CASE
                           WHEN CHARINDEX('/', STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0') )= 2
                                THEN STUFF(STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0'), 1, 0, '0') 
                           ELSE STUFF(STUFF(week_date, LEN(week_date)-1, 0, '20'), (CHARINDEX('/', STUFF(week_date, LEN(week_date)-1, 0, '20')) + 1), 0, '0')
                        END, 103);


SELECT *
FROM clean_weekly_sales_1
ORDER BY week_date;
```
**Note:** In this exercise I just wanted to practice my use of string functions and manipulation, because I could've just used the convert function 
```SQL 
CONVERT(DATE, week_date, 3)
```

![Alt text](<Data Mart pics/dms1.png>)


# 2. Data Exploration


1. What day of the week is used for each week_date value?

```SQL
SELECT week_date, DATENAME(WEEKDAY, week_date) AS 'Day_of_the_Week'
FROM clean_weekly_sales
GROUP BY week_date
ORDER BY week_date
```
![Alt text](<Data Mart pics/dms2.png>)


2. What range of week numbers are missing from the dataset?

```SQL
WITH las AS
	(SELECT DATEADD(WEEK, VALUE, CAST('2018-01-01' AS DATE)) AS 'date_missing'
	 FROM GENERATE_SERIES(0, 156, 1)),
	 les AS
	(SELECT date_missing, CASE
	                         WHEN date_missing = '2018-01-01' THEN 1
	                         WHEN YEAR(date_missing) = '2018' THEN DATEPART(WEEK, DATEADD(DAY, -1, date_missing))
	                         WHEN YEAR(date_missing) = '2019' THEN DATEPART(WEEK, DATEADD(DAY, -2, date_missing))
	                         WHEN YEAR(date_missing) = '2020' THEN DATEPART(WEEK, DATEADD(DAY, -3, date_missing))
	                         END AS 'week_number_missing'
	FROM las),
	lis AS
	(SELECT c.week_date, c.week_number, l.date_missing, l.week_number_missing
	FROM clean_weekly_sales c
	RIGHT JOIN les l
	ON c.week_date = l.date_missing
	WHERE week_date IS NULL),
	los AS
	(SELECT date_missing, week_number_missing, CASE
	                                              WHEN LAG(week_number_missing) OVER (PARTITION BY YEAR(date_missing) ORDER BY date_missing) + 1 = week_number_missing
	                                                 AND LEAD(week_number_missing) OVER (PARTITION BY YEAR(date_missing) ORDER BY date_missing) - 1 = week_number_missing
	                                                   THEN 0
	                                              ELSE week_number_missing
	                                           END AS 'week_missing_range',
	        ROW_NUMBER() OVER (PARTITION BY YEAR(date_missing) ORDER BY date_missing) AS 'record'
	FROM lis),
	lus AS
	(SELECT date_missing, week_missing_range,  ROW_NUMBER() OVER (PARTITION BY YEAR(date_missing) ORDER BY date_missing) AS 'record'
	FROM los
	WHERE week_missing_range <> 0),
	luus AS
	(SELECT YEAR(date_missing) AS 'Calender_year', CAST(date_missing AS VARCHAR(20)) + '  ->  ' + CAST(LEAD(date_missing) OVER (PARTITION BY YEAR(date_missing) ORDER BY date_missing) AS VARCHAR(20))  AS 'missing_date_range',
	        CAST(week_missing_range AS VARCHAR(20)) + '  ->  ' + CAST(LEAD(week_missing_range) OVER (PARTITION BY YEAR(date_missing) ORDER BY date_missing) AS VARCHAR(20)) AS 'range of week numbers missing', record
	FROM lus)
SELECT Calender_year, missing_date_range, [range of week numbers missing]
FROM luus
WHERE record % 2 <> 0;
```
![Alt text](<Data Mart pics/dms3.png>)


3. How many total transactions were there for each year in the dataset?

```SQL
SELECT DISTINCT calender_year, SUM(transactions) OVER (PARTITION BY calender_year ORDER BY calender_year RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'Total_transactions_per_year'
FROM clean_weekly_sales_1
ORDER BY calender_year
```
![Alt text](<Data Mart pics/dms4.png>)


4. What is the total sales for each region for each month?

```SQL
SELECT DISTINCT calender_year, month_number, region, 
	   SUM(CAST(sales AS BIGINT)) OVER (PARTITION BY region, month_number, calender_year ORDER BY calender_year, month_number RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'Total_sales_per_year'
FROM clean_weekly_sales_1
ORDER BY region, calender_year, month_number
```
![Alt text](<Data Mart pics/dms6.png>)


5. What is the total count of transactions for each platform?

```SQL
SELECT DISTINCT platformm, SUM(transactions) OVER (PARTITION BY platformm ORDER BY week_date RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'Total_count_of_transactions_per_platform'
FROM clean_weekly_sales_1
ORDER BY platformm
```
![Alt text](<Data Mart pics/dms7.png>)


6. What is the percentage of sales for Retail vs Shopify for each month?

```SQL
WITH kas AS
	(SELECT platformm, calender_year, month_number, SUM(CAST(sales AS BIGINT)) AS 'sales_per_month_per_platform'
	FROM clean_weekly_sales_1
	GROUP BY platformm, calender_year, month_number),
	kes AS
	(SELECT platformm, calender_year, month_number, sales_per_month_per_platform, 
			SUM(CAST(sales_per_month_per_platform AS BIGINT)) OVER (PARTITION BY calender_year, month_number ORDER BY calender_year, month_number RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'Total_sales_per_month'
	FROM kas)
SELECT platformm, calender_year, month_number, sales_per_month_per_platform, Total_sales_per_month,
			ROUND(((CAST(sales_per_month_per_platform AS FLOAT)/CAST(Total_sales_per_month AS FLOAT)) * 100), 2) AS 'percentage_of_sales_per_month_per_platform'
FROM kes
ORDER BY calender_year, month_number
```
![Alt text](<Data Mart pics/dms8.png>)


7. What is the percentage of sales by demographic for each year in the dataset?

```SQL
WITH las AS
	(SELECT demographic, calender_year, SUM(CAST(sales AS BIGINT)) AS 'sales_by_demographic_per_year'
	FROM clean_weekly_sales_1
	GROUP BY demographic, calender_year),
	les AS
	(SELECT *, SUM(sales_by_demographic_per_year) OVER (PARTITION BY calender_year ORDER BY calender_year RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'total_sales_per_year'
	FROM las)
SELECT *, ROUND((CAST(sales_by_demographic_per_year AS FLOAT)/CAST(total_sales_per_year AS FLOAT)) * 100, 2) AS 'percentage of sales by demographic for each year'
FROM les
ORDER BY calender_year, demographic
```
![Alt text](<Data Mart pics/dms9.png>)


8. Which age_band and demographic values contribute the most to Retail sales?

```SQL
WITH jas AS
	(SELECT DISTINCT age_band, demographic, SUM(CAST(sales AS BIGINT)) OVER (PARTITION BY age_band, demographic ORDER BY week_date RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'sales_per_ageband_and_demographic'
	FROM clean_weekly_sales_1
	WHERE platformm = 'Retail'),
	jes AS
	(SELECT *, SUM(sales_per_ageband_and_demographic) OVER (ORDER BY age_band RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'total_contribution_of_age_band_and_demographic'
	FROM jas),
	jis AS
	(SELECT *, ROUND(CAST(sales_per_ageband_and_demographic AS FLOAT)/ CAST(total_contribution_of_age_band_and_demographic AS FLOAT) * 100, 2) AS 'percentage_contribution_of_age_band_and_demographic'
	FROM jes)
SELECT *
FROM jis
ORDER BY percentage_contribution_of_age_band_and_demographic DESC
```
![Alt text](<Data Mart pics/dms10.png>)


9.  Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?

```SQL
SELECT calender_year, platformm, ROUND(AVG(avg_transaction), 2) AS 'average transaction size for each year per platform'
FROM clean_weekly_sales_1
GROUP BY calender_year, platformm
ORDER BY calender_year
```
YES, we can use the avg_transaction column to find the average transaction size for each year.

![Alt text](<Data Mart pics/dms11.png>)



# 3. Before & After Analysis

This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time.

Taking the week_date value of 2020-06-15 as the baseline week where the Data Mart sustainable packaging changes came into effect.

We would include all week_date values for 2020-06-15 as the start of the period after the change and the previous week_date values would be before

Using this analysis approach - answer the following questions:

 1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?

```SQL
WITH dan AS
	(SELECT DATEADD(WEEK, -VALUE,CAST('2020-06-15' AS DATE)) AS '4_weeks_before_and_after', 'pre_4_weeks' AS 'pre_and_post_date'
	FROM GENERATE_SERIES(1, 4)
	UNION ALL
	SELECT DATEADD(WEEK, VALUE,CAST('2020-06-15' AS DATE)), 'after_4_weeks' 
	FROM GENERATE_SERIES(0, 3)),
	den AS
	(SELECT *
	FROM clean_weekly_sales_1 c
	INNER JOIN dan d
	ON c.week_date = d.[4_weeks_before_and_after]),
	din AS
	(SELECT pre_and_post_date, SUM(CAST(sales AS BIGINT)) AS 'total_sales_before_and_after_4weeks'
	FROM den
	GROUP BY pre_and_post_date),
	don AS
	(SELECT *, [total_sales_before_and_after_4weeks] - 
										LAG([total_sales_before_and_after_4weeks]) OVER (ORDER BY [pre_and_post_date] DESC) AS 'reduction rate in actual sales values'
	FROM din)
SELECT *, ROUND(CAST([reduction rate in actual sales values] AS FLOAT)/
							CAST(LAG([total_sales_before_and_after_4weeks]) OVER (ORDER BY [pre_and_post_date] DESC) AS FLOAT) * 100, 2) AS 'reduction rate in percentage of sales'
FROM don
```

![Alt text](<Data Mart pics/dms12.png>)


 2. What about the entire 12 weeks before and after?

```SQL
WITH dan AS
	(SELECT DATEADD(WEEK, -VALUE,CAST('2020-06-15' AS DATE)) AS '12_weeks_before_and_after', 'pre_12_weeks' AS 'pre_and_post_date'
	FROM GENERATE_SERIES(1, 12)
	UNION ALL
	SELECT DATEADD(WEEK, VALUE,CAST('2020-06-15' AS DATE)), 'after_12_weeks' 
	FROM GENERATE_SERIES(0, 11)),
	den AS
	(SELECT *
	FROM clean_weekly_sales_1 c
	INNER JOIN dan d
	ON c.week_date = d.[12_weeks_before_and_after]),
	din AS
	(SELECT pre_and_post_date, SUM(CAST(sales AS BIGINT)) AS 'total_sales_before_and_after_12weeks'
	FROM den
	GROUP BY pre_and_post_date),
	don AS
	(SELECT *, [total_sales_before_and_after_12weeks] - 
										LAG([total_sales_before_and_after_12weeks]) OVER (ORDER BY [pre_and_post_date] DESC) AS 'reduction rate in actual sales values'
	FROM din)
SELECT *, ROUND(CAST([reduction rate in actual sales values] AS FLOAT)/
							CAST(LAG([total_sales_before_and_after_12weeks]) OVER (ORDER BY [pre_and_post_date] DESC) AS FLOAT) * 100, 2) AS 'reduction rate in percentage of sales'
FROM don
```
![Alt text](<Data Mart pics/dms13.png>)


 3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?

```SQL
WITH las AS
	(SELECT DISTINCT week_date, calender_year
	FROM clean_weekly_sales_1
	WHERE week_number =	CASE
				    WHEN calender_year = '2020' THEN 24
				    WHEN calender_year = '2019' THEN 24
				    WHEN calender_year = '2018' THEN 25
				END ),
	les AS
	(SELECT *, DATEADD(WEEK, -VALUE, week_date) AS 'weeks_before_and_after', 'pre_4_weeks' AS 'pre_and_post_date'
	FROM las, GENERATE_SERIES(1, 4)
	UNION ALL
	SELECT *, DATEADD(WEEK, VALUE, week_date), 'after_4_weeks' 
	FROM las, GENERATE_SERIES(0, 3)
	UNION ALL
	SELECT *, DATEADD(WEEK, -VALUE, week_date) AS '12_weeks_before_and_after', 'pre_12_weeks' AS 'pre_and_post_date'
	FROM las,GENERATE_SERIES(1, 12)
	UNION ALL
	SELECT *, DATEADD(WEEK, VALUE, week_date), 'after_12_weeks' 
	FROM las,GENERATE_SERIES(0, 11)),
	lis AS
	(SELECT c.*, l.pre_and_post_date
	FROM clean_weekly_sales_1 c 
	INNER JOIN les l
	ON c.week_date = l.weeks_before_and_after),
	los AS
	(SELECT calender_year, pre_and_post_date, SUM(CAST(sales AS BIGINT)) AS 'total_sales_before_and_after_4_and_12weeks', 
											ROW_NUMBER() OVER (ORDER BY calender_year) AS 'record'
	FROM lis
	GROUP BY calender_year, pre_and_post_date),
	lus AS
	(SELECT *, NTILE(6) OVER (ORDER BY record) AS 'unique_record'
	FROM los),
	kas AS
	(SELECT *, [total_sales_before_and_after_4_and_12weeks] - 
										LAG([total_sales_before_and_after_4_and_12weeks]) OVER (PARTITION BY unique_record ORDER BY [pre_and_post_date] DESC) AS 'growth or reduction rate in actual sales values'
	FROM lus),
	kes AS
	(SELECT *, ROUND(CAST([growth or reduction rate in actual sales values] AS FLOAT)/
							CAST(LAG([total_sales_before_and_after_4_and_12weeks]) OVER (PARTITION BY unique_record ORDER BY [pre_and_post_date] DESC) AS FLOAT) * 100, 2) AS 'growth or reduction rate in percentage of sales'
	FROM kas)
SELECT calender_year, pre_and_post_date, [total_sales_before_and_after_4_and_12weeks], 
			[growth or reduction rate in actual sales values], [growth or reduction rate in percentage of sales]
FROM kes;
```
![Alt text](<Data Mart pics/dms15.png>)



# 4. Bonus Question

Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?

   - region

```SQL
WITH las AS
	(SELECT DISTINCT week_date, calender_year
	FROM clean_weekly_sales_1
	WHERE week_number =	CASE
				   WHEN calender_year = '2020' THEN 24
				END ),
	les AS
	(
	SELECT *, DATEADD(WEEK, -VALUE, week_date) AS 'weeks_before_and_after', 'pre_12_weeks' AS 'pre_and_post_date'
	FROM las,GENERATE_SERIES(1, 12)
	UNION ALL
	SELECT *, DATEADD(WEEK, VALUE, week_date), 'after_12_weeks' 
	FROM las,GENERATE_SERIES(0, 11)),
	lis AS
	(SELECT c.*, l.pre_and_post_date
	FROM clean_weekly_sales_1 c 
	INNER JOIN les l
	ON c.week_date = l.weeks_before_and_after),
	los AS
	(SELECT region, pre_and_post_date, SUM(CAST(sales AS BIGINT)) AS 'total_sales_before_and_after_4_and_12weeks', 
											ROW_NUMBER() OVER (ORDER BY region) AS 'record'
	FROM lis
	GROUP BY region, pre_and_post_date),
	lus AS
	(SELECT *, NTILE(7) OVER (ORDER BY record) AS 'unique_record'
	FROM los),
	kas AS
	(SELECT *, [total_sales_before_and_after_4_and_12weeks] - 
										LAG([total_sales_before_and_after_4_and_12weeks]) OVER (PARTITION BY unique_record ORDER BY [pre_and_post_date] DESC) AS 'growth or reduction rate in actual sales values'
	FROM lus),
	kes AS
	(SELECT *, ROUND(CAST([growth or reduction rate in actual sales values] AS FLOAT)/
							CAST(LAG([total_sales_before_and_after_4_and_12weeks]) OVER (PARTITION BY unique_record ORDER BY [pre_and_post_date] DESC) AS FLOAT) * 100, 2) AS 'growth or reduction rate in percentage of sales'
	FROM kas)
SELECT region, pre_and_post_date, [total_sales_before_and_after_4_and_12weeks], 
			[growth or reduction rate in actual sales values], [growth or reduction rate in percentage of sales]
FROM kes
```
![Alt text](<Data Mart pics/dms16.png>)

   - platform

```SQL
WITH las AS
	(SELECT DISTINCT week_date, calender_year
	FROM clean_weekly_sales_1
	WHERE week_number =	CASE
				   WHEN calender_year = '2020' THEN 24
				END ),
	les AS
	(
	SELECT *, DATEADD(WEEK, -VALUE, week_date) AS 'weeks_before_and_after', 'pre_12_weeks' AS 'pre_and_post_date'
	FROM las,GENERATE_SERIES(1, 12)
	UNION ALL
	SELECT *, DATEADD(WEEK, VALUE, week_date), 'after_12_weeks' 
	FROM las,GENERATE_SERIES(0, 11)),
	lis AS
	(SELECT c.*, l.pre_and_post_date
	FROM clean_weekly_sales_1 c 
	INNER JOIN les l
	ON c.week_date = l.weeks_before_and_after),
	los AS
	(SELECT platformm, pre_and_post_date, SUM(CAST(sales AS BIGINT)) AS 'total_sales_before_and_after_4_and_12weeks', 
											ROW_NUMBER() OVER (ORDER BY platformm) AS 'record'
	FROM lis
	GROUP BY platformm, pre_and_post_date),
	lus AS
	(SELECT *, NTILE(2) OVER (ORDER BY record) AS 'unique_record'
	FROM los),
	kas AS
	(SELECT *, [total_sales_before_and_after_4_and_12weeks] - 
										LAG([total_sales_before_and_after_4_and_12weeks]) OVER (PARTITION BY unique_record ORDER BY [pre_and_post_date] DESC) AS 'growth or reduction rate in actual sales values'
	FROM lus),
	kes AS
	(SELECT *, ROUND(CAST([growth or reduction rate in actual sales values] AS FLOAT)/
							CAST(LAG([total_sales_before_and_after_4_and_12weeks]) OVER (PARTITION BY unique_record ORDER BY [pre_and_post_date] DESC) AS FLOAT) * 100, 2) AS 'growth or reduction rate in percentage of sales'
	FROM kas)
SELECT platformm, pre_and_post_date, [total_sales_before_and_after_4_and_12weeks], 
			[growth or reduction rate in actual sales values], [growth or reduction rate in percentage of sales]
FROM kes
```
![Alt text](<Data Mart pics/dms17.png>)


   - age_band

```SQL
WITH las AS
	(SELECT DISTINCT week_date, calender_year
	FROM clean_weekly_sales_1
	WHERE week_number =	CASE
				   WHEN calender_year = '2020' THEN 24
				END ),
	les AS
	(
	SELECT *, DATEADD(WEEK, -VALUE, week_date) AS 'weeks_before_and_after', 'pre_12_weeks' AS 'pre_and_post_date'
	FROM las,GENERATE_SERIES(1, 12)
	UNION ALL
	SELECT *, DATEADD(WEEK, VALUE, week_date), 'after_12_weeks' 
	FROM las,GENERATE_SERIES(0, 11)),
	lis AS
	(SELECT c.*, l.pre_and_post_date
	FROM clean_weekly_sales_1 c 
	INNER JOIN les l
	ON c.week_date = l.weeks_before_and_after),
	los AS
	(SELECT age_band, pre_and_post_date, SUM(CAST(sales AS BIGINT)) AS 'total_sales_before_and_after_4_and_12weeks', 
											ROW_NUMBER() OVER (ORDER BY age_band) AS 'record'
	FROM lis
	GROUP BY age_band, pre_and_post_date),
	lus AS
	(SELECT *, NTILE(4) OVER (ORDER BY record) AS 'unique_record'
	FROM los),
	kas AS
	(SELECT *, [total_sales_before_and_after_4_and_12weeks] - 
										LAG([total_sales_before_and_after_4_and_12weeks]) OVER (PARTITION BY unique_record ORDER BY [pre_and_post_date] DESC) AS 'growth or reduction rate in actual sales values'
	FROM lus),
	kes AS
	(SELECT *, ROUND(CAST([growth or reduction rate in actual sales values] AS FLOAT)/
							CAST(LAG([total_sales_before_and_after_4_and_12weeks]) OVER (PARTITION BY unique_record ORDER BY [pre_and_post_date] DESC) AS FLOAT) * 100, 2) AS 'growth or reduction rate in percentage of sales'
	FROM kas)
SELECT age_band, pre_and_post_date, [total_sales_before_and_after_4_and_12weeks], 
			[growth or reduction rate in actual sales values], [growth or reduction rate in percentage of sales]
FROM kes
```
![Alt text](<Data Mart pics/dms18.png>)


   - demographic

```SQL
WITH las AS
	(SELECT DISTINCT week_date, calender_year
	FROM clean_weekly_sales_1
	WHERE week_number =	CASE
				   WHEN calender_year = '2020' THEN 24
				END ),
	les AS
	(
	SELECT *, DATEADD(WEEK, -VALUE, week_date) AS 'weeks_before_and_after', 'pre_12_weeks' AS 'pre_and_post_date'
	FROM las,GENERATE_SERIES(1, 12)
	UNION ALL
	SELECT *, DATEADD(WEEK, VALUE, week_date), 'after_12_weeks' 
	FROM las,GENERATE_SERIES(0, 11)),
	lis AS
	(SELECT c.*, l.pre_and_post_date
	FROM clean_weekly_sales_1 c 
	INNER JOIN les l
	ON c.week_date = l.weeks_before_and_after),
	los AS
	(SELECT demographic, pre_and_post_date, SUM(CAST(sales AS BIGINT)) AS 'total_sales_before_and_after_4_and_12weeks', 
											ROW_NUMBER() OVER (ORDER BY demographic) AS 'record'
	FROM lis
	GROUP BY demographic, pre_and_post_date),
	lus AS
	(SELECT *, NTILE(3) OVER (ORDER BY record) AS 'unique_record'
	FROM los),
	kas AS
	(SELECT *, [total_sales_before_and_after_4_and_12weeks] - 
										LAG([total_sales_before_and_after_4_and_12weeks]) OVER (PARTITION BY unique_record ORDER BY [pre_and_post_date] DESC) AS 'growth or reduction rate in actual sales values'
	FROM lus),
	kes AS
	(SELECT *, ROUND(CAST([growth or reduction rate in actual sales values] AS FLOAT)/
							CAST(LAG([total_sales_before_and_after_4_and_12weeks]) OVER (PARTITION BY unique_record ORDER BY [pre_and_post_date] DESC) AS FLOAT) * 100, 2) AS 'growth or reduction rate in percentage of sales'
	FROM kas)
SELECT demographic, pre_and_post_date, [total_sales_before_and_after_4_and_12weeks], 
			[growth or reduction rate in actual sales values], [growth or reduction rate in percentage of sales]
FROM kes
```

![Alt text](<Data Mart pics/dms19.png>)


   - customer_type

```SQL
WITH las AS
	(SELECT DISTINCT week_date, calender_year
	FROM clean_weekly_sales_1
	WHERE week_number =	CASE
				   WHEN calender_year = '2020' THEN 24
				END ),
	les AS
	(
	SELECT *, DATEADD(WEEK, -VALUE, week_date) AS 'weeks_before_and_after', 'pre_12_weeks' AS 'pre_and_post_date'
	FROM las,GENERATE_SERIES(1, 12)
	UNION ALL
	SELECT *, DATEADD(WEEK, VALUE, week_date), 'after_12_weeks' 
	FROM las,GENERATE_SERIES(0, 11)),
	lis AS
	(SELECT c.*, l.pre_and_post_date
	FROM clean_weekly_sales_1 c 
	INNER JOIN les l
	ON c.week_date = l.weeks_before_and_after),
	los AS
	(SELECT customer_type, pre_and_post_date, SUM(CAST(sales AS BIGINT)) AS 'total_sales_before_and_after_4_and_12weeks', 
											ROW_NUMBER() OVER (ORDER BY customer_type) AS 'record'
	FROM lis
	GROUP BY customer_type, pre_and_post_date),
	lus AS
	(SELECT *, NTILE(3) OVER (ORDER BY record) AS 'unique_record'
	FROM los),
	kas AS
	(SELECT *, [total_sales_before_and_after_4_and_12weeks] - 
										LAG([total_sales_before_and_after_4_and_12weeks]) OVER (PARTITION BY unique_record ORDER BY [pre_and_post_date] DESC) AS 'growth or reduction rate in actual sales values'
	FROM lus),
	kes AS
	(SELECT *, ROUND(CAST([growth or reduction rate in actual sales values] AS FLOAT)/
							CAST(LAG([total_sales_before_and_after_4_and_12weeks]) OVER (PARTITION BY unique_record ORDER BY [pre_and_post_date] DESC) AS FLOAT) * 100, 2) AS 'growth or reduction rate in percentage of sales'
	FROM kas)
SELECT customer_type, pre_and_post_date, [total_sales_before_and_after_4_and_12weeks], 
			[growth or reduction rate in actual sales values], [growth or reduction rate in percentage of sales]
FROM kes
```
![Alt text](<Data Mart pics/dms20.png>)


Do you have any further recommendations for Danny’s team at Data Mart or any interesting insights based off this analysis?

**[TO BE COMPLETED]**