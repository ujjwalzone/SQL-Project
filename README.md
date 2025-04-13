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
