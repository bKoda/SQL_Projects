# Bradley's Portfolio

# Case studies taken from https://8weeksqlchallenge.com/
## Week 1
### 1. What is the total amount each customer spent at the restaurant?

  SELECT
        customer_id
      , SUM(price) AS 'total_amt'
    FROM
        sales
    JOIN
        menu ON sales.product_id = menu.product_id
GROUP BY
        customer_id
ORDER BY
        SUM(price) DESC
        
### 2. How many days has each customer visited the restaurant?

  SELECT DISTINCT
         customer_id
       , count(order_date) AS 'total_visits'
    FROM
         sales
GROUP BY
         customer_id
ORDER BY 
		     COUNT(order_date) DESC
