# Pizza-Sales-MySQL-Project
Deep dive into pizza sales data using SQL queries to uncover sales trends, customer preferences, and business insights.

Will do some basic checking of data and solve below questions. You will find the SQL code below the mentioned questions.

Basic:
1) Retrieve the total number of orders placed.
2) Calculate the total revenue generated from pizza sales.
3) Identify the highest-priced pizza.
4) Identify the most common pizza size ordered.
5) List the top 5 most ordered pizza types along with their quantities.


Intermediate:
1) Join the necessary tables to find the total quantity of each pizza category ordered.
2) Determine the distribution of orders by hour of the day.
3) Group the orders by date and calculate the average number of pizzas ordered per day.
4) Determine the top 3 most ordered pizza types based on revenue.

Advanced:
1) Calculate the percentage contribution of each pizza type to total revenue.
2) Analyze the cumulative revenue generated over time.
3) Determine the top 3 most ordered pizza types based on revenue for each pizza category.

Let's get started

#creating database pizza_sale

create database pizza_sale;

#checking whether database created or not

show databases;

#using pizza_sale database

use pizza_sale;

#creating table orders, order_details since it has many rows

create table orders (
order_id int not null,
order_date date not null,
order_time time not null,
primary key(order_id) );

create table order_details (
order_details_id int not null,
order_id int not null,
pizza_id text not null,
quantity int not null,
primary key(order_details_id) );

#Basic data checking

#checking data in tables

select * from pizzas;

select * from pizza_types;

select * from orders;

select * from order_details;

#checking count of all tables

select count(*) from pizzas;

select count(*) from pizza_types;

select count(*) from orders;

select count(*) from order_details;

#checking if we have duplicates in pizza id

select pizza_id, count(*) as count from pizzas group by pizza_id having count(*)>1;

select pizza_type_id, count(*) as count from pizza_types group by pizza_type_id having count(*)>1;

select order_id, count(*) as count from orders group by order_id having count(*)>1;

select order_details_id, count(*) as count from order_details group by order_details_id having count(*)>1;

#no duplicates value found

#checking if data is missing or not

select * from pizzas where pizza_id is null
                      or pizza_type_id is null
                      or size is null
                      or price is null;

select * from pizza_types where pizza_type_id is null
                           or name is null
                           or category is null
                           or ingredients is null;
				
select * from orders where order_id is null
                     or order_date is null
                     or order_time is null;
                     
select * from order_details where order_details_id is null
                           or order_id is null
                           or pizza_id is null
                           or quantity is null;

#No missing data is present                      


#Basic questions

#1)Retrieve the total number of orders placed.

select count(order_id) from orders;

#2)Calculate the total revenue generated from pizza sales.

select round(sum(pizzas.price*order_details.quantity),2) as Total_revenue
from pizzas inner join order_details
on pizzas.pizza_id=order_details.pizza_id;

#3)Identify the highest-priced pizza.

select pizza_types.name, pizzas.price from pizza_types inner join pizzas
on pizza_types.pizza_type_id= pizzas.pizza_type_id order by pizzas.price desc limit 1;

#4)Identify the most common pizza size ordered.

select pizzas.size, count(order_details.order_details_id) as order_count from pizzas inner join order_details 
on pizzas.pizza_id=order_details.pizza_id
group by pizzas.size
order by order_count desc
limit 1;

#5)List the top 5 most ordered pizza types along with their quantities.

select pizza_types.name, sum(order_details.quantity) as quantity_of_pizza from pizza_types
inner join pizzas on pizza_types.pizza_type_id=pizzas.pizza_type_id
inner join order_details on pizzas.pizza_id=order_details.pizza_id
group by pizza_types.name
order by quantity_of_pizza desc
limit 5;

#Intermediate

#1)Join the necessary tables to find the total quantity of each pizza category ordered.

select pizza_types.category, sum(order_details.quantity) as quantity_of_pizza from pizza_types
inner join pizzas on pizza_types.pizza_type_id=pizzas.pizza_type_id
inner join order_details on pizzas.pizza_id=order_details.pizza_id
group by pizza_types.category
order by quantity_of_pizza desc;

#2)Determine the distribution of orders by hour of the day.

select hour(order_time) as hours, order_date, count(order_id) from orders
group by hour(order_time), order_date;

#3)Group the orders by date and calculate the average number of pizzas ordered per day.

select round(avg(quantity),0) as avg_no_of_pizzas_ordered_per_day from
( select orders.order_date, sum(order_details.quantity) as quantity from orders
inner join order_details
on orders.order_id=order_details.order_id
group by orders.order_date)  as order_quantity;


#4)Determine the top 3 most ordered pizza types based on revenue.

select pizza_types.name, round(sum(pizzas.price*order_details.quantity),2) as revenue from pizza_types
inner join pizzas
on pizza_types.pizza_type_id=pizzas.pizza_type_id
inner join order_details
on pizzas.pizza_id=order_details.pizza_id
group by pizza_types.name
order by revenue desc
limit 3;

#Advance

#1)Calculate the percentage contribution of each pizza type to total revenue.

select pizza_types.category, round(sum(pizzas.price*order_details.quantity)/ 
(select round(sum(pizzas.price*order_details.quantity),2) as revenue
from pizzas inner join order_details
on pizzas.pizza_id=order_details.pizza_id)*100,2) as total_revenue from pizza_types
inner join pizzas
on pizza_types.pizza_type_id=pizzas.pizza_type_id
inner join order_details
on pizzas.pizza_id=order_details.pizza_id
group by pizza_types.category
order by total_revenue desc
;

#2)Analyze the cumulative revenue generated over time.

select order_date, round(sum(revenue) over(order by order_date), 2) as cumulative_revenue from
(select orders.order_date, sum(pizzas.price*order_details.quantity) as revenue from order_details
inner join pizzas on order_details.pizza_id=pizzas.pizza_id
inner join orders on order_details.order_id=orders.order_id
group by orders.order_date) as sales_data ;

#3)Determine the top 3 most ordered pizza types based on revenue for each pizza category.

select category,name, round(revenue,2) from
( select category, name, revenue, rank() over(partition by category order by revenue desc) as rank_pizzaname from
(select pizza_types.category, pizza_types.name, sum(pizzas.price*order_details.quantity) as revenue from pizzas
inner join order_details on pizzas.pizza_id= order_details.pizza_id
inner join pizza_types on pizza_types.pizza_type_id=pizzas.pizza_type_id
group by pizza_types.category, pizza_types.name) as pizza_data) as pizza_final_data
where rank_pizzaname<=3 ;


Thankyou!

