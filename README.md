# Consumer_Goods_Adhoc_Insights_UsingSQL
 Analyze the given database and provide insights to stakeholders.

 Hello everyone,
"Ever juggled with more than a million records? I have! Let's chat about the adventures in handling a colossal sea of data", from the resume project challenge#4 by @Codebasics.

**Problem Statement:**

AtliQ is a fast-growing hardware company that sells various products to customers in many countries through various channels, including retail, direct sales, and distributor networks, both online and in brick-and-mortar stores. To support its growth and make data-driven decisions, AtliQ needs a data analytics team.

**𝐓ools 𝐔𝐬𝐞𝐝 -**
- MySQL Workbench (For getting insights using SQL)
- Microsoft Excel (For data visualization)
- Microsoft PowerPoint (For presentation)
  
**Knowledge gained in:**

**1.	Database Exploration with MySQL Workbench:**
     Loading databases into MySQL Workbench, Navigating and exploring tables within the database.

**2.	Data Management Concepts:** 
     Understanding ETL (Extract, Transform, Load) processes, Grasping the distinctions between Data Warehouses, OLAP (Online Analytical Processing), and OLTP (Online Transactional Processing).
    
**3.	Database Design Principles:**
     Implementing normalization techniques, Creating fact tables and dimension tables, Constructing star schema and snowflake schema for efficient data organization.
     
**4.	Data Integrity and Relationships:**
     Addressing data duplication issues, Ensuring data integrity through primary keys and foreign keys.
     
**5.	SQL Querying Expertise:**
     Proficient in basic SQL queries,	Mastery of various JOIN types for effective data retrieval.
     
**6.	Advanced Query Techniques:**
    	Utilizing subqueries and correlated subqueries, Implementing Common Table Expressions (CTEs), Views, and temporary tables for improved query organization.
     
**7.	Custom Functions and Stored Procedures:**
     Writing custom functions to enhance data manipulation, Creating and optimizing stored procedures for efficient database operations.

**8.	Understanding Datatypes:**
     Knowledge of diverse data types and their applications in database design.

**9.	Window Functions:**
     Expertise in window functions such as OVER, ROW_NUMBER, RANK, DENSE_RANK for advanced data analysis.
     
**10.	Database Optimization Techniques:**
      Implementing indexes for faster query performance, Creating triggers and events for automated database responses, Managing user accounts and permissions to ensure secure and controlled access to the database.

**10 Ad_Hoc Queries to solve:**

#1. Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region.

Solution:
```
select distinct(market) from dim_customer
where customer = "Atliq Exclusive" and region = "APAC";
```
#2. What is the percentage of unique product increase in 2021 vs. 2020? The final output contains these fields: unique_products_2020, unique_products_2021, percentage_chg.

Solution:
```
with cte1 as
(select count(distinct(product_code)) as unique_products_2020 from fact_sales_monthly where fiscal_year = 2020),
cte2 as
(select count(distinct(product_code)) as unique_products_2021 from fact_sales_monthly where fiscal_year = 2021)
select unique_products_2020, 
unique_products_2021, 
round((unique_products_2021-unique_products_2020)*100/unique_products_2020,2) as percentage_chg 
from cte1, cte2;
```
#3. Provide a report with all the unique product counts for each segment and sort them in descending order of product counts. The final output contains 2 fields: segment,product_count.

Solution:
```
select segment, 
count(distinct(product_code))as product_count 
from dim_product
group by segment
order by product_count desc;
```
#4. Follow-up: Which segment had the most increase in unique products in 2021 vs 2020? The final output contains these fields: segment, product_count_2020, product_count_2021, difference.

