# 🍕 Case Study #2 Pizza Runner

<img src="https://user-images.githubusercontent.com/81607668/127271856-3c0d5b4a-baab-472c-9e24-3c1e3c3359b2.png" alt="Image" width="500" height="520">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Techniques Used](#techniques-used)
- [Question and Solution](#question-and-solution)
  - Schema Editing
    - [Data Cleaning and Transformation](#data-cleaning-and-transformation)
  - Querying
    - [A. Pizza Metrics](#a-pizza-metrics)
    - [B. Runner and Customer Experience](#b-runner-and-customer-experience)
    - [C. Ingredient Optimisation](#c-ingredient-optimisation)
    - [D. Pricing and Ratings](#d-pricing-and-ratings)

***

## Business Task
Danny is expanding his new Pizza Empire and at the same time, he wants to Uberize it, so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers. 

## Entity Relationship Diagram

![Pizza Runner](https://github.com/katiehuangx/8-Week-SQL-Challenge/assets/81607668/78099a4e-4d0e-421f-a560-b72e4321f530)

***
## Question and Solution

## Data Cleaning and Transformation
In case study 2, we are required to clean the table for analysis

**Table 2: `customer_orders`**

| order_id | customer_id | pizza_id | exclusions | extras | order_time           |
|----------|-------------|----------|------------|--------|----------------------|
| 1        | 101         | 1        |            |        | 2021-01-01 18:05:02  |
| 2        | 101         | 1        |            |        | 2021-01-01 19:00:52  |
| 3        | 102         | 1        |            |        | 2021-01-02 23:51:23  |
| 3        | 102         | 2        | NaN        |        | 2021-01-02 23:51:23  |
| 4        | 103         | 1        | 4          |        | 2021-01-04 13:23:46  |
| 4        | 103         | 1        | 4          |        | 2021-01-04 13:23:46  |
| 4        | 103         | 2        | 4          |        | 2021-01-04 13:23:46  |
| 5        | 104         | 1        | null       | 1      | 2021-01-08 21:00:29  |
| 6        | 101         | 2        | null       | null   | 2021-01-08 21:03:13  |
| 7        | 105         | 2        | null       | 1      | 2021-01-08 21:20:29  |
| 8        | 102         | 1        | null       | null   | 2021-01-09 23:54:33  |
| 9        | 103         | 1        | 4          | 1, 5   | 2021-01-10 11:22:59  |
| 10       | 104         | 1        | null       | null   | 2021-01-11 18:34:49  |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2021-01-11 18:34:49  |

**Table Cleansing Methodology:**

| Column  | Solution |
|----------|----------|
| exclusions         | convert 'null' values to NULL        |
| extras        | convert 'null' values to NULL        |

**New Schema:**
```sql
-- Copying table to new table for Customer Orders
DROP TABLE IF EXISTS customer_orders1;
CREATE TABLE customer_orders1 AS
(
    SELECT 
        order_id, 
        customer_id, 
        pizza_id, 
        CASE WHEN exclusions = 'null' THEN NULL ELSE exclusions END AS exclusions,
        CASE WHEN extras = 'null' THEN NULL ELSE extras END AS extras,
        order_time 
    FROM 
        customer_orders
);

-- Define data types for columns in Customer Orders
ALTER TABLE customer_orders1
ALTER COLUMN order_id TYPE INTEGER,
ALTER COLUMN customer_id TYPE INTEGER,
ALTER COLUMN pizza_id TYPE INTEGER,
ALTER COLUMN exclusions TYPE VARCHAR(255),
ALTER COLUMN extras TYPE VARCHAR(255),
ALTER COLUMN order_time TYPE TIMESTAMP;
```
**New Table 2: `customer_orders1`**

| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ |
| 1        | 101         | 1        |            |        | 2020-01-01T18:05:02.000Z |
| 2        | 101         | 1        |            |        | 2020-01-01T19:00:52.000Z |
| 3        | 102         | 1        |            |        | 2020-01-02T23:51:23.000Z |
| 3        | 102         | 2        |            |        | 2020-01-02T23:51:23.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 2        | 4          |        | 2020-01-04T13:23:46.000Z |
| 5        | 104         | 1        |            | 1      | 2020-01-08T21:00:29.000Z |
| 6        | 101         | 2        |            |        | 2020-01-08T21:03:13.000Z |
| 7        | 105         | 2        |            | 1      | 2020-01-08T21:20:29.000Z |
| 8        | 102         | 1        |            |        | 2020-01-09T23:54:33.000Z |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10T11:22:59.000Z |
| 10       | 104         | 1        |            |        | 2020-01-11T18:34:49.000Z |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11T18:34:49.000Z |


***
**Table 3: `runner_orders`**

| order_id | runner_id | pickup_time          | distance | duration | cancellation             |
|----------|-----------|----------------------|----------|----------|--------------------------|
| 1        | 1         | 2021-01-01 18:15:34  | 20km     | 32 minutes |                          |
| 2        | 1         | 2021-01-01 19:10:54  | 20km     | 27 minutes |                          |
| 3        | 1         | 2021-01-03 00:12:37  | 13.4km   | 20 mins   | NaN                      |
| 4        | 2         | 2021-01-04 13:53:03  | 23.4     | 40       | NaN                      |
| 5        | 3         | 2021-01-08 21:10:57  | 10       | 15       | NaN                      |
| 6        | 3         | null                 | null     | null     | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45  | 25km     | 25mins   | null                     |
| 8        | 2         | 2020-01-10 00:15:02  | 23.4 km  | 15 minute | null                     |
| 9        | 2         | null                 | null     | null     | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20  | 10km     | 10 minutes| null                     |

**Table Cleansing Methodology:**

| Column  | Solution |
|----------|----------|
| pickup_time         | converted 'null' values to NULL <br> converted datatype to timestamp        |
| distance        | converted 'null' values to NULL  <br>  converted datatype to real number   |
| duration         | converted 'null' values to NULL  <br>  converted datatype to integer    |
| cancellation        | converted 'null' values to NULL        |

**New Schema:**
```sql
-- Copying table to new table for Runner Orders
DROP TABLE IF EXISTS runner_orders1;
CREATE TABLE runner_orders1 AS 
(
    SELECT 
        order_id, 
        runner_id, 
        pickup_time,
        CASE 
            WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
            ELSE distance 
        END AS distance,
        CASE 
            WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
            WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
            WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
            ELSE duration
        END AS duration,
        CASE WHEN cancellation = 'null' THEN NULL ELSE cancellation END AS cancellation 
    FROM 
        runner_orders
);

-- Cleaning data for Runner Orders
UPDATE runner_orders1
SET 
    pickup_time = CASE WHEN pickup_time = 'null' THEN NULL ELSE pickup_time END,
    distance = CASE WHEN distance = 'null' THEN NULL ELSE distance END,
    duration = CASE WHEN duration = 'null' THEN NULL ELSE duration END,
    cancellation = CASE WHEN cancellation = 'null' THEN NULL ELSE cancellation END;

-- Change data types for Runner Orders
ALTER TABLE runner_orders1
ALTER COLUMN order_id TYPE INTEGER,
ALTER COLUMN runner_id TYPE INTEGER,
ALTER COLUMN pickup_time TYPE TIMESTAMP USING TO_TIMESTAMP(pickup_time, 'YYYY-MM-DD HH24:MI:SS'), 
ALTER COLUMN distance TYPE REAL USING distance::REAL,
ALTER COLUMN duration TYPE INTEGER USING duration::INTEGER,
ALTER COLUMN cancellation TYPE VARCHAR(255);
```
**New Table 3: `runner_orders1`**

| order_id | runner_id | pickup_time              | distance | duration | cancellation            |
| -------- | --------- | ------------------------ | -------- | -------- | ----------------------- |
| 1        | 1         | 2020-01-01T18:15:34.000Z | 20       | 32       |                         |
| 2        | 1         | 2020-01-01T19:10:54.000Z | 20       | 27       |                         |
| 3        | 1         | 2020-01-03T00:12:37.000Z | 13.4     | 20       |                         |
| 4        | 2         | 2020-01-04T13:53:03.000Z | 23.4     | 40       |                         |
| 5        | 3         | 2020-01-08T21:10:57.000Z | 10       | 15       |                         |
| 6        | 3         |                          |          |          | Restaurant Cancellation |
| 7        | 2         | 2020-01-08T21:30:45.000Z | 25       | 25       |                         |
| 8        | 2         | 2020-01-10T00:15:02.000Z | 23.4     | 15       |                         |
| 9        | 2         |                          |          |          | Customer Cancellation   |
| 10       | 1         | 2020-01-11T18:50:20.000Z | 10       | 10       |                         |

***
**New Table: `ratings`**
```sql
-- Normalize Pizza Recipe table
DROP TABLE IF EXISTS ratings;
CREATE TABLE ratings (
    order_id INTEGER,
    rating INTEGER
);

-- Insert sample data into Ratings table
INSERT INTO ratings (order_id, rating)
VALUES
    (1, 3),
    (2, 5),
    (3, 3),
    (4, 1),
    (5, 5),
    (7, 3),
    (8, 4),
    (10, 3);
```


| order_id | rating |
| -------- | ------ |
| 1        | 3      |
| 2        | 5      |
| 3        | 3      |
| 4        | 1      |
| 5        | 5      |
| 7        | 3      |
| 8        | 4      |
| 10       | 3      |

***

## Techniques Used

| Technique                                 | Count of Usage | Questions                                  |
|-------------------------------------------|----------------|--------------------------------------------|
| COUNT() Function                          | 10             | A.1, A.2, A.3, A.5, A.6, A.7, A.8, A.9, A.10, B.1  |
| DISTINCT Keyword                          | 5              | A.2, A.4, B.5, C.1, D.5                     |
| SUM() Function                            | 12             | A.3, A.6, A.9, A.10, B.2, B.4, B.5, B.6, B.7, C.2, C.3, D.1  |
| JOIN Operation                            | 19             | A.4, A (all), B (all), C.1, C.2, C.3, C.4, C.5, C.6, C.8, C.9, D.1, D.5  |
| GROUP BY Clause                           | 13             | A.3, A.5, A.6, A.7, A.9, B.4, B.6, B.7, C.1, C.2, C.3, C.4, D.9  |
| ORDER BY Clause                           | 5              | A.3, A.6, A.7, A.9, C.9                     |
| LIMIT Clause                              | 2              | A.6, B.5                                   |
| CASE Statement                            | 3              | A.7, A.8, A.9                             |
| WITH Clause (Common Table Expression)     | 4              | A.8, B.3, C.1, D.5                         |
| Window Functions (RANK(), Dense_Rank())   | 4              | A.5, A.6, B.5, B.6                         |
| Date Functions (TO_CHAR(), Extract())     | 2              | A.9, A.10                                  |
| PARTITION BY Clause                       | 1              | A.5                                       |


***

## A. Pizza Metrics

**1. How many pizzas were ordered?**
```sql
    SELECT Count(order_id) AS pizzas_ordered
    FROM   pizza_runner.customer_orders;
```

| pizzas_ordered |
| -------------- |
| 14             |

**2. How many unique customer orders were made?**
```sql
    SELECT Count(distinct order_id) AS unique_pizzas_ordered
    FROM   pizza_runner.customer_orders;
```
| unique_pizzas_ordered |
| --------------------- |
| 10                    |

**3. How many successful orders were delivered by each runner?**
```sql
    SELECT runner_id,
           Count(order_id) AS OrdersDelivered
    FROM   pizza_runner.runner_orders1
    WHERE  distance IS NOT NULL
    GROUP  BY runner_id
    ORDER  BY ordersdelivered DESC;
```
| runner_id | ordersdelivered |
| --------- | --------------- |
| 1         | 4               |
| 2         | 3               |
| 3         | 1               |


**4. How many of each type of pizza was delivered?**
```sql
    SELECT pizza_names.pizza_name,
           Count(customer_orders1.pizza_id) AS TotalDelivered
    FROM   pizza_runner.customer_orders1
           INNER JOIN pizza_runner.runner_orders1
                   ON customer_orders1.order_id = runner_orders1.order_id
           INNER JOIN pizza_runner.pizza_names
                   ON pizza_names.pizza_id = customer_orders1.pizza_id
    WHERE  runner_orders1.distance IS NOT NULL
    GROUP  BY customer_orders1.pizza_id,
              pizza_names.pizza_name;
```
| pizza_name | totaldelivered |
| ---------- | -------------- |
| Meatlovers | 9              |
| Vegetarian | 3              |

**5. How many Vegetarian and Meatlovers were ordered by each customer?**
```sql
    SELECT customer_orders1.customer_id,
           pizza_names.pizza_name,
           Count(customer_orders1.pizza_id) AS TotalPizzaOrdered
    FROM   pizza_runner.customer_orders1
           INNER JOIN pizza_runner.pizza_names
                   ON customer_orders1.pizza_id = pizza_names.pizza_id
    GROUP  BY customer_orders1.customer_id,
              pizza_names.pizza_name
    ORDER  BY customer_orders1.customer_id;
```
| customer_id | pizza_name | totalpizzaordered |
| ----------- | ---------- | ----------------- |
| 101         | Meatlovers | 2                 |
| 101         | Vegetarian | 1                 |
| 102         | Meatlovers | 2                 |
| 102         | Vegetarian | 1                 |
| 103         | Meatlovers | 3                 |
| 103         | Vegetarian | 1                 |
| 104         | Meatlovers | 3                 |
| 105         | Vegetarian | 1                 |

**6. What was the maximum number of pizzas delivered in a single order?**
```sql
    SELECT customer_orders1.order_id,
           Count(customer_orders1.pizza_id) AS TotalPizzaOrdered
    FROM   pizza_runner.customer_orders1
           INNER JOIN pizza_runner.pizza_names
                   ON customer_orders1.pizza_id = pizza_names.pizza_id
    GROUP  BY customer_orders1.order_id
    ORDER  BY totalpizzaordered DESC
    LIMIT  1;
```

| order_id | totalpizzaordered |
| -------- | ----------------- |
| 4        | 3                 |

**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**
```sql
    SELECT customer_id,
           Sum(CASE
                 WHEN ( ( Length(exclusions) > 0 )
                         OR ( Length(extras) > 0 ) ) THEN 1
                 ELSE 0
               END) AS changes,
           Sum(CASE
                 WHEN ( ( Length(exclusions) > 0 )
                         OR ( Length(extras) > 0 ) ) THEN 0
                 ELSE 1
               END) AS nochanges
    FROM   pizza_runner.customer_orders1
           JOIN pizza_runner.runner_orders1
             ON ( customer_orders1.order_id = runner_orders1.order_id )
    WHERE  Length(runner_orders1.distance) > 0
    GROUP  BY customer_id
    ORDER  BY customer_id;
```
| customer_id | changes | nochanges |
| ----------- | ------- | --------- |
| 101         | 0       | 2         |
| 102         | 0       | 3         |
| 103         | 3       | 0         |
| 104         | 2       | 1         |
| 105         | 1       | 0         |

**8. How many pizzas were delivered that had both exclusions and extras?**
```sql
    WITH change
         AS (SELECT customer_id,
                    Sum(CASE
                          WHEN ( ( Length(exclusions) > 0 )
                                  OR ( Length(extras) > 0 ) ) THEN 1
                          ELSE 0
                        END) AS changes,
                    Sum(CASE
                          WHEN ( ( Length(exclusions) > 0 )
                                  OR ( Length(extras) > 0 ) ) THEN 0
                          ELSE 1
                        END) AS nochanges
             FROM   pizza_runner.customer_orders1
                    JOIN pizza_runner.runner_orders1
                      ON ( customer_orders1.order_id = runner_orders1.order_id )
             WHERE  Length(runner_orders1.distance) > 0
             GROUP  BY customer_id
             ORDER  BY customer_id)
    SELECT *
    FROM   (SELECT customer_id,
                   CASE
                     WHEN ( changes > 0
                            AND nochanges > 0 ) THEN 1
                     ELSE 0
                   END AS both
            FROM   change) AS filtered_results
    WHERE  filtered_results.both > 0;
```
| customer_id | both |
| ----------- | ---- |
| 104         | 1    |


**9. What was the total volume of pizzas ordered for each hour of the day?**
```sql
    SELECT Extract(hour FROM order_time) AS hour_of_day,
           Count(customer_id)            AS orders
    FROM   pizza_runner.customer_orders1
           JOIN pizza_runner.runner_orders1
             ON ( customer_orders1.order_id = runner_orders1.order_id )
    GROUP  BY hour_of_day
    ORDER  BY hour_of_day;
```
| hour_of_day | orders |
| ----------- | ------ |
| 11          | 1      |
| 13          | 3      |
| 18          | 3      |
| 19          | 1      |
| 21          | 3      |
| 23          | 3      |


**10. What was the volume of orders for each day of the week?**
```sql
    SELECT (SELECT To_char(order_time, 'Day')) AS day,
           Count(customer_id)                  AS orders
    FROM   pizza_runner.customer_orders1
           JOIN pizza_runner.runner_orders1
             ON ( customer_orders1.order_id = runner_orders1.order_id )
    GROUP  BY day
    ORDER  BY orders;
```
| day       | orders |
| --------- | ------ |
| Friday    | 1      |
| Thursday  | 3      |
| Saturday  | 5      |
| Wednesday | 5      |

***
## B. Runner and Customer Experience

1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
  ```sql
    SELECT Extract('week' FROM registration_date) AS week,
           Count(runner_id)
    FROM   pizza_runner.runners
    GROUP  BY week;
```
| week | count |
| ---- | ----- |
| 53   | 2     |
| 1    | 1     |
| 2    | 1     |


2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
  ```sql
    SELECT runner_id,
           Avg(pickup_time :: timestamp - order_time :: timestamp) as average_pickup_time
    FROM   pizza_runner.customer_orders1
           join pizza_runner.runner_orders1
             ON ( runner_orders1.order_id = customer_orders1.order_id )
    GROUP  BY runner_id;
```
| runner_id | average_pickup_time                                                |
| --------- | -------------------------------------------------- |
| 3         | {"minutes":10,"seconds":28}                        |
| 2         |  	{"minutes":23,"seconds":43,"milliseconds":200}   |
| 1         | {"minutes":15,"seconds":40,"milliseconds":666.667} |

3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
  ```sql
    WITH cte
         AS (SELECT ( customer_orders1.order_id ),
                    Count(pizza_id)                                             AS
                       pizza_count,
                    Avg(( pickup_time :: timestamp - order_time :: timestamp )) AS
                    avg_time
             FROM   pizza_runner.customer_orders1
                    join pizza_runner.runner_orders1
                      ON ( runner_orders1.order_id = customer_orders1.order_id )
             WHERE  Length(runner_orders1.distance) > 0
             GROUP  BY customer_orders1.order_id)
    SELECT pizza_count,
           Avg(avg_time) as prep_time
    FROM   cte
    GROUP  BY pizza_count;
```
| pizza_count | prep_time                                      |
| ----------- | ---------------------------------------------- |
| 3           | {"minutes":29,"seconds":17}                    |
| 2           | {"minutes":18,"seconds":22,"milliseconds":500} |
| 1           | {"minutes":12,"seconds":21,"milliseconds":400} |

4. What was the average distance travelled for each customer?
  ```sql
    SELECT  customer_orders1.customer_id ,
           Round(Cast(Avg(distance)AS NUMERIC), 2)
    FROM   pizza_runner.customer_orders1
           JOIN pizza_runner.runner_orders1
             ON ( runner_orders1.order_id = customer_orders1.order_id )
    WHERE  ( runner_orders1.distance ) > 0
    GROUP  BY customer_orders1.customer_id
    ORDER  BY customer_id;

```
| customer_id | distance |
| ----------- | -------- |
| 101         | 20.00    |
| 102         | 16.73    |
| 103         | 23.40    |
| 104         | 10.00    |
| 105         | 25.00    |

5. What was the difference between the longest and shortest delivery times for all orders?
  ```sql
    WITH cte
         AS (SELECT customer_orders1.order_id,
                    ( duration )
             FROM   pizza_runner.customer_orders1
                    JOIN pizza_runner.runner_orders1
                      ON ( runner_orders1.order_id = customer_orders1.order_id )
             WHERE  ( runner_orders1.distance ) > 0)
    SELECT Max(duration) - Min(duration) AS timediff
    FROM   cte;
```
| timediff |
| -------- |
| 30       |


6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
  ```sql
    SELECT runner_orders1.runner_id,
           customer_orders1.order_id,
           Round(Avg(distance / duration):: NUMERIC*60, 2) AS km_per_hr
    FROM   pizza_runner.customer_orders1
           join pizza_runner.runner_orders1
             ON ( runner_orders1.order_id = customer_orders1.order_id )
    WHERE  ( runner_orders1.distance ) > 0
    GROUP  BY runner_orders1.runner_id,
              customer_orders1.order_id
    ORDER  BY runner_orders1.runner_id;
```
| runner_id | order_id | km_per_hr |
| --------- | -------- | --------- |
| 1         | 1        | 37.50     |
| 1         | 2        | 44.44     |
| 1         | 3        | 40.20     |
| 1         | 10       | 60.00     |
| 2         | 4        | 35.10     |
| 2         | 7        | 60.00     |
| 2         | 8        | 93.60     |
| 3         | 5        | 40.00     |

7. What is the successful delivery percentage for each runner?
  ```sql
    SELECT runner_orders1.runner_id,
           Avg(CASE
                 WHEN distance > 0 THEN 1
                 ELSE 0
               END) :: REAL AS delivery_success
    FROM   pizza_runner.runner_orders1
    GROUP  BY runner_orders1.runner_id
    ORDER  BY runner_orders1.runner_id;
```

| runner_id | delivery_success |
| --------- | ---------------- |
| 1         | 1                |
| 2         | 0.75             |
| 3         | 0.5              |

***
## C. Ingredient Optimisation
    1.What are the standard ingredients for each pizza?
    ```sql
    WITH cte
         AS (SELECT pizza_id,
                    Unnest(String_to_array(toppings, ',')) :: INTEGER AS topping_num
             FROM   pizza_runner.pizza_recipes)
    SELECT pizza_id,
           String_agg(topping_name, ', ') AS topping_names
    FROM   cte
           join pizza_runner.pizza_toppings
             ON ( cte.topping_num = pizza_toppings.topping_id )
    GROUP  BY pizza_id
    ORDER  BY pizza_id;
    ```
| pizza_id | topping_names                                                         |
| -------- | --------------------------------------------------------------------- |
| 1        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 2        | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce            |

    
    2.What was the most commonly added extra?
        ```sql
    WITH cte
         AS (SELECT customer_orders1.order_id,
                    Unnest(String_to_array(extras, ',')) :: INTEGER AS extras
             FROM   pizza_runner.customer_orders1)
    SELECT topping_name,
           Count(extras) AS count_extra
    FROM   cte
           join pizza_runner.pizza_toppings
             ON ( cte.extras = pizza_toppings.topping_id )
    GROUP  BY topping_name
    ORDER  BY count_extra DESC;
    ```
| topping_name | count_extra |
| ------------ | ----------- |
| Bacon        | 4           |
| Chicken      | 1           |
| Cheese       | 1           |
    
    3.What was the most common exclusion?
        ```sql
    WITH cte
         AS (SELECT customer_orders1.order_id,
                    Unnest(String_to_array(exclusions, ',')) :: INTEGER AS exclusions
             FROM   pizza_runner.customer_orders1)
    SELECT topping_name,
           Count(exclusions) AS count_exclusions
    FROM   cte
           join pizza_runner.pizza_toppings
             ON ( cte.exclusions = pizza_toppings.topping_id )
    GROUP  BY topping_name
    ORDER  BY count_exclusions DESC;
    ```
| topping_name | count_exclusions |
| ------------ | ---------------- |
| Cheese       | 4                |
| Mushrooms    | 1                |
| BBQ Sauce    | 1                |
    
    4.Generate an order item for each record in the customers_orders table in the format of one of the following:
        *Meat Lovers
        *Meat Lovers - Exclude Beef
        *Meat Lovers - Extra Bacon
        *Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
            ```sql
    WITH record AS
    (
               SELECT     co.order_id,
                          pn.pizza_name AS basepizza,
                          COALESCE(' - Exclude '
                                     || String_agg(DISTINCT tex.topping_name, ', Exclude '), '') AS exclusions,
                          COALESCE(' - Extra '
                                     || String_agg(DISTINCT textra.topping_name, ', Extra '), '') AS extras,
                          pn.pizza_name
                                     || COALESCE(' - Exclude '
                                     || String_agg(DISTINCT tex.topping_name, ', Exclude '), '')
                                     || COALESCE(' - Extra '
                                     || String_agg(DISTINCT textra.topping_name, ', Extra '), '') AS concat
               FROM       pizza_runner.customer_orders co
               LEFT JOIN  pizza_runner.pizza_toppings tex
               ON         tex.topping_id = ANY(String_to_array(NULLIF(co.exclusions, 'null'), ',')::int[])
               LEFT JOIN  pizza_runner.pizza_toppings textra
               ON         textra.topping_id = ANY(String_to_array(NULLIF(co.extras, 'null'), ',')::int[])
               INNER JOIN pizza_runner.pizza_names pn
               ON         co.pizza_id = pn.pizza_id
               GROUP BY   co.order_id,
                          pn.pizza_name)
    SELECT concat AS records
    FROM   record;
    ```
| records                                                                       |
| ----------------------------------------------------------------------------- |
| Meatlovers                                                                    |
| Meatlovers                                                                    |
| Meatlovers                                                                    |
| Vegetarian                                                                    |
| Meatlovers - Exclude Cheese                                                   |
| Vegetarian - Exclude Cheese                                                   |
| Meatlovers - Extra Bacon                                                      |
| Vegetarian                                                                    |
| Vegetarian - Extra Bacon                                                      |
| Meatlovers                                                                    |
| Meatlovers - Exclude Cheese - Extra Bacon, Extra Chicken                      |
| Meatlovers - Exclude BBQ Sauce, Exclude Mushrooms - Extra Bacon, Extra Cheese |
    
    5.Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
        *For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

    
    6.What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

***
## D. Pricing and Ratings

1.    If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
  
```sql
    SELECT Sum(CASE
                 WHEN pizza_name = 'Meatlovers' THEN 12
                 ELSE 10
               END) AS revenue
    FROM   pizza_runner.customer_orders1
           JOIN pizza_runner.pizza_names
             ON ( customer_orders1.pizza_id = pizza_names.pizza_id )
           JOIN pizza_runner.runner_orders1
             ON ( customer_orders1.order_id = runner_orders1.order_id )
    WHERE  distance > 0;

```

| revenue |
| ----- |
| 138   |

2.    What if there was an additional $1 charge for any pizza extras?
        *Add cheese is $1 extra
```sql
    WITH cte
         AS (SELECT ( Unnest(String_to_array(extras, ',')) :: INTEGER ) AS
                    revenue_per_extra
             FROM   pizza_runner.customer_orders1
                    join pizza_runner.pizza_names
                      ON ( customer_orders1.pizza_id = pizza_names.pizza_id )
                    join pizza_runner.runner_orders1
                      ON ( customer_orders1.order_id = runner_orders1.order_id )
             WHERE  distance > 0)
    SELECT 138 + Count(revenue_per_extra) AS revenue
    FROM   cte;
```
| revenue |
| ------- |
| 142     |


3.    The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

```sql
    insert into ratings
    (order_id, rating)
    values
    (1,3),
    (2,5),
    (3,3),
    (4,1),
    (5,5),
    (7,3),
    (8,4),
    (10,3);

    select * from
    pizza_runner.ratings;
```
| order_id | rating |
| -------- | ------ |
| 1        | 3      |
| 2        | 5      |
| 3        | 3      |
| 4        | 1      |
| 5        | 5      |
| 7        | 3      |
| 8        | 4      |
| 10       | 3      |

4.    Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
        customer_id
        order_id
        runner_id
        rating
        order_time
        pickup_time
        Time between order and pickup
        Delivery duration
        Average speed
        Total number of pizzas



5.    If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

```sql
    SELECT 138 - ( Sum(distance) ) * 0.3 AS Finalamount
    FROM   pizza_runner.runner_orders1;

```
| finalamount       |
| ----------------- |
| 94.44|

