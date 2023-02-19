Case Study #1 - Danny's Diner
Date - 19/02/2023
Tools Used - PostgreSQL

-- 1. What is the total amount each customer spent at the restaurant?
SELECT sales.customer_id, SUM(price) AS Total_Amount
FROM sales
JOIN menu 
ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY total_amount DESC;

-- 2. How many days has each customer visited the restaurant?
SELECT customer_id, COUNT(DISTINCT(order_date)) AS number_of_visits
FROM sales
GROUP BY customer_id
ORDER BY number_of_visits DESC

--3. What was the first item from the menu purchased by each customer?

SELECT s.customer_id, MIN(order_date), m.product_name
FROM sales as s
INNER JOIN menu as m
ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name,s.order_date
ORDER BY s.order_date ASC
LIMIT 4

--4. What is the most purchased item on the menu and how many times was it purchased by all customers?

SELECT COUNT(sales.product_id) AS most_purchased, menu.product_name
FROM sales
INNER JOIN menu
ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY most_purchased DESC
LIMIT 1


--5. Which item was the most popular for each customer?

SELECT * FROM
(SELECT *, RANK () OVER (
	PARTITION BY dbe.customer_id
	ORDER BY dbe.order_count DESC) AS product_rank
FROM (
	SELECT sales.customer_id, COUNT(sales.product_id) AS order_count, menu.product_name
FROM sales
INNER JOIN menu
ON sales.product_id = menu.product_id
GROUP BY menu.product_name, sales.customer_id) dbe) AS OP
WHERE op.product_rank = 1

--6.  Which item was purchased first by the customer after they became a member?

SELECT * FROM
(SELECT *, RANK() OVER (
    PARTITION by all_data.customer_id
    ORDER BY all_data.order_date
) AS rank
FROM 
(
    SELECT  m.customer_id, mn.product_name, s.order_date, m.join_date
    FROM sales as s
    JOIN members as m
    ON s.customer_id = m.customer_id
    JOIN menu as mn
    ON s.product_id = mn.product_id
    WHERE s.order_date >= m.join_date
) all_data) AS OP
WHERE op.rank = 1

--7. Which item was purchased just before the customer became a member?

SELECT * FROM
(SELECT *, RANK() OVER (
    PARTITION by all_data.customer_id
    ORDER BY all_data.order_date DESC
) AS rank
FROM 
(
    SELECT  m.customer_id, mn.product_name, s.order_date, m.join_date
    FROM sales as s
    JOIN members as m
    ON s.customer_id = m.customer_id
    JOIN menu as mn
    ON s.product_id = mn.product_id
    WHERE s.order_date < m.join_date
) all_data) AS OP
WHERE op.rank = 1

--8. What is the total items and amount spent for each member before they became a member?

SELECT  m.customer_id, COUNT(DISTINCT(s.product_id)), SUM(mn.price) as amt_spent
    FROM sales as s
    JOIN members as m
    ON s.customer_id = m.customer_id
    JOIN menu as mn
    ON s.product_id = mn.product_id
    WHERE s.order_date < m.join_date
	GROUP BY m.customer_id
	
	
--9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

SELECT  s.customer_id,SUM(p.cust_points) as total_points
FROM
(SELECT *, CASE WHEN product_name = 'sushi' THEN price*20
ELSE price*10 END as cust_points
FROM menu) as p
JOIN sales as s
ON p.product_id = s.product_id
GROUP BY s.customer_id
ORDER BY total_points DESC


--10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

SELECT j_d.customer_id, SUM(cust_points)
FROM
(SELECT s.customer_id, s.order_date, mn.product_name, mn.price, me.join_date,
(CASE WHEN mn.product_name='sushi' THEN mn.price*20
WHEN s.order_date BETWEEN me.join_date AND me.join_date + integer '6' THEN mn.price*20
ELSE mn.price*10 END) as cust_points
FROM sales as s
JOIN menu as mn
ON s.product_id = mn.product_id
JOIN members as me
ON me.customer_id = s.customer_id
WHERE s.order_date < '2021-01-31'
GROUP BY s.customer_id, s.order_date, mn.product_name, mn.price, me.join_date) j_d
GROUP BY 1
ORDER BY 1

Bonus Questions

--11. The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.Recreate the following table output using the available data:

SELECT s.customer_id, s.order_date, mn.product_name, mn.price,
CASE WHEN join_date <= order_date THEN 'Y'
WHEN join_date > order_date THEN 'N'
ELSE 'N'
END as member
FROM sales as s
LEFT JOIN menu as mn
ON s.product_id = mn.product_id
LEFT JOIN members as me
ON me.customer_id = s.customer_id

--12.Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program. Let rank the table we just created.

SELECT *,
CASE WHEN member = 'N' THEN NULL
ELSE
RANK () OVER (
PARTITION BY customer_id, member
ORDER BY order_date ) END as rank
FROM
(SELECT s.customer_id, s.order_date, mn.product_name, mn.price,
CASE WHEN join_date <= order_date THEN 'Y'
WHEN join_date > order_date THEN 'N'
ELSE 'N'
END as member
FROM sales as s
LEFT JOIN menu as mn
ON s.product_id = mn.product_id
LEFT JOIN members as me
ON me.customer_id = s.customer_id) order_rank







 









