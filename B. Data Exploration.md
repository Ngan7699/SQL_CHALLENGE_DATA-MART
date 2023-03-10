# 🛒 SQL_CHALLENGE_DATA-MART

## 🛍 Solution - B. Data Exploration

**1. What day of the week is used for each week_date value?**

````sql
SELECT DISTINCT(DATENAME(WEEKDAY,week_date)) AS day_of_week
FROM clean_weekly_sales;
````

**Answer:**

![image](https://user-images.githubusercontent.com/125182638/224203191-c4b4693d-471e-405e-80f3-48e8873e0ea7.png)

- Monday is used for each `week_date` value.

**2. What range of week numbers are missing from the dataset?**
- First, generate the full range of week numbers for the entire year from 1st week to 52nd week.
- Then, do a RIGHT OUTER JOIN of `clean_weekly_sales` with `#temp`.

````sql
CREATE TABLE #temp (num INT);

WITH nums AS (
  SELECT ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS num
  FROM master..spt_values
)
INSERT INTO #temp
SELECT num FROM nums 
WHERE num<=52

SELECT DISTINCT(b.num)
FROM clean_weekly_sales a
RIGHT OUTER JOIN #temp b
ON a.week_number=b.num
WHERE a.week_number is NULL; -- Filter for the missing week numbers whereby the values would be `null`
````

**Answer:**

_I'm posting only 5 rows here - ensure that you retrieved 28 rows!_

<img width="239" alt="image" src="https://user-images.githubusercontent.com/81607668/131644275-6a91200f-61fe-4b71-83d4-7b51945e4531.png">

- 28 `week_number`s are missing from the dataset.

**3. How many total transactions were there for each year in the dataset?**

````sql
SELECT 
  calendar_year, 
  SUM(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY calendar_year
ORDER BY calendar_year;
````

**Answer:**

![image](https://user-images.githubusercontent.com/125182638/224205563-3e8337c1-4917-44e3-87e9-3a25d2561fb7.png)

**4. What is the total sales for each region for each month?**

````sql
SELECT 
  region, 
  month_number, 
  SUM(CAST(sales as float)) AS total_sale
FROM clean_weekly_sales
GROUP BY region, month_number
ORDER BY region, month_number;
````

**Answer:**

_As there are 7 regions and results came up to 49 rows, I'm only showing solution for AFRICA and ASIA._

![image](https://user-images.githubusercontent.com/125182638/224205777-f99aeb13-79cb-4b77-bdc9-07a255e9f81d.png)

**5. What is the total count of transactions for each platform?**

````sql
SELECT 
  platform, 
  SUM(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY platform
ORDER BY platform;
````

**Answer:**

![image](https://user-images.githubusercontent.com/125182638/224206467-3ef2d9f9-819d-4e0f-8c02-d7853a5ec7a2.png)

**6. What is the percentage of sales for Retail vs Shopify for each month?**

````sql
WITH transactions_cte AS (
  SELECT 
    calendar_year, 
    month_number, 
    platform, 
    SUM(cast(sales as float)) AS monthly_sales
  FROM clean_weekly_sales
  GROUP BY calendar_year, month_number, platform
  )

SELECT 
  calendar_year, 
  month_number, 
  ROUND(100 * sum (CASE WHEN platform = 'Retail' THEN monthly_sales ELSE NULL END) / 
      SUM(monthly_sales),2) AS retail_percentage,
  ROUND(100 * sum (CASE WHEN platform = 'Shopify' THEN monthly_sales ELSE NULL END) / 
      SUM(monthly_sales),2) AS shopify_percentage
  FROM transactions_cte
  GROUP BY calendar_year, month_number
  ORDER BY calendar_year, month_number;
````

**Answer:**

_The results came up to 20 rows, so I'm only showing solution year 2018._

![image](https://user-images.githubusercontent.com/125182638/224211527-79746b16-9e9c-4e81-bd1c-4467ea7fa540.png)

**7. What is the percentage of sales by demographic for each year in the dataset?**

````sql
with sales_cte as(
select
demographic,
calendar_year,
sum(cast(sales as float)) as sales_by_year
from clean_weekly_sales
group by demographic, calendar_year)
select
calendar_year,
round(100*max(case when demographic = 'Couples' then sales_by_year else null end)/sum(sales_by_year),2) as sales_by_couples,
round(100*max(case when demographic = 'Families' then sales_by_year else null end)/sum(sales_by_year),2) as sales_by_families,
round(100*max(case when demographic = 'Unknown' then sales_by_year else null end)/sum(sales_by_year),2) as sales_unknonw
from sales_cte
group by  calendar_year
order by calendar_year;
````

**Answer:**

![image](https://user-images.githubusercontent.com/125182638/224211655-1d88b5b7-2706-44ab-adeb-508dc04d463c.png)

**8. Which age_band and demographic values contribute the most to Retail sales?**

````sql
with sales_retail_cte as
(
select age_band,
	   demographic,
	   sum(cast(sales as float)) as total_sales
from [dbo].[clean_weekly_sales]
where platform='Retail'
group by age_band,demographic
)
select
age_band,
demographic,
round(100* sum(total_sales)/(select sum(total_sales) from  sales_retail_cte),2)  as contribute_sales
from sales_retail_cte
group by age_band,demographic
order by contribute_sales desc;
````

**Answer:**

![image](https://user-images.githubusercontent.com/125182638/224211739-0bfe366e-1ab3-42f0-9448-c56e8c6d2c84.png)

The highest retail sales are contributed by unknown `age_band` and `demographic` at 42% followed by retired families at 16.73% and retired couples at 16.07%.

***
