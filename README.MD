# Assignment 1: Sunrise Supermarket Database
- Name: Shyaka Ntaganda Yannick
- ID: 29834
- Database used: Oracle Database 21c

## Scenario 

Sunrise Supermarket is a retail store that sells a variety of products such as groceries. Customers visit the supermarket to purchase products and place orders containing one or more items. The supermarket serves customers from different locations and records their purchases and sales.


### Customer's order
>Listing every order with the customer's name, city, and order date.

```sql

select o.order_id,
       c.customer_name as name,
       c.city,
       o.order_date
  from orders o
 inner join customers c
on o.customer_id = c.customer_id;
```
![Customer Order](screenshot\question1.png)


### Order Item 
>Listing every order item with product name, category, price, and quantity.

```sql 
select oi.order_item_id as id,
       p.product_name as product,
       p.category,
       p.price,
       oi.quantity
  from order_items oi
 inner join products p
on oi.product_id = p.product_id;
```
![order item information](screenshot\question2.png)


### Customer With Orders
>Listing all customers and their orders where they exist, including customers with no orders.

```sql
select c.customer_id,
       c.customer_name as customer,
       c.city,
       o.order_id,
       o.order_date
  from customers c
  left join orders o
on c.customer_id = o.customer_id
 order by c.customer_id,
          o.order_date;
```
![Numbering in order](screenshot\question3.png)
### Spend Above average 
>Customer total spend (quantity × price) and showing customers above average spend. Using CTE.

```sql 
WITH customer_totals AS (
    SELECT
        o.customer_id,
        SUM(oi.quantity * p.price) AS total_spend
    FROM orders o
    JOIN order_items oi
        ON o.order_id = oi.order_id
    JOIN products p
        ON oi.product_id = p.product_id
    GROUP BY o.customer_id
)
SELECT
    c.customer_id,
    c.customer_name,
    ct.total_spend
FROM customer_totals ct
JOIN customers c
    ON c.customer_id = ct.customer_id
WHERE ct.total_spend > (
    SELECT AVG(total_spend)
    FROM customer_totals
)
ORDER BY ct.total_spend DESC;
```
![Spend above average](screenshot\question4.png)


### Ranking By Spend
> Rank customers by total amount spent, highest first.

```sql

with customer_totals as (
   select o.customer_id,
          sum(oi.quantity * p.price) as total_spend
     from orders o
     join order_items oi
   on o.order_id = oi.order_id
     join products p
   on oi.product_id = p.product_id
    group by o.customer_id
)
select c.customer_name,
       ct.total_spend,
       rank()
       over(
           order by ct.total_spend desc
       ) as spending_rank
  from customer_totals ct
  join customers c
on c.customer_id = ct.customer_id;
```

![Ranking By Spend](screenshot\question5.png)


### Customer Order Number 
> Number each customer's orders in the order placed.

```sql

select o.customer_id,
       c.customer_name as customer,
       o.order_id,
       o.order_date,
       row_number()
       over(partition by o.customer_id
            order by o.order_date
       ) as order_number
  from orders o
  join customers c
on o.customer_id = c.customer_id
 order by o.customer_id,
          o.order_date;
```

![Order Numbers](screenshot\question6.png)

### Revenue
>Showing a running total of revenue over time, ordered by order date.

```sql 
with order_totals as (
   select o.order_id,
          o.order_date,
          sum(oi.quantity * p.price) as order_total
     from orders o
     join order_items oi
   on o.order_id = oi.order_id
     join products p
   on oi.product_id = p.product_id
    group by o.order_id,
             o.order_date
)
select order_id,
       order_date,
       order_total,
       sum(order_total)
       over(
           order by order_date,
                    order_id
       ) as running_revenue
  from order_totals
 order by order_date,
          order_id;
```

![Revenue](screenshot\question7.png)

### Days Between orders
>For each customer with more than one order, show days between the current and previous order.
```sql
SELECT
    customer_id,
    order_id,
    order_date,
    previous_order_date,
    order_date - previous_order_date AS days_between_orders
FROM (
    SELECT
        customer_id,
        order_id,
        order_date,
        LAG(order_date) OVER (
            PARTITION BY customer_id
            ORDER BY order_date
        ) AS previous_order_date
    FROM orders
) x
WHERE previous_order_date IS NOT NULL
ORDER BY customer_id, order_date;
 ```
![Days Between](screenshot\question8.png)