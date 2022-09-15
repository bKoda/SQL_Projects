# Bradley's Portfolio

# Case studies taken from https://8weeksqlchallenge.com/
The purpose of these exercises is to solve real world issues using data.
## Week 1

Week 1 is based on a new ramen shop looking to analyze current customers along with the performance of a rewards program.

### 1. What is the total amount each customer spent at the restaurant?
```
  SELECT
        sales.customer_id
      , SUM(sales.price) AS 'total_amt'
    FROM
        dannys_diner.sales
    JOIN
        dannys_diner.menu ON sales.product_id = menu.product_id
GROUP BY
        customer_id
ORDER BY
        SUM(price) DESC
```        
### 2. How many days has each customer visited the restaurant?
```
  SELECT DISTINCT
         sales.customer_id
       , COUNT(sales.order_date) AS 'total_visits'
    FROM
         dannys_diner.sales
GROUP BY
         sales.customer_id
ORDER BY 
	 COUNT(sales.order_date) DESC
```
### 3. What was the first item from the menu purchased by each customer?
```
   SELECT 
	  sales.customer_id
	, menu.product_name
     FROM
	  dannys_diner.sales
     JOIN
 	  (
 	   SELECT
  	 	  sales.customer_id
      	   	, MIN(sales.order_date) AS min_sale_date
    	     FROM
   		  dannys_diner.sales
	 GROUP BY
  	  	  sales.customer_id
  	   ) AS min_date ON min_date.customer_id = dannys_diner.customer_id
      JOIN
   	   dannys_diner.menu ON sales.product_id = menu.product_id
```	   
### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?	   
```
  SELECT
	 menu.product_name
       , COUNT(sales.product_id) AS product_count
    FROM
  	 dannys_diner.sales
    JOIN
     	 dannys_diner.menu ON sales.product_id = menu.product_id
GROUP BY
 	 menu.product_name
ORDER BY
 	 COUNT(sales.product_id) DESC
```
### 5. Which item was the most popular for each customer?
```
SELECT
      product_rank.customer_id
    , product_rank.product_name
    , product_rank.product_count
  FROM
      (
       SELECT
	     menu.product_name
           , sales.customer_id
           , COUNT(sales.product_id) AS product_count
           , RANK() OVER(PARTITION BY sales.customer_id ORDER BY COUNT(sales.product_id) DESC) AS product_order_rank
         FROM
  	     dannys_diner.sales
     	 JOIN
     	     dannys_diner.menu ON sales.product_id = menu.product_id
     GROUP BY
 	     menu.product_name
       	   , customer_id	 
      ) AS product_rank
 WHERE
      product_order_rank = 1
```
### 6. Which item was purchased first by the customer after they became a member?

```
SELECT
       order_rank_table.customer_id
     , order_rank_table.join_date
     , order_rank_table.order_date
     , order_rank_table.product_name
  FROM
      (
	SELECT
	       sales.customer_id
     	     , TO_CHAR(members.join_date :: DATE, 'yyyy-mm-dd') AS join_date
     	     , sales.order_date
     	     , menu.product_name
     	     , RANK() OVER(PARTITION BY members.customer_id ORDER BY sales.order_date) AS order_rank
 	  FROM
  	       dannys_diner.sales
  	  JOIN dannys_diner.members ON dannys_diner.members.customer_id = dannys_diner.sales.customer_id
  	  JOIN dannys_diner.menu ON dannys_diner.menu.product_id = dannys_diner.sales.product_id
 	 WHERE
 	       members.join_date < sales.order_date
       ) order_rank_table
 WHERE
       order_rank = 1
   ```
### 7. Which item was purchased just before the customer became a member?

```
SELECT
       order_rank_table.customer_id
     , order_rank_table.join_date
     , order_rank_table.order_date
     , order_rank_table.product_name
  FROM
      (
	SELECT
	       sales.customer_id
     	     , TO_CHAR(members.join_date :: DATE, 'yyyy-mm-dd') AS join_date
     	     , sales.order_date
     	     , menu.product_name
     	     , RANK() OVER(PARTITION BY members.customer_id ORDER BY sales.order_date DESC) AS order_rank
 	  FROM
  	       dannys_diner.sales
  	  JOIN dannys_diner.members ON dannys_diner.members.customer_id = dannys_diner.sales.customer_id
  	  JOIN dannys_diner.menu ON dannys_diner.menu.product_id = dannys_diner.sales.product_id
 	 WHERE
 	       members.join_date > sales.order_date
       ) order_rank_table
 WHERE
       order_rank = 1
   ```
### 8. What is the total items and amount spent for each member before they became a member?
```
  SELECT
         sales.customer_id
       , TO_CHAR(members.join_date :: DATE, 'yyyy-mm-dd') AS join_date
       , COUNT(sales.product_id) AS total_items
       , SUM(menu.price) AS total_price
    FROM
  	 dannys_diner.sales
    JOIN dannys_diner.members ON dannys_diner.members.customer_id = dannys_diner.sales.customer_id
    JOIN dannys_diner.menu ON dannys_diner.menu.product_id = dannys_diner.sales.product_id
   WHERE
 	 members.join_date > sales.order_date
GROUP BY 
 	 sales.customer_id
       , members.join_date
```
### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```
  SELECT
	 sales.customer_id
       , SUM(CASE 
           	 WHEN menu.product_id = 1 THEN (20 * menu.price)
           	 WHEN menu.product_id != 1 THEN (10 * menu.price)
           	 ELSE 0
           	 END
             ) AS total_points
    FROM
  	 dannys_diner.sales
    JOIN dannys_diner.menu ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY
	 sales.customer_id
ORDER BY
	 SUM(CASE 
           	 WHEN menu.product_id = 1 THEN (20 * menu.price)
           	 WHEN menu.product_id != 1 THEN (10 * menu.price)
           	 ELSE 0
           	 END) DESC
```
### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```
--Calculate members join date + 1 week after for promotion period
WITH join_dates AS
(
SELECT
      customer_id
    , join_date AS join_date_start
    , join_date + INTERVAL '7 day' AS join_date_end
  FROM
      dannys_diner.members
)
  SELECT
        sales.customer_id
      , SUM(CASE 
                WHEN (sales.order_date < join_dates.join_date_end AND sales.order_date > join_dates.join_date_start) AND menu.product_id IS NOT NULL THEN (20 * menu.price)
	        WHEN menu.product_id = 1 THEN (20 * menu.price)
                WHEN menu.product_id != 1 THEN (10 * menu.price)
                ELSE 0
                END) AS total_points
    FROM 
         dannys_diner.sales
    JOIN join_dates ON sales.customer_id = join_dates.customer_id
    JOIN dannys_diner.menu ON sales.product_id = menu.product_id
   WHERE
         sales.order_date BETWEEN '2021-01-01' AND '2021-01-31'
GROUP BY
	 sales.customer_id 

```
