# Case Study Questions

The following case study questions include some general data exploration analysis for the nodes and transactions before diving right into the core business questions and finishes with a challenging final request!


### A. Customer Nodes Exploration

1. How many unique nodes are there on the Data Bank system?

```SQL
SELECT COUNT(DISTINCT node_id) AS 'Number_of_unique_nodes'
FROM customer_nodes;
```
![Alt text](<data bank pics/dbs1.png>)


2. What is the number of nodes per region?

```SQL
SELECT r.region_name, COUNT(node_id) 'Number of nodes per region'
FROM customer_nodes c
LEFT JOIN regions r
ON C.region_id = R.region_id
GROUP BY r.region_name
```
![Alt text](<data bank pics/dbs2.png>)


3. How many customers are allocated to each region?

```SQL
SELECT r.region_name, COUNT(DISTINCT customer_id) AS 'Number of customers allocated to each region'
FROM customer_nodes c
LEFT JOIN regions r
ON C.region_id = R.region_id
GROUP BY r.region_name
```
![Alt text](<data bank pics/dbs3.png>)


4. How many days on average are customers reallocated to a different node?

```SQL
SELECT *
INTO temp_nodes
FROM customer_nodes 
WHERE YEAR(end_date) != '9999'
ORDER BY customer_id, start_date

WITH las AS
	(SELECT *, CASE
				WHEN LAG(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) = node_id 
				  AND LEAD(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) = node_id THEN 1
			  END AS 'release'
	FROM temp_nodes),
	les AS
	(SELECT customer_id, region_id, node_id, start_date, end_date
	FROM las
	WHERE release IS NULL),
	lis AS
	(SELECT *, CASE
				WHEN LAG(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) = node_id 
					THEN LAG(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) 
				WHEN LEAD(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) = node_id
					THEN NULL
				ELSE start_date
			  END AS 'final_start_date'
	FROM les),
	los AS
	(SELECT customer_id, region_id, node_id, final_start_date, end_date
	FROM lis
	WHERE final_start_date IS NOT NULL),
	lus AS
	(SELECT *, DATEDIFF(DAY, final_start_date, end_date) + 1 AS 'days_of_node'
	FROM los),
	luus AS
	(SELECT customer_id, AVG(days_of_node) AS 'customer_average'
	FROM lus
	GROUP BY customer_id)
SELECT AVG(customer_average) AS 'Overall_average'
FROM luus
```
![Alt text](<data bank pics/dbs4.png>)


5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

```SQL
WITH las AS
	(SELECT *, CASE
				WHEN LAG(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) = node_id AND LEAD(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) = node_id THEN 1
			  END AS 'release'
	FROM temp_nodes),
	les AS
	(SELECT customer_id, region_id, node_id, start_date, end_date
	FROM las
	WHERE release IS NULL),
	lis AS
	(SELECT *, CASE
				WHEN LAG(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) = node_id 
					THEN LAG(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) 
				WHEN LEAD(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) = node_id
					THEN NULL
				ELSE start_date
			  END AS 'final_start_date'
	FROM les),
	los AS
	(SELECT customer_id, region_id, node_id, final_start_date, end_date
	FROM lis
	WHERE final_start_date IS NOT NULL),
	lus AS
	(SELECT *, DATEDIFF(DAY, final_start_date, end_date) + 1 AS 'days_of_node'
	FROM los),
	luus AS
	(SELECT customer_id, region_id, AVG(days_of_node) AS 'customer_average'
	FROM lus
	GROUP BY customer_id, region_id)
SELECT DISTINCT r.region_name, l.region_id, PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY l.customer_average) OVER (PARTITION BY l.region_id) AS 'median', PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY l.customer_average) OVER (PARTITION BY l.region_id) AS '80th percentile', PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY l.customer_average) OVER (PARTITION BY l.region_id) AS '95th percentile'
FROM luus l
LEFT JOIN regions r
ON l.region_id = r.region_id
```
![Alt text](<data bank pics/dbs5.png>)




### B. Customer Transactions

1. What is the unique count and total amount for each transaction type?

```SQL
SELECT txn_type, COUNT(txn_type) AS 'unique_count', SUM(txn_amount) AS 'total_amount_for_each transaction_type'
FROM customer_transactions
GROUP BY txn_type
```
![Alt text](<data bank pics/dbs6.png>)


