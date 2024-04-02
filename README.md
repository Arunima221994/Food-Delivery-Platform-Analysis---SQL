1.	How many customers are repeating the purchase within the 7 days-

with repeating_customer as
(select customer_id, order_date, count(1) as repeating_cutomer
from customer
where order_date>= current_date-interval '7days'
group by customer_id
having count(1)>1)

select count(cust_id)
from repeating_customer

2.	How many customers are repeating the same food category within 7 days or per week-

with last_sevendays as
(select customer_id, date(order_date) as date, category
from customer
where date>= current_date-interval '7days'
group by customer_id
order by date asc)
,
same_category as
(select  s1.*,s2.*
from last_sevendays s1
left join last_sevendays1 s2
on s1.customer_id=s2.customer_id
and s1.category=s2.category
and s2.date>s1.date
group by s1.customer_id) as t

select count(distinct customer_id)
from same_category

3. What are the top 3 category basis order value for last month

with cte as
(select customer_id, category, sales, date(order_date) as date
from customer) as t
group by category) 
,
cte1 as
(select category, sales,
extract(month from date-interval '1' month ) as previous_month
from cte
group by category)
,
total_sales as 
(select category, sum(sales) as total_sales
from cte1
group by category)
,
rank as
(select category, total_sales,
rank() over(partition by category order by total_sales desc) as rn
from total_sales
group by category)
,
select category, total_sales
from rank
where rn<=3
group by category



4.	What are the weekwise top 3 categories basis the ordering value for the last 4 weeks in a month

with order_date as 
(select customer_id, category, sales, date(order_date) as date
from customers
where datediff(month, order_date, getdate())=1) 
,
weekly as
(select *, extract ('week' from date) as week_number
from order_date
,
total_sales as
(select category, sum(sales) as total_sales, week_number
from weekly
group by category, week_number
order by week_number asc)
,
rank as
(select category,total_Sales,week_number,
rank() over(partition by category, week_number order by total_sales desc) as rn
from total_sales
group by)
,
select category, total_sales,week_number
from rank
where rn<=3
group by category, week_number
order by total_sales desc



5.	What is the rolling average,rolling sum, max rolling, min rolling order value for each week of last month- 

with order_date as
(select customer_id, category, sales, date(order_date) as date
from customers) as t
where datediff (month, order_date, getdate())=1) 
,
weekly as
(select *, extract ('week' from date) as week_number
from order_date)
,
total_sales as
(select sum(sales) as total_sales, week_number
from weekly
group by week_number
order by week_number asc)
,
rolling_avg as(
select  total_sales, week_number,
sum(total_sales) over(order by  week_number asc between 1 preceeding and 1 following) as rolling_sum,
Avg(total_sales) over(order by  week_number asc between 1 preceeding and 1 following) as rolling_avg,
max(total_sales) over(order by  week_number asc between 1 preceeding and 1 following) as rolling_max,
min(total_sales) over(order by  week_number asc between 1 preceeding and 1 following) as rolling_min
 from total_sales)


6.	 Which is the most ordered food for each city in 2023?

with total_sale as
(select category, sub_category as food, sum(sales) as total_sale, date(order_date) as sale_date
from customer
where datepart(year, sale_date)='2023'
group by category, sub_category)

select city, category, sub_category, total_sale,
rank() over (partition by category order bu total_sale desc) as rn
from total_sale
group by city, category

7.	Which category has the top sale value in Bangalore for last month at lunch time (12 pm to 3 pm)

with location as(
select category,location, order_value, 
date(order_date) as order_date, time(order_date) as time_zone
from customer
where location ='Bangalore' and datepart(month,order_date)=2
and datepart(hour,time_zone) between ‘12:00:00’ and ‘15:00:00’

select category, sum(order_value) as total_sales
from location
group by category
order by total_sales desc


   
8.	 No. of 5 star rating for a Delivery Executive in last 4 weeks consecutively in Bangalore location

with location as(
select DE_id,rating,
date(order_pickup_date) as pickup_date, date(order_delivered_date) as delivered_date
from DE
where Location ='Bangalore'
and order_pickup_time is not null 
and order_delivered_time is not null)
,
week_number as (
select DE_id, rating,
extract('week' from delivered_date) as week_number
from location
where datepart(month,pickup_date)=3 and datepart(month,delivered_date)=3
order by week_number asc)

select DE_id, avg(rating) as avg_rating
from week_number
group by DE_id,week_number
having avg(rating)>=5



9.	Fastest delivery by the Delivery executives in last 4 weeks

with location as(
select DE_id,rating,
date(order_pickup_time) as pickup_date, date(order_delivered_time) as delivered_date,
time(order_pickup_time) as pickup_time,time(order_delivered_time) as Delivered_time
from DE
where Location ='Bangalore'
and order_pickup_time is not null 
and order_delivered_time is not null)
,
week_number as
(select  DE_id,
pickup_time,Delivered_time,
datediff(minute,pickup_time,Delivered_time) as diff,
extract('week' from delivered_date) as week_number
from location
order by  week_number asc)
,
avg_time as
(select DE_id,
avg(diff) as avg_delivery_time
from week_number
group by DE_id, week_number
order by week_number asc, average_delivery_time desc
limit 1)



10.	Top 3 male Delivery executives based on the maximum number of experiences in Bangalore Location

ith info as(
select DE_name,
datediff(current_date(), joining_date) as exp_days 
from DE_info
where Gender='M')
,
cte as
(select DE_name, exp_days,
dense_rank() over (order by exp_days desc,DE_name asc) as rn
from info
group by De_name)

select DE_name
from cte
wherern<=4



11.	Find out the DE_id who have committed fraud

select distinct DE_id
from DE
where pickup_order_time is not null and delivered_order_time is null



12.	Top 3 areas contributing the highest in terms of orders in Bangalore

with city as
(select c.order_id, c.city, c.area_id, a.name
from customer c
join name a
on  c.area_id= a.area_id
and c.city=a.city)

select name, count(order_id) as total_order
from city
where city='Bangalore'
group by name
order by order_count desc

