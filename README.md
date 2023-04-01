# Danny-s-Diner
```
CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
 

CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
```
  
# SOLUTIONS
  
--1. What is the total amount each customer spent at the restaurant?
 ```
  select s.customer_id, Sum(m.price)
  from menu m
  join sales s
  on s.product_id = m.product_id
  group by s.customer_id
```
  -- 2. How many days has each customer visited the restaurant?
```
select customer_id, Count(distinct order_date) -- Distinct is used becoz one customer might order two different product/day)
  from sales s
  group by customer_id
```

  --3. What was the first item from the menu purchased by each customer?
 ```
  select count(*), product_id
  from sales
  group by customer_id
 ```
 -- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
  ```
  select top 1 count(order_date) as selling_sount,s.product_id, m.product_name
  from sales s
  join menu m
  on s.product_id = m.product_id
  group by s.product_id,m.product_name
  order by 1 desc
 -- Ramen was sold 8 times.
```
 --5.Which item was the most popular for each customer?
```
with cte as
(select count(*) as num, customer_id, product_name,
DENSE_RANK() over (partition by s.customer_id order by count(*) desc) as dnr
from sales s
join menu m
on s.product_id = m.product_id
group by s.customer_id, product_name)
select num,customer_id,product_name
from cte 
where dnr = 1
```

--6.Which item was purchased first by the customer after they became a member?
```
with cte as
(select product_name, customer_id,
row_number() over (partition by customer_id order by customer_id) as rn
from sales
join menu 
on menu.product_id =	sales.product_id
where order_date >= '2021-01-07'
or order_date >= '2021-01-09')
select cte.product_name, mm.customer_id
from cte
join members mm
on mm.customer_id = cte.customer_id
where rn = 1
-- curry - A 
-- Sushi - B
```

--7.Which item was purchased just before the customer became a member?

```
with cte as
(select product_name, s.customer_id, s.order_date,
dense_rank() over (partition by s.customer_id order by s.order_date) as dnr
from menu m
join sales s
on s.product_id = m.product_id
join members mem
on s.customer_id = mem.customer_id
where order_date < mem.join_date)
select product_name, customer_id
from cte
where dnr = 1
```
-- 8. What is the total items and amount spent for each member before they became a member?
```
select SUM(m.price) as amount_spent, s.customer_id, count(s.product_id) as quantity
from menu m
join sales s
on s.product_id = m.product_id
join members mem
on mem.customer_id = s.customer_id
where order_date < mem.join_date
group by s.customer_id
```
--9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```
with cte as
(select s.customer_id, 
case when m.product_id = 1 then m.price*20 else m.price* 10 end as points 
from sales s
join menu m
on m.product_id = s.product_id)
select cte.customer_id, Sum(cte.points) as total_points
from cte
group by cte.customer_id
```
--10.The first week after a customer joins the program (including their join date) they earn 2x points on all items, 
-- not just sushi- how many points do customer A and B have at the end of January?
```
select s.customer_id, 
SUM(case when order_date between '2021-01-07' and '2021-01-15' then m.price*20
	     when m.product_id = 1 then m.price * 20
	     else m.price* 10 end) as points 
from sales s
join menu m
on m.product_id = s.product_id
join members mem
on mem.customer_id = s.customer_id
--where order_date <= '2021-01-31'
--or order_date > '2021-01-06'
group by s.customer_id
```
--BONUS
-- 1.
```
select s.customer_id, s.order_date, m.product_name,m.price,
case when s.order_date >= mem.join_date then 'YES'
else 'NO' end as memb
from sales s
join menu m
on m.product_id = s.product_id
left join members mem
on mem.customer_id = s.customer_id
```
--2.
```
with cte as
(select s.customer_id, s.order_date, m.product_name,m.price,
case when s.order_date >= mem.join_date then 'YES'
else 'NO' end as member
from sales s
left join menu m
on m.product_id = s.product_id
left join members mem
on mem.customer_id = s.customer_id)
select cte.*,
case when member = 'NO' then Null
else RANK() over (partition by customer_id,member
                 order by order_date) end as ranking
from cte 
```
