# 8 Weeks SQL Challenge - 1st Week

### This [case study](https://8weeksqlchallenge.com/case-study-1/)  is provided by Danny Ma 


## Introduction
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.


## Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

 * sales
 * menu
 * members

*See the complete tables [here](https://8weeksqlchallenge.com/case-study-1/)*

## Case Study Questions
Each of the following case study questions can be answered using a single SQL statement:

* What is the total amount each customer spent at the restaurant?
* How many days has each customer visited the restaurant?
* What was the first item from the menu purchased by each customer?
* What is the most purchased item on the menu and how many times was it purchased by all customers?
* Which item was the most popular for each customer?
* Which item was purchased first by the customer after they became a member?
* Which item was purchased just before the customer became a member?
* What is the total items and amount spent for each member before they became a member?
* If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
* In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

## Create Tables
```sql
CREATE TABLE IF NOT EXISTS sales (
  customer_id VARCHAR(1),
  order_date DATE,
  product_id INTEGER
);

CREATE TABLE menu (
product_id INTEGER PRIMARY KEY,
product_name VARCHAR(100),
price INTEGER
);

CREATE TABLE members (
customer_id VARCHAR(1),
join_date DATE
);
```

```sql
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

INSERT INTO menu 
  (product_id, product_name, price)
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  
INSERT INTO members
  (customer_id, join_date)
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
```

## Problem solving
1. What is the total amount each customer spent at the restaurant?
```sql
WITH temp AS (
SELECT sales.customer_id, sales.product_id, menu.price
FROM sales
JOIN menu
ON sales.product_id = menu.product_id
)
SELECT temp.customer_id, sum(price) AS total_amount
FROM temp
GROUP BY customer_id
```
| customer_id | total_amount
|---|---
|A|76
|B|74
|C|36

2. How many days has each customer visited the restaurant?
```sql
SELECT customer_id, COUNT(order_date) AS day_visit
FROM sales
GROUP BY customer_id
```
| customer_id | day_visit
|---|---
|A|6
|B|6
|C|3

3. What was the first item from the menu purchased by each customer?
```sql
SELECT sales.customer_id, sales.order_date, sales.product_id, menu.product_name
FROM sales
JOIN menu
ON sales.product_id = menu.product_id
ORDER BY order_date, customer_id ASC
LIMIT 5
```
| customer_id | order_date | product_id | product_name
|---|---|---|---
| A | 2021-01-01 | 1 | sushi
| A | 2021-01-01 | 2 | curry
| B | 2021-01-01 | 2 | curry
| C | 2021-01-01 | 3 | ramen
| C | 2021-01-01 | 3 | ramen

4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
WITH temp AS (
SELECT sales.customer_id, sales.order_date, sales.product_id, menu.product_name
FROM sales
JOIN menu
ON sales.product_id = menu.product_id
)
SELECT product_name, count(product_name) AS order_times
FROM temp
GROUP BY product_name
ORDER BY order_times DESC
```
| product_name | order_times
|---|---
|ramen|8
|curry|4
|sushi|3

5. Which item was the most popular for each customer?
```sql
WITH temp AS (
SELECT sales.customer_id, sales.product_id, menu.product_name
FROM sales
JOIN menu
ON sales.product_id = menu.product_id
)
SELECT customer_id, product_name, COUNT(product_name) AS order_time
FROM temp
GROUP BY customer_id, product_name
ORDER BY customer_id ASC, order_time DESC 
```
customer_id | product_name | order_times
|---|---|---
| A | ramen | 3 
| A | curry | 2
| A | sushi | 1
| B | curry | 2
| B | sushi | 2
| B | ramen | 2
| C | ramen | 3

6. Which item was purchased first by the customer after they became a member?
```sql
SELECT sales.customer_id, sales.order_date, menu.product_name, members.join_date
FROM sales
JOIN menu
ON sales.product_id = menu.product_id
LEFT JOIN members
ON sales.customer_id = members.customer_id
WHERE order_date >= join_date
ORDER BY order_date 
)
```
| customer_id | order_date | product_name | join_date
|---|---|---|---
| A | 2021-01-07 | curry | 2021-01-07
| A | 2021-01-10 | ramen | 2021-01-07
| A | 2021-01-11 | ramen | 2021-01-07
| A | 2021-01-11 | ramen | 2021-01-07
| B | 2021-01-11 | sushi | 2021-01-09
| B | 2021-01-16 | ramen | 2021-01-09
| B | 2021-02-01 | ramen | 2021-01-09

7. Which item was purchased just before the customer became a member?
```sql
SELECT sales.customer_id, sales.order_date, menu.product_name, members.join_date
FROM sales
JOIN menu
ON sales.product_id = menu.product_id
LEFT JOIN members
ON sales.customer_id = members.customer_id
WHERE order_date < join_date
ORDER BY order_date DESC
```
| customer_id | order_date | product_name | join_date
|---|---|---|---
| B | 2021-01-04 | sushi | 2021-01-09
| B | 2021-01-02 | curry | 2021-01-09
| A | 2021-01-01 | sushi | 2021-01-07
| A | 2021-01-01 | curry | 2021-01-07
| B | 2021-01-01 | curry | 2021-01-09

8. What is the total items and amount spent for each member before they became a member?
```sql
WITH temp AS (
SELECT sales.customer_id, sales.order_date, menu.product_name, members.join_date, menu.price
FROM sales
JOIN menu
ON sales.product_id = menu.product_id
LEFT JOIN members
ON sales.customer_id = members.customer_id
WHERE order_date < join_date
)
SELECT customer_id, count(product_name) AS order_times, sum(price) AS total_amount
FROM temp
GROUP BY customer_id
```
customer_id | order_times | total_amount
|---|---|---
| A | 2 | 25 
| B | 3 | 40

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
ALTER TABLE menu
ADD price_point INTEGER
```
```sql
UPDATE menu
SET price_point = CASE
WHEN product_name = 'sushi' THEN
price * 20
ELSE
price *10
END
WHERE price_point IS NULL
```
```sql
WITH temp AS
(SELECT sales.customer_id, menu.price_point
FROM sales
JOIN menu
ON sales.product_id = menu.product_id
)
SELECT customer_id, SUM(price_point) AS total_point
FROM temp
GROUP BY customer_id
```
customer_id | total_point
|---|---
| A | 860
| B | 940
| C | 360

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
WITH temp2 AS (
WITH temp1 AS (
SELECT sales.customer_id, sales.order_date, menu.product_name, menu.price, members.join_date, price*10 AS single_price_point
FROM sales
JOIN menu
ON sales.product_id = menu.product_id
LEFT JOIN members
ON sales.customer_id = members.customer_id
)
SELECT Customer_id, product_name,
CASE WHEN order_date >= join_date
THEN single_price_point *2
WHEN order_date < join_date AND product_name = 'sushi'
THEN single_price_point * 2
ELSE single_price_point * 1
END AS final_point
FROM temp1
WHERE join_date IS NOT NULL)
SELECT customer_id, SUM(final_point) AS Final_points
FROM temp2
GROUP BY customer_id
```