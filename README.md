# Retail_Sales_Data_Anlalysis_SQL-
This project analyzes retail sales data using SQL to identify top-performing products and customer purchasing behavior by age and gender.
I designed a star schema and performed analytical queries using joins, aggregations, and window functions.

# Data Model
I structured the data into one fact table and multiple dimension tables to support analytical queries efficiently.

. fact_sales

. dim_customers

. dim_products

# Skills Used
. SQL (MySQL)

. Joins & Aggregations

. CASE statements

. Window Functions (ROW_NUMBER)

. Data Cleaning

. Star Schema Modeling

# Business Insights

. Indentifies best-selling products

. Understands customer behavior by age_grop and gender

. Supports demographic-based targeting

# Step by Step Project Workflow
1: Load Raw Data

Imported Excel Data into Mysql.
```
create database sales_data_analysis;
use sales_data_analysis;
CREATE TABLE sales_dataset_raw(
    transaction_id BIGINT,
    order_date DATEtime,
    customer_id VARCHAR(100),
    gender varchar (20),
    age int,
	product_category VARCHAR(100),
    quantity INT,
    unit_price DECIMAL(10,2),
    total_amount DECIMAL(10,2));
```
<img width="481" height="215" alt="image" src="https://github.com/user-attachments/assets/ed1d455c-65bf-4a08-aade-fe0eab7ea64d" />

2: Create Dimension Tables

Separated descriptive data into dimension tables for clean struture and ready for analysis.

```
Create table dim_customers(customer_id varchar (100) Primary key,
gender varchar (20),
age int);
```
<img width="448" height="119" alt="image" src="https://github.com/user-attachments/assets/36b729c3-a0b5-4ff9-b1b6-6f820787fa7c" />

3: Populate Dimension Tables

Inserted distinct values from raw data.

```
INSERT INTO dim_customers (customer_id, gender, age)
SELECT DISTINCT
    TRIM(customer_id),
    LOWER(TRIM(gender)),
    age
FROM sales_dataset_raw
WHERE customer_id IS NOT NULL;
```
<img width="321" height="240" alt="image" src="https://github.com/user-attachments/assets/1a2aabe4-c305-480e-afc1-3b7ddeae1bc2" />

4: Create Fact Table

Star schema implemented and make central table storing sales metrics.

```
CREATE TABLE fact_sales (
    transaction_id BIGINT PRIMARY KEY,
    order_date DATETIME,
    customer_id VARCHAR(100),
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2),
    total_amount DECIMAL(10,2),

    CONSTRAINT fk_fact_customer
        FOREIGN KEY (customer_id)
        REFERENCES dim_customers(customer_id),

    CONSTRAINT fk_fact_product
        FOREIGN KEY (product_id)
        REFERENCES dim_products(product_id)
)
ENGINE=InnoDB;
```
<img width="462" height="181" alt="image" src="https://github.com/user-attachments/assets/d6659426-ced2-4ff9-a412-13e98ed12235" />

5: Insert Data into Fact Table

Joined raw data with dimension tables.
```
INSERT INTO fact_sales (
    transaction_id,
    order_date,
    customer_id,
    product_id,
    quantity,
    unit_price,
    total_amount
)
SELECT
    s.transaction_id,
    s.order_date,
    TRIM(s.customer_id),
    p.product_id,
    s.quantity,
    s.unit_price,
    s.total_amount
FROM sales_dataset_raw s
JOIN dim_products p
  ON LOWER(TRIM(s.product_category)) = p.product_category
JOIN dim_customers c
  ON TRIM(s.customer_id) = c.customer_id;
```

<img width="719" height="238" alt="image" src="https://github.com/user-attachments/assets/caad6191-7007-4dc6-8ce7-e6167312e7f8" />

6: Create Age Group
Derived analytical attribute by using CASE.
```
UPDATE dim_customers
SET age_group =
CASE
    WHEN age < 18 THEN 'Under 18'
    WHEN age BETWEEN 18 AND 24 THEN '18-24'
    WHEN age BETWEEN 25 AND 34 THEN '25-34'
    WHEN age BETWEEN 35 AND 44 THEN '35-44'
    WHEN age BETWEEN 45 AND 54 THEN '45-54'
    ELSE '55+'
END
WHERE customer_id <> '';
```
<img width="325" height="133" alt="image" src="https://github.com/user-attachments/assets/ff9307b6-63d6-409f-9ad4-11d05efc4fd6" />


# Sales Analysis

1: Sales Analysis per product

Insight : Revealing top revenue-generating producuts.
```
SELECT
    p.product_category,
    SUM(f.total_amount) AS total_sales
FROM fact_sales f
JOIN dim_products p
    ON f.product_id = p.product_id
GROUP BY p.product_category;
```
<img width="247" height="99" alt="image" src="https://github.com/user-attachments/assets/b55f4fa9-e197-4963-b797-88470f7c0a7d" />

2: Sales Analysis by Age_Group x Gender

```
WITH segment_sales AS (
    SELECT
        c.age_group,
        c.gender,
        p.product_category,
        SUM(f.total_amount) AS total_sales,
        ROW_NUMBER() OVER (
            PARTITION BY c.age_group, c.gender
            ORDER BY SUM(f.total_amount) DESC
        ) AS rn
    FROM fact_sales f
    JOIN dim_customers c
        ON f.customer_id = c.customer_id
    JOIN dim_products p
        ON f.product_id = p.product_id
    GROUP BY
        c.age_group,
        c.gender,
        p.product_category
)
SELECT
    age_group,
    gender,
    product_category,
    total_sales
FROM segment_sales
WHERE rn = 1;
```
<img width="399" height="245" alt="image" src="https://github.com/user-attachments/assets/c294be25-878e-4b4a-8e59-1776f14a960b" />

3: Customer Behavior Analysis

insight : Know which customer spent most on which product.

```
WITH customer_product AS (
    SELECT
        f.customer_id,
        p.product_category,
        SUM(f.total_amount) AS total_spent,
        ROW_NUMBER() OVER (
            PARTITION BY f.customer_id
            ORDER BY SUM(f.total_amount) DESC
        ) AS rn
    FROM fact_sales f
    JOIN dim_products p
        ON f.product_id = p.product_id
    GROUP BY f.customer_id, p.product_category
)
SELECT customer_id, product_category, total_spent
FROM customer_product
WHERE rn = 1;
```
<img width="351" height="368" alt="image" src="https://github.com/user-attachments/assets/b10fbb04-245b-4308-a290-510d1ba56044" />

# Conclusion

This project shows my ability to

. Design analytical databases

. Transfrom raw data into insights

. Apply SQL to real business decisions


