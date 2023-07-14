## Case Study Solutions Part B: Runner and Customer Experience

#### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

In this question, the only trick was to make sure that I was using 1/1/21 as the beginning of the week. Adding 3 to registration date made that possible.

```sql
SELECT WEEK(registration_date + 3) as signup_week,
       COUNT(runner_id)
FROM runners
GROUP BY signup_week;
```
##### Result Table:
|   signup_week | COUNT(runner_id) |
|---------------|------------------|
| 1             | 2                |
| 2             | 1                |
| 3             | 1                | 
#### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

here the two main things I had to look out for were making sure that each order only counted towards the average once, and making sure that the answer was in a format that was easy to understand (hence the unwieldy conversion statement).

```sql
WITH time_diff AS (
  SELECT DISTINCT
    orders.order_id,
    runners.runner_id,
    orders.order_time,
    runners.pickup_time
  FROM
    customer_orders_temp AS orders
      JOIN
    runner_orders_temp AS runners ON orders.order_id = runners.order_id
  WHERE
    runners.cancellation = ''
)
SELECT 
    runner_id,
    ROUND(AVG(TIMESTAMPDIFF(MINUTE,
                order_time,
                pickup_time)),
            2) AS avg_min_to_pickup
FROM
    time_diff
GROUP BY runner_id
```
##### Result Table:
| # runner_id | avg_min_to_pickup |
|-------------|-------------------|
| 1           | 00:14:19          |
| 2           | 00:20:00          |
| 3           | 00:10:28          |
#### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

```sql
WITH CTE AS (
  SELECT 
    orders.order_id,
    COUNT(orders.order_id) AS pizzas_in_order,
    TIMESTAMPDIFF(MINUTE,
      orders.order_time,
      runners.pickup_time) AS prep_time
  FROM
    customer_orders_temp AS orders
      LEFT JOIN
    runner_orders_temp AS runners ON orders.order_id = runners.order_id
WHERE
    cancellation = ''
GROUP BY order_id , prep_time
)
SELECT 
    pizzas_in_order, ROUND(AVG(prep_time), 2)
FROM
    CTE
GROUP BY pizzas_in_order
```
##### Result Table:
|   pizzas_in_order | round(AVG(prep_time), 2) |
|-------------------|--------------------------|
| 1                 | 12.00                    |
| 2                 | 18.00                    |
| 3                 | 29.00                    |
#### 4. What was the average distance travelled for each customer?

Again the important thing for this question was to make sure that you only counted each order once towards the average. To do this I created a CTE that gave me only the distinct orders and used that to get the average distances.

```sql
WITH order_cte AS (
  SELECT DISTINCT
    customers.order_id, customers.customer_id, runners.distance
  FROM
    customer_orders_temp AS customers
  JOIN
    runner_orders_temp AS runners ON customers.order_id = runners.order_id
  WHERE
    cancellation = ''
)
SELECT 
  customer_id, ROUND(AVG(distance), 2)
FROM
  order_cte
GROUP BY customer_id;
```
##### Result Table:
|   customer_id | round(AVG(distance), 2) |
|---------------|-------------------------|
| 101           | 20.00                   |
| 102           | 18.40                   |
| 103           | 23.40                   |
| 104           | 10.00                   |
| 105           | 25.00                   |
#### 5. What was the difference between the longest and shortest delivery times for all orders?

nothing particularly interesting about this question
```sql
SELECT 
    MAX(duration) - MIN(duration) AS delivery_time_diff
FROM
    runner_orders_temp AS runners
```
##### Result Table:
|   time_diff |
|-------------|
| 30          |
#### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?


```sql
SELECT 
    order_id,
    runner_id,
    distance AS distance_km,
    ROUND(duration / 60, 2) AS duration_hr,
    ROUND((distance / (duration / 60)), 2) AS avg_speed_km_per_hour
FROM
    runner_orders_temp
WHERE
    cancellation = ''
```
##### Result Table:
|   order_id | runner_id | distance_km | duration_hr | avg_speed_km_per_hour |
|------------|-----------|-------------|-------------|-----------------------|
| 1          | 1         | 20.00       | 0.53        | 37.50                 |
| 2          | 1         | 20.00       | 0.45        | 44.44                 |
| 3          | 1         | 13.40       | 0.33        | 40.20                 |
| 4          | 2         | 23.40       | 0.67        | 35.10                 |
| 5          | 3         | 10.00       | 0.25        | 40.00                 |
| 7          | 2         | 25.00       | 0.42        | 60.00                 |
| 8          | 2         | 23.40       | 0.25        | 93.60                 |
| 10         | 1         | 10.00       | 0.17        | 60.00                 |
#### 7. What is the successful delivery percentage for each runner?

```sql

```
##### Result Table:
