# Bradley's Portfolio

# Case studies taken from https://8weeksqlchallenge.com/
## Week 1
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
         customer_id
       , count(order_date) AS 'total_visits'
    FROM
         dannys_diner.sales
GROUP BY
         customer_id
ORDER BY 
	 COUNT(order_date) DESC
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
