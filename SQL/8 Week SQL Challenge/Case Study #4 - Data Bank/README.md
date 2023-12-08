## Case Study #4: Data Bank

<img src="https://user-images.githubusercontent.com/81607668/130343294-a8dcceb7-b6c3-4006-8ad2-fab2f6905258.png" alt="Image" width="500" height="520">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Techniques Used](#techniques-used)
- [Question and Solution](#question-and-solution)


***

## Business Task
Danny launched a new initiative, Data Bank which runs **banking activities** and also acts as the world’s most secure distributed **data storage platform**!

Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. 

The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.

This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!

## Entity Relationship Diagram

<img width="631" alt="image" src="https://user-images.githubusercontent.com/81607668/130343339-8c9ff915-c88c-4942-9175-9999da78542c.png">

**Table 1: `regions`**

This regions table contains the `region_id` and their respective `region_name` values.

<img width="176" alt="image" src="https://user-images.githubusercontent.com/81607668/130551759-28cb434f-5cae-4832-a35f-0e2ce14c8811.png">

**Table 2: `customer_nodes`**

Customers are randomly distributed across the nodes according to their region. This random distribution changes frequently to reduce the risk of hackers getting into Data Bank’s system and stealing customer’s money and data!

<img width="412" alt="image" src="https://user-images.githubusercontent.com/81607668/130551806-90a22446-4133-45b5-927c-b5dd918f1fa5.png">

**Table 3: Customer Transactions**

This table stores all customer deposits, withdrawals and purchases made using their Data Bank debit card.

<img width="343" alt="image" src="https://user-images.githubusercontent.com/81607668/130551879-2d6dfc1f-bb74-4ef0-aed6-42c831281760.png">

***
## Techniques Used

| SQL Technique                                  | Count | Questions Using the Technique                                                                                                            |
|------------------------------------------------|-------|---------------------------------------------------------------------------------------------------------------------------------------------|
| **Aggregation Function (COUNT, SUM, AVG)**       | 9     | Question A.1, Question A.2, Question A.3, Question A.4, Question A.5, Question B.1, Question B.2, Question B.3, Question B.5             |
| **DISTINCT**                                   | 7     | Question A.1, Question A.2, Question A.3, Question A.5, Question B.1, Question B.2, Question B.5                                         |
| **GROUP BY**                                   | 3     | Question A.2, Question A.3, Question B.3                                                                                                  |
| **ORDER BY**                                   | 6     | Question A.2, Question A.3, Question A.5, Question B.2, Question B.3, Question B.4                                                      |
| **JOIN**                                       | 4     | Question A.3, Question A.5, Question B.4, Question B.5                                                                                    |
| **CASE Statement**                             | 2     | Question A.4, Question B.10                                                                                                              |
| **Window Function (LEAD, ROW_NUMBER, RANK)**    | 5     | Question A.5, Question B.4, Question B.5, Question B.6, Question B.11                                                                     |
| **Common Table Expressions (WITH clause)**      | 5     | Question A.4, Question A.5, Question B.4, Question B.6, Question B.9                                                                      |
| **Subquery (Scalar Subquery)**                 | 4     | Question A.5, Question B.6, Question B.8, Question B.9                                                                                    |
| **Date Functions (Extract)**                   | 3     | Question A.2, Question A.3, Question B.11                                                                                               |
| **Filtering Rows (WHERE clause)**              | 9     | Question A.3, Question A.5, Question B.4, Question B.6, Question B.7, Question B.8, Question B.9, Question B.10, Question B.11            |
| **Percentage Calculation**                    | 6     | Question A.4, Question A.5, Question B.4, Question B.5, Question B.6, Question B.11                                                      |
| **WITH clause for Multi-step Processing**      | 4     | Question A.4, Question A.5, Question B.6, Question B.9                                                                                   |


***
## Question and Solution


## 🏦 A. Customer Nodes Exploration

1.    How many unique nodes are there on the Data Bank system?
```sql
    SELECT Count (DISTINCT node_id)
    FROM   data_bank.customer_nodes;
```
| count |
| ----- |
| 5     |

2.    What is the number of nodes per region?
```sql
    SELECT region_name,
               Count(distinct node_id)
        FROM   data_bank.customer_nodes
               JOIN data_bank.regions
                 ON( customer_nodes.region_id = regions.region_id )
        GROUP  BY region_name;
```
| region_name | count |
| ----------- | ----- |
| Africa      | 5     |
| America     | 5     |
| Asia        | 5     |
| Australia   | 5     |
| Europe      | 5     |


3.    How many customers are allocated to each region?
```sql
    SELECT region_name,
           Count(distinct customer_id)
    FROM   data_bank.customer_nodes
           JOIN data_bank.regions
             ON( customer_nodes.region_id = regions.region_id )
    GROUP  BY region_name;
```
| region_name | count |
| ----------- | ----- |
| Africa      | 102   |
| America     | 105   |
| Asia        | 95    |
| Australia   | 110   |
| Europe      | 88    |


4.    How many days on average are customers reallocated to a different node?
```sql
    WITH cte
         AS (SELECT node_id,
                    Lead(node_id, 1)
                      OVER(
                        partition BY customer_id) AS new_node,
                    start_date,
                    end_date
             FROM   data_bank.customer_nodes
                    JOIN data_bank.regions
                      ON( customer_nodes.region_id = regions.region_id )
             WHERE  Extract(year FROM end_date) <> 9999)
    SELECT Avg(end_date - start_date)
    FROM   cte
    WHERE  node_id <> new_node;
```
| avg                 |
| ------------------- |
| 14.8960910440376051 |


5.    What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
```sql
    WITH cte
         AS (SELECT node_id,
                    Lead(node_id, 1)
                      OVER(
                        partition BY customer_id) AS new_node,
                    start_date,
                    end_date
             FROM   data_bank.customer_nodes
                    JOIN data_bank.regions
                      ON( customer_nodes.region_id = regions.region_id )
             WHERE  Extract(year FROM end_date) <> 9999)
    SELECT
    
    	PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY end_date - start_date) AS median,
        
        PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY end_date - start_date) AS Eighty,
    
        PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY end_date - start_date) AS NinetyFive
    FROM
        cte
    WHERE
        node_id <> new_node;
```
| median | eighty | ninetyfive |
| ------ | ------ | ---------- |
| 15     | 24     | 28         |

***
## B. Customer Transactions
1.    What is the unique count and total amount for each transaction type?
```sql
    SELECT txn_type, count( customer_id),sum(txn_amount)
    FROM   data_bank.customer_transactions  
    group by txn_type;
```
| txn_type   | count | sum     |
| ---------- | ----- | ------- |
| purchase   | 1617  | 806537  |
| deposit    | 2671  | 1359168 |
| withdrawal | 1580  | 793003  |

2.    What is the average total historical deposit counts and amounts for all customers?
```sql
    WITH cte
         AS (SELECT DISTINCT customer_id,
                             Count(txn_type)
                               OVER(
                                 partition BY customer_id) AS deposit_count,
                             Avg(txn_amount)
                               OVER(
                                 partition BY customer_id) AS deposit_sum
             FROM   data_bank.customer_transactions
             WHERE  txn_type = 'deposit')
    SELECT Avg(deposit_count),
           Avg(deposit_sum)
    FROM   cte;
```
| avg                | avg                  |
| ------------------ | -------------------- |
| 5.3420000000000000 | 508.6127820956820957 |

3.    For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```sql
    WITH cte
         AS (SELECT Extract(month FROM txn_date) AS month,
                    ( customer_id ),
                    Sum(CASE
                          WHEN txn_type = 'deposit' THEN 1
                          ELSE 0
                        END)                     AS deposits,
                    Sum(CASE
                          WHEN txn_type <> 'deposit' THEN 1
                          ELSE 0
                        END)                     AS purch_width
             FROM   data_bank.customer_transactions
             GROUP  BY Extract(month FROM txn_date),
                       customer_id
             HAVING Sum(CASE
                          WHEN txn_type = 'deposit' THEN 1
                          ELSE 0
                        END) > 1
                    AND Sum(CASE
                              WHEN txn_type <> 'deposit' THEN 1
                              ELSE 0
                            END) = 1)
    SELECT month,
           Count(customer_id)
    FROM   cte
    GROUP  BY month;
```
| month | count |
| ----- | ----- |
| 1     | 53    |
| 2     | 36    |
| 3     | 38    |
| 4     | 22    |

4.    What is the closing balance for each customer at the end of the month?
```sql
    WITH cte
             AS (SELECT txn_date,
                        Extract(month FROM txn_date) AS month,
                        customer_id,
                        Sum(CASE
                              WHEN txn_type = 'deposit' THEN txn_amount
                              ELSE 0
                            END)                     AS deposits,
                        -Sum(CASE
                               WHEN txn_type <> 'deposit' THEN txn_amount
                               ELSE 0
                             END)                    AS out
                 FROM   data_bank.customer_transactions
                 GROUP  BY txn_date,
                           customer_id),
             running_date
             AS (SELECT txn_date,
                        month,
                        customer_id,
                        ( Sum(deposits) OVER ( partition BY customer_id ORDER BY
                          txn_date )
                          + Sum(out) OVER ( partition BY customer_id ORDER BY txn_date )
                        ) AS
                        running_total,
                        Rank()
                          OVER (
                            partition BY month, customer_id
                            ORDER BY txn_date DESC)
                        AS
                           rank_date
                 FROM   cte)
        SELECT customer_id,
               month,
               running_total
        FROM   running_date
        WHERE  rank_date = 1
        LIMIT 20;
```
| customer_id | month | running_total |
| ----------- | ----- | ------------- |
| 1           | 1     | 312           |
| 1           | 3     | -640          |
| 2           | 1     | 549           |
| 2           | 3     | 610           |
| 3           | 1     | 144           |
| 3           | 2     | -821          |
| 3           | 3     | -1222         |
| 3           | 4     | -729          |
| 4           | 1     | 848           |
| 4           | 3     | 655           |
| 5           | 1     | 954           |
| 5           | 3     | -1923         |
| 5           | 4     | -2413         |
| 6           | 1     | 733           |
| 6           | 2     | -52           |
| 6           | 3     | 340           |
| 7           | 1     | 964           |
| 7           | 2     | 3173          |
| 7           | 3     | 2533          |
| 7           | 4     | 2623          |

5.    What is the percentage of customers who increase their closing balance by more than 5%?
```sql
    WITH cte
         AS (SELECT txn_date,
                    Extract(month FROM txn_date) AS month,
                    customer_id,
                    SUM(CASE
                          WHEN txn_type = 'deposit' THEN txn_amount
                          ELSE 0
                        END)                     AS deposits,
                    -SUM(CASE
                           WHEN txn_type <> 'deposit' THEN txn_amount
                           ELSE 0
                         END)                    AS OUT
             FROM   data_bank.customer_transactions
             GROUP  BY txn_date,
                       customer_id),
         running_date
         AS (SELECT txn_date,
                    month,
                    customer_id,
                    ( SUM(deposits) over (PARTITION BY customer_id ORDER BY txn_date
                      )
                      + SUM(OUT) over (PARTITION BY customer_id ORDER BY txn_date) )
                    AS
                    running_total,
                    Rank()
                      over (
                        PARTITION BY month, customer_id
                        ORDER BY txn_date DESC)
                    AS
                       rank_date
             FROM   cte),
         closing_balance
         AS (SELECT customer_id,
                    month,
                    running_total,
                    Lead(running_total)
                      over (
                        ORDER BY month) AS next_month_close
             FROM   running_date
             WHERE  rank_date = 1),
         percent_change
         AS (SELECT customer_id,
                    SUM(( CASE
                            WHEN ( next_month_close - running_total ) /
                                 Abs(running_total) >
                                 .05
                          THEN 1
                            ELSE 0
                          END )) AS five_percent_change
             FROM   closing_balance
             WHERE  running_total <> 0
             GROUP  BY customer_id),
         count_five_percent
         AS (SELECT Count(customer_id) AS customers
             FROM   percent_change
             WHERE  five_percent_change > 0)
    SELECT customers / (SELECT Count(DISTINCT customer_id)
                        FROM   data_bank.customer_transactions) :: REAL AS
           percentage_of_customers_with_five_percent_increase
    FROM   count_five_percent;
```
| percentage_of_customers_with_five_percent_increase |
| -------------------------------------------------- |
| 0.826                                              |