2. What is the average total historical deposit counts and amounts for all customers?

```SQL
WITH les AS
	(SELECT customer_id,  COUNT(txn_type) AS 'unique_count', SUM(txn_amount) AS 'total_amount_for_each_deposit_transaction_type'
	FROM customer_transactions
	WHERE txn_type = 'deposit'
	GROUP BY customer_id)
SELECT AVG(unique_count) AS 'average total historical deposit counts',
			AVG(total_amount_for_each_deposit_transaction_type) AS 'average amounts for all customers'
FROM les
```
![Alt text](<data bank pics/dbs7.png>)


3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

```SQL
WITH las AS
	(SELECT customer_id, MONTH(txn_date) AS 'trans_month', 
			CASE
				WHEN txn_type = 'deposit' THEN 1
				ELSE 0
			END AS 'deposit_count',
			CASE
				WHEN txn_type = 'purchase' THEN 1
				ELSE 0
			END AS 'purchase_count',
			CASE
				WHEN txn_type = 'withdrawal' THEN 1
				ELSE 0
			END AS 'withdrawal_count'
	FROM customer_transactions),
	les AS
	(SELECT customer_id, trans_month, SUM(deposit_count) AS 'deposit_month_count',
			SUM(purchase_count) AS 'purchase_month_count',
			SUM(withdrawal_count) AS 'withdrawal_month_count'
	FROM las
	GROUP BY customer_id, trans_month)
SELECT trans_month, DATENAME(MONTH, CAST('2020-'+CAST(trans_month AS VARCHAR(20))+ '-01' AS DATE)) AS 'month', COUNT(customer_id) AS 'customer_count'
FROM les
WHERE (deposit_month_count > 1 AND purchase_month_count >= 1) OR (deposit_month_count > 1 AND withdrawal_month_count >= 1)
GROUP BY trans_month
ORDER BY trans_month
```
![Alt text](<data bank pics/dbs8.png>)


4. What is the closing balance for each customer at the end of the month?

```SQL
WITH les AS
		 (SELECT customer_id, txn_month, txn_type, txn_amount, record, 
				CASE
					WHEN txn_type = 'deposit' THEN 1 * txn_amount
					ELSE -1 * txn_amount
				END AS 'running_total_amount'														
		  FROM temp_customer_transactions
		  WHERE record = 1

		  UNION ALL

		  SELECT t.customer_id, t.txn_month, t.txn_type, t.txn_amount, t.record,
				CASE
					WHEN t.txn_type = 'deposit' THEN r.running_total_amount + (1*t.txn_amount)
					WHEN t.txn_type = 'withdrawal' THEN r.running_total_amount + (-1*t.txn_amount)
					WHEN t.txn_type = 'purchase' THEN r.running_total_amount + (-1*t.txn_amount)
				END AS running_total_amount		
		  FROM temp_customer_transactions t
		  INNER JOIN les r
		  ON t.record = r.record + 1 AND t.customer_id = r.customer_id),
		  las AS
			  (SELECT *,
					 LAST_VALUE(running_total_amount) OVER (PARTITION BY customer_id, txn_month ORDER BY record RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'month_total'
			  FROM les),
		   lis AS
		   (SELECT customer_id, txn_month, month_total
		   FROM las
		   GROUP BY customer_id, txn_month,month_total),
		   los AS
		   (SELECT DISTINCT customer_id, VALUE AS 'txn_month_1'
		    FROM temp_customer_transactions, generate_series(1,4)),
			lus AS
			(SELECT o.customer_id, o.txn_month_1, i.month_total
			FROM los o
			FULL JOIN lis i
			ON o.customer_id = i.customer_id AND o.txn_month_1 = i.txn_month)
SELECT *, CASE
			WHEN month_total IS NULL AND LAG(month_total) OVER (PARTITION BY customer_id ORDER BY txn_month_1) IS NULL
				AND LAG(month_total,2) OVER (PARTITION BY customer_id ORDER BY txn_month_1) IS NULL
					THEN LAG(month_total,3) OVER (PARTITION BY customer_id ORDER BY txn_month_1)
			WHEN month_total IS NULL AND LAG(month_total) OVER (PARTITION BY customer_id ORDER BY txn_month_1) IS NULL
					THEN LAG(month_total,2) OVER (PARTITION BY customer_id ORDER BY txn_month_1)
			WHEN month_total IS NULL THEN LAG(month_total) OVER (PARTITION BY customer_id ORDER BY txn_month_1) 
			ELSE month_total
		  END AS 'final_month_total'
INTO temp_month_total            --Next question prep
FROM lus
ORDER BY customer_id, txn_month_1
			
SELECT customer_id, txn_month_1, final_month_total
FROM temp_month_total
```
![Alt text](<data bank pics/dbs9.png>)


