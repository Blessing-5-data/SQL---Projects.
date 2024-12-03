# SQL---Projects.
8 Weeks SQL Challenge by danny ma
### Week 1 Danny's Diner

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

Tools Used
- DB Fiddle, PostgreSQL V13 

Learnings;
- Aggregate Function
- Common Table Expression 
- Joins
- Data calculations
- CASE statement
- PARTITION BY clause
- Window Functions.

### Exploratory Data Analysis
1 What is the total amount each customer spent at the restaurant?

2 How many days has each customer visited the restaurant?

3 What was the first item from the menu purchased by each customer?

4 What is the most purchased item on the menu and how many times was it purchased by all customers?

5 Which item was the most popular for each customer?

6 Which item was purchased first by the customer after they became a member?

7 Which item was purchased just before the customer became a member?

8 What is the total items and amount spent for each member before they became a member?

9 If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

10 In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

### Data Analysis
```  1. What is the total amount each customer spent at the restaurant?

 SELECT 
 customer_id,
 SUM(price) AS total_spend
 FROM sales AS S
 INNER JOIN menu AS M ON S.product_id = M.product_id
 GROUP BY customer_id; 

``` 2. How many days has each customer visited the restaurant?

 SELECT 
 customer_id, 
 COUNT(DISTINCT order_date)
 FROM sales
 GROUP BY customer_id;

``` 3. What was the first item from the menu purchased by each customer?

``` [Screenshot 2024-11-29 153949](https://github.com/user-attachments/assets/916b06d1-dc2f-4637-a918-6ff2a8a825b9)


``` 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

 SELECT 
 product_name,
 COUNT (order_date) AS orders
 FROM sales AS S
 INNER JOIN menu AS M ON S.product_id = M.product_id
 GROUP BY  product_name
 ORDER BY COUNT (order_date) DESC
 LIMIT 1 ;

``` 5. Which item was the most popular for each customer?

 WITH items AS(
 SELECT 
 product_name,
 customer_id,
 COUNT (order_date) AS orders,
 RANK()OVER (PARTITION BY customer_id ORDER BY COUNT (order_date) DESC ) AS rnk 
 FROM sales AS S
 INNER JOIN menu AS M ON S.product_id = M.product_id
 GROUP BY  product_name,customer_id
   )
   SELECT 
 product_name,
 customer_id
 FROM items
 WHERE rnk = 1;

``` 6  Which item was the most popular for each customer?

 WITH items AS(
 SELECT 
 product_name,
 customer_id,
 COUNT (order_date) AS orders,
 RANK()OVER (PARTITION BY customer_id ORDER BY COUNT (order_date) DESC ) AS rnk 
 FROM sales AS S
 INNER JOIN menu AS M ON S.product_id = M.product_id
 GROUP BY  product_name,customer_id
   )
   SELECT 
 product_name,
 customer_id
 FROM items
 WHERE rnk = 1;

``` 7. Which item was purchased just before the customer became a member?

 WITH firstItem AS(
 SELECT 
 s.customer_id,
 order_date,
 join_date,
 product_name,
 RANK() OVER( PARTITION BY S.customer_id ORDER BY S.order_date ) AS rnk
 FROM sales AS S
 INNER JOIN members AS MEN ON MEN.customer_id = S.customer_id
 INNER JOIN menu AS M ON S.product_id = M.product_id
 WHERE order_date <  join_date
 )
 SELECT 
 customer_id,
 product_name
 FROM firstItem 
 WHERE rnk = 1;
 
```  8. What is the total items and amount spent for each member before they became a member?

SELECT 
 S.customer_id,
 COUNT(product_name) AS total_items,
 SUM(price) AS total_amount_spend
 FROM sales AS S
 INNER JOIN members AS MEN ON MEN.customer_id = S.customer_id
 INNER JOIN menu AS M ON S.product_id = M.product_id
 WHERE order_date <  join_date
 GROUP BY  S.customer_id;

``` 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

 SELECT
 customer_id,
 SUM(CASE
 WHEN product_name = 'sushi' THEN price *10 * 2
 ELSE price * 10 
 END)AS points
 FROM menu AS M 
 INNER JOIN sales AS S ON S.product_id = M.product_id
 GROUP BY customer_id 
 ORDER BY customer_id ASC;

``` 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

SELECT
 S.customer_id,
SUM(CASE
 WHEN order_date BETWEEN MEM. join_date AND ( MEM.join_date + INTERVAL '6 day' ) THEN M.price * 10 * 2  
 WHEN product_name = 'sushi' THEN price *10 * 2
 ELSE price * 10 
 END) AS points
 FROM menu AS M 
 INNER JOIN sales AS S ON S.product_id = M.product_id
 INNER JOIN members AS MEM ON MEM.customer_id = S.customer_id
 WHERE S.order_date <= '2021-01-31'
 AND S.order_date >= MEM.join_date
 GROUP BY S.customer_id
 ORDER BY  S.customer_id ASC ;



```  BONUS QUESTIONS AND SOLUTION :
SELECT
S.customer_id,
order_date,
product_name,
price,
CASE
WHEN order_date < join_date THEN 'N'
WHEN join_date IS NULL THEN 'N'
ELSE 'Y'
END AS member
FROM sales AS S 
INNER JOIN menu AS M ON M.product_id = S.product_id
LEFT JOIN members AS MEM ON MEM.customer_id = S.customer_id
ORDER BY S.customer_id,
order_date,price DESC;

```  RANKING ALL THE THINGS :
WITH CTE AS (
SELECT
S.customer_id,
order_date,
product_name,
price,
CASE
WHEN order_date < join_date THEN 'N'
WHEN join_date IS NULL THEN 'N'
ELSE 'Y'
END AS member
FROM sales AS S 
INNER JOIN menu AS M ON M.product_id = S.product_id
LEFT JOIN members AS MEM ON MEM.customer_id = S.customer_id
ORDER BY S.customer_id,
order_date,price DESC
)
 SELECT 
*
,CASE
WHEN member = 'N' THEN NULL
ELSE RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date)
END AS rnk
FROM CTE;


