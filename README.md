# ☕ Monday Coffee Expansion SQL Project

![Company Logo](https://github.com/najirh/Monday-Coffee-Expansion-Project-P8/blob/main/1.png)

---

## 📍 **Objective**
Analyze the sales data of **Monday Coffee**, which has been selling products online since **January 2023**, to recommend the **top three major cities in India** for opening new coffee shop locations. Recommendations are based on **consumer demand, sales performance, and market potential**.

---

## 🗂️ **Project Structure**
```
![ERD](/assets/eer_diagram.png)
```
## 🗂️ Database Structure
The database consists of four interrelated tables:

### 1️⃣ **city**
- `city_id` (INT, Primary Key): Unique city identifier
- `city_name` (VARCHAR(15)): Name of the city
- `population` (BIGINT): City population
- `estimated_rent` (FLOAT): Average rent in the city
- `city_rank` (INT): City rank based on certain criteria

### 2️⃣ **customers**
- `customer_id` (INT, Primary Key): Unique customer identifier
- `customer_name` (VARCHAR(25)): Customer name
- `city_id` (INT, Foreign Key): References `city(city_id)`

### 3️⃣ **products**
- `product_id` (INT, Primary Key): Unique product identifier
- `product_name` (VARCHAR(35)): Product name
- `price` (FLOAT): Product price

### 4️⃣ **sales**
- `sale_id` (INT, Primary Key): Unique sale identifier
- `sale_date` (DATE): Date of the sale
- `product_id` (INT, Foreign Key): References `products(product_id)`
- `customer_id` (INT, Foreign Key): References `customers(customer_id)`
- `total` (FLOAT): Total amount for the sale
- `rating` (INT): Customer rating for the sale

---

## 📝 **Key Questions Answered**
1. **Coffee Consumers Count:** Estimated coffee consumers per city (25% of the population).
2. **Total Revenue:** Total revenue from coffee sales in Q4 2023.
3. **Product Sales Count:** Units sold per coffee product.
4. **Average Sales per City:** Average sales amount per customer in each city.
5. **City Population & Coffee Consumers:** Population and estimated coffee consumers per city.
6. **Top Selling Products:** Top 3 selling products in each city.
7. **Customer Segmentation:** Unique coffee product buyers per city.
8. **Average Sale vs. Rent:** Compare average sale per customer with average rent per customer.
9. **Monthly Sales Growth:** Monthly sales trends and growth rates.
10. **Market Potential:** Top 3 cities by sales, rent, customer base, and coffee consumer estimates.

---
## 📦 **SQL Code Overview**

### 🗃️ **Code 1: Database Schema (`1_schemas.sql`)**
- Creates four main tables: 
  - `city`: City details with population and rent estimates.
  - `customers`: Customer information linked to cities.
  - `products`: Coffee product catalog.
  - `sales`: Transaction data with sales amounts and product ratings.

### 📈 **Code 2: Data Analysis (`2_analysis.sql`)**
- Provides solutions for the 10 key business questions.
- Utilizes CTEs, JOINs, aggregate functions, and window functions.
- Analyzes sales trends, customer segmentation, and market potential.

---

## 🧩 **Technologies Used**
- **SQL:** Data cleaning, analysis, and reporting.
- **MySQL:** Database management system.
- **Git & GitHub:** Version control and project hosting.


## 📝 Setup Instructions

### 🚀 Step 1: Create Tables
Run the SQL script in the following order to avoid foreign key issues:
1. **city**
2. **products**
3. **customers**
4. **sales**

```sql
DROP TABLE IF EXISTS sales, customers, products, city;

CREATE TABLE city (
  city_id INT PRIMARY KEY,
  city_name VARCHAR(15),
  population BIGINT,
  estimated_rent FLOAT,
  city_rank INT
);

CREATE TABLE products (
  product_id INT PRIMARY KEY,
  product_name VARCHAR(35),
  price FLOAT
);

CREATE TABLE customers (
  customer_id INT PRIMARY KEY,
  customer_name VARCHAR(25),
  city_id INT,
  FOREIGN KEY (city_id) REFERENCES city(city_id)
);

CREATE TABLE sales (
  sale_id INT PRIMARY KEY,
  sale_date DATE,
  product_id INT,
  customer_id INT,
  total FLOAT,
  rating INT,
  FOREIGN KEY (product_id) REFERENCES products(product_id),
  FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

---

## 📈 Data Analysis Queries

### 🔢 1. Coffee Consumers Count
Estimate how many people consume coffee in each city (25% of population):
```sql
SELECT city_name, ROUND((population * 0.25)/1000000, 2) AS coffee_consumers_in_millions, city_rank
FROM city
ORDER BY coffee_consumers_in_millions DESC;
```

### 💵 2. Total Revenue from Coffee Sales in Q4 2023
```sql
SELECT SUM(total) AS total_revenue
FROM sales
WHERE YEAR(sale_date) = 2023 AND QUARTER(sale_date) = 4;
```


### ☕ 3. Sales Count for Each Product
```sql
SELECT p.product_name, COUNT(s.sale_id) AS total_orders
FROM products p
LEFT JOIN sales s ON s.product_id = p.product_id
GROUP BY p.product_name
ORDER BY total_orders DESC;
```

### 📊 4. Average Sales Amount per Customer by City
```sql
SELECT ci.city_name, SUM(s.total) AS total_revenue, COUNT(DISTINCT s.customer_id) AS total_customers,
ROUND(SUM(s.total)/COUNT(DISTINCT s.customer_id), 2) AS avg_sale_per_customer
FROM sales s
JOIN customers c ON s.customer_id = c.customer_id
JOIN city ci ON ci.city_id = c.city_id
GROUP BY ci.city_name
ORDER BY total_revenue DESC;
```

### 🏆 5. City Population and Coffee Consumers

```sql
WITH city_table as 
(
	SELECT 
		city_name,
		ROUND((population * 0.25)/1000000, 2) as coffee_consumers
	FROM city
),
customers_table AS
(
	SELECT 
		ci.city_name,
		COUNT(DISTINCT c.customer_id) as unique_cx
	FROM sales as s
	JOIN customers as c
	ON c.customer_id = s.customer_id
	JOIN city as ci
	ON ci.city_id = c.city_id
	GROUP BY 1
)
SELECT 
	customers_table.city_name,
	city_table.coffee_consumers as coffee_consumer_in_millions,
	customers_table.unique_cx
FROM city_table
JOIN customers_table ON city_table.city_name = customers_table.city_name;
```

### 📅 6. Top Selling Products by City
```sql
SELECT * 
FROM -- table
(
	SELECT 
		ci.city_name,
		p.product_name,
		COUNT(s.sale_id) as total_orders,
		DENSE_RANK() OVER(PARTITION BY ci.city_name ORDER BY COUNT(s.sale_id) DESC) as rank_
	FROM sales as s
	JOIN products as p
	ON s.product_id = p.product_id
	JOIN customers as c
	ON c.customer_id = s.customer_id
	JOIN city as ci
	ON ci.city_id = c.city_id
	GROUP BY 1, 2
) as t1
WHERE rank_ <= 3;
  
```

### 🏙️ 7. Customer Segmentation by City

```sql
SELECT 
	ci.city_name,
	COUNT(DISTINCT c.customer_id) as unique_cx
FROM city ci
LEFT JOIN customers c ON c.city_id = ci.city_id
JOIN sales s ON s.customer_id = c.customer_id
WHERE s.product_id IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14)
GROUP BY ci.city_name;
```
### 🏙️ 8. Average Sale vs Rent

```sql
WITH city_table AS
(
	SELECT 
		ci.city_name,
		SUM(s.total) as total_revenue,
		COUNT(DISTINCT s.customer_id) as total_cx,
		ROUND(SUM(s.total) / COUNT(DISTINCT s.customer_id), 2) as avg_sale_pr_cx
	FROM sales as s
	JOIN customers as c
	ON s.customer_id = c.customer_id
	JOIN city as ci
	ON ci.city_id = c.city_id
	GROUP BY ci.city_name
),
city_rent AS
(
	SELECT 
		city_name, 
		estimated_rent
	FROM city
)
SELECT 
	cr.city_name,
	cr.estimated_rent,
	ct.total_cx,
	ct.avg_sale_pr_cx,
	ROUND(cr.estimated_rent / ct.total_cx, 2) as avg_rent_per_cx