5. What is the percentage of customers who increase their closing balance by more than 5%?

```SQL
WITH las AS 
	(SELECT customer_id, txn_month_1, final_month_total, 
			CASE
				WHEN txn_month_1 = 1 THEN 0
				WHEN LAG(final_month_total) OVER (PARTITION BY customer_id ORDER BY txn_month_1) = 0 THEN NULL
				ELSE (CAST(final_month_total AS FLOAT) - LAG(final_month_total) OVER (PARTITION BY customer_id ORDER BY txn_month_1) )/ ABS(CAST((LAG(final_month_total) OVER (PARTITION BY customer_id ORDER BY txn_month_1)) AS FLOAT)) * 100
			END AS 'percentage_increase'
	FROM temp_month_total),
	les AS
	(SELECT COUNT(DISTINCT customer_id) AS 'Total_customers' 
	FROM las),
	lis AS
	(SELECT DISTINCT txn_month_1, COUNT(percentage_increase) OVER (PARTITION BY txn_month_1 ORDER BY txn_month_1) AS 'customer_count'
	FROM las
	WHERE percentage_increase > 5)
SELECT 0 AS 'month_0 -> 1_percentage_of_customers_with_>5%', 
(CAST((SELECT customer_count FROM lis WHERE txn_month_1=2) AS FLOAT)/(SELECT Total_customers FROM les))*100 AS 'month_1 -> 2_percentage_of_customers_with_>5%',
(CAST((SELECT customer_count FROM lis WHERE txn_month_1=3) AS FLOAT)/(SELECT Total_customers FROM les))*100 AS 'month_2 -> 3_percentage_of_customers_with_>5%',
(CAST((SELECT customer_count FROM lis WHERE txn_month_1=4) AS FLOAT)/(SELECT Total_customers FROM les))*100 AS 'month_3 -> 4_percentage_of_customers_with_>5%'
```
![Alt text](<data bank pics/dbs10.png>)



### C. Data Allocation Challenge

To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:


   - Option 1: data is allocated based off the amount of money at the end of the previous month

```SQL
WITH kas AS
	(SELECT customer_id, txn_month, month_balance, minimum_running_balance, average_running_balance, maximum_running_balance, COUNT(record) AS 'count_txns'
	FROM tempy_data_allo
	GROUP BY customer_id, txn_month, month_balance, minimum_running_balance, average_running_balance, maximum_running_balance
	),
	kes AS
	(SELECT *, 
		CASE
			WHEN month_balance < 0 THEN 0
			ELSE month_balance
		END AS 'data_to_be_allocated'
	FROM kas
	),
	kis AS
	(SELECT txn_month, SUM(data_to_be_allocated) AS 'data_to_be_allocated_monthly'
	FROM kes
	GROUP BY txn_month
	)
SELECT CASE
		WHEN txn_month = 1 THEN 'Data allocated at the start of February'
		WHEN txn_month = 2 THEN 'Data allocated at the start of March'
		WHEN txn_month = 3 THEN 'Data allocated at the start of April'
		WHEN txn_month = 4 THEN 'Data allocated at the start of May'
	   END AS 'data_month', data_to_be_allocated_monthly
FROM kis
ORDER BY txn_month
```
![Alt text](<data bank pics/dbs14.png>)


   - Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days

```SQL
WITH kas AS
	(SELECT customer_id, txn_month, month_balance, minimum_running_balance, average_running_balance, maximum_running_balance, COUNT(record) AS 'count_txns'
	FROM tempy_data_allo
	GROUP BY customer_id, txn_month, month_balance, minimum_running_balance, average_running_balance, maximum_running_balance
	),
	kes AS
	(SELECT *, CASE
				WHEN average_running_balance < 0 THEN 0
				ELSE average_running_balance
			  END AS 'data_to_be_allocated'
	FROM kas
	),
	kis AS
	(SELECT txn_month, SUM(data_to_be_allocated) AS 'data_to_be_allocated_monthly'
	FROM kes
	GROUP BY txn_month
	)
SELECT CASE
		WHEN txn_month = 1 THEN 'Data allocated at the start of February'
		WHEN txn_month = 2 THEN 'Data allocated at the start of March'
		WHEN txn_month = 3 THEN 'Data allocated at the start of April'
		WHEN txn_month = 4 THEN 'Data allocated at the start of May'
	   END AS 'data_month', data_to_be_allocated_monthly
FROM kis
ORDER BY txn_month
```
![Alt text](<data bank pics/dbs15.png>)


   - Option 3: data is updated real-time

```SQL
WITH kas AS
	(SELECT customer_id, txn_month, month_balance, minimum_running_balance, average_running_balance, maximum_running_balance, COUNT(record) AS 'count_txns'
	FROM tempy_data_allo
	GROUP BY customer_id, txn_month, month_balance, minimum_running_balance, average_running_balance, maximum_running_balance
	),
	kes AS
	(SELECT *, CASE
				WHEN minimum_running_balance < 0 THEN 0
				ELSE minimum_running_balance
			  END AS 'minimum_data_to_be_allocated',
			  CASE
				WHEN maximum_running_balance < 0 THEN 0
				ELSE maximum_running_balance
			  END AS 'maximum_data_to_be_allocated'

	FROM kas
	),
	kis AS
	(SELECT txn_month, SUM(minimum_data_to_be_allocated) AS 'minimum_data_to_be_allocated_monthly',
			SUM(maximum_data_to_be_allocated) AS 'maximum_data_to_be_allocated_monthly'
	FROM kes
	GROUP BY txn_month
	)
SELECT CASE
		WHEN txn_month = 1 THEN 'Data allocated at the start of February'
		WHEN txn_month = 2 THEN 'Data allocated at the start of March'
		WHEN txn_month = 3 THEN 'Data allocated at the start of April'
		WHEN txn_month = 4 THEN 'Data allocated at the start of May'
	   END AS 'data_month', minimum_data_to_be_allocated_monthly, maximum_data_to_be_allocated_monthly
FROM kis
ORDER BY txn_month
```
![Alt text](<data bank pics/dbs16.png>)


    
For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

   - running customer balance column that includes the impact each transaction

```SQL
SELECT customer_id, MONTH(txn_date) AS txn_month, txn_type, txn_amount, 
		CASE
			WHEN txn_type = 'deposit' THEN txn_amount
			ELSE -1*txn_amount
		END AS 'running_balance',
		ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY txn_date) AS 'record'
INTO temp_data_allocation
FROM customer_transactions 

SELECT *
FROM temp_data_allocation
```
![Alt text](<data bank pics/dbs11.png>)

   - customer balance at the end of each month

```SQL
WITH data_run_balance AS
			( 
			 SELECT customer_id, txn_month, txn_type, txn_amount, running_balance, record
			 FROM temp_data_allocation
			 WHERE record = 1

			 UNION ALL

			 SELECT t.customer_id, t.txn_month, t.txn_type, t.txn_amount, 
			 		CASE
						WHEN t.txn_type = 'deposit' THEN r.running_balance + t.txn_amount
						ELSE r.running_balance - t.txn_amount
					END AS running_balance,
					t.record
			FROM temp_data_allocation t
			INNER JOIN data_run_balance r
			ON t.customer_id = r.customer_id AND t.record = r.record + 1        
			),
			las AS
			(SELECT customer_id, txn_month, txn_type, txn_amount, running_balance, 
					LAST_VALUE(running_balance) OVER (PARTITION BY customer_id, txn_month ORDER BY txn_month RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'month_balance',
					MIN(running_balance) OVER (PARTITION BY customer_id, txn_month ORDER BY txn_month RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'minimum_running_balance',
					AVG(running_balance) OVER (PARTITION BY customer_id, txn_month ORDER BY txn_month RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'average_running_balance',
					MAX(running_balance) OVER (PARTITION BY customer_id, txn_month ORDER BY txn_month RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS 'maximum_running_balance',
					record
			FROM data_run_balance)
SELECT *
INTO tempy_data_allo
FROM las
ORDER BY customer_id, txn_month, record;

SELECT *
FROM tempy_data_allo
ORDER BY customer_id, txn_month, record;
```
![Alt text](<data bank pics/dbs12.png>)


   - minimum, average and maximum values of the running balance for each customer

