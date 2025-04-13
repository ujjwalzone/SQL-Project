# SQL-Project

Below image is a cohort analysis done via PowerBI using 100% stacked column chart.
![image](https://github.com/user-attachments/assets/8920c680-8c33-4383-a959-89d3f79f28be)

But before everything, we need to run SQL queries using SSMS and connect to local SQL Server.

Firstly create using client revenue table using below query

create view client_revenue as 
select  
cast(sum((s.OrderQuantity*p.productprice) - (s.OrderQuantity*p.ProductCost)) as decimal(19,2)) as 'NetRevenue',
productname,
p.ProductKey,
s.CustomerKey,
s.OrderDate,
p.ModelName
from sales as s
left join products as p 
on s.productkey = p.productkey
GROUP BY productname,s.CustomerKey,s.OrderDate,
p.ModelName,p.ProductKey;

and create cohort year table for our visualisation on Power BI

drop table if exists cohort_analysis;
with cohort_year as (
select  
distinct 
year(min(orderdate) over(partition by customerkey)) as cohort_year ,
customerkey
from sales 
)
select 
cy.cohort_year,
year(cr.orderdate) as Purchase_Year,
sum(cr.netrevenue) as totalrevenue
into cohort_analysis
from client_revenue as cr
left join cohort_year as cy 
on cr.CustomerKey = cy.CustomerKey
group by year(cr.orderdate), cy.cohort_year

Once the table is created, connect your power BI to this particular SQL server and import the table cohort analysis

Create a customer segmentation to get the visualisation from powerBi using following query
![image](https://github.com/user-attachments/assets/652f3998-7932-470e-9563-0f1327560498)

create view customer_segment as 
with get_customer_revenue as (
select 
ModelName,
CustomerKey,
sum(netrevenue) as netrevenue -- aggregate function (sum,count,min,max)
from client_revenue
group by ModelName,
CustomerKey
), customer_segment as (
-- 25% data nikal..kasari? -- ascending order netrevenue and group garne modelname le
select 
PERCENTILE_CONT(0.25) within group (order by netrevenue) over(partition by Modelname) as '25_percentile',
PERCENTILE_CONT(0.75) within group (order by netrevenue) over(partition by Modelname) as '75_percentile',
*
from get_customer_revenue
), segment_summary as (
select 
case when netrevenue < [25_percentile] then '1 - Low Value Client'
when netrevenue <= [75_percentile] then '2 - Mid Value Client'
else '3- High-value' end as customer_segment,
*
from 
customer_segment
)
select customer_segment,sum(netrevenue) as  netrevenue
from segment_summary
group by customer_segment