FROM city_rent as cr
JOIN city_table as ct
ON cr.city_name = ct.city_name
ORDER BY ct.avg_sale_pr_cx DESC;
```

### 🏙️ 9. Monthly Sales Growth

```sql
WITH monthly_sales AS
(
	SELECT 
		ci.city_name,
		MONTH(sale_date) AS month,
		YEAR(sale_date) AS year,
		SUM(s.total) AS total_sale
	FROM sales s
	JOIN customers c ON c.customer_id = s.customer_id
	JOIN city ci ON ci.city_id = c.city_id
	GROUP BY ci.city_name, MONTH(sale_date), YEAR(sale_date)
),
growth_ratio AS
(
	SELECT
		city_name,
		month,
		year,
		total_sale AS cr_month_sale,
		LAG(total_sale) OVER(PARTITION BY city_name ORDER BY year, month) AS last_month_sale
	FROM monthly_sales
)
SELECT
	city_name,
	month,
	year,
	cr_month_sale,
	last_month_sale,
	ROUND((cr_month_sale - last_month_sale) / last_month_sale * 100, 2) AS growth_ratio
FROM growth_ratio
WHERE last_month_sale IS NOT NULL;
```
### 🏙️ 10. Market Potential Analysis

```sql
WITH city_table AS
(
	SELECT 
		ci.city_name,
		SUM(s.total) AS total_revenue,
		COUNT(DISTINCT s.customer_id) AS total_cx,
		ROUND(SUM(s.total)/COUNT(DISTINCT s.customer_id), 2) AS avg_sale_pr_cx
	FROM sales s
	JOIN customers c ON s.customer_id = c.customer_id
	JOIN city ci ON ci.city_id = c.city_id
	GROUP BY ci.city_name
),
city_rent AS
(
	SELECT 
		city_name, 
		estimated_rent,
		ROUND((population * 0.25)/1000000, 3) AS estimated_coffee_consumer_in_millions
	FROM city
)
SELECT 
	cr.city_name,
	total_revenue,
	cr.estimated_rent AS total_rent,
	ct.total_cx,
	estimated_coffee_consumer_in_millions,
	ct.avg_sale_pr_cx,
	ROUND(cr.estimated_rent / ct.total_cx, 2) AS avg_rent_per_cx
FROM city_rent cr
JOIN city_table ct ON cr.city_name = ct.city_name
ORDER BY total_revenue DESC;
```

---





## 🚀 **Key Insights**

### 📊 **1. Revenue Leaders vs. Rent Costs**
- **Top 3 Revenue Cities:**
  - 🥇 Pune: ₹1,258,290 *(Avg. rent per customer: ₹294.23)*  
  - 🥈 Chennai: ₹944,120 *(Avg. rent per customer: ₹407.14)*  
  - 🥉 Bangalore: ₹860,110 *(Avg. rent per customer: ₹761.54)*  

🔎 **Insight:**  
✅ Pune has the highest revenue with efficient rent costs.  
⚠️ Bangalore’s high rent per customer may reduce profitability.

---

### 🧑‍💼 **2. Customer Acquisition Efficiency**
- **Cities with most customers:**
  - Jaipur: 69 customers *(Avg. rent per customer: ₹156.52)*  
  - Delhi: 68 customers *(Avg. rent per customer: ₹330.88)*  

🔎 **Insight:**  
✅ Jaipur offers **low rent costs** and a large customer base, making it cost-effective for customer acquisition.

---

### 🏠 **3. Rent Burden Analysis**
- **Highest rent per customer:**
  - Mumbai: ₹1,166.67  
  - Hyderabad: ₹1,071.43  
  - Ahmedabad: ₹626.09  

🔎 **Insight:**  
⚠️ Mumbai and Hyderabad’s high rents warrant a focus on **premium product sales** or expense reduction.

---

### ☕ **4. Coffee Consumer Market Potential**
- **Cities with largest estimated coffee consumer base:**
  - Delhi: 7.75M  
  - Mumbai: 5.10M  
  - Kolkata: 3.73M  

🔎 **Insight:**  
📈 Mumbai has **untapped potential** with high population but lower-than-expected revenue.  
✅ Delhi’s large consumer base presents opportunities for targeted marketing.

---

### 💵 **5. Sales Efficiency (Avg. Sale per Customer)**
- **Highest average sales per customer:**
  - Pune: ₹24,197.88  
  - Chennai: ₹22,479.05  
  - Bangalore: ₹22,054.10  

🔎 **Insight:**  
✅ Focus on **premium products** and **customer retention** in these cities to maintain spending levels.

---

## 📢 **Recommendations**
✅ **Invest in Pune & Jaipur:** High revenue, strong customer base, and cost-effective rents.  
✅ **Boost marketing in Mumbai & Delhi:** High market potential with room for growth.  
✅ **Reassess operations in Bangalore & Hyderabad:** Reduce rent burdens for better margins.  
✅ **Introduce premium offers in Chennai:** Consumers willing to spend more.  

---


---

