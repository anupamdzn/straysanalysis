# Strays: Food Delivery App Analysis

## Project Overview

- Project Title: Strays - Data Analysis of a Food Delivery App
- Database Name: strays_db
- Technology: PostgreSQL

This project simulates and analyzes a food delivery app similar to Zomato, aimed at understanding customer behavior, delivery efficiency, rider performance, and restaurant trends.

## Objectives
- Database Creation & Population: Design and populate a relational database with realistic, structured data across five key entities.
- Data Cleaning: Ensure the data is usable by removing or handling invalid or missing entries.
- Exploratory Data Analysis (EDA): Uncover insights and patterns in customer, order, delivery, and restaurant data.
- Business Intelligence: Solve at least 15 realistic business problems using SQL queries to support decision-making.

## Dataset

- The data for this project is sourced from ChatGPT.

## Schema

  ```sql
  CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100),
    reg_date DATE
);

CREATE TABLE restaurants (
    restaurant_id SERIAL PRIMARY KEY,
    restaurant_name VARCHAR(100),
    city VARCHAR(50),
    opening_hours VARCHAR(50)
);

CREATE TABLE riders (
    rider_id SERIAL PRIMARY KEY,
    rider_name VARCHAR(100),
    signup_date DATE
);

CREATE TABLE deliveries (
    delivery_id SERIAL PRIMARY KEY,
    delivery_status VARCHAR(20),
    delivery_time VARCHAR(20),
    rider_id INT REFERENCES riders(rider_id)
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    restaurant_id INT REFERENCES restaurants(restaurant_id),
    order_item VARCHAR(100),
    order_date DATE,
    order_time TIME,
    order_status VARCHAR(20),
    total_amount FLOAT
);
```

## Business Problems and Solutions

### 1. How many active customers are there on the platform?

```sql
SELECT COUNT(DISTINCT customer_id) AS active_customers
FROM orders
WHERE order_date >= CURRENT_DATE - INTERVAL '30 days';
```

### 2. Which cities have the highest restaurant density on the platform?

```sql
SELECT city, COUNT(*) AS total_restaurants
FROM restaurants
GROUP BY city
ORDER BY total_restaurants DESC;
```

### 3. Who are the top 10 high-value customers based on total spending?

```sql
SELECT c.customer_name, SUM(o.total_amount) AS total_spent
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_status = 'Completed'
GROUP BY c.customer_name
ORDER BY total_spent DESC
LIMIT 10;
```

### 4. Which are the top 5 best-performing restaurants based on number of completed orders? 

```sql
SELECT r.restaurant_name, COUNT(*) AS completed_orders
FROM orders o
JOIN restaurants r ON o.restaurant_id = r.restaurant_id
WHERE o.order_status = 'Completed'
GROUP BY r.restaurant_name
ORDER BY completed_orders DESC
LIMIT 5;
```

### 5. What are the most frequently ordered food items across the platform?

```sql
SELECT order_item, COUNT(*) AS times_ordered
FROM orders
GROUP BY order_item
ORDER BY times_ordered DESC
LIMIT 10;
```

### 6. Daily revenue in the last month.
```sql
SELECT order_date, SUM(total_amount) AS daily_revenue
FROM orders
WHERE order_date >= CURRENT_DATE - INTERVAL '30 days'
AND order_status = 'Completed'
GROUP BY order_date
ORDER BY order_date;
```

### 7. What time of day sees the most orders?

```sql
SELECT EXTRACT(HOUR FROM order_time) AS hour_of_day,
       COUNT(*) AS total_orders
FROM orders
GROUP BY hour_of_day
ORDER BY total_orders DESC;
```

### 8.  Find the number of unique customers who placed orders in each restaurant.

```sql
SELECT 
  r.restaurant_name, 
  COUNT(DISTINCT o.customer_id) AS unique_customers
FROM orders o
JOIN restaurants r ON o.restaurant_id = r.restaurant_id
GROUP BY r.restaurant_name;
```

### 9. Which riders have the fastest average delivery times (min 10 completed deliveries)?
  
```sql
  SELECT r.rider_name,
       ROUND(AVG(CAST(SPLIT_PART(d.delivery_time, ' ', 1) AS INTEGER)), 2) AS avg_minutes
FROM deliveries d
JOIN riders r ON d.rider_id = r.rider_id
WHERE d.delivery_status = 'Completed'
GROUP BY r.rider_name
HAVING COUNT(*) >= 10
ORDER BY avg_minutes ASC
LIMIT 10;
```

### 10. What is the cancellation rate per city?

```sql
SELECT r.city,
       ROUND(SUM(CASE WHEN o.order_status = 'Cancelled' THEN 1 ELSE 0 END)::DECIMAL / COUNT(*) * 100, 2) AS cancellation_rate
FROM orders o
JOIN restaurants r ON o.restaurant_id = r.restaurant_id
GROUP BY r.city
ORDER BY cancellation_rate DESC;
```

### 11. Which customers placed more than 5 orders overall?

```sql
SELECT c.customer_name, COUNT(*) AS total_orders
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
GROUP BY c.customer_name
HAVING COUNT(*) > 5
ORDER BY total_orders DESC;
```

### 12. What is the average order value per restaurant?

```sql
SELECT 
  r.restaurant_name, 
  ROUND(AVG(o.total_amount)::numeric, 2) AS avg_order_value
FROM orders o
JOIN restaurants r ON o.restaurant_id = r.restaurant_id
WHERE o.order_status = 'Completed'
GROUP BY r.restaurant_name
ORDER BY avg_order_value DESC;
```

### 13. What percentage of total orders are completed, pending, or cancelled?

```sql
SELECT order_status,
       ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS percentage
FROM orders
GROUP BY order_status;
```

### 14. Which riders completed the most deliveries last month?

```sql
SELECT r.rider_name, COUNT(*) AS deliveries_completed
FROM deliveries d
JOIN riders r ON d.rider_id = r.rider_id
WHERE d.delivery_status = 'Completed'
  AND d.delivery_id IN (
    SELECT order_id FROM orders
    WHERE order_date >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1 month')
      AND order_status = 'Completed'
)
GROUP BY r.rider_name
ORDER BY deliveries_completed DESC
LIMIT 10;
```

### 15. Find the average delivery time for each rider (in minutes).

```sql
SELECT r.rider_name, 
  ROUND(AVG(CASE 
    WHEN d.delivery_time LIKE '%minutes' THEN 
        CAST(SUBSTRING(d.delivery_time FROM 1 FOR POSITION(' ' IN d.delivery_time) - 1) AS INT)
    ELSE 0 
  END), 2) AS avg_delivery_time
FROM deliveries d
JOIN riders r ON d.rider_id = r.rider_id
GROUP BY r.rider_name
order by avg_delivery_time asc
```


## 🧾 Findings & Conclusion

Through this PostgreSQL-powered analysis of the Strays food delivery platform, we uncovered valuable business insights such as:
- Identification of top-performing restaurants based on total sales and average order value.
- Discovery of the most loyal and high-spending customers, enabling targeted marketing strategies.
- Evaluation of rider performance by delivery count and average delivery time, aiding operational efficiency.
- Analysis of order trends and popular items to support inventory and menu optimization.
- Insight into customer behavior across cities, helping the business expand intelligently.

This project showcases how structured data and SQL queries can drive data-backed decisions and improve the efficiency and profitability of a real-world delivery business.









