# Case Study #1: Danny's Diner

## Case Study Questions

1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
10. What is the total items and amount spent for each member before they became a member?
11. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
12. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

###  1. What is the total amount each customer spent at the restaurant?

```sql
SELECT 
    customer_id, SUM(price)
FROM
    sales
        LEFT JOIN
    menu ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
```

#### Result Grid:

| customer_id | total_sales |
| ----------- | ----------- |
| A           | $76         |
| B           | $74         |
| C           | $36         |

###   2. How many days has each customer visited the restaurant?

```sql
SELECT 
    customer_id, COUNT(DISTINCT order_date) AS days_visited
FROM
    sales
GROUP BY customer_id
ORDER BY customer_id
```

#### Result Grid:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 4           |
| B           | 6           |
| C           | 2           |
  
###   3. What was the first item from the menu purchased by each customer?

```sql
WITH CTE AS (
	SELECT customer_id,
    	   order_date,
    	   product_name,  
    	   RANK() OVER(PARTITION BY customer_id ORDER BY order_date ASC) as FULL_ORDER,
		   ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date ASC) as JUST_FIRST_ITE
	FROM sales
	INNER JOIN menu
    ON sales.product_id = menu.product_id
	)
SELECT *
FROM CTE
WHERE rnk = 1;
```
#### Result Grid:
|customer_id|	order_date|	product_name |rnk|rn|
|-----------|-----------|--------------|---|--|
|A	        |2021-01-01 |	sushi	       |1	 |1 |
|A	        |2021-01-01 |	curry	       |1	 |2 |
|B	        |2021-01-01 |	curry	       |1  |1 |
|C	        |2021-01-01 |	ramen	       |1  |1 |
|C	        |2021-01-01 |	ramen	       |1	 |2 |


  
###   4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT 
    menu.product_name, COUNT(sales.product_id) AS num_orders
FROM
    sales
        INNER JOIN
    menu ON sales.product_id = menu.product_id
GROUP BY sales.product_id , menu.product_name
ORDER BY num_orders DESC
LIMIT 1
```

#### Result Grid:
|product_name|num_orders|
|------------|----------|
|ramen	     |8         |

  
###   5. Which item was the most popular for each customer?

```sql
WITH CTE AS (
	SELECT customer_id,
		   product_name,
		   COUNT(sales.product_id) as num_orders,
		   RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(sales.product_id) DESC) as order_rank
	FROM sales
	INNER JOIN menu
	ON sales.product_id = menu.product_id
	GROUP BY sales.customer_id, menu.product_name
    )
SELECT customer_id,
	   product_name,
	   num_orders
FROM CTE
```
#### Result Grid:
|customer_id|	product_name|	num_orders|
|-----------|-------------|-----------|
|A          |ramen        |3          |
|B         	|curry	      |2          |
|B	        |sushi	      |2          |
|B        	|ramen	      |2          |
|C	        |ramen	      |3          |

  
###   6. Which item was purchased first by the customer after they became a member?

(Assuming they became a member before ordering on the day they became members)
```sql
WITH CTE AS (
     SELECT sales.customer_id,
            members.join_date,
            product_name,
            sales.order_date,
            RANK() OVER(PARTITION BY customer_id 
                        ORDER BY sales.order_date) as order_rank
    FROM members
    LEFT JOIN sales ON members.customer_id = sales.customer_id
    LEFT JOIN menu ON sales.product_id = menu.product_id
    WHERE order_date >= join_date
    )
SELECT *
FROM CTE
```

#### Result Grid:
|   customer_id | join_date  | product_name | order_date | order_rank |
|---------------|------------|--------------|------------|------------|
| A             | 2021-01-07 | curry        | 2021-01-07 | 1          |
| B             | 2021-01-09 | sushi        | 2021-01-11 | 1          |

  
###   7. Which item was purchased just before the customer became a member?

(Assuming they became a member before ordering on the day they became members and all items with the same order date were ordered at the same time)
```sql
WITH CTE AS 
	(SELECT sales.customer_id,
		members.join_date,
		product_name,
		sales.order_date,
		RANK() OVER(PARTITION BY customer_id 
            		ORDER BY sales.order_date DESC) as order_rank
	FROM members
	LEFT JOIN sales ON members.customer_id = sales.customer_id
	LEFT JOIN menu ON sales.product_id = menu.product_id
	WHERE order_date < join_date
	)
SELECT *
FROM CTE
WHERE order_rank = 1
```

#### Result Grid:
|   customer_id | join_date  | product_name | order_date | order_rank |
|---------------|------------|--------------|------------|------------|
| A             | 2021-01-07 | sushi        | 2021-01-01 | 1          |
| A             | 2021-01-07 | curry        | 2021-01-01 | 1          |
| B             | 2021-01-09 | sushi        | 2021-01-04 | 1          |
  
###   8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT 
    sales.customer_id,
    COUNT(menu.product_name) AS total_items,
    CONCAT('$', SUM(menu.price)) AS amt_spent
FROM
    members
        LEFT JOIN
    sales ON members.customer_id = sales.customer_id
        LEFT JOIN
    menu ON sales.product_id = menu.product_id
WHERE
    order_date < join_date
GROUP BY sales.customer_id , members.join_date
```

