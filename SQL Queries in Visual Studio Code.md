select * from pizzas
select * from pizza_types
select * from customers
select * from order_details
select * from orders


1. To check for duplicates

--1 way

delete from customers
where customers not in (
	select min(order_id)
	from orders
	group by custid
)

--2 way


delete from customers
where custid in (
		select custid
		from (
			select custid, email, row_number() over (partition by email order by custid) as rn
			from customers
			 ) t
where t.rn > 1
)


2. Check for null Values
select * from customers where phone = null


3. Treating null Values

update customers set phone = 0 where phone is null

update customers set first_name = ' - ' where phone is null


4. Handling Negative Values

update customers set quantity = 0 where quantity is null


5. Handling Inconsistent and Invalid Date Format

select * from orders where order_date is null

select order_id, order_date
from orders
where order_date !~ '^(19|20)\d\d-(0[1-9]|1[0-2])-(0[1-9]|12(0-9)|3[0-1])$'

















### 1. Orders Volume Analysis
- Total unique orders, orders by month, day-of-week analysis, repeat customers, average orders per customer, cumulative order trend.

select * from orders;


with monthly_orders as (
		select date_trunc('month',order_date) as month,
		count(order_id) as order_count
		from orders 
		group by date_trunc('month',order_date)
)
select month,
		order_count,
		lag(order_count) over (order by month) as prev_month,
		(order_count - lag(order_count) over (order by month)) / nullif(lag(order_count) over(order by month,0),2) as mom_growth_pct
		from monthly_orders
order by month;


--Cumlative order trend


select order_date,
		count(order_id) as daily_orders,
		sum(count(order_id)) over (order by order_date) as cumlative_orders
		from orders
group by order_date
order by order_date


### 2. Total Revenue from Pizza Sales
- Calculate total revenue from all pizza sales.

select sum(od.quantity * p.price) as total_revenue
from order_details od
join pizzas p on p.pizza_id = od.pizza_id

### 3. Highest-Priced Pizza
- Identify the most expensive pizza on the menu.

select pt.name, p.size, concat('$ ',p.price) as price
from pizzas p
join pizza_types pt on pt.pizza_type_id = p.pizza_type_id
order by p.price desc;

### 4. Most Common Pizza Size Ordered
- Determine the most frequently ordered pizza size.

select p.size, count(1) as total_orders
from order_details od 
join pizzas p on od.pizza_id = p.pizza_id
join pizza_types pt on p.pizza_type_id = pt.pizza_type_id
group by p.size 
order by total_orders desc

### 5. Top 5 Most Ordered Pizza Types
- Find the top 5 pizza types based on quantity sold.

select p.pizza_id, sum(od.quantity) as total_qty
from order_details od
join pizzas p on od.pizza_id = p.pizza_id
join pizza_types pt on pt.pizza_type_id = p.pizza_type_id
group by p.pizza_id
order by total_qty desc
limit 5

### 6. Total Quantity by Pizza Category
- Calculate total pizzas sold in each category.

select pt.category, sum(od.quantity) as total_qty
from order_details od
join pizzas p on p.pizza_id = od.pizza_id
join pizza_types pt on pt.pizza_type_id = p.pizza_type_id
group by pt.category
order by total_qty

### 7. Orders by Hour of the Day
- Understand peak ordering hours to optimize staffing.

select to_char(order_time::time,'HH24:00') as order_hour,
count(*) as order_count
from orders
group by order_hour
order by order_hour

### 8. Category-Wise Pizza Distribution
- Analyze category-wise sales and percentage share.

### 9. Average Pizzas Ordered per Day
- Measure daily pizza demand consistency.

select round(avg(daily_total),2) as avg_pizza_per_day
from (
		select o.order_date, sum(od.quantity) as daily_total
		from orders o
		join order_details od on o.order_id = od.order_id
		group by o.order_date
)