Solution:
```
with cte1 as
(select segment, 
count(distinct(p1.product_code))as product_count_2020 
from dim_product p1 
join fact_sales_monthly s
using (product_code)
where s.fiscal_year = 2020
group by segment
),
cte2 as
(select segment, 
count(distinct(p2.product_code))as product_count_2021 
from dim_product p2 
join fact_sales_monthly s
using (product_code)
where s.fiscal_year = 2021
group by segment
)
select segment, product_count_2020,  product_count_2021,  (product_count_2021- product_count_2020) as difference
from cte1
join cte2 
using (segment)
order by difference desc;
```
#5. Get the products that have the highest and lowest manufacturing costs. The final output should contain these fields: product_code, product, manufacturing_cost.

Solution:
```
select product_code, product, manufacturing_cost 
from fact_manufacturing_cost m
join dim_product p
using (product_code)
where manufacturing_cost = (select min(manufacturing_cost) from fact_manufacturing_cost)

or
manufacturing_cost = (select max(manufacturing_cost) from fact_manufacturing_cost);
```
#6. Generate a report which contains the top 5 customers who received an average high pre_invoice_discount_pct for the fiscal year 2021 and in the Indian market. The final output contains these fields customer_code, customer, average_discount_percentage.

Solution:
```
select customer_code, customer, pre_invoice_discount_pct as average_discount_percentage
from dim_customer c
join fact_pre_invoice_deductions pre_inv
using (customer_code)
where fiscal_year = 2021 
and market = 'India'
order by average_discount_percentage desc
limit 5;
```
#7. Get the complete report of the Gross sales amount for the customer “Atliq Exclusive” for each month. This analysis helps to get an idea of low and high-performing months and take strategic decisions. The final report contains these columns: Month, Year,Gross sales Amount.

Solution:
```
select monthname(date) as month, fiscal_year, round(sum(gross_price*sold_quantity)/1000000,2) as gross_sales_mln
from fact_sales_monthly s
join fact_gross_price fgp
using (product_code, fiscal_year)
join dim_customer c
using (customer_code)
where customer = "Atliq Exclusive" 
group by month, fiscal_year
order by fiscal_year;
```
#8. In which quarter of 2020, got the maximum total_sold_quantity? The final output contains these fields sorted by the total_sold_quantity. Quarter, total_sold_quantity.

Solution:
```
select get_qtr(date) as quarter, sum(sold_quantity) as total_sold_quantity
from fact_sales_monthly
where fiscal_year= 2020
group by quarter
order by total_sold_quantity desc;
```
#9. Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution? The final output contains these fields: channel, gross_sales_mln, percentage.

Solution:
```
with cte as
(select channel, round(sum(gross_price*sold_quantity)/1000000,2) as gross_sales_mln
from fact_sales_monthly s
join fact_gross_price fgp
using (product_code, fiscal_year)
join dim_customer c
using (customer_code) 
where fiscal_year = 2021
group by channel)

select channel, gross_sales_mln, (gross_sales_mln*100)/sum(gross_sales_mln) over() as pct
from cte
order by pct desc;
```

#10. Get the Top 3 products in each division that have a high total_sold_quantity in the fiscal_year 2021? The final output contains these fields: division, product_code, product, total_sold_quantity, rank_order.

Solution:
```
with cte as
(select division, product_code, product, sum(sold_quantity) as total_sold_quantity,
rank() over(partition by division order by sum(sold_quantity) desc) as rank_order
from dim_product p
join fact_sales_monthly s
using (product_code)
where fiscal_year = 2021
group by product_code, product, division
order by total_sold_quantity desc)
select division, product_code, product, total_sold_quantity, rank_order
from cte
where rank_order <=3;
```
**Insights and Conclusion:**

In the project, I delved into 10 Ad-hoc queries, unraveling key details about top products, customers, markets, and crucial metrics like gross price, pre-invoice deductions, manufacturing costs, and sales quantity. “It's all about turning data into actionable insights!"

LinkedIn: https://www.linkedin.com/in/venkata-siva-santhi-anantha-71634a27a/

I would like to thank @Dhaval Patel, @Hemanand Vadivel, and the entire @Codebasics team for this wonderful opportunity. 


