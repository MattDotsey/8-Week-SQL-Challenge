## Data Cleaning

The case study mentions that there were some issues with the tables, including various different kinds of null values (empty fields, 'null' in string format, and NULL values) 

The following queries were attempts to clean up those null values.

### customer_orders

```sql
DROP TABLE IF EXISTS customer_orders_temp;
CREATE TABLE customer_orders_temp AS SELECT order_id,
    customer_id,
    pizza_id,
    CASE
        WHEN
            exclusions = 'null'
                OR exclusions IS NULL
        THEN
            ''
        ELSE exclusions
    END AS exclusions,
    CASE
        WHEN extras = 'null' OR exclusions IS NULL THEN ''
        ELSE extras
    END AS extras FROM
    customer_orders;
```

### runner_orders

```sql
DROP TABLE IF EXISTS runner_orders_temp;
CREATE TABLE runner_orders_temp AS SELECT order_id,
    runner_id,
    CASE
        WHEN pickup_time = 'null' THEN NULL
        ELSE pickup_time
    END AS pickup_time,
    CASE
        WHEN distance = 'null' THEN NULL
        WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
        ELSE distance
    END AS distance,
    CASE
        WHEN duration = 'null' THEN NULL
        WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
        WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
        WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
        ELSE duration
    END AS duration,
    CASE
        WHEN
            cancellation IS NULL
                OR cancellation = 'null'
        THEN
            ''
        ELSE cancellation
    END AS cancellation FROM
    runner_orders;
```
