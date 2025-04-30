# Pizza_sales_Analysis-SQL-
An end-to-end SQL project analyzing pizza sales data to uncover actionable insights such as top-selling categories, peak sales times, and customer purchasing patterns.

### Basic:
1. **Retrieve the total number of orders placed.**
```sql
SELECT 
    COUNT(order_id) AS Total_orders
FROM
    Orders;
```

2. **Calculate the total revenue generated from pizza sales.**
```sql
SELECT 
    ROUND(SUM(Od.quantity * p.price), 2) AS Total_Sales
FROM
    order_details od
        JOIN
    pizzas p ON od.Pizza_id = p.pizza_id;
```

3. **Identify the highest-priced pizza.**
```sql
SELECT 
    pt.name, p.price
FROM
    pizza_types pt
        JOIN
    Pizzas p ON pt.pizza_type_id = p.pizza_type_id
ORDER BY p.price DESC
LIMIT 1;
```

4. **Identify the most common pizza size ordered.**
```sql
SELECT 
    p.size, COUNT(od.order_details_id) AS Order_cnt
FROM
    pizzas p
        JOIN
    order_details od ON p.pizza_id = od.Pizza_id
GROUP BY p.size
ORDER BY order_cnt DESC;
```

5. **List the top 5 most ordered pizza types along with their quantities.**
```sql
SELECT 
    pt.name, SUM(od.quantity) AS Quantity
FROM
    pizza_types pt
        JOIN
    pizzas p ON pt.pizza_type_id = p.pizza_type_id
        JOIN
    order_details od ON p.pizza_id = od.Pizza_id
GROUP BY pt.name
ORDER BY Quantity DESC
LIMIT 5;
```

### Intermediate:
6. **Join the necessary tables to find the total quantity of each pizza category ordered.**
```sql
SELECT 
    pt.category, SUM(od.quantity) AS Quantity
FROM
    pizza_types pt
        JOIN
    pizzas p ON pt.pizza_type_id = p.pizza_type_id
        JOIN
    order_details od ON p.pizza_id = od.Pizza_id
GROUP BY pt.category
ORDER BY Quantity DESC;
```

7. **Determine the distribution of orders by hour of the day.**
```sql
SELECT 
    HOUR(order_time) AS Hour, COUNT(order_id) AS Order_cnt
FROM
    orders
GROUP BY HOUR(order_time);
```

8. **Join relevant tables to find the category-wise distribution of pizzas.**
```sql
SELECT 
    category, COUNT(name) AS Total_distribution
FROM
    pizza_types
GROUP BY category;
```

9. **Group the orders by date and calculate the average number of pizzas ordered per day.**
```sql
SELECT 
    ROUND(AVG(Quantity), 0) AS Avg_pizza_ordered_per_day
FROM
    (SELECT 
        o.order_date, SUM(od.quantity) AS quantity
    FROM
        orders o
    JOIN order_details od ON o.Order_id = od.Order_id
    GROUP BY o.Order_date) AS order_quantity;
```

10. **Determine the top 3 most ordered pizza types based on revenue.**
```sql
SELECT 
    Pt.name, SUM(Od.quantity * p.price) AS Revenue
FROM
    pizza_types pt
        JOIN
    pizzas p ON pt.pizza_type_id = p.pizza_type_id 
		JOIN
	order_details od ON p.Pizza_id= Od.Pizza_id
GROUP BY Pt.name
ORDER BY Revenue DESC
LIMIT 3;
```

### Advanced:
11. **Calculate the percentage contribution of each pizza type to total revenue.**
```sql
SELECT 
    pt.category,
    ROUND(SUM(od.quantity * p.price) / (SELECT 
                    ROUND(SUM(od.quantity * p.price), 2) AS total_sales
                FROM
                    order_details od
                        JOIN
                    pizzas p ON od.pizza_id = p.pizza_id) * 100,
            2) AS revenue
FROM
    pizza_types pt
        JOIN
    pizzas p ON pt.pizza_type_id = p.pizza_type_id
        JOIN
    order_details od ON od.pizza_id = p.pizza_id
GROUP BY pt.category
ORDER BY revenue DESC;
```

12. **Analyze the cumulative revenue generated over time.**
```sql
SELECT 
    order_date, ROUND(SUM(revenue), 2) AS cum_revenue
FROM
    (SELECT 
        o.order_date, SUM(od.quantity * p.price) AS revenue
    FROM
        order_details od
    JOIN pizzas p ON od.pizza_id = p.pizza_id
    JOIN orders o ON o.order_id = od.order_id
    GROUP BY o.order_date) AS sales
GROUP BY order_date;
```

13. **Determine the top 3 most ordered pizza types based on revenue for each pizza category.**
```sql
SELECT 
    category,
    name,
    revenue
FROM (
    SELECT 
        pt.category,
        pt.name,
        SUM(od.quantity * p.price) AS revenue,
        RANK() OVER (PARTITION BY pt.category ORDER BY SUM(od.quantity * p.price) DESC) AS rn
    FROM 
        pizza_types pt
    JOIN 
        pizzas p ON pt.pizza_type_id = p.pizza_type_id
    JOIN 
        order_details od ON p.pizza_id = od.pizza_id
    GROUP BY 
        pt.category, pt.name
) AS ranked_pizzas
WHERE rn <= 3;
```