#### Result Grid:
|   customer_id | total_items | amt_spent |
|---------------|-------------|-----------|
| A             | 2           | $25       |
| B             | 3           | $40       |
  
###   9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
SELECT 
    sales.customer_id,
    SUM(CASE
        WHEN menu.product_id = 1 THEN menu.price * 20
        ELSE menu.price * 10
    END) AS total_points
FROM
    sales
        LEFT JOIN
    menu ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
```

#### Result Grid:
|   customer_id | total_points |
|---------------|--------------|
| A             | 860          |
| B             | 940          |
| C             | 360          |
  
###   10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
SELECT 
    sales.customer_id,
    SUM(CASE
        WHEN
            sales.order_date - members.join_date < 7
                OR menu.product_id = 1
        THEN
            menu.price * 20
        ELSE menu.price * 10
    END) AS bonus_pts_applied
FROM
    members
        LEFT JOIN
    sales ON members.customer_id = sales.customer_id
        LEFT JOIN
    menu ON sales.product_id = menu.product_id
WHERE
    sales.order_date <= '2021-01-31'
        AND sales.order_date >= join_date
GROUP BY sales.customer_id
```

#### Result Grid:
| # customer_id | bonus_pts_applied |
|---------------|-------------------|
| A             | 1020              |
| B             | 320               |

#### BONUS QUESTION 1 "JOIN ALL THE THINGS"
 here I am trying to recreate a table given a picture of the table from the website

```sql
SELECT 
    sales.customer_id,
    sales.order_date,
    menu.product_name,
    menu.price,
    IF(order_date >= join_date, 'Y', 'N') AS member
FROM
    sales
        LEFT JOIN
    members ON sales.customer_id = members.customer_id
        JOIN
    menu ON sales.product_id = menu.product_id
ORDER BY sales.customer_id , sales.order_date
```
| # customer_id | order_date | product_name | price | member |
|---------------|------------|--------------|-------|--------|
| A             | 2021-01-01 | sushi        | 10    | N      |
| A             | 2021-01-01 | curry        | 15    | N      |
| A             | 2021-01-07 | curry        | 15    | Y      |
| A             | 2021-01-10 | ramen        | 12    | Y      |
| A             | 2021-01-11 | ramen        | 12    | Y      |
| A             | 2021-01-11 | ramen        | 12    | Y      |
| B             | 2021-01-01 | curry        | 15    | N      |
| B             | 2021-01-02 | curry        | 15    | N      |
| B             | 2021-01-04 | sushi        | 10    | N      |
| B             | 2021-01-11 | sushi        | 10    | Y      |
| B             | 2021-01-16 | ramen        | 12    | Y      |
| B             | 2021-02-01 | ramen        | 12    | Y      |
| C             | 2021-01-01 | ramen        | 12    | N      |
| C             | 2021-01-01 | ramen        | 12    | N      |
| C             | 2021-01-07 | ramen        | 12    | N      |

#### BONUS QUESTION 2 "RANK ALL THE THINGS"
same as bonus question 1, was given a table to recreate
```sql
WITH CTE AS
	(SELECT 
		sales.customer_id,
		sales.order_date,
		menu.product_name,
		menu.price,
		IF(order_date >= join_date, 'Y', 'N') AS member
	FROM
		sales
			LEFT JOIN
		members ON sales.customer_id = members.customer_id
			JOIN
		menu ON sales.product_id = menu.product_id
	ORDER BY sales.customer_id , sales.order_date)
SELECT *, 
       IF(member = 'N', NULL, DENSE_RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date)) as ranking
FROM CTE
```

| # customer_id | order_date | product_name | price | member | ranking |
|---------------|------------|--------------|-------|--------|---------|
| A             | 2021-01-01 | sushi        | 10    | N      | null    |
| A             | 2021-01-01 | curry        | 15    | N      | null    |
| A             | 2021-01-07 | curry        | 15    | Y      | 1       |
| A             | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A             | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A             | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B             | 2021-01-01 | curry        | 15    | N      | null    |
| B             | 2021-01-02 | curry        | 15    | N      | null    |
| B             | 2021-01-04 | sushi        | 10    | N      | null    |
| B             | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B             | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B             | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C             | 2021-01-01 | ramen        | 12    | N      | null    |
| C             | 2021-01-01 | ramen        | 12    | N      | null    |
| C             | 2021-01-07 | ramen        | 12    | N      | null    |
