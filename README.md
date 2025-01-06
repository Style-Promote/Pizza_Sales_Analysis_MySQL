# Pizza_Sales_Analysis_MySQL
Basic:
1.	Retrieve the total number of orders placed.
2.	Calculate the total revenue generated from pizza sales.
3.	Identify the highest-priced pizza.
4.	Identify the most common pizza size ordered.
5.	List the top 5 most ordered pizza types along with their quantities.

Intermediate:
1.	Join the necessary tables to find the total quantity of each pizza category ordered.
2.	Determine the distribution of orders by hour of the day.
3.	Join relevant tables to find the category-wise distribution of pizzas.
4.	Group the orders by date and calculate the average number of pizzas ordered per day.
5.	Determine the top 3 most ordered pizza types based on revenue.

Advanced:
1.	Calculate the percentage contribution of each pizza type to total revenue.
2.	Analyse the cumulative revenue generated over time.
3.	Determine the top 3 most ordered pizza types based on revenue for each pizza category.

NOTES
To beautify to the query, select & Ctrl + B.


















Basic:
1)	Retrieve the total number of orders placed.

select count(order_id) as Total_Orders from orders;

2)	Calculate the total revenue generated from pizza sales.
SELECT 
    ROUND(SUM(order_details.quantity * pizzas.price),
            2) AS Total_Sales
FROM
    order_details
        JOIN
    pizzas ON pizzas.pizza_id = order_details.pizza_id

3)	Identify the highest-priced pizza.
SELECT 
    pizza_types.name, pizzas.price
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;

4)	Identify the most common pizza size ordered.
SELECT 
    pizzas.size,
    COUNT(order_details.order_details_id) AS Order_Count
FROM
    pizzas
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size
ORDER BY Order_Count DESC
LIMIT 1;

5)	List the top 5 most ordered pizza types along with their quantities.
SELECT 
    pizza_types.name,
    SUM(order_details.quantity) AS Total_Quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY Total_Quantity DESC
LIMIT 5;
 
Intermediate:
6)	Join the necessary tables to find the total quantity of each pizza category ordered.
SELECT 
    pizza_types.category,
    SUM(order_details.quantity) AS Total_quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category;

7)	Determine the distribution of orders by hour of the day.
SELECT 
    hour(time) AS order_hour, COUNT(order_id) AS order_count
FROM
    orders
GROUP BY order_hour
ORDER BY order_hour;

8)	Join relevant tables to find the category-wise distribution of pizzas.
SELECT 
    pizza_types.category, COUNT(pizza_types.name) AS name_count
FROM
    pizza_types
GROUP BY pizza_types.category;

9)	Group the orders by date and calculate the average number of pizzas ordered per day.
SELECT 
    ROUND(AVG(quantity), 0)
FROM
    (SELECT 
        orders.date, SUM(order_details.quantity) AS quantity
    FROM
        orders
    JOIN order_details ON orders.order_id = order_details.order_id
    GROUP BY orders.date) AS order_quantity

10)	Determine the top 3 most ordered pizza types based on revenue.
SELECT 
    pizza_types.name,
    SUM(pizzas.price * order_details.quantity) AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 3;


11)	Calculate the percentage contribution of each pizza type to total revenue.
SELECT 
    pizza_types.category,
    (SUM(pizzas.price * order_details.quantity) / (SELECT 
            ROUND(SUM(order_details.quantity * pizzas.price),
                        2) AS Total_Sales
        FROM
            order_details
                JOIN
            pizzas ON pizzas.pizza_id = order_details.pizza_id)) * 100 AS contribution_in_percentage
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category;

12)	Analyse the cumulative revenue generated over time.
select date
    date,
    SUM(revenue) OVER (ORDER BY date) AS cum_revenue
from
(select orders.date, 
sum(pizzas.price*order_details.quantity) as revenue
from orders join order_details
on orders.order_id = order_details.order_id
join pizzas
on pizzas.pizza_id = order_details.pizza_id
group by orders.date) as sales;

13)	Determine the top 3 most ordered pizza types based on revenue for each pizza category.
SELECT name, revenue 
FROM (
    SELECT 
        pizza_types.category, 
        pizza_types.name,
        round(SUM(order_details.quantity * pizzas.price),0) AS revenue,
        RANK() OVER (PARTITION BY pizza_types.category ORDER BY SUM(order_details.quantity * pizzas.price) DESC) AS ranks
    FROM 
        pizza_types 
    JOIN 
        pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
    JOIN 
        order_details ON pizzas.pizza_id = order_details.pizza_id
    GROUP BY 
        pizza_types.category, 
        pizza_types.name
) AS niv
WHERE ranks <= 3;