```SQL
SELECT customer_id, txn_month, month_balance, minimum_running_balance, average_running_balance, maximum_running_balance, COUNT(record) AS 'count_txns'
FROM tempy_data_allo
GROUP BY customer_id, txn_month, month_balance, minimum_running_balance, average_running_balance, maximum_running_balance
ORDER BY customer_id, txn_month;
```
![Alt text](<data bank pics/dbs13.png>)

  
Using all of the data available - how much data would have been required for each option on a monthly basis?





### D. Extra Challenge

Data Bank wants to try another option which is a bit more difficult to implement - they want to calculate data growth using an interest calculation, just like in a traditional savings account you might have with a bank.

If the annual interest rate is set at 6% and the Data Bank team wants to reward its customers by increasing their data allocation based off the interest calculated on a daily basis at the end of each day, how much data would be required for this option on a monthly basis?

Special notes:

   - Data Bank wants an initial calculation which does not allow for compounding interest, however they may also be interested in a daily compounding interest calculation so you can try to perform this calculation if you have the stamina!

1. Simple Interest Scenario:

```SQL
WITH les AS
	(SELECT DISTINCT customer_id AS customer_id_1, CAST(DATEADD(DAY,VALUE,'2020-01-01') AS DATE) AS 'txn_date_1'
	FROM customer_transactions, GENERATE_SERIES(0,120,1)
	),
	las AS
	(SELECT *
	FROM customer_transactions c
	RIGHT JOIN les l
	ON c.customer_id = l.customer_id_1 AND c.txn_date = l.txn_date_1),
	lis AS
	(SELECT customer_id_1, txn_date_1, txn_type, txn_amount
	FROM las
	),
	los AS
	(SELECT customer_id_1, txn_date_1, 
			CASE
				WHEN txn_type IS NULL THEN ''
				ELSE txn_type 
			END AS 'txn_type_1',
			CASE 
				WHEN txn_amount IS NULL THEN 0
				ELSE txn_amount
			END AS 'txn_amount_1',
			CAST(0 AS FLOAT) AS 'running', CAST(0 AS FLOAT) AS 'interest_earned', CAST(0 AS FLOAT) AS 'total_amount',
		   ROW_NUMBER() OVER (PARTITION BY customer_id_1 ORDER BY txn_date_1)  AS 'record'  
	FROM lis
	),	
	lus AS
	(SELECT customer_id_1, txn_date_1, txn_type_1, txn_amount_1,
			 CASE
				WHEN txn_type_1 = 'deposit' THEN CAST(txn_amount_1 AS FLOAT)
				WHEN txn_type_1 = 'withdrawal' THEN CAST(-1*txn_amount_1 AS FLOAT)
				WHEN txn_type_1 = 'purchase' THEN CAST(-1*txn_amount_1 AS FLOAT)
				ELSE CAST(0 AS FLOAT)
			END AS 'running_balance',
			0 AS 'day_number',
			interest_earned,
			CASE
				WHEN txn_type_1 = 'deposit' THEN CAST(txn_amount_1 AS FLOAT)
				WHEN txn_type_1 = 'withdrawal' THEN CAST(-1*txn_amount_1 AS FLOAT)
				WHEN txn_type_1 = 'purchase' THEN CAST(-1*txn_amount_1 AS FLOAT)
				ELSE 0
			END AS 'total_amount',
			record	
	FROM los
	WHERE record = 1 
	
	UNION ALL

	SELECT t.customer_id_1, t.txn_date_1,t.txn_type_1, t.txn_amount_1, 
			CASE
				WHEN t.txn_type_1 = 'deposit' THEN ROUND(CAST(r.total_amount + t.txn_amount_1 AS FLOAT), 2)
				WHEN t.txn_type_1 = 'purchase' THEN ROUND(CAST(r.total_amount - t.txn_amount_1 AS FLOAT),2)
				WHEN t.txn_type_1 = 'withdrawal' THEN ROUND(CAST(r.total_amount - t.txn_amount_1 AS FLOAT),2)
				ELSE ROUND(CAST(r.running_balance AS FLOAT),2)
			END AS 'running_balance',
			CASE
				WHEN t.txn_type_1 IN ('deposit', 'purchase', 'withdrawal') THEN 0
				ELSE r.day_number + 1
			END AS 'day_number',
			CASE
				WHEN t.txn_type_1 = '' THEN ROUND(CAST((r.running_balance * 0.06 * (r.day_number + 1))/365 AS FLOAT),2) 
				ELSE CAST(0 AS FLOAT)
			END AS 'interest_earned',
			CASE
				WHEN t.txn_type_1 = 'deposit' THEN ROUND(CAST(r.total_amount + t.txn_amount_1 AS FLOAT),2)
				WHEN t.txn_type_1 = 'purchase' THEN ROUND(CAST(r.total_amount - t.txn_amount_1 AS FLOAT),2)
				WHEN t.txn_type_1 = 'withdrawal' THEN ROUND(CAST(r.total_amount - t.txn_amount_1 AS FLOAT),2)
				ELSE ROUND(CAST(r.running_balance + ((r.running_balance * 0.06 * (r.day_number + 1))/365) AS FLOAT),2)
			END AS 'total_amount',
			t.record
	FROM los t
	INNER JOIN lus r
	ON t.customer_id_1 = r.customer_id_1 AND t.record = r.record + 1)
SELECT *
INTO data_bank_interest
FROM lus
ORDER BY customer_id_1, txn_date_1									  
										  
OPTION (MAXRECURSION 150);

SELECT *
FROM data_bank_interest
ORDER BY customer_id_1, txn_date_1

```
![Alt text](<data bank pics/dbs17.png>)


