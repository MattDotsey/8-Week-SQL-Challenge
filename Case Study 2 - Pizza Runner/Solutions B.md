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
  SELECT DISTINCT orders.order_id,
    runners.runner_id,
    orders.order_time,
  runners.pickup_time
  FROM customer_orders_temp AS orders
  JOIN runner_orders_temp AS runners
  ON orders.order_id = runners.order_id
  WHERE runners.cancellation = ''
)
SELECT runner_id,
  TIME_FORMAT(SEC_TO_TIME(AVG(TIME_TO_SEC(TIMEDIFF(pickup_time, order_time)))), '%H:%i:%s') AS avg_min_to_pickup
FROM time_diff
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

```
##### Result Table:

#### 4. What was the average distance travelled for each customer?

```sql

```
##### Result Table:

#### 5. What was the difference between the longest and shortest delivery times for all orders?

```sql

```
##### Result Table:

#### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

```sql

```
##### Result Table:

#### 7. What is the successful delivery percentage for each runner?

```sql

```
##### Result Table:
