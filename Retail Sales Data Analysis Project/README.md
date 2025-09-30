# 📊 Retail Sales Analysis SQL Project

## 🎯 Project Objectives

-- Database Setup.
-- Data Cleaning.
-- Exploratory Data Analysis (EDA).
-- Business Analysis.

---------------------------------------------------------------------------------------------------

## 📂 Project Structure

### 1. Database Setup
```sql
CREATE DATABASE p1_retail_db;
USE p1_retail_db;

CREATE TABLE retail_sales (
    transactions_id INT PRIMARY KEY,
    sale_date DATE,
    sale_time TIME,
    customer_id INT,
    gender VARCHAR(10),
    age INT,
    category VARCHAR(35),
    quantity INT,
    price_per_unit FLOAT,
    cogs FLOAT,
    total_sale FLOAT
);
```
### 2. Data Cleaning & Exploration

-- Count all records to determine the dataset size.
SELECT COUNT(*) FROM retail_sales;

-- Check for and remove any records with null values.
SELECT * FROM retail_sales
WHERE
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR
    gender IS NULL OR age IS NULL OR category IS NULL OR
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;

-- Temporarily disable safe updates to allow for mass deletion.
```sql
SET SQL_SAFE_UPDATES = 0;
```
-- Delete records with any missing data.
```sql
DELETE FROM retail_sales
WHERE
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR
    gender IS NULL OR age IS NULL OR category IS NULL OR
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;
```
-- Re-enable safe updates.
```sql
SET SQL_SAFE_UPDATES = 1;
```
-- Find the total number of unique customers.
```sql
SELECT COUNT(DISTINCT customer_id) FROM retail_sales;
```
-- Identify all unique product categories.
```sql
SELECT DISTINCT category FROM retail_sales;
```
---------------------------------------------------------------------------------------------------

## 📈 Data Analysis & Insights

### 1. Sales on a Specific Date
-- Query: Retrieve all sales transactions that occurred on '2022-11-05'.
```sql
SELECT *
FROM retail_sales
WHERE sale_date = '2022-11-05';
```
### 2. Transactions by Category and Quantity
-- Query: Find all 'Clothing' transactions in November 2022 where the quantity sold was 4 or more.
```sql
SELECT *
FROM retail_sales
WHERE
    category = 'Clothing'
    AND sale_date BETWEEN '2022-11-01' AND '2022-11-30'
    AND quantity >= 4;
```
### 3. Total Sales per Category
-- Query: Calculate the total sales and number of orders for each product category.
```sql
SELECT
    category,
    SUM(total_sale) AS net_sale,
    COUNT(*) AS total_orders
FROM retail_sales
GROUP BY category
ORDER BY net_sale DESC;
```

### 4. Customer Demographics
-- Query: Determine the average age of customers who purchased 'Beauty' products.
```sql
SELECT ROUND(AVG(age), 2) AS avg_age
FROM retail_sales
WHERE category = 'Beauty';
```
### 5. High-Value Transactions
-- Query: Find all transactions with a total sale amount greater than $1,000.
```sql
SELECT *
FROM retail_sales
WHERE total_sale > 1000;
```
### 6. Transactions by Gender and Category
-- Query: Count the total number of transactions for each gender within each product category.
```sql
SELECT
    category,
    gender,
    COUNT(*) AS total_transactions
FROM retail_sales
GROUP BY category, gender
ORDER BY category, total_transactions DESC;
```
### 7. Best-Selling Month by Year
-- Query: Identify the best-selling month in each year based on average sales.
```sql
SELECT
    year,
    month,
    avg_sale
FROM
    (
    SELECT
        EXTRACT(YEAR FROM sale_date) AS year,
        EXTRACT(MONTH FROM sale_date) AS month,
        AVG(total_sale) AS avg_sale,
        RANK() OVER(PARTITION BY EXTRACT(YEAR FROM sale_date) ORDER BY AVG(total_sale) DESC) AS rnk
    FROM retail_sales
    GROUP BY 1, 2
    ) AS t1
WHERE rnk = 1;
```
### 8. Top 5 Customers by Sales
-- Query: Find the top 5 customers with the highest total sales.
```sql
SELECT
    customer_id,
    SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY customer_id
ORDER BY total_sales DESC
LIMIT 5;
```
### 9. Unique Customers per Category
-- Query: Count the number of unique customers for each product category.
```sql
SELECT
    category,
    COUNT(DISTINCT customer_id) AS unique_customers
FROM retail_sales
GROUP BY category
ORDER BY unique_customers DESC;
```
### 10. Orders by Time of Day
-- Query: Categorize and count the number of orders by time of day (Morning, Afternoon, Evening).
```sql
WITH hourly_sales AS (
    SELECT
        *,
        CASE
            WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
            WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
            ELSE 'Evening'
        END AS shift
    FROM retail_sales
)
SELECT
    shift,
    COUNT(*) AS total_orders
FROM hourly_sales
GROUP BY shift
ORDER BY total_orders DESC;
```
---------------------------------------------------------------------------------------------------

## ✅ Conclusion
-- This project serves as a practical, hands-on introduction to SQL for data analysts.
-- The insights gained from this analysis, such as identifying sales trends and top-performing customers and products, can directly inform business decisions.
