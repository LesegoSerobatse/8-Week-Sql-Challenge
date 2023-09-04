1. What is the total amount each customer spent at the restaurant?

```SQL
SELECT Customer_ID, SUM(Price) AS 'Total Amount For Each Customer'
FROM Sales
LEFT JOIN Menu 
ON Sales.Product_ID = Menu.Product_ID
GROUP BY Customer_ID
```
![Alt text](<Danny's Diner Pics/image-7.png>)


2. How many days has each customer visited the restaurant?

```SQL
SELECT Customer_ID, COUNT(DISTINCT Order_Date) AS 'No.of Days Per Customer'
FROM Sales
GROUP BY Customer_ID
```
![Alt text](<Danny's Diner Pics/image-8.png>)


3. What was the first item from the menu purchased by each customer?

```SQL
WITH Rank_Table	AS 
		(SELECT Customer_ID, Order_Date, Product_Name, DENSE_RANK() OVER (PARTITION BY Customer_ID ORDER BY Order_Date) AS Ranking
		 FROM Sales
		 LEFT JOIN Menu
		 ON Sales.Product_ID = Menu.Product_ID)
		 --ORDER BY Customer_ID, Order_Date)
SELECT DISTINCT Customer_ID, Order_Date, Product_Name, Ranking
FROM Rank_Table
WHERE Ranking = 1
ORDER BY Customer_ID;
```
![Alt text](<Danny's Diner Pics/image-9.png>)

4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```SQL
WITH tmp AS
		(SELECT Menu.Product_Name AS Product, COUNT(Menu.Product_Name) AS 'Item ordered frequency'
		FROM Sales
		LEFT JOIN Menu
		ON Sales.Product_ID = Menu.Product_ID
		GROUP BY Product_Name)
		
SELECT TOP 1 *
FROM tmp
ORDER BY [Item ordered frequency] DESC;
```
![Alt text](<Danny's Diner Pics/image-10.png>)

5. Which item was the most popular for each customer?

```SQL
WITH Lavino AS		
		(
		SELECT S.Customer_ID, M.Product_ID, M.Product_Name, COUNT(M.Product_ID)  AS 'Counting'
		FROM Sales S
		LEFT JOIN Menu M
		ON S.Product_ID = M.Product_ID
		GROUP BY S.Customer_ID, M.Product_ID, M.Product_Name
		),
	Lese AS
		(
		SELECT Customer_ID, Product_ID, Product_Name, Counting, DENSE_RANK() OVER (PARTITION BY Customer_ID ORDER BY Counting DESC) AS 'Rankor'
		FROM Lavino
		)
SELECT Customer_ID, Product_Name, Counting, Rankor
FROM Lese
WHERE Rankor = 1
```
![Alt text](<Danny's Diner Pics/image-11.png>)

6. Which item was purchased first by the customer after they became a member?

```SQL
WITH Goodness AS
		(
		SELECT S.Customer_ID, S.Order_Date, S.Product_ID, M.Join_Date, DENSE_RANK() OVER (PARTITION BY S.Customer_ID ORDER BY S.Order_Date) AS 'Rankee'
		FROM Sales S
		LEFT JOIN Members M
		ON M.Customer_ID = S.Customer_ID 
		WHERE M.Join_Date < S.Order_Date
		)

SELECT Goodness.Customer_ID, Goodness.Order_Date, Menu.Product_Name
FROM Goodness
LEFT JOIN Menu
ON Goodness.Product_ID = Menu.Product_ID
WHERE Goodness.Rankee = 1
```
![Alt text](<Danny's Diner Pics/image-13.png>)


7. Which item was purchased just before the customer became a member?

```SQL
WITH Goody AS
		(
		SELECT S.Customer_ID, S.Order_Date, S.Product_ID, M.Join_Date, DENSE_RANK() OVER (PARTITION BY S.Customer_ID ORDER BY S.Order_Date DESC) AS 'Rankee'
		FROM Sales S
		LEFT JOIN Members M
		ON M.Customer_ID = S.Customer_ID 
		WHERE M.Join_Date > S.Order_Date
		)
SELECT Goody.Customer_ID, Goody.Order_Date, Menu.Product_Name
FROM Goody
LEFT JOIN Menu
ON Goody.Product_ID = Menu.Product_ID
WHERE Goody.Rankee = 1
```
![Alt text](<Danny's Diner Pics/image-14.png>)

8. What is the total items and amount spent for each member before they became a member?

```SQL 
WITH TloTlo AS
		(
		SELECT S.Customer_ID, S.Order_Date, S.Product_ID, M.Join_Date, Mu.Product_Name, Mu.Price 
		FROM Sales S
		LEFT JOIN Members M
		ON M.Customer_ID = S.Customer_ID 
		LEFT JOIN Menu Mu
		ON S.Product_ID = Mu.Product_ID
		WHERE M.Join_Date > S.Order_Date
		OR S.Customer_ID NOT IN (SELECT Customer_ID FROM Members)
		)

SELECT DISTINCT Customer_ID, COUNT(Product_ID) OVER (PARTITION BY Customer_ID) AS 'Number of Items Ordered Before Member Join Date', SUM(Price) OVER (PARTITION BY Customer_ID) AS 'Total Amount Spent by Each Customer Before Member Join Date'
FROM TloTlo
```
![Alt text](<Danny's Diner Pics/image-15.png>)

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```SQL
WITH Mpho AS
		(
		SELECT s.Customer_ID, s.Product_ID, m.Product_Name, m.Price,
		CASE
			WHEN m.Product_Name = 'sushi' THEN m.Price*20
			ELSE m.Price*10
		END AS Points_Earned
		FROM Sales s
		LEFT JOIN Menu m
		ON s.Product_ID = m.Product_ID
		)
SELECT DISTINCT Customer_ID, SUM(Points_Earned) OVER (PARTITION BY Customer_ID) AS Total_Points_Earned_By_Customer
FROM Mpho
```
![Alt text](<Danny's Diner Pics/image-16.png>)

10.  In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```SQL
WITH Tshiamo AS		
		(
		SELECT S.Customer_ID, S.Order_Date, S.Product_ID, M.Join_Date, Mn.Price,-- DATEADD(WEEK, 1, M.Join_Date),
		CASE
			WHEN S.Order_Date < DATEADD(WEEK, 1, M.Join_Date) THEN Mn.Price*20
			WHEN S.Product_ID = 1 THEN Mn.Price*20
			ELSE Mn.Price*10
		END AS Points_Earned
		FROM Sales S
		LEFT JOIN Members M
		ON S.Customer_ID = M.Customer_ID
		LEFT JOIN Menu Mn
		ON S.Product_ID = Mn.Product_ID
		WHERE S.Order_Date >= M.Join_Date 
		AND S.Order_Date <= '2021-01-31'
		)

SELECT Customer_ID, SUM(Points_Earned) AS 'Members January Points'
FROM Tshiamo
GROUP BY Customer_ID
```
![Alt text](<Danny's Diner Pics/image-17.png>)