```SQL
WITH fas AS
	(SELECT customer_id_1, MONTH(txn_date_1) AS month_number, total_amount
	FROM data_bank_interest
	WHERE txn_date_1 IN ('2020-01-31', '2020-02-29', '2020-03-31', '2020-04-30')),
	fes AS
	(SELECT *, CASE
				WHEN total_amount < 0 THEN 0
				ELSE total_amount
			  END AS 'final_total_amount'
	FROM fas)
SELECT month_number, SUM(final_total_amount) AS 'Total_data_provision'
FROM fes
GROUP BY month_number
ORDER BY month_number
```
![Alt text](<data bank pics/dbs18.png>)


2. Compound Interest Scenario:

```SQL
WITH les AS
	(SELECT DISTINCT customer_id AS customer_id_1, CAST(DATEADD(DAY,VALUE,'2020-01-01') AS DATE) AS 'txn_date_1'
	FROM customer_transactions, GENERATE_SERIES(0,120,1)
	),
	las AS
	(SELECT *
	FROM customer_transactions c
	RIGHT JOIN les l
	ON c.customer_id = l.customer_id_1 AND c.txn_date = l.txn_date_1),
	lis AS
	(SELECT customer_id_1, txn_date_1, txn_type, txn_amount
	FROM las
	),
	los AS
	(SELECT customer_id_1, txn_date_1,
			 CASE
				WHEN txn_type IS NULL THEN ''
				ELSE txn_type 
			END AS 'txn_type_1',
			CASE 
				WHEN txn_amount IS NULL THEN 0
				ELSE txn_amount
			END AS 'txn_amount_1',
			CAST(0 AS FLOAT) AS 'running', CAST(0 AS FLOAT) AS 'interest_earned', CAST(0 AS FLOAT) AS 'total_amount',
			ROW_NUMBER() OVER (PARTITION BY customer_id_1 ORDER BY txn_date_1)  AS 'record'  
	FROM lis
	),	
	lus AS
	(SELECT customer_id_1, txn_date_1, txn_type_1, txn_amount_1, 
			CASE
				WHEN txn_type_1 = 'deposit' THEN CAST(txn_amount_1 AS FLOAT)
				WHEN txn_type_1 = 'withdrawal' THEN CAST(-1*txn_amount_1 AS FLOAT)
				WHEN txn_type_1 = 'purchase' THEN CAST(-1*txn_amount_1 AS FLOAT)
				ELSE CAST(0 AS FLOAT)
			END AS 'running_balance',
			0 AS 'day_number',
			interest_earned,
			CASE
				WHEN txn_type_1 = 'deposit' THEN CAST(txn_amount_1 AS FLOAT)
				WHEN txn_type_1 = 'withdrawal' THEN CAST(-1*txn_amount_1 AS FLOAT)
				WHEN txn_type_1 = 'purchase' THEN CAST(-1*txn_amount_1 AS FLOAT)
				ELSE 0
			END AS 'total_amount',
			record	
	FROM los
	WHERE record = 1 
	
	UNION ALL

	SELECT t.customer_id_1, t.txn_date_1,t.txn_type_1, t.txn_amount_1,
			 CASE
				WHEN t.txn_type_1 = 'deposit' THEN ROUND(CAST(r.total_amount + t.txn_amount_1 AS FLOAT), 2)
				WHEN t.txn_type_1 = 'purchase' THEN ROUND(CAST(r.total_amount - t.txn_amount_1 AS FLOAT),2)
				WHEN t.txn_type_1 = 'withdrawal' THEN ROUND(CAST(r.total_amount - t.txn_amount_1 AS FLOAT),2)
				ELSE ROUND(CAST(r.running_balance AS FLOAT),2)
			END AS 'running_balance',
			CASE
				WHEN t.txn_type_1 IN ('deposit', 'purchase', 'withdrawal') THEN 0
				ELSE r.day_number + 1
			END AS 'day_number',
			CASE
				WHEN t.txn_type_1 = '' THEN ROUND(CAST((r.running_balance * POWER(CAST(( 1 + 0.06) AS DECIMAL(18, 15)), CAST(((CAST(r.day_number AS FLOAT)+ CAST(1 AS FLOAT))/CAST(365 AS FLOAT)) AS DECIMAL(18, 15))) - r.running_balance) AS FLOAT),2) 
				ELSE CAST(0 AS FLOAT)
			END AS 'interest_earned',
			CASE
				WHEN t.txn_type_1 = 'deposit' THEN ROUND(CAST(r.total_amount + t.txn_amount_1 AS FLOAT),2)
				WHEN t.txn_type_1 = 'purchase' THEN ROUND(CAST(r.total_amount - t.txn_amount_1 AS FLOAT),2)
				WHEN t.txn_type_1 = 'withdrawal' THEN ROUND(CAST(r.total_amount - t.txn_amount_1 AS FLOAT),2)
				ELSE ROUND(CAST(r.running_balance + ((r.running_balance * POWER(CAST(( 1 + 0.06) AS DECIMAL(18, 15)), CAST(((CAST(r.day_number AS FLOAT)+ CAST(1 AS FLOAT))/CAST(365 AS FLOAT)) AS DECIMAL(18, 15))) - r.running_balance)) AS FLOAT),2)
			END AS 'total_amount',
			t.record
	FROM los t
	INNER JOIN lus r
	ON t.customer_id_1 = r.customer_id_1 AND t.record = r.record + 1)
SELECT *
INTO data_bank_interest_2
FROM lus
ORDER BY customer_id_1, txn_date_1;

OPTION (MAXRECURSION 150)

SELECT *
FROM data_bank_interest_2
ORDER BY customer_id_1, txn_date_1;
```
![Alt text](<data bank pics/dbs20.png>)

```SQL
WITH kas AS
	(SELECT customer_id_1, MONTH(txn_date_1) AS month_number, total_amount
	FROM data_bank_interest_2
	WHERE txn_date_1 IN ('2020-01-31', '2020-02-29', '2020-03-31', '2020-04-30')),
	kes AS
	(SELECT *, CASE
				WHEN total_amount < 0 THEN 0
				ELSE total_amount
			  END AS 'final_total_amount'
	FROM kas)
SELECT month_number, SUM(final_total_amount) AS 'Total_data_provision'
FROM kes
GROUP BY month_number
ORDER BY month_number
```
![Alt text](<data bank pics/dbs21.png>)




### Extension Request

The Data Bank team wants you to use the outputs generated from the above sections to create a quick Powerpoint presentation which will be used as marketing materials for both external investors who might want to buy Data Bank shares and new prospective customers who might want to bank with Data Bank.

1. Using the outputs generated from the customer node questions, generate a few headline insights which Data Bank might use to market it’s world-leading security features to potential investors and customers.

2. With the transaction analysis - prepare a 1 page presentation slide which contains all the relevant information about the various options for the data provisioning so the Data Bank management team can make an informed decision.

**[TO BE COMPLETED]**