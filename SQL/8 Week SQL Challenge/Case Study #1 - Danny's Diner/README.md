# 🍜 Case Study #1: Danny's Diner 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Techniques Used](#techniques-used)
- [Solution](#solution)



***

## Business Task
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. 

***

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

***

## Techniques Used
| Technique                                | Count of Usage | Queries                               |
|------------------------------------------|----------------|---------------------------------------|
| DISTINCT Keyword                         | 4              | Query #1, Query #2, Query #5, Query #9 |
| SUM() Function                           | 7              | Query #1, Query #8, Query #9, Query #10|
| JOIN Operation                           | 17             | All queries except Query #4, Query #10 |
| GROUP BY Clause                          | 11             | Query #1, Query #2, Query #3, Query #4, Query #5, Query #6, Query #8, Query #9, Query #10 |
| ORDER BY Clause                          | 8              | Query #1, Query #2, Query #3, Query #4, Query #5, Query #6, Query #7, Query #9 |
| COUNT() Function                         | 7              | Query #2, Query #4, Query #5, Query #6, Query #8, Query #9, Query #10 |
| Common Table Expression (CTE)            | 4              | Query #3, Query #5, Query #6, Query #7 |
| Window Functions (RANK(), Dense_Rank())  | 4              | Query #3, Query #5, Query #6, Query #7 |
| CASE Statement                           | 3              | Query #5, Query #9, Query #10          |
| LIMIT Clause                             | 1              | Query #4                               |
| Date Functions (TO_CHAR(), Extract())    | 2              | Query #3, Query #10                    |
| PARTITION BY Clause                      | 1              | Query #5                               |


***

## Solution

**1. What is the total amount each customer spent at the restaurant?**
```sql
    SELECT DISTINCT customer_id,
                    Sum(price) AS total_price
    FROM   dannys_diner.sales
           JOIN dannys_diner.menu
             ON ( sales.product_id = menu.product_id )
    GROUP  BY customer_id
    ORDER  BY customer_id;
```
| customer_id | total_price |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

**2. How many days has each customer visited the restaurant?**
```sql
    SELECT DISTINCT customer_id,
                    count(distinct order_date) AS days_visited
    FROM   dannys_diner.sales
           JOIN dannys_diner.menu
             ON ( sales.product_id = menu.product_id )
    GROUP  BY customer_id
    ORDER  BY customer_id;
```
| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |

**3. What was the first item from the menu purchased by each customer?**
```sql
    WITH first_order
         AS (SELECT customer_id,
                    product_name,
                    order_date,
                    Rank()
                      OVER(
                        ORDER BY order_date) AS first_order_date
             FROM   dannys_diner.sales
                    JOIN dannys_diner.menu
                      ON ( sales.product_id = menu.product_id ))
    SELECT customer_id,
           product_name,
           To_char(order_date, 'Month DDth, YYYY') AS month
    FROM   first_order
    WHERE  first_order_date = 1
    GROUP  BY first_order.customer_id,
              first_order.product_name,
              first_order.order_date;
```
| customer_id | product_name | month                |
| ----------- | ------------ | -------------------- |
| A           | curry        | January   01st, 2021 |
| A           | sushi        | January   01st, 2021 |
| B           | curry        | January   01st, 2021 |
| C           | ramen        | January   01st, 2021 |

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
```sql
    SELECT product_name,
           Count(sales.product_id) AS ordered_count
    FROM   dannys_diner.sales
           JOIN dannys_diner.menu
             ON ( sales.product_id = menu.product_id )
    GROUP  BY product_name
    ORDER  BY ordered_count DESC
    LIMIT  1;
```
| product_name | ordered_count |
| ------------ | ------------- |
| ramen        | 8             |

**5. Which item was the most popular for each customer?**
```sql
    with ranking as(SELECT 
    customer_id,
    product_name,
    count(sales.product_id) as purchase_count,
    rank() over(partition by customer_id order by count(sales.product_id) desc) as purchase_ranking
    FROM   dannys_diner.sales
           JOIN dannys_diner.menu
             ON ( sales.product_id = menu.product_id )
    GROUP BY customer_id, menu.product_name,sales.product_id
    ORDER BY customer_id, menu.product_name,purchase_count DESC)
    
    select customer_id,product_name,purchase_count
    from ranking
    where purchase_ranking = 1;
```
| customer_id | product_name | purchase_count |
| ----------- | ------------ | -------------- |
| A           | ramen        | 3              |
| B           | curry        | 2              |
| B           | ramen        | 2              |
| B           | sushi        | 2              |
| C           | ramen        | 3              |

**6. Which item was purchased first by the customer after they became a member?**
```sql
    WITH member_date
         AS (SELECT sales.customer_id,
                    product_name,
                    Dense_rank()
                      OVER(
                        partition BY sales.customer_id
                        ORDER BY order_date) AS rank_date
             FROM   dannys_diner.sales
                    JOIN dannys_diner.menu
                      ON ( sales.product_id = menu.product_id )
                    JOIN dannys_diner.members
                      ON ( members.customer_id = sales.customer_id )
             WHERE  sales.order_date > members.join_date)
    SELECT *
    FROM   member_date
    WHERE  rank_date = 1;
```
| customer_id | product_name | rank_date |
| ----------- | ------------ | --------- |
| A           | ramen        | 1         |
| B           | sushi        | 1         |

**7. Which item was purchased just before the customer became a member?**
```sql
    WITH member_date
         AS (SELECT sales.customer_id,
                    product_name,
                    Dense_rank()
                      OVER(
                        partition BY sales.customer_id
                        ORDER BY order_date desc) AS rank_date
             FROM   dannys_diner.sales
                    JOIN dannys_diner.menu
                      ON ( sales.product_id = menu.product_id )
                    JOIN dannys_diner.members
                      ON ( members.customer_id = sales.customer_id )
             WHERE  sales.order_date < members.join_date)
    SELECT *
    FROM   member_date
    WHERE  rank_date = 1;
```
| customer_id | product_name | rank_date |
| ----------- | ------------ | --------- |
| A           | sushi        | 1         |
| A           | curry        | 1         |
| B           | sushi        | 1         |

**8. What is the total items and amount spent for each member before they became a member?**
```sql
    SELECT sales.customer_id,
           Count(product_name),
           Sum(price)
    FROM   dannys_diner.sales
           JOIN dannys_diner.menu
             ON ( sales.product_id = menu.product_id )
           JOIN dannys_diner.members
             ON ( members.customer_id = sales.customer_id )
    WHERE  sales.order_date < members.join_date
    GROUP  BY sales.customer_id;
```
| customer_id | count | sum |
| ----------- | ----- | --- |
| B           | 3     | 40  |
| A           | 2     | 25  |

**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```sql
    WITH points
         AS (SELECT *,
                    CASE
                      WHEN product_name = 'sushi' THEN price * 20
                      ELSE price * 10
                    END AS Points
             FROM   dannys_diner.menu)
    SELECT sales.customer_id,
           Sum(points.points) AS Points
    FROM   dannys_diner.sales
           JOIN points
             ON points.product_id = sales.product_id
    GROUP  BY sales.customer_id
    ORDER  BY points DESC;
```
| customer_id | points |
| ----------- | ------ |
| B           | 940    |
| A           | 860    |
| C           | 360    |

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
```sql
    SELECT sales.customer_id,
           Sum(CASE
                 WHEN product_name = 'sushi'
                       OR ( ( Extract(day FROM order_date) -
                              Extract(day FROM join_date) )
                            BETWEEN 0 AND 7
                          ) THEN price * 20
                 ELSE price * 10
               END) AS points
    FROM   dannys_diner.sales
           JOIN dannys_diner.menu
             ON ( sales.product_id = menu.product_id )
           JOIN dannys_diner.members
             ON ( members.customer_id = sales.customer_id )
    WHERE  Date_trunc('month', order_date) = '2021-01-01'
    GROUP  BY sales.customer_id;
```
| customer_id | points |
| ----------- | ------ |
| B           | 940    |
| A           | 1370   |
