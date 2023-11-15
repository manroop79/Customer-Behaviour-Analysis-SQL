# Restaurant SQL Case Study

## Overview

This repository contains SQL queries and scripts to analyze the data provided by a Japanese restaurant that specializes in sushi, curry, and ramen. The goal is to help the restaurant gain insights into customer spending patterns, preferences, and behaviors. The queries use SQL features such as Window Functions, Common Table Expressions (CTEs), Aggregations, and JOINs to answer specific questions and generate useful reports.

## Problem Statement

The restaurant wants to use the data to answer a few simple questions about their customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favorite. Having this deeper connection with their customers will help them deliver a better and more personalized experience for their loyal customers.

They plan on using these insights to help them decide whether they should expand the existing customer loyalty program - additionally they needs help to generate some basic datasets so the team can easily inspect the data without needing to use SQL.

## Data Model

The data is structured into three tables:

- `sales`: Contains information about customer orders, including customer ID, order date, and product ID.
- `menu`: Provides details about menu items, such as product ID, product name, and price.
- `members`: Includes data on customer membership, including customer ID and join date.

## Entity Relationship Diagram

![image](https://github.com/Irene-arch/Customer-Behaviour-Analysis-MSSQL-Server/assets/56026296/5e0c9ea2-cfcf-400e-b164-6955b46f0e9f)

## Skills Applied

- Window Functions
- CTEs
- Aggregations
- JOINs
- Write scripts to generate basic reports that can be run every period

## Queries and Reports

The SQL queries and reports are organized based on the questions explored in the case study. Each query is designed to answer a specific question or generate a report related to customer spending, preferences, and loyalty program insights.

Feel free to explore the provided queries in the SQL scripts for detailed solutions.

## Usage

1. Create tables using the provided `CREATE TABLE` statements.
2. Insert data into tables using the provided `INSERT INTO` statements.
3. Run the queries in a SQL environment to analyze and generate reports.

## Questions Explored

1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

## Some Interesting Quesries

**Q - What is the first item purchased in the menu by each customer?**
 
```sql
SELECT sales.customer_id, sales.order_date, menu.product_name
FROM sales
JOIN menu ON sales.product_id = menu.product_id
JOIN (
    SELECT customer_id, MIN(order_date) AS first_order_date
    FROM sales
    GROUP BY customer_id
) first_orders
ON sales.customer_id = first_orders.customer_id 
AND sales.order_date = first_orders.first_order_date
ORDER BY sales.order_date ASC
LIMIT 3;
```


**Q - What were the items purchased by the customer after they became a member?**

```sql
SELECT members.customer_id, sales.order_date,
members.join_date, menu.product_name
FROM members
INNER JOIN sales
ON sales.customer_id = members.customer_id
INNER JOIN menu
ON sales.product_id = menu.product_id
WHERE sales.order_date >= members.join_date
```


**Q - In the first week after a customer joins the program they earn 2x points on all items (not just sushi) - how many points do customer A and B have at the end of January?**

```sql
SELECT 
    mb.customer_id,
    mb.join_date,
    SUM(m.price * CASE 
            WHEN mb.customer_id = 'A' AND s.order_date BETWEEN '2021-01-07' AND '2021-01-14' THEN 2
            WHEN mb.customer_id = 'B' AND s.order_date BETWEEN '2021-01-09' AND '2021-01-16' THEN 2
            WHEN mb.customer_id = 'C' AND s.order_date BETWEEN '2021-01-08' AND '2021-01-15' THEN 2
            ELSE 1
        END) AS total_points
FROM 
    members mb
JOIN 
    sales s ON s.customer_id = mb.customer_id
JOIN 
    menu m ON s.product_id = m.product_id
GROUP BY 
    mb.customer_id, mb.join_date;
```
    
## Insights

- Customer B is the most frequent visitor.
- The restaurant's most popular item is ramen, followed by curry and sushi.
- Customer A loves ramen, Customer C loves only ramen whereas Customer B seems to enjoy sushi, curry and ramen equally.
- The last item ordered by Customers A and B before they became members are sushi and curry.
