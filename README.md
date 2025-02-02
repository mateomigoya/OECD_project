# ![Apple_Changsha_RetailTeamMembers_09012021_big jpg slideshow-xlarge_2x](https://github.com/user-attachments/assets/3c0c7fe9-7dff-4ca6-8487-952168345c93)
# OECD Analysis – Using Python to Analyze the Use of R&D Tax Incentives and their Impact in Promoting R&D in OECD Countries

## Project Overview

This project is designed to showcase profieiency in Python, particularly with cleaning and wrangling data, along with demonstrating a statistical analysis and data storytelling. The dataset includes information about countries (and their OECD membership status), various tax rates, R&D incentives and support within countries, and R&D expenditures.

---

### Why I Chose This Project?
- **Hands-on Learning**: Practical experience with real macroeconomic data and advanced economic problem-solving.
- **Practicality**: This analysis appears to be relevant to that work that is done in the OECD, as this analysis was inspired by an economist working in the organization's Centre for Tax Policy and Administration.

## Data Sheets from the .CSV File

The project uses three main sheets:

1. **ISO**: 3-digit country codes and a dummy for OECD membership.
   - `country`: Country name.
   - `code`: ISO code.
   - `OECD`: Indicator for OECD membership.

2. **EATR**: Estimates of the Effective Average Tax Rate (EATR) for R&D and non-R&D investments for different countries for the years 2019 and 2020. These indicators provide an overview of the tax burden on a hypothetical firm making a standardised R&D investment and benefitting from R&D tax incentives versus a comparable non-R&D investment that does not benefit from R&D tax incentives.
   - `code`: ISO Code.
   - `year`: Year.
   - `STR`: Statutory CIT rate.
   - `EATR_rd`: Effective Average Tax Rate for an R&D investment (including R&D tax incentives).
   - `EATR_nrd`: Effective Average Tax Rate for an non-R&D investment (excluding R&D tax incentives).

3. **R&D intensity and government support**: An indicator on R&D intensity by businesses and an indicator for the amount of support governments provide for R&D through direct spending (as opposed to tax breaks).
   - `rd_gdp`: Business R&D expenditure (% GDP).
   - `df_gdp`: Direct government support for R&D, i.e. subsidies or grants given to R&D investment (% GDP).

## Objective and Workflow

The project is split into three tiers of questions to test SQL skills of increasing complexity:

