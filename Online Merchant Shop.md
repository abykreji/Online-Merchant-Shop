> DB: Online Merchant Shop

### Question:1

#### How many users have registered each month for the past year?

### Solution:

```python
SELECT
    DATE_TRUNC('month', "created_at") AS registration_month,
    COUNT(*) AS number_of_users
FROM
    online_merchant_shop.users
WHERE
    "created_at" >= NOW() - INTERVAL '1 year'
GROUP BY
    DATE_TRUNC('month', "created_at")
ORDER BY
    registration_month
LIMIT 5;
```
##### Result:



### Question:2

####  What is the average number of orders placed by users within 30 days of registration?

#### Solution:


```python
Select
    u.user_id,
	o.order_date,
	ROUND(AVG(o.order_id),2) AS average_orders_within_30days
FROM
	online_merchant_shop.users AS u
LEFT JOIN online_merchant_shop.orders AS o
	ON u.user_id = o.order_id
GROUP BY 
    u.user_id,o.order_date
ORDER BY 
    u.user_id
LIMIT 5;

```
##### Result:



### Question:3

#### Which state or country has the highest number of registered users?

### Solution:

```python
SELECT 
	a.state,
 	a.country,
	Count(u.user_id) AS highest_number_of_users
FROM
	online_merchant_shop.addresses as a 
LEFT JOIN
	online_merchant_shop.users as u
	ON a.address_id = u.user_id
GROUP BY
	a.country,a.state
LIMIT 5;
```
##### Result:



### Question:4
#### Which are the top 5 best-selling products in the last quarter?

### Solution:

```python
SELECT
    p.product_id,
    p.product_name,
    SUM(oi.quanty) AS total_quantity_sold
FROM
    online_merchant_shop.orderdetails oi
JOIN
    online_merchant_shop.products p
ON
    oi.product_id = p.product_id
JOIN
    online_merchant_shop.orders o
ON
    oi.order_id = o.order_id
WHERE
    o.order_date >= NOW() - INTERVAL '3 months'
GROUP BY
    p.product_id, p.product_name
ORDER BY
    total_quantity_sold DESC
LIMIT 5;

```
##### Result:



### Question:5


#### What is the average stock quantity of products per category?


### Solution:
```python
SELECT
    c.category_id,
    c.category_name,
    ROUND(AVG(p.stock_quantity),2) AS average_stock
FROM
    online_merchant_shop.products AS p
JOIN
    online_merchant_shop.categories AS c
    ON p.product_id = c.category_id
GROUP BY
    c.category_id, c.category_name
ORDER BY
    c.category_id
LIMIT 5;

```

##### Result:



### Question:6

####  Which category has the highest number of products listed?

#### Solution 	

```python
SELECT
    c.category_name,
    COUNT(p.product_id) AS product_count
FROM
    online_merchant_shop.categories AS c
JOIN
    online_merchant_shop.productcategories AS p
ON
    c.category_id = p.product_category_id
GROUP BY
    c.category_name
ORDER BY
    product_count DESC
LIMIT 1;

```

##### Result:



### Question:7
7. What is the total revenue generated each month for the past year?

#### Solution:

```python
SELECT
    DATE_TRUNC('month', order_date) AS month,
    SUM(total_amount) AS total_revenue
FROM
    online_merchant_shop.orders
WHERE
    order_date >= NOW() - INTERVAL '1 year'
GROUP BY
    DATE_TRUNC('month', order_date)
ORDER BY
    month;

```
##### Result:



### Question:8

#### Which month had the highest number of orders placed?

### Solution:

```python
SELECT
    TO_CHAR(order_date, 'YYYY-MM') AS month,
    COUNT(*) AS order_count
FROM
    online_merchant_shop.orders
GROUP BY
    TO_CHAR(order_date, 'YYYY-MM')
ORDER BY
    order_count DESC
LIMIT 5;
```

##### Result:



### Question:9

#### What is the average order value for the store?

### Solution:

```python
SELECT
	ROUND(AVG(total_amount), 2) AS average_order_value
FROM
    online_merchant_shop.orders;
```

##### Result:



### Question:10

####  What is the percentage of orders that have been delivered late?

### Solution:

```python
SELECT
	(COUNT(CASE WHEN status = 'pending' THEN 1 ELSE NULL END) * 100 / COUNT(*)) AS pending_order_percentage
FROM
    online_merchant_shop.orders;

```

##### Result:




### Question:11

#### What is the average number of items in users' carts?

#### Solution:

```python
SELECT ROUND(AVG(cart_item_id),2) AS average_items_in_cart
FROM online_merchant_shop.cartitems;
```
##### Result:


### Question:12

#### How many users have items in their cart but have not placed an order?

#### Solution 

```python
SELECT
	COUNT(DISTINCT c.user_id) AS users_with_items_in_cart
FROM
	online_merchant_shop.cart c
LEFT JOIN
	online_merchant_shop.orders o
	ON c.user_id = o.user_id
WHERE o.order_id IS NULl;
```

##### Result:



### Question: 13

#### What is the total potential revenue from all current cart items?

#### Solution:

```python
SELECT
	SUM(price) AS potential_revenue
FROM
	online_merchant_shop.cartitems c
LEFT JOIN
	online_merchant_shop.orderdetails o
	ON c.cart_item_id = o.order_id;
```

##### Result:



### Question:14

#### What is the most popular payment type among users?

#### Solution:

```python
SELECT 
	payment_type,
	COUNT(*) as payment_count  
FROM 
	online_merchant_shop.payment
GROUP BY
	payment_type	
ORDER BY
	payment_type
LIMIT 1;	
```
##### Results:





### Question:15

#### What is the total amount of pending payments?

#### Solution:

SELECT 
	SUM(amount) AS total_pending_amount
FROM 
	online_merchant_shop.payment
WHERE
	payment_status = 'pending' IS NOT NULL;

##### Result


#### Question:16

How many orders have not been paid for longer than 30 days?

#### Solution:

```python
SELECT COUNT(*) AS unpaid_orders_count
FROM online_merchant_shop.payment p
WHERE NOT EXISTS (
    SELECT 1
    FROM online_merchant_shop.orders o
    WHERE o.order_id = p.order_id
    AND (current_date - o.order_date) <= 30);

```

##### Result:




### Question:17

#### Which city has the highest number of deliveries?

#### Solution:

```python
SELECT 
	city,
	COUNT(*) AS delivery_count
FROM 
	online_merchant_shop.addresses
GROUP BY 
	city
LIMIT 5;
```

##### Result:



### Question:18

What is the average number of shipping addresses per user?

#### Solution:

```python
SELECT 
	user_id, 
	ROUND(AVG(num_distinct_addresses),0) AS average_distinct_addresses
FROM (
  	SELECT
		user_id, 
		COUNT(DISTINCT address_id) AS num_distinct_addresses
	FROM 
		online_merchant_shop.addresses
  	GROUP BY 
		user_id) AS subquery
GROUP BY 
	user_id
LIMIT 5;

```

##### Result:




### Question:19

What is the conversion rate (from adding items to cart to placing an order)?

### Solution:
```python
WITH cart_counts AS (
    SELECT
        product_id,
        COUNT(DISTINCT product_id) AS cart_count
    FROM
        online_merchant_shop.cartitems
    GROUP BY product_id
),
order_counts AS (
    SELECT
        product_id,
        COUNT(DISTINCT product_id) AS order_count
    FROM
        online_merchant_shop.orderdetails
    GROUP BY product_id
)
SELECT
	a.product_id, 
	a.cart_count/b.order_count AS conversion_rate
FROM 
	cart_counts a
LEFT JOIN 
	order_counts b 
ON 
	a.product_id = b.product_id
LIMIT 5;

```

##### Result:



### Question:20


 What is the ratio of stock quantity to the number of orders for each product?


 ### Solution:

 ```python
 SELECT
    p.product_id,
    p.product_name,
    SUM(p.stock_quantity) AS total_stock_quantity,
    COUNT(o.order_id) AS total_orders,
    CASE
        WHEN COUNT(o.order_id) = 0 THEN 0
        ELSE SUM(p.stock_quantity) / COUNT(o.order_id)
    END AS stock_to_orders_ratio
FROM
    online_merchant_shop.products p
LEFT JOIN
    online_merchant_shop.orderdetails od ON p.product_id = od.product_id
LEFT JOIN
    online_merchant_shop.orders o ON od.order_id = o.order_id
GROUP BY
    p.product_id, p.product_name
ORDER BY
	p.product_id, p.product_name
LIMIT 5;
 ```



 ##### Result:

 


 ### Question:21

 How many users have made repeat purchases within 6 months?

 ### Solution:

```python
 SELECT
    user_id,
    COUNT(DISTINCT order_id) AS repeat_purchase_count
FROM
    online_merchant_shop.orders
WHERE
    order_date BETWEEN CURRENT_DATE - INTERVAL '6 months' AND CURRENT_DATE
GROUP BY
    user_id
HAVING
    COUNT(DISTINCT order_id) > 1;
```


 ##### Result:




 ### Question:22

 Which product category is most often included in large orders (e.g., total order value above $200)?

 ### Solution:

 ```python
 WITH LargeOrders AS (
    SELECT
        od.order_id,
        c.category_name
    FROM
        online_merchant_shop.orderdetails od
    JOIN
        online_merchant_shop.productcategories p ON od.product_id = p.product_id
    JOIN
        online_merchant_shop.categories c ON p.category_id = c.category_id
    GROUP BY
        od.order_id, c.category_name
    HAVING
        SUM(od.price * od.quantity) > 200
)
SELECT
    category_name,
    COUNT(*) AS order_count
FROM
    LargeOrders
GROUP BY
    category_name
ORDER BY
    order_count DESC
LIMIT 1;
 ```




 ##### Result:


