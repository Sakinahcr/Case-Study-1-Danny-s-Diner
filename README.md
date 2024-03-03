# Case Study #1 - Danny's Diner

## Table of Contents
- Problem statement
- Entity Relationship Diagram
- Questions and Solutions

  
## Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite

## Entity Relationship Diagram

<kbd>![image](https://github.com/Sakinahcr/Case-Study-1-Danny-s-Diner/assets/132161850/37d0d55d-8057-43d8-ad39-ff938a0e4ad3)


## Questions and Solutions

**1. What is the total amount each customer spent at the restaurant?**
```sql
SELECT 
    s.customer_id, SUM(n.price) AS total_spent
FROM
    sales s
        JOIN
    menu n ON s.product_id = n.product_id
GROUP BY customer_id
ORDER BY total_spent DESC;
```
**Steps**

**Findings**

<kbd>![image](https://github.com/Sakinahcr/Case-Study-1-Danny-s-Diner/assets/132161850/de24169d-38b8-4eb5-a7ab-4557a56fa70c)

**2. How many days has each customer visited the restaurant?**

```sql
SELECT 
    customer_id, COUNT(DISTINCT (order_date)) AS days_visited
FROM
    sales
GROUP BY customer_id
ORDER BY days_visited DESC;
```
**Steps**

**Findings**

<kbd>![image](https://github.com/Sakinahcr/Case-Study-1-Danny-s-Diner/assets/132161850/ac4affa3-d968-483e-b71d-ae3d32915390)

**3. What was the first item from the menu purchased by each customer?**

```sql
WITH order_rank AS (SELECT 
	s.customer_id, 
	n.product_name, 
	s.order_date, 
	DENSE_RANK () OVER (PARTITION BY customer_id ORDER BY order_date) AS num_rank
FROM 
	sales s
JOIN menu n ON s.product_id = n.product_id)
SELECT 
    customer_id, product_name, order_date
FROM
    order_rank
WHERE
    num_rank = 1
GROUP BY customer_id , product_name , order_date; 
```   
**Steps**

**Findings**

<kbd>![image](https://github.com/Sakinahcr/Case-Study-1-Danny-s-Diner/assets/132161850/81ba65b5-f853-4250-9162-aa0246a4058b)

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
```sql
WITH most_purchased AS 
(SELECT 
    s.product_id,
    n.product_name,
    COUNT(s.order_date) AS count_of_purchase
FROM
    sales s
        JOIN
    menu n ON s.product_id = n.product_id
GROUP BY product_id , product_name
ORDER BY count_of_purchase DESC
LIMIT 1)
SELECT 
    s.customer_id,
    p.product_name,
    COUNT(s.order_date) AS number_of_purchase
FROM
    sales s
        JOIN
    most_purchased p ON s.product_id = p.product_id
GROUP BY s.customer_id , p.product_name; 
```

**Steps**

**Findings**

<kbd>![image](https://github.com/Sakinahcr/Case-Study-1-Danny-s-Diner/assets/132161850/27b16242-b0ca-4bdb-b6fb-2c21579cfbcf)


**5. Which item was the most popular for each customer?**

```sql
WITH most_popular AS (
SELECT 
	s.customer_id, 
	n.product_name, 
	count(order_date) AS order_count,
	DENSE_RANK () OVER (PARTITION BY customer_id ORDER BY count(order_date) DESC) AS order_rank
FROM 
	sales s
JOIN 
	menu n ON s.product_id = n.product_id
GROUP BY s.customer_id, product_name)
SELECT 
    customer_id, product_name, order_count
FROM
    most_popular
WHERE
    order_rank = 1;
```
**Steps**

**Findings** 	

<kbd>![image](https://github.com/Sakinahcr/Case-Study-1-Danny-s-Diner/assets/132161850/6a9657af-1f46-4dee-87da-739d0bf8788e)


**6. Which item was purchased first by the customer after they became a member?**

```sql
WITH purchase_after_membership AS (
SELECT 
	m.customer_id, 
    s.product_id, 
    DENSE_RANK () OVER (PARTITION BY m.customer_id ORDER BY s.order_date) AS order_rank
FROM 
	members m
JOIN 
	sales s ON m.customer_id = s.customer_id AND s.order_date > m.join_date)
SELECT 
    p.customer_id, n.product_name
FROM
    purchase_after_membership p
        JOIN
    menu n ON p.product_id = n.product_id
WHERE
    order_rank = 1
ORDER BY customer_id;
```

**Steps**

**Findings** 

<kbd>![image](https://github.com/Sakinahcr/Case-Study-1-Danny-s-Diner/assets/132161850/65aca734-d26e-47bb-bb55-2f172caae46d)

**7. Which item was purchased just before the customer became a member?**

```sql
WITH purchase_after_membership AS (
SELECT 
	m.customer_id, 
    s.product_id, 
    DENSE_RANK () OVER (PARTITION BY m.customer_id ORDER BY s.order_date DESC) AS order_rank
FROM 
	members m
JOIN 
	sales s ON m.customer_id = s.customer_id
AND s.order_date < m.join_date)
SELECT 
    p.customer_id, n.product_name
FROM
    purchase_after_membership p
        JOIN
    menu n ON p.product_id = n.product_id
WHERE
    order_rank = 1
ORDER BY customer_id;
```

**Steps**

**Findings** 

<kbd>![image](https://github.com/Sakinahcr/Case-Study-1-Danny-s-Diner/assets/132161850/e8d31e07-e01d-4e1b-9bb1-08ea82128b4a)


**8. What is the total items and amount spent for each member before they became a member?**

```sql
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
```
**Steps**

**Findings** 

<kbd>![image](https://github.com/Sakinahcr/Case-Study-1-Danny-s-Diner/assets/132161850/627b97b2-69cc-455a-91f9-247db87bcfea)


**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

```sql
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
```
**Steps**

**Findings** 

<kbd>![image](https://github.com/Sakinahcr/Case-Study-1-Danny-s-Diner/assets/132161850/db3feb74-cf69-4440-9ff3-dbe10e890869)


**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**


```sql
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
**Steps**

**Findings** 

<kbd>![image](https://github.com/Sakinahcr/Case-Study-1-Danny-s-Diner/assets/132161850/b603318f-9264-4666-aa40-9ee4da1bf337)




