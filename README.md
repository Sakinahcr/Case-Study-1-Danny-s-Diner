# Case Study #1:Danny's Diner

# Questions and Solutions
```sql
-- to create database and table

CREATE DATABASE IF NOT EXISTS dannys_diner;

CREATE TABLE sales (
    customer_id VARCHAR(1),
    order_date DATE,
    product_id INTEGER
);

INSERT INTO sales
  (customer_id, order_date, product_id)
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
 

CREATE TABLE menu (
    product_id INTEGER,
    product_name VARCHAR(5),
    price INTEGER
);

INSERT INTO menu
  (product_id, product_name, price)
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE members (
    customer_id VARCHAR(1),
    join_date DATE
);

INSERT INTO members
  (customer_id, join_date)
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
  
  
  /* --------------------
   Case Study Questions
   --------------------*/

SELECT 
    s.customer_id, SUM(n.price) AS total_spent
FROM
    sales s
        JOIN
    menu n ON s.product_id = n.product_id
GROUP BY customer_id
ORDER BY total_spent DESC;

-- 2. How many days has each customer visited the restaurant?
SELECT 
    customer_id, COUNT(DISTINCT (order_date)) AS days_visited
FROM
    sales
GROUP BY customer_id
ORDER BY days_visited DESC;

-- 3. What was the first item from the menu purchased by each customer?
WITH order_rank AS (SELECT 
	s.customer_id, 
	n.product_name, 
	s.order_date, 
	DENSE_RANK () OVER (PARTITION BY customer_id ORDER BY order_date) AS num_rank
FROM 
	sales s
JOIN menu n ON s.product_id = n.product_id)
SELECT customer_id, product_name, order_date
FROM order_rank
WHERE num_rank = 1
GROUP BY customer_id, product_name, order_date;  
    

-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

WITH most_purchased AS 
(SELECT s.product_id, n.product_name, COUNT(s.order_date) AS count_of_purchase
FROM sales s
JOIN menu n ON s.product_id = n.product_id
GROUP BY product_id, product_name
ORDER BY count_of_purchase DESC
LIMIT 1)
SELECT s.customer_id, p.product_name, count(s.order_date) AS number_of_purchase
FROM sales s
JOIN most_purchased p ON s.product_id = p.product_id
GROUP BY s.customer_id, p.product_name;


-- 5. Which item was the most popular for each customer?

WITH most_popular AS (SELECT s.customer_id, 
n.product_name, 
count(order_date) AS order_count,
DENSE_RANK () OVER (PARTITION BY customer_id ORDER BY count(order_date) DESC) AS order_rank
FROM sales s
JOIN menu n ON s.product_id = n.product_id
GROUP BY s.customer_id, product_name)
SELECT customer_id, product_name, order_count
FROM most_popular
WHERE order_rank = 1;


-- 6. Which item was purchased first by the customer after they became a member?

WITH purchase_after_membership AS (
SELECT m.customer_id, s.product_id, DENSE_RANK () OVER (PARTITION BY m.customer_id ORDER BY s.order_date) AS order_rank
FROM members m
JOIN sales s ON m.customer_id = s.customer_id
AND s.order_date > m.join_date)
SELECT p.customer_id, n.product_name
FROM purchase_after_membership p
JOIN menu n ON p.product_id = n.product_id
WHERE order_rank = 1
ORDER BY customer_id;


-- 7. Which item was purchased just before the customer became a member?
WITH purchase_after_membership AS (
SELECT m.customer_id, s.product_id, DENSE_RANK () OVER (PARTITION BY m.customer_id ORDER BY s.order_date DESC) AS order_rank
FROM members m
JOIN sales s ON m.customer_id = s.customer_id
AND s.order_date < m.join_date)
SELECT p.customer_id, n.product_name
FROM purchase_after_membership p
JOIN menu n ON p.product_id = n.product_id
WHERE order_rank = 1
ORDER BY customer_id;


-- 8. What is the total items and amount spent for each member before they became a member?

SELECT 
    s.customer_id,
    COUNT(n.product_name) AS total_item,
    SUM(n.price) AS amount_spent
FROM
    sales s
        JOIN
    menu n ON s.product_id = n.product_id
        JOIN
    members m ON s.customer_id = m.customer_id
        AND s.order_date < m.join_date
GROUP BY customer_id
ORDER BY customer_id;

-- 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

WITH points AS(SELECT 
    n.product_id,
    CASE
        WHEN n.product_name <> 'sushi' THEN n.price * 10
        ELSE 2 * (n.price * 10)
    END AS customer_points
FROM
    menu n)
SELECT 
    s.customer_id, SUM(p.customer_points) AS points_collected
FROM
    sales s
        JOIN
    points p ON s.product_id = p.product_id
GROUP BY customer_id;

-- 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

WITH customer_point AS
(SELECT 
    m.customer_id,
    s.order_date,
    m.join_date,
    CASE
        WHEN n.product_name = 'sushi' THEN n.price * 20
        WHEN s.order_date BETWEEN m.join_date AND DATE(join_date + 6) THEN n.price * 20
        ELSE n.price * 10
    END AS points
FROM
    sales s
        JOIN
    members m ON s.customer_id = m.customer_id
        JOIN
    menu n ON s.product_id = n.product_id)
SELECT 
    customer_id, SUM(points) AS points_collected
FROM
    customer_point
WHERE
    order_date >= join_date
        AND order_date < '2021-01-31'
GROUP BY customer_id
ORDER BY customer_id;

```





