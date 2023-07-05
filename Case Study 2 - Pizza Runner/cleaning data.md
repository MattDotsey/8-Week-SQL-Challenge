####Data Cleaning

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
