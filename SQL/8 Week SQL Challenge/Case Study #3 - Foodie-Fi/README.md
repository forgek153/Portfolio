# 🥑 Case Study #3: Foodie-Fi

<img src="https://user-images.githubusercontent.com/81607668/129742132-8e13c136-adf2-49c4-9866-dec6be0d30f0.png" width="500" height="520" alt="image">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Techniques Used](#techniques-used)
- [Question and Solution](#question-and-solution)

***

## Business Task
Danny and his friends launched a new startup Foodie-Fi and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world.

This case study focuses on using subscription style digital data to answer important business questions on customer journey, payments, and business performances.

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/129744449-37b3229b-80b2-4cce-b8e0-707d7f48dcec.png)

**Table 1: `plans`**

<img width="207" alt="image" src="https://user-images.githubusercontent.com/81607668/135704535-a82fdd2f-036a-443b-b1da-984178166f95.png">

There are 5 customer plans.

- Trial — Customer sign up to an initial 7 day free trial and will automatically continue with the pro monthly subscription plan unless they cancel, downgrade to basic or upgrade to an annual pro plan at any point during the trial.
- Basic plan — Customers have limited access and can only stream their videos and is only available monthly at $9.90.
- Pro plan — Customers have no watch time limits and are able to download videos for offline viewing. Pro plans start at $19.90 a month or $199 for an annual subscription.

When customers cancel their Foodie-Fi service — they will have a Churn plan record with a null price, but their plan will continue until the end of the billing period.

**Table 2: `subscriptions`**

<img width="245" alt="image" src="https://user-images.githubusercontent.com/81607668/135704564-30250dd9-6381-490a-82cf-d15e6290cf3a.png">

Customer subscriptions show the **exact date** where their specific `plan_id` starts.

If customers downgrade from a pro plan or cancel their subscription — the higher plan will remain in place until the period is over — the `start_date` in the subscriptions table will reflect the date that the actual plan changes.

When customers upgrade their account from a basic plan to a pro or annual pro plan — the higher plan will take effect straightaway.

When customers churn, they will keep their access until the end of their current billing period, but the start_date will be technically the day they decided to cancel their service.

***
## Techniques Used

| SQL Technique                                  | Count | Questions Using the Technique                                                                                                            |
|------------------------------------------------|-------|---------------------------------------------------------------------------------------------------------------------------------------------|
| **Aggregation Function (COUNT, SUM, AVG)**       | 9     | Question #1, Question #2, Question #3, Question #4, Question #5, Question #6, Question #7, Question #8, Question #9                        |
| **DISTINCT**                                   | 7     | Question #1, Question #2, Question #3, Question #5, Question #6, Question #7, Question #11                                                |
| **GROUP BY**                                   | 3     | Question #2, Question #3, Question #7                                                                                                      |
| **ORDER BY**                                   | 6     | Question #2, Question #3, Question #6, Question #7, Question #8, Question #10                                                             |
| **JOIN**                                       | 4     | Question #3, Question #6, Question #7, Question #8                                                                                       |
| **CASE Statement**                             | 2     | Question #4, Question #10                                                                                                                 |
| **Window Function (LEAD, ROW_NUMBER, RANK)**    | 5     | Question #5, Question #6, Question #7, Question #8, Question #11                                                                         |
| **Common Table Expressions (WITH clause)**      | 5     | Question #5, Question #6, Question #7, Question #8, Question #9                                                                         |
| **Subquery (Scalar Subquery)**                 | 4     | Question #5, Question #6, Question #8, Question #9                                                                                       |
| **Date Functions (Extract)**                   | 3     | Question #2, Question #3, Question #11                                                                                                   |
| **Filtering Rows (WHERE clause)**              | 9     | Question #3, Question #5, Question #6, Question #7, Question #8, Question #9, Question #10, Question #11                                 |
| **Percentage Calculation**                    | 6     | Question #4, Question #5, Question #6, Question #7, Question #8, Question #11                                                            |
| **WITH clause for Multi-step Processing**      | 4     | Question #5, Question #6, Question #8, Question #9                                                                                        |

***
## Question and Solution

1.    How many customers has Foodie-Fi ever had?
```sql
    SELECT Count(DISTINCT customer_id)
    FROM   foodie_fi.subscriptions;
```
| count |
| ----- |
| 1000  |


2.    What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
```sql
    SELECT DISTINCT Extract(month FROM start_date)        AS month,
                    Count(Extract(month FROM start_date)) AS count
    FROM   foodie_fi.subscriptions
    GROUP  BY month
    ORDER  BY month;
```
| month | count |
| ----- | ----- |
| 1     | 236   |
| 2     | 195   |
| 3     | 245   |
| 4     | 217   |
| 5     | 214   |
| 6     | 204   |
| 7     | 221   |
| 8     | 235   |
| 9     | 225   |
| 10    | 230   |
| 11    | 208   |
| 12    | 220   |



3.    What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
```sql

    SELECT DISTINCT plan_name,
                    Count(subscriptions.plan_id) AS CoE
    FROM   foodie_fi.subscriptions
           JOIN foodie_fi.plans
             ON ( subscriptions.plan_id = plans.plan_id )
    WHERE  Extract(year FROM start_date) > 2020
    GROUP  BY plan_name
    ORDER  BY coe;
```

| plan_name     | coe |
| ------------- | --- |
| basic monthly | 8   |
| pro monthly   | 60  |
| pro annual    | 63  |
| churn         | 71  |

4.    What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```sql
    SELECT SUM(CASE
                 WHEN subscriptions.plan_id = 4 THEN 1
                 ELSE 0
               END)                                                           AS
           churned,
           SUM(CASE
                 WHEN subscriptions.plan_id = 4 THEN 1
                 ELSE 0
               END) / Count(DISTINCT subscriptions.customer_id) :: REAL * 100 AS
           churned_rate
    FROM   foodie_fi.subscriptions
           join foodie_fi.plans
             ON ( subscriptions.plan_id = plans.plan_id );
```
| churned | churned_rate |
| ------- | ------------ |
| 307     | 30.7         |

5.    How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```sql
    WITH cte
         AS (SELECT customer_id,
                    plan_id,
                    Lead(plan_id, 1)
                      over(
                        ORDER BY customer_id) AS next_plan
             FROM   foodie_fi.subscriptions
             ORDER  BY customer_id)
    SELECT Count(DISTINCT customer_id)                                         AS
           total_customers,
           Count(customer_id) * 100 / (SELECT Count(DISTINCT customer_id)
                                       FROM   foodie_fi.subscriptions) :: REAL AS
           percentage
    FROM   cte
    WHERE  plan_id = 0
           AND next_plan = 4;
```
| total_customers | percentage |
| --------------- | ---------- |
| 92              | 9.2        |

6.    What is the number and percentage of customer plans after their initial free trial?
```sql
    WITH cte_rank
         AS (SELECT s.customer_id,
                    s.plan_id,
                    Row_number()
                      OVER (
                        partition BY s.customer_id) AS ranking
             FROM   foodie_fi.subscriptions AS s)
    SELECT p.plan_name,
           c.plan_id,
           Count(c.customer_id)
           AS Customers,
           Round(Count(c.customer_id) * 100 / (SELECT Count(DISTINCT customer_id)
                                               FROM   foodie_fi.subscriptions), 1)
           AS
           PERCENT
    FROM   cte_rank AS c
           INNER JOIN foodie_fi.plans AS p
                   ON p.plan_id = c.plan_id
    WHERE  ranking = 2
    GROUP  BY p.plan_name,
              c.plan_id;
```
| plan_name     | plan_id | customers | percent |
| ------------- | ------- | --------- | ------- |
| pro monthly   | 2       | 325       | 32.0    |
| pro annual    | 3       | 37        | 3.0     |
| churn         | 4       | 92        | 9.0     |
| basic monthly | 1       | 546       | 54.0    |

7.    What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
```sql
    WITH cte
         AS (SELECT customer_id,
                    plan_id,
                    start_date,
                    Rank()
                      over (
                        PARTITION BY customer_id
                        ORDER BY start_date DESC)
             FROM   foodie_fi.subscriptions
             WHERE  start_date <= '2020-12-31')
    SELECT plan_name,
           Count(customer_id),
           ( Count(customer_id) :: REAL / (SELECT Count(DISTINCT customer_id)
                                           FROM   cte) ) * 100 as percent
    FROM   cte
           join foodie_fi.plans
             ON ( cte.plan_id = plans.plan_id )
    WHERE  rank = 1
    GROUP  BY plan_name;
```
| plan_name     | count | percent           |
| ------------- | ----- | ------------------ |
| pro annual    | 195   | 19.5               |
| trial         | 19    | 1.9                |
| churn         | 236   | 23.599999999999998 |
| basic monthly | 224   | 22.400000000000002 |
| pro monthly   | 326   | 32.6               |

8.    How many customers have upgraded to an annual plan in 2020?
```sql
    WITH cte
         AS (SELECT customer_id,
                    plan_id,
                    Lead(plan_id, 1)
                      over(
                        ORDER BY customer_id) AS next_plan,
                    start_date
             FROM   foodie_fi.subscriptions
             ORDER  BY customer_id)
    SELECT Count(DISTINCT customer_id)                                         AS
           total_customers,
           Count(customer_id) * 100 / (SELECT Count(DISTINCT customer_id)
                                       FROM   foodie_fi.subscriptions) :: REAL AS
           percentage
    FROM   cte
    WHERE  ( plan_id <> 0
             AND next_plan = 3 )
           AND Extract(year FROM start_date) = 2020;
```
| total_customers | percentage |
| --------------- | ---------- |
| 216             | 21.6       |


9.    How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
```sql
    WITH start_cte
         AS (SELECT customer_id,
                    start_date AS trial_date
             FROM   foodie_fi.subscriptions
             WHERE  plan_id = 0),
         annual_cte
         AS (SELECT customer_id,
                    start_date AS annual_date
             FROM   foodie_fi.subscriptions
             WHERE  plan_id = 3)
    SELECT ( Round(Avg(annual_cte.annual_date - start_cte.trial_date), 2) ) AS
           convert_days
    FROM   annual_cte
           JOIN start_cte
             ON( start_cte.customer_id = annual_cte.customer_id );
```
| convert_days |
| ------------ |
| 104.62       |

10.    Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
```sql
    WITH start_cte
         AS (SELECT customer_id,
                    start_date AS trial_date
             FROM   foodie_fi.subscriptions
             WHERE  plan_id = 0),
         annual_cte
         AS (SELECT customer_id,
                    start_date AS annual_date
             FROM   foodie_fi.subscriptions
             WHERE  plan_id = 3)
    SELECT CASE
             WHEN Round(annual_cte.annual_date - start_cte.trial_date, 0) <= 30 THEN
             '0-30 days'
             WHEN Round(annual_cte.annual_date - start_cte.trial_date, 0) <= 60 THEN
             '31-60 days'
             WHEN Round(annual_cte.annual_date - start_cte.trial_date, 0) <= 90 THEN
             '61-90 days'
             WHEN Round(annual_cte.annual_date - start_cte.trial_date, 0) <= 120
           THEN
             '91-120 days'
             WHEN Round(annual_cte.annual_date - start_cte.trial_date, 0) <= 150
           THEN
             '121-150 days'
             WHEN Round(annual_cte.annual_date - start_cte.trial_date, 0) <= 180
           THEN
             '151-180 days'
             WHEN Round(annual_cte.annual_date - start_cte.trial_date, 0) <= 210
           THEN
             '181-210 days'
             WHEN Round(annual_cte.annual_date - start_cte.trial_date, 0) <= 240
           THEN
             '211-240 days'
             WHEN Round(annual_cte.annual_date - start_cte.trial_date, 0) <= 270
           THEN
             '241-270 days'
             WHEN Round(annual_cte.annual_date - start_cte.trial_date, 0) <= 300
           THEN
             '271-300 days'
             WHEN Round(annual_cte.annual_date - start_cte.trial_date, 0) <= 330
           THEN
             '301-330 days'
             WHEN Round(annual_cte.annual_date - start_cte.trial_date, 0) <= 360
           THEN
             '331-360 days'
           END                                                          AS NumOfDays
           ,
           Round(Avg(annual_cte.annual_date - start_cte.trial_date), 0) AS
           AvgDays,
           Count(DISTINCT start_cte.customer_id)                        AS Customers
    FROM   annual_cte
           JOIN start_cte
             ON start_cte.customer_id = annual_cte.customer_id
    GROUP  BY numofdays
    ORDER  BY avgdays;
```
| numofdays    | avgdays | customers |
| ------------ | ------- | --------- |
| 0-30 days    | 10      | 49        |
| 31-60 days   | 42      | 24        |
| 61-90 days   | 71      | 34        |
| 91-120 days  | 101     | 35        |
| 121-150 days | 133     | 42        |
| 151-180 days | 162     | 36        |
| 181-210 days | 191     | 26        |
| 211-240 days | 224     | 4         |
| 241-270 days | 257     | 5         |
| 271-300 days | 285     | 1         |
| 301-330 days | 327     | 1         |
| 331-360 days | 346     | 1         |


11.   How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```sql
    WITH cte
         AS (SELECT customer_id,
                    plan_id,
                    Lead(plan_id, 1)
                      over(
                        ORDER BY customer_id) AS next_plan,
                    start_date
             FROM   foodie_fi.subscriptions
             ORDER  BY customer_id)
    SELECT Count(DISTINCT customer_id)                                         AS
           total_customers,
           Count(customer_id) * 100 / (SELECT Count(DISTINCT customer_id)
                                       FROM   foodie_fi.subscriptions) :: REAL AS
           percentage
    FROM   cte
    WHERE  ( plan_id = 2
             AND next_plan = 1 )
           AND Extract(year FROM start_date) = 2020;
```
| total_customers | percentage |
| --------------- | ---------- |
| 0               | 0          |
