# Bradley's Portfolio

# Case studies taken from https://8weeksqlchallenge.com/
The purpose of these exercises are to solve real world issues using data. Data queried using PostgreSQL.
## Week 1

Week 1 is based on a new ramen shop looking to analyze current customers along with the performance of a rewards program.

### 1. What is the total amount each customer spent at the restaurant?
```
  SELECT
        sales.customer_id
      , SUM(sales.price) AS total_amt
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
  SELECT
         sales.customer_id
       , COUNT(DISTINCT(sales.order_date)) AS total_visits
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
     	     , RANK() OVER(PARTITION BY members.customer_id ORDER BY sales.order_date DESC) AS 'order_rank'
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
                END) AS 'total_points'
    FROM 
         dannys_diner.sales
    JOIN join_dates ON sales.customer_id = join_dates.customer_id
    JOIN dannys_diner.menu ON sales.product_id = menu.product_id
   WHERE
         sales.order_date BETWEEN '2021-01-01' AND '2021-01-31'
GROUP BY
	 sales.customer_id 

```
## Week 2

Week 2 is based on the delivery service of a pizza place.

## A. Pizza Metrics

### 1. How many pizzas were ordered?

```
SELECT 
      COUNT(order_id) AS total_pizza_orders
  FROM
      pizza_runner.customer_orders
```

### 2. How many unique customer orders were made?

```
SELECT 
      COUNT(DISTINCT(customer_id)) AS total_customers
  FROM
      pizza_runner.customer_orders
```
3. How many successful orders were delivered by each runner?
```
  SELECT 
         runner_id
       , COUNT(order_id) AS total_orders
    FROM
         pizza_runner.runner_orders
GROUP BY
	 runner_id
```
4. How many of each type of pizza was delivered?

```
  SELECT 
         pizza_name
       , COUNT(*) AS total_delivered_pizzas
    FROM
         pizza_runner.customer_orders
    JOIN pizza_runner.runner_orders ON customer_orders.order_id = runner_orders.order_id --Inner join on orders that were ordered + delivered
     AND cancellation IN ('','null') --Only pull orders that were not canceled
    JOIN pizza_runner.pizza_names ON pizza_names.pizza_id = customer_orders.pizza_id 
GROUP BY
	 pizza_name
```

5. How many Vegetarian and Meatlovers were ordered by each customer?

```
  SELECT 
         customer_id
       , pizza_name
       , COUNT(*) AS total_delivered_pizzas
    FROM
         pizza_runner.customer_orders
    JOIN pizza_runner.pizza_names ON pizza_names.pizza_id = customer_orders.pizza_id 
GROUP BY
	 customer_id
       , pizza_name
ORDER BY
	 customer_id
       , COUNT(*) DESC
```

6. What was the maximum number of pizzas delivered in a single order?

```
  SELECT 
         order_id
       , COUNT(*) AS total_delivered_pizzas
    FROM
         pizza_runner.customer_orders
GROUP BY
         order_id
ORDER BY
	 COUNT(*) DESC
   LIMIT 1
```

7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```
  SELECT
	 customer_id
       , SUM(changes) AS total_order_changes
    FROM
	 (    
  	  SELECT 
           	 customer_orders.customer_id
               , customer_orders.order_id
               , MAX(CASE 
                	  WHEN exclusions ~ '[0-9]' OR extras ~ '[0-9]' THEN '1'
                	  ELSE 0
                	  END) Changes
    	    FROM
    	 	  pizza_runner.customer_orders
    	    JOIN
         	 pizza_runner.runner_orders ON customer_orders.order_id = runner_orders.order_id
   	     AND cancellation IN ('','null') --Only pull orders that were not canceled
	GROUP BY
           	 customer_orders.customer_id
               , customer_orders.order_id
        ) change_table
GROUP BY
 	  customer_id
```
8. How many pizzas were delivered that had both exclusions and extras?

```
SELECT 
       customer_orders.customer_id
     , customer_orders.order_id
     , CASE 
      	    WHEN exclusions ~ '[0-9]' AND extras ~ '[0-9]' THEN '1'
      	    ELSE 0
      	    END Changes
  FROM
       pizza_runner.customer_orders
  JOIN
       pizza_runner.runner_orders ON customer_orders.order_id = runner_orders.order_id
   AND cancellation IN ('','null') --Only pull orders that were not canceled
 WHERE
      (CASE 
            WHEN exclusions ~ '[0-9]' AND extras ~ '[0-9]' THEN '1'
            ELSE 0
            END) > 0
```

9. What was the total volume of pizzas ordered for each hour of the day?

```
  SELECT
	 COUNT(order_id) AS total_orders
       , EXTRACT(HOUR FROM order_time) AS order_hour
    FROM
  	 pizza_runner.customer_orders
GROUP BY
	 EXTRACT(HOUR FROM order_time)
ORDER BY 
  	 EXTRACT(HOUR FROM order_time)
```

10. What was the volume of orders for each day of the week?

```
  SELECT
	 COUNT(*) AS total_orders 
       , TO_CHAR(order_time, 'DY') AS day
    FROM
  	 pizza_runner.customer_orders
GROUP BY
	 TO_CHAR(order_time, 'DY')
ORDER BY
	 COUNT(*) DESC
```