### 10. Top 3 Pizzas by Revenue
- Identify pizzas generating the highest revenue.

with pizza_revenue as(
		select pt.name, sum(od.quantity * p.price) as revenue,
		rank() over(order by sum(od.quantity * p.price) desc) as rnk
		from order_details od
		join pizzas p on p.pizza_id = od.pizza_id
		join pizza_types pt on pt.pizza_type_id = p.pizza_type_id
		group by pt.name
)
select name, revenue, rnk
from pizza_revenue
where rnk <=3

### 11. Revenue Contribution per Pizza
- Percentage contribution of each pizza to total revenue.

select pt.name, sum(od.quantity * p.price) as revenue,
		concat(round(100.0 * sum(od.quantity * p.price) / 
		sum(sum(od.quantity * p.price)) over() ,2),'%') as pct_contribution
		from order_details od
		join pizzas p on od.pizza_id = p.pizza_id
		join pizza_types pt on pt.pizza_type_id = p.pizza_type_id
		group by pt.name
		order by pct_contribution desc

### 12. Cumulative Revenue Over Time
- Monthly cumulative revenue trend since launch.

select order_date, sum(daily_revenue) over (order by order_date) as cumulative_revenue
from (
		select o.order_date, sum(od.quantity * p.price) as daily_revenue
		from orders o
		join order_details od on o.order_id = od.order_id
		join pizzas p on od.pizza_id = p.pizza_id
		group by order_date
	 )t

### 13. Top 3 Pizzas by Category (Revenue-Based)
- Top 3 pizzas by revenue in each category.

with pizza_revenue as(
		select pt.name, sum(od.quantity * p.price) as revenue,
		pt.category, rank() over(partition by pt.category order by sum(od.quantity * p.price) desc) as rnk
		from order_details od
		join pizzas p on p.pizza_id = od.pizza_id
		join pizza_types pt on pt.pizza_type_id = p.pizza_type_id
		group by pt.name, pt.category
)
select name, revenue, category, rnk
from pizza_revenue
where rnk<= 3

### 14. Top 10 Customers by Spending
- Identify the highest-spending customers.

select c.custid, concat(c.first_name,' ',c.last_name) as name,             
sum(od.quantity * p.price) as total_spent
from customers c
join orders o on c.custid = o.custid
join order_details od on od.order_id = o.order_id
join pizzas p on p.pizza_id = od.pizza_id
group by c.custid, name
order by total_spent desc
limit 10

### 15. Orders by Weekday
- Determine busiest days of the week for orders.

### 16. Average Order Size
- Calculate average number of pizzas per order.

select round(avg(order_size),2) as avg_order_size
from (
		select od.order_id, sum(od.quantity) as order_size
		from order_details od
		group by od.order_id
		order by order_size desc
)t

### 17. Seasonal Trends
- Analyze sales patterns by month and holidays.

select extract(month from order_date) as month,
count(*) as total_orders
from orders o
group by extract(month from order_date)
order by month

### 18. Revenue by Pizza Size
- Revenue contribution of each pizza size (S, M, L, XL, XXL).

### 19. Customer Segmentation
- Classify customers as High Value or Regular based on spend.

with cust_spent as (
		select c.custid, sum(od.quantity * p.price) as total_spent
		from customers c
		join orders o on o.custid = c.custid
		join order_details od on od.order_id = o.order_id
		join pizzas p on p.pizza_id = od.pizza_id
		group by c.custid
)
select case when total_spent > 50000 then 'High Value' else 'Regular' end as segment,
count(*) as customer_count
from cust_spent
group by segment

### 20. Repeat Customer Rate
- Percentage of repeat customers versus one-time buyers.

with cust_orders as (
		select custid, count(distinct order_id) as order_count
		from orders
		group by order_id
)
select round(100.0 * sum(case when order_count > 100 then 1 else 0 end) / count(*),2) as repeat_rate
from cust_orders