1. Find the number of stores in each country.
```sql
   select 
	country,
	count(store_id) as total_stores
from stores
group by(1)
order by(2) desc;
```
2. Calculate the total number of units sold by each store.
```sql
select
	s.store_id,
	st.store_name,
	sum (s.quantity) as total_unit_sold
from sales as s
join stores as st
on st.store_id = s.store_id
group by 1, 2
order by 3 desc;

```
3. Identify how many sales occurred in December 2023.
```sql
select 
	count(sale_id) as total_sale
from sales
where to_char(sale_date,'mm-yyyy') = '12-2023';
```
4. Determine how many stores have never had a warranty claim filed.
```sql
select count(*) from stores
where store_id not in (
						select 
							distinct store_id
						from sales as s
						right join warranty as w
						on s.sale_id = w.sale_id
						);
```
5. Calculate the percentage of warranty claims marked as "Warranty Void".
```sql
select
	round
		(count(claim_id)/
						(select count(*) from warranty)::numeric
		* 100,
	2) as warranty_void_percentage
from warranty
where repair_status = 'Warranty Void';
```
6. Identify which store had the highest total units sold in the last year.
```sql
select
	round
		(count(claim_id)/
						(select count(*) from warranty)::numeric
		* 100,
	2) as warranty_void_percentage
from warranty
where repair_status = 'Warranty Void';
```
7. Count the number of unique products sold in the last year.
```sql
select 
	count(distinct product_id)
from sales 
where sale_date >= (current_date - interval '1 year');
```
8. Find the average price of products in each category.
```sql
select
	p.category_id,
	c.category_name,
	avg(p.price) as average_price
from products as p
join
category as c
on p.category_id = c.category_id
group by 1, 2
order by 3 desc;
```
9. How many warranty claims were filed in 2020?
```sql
select
	count(*) as warranty_claim
from warranty
where extract(year from claim_date) = 2020;
```
10. For each store, identify the best-selling day based on highest quantity sold.
```sql
select *
from 
(
	select
		store_id,
		to_char(sale_date, 'Day') as day_name,
		sum(quantity) as total_units_sold,
		rank() over(partition by store_id order by sum(quantity) desc) as rank
	from sales
	group by 1,2
) as tl
where rank = 1;
```
11. Identify the least selling product in each country for each year based on total units sold.
```sql
with product_rank
as
(
select
	st.country,
	p.product_name,
	sum(s.quantity) as total_qty_sold,
	rank() over(partition by st.country order by sum(s.quantity)) as rank
from sales as s
join 
stores as st
on s.store_id = st.store_id
join
products as p
on s.product_id = p.product_id
group by 1,2
)
select
*
from product_rank
where rank = 1;
```
12. Calculate how many warranty claims were filed within 180 days of a product sale.
```sql
select
	count(*)
from warranty as w
left join
sales as s
on s.sale_id = w.sale_id
where
	w.claim_date - sale_date <=180;
```
13. Determine how many warranty claims were filed for products launched in the last two years.
```sql
select
	p.product_name,
	count(w.claim_id)as no_claim,
	count(s.sale_id)
from warranty as w
right join
sales as s
on s.sale_id = w.sale_id
join
products as p
on p.product_id = s.product_id
where p.launch_date >= current_date - interval '2 years'
group by 1
having count(w.claim_id) > 0;
```
14. List the months in the last three years where sales exceeded 5,000 units in the USA.
```sql
select
	to_char(sale_date, 'MM-YYY') as month,
	sum(s.quantity) as total_units_sold
from sales as s
join
stores as st
on s.store_id = st.store_id
where
	st.country = 'USA'
	and
	s.sale_date >= current_date - interval '3 year'
group by 1
having sum(s.quantity) >= 5000;
```
15. Identify the product category with the most warranty claims filed in the last two years.
```sql
select
	c.category_name,
	count(w.claim_id) as total_claims
from warranty as w
left join
sales as s
on w.sale_id = s.sale_id
join products as p
on p.product_id = s.product_id
join category as c
on c.category_id = p.category_id
where
	w.claim_date >= current_date - interval '2 years'
group by 1;
```
16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
```sql
select
	country,
	total_units_sold,
	total_claim,
	coalesce(total_claim::numeric/total_units_sold::numeric * 100, 0)
	as risk
from
(select
	st.country,
	sum(s.quantity) as total_units_sold,
	count(w.claim_id) as total_claim
from sales as s
join stores as st
on s.store_id = st.store_id
left join warranty as w
on w.sale_id = s.sale_id
group by 1) t1
order by 4 desc;
```
17. Analyze the year-by-year growth ratio for each store.
```sql
-- each store and their yearly sale 
WITH yearly_sales
AS
(
	SELECT 
		s.store_id,
		st.store_name,
		EXTRACT(YEAR FROM sale_date) as year,
		SUM(s.quantity * p.price) as total_sale
	FROM sales as s
	JOIN
	products as p
	ON s.product_id = p.product_id
	JOIN stores as st
	ON st.store_id = s.store_id
	GROUP BY 1, 2, 3
	ORDER BY 2, 3 
),
growth_ratio
AS
(
SELECT 
	store_name,
	year,
	LAG(total_sale, 1) OVER(PARTITION BY store_name ORDER BY year) as last_year_sale,
	total_sale as current_year_sale
FROM yearly_sales
)

SELECT 
	store_name,
	year,
	last_year_sale,
	current_year_sale,
	ROUND(
			(current_year_sale - last_year_sale)::numeric/
							last_year_sale::numeric * 100
	,3) as growth_ratio
FROM growth_ratio
WHERE 
	last_year_sale IS NOT NULL
	AND 
	YEAR <> EXTRACT(YEAR FROM CURRENT_DATE)
```
18. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
```sql
-- products sold in the last five years, segmented by price range.

SELECT 
	
	CASE
		WHEN p.price < 500 THEN 'Less Expenses Product'
		WHEN p.price BETWEEN 500 AND 1000 THEN 'Mid Range Product'
		ELSE 'Expensive Product'
	END as price_segment,
	COUNT(w.claim_id) as total_Claim
FROM warranty as w
LEFT JOIN
sales as s
ON w.sale_id = s.sale_id
JOIN 
products as p
ON p.product_id = s.product_id
WHERE claim_date >= CURRENT_DATE - INTERVAL '5 year'
GROUP BY 1
```
19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
```sql
WITH paid_repair
AS
(SELECT 
	s.store_id,
	COUNT(w.claim_id) as paid_repaired
FROM sales as s
RIGHT JOIN warranty as w
ON w.sale_id = s.sale_id
WHERE w.repair_status = 'Paid Repaired'
GROUP BY 1
),

total_repaired
AS
(SELECT 
	s.store_id,
	COUNT(w.claim_id) as total_repaired
FROM sales as s
RIGHT JOIN warranty as w
ON w.sale_id = s.sale_id
GROUP BY 1)

SELECT 
	tr.store_id,
	st.store_name,
	pr.paid_repaired,
	tr.total_repaired,
	ROUND(pr.paid_repaired::numeric/
			tr.total_repaired::numeric * 100
		,2) as percentage_paid_repaired
FROM paid_repair as pr
JOIN 
total_repaired tr
ON pr.store_id = tr.store_id
JOIN stores as st
ON tr.store_id = st.store_id
```
20. Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.
```sql
WITH monthly_sales
AS
(SELECT 
	store_id,
	EXTRACT(YEAR FROM sale_date) as year,
	EXTRACT(MONTH FROM sale_date) as month,
	SUM(p.price * s.quantity) as total_revenue
FROM sales as s
JOIN 
products as p
ON s.product_id = p.product_id
GROUP BY 1, 2, 3
ORDER BY 1, 2,3
)
SELECT 
	store_id,
	month,
	year,
	total_revenue,
	SUM(total_revenue) OVER(PARTITION BY store_id ORDER BY year, month) as running_total
FROM monthly_sales
```
21. Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
```sql
SELECT 
	p.product_name,
	CASE 
		WHEN s.sale_date BETWEEN p.launch_date AND p.launch_date + INTERVAL '6 month' THEN '0-6 month'
		WHEN s.sale_date BETWEEN  p.launch_date + INTERVAL '6 month'  AND p.launch_date + INTERVAL '12 month' THEN '6-12' 
		WHEN s.sale_date BETWEEN  p.launch_date + INTERVAL '12 month'  AND p.launch_date + INTERVAL '18 month' THEN '6-12'
		ELSE '18+'
	END as plc,
	SUM(s.quantity) as total_qty_sale
	
FROM sales as s
JOIN products as p
ON s.product_id = p.product_id
GROUP BY 1, 2
ORDER BY 1, 3 DESC 
```

## Project Focus

This project primarily focuses on developing and showcasing the following SQL skills:

- **Complex Joins and Aggregations**: Demonstrating the ability to perform complex SQL joins and aggregate data meaningfully.
- **Window Functions**: Using advanced window functions for running totals, growth analysis, and time-based queries.
- **Data Segmentation**: Analyzing data across different time frames to gain insights into product performance.
- **Correlation Analysis**: Applying SQL functions to determine relationships between variables, such as product price and warranty claims.
- **Real-World Problem Solving**: Answering business-related questions that reflect real-world scenarios faced by data analysts.


## Dataset

- **Size**: 1 million+ rows of sales data.
- **Period Covered**: The data spans multiple years, allowing for long-term trend analysis.
- **Geographical Coverage**: Sales data from Apple stores across various countries.

## Conclusion

By completing this project, I have developed advanced SQL querying skills, improved my ability to handle large datasets, and gained practical experience in solving complex data analysis problems that are crucial for business decision-making.

---
