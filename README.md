# Bikestore Sales Dataset Analysis 

```sql
Tools being used for the project: SQL & Excel
```
## Overview
I will be conducting an in-depth data analysis project on the Bikestore sales dataset. Using SQL, I will clean, explore, and analyze the data to extract meaningful insights. Key findings that could benefit the bike store owner will be visualized using Excel charts. This project is approached as if I am presenting the results to the store owner or a senior-level executive, providing them with a clear overview of the store's performance.

## Collecting the dataset 
The dataset that i will be using for this dataset will be attatched in a seperate folder in this repository. 

## Understanding the dataset 
This is the overall entity relationship diagram for all of the different tables. 
![Screenshot 2025-02-26 085733](https://github.com/user-attachments/assets/84e3127e-27d0-43fe-ac7b-a4875aff6712)
As this is a bike store sales dataset, there are several interesting aspects we can explore. However, to gain a comprehensive understanding of the performance of specific bike brands and categories, we would need to perform multiple join statements.

## Questions that i will be exploring 
- What is the **total revenue** generated by the bike store?
- Which product has the **highest sales revenue**?
- **How many orders** have been placed in total?
- Which store has processed the **most orders**?
- What is the **total number of bikes sold per brand**?
- Which customers have placed more than X orders and spent more than $Y in total?
- What are the **top 5 best-selling products in each category**?
- **How does sales revenue trend over time** (daily, monthly, yearly)?
- Which products are **frequently bought together**?
- What is the **average time between order placement and shipment**?
- **Which store has the highest revenue per staff member**?
- What is **the customer retention rate** (percentage of repeat customers)?
- Which products are **underperforming despite having high stock levels**?
- What is the **revenue share of each brand relative to total revenue**?
- Which staff members consistently **achieve higher-than-average sales**?

## Queries
**What is the total revenue generated by the bike store?**: 
```sql
with cte as 
(select s.store_id, s.store_name, o.order_id, oi.quantity, oi.list_price, oi.discount
from stores s
join orders o 
on s.store_id = o.store_id
join order_items oi
on o.order_id = oi.order_id)
select 
	store_name,  
	sum((Quantity * list_price) - (list_price * Discount)) AS Total_Revenue
from cte
group by store_name
order by Total_Revenue desc;
```
![Screenshot 2025-02-26 115041](https://github.com/user-attachments/assets/3d9879ed-67cb-479b-84a1-dabed4c4e34a)

Which product has the highest sales revenue?
```sql
select top 5 p.product_name, 
	sum((o.quantity * o.list_price) - (o.list_price * o.discount)) as Total_Revenue
from order_items o
join products p
on o.product_id = p.product_id 
group by p.product_name
order by Total_Revenue desc;
```
![Screenshot 2025-02-26 115137](https://github.com/user-attachments/assets/7bbc679b-b259-42f8-8fd6-85399d1d0d68)

How many orders have been placed in total?
```sql
select sum(quantity) as Total_Orders_Placed
from order_items
```
![Screenshot 2025-02-26 115219](https://github.com/user-attachments/assets/fdc9ecf9-6809-4293-a5af-ba7685147942)

Which store has processed the most orders?
```sql
select s.store_name, sum(oi.quantity) as Total_Order_Quantity
from stores s
join orders o 
on s.store_id = o.store_id
join order_items oi
on o.order_id = oi.order_id
group by s.store_name
order by Total_Order_Quantity desc;
```
![Screenshot 2025-02-26 115240](https://github.com/user-attachments/assets/3f5af6f4-39fe-47fc-a5a7-3249cbea6914)

What is the total number of bikes sold per brand?
```sql
select b.brand_name, sum((o.quantity * o.list_price) - (o.list_price * o.discount)) as Total_Revenue
from order_items o
join products p
on o.product_id = p.product_id
join brands b
on p.brand_id = b.brand_id
group by b.brand_name
order by Total_Revenue
```
![Screenshot 2025-02-26 115307](https://github.com/user-attachments/assets/87e45ae7-788a-433d-8cd0-cc266e4fd9dd)

Which customers have spent more than 10000 in total?
```sql
select CONCAT(c.first_name, ' ', c.last_name) as Full_Name, sum ((i.quantity * i.list_price) - (i.quantity * i.discount)) as Total_Revenue
from orders o 
join customers c
on o.customer_id = c.customer_id
join order_items i
on i.order_id = o.order_id
group by CONCAT(c.first_name, ' ', c.last_name)
having sum ((i.quantity * i.list_price) - (i.quantity * i.discount)) > '10000'
```
![Screenshot 2025-02-26 115349](https://github.com/user-attachments/assets/e0bd3443-0b8f-43a6-bf85-8f3fd1334a0f)

What are the top 2 best-selling products in each category?
```sql
select c.category_name, 
	p.product_name, 
	sum((o.quantity * o.list_price) - (o.list_price * o.discount)) as Total_Revenue , 
	RANK() over (partition by p.product_name order by sum((o.quantity * o.list_price) - (o.list_price * o.discount)) desc) as ranks
from order_items o
join products p
on o.product_id = p.product_id
join categories c
on p.category_id = c.category_id
group by c.category_name, p.product_name
```
![Screenshot 2025-02-26 115423](https://github.com/user-attachments/assets/83828ef6-e1ab-41f2-966e-10f2cb6ce911)

How does sales revenue trend over time (monthly, yearly)?
```sql
with cte as (select i.order_id, i.quantity, i.list_price, i.discount, o.order_date
from order_items i
join orders o
on i.order_id = o.order_id
)
select FORMAT(order_date, 'MMMM') as months, sum((quantity * list_price) - (list_price * discount)) as Total_Revenue
from cte 
group by FORMAT(order_date, 'MMMM') -- monthly revenue
![Screenshot 2025-02-26 115532](https://github.com/user-attachments/assets/1b7587a1-81cb-4302-89d3-b95cebf822ad)

with cte as (select i.order_id, i.quantity, i.list_price, i.discount, o.order_date
from order_items i
join orders o
on i.order_id = o.order_id
)
select FORMAT(order_date, 'yyyy') as years, sum((quantity * list_price) - (list_price * discount)) as Total_Revenue
from cte 
group by FORMAT(order_date, 'yyyy') -- yearly revenue
```
![Screenshot 2025-02-26 115606](https://github.com/user-attachments/assets/fefb879c-8c64-4fe7-aae6-ce139f8383d1)

What is the average time between order placement and shipment in orders by city? (i also want to compare the top perfoming store with the average shipment time to see if better shipment correlates with more revenue)
```sql
with cte as (select c.city, datediff(dd, o.order_date, o.shipped_date) shipment_time
from orders o
join customers c
on o.customer_id = c.customer_id)
select city, AVG(shipment_time) as avg_shipment_time
from cte
group by city
order by AVG(shipment_time) -- This query is showing the average shipment time based on the city 
 
with cte as (select s.store_name, datediff(dd, o.order_date, o.shipped_date) shipment_time
from orders o
join stores s
on o.store_id = s.store_id
)
select store_name, avg(shipment_time) avg_shipment_time
from cte 
group by store_name
order by avg_shipment_time desc -- This query is showing the average shipping time based on the store

select s.store_name, sum((i.quantity * i.list_price) - (i.list_price * i.discount)) as Total_Revenue
from order_items i
join orders o
on i.order_id = o.order_id
join stores s
on o.store_id = s.store_id
group by s.store_name
order by Total_Revenue desc -- This query shows the revenue based on the store we can see that shipment time doesnt have has that much of an impact on the total revenue by store since santa cruiz ships on average in 2 days however they are ranked the second best performing store
```
![Screenshot 2025-02-26 115719](https://github.com/user-attachments/assets/9635e298-70d9-482c-afcb-2ac51db3d71f)
![Screenshot 2025-02-26 115700](https://github.com/user-attachments/assets/61966762-dda5-4384-b8c4-86edc56f837f)

Which store has the highest performing staff member?
```sql
select o.staff_id, CONCAT(s.first_name, ' ', s.last_name) as Full_name, sum((i.quantity * i.list_price) - (i.list_price * i.discount)) as Total_Revenue
from orders o
join order_items i
on o.order_id = i.order_id
join staffs s
on o.staff_id = s.staff_id
group by o.staff_id, CONCAT(s.first_name, ' ', s.last_name)
order by Total_Revenue desc
```
![Screenshot 2025-02-26 115752](https://github.com/user-attachments/assets/6e241e5a-d164-4433-90ff-0081c9c1fbcb)

Which products are underperforming despite having high stock levels?
```sql
with cte as (select o.order_id, s.product_id, s.quantity, os.order_date
from order_items o
join stocks s
on o.product_id = s.product_id 
join orders os
on os.order_id = o.order_id)
select top 3 product_id, quantity, count(product_id) as order_amount
from cte
group by product_id, quantity
order by quantity desc, order_amount asc
```
![Screenshot 2025-02-26 115819](https://github.com/user-attachments/assets/330bf15f-2394-4de3-b1c9-766420e57833)

## Conclusion
I have attached the queries I used, along with the dataset sources and screenshots of the query outputs. There are some interesting findings from this project. The way I would present this to a senior-level member of the organization would vary depending on their role—whether it's a senior marketer, a board member, or another stakeholder. However, I believe the outputs I have developed can be effectively presented to any upper-level stakeholder.
