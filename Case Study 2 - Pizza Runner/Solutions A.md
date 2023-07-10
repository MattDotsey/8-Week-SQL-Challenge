## Case Study Questions Part A: Pizza Metrics
#### 1. How many pizzas were ordered?

```sql
SELECT 
    COUNT(order_id) AS pizzas_ordered
FROM
    customer_orders_temp;
```
#### 2. How many unique customer orders were made?

```sql
SELECT 
    COUNT(DISTINCT order_id) AS pizzas_ordered
FROM
    customer_orders_temp;
```

#### 3. How many successful orders were delivered by each runner?

```sql
SELECT 
    runner_id, 
    COUNT(order_id) AS successful_orders
FROM
    runner_orders_temp
WHERE
    cancellation = ''
GROUP BY runner_id;
```

#### 4. How many of each type of pizza was delivered?

```sql
SELECT 
    pizza_name, COUNT(orders.pizza_id) AS number_ordered
FROM
    customer_orders_temp AS orders
        LEFT JOIN
    runner_orders_temp AS runners ON orders.order_id = runners.order_id
        LEFT JOIN
    pizza_names AS pizzas ON orders.pizza_id = pizzas.pizza_id
WHERE
    runners.cancellation = ''
GROUP BY orders.pizza_id , pizza_name;
```

#### 5. How many Vegetarian and Meatlovers were ordered by each customer?

```sql
SELECT customer_id, 
	   pizza_name,
	   COUNT(orders.pizza_id) as number_ordered
FROM customer_orders_temp as orders
LEFT JOIN pizza_names as pizzas
	   ON orders.pizza_id = pizzas.pizza_id
GROUP BY orders.customer_id, orders.pizza_id, pizza_name
ORDER BY customer_id;
```
 
#### 6. What was the maximum number of pizzas delivered in a single order?

```sql
SELECT 
    orders.order_id, COUNT(orders.order_id) AS pizzas_ordered
FROM
    customer_orders_temp AS orders
        LEFT JOIN
    runner_orders_temp AS runners ON orders.order_id = runners.order_id
GROUP BY orders.order_id
ORDER BY pizzas_ordered DESC
LIMIT 1;
```

#### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
SELECT 
    customer_id,
    COUNT(CASE
        WHEN exclusions <> '' OR extras <> '' THEN 1
    END) AS changes,
    COUNT(CASE
        WHEN exclusions = '' AND extras = '' THEN 1
    END) AS no_changes
FROM
    customer_orders_temp AS orders
        LEFT JOIN
    runner_orders_temp AS runners ON orders.order_id = runners.order_id
WHERE
    CANCELLATION = ''
GROUP BY customer_id;
```
 
#### 8. How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT 
    COUNT(CASE
        WHEN exclusions <> '' AND extras <> '' THEN 1
    END) AS exclusion_extra
FROM
    customer_orders_temp AS orders
        LEFT JOIN
    runner_orders_temp AS runners ON orders.order_id = runners.order_id
WHERE
    CANCELLATION = ''
```
 
#### 9. What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT 
    EXTRACT(HOUR FROM order_time) AS hour_of_day,
    COUNT(pizza_id) AS pizza_count
FROM
    customer_orders_temp
GROUP BY hour_of_day
ORDER BY hour_of_day;
```
 
#### 10. What was the volume of orders for each day of the week?

```sql
SELECT 
    DAYOFWEEK(order_time) AS day_of_week,
    COUNT(pizza_id) AS pizza_count
FROM
    customer_orders_temp
GROUP BY day_of_week
ORDER BY day_of_week;
```
