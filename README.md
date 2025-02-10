# Monday Coffee Analysis 
![Alt Text](logo_redandwhite+(1).png)

## Introduction
This project consists of a database schema and SQL queries designed to analyze sales data, customer behavior, and city-related insights for a Monday-coffee Analysis.

## Database Schema
The database contains four main tables:
- `city`
- `customers`
- `products`
- `sales`

### 1. Creating Tables

#### City Table
```sql
CREATE TABLE city (
    city_id INT PRIMARY KEY,
    city_name VARCHAR(15),    
    population BIGINT,
    estimated_rent FLOAT,
    city_rank INT
);
```

#### Customers Table
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,    
    customer_name VARCHAR(25),    
    city_id INT,
    CONSTRAINT fk_city FOREIGN KEY (city_id) REFERENCES city(city_id)
);
```

#### Products Table
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(35),    
    Price FLOAT
);
```

#### Sales Table
```sql
CREATE TABLE sales (
    sale_id INT PRIMARY KEY,
    sale_date DATE,
    product_id INT,
    customer_id INT,
    total FLOAT,
    rating INT,
    CONSTRAINT fk_products FOREIGN KEY (product_id) REFERENCES products(product_id),
    CONSTRAINT fk_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id) 
);
```

## Queries

### 1. View All Cities
```sql
SELECT * FROM city;
```

### 2. Coffee Consumer Estimation in Each City
```sql
SELECT 
    city_name,
    ROUND((population * 0.25)/1000000, 2) AS coffee_consumer_in_millions,
    city_rank
FROM city
ORDER BY 2 DESC;
```

### 3. Total Revenue for Q4 of 2023
```sql
SELECT 
    SUM(total) AS total_revenue
FROM sales
WHERE
    EXTRACT(YEAR FROM sale_date) = 2023
    AND EXTRACT(QUARTER FROM sale_date) = 4;
```

### 4. Total Revenue by City for Q4 of 2023
```sql
SELECT 
    ci.city_name,
    SUM(s.total) AS total_revenue
FROM sales AS s
JOIN customers AS c ON s.customer_id = c.customer_id
JOIN city AS ci ON ci.city_id = c.city_id
WHERE 
    EXTRACT(YEAR FROM s.sale_date) = 2023
    AND EXTRACT(QUARTER FROM s.sale_date) = 4
GROUP BY 1
ORDER BY 2 DESC;
```

### 5. Total Orders by Product
```sql
SELECT 
    p.product_name,
    COUNT(s.sale_id) AS total_orders
FROM products AS p
LEFT JOIN sales AS s ON s.product_id = p.product_id
GROUP BY 1
ORDER BY 2 DESC;
```

### 6. Average Sale per Customer by City
```sql
SELECT 
    ci.city_name,
    SUM(s.total) AS total_revenue,
    COUNT(DISTINCT s.customer_id) AS total_cx,
    ROUND(SUM(s.total)::NUMERIC / COUNT(DISTINCT s.customer_id)::NUMERIC, 2) AS avg_sale_per_cx
FROM sales AS s
JOIN customers AS c ON s.customer_id = c.customer_id
JOIN city AS ci ON ci.city_id = c.city_id
GROUP BY 1
ORDER BY 2 DESC;
```

### 7. Coffee Consumers and Unique Customers by City
```sql
WITH city_table AS (
    SELECT 
        city_name,
        ROUND((population * 0.25)/1000000, 2) AS coffee_consumers
    FROM city
),
customers_table AS (
    SELECT 
        ci.city_name,
        COUNT(DISTINCT c.customer_id) AS unique_cx
    FROM sales AS s
    JOIN customers AS c ON c.customer_id = s.customer_id
    JOIN city AS ci ON ci.city_id = c.city_id
    GROUP BY 1
)
SELECT 
    customers_table.city_name,
    city_table.coffee_consumers AS coffee_consumer_in_millions,
    customers_table.unique_cx
FROM city_table
JOIN customers_table ON city_table.city_name = customers_table.city_name;
```

### 8. Top 3 Best-Selling Products per City
```sql
SELECT * FROM (
    SELECT 
        ci.city_name,
        p.product_name,
        COUNT(s.sale_id) AS total_orders,
        DENSE_RANK() OVER (PARTITION BY ci.city_name ORDER BY COUNT(s.sale_id) DESC) AS rank
    FROM sales AS s
    JOIN products AS p ON s.product_id = p.product_id
    JOIN customers AS c ON c.customer_id = s.customer_id
    JOIN city AS ci ON ci.city_id = c.city_id
    GROUP BY 1, 2
) AS t1
WHERE rank <= 3;
```

### 9. View All Products
```sql
SELECT * FROM products;
```

### 10. Unique Customers Buying Specific Products in Each City
```sql
SELECT 
    ci.city_name,
    COUNT(DISTINCT c.customer_id) AS unique_cx
FROM city AS ci
LEFT JOIN customers AS c ON c.city_id = ci.city_id
JOIN sales AS s ON s.customer_id = c.customer_id
WHERE s.product_id IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14)
GROUP BY 1;
```

## Conclusion
This project provides structured data analysis of sales and customer behavior in different cities, along with insights into revenue generation and product demand. The SQL queries can be used to extract meaningful business insights from the dataset.
