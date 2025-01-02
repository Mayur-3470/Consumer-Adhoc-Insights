# Consumer-Adhoc-Insights

AD-Hoc Insight (MYSQL Queries)

1) Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region?
Ans: Select market 
from dim_customer
where customer = "Atliq Exclusive"
AND region = "APAC"
Order by market

2) What is the percentage of unique product increase in 2021 vs. 2020? The final output contains these fields, 
unique_products_2020 
unique_products_2021 
percentage_chg?
Ans: Select 
A.unq_2020 as unique_products_2020,
B.unq_2021 as unique_products_2021,
ROUND((A.unq_2020-B.unq_2021)*100/A.unq_2020,2) as pct_change
FROM
(Select COUNT(distinct(product_code)) as unq_2020
from fact_sales_monthly
where fiscal_year = 2020) A,

(Select COUNT(distinct(product_code)) as unq_2021
from fact_sales_monthly
where fiscal_year = 2021) B

3) Provide a report with all the unique product counts for each segment and sort them in descending order of product counts. The final output contains 2 fields,
segment
product_count?
Ans: Select Segment , count(product) as product_count
from dim_product
group by segment
order by product_count DESC

4) Follow-up: Which segment had the most increase in unique products in 2021 vs 2020? The final output contains these fields, 
segment product_count_2020 
product_count_2021 
difference?
Ans: WITH cte1 AS 
(SELECT
segment as s1,
COUNT(DISTINCT(s.product_code)) AS cnt1
FROM dim_product p, fact_sales_monthly s
WHERE p.product_code=s.product_code
GROUP BY p.segment, s.fiscal_year
HAVING s.fiscal_year='2020'),

cte2 AS
(SELECT
segment AS s2,
COUNT(DISTINCT(s.product_code)) AS cnt2
FROM dim_product p, fact_sales_monthly s
WHERE p.product_code=s.product_code
GROUP BY p.segment, s.fiscal_year
HAVING s.fiscal_year='2021')

SELECT
cte1.s1 AS segment,
cte1.cnt1 AS product_count_2020,
cte2.cnt2 AS product_count_2021,
(cte2.cnt2-cte1.cnt1) AS difference
FROM cte1, cte2
WHERE cte1.s1=cte2.s2;

5) Get the products that have the highest and lowest manufacturing costs. The final output should contain these fields, 
product_code 
product 
manufacturing_cost?
Ans: (SELECT
m.product_code,
p.product,
m.manufacturing_cost
FROM fact_manufacturing_cost m 
JOIN dim_product p 
ON m.product_code=p.product_code
ORDER BY m.manufacturing_cost DESC
LIMIT 1) -- Highest cost product
UNION
(SELECT
m.product_code,
p.product,
m.manufacturing_cost
FROM fact_manufacturing_cost m
JOIN dim_product p
ON m.product_code=p.product_code
ORDER BY m.manufacturing_cost ASC
LIMIT 1) -- Lowest cost product;

6) Generate a report which contains the top 5 customers who received an average high pre_invoice_discount_pct for the fiscal year 2021 and in the Indian market. The final output contains these fields, 
customer_code 
customer 
average_discount_percentage?
Ans: Select 
pr.customer_code,
c.customer,
pr.pre_invoice_discount_pct as Average_dst_pct
from fact_pre_invoice_deductions pr
join dim_customer c
on pr.customer_code = c.customer_code
where pre_invoice_discount_pct > (Select avg(pre_invoice_discount_pct) from fact_pre_invoice_deductions)
And c.market = "india"
order by average_dst_pct DESC
limit 5

7) Get the complete report of the Gross sales amount for the customer “Atliq Exclusive” for each month . This analysis helps to get an idea of low and high-performing months and take strategic decisions. The final report contains these columns: 
Month 
Year 
Gross sales Amount?
Ans:SELECT
MONTHNAME(s.date)  AS month,
s.fiscal_year,
ROUND(SUM(s.sold_quantity*g.gross_price),2) as gross_sales_amount
FROM fact_sales_monthly s
JOIN fact_gross_price g
ON s.product_code=g.product_code
JOIN dim_customer c
ON s.customer_code=c.customer_code
WHERE c.customer='Atliq Exclusive'
GROUP BY month , s.fiscal_year
ORDER BY s.fiscal_year;

8) In which quarter of 2020, got the maximum total_sold_quantity? The final output contains these fields sorted by the total_sold_quantity, Quarter total_sold_quantity?
Ans: SELECT
CASE
WHEN date BETWEEN '2019-09-01' AND '2019-11-01' THEN 'Q1'
WHEN date BETWEEN '2019-12-01' AND '2020-02-01' THEN 'Q2'
WHEN date BETWEEN '2020-03-01' AND '2020-05-01' THEN 'Q3'
WHEN date BETWEEN '2020-06-01' AND '2020-08-01' THEN 'Q4'
END AS quarters,
ROUND(SUM(sold_quantity)/1000000,2) AS total_sold_quantity_in_mln
FROM fact_sales_monthly
WHERE fiscal_year = 2020
GROUP BY quarters
ORDER BY total_sold_quantity_in_mln DESC;

9) Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution? The final output contains these fields, 
channel 
gross_sales_mln 
percentage?
Ans: WITH cte1 AS (
SELECT c.channel, SUM(s.sold_quantity*g.gross_price) AS total_sales
FROM fact_sales_monthly s
JOIN dim_customer c ON s.customer_code=c.customer_code
JOIN fact_gross_price g ON s.product_code=g.product_code
WHERE s.fiscal_year=2021
GROUP BY c.channel
ORDER BY total_sales DESC)

SELECT channel, ROUND(total_sales/1000000,2) AS gross_sales_in_mln,
ROUND(total_sales/(SUM(total_sales) OVER())*100,2) AS percentage
FROM cte1;

10) Get the Top 3 products in each division that have a high total_sold_quantity in the fiscal_year 2021? The final output contains these fields, 
division 
product_code?
Ans: WITH cte1 AS (
SELECT
p.division, s.product_code, p.product,
SUM(s.sold_quantity) as total_sold_quantity
FROM fact_sales_monthly s
JOIN dim_product P 
ON s.product_code=p.product_code
WHERE s.fiscal_year = 2021 
GROUP BY s.product_code, division, P.product),

cte2 AS (
SELECT division, product_code, product, total_sold_quantity,
RANK() OVER(PARTITION BY division ORDER BY total_sold_quantity DESC) AS 'rank_order'
FROM cte1)

SELECT cte1.division, cte1.product_code, cte1.product, cte2.total_sold_quantity, cte2.rank_order
FROM cte1 JOIN cte2 ON cte1.product_code=cte2.product_code
WHERE cte2.rank_order IN (1,2,3);
