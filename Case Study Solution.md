# <h1 align="center" > 🍜 Danny's Diner 🍜 
 
<p align="center">
<kbd>  <img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" alt="Image" width="450" height="450"></kbd>

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

:ramen: :curry: :sushi:
If you have any questions, reach out to me on [LinkedIn](https://www.linkedin.com/in/pavithran93/).

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-1/). 


***

## 🔎Business Task

### 🏨Introduction

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: **sushi, curry and ramen**

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

### 🤷‍♂️Problem Statement

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

- sales
- menu
- members
  
You can inspect the entity relationship diagram data below.

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. 

***

## 🪢Entity Relationship Diagram

<kbd>![ER Diagram.png](https://github.com/sivapavithran93/Danny-s-Diner-Case-Study-Using-SQL/blob/main/Images/ER%20Diagram.png)
</kbd>

***

## 🤔Question and Solution 

🔖 Creating a view for the whole data make it easier to visualize the data and explore to an extent. The advantages of creating views in SQL in a concise manner:

- Simplification: Views simplify complex queries, making it easier for developers to access and manipulate data.

- Security: Views enhance data security by controlling who can access specific data, without affecting underlying tables.

- Performance: Views can improve query performance by storing results and optimizing SQL execution plans.

- Maintenance: Views ease database maintenance by isolating changes and providing code reusability.

```sql
CREATE VIEW diner_res AS(
    SELECT 
        *
    FROM
        sales s
            JOIN
        menu m USING (product_id)
            LEFT JOIN
        members ms USING (customer_id)
);

SELECT 
    *
FROM
    diner_res;

```

### Output:
<kbd>![Joined Output.png](https://github.com/sivapavithran93/Danny-s-Diner-Case-Study-Using-SQL/blob/main/Images/Joined%20Output.png)</kbd>

***

**Q-1. What is the total amount each customer spent at the restaurant?**

```sql
SELECT 
    customer_id, concat('$ ',SUM(price)) AS tot_amt
FROM
    diner_res
GROUP BY 1;
```


#### Output:

<kbd>![Solution 1.png](https://github.com/sivapavithran93/Danny-s-Diner-Case-Study-Using-SQL/blob/main/Images/Solution%201.png)</kbd>

#### Insights:

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

*** 

**Q-2. How many days has each customer visited the restaurant?**

```sql
SELECT 
    customer_id, concat(COUNT(distinct order_date),' days') AS cust_visit
FROM
    diner_res
GROUP BY 1;
```

#### Steps:
- To determine the unique number of visits for each customer, utilize **COUNT(DISTINCT `order_date`)**.
- It's important to apply the **DISTINCT** keyword while calculating the visit count to avoid duplicate counting of days. For instance, if Customer A visited the restaurant twice on '2021–01–07', counting without **DISTINCT** would result in 2 days instead of the accurate count of 1 day.

#### Output:
<kbd>![Solution 2.png](https://github.com/sivapavithran93/Danny-s-Diner-Case-Study-Using-SQL/blob/main/Images/Solution%202.png)</kbd>

#### Insights:

- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

***

**Q-3. What was the first item from the menu purchased by each customer?**

```sql
SELECT
    customer_id,product_name
FROM (
    SELECT *,
    dense_rank() over(partition by customer_id order by order_date) as rn
FROM
    diner_res
)AS c 
WHERE
    rn=1
GROUP BY 1,2;
```
#### Steps:
- Create a Subquery, create a new column `rank` as rn and calculate the row number using **DENSE_RANK()** window function. The **PARTITION BY** clause divides the data by `customer_id`, and the **ORDER BY** clause orders the rows within each partition by `order_date`.
- In the outer query, select the appropriate columns and apply a filter in the **WHERE** clause to retrieve only the rows where the rank column equals 1, which represents the first row within each `customer_id` partition.
- Use the GROUP BY clause to group the result by `customer_id` and `product_name`.
- we could have also used Common Table Expression CTE for the same.

#### Output: 
<kbd>![Solution 3.png](https://github.com/sivapavithran93/Danny-s-Diner-Case-Study-Using-SQL/blob/main/Images/Solution%203.png)</kbd>

#### Insights:

- Customer A placed an order for both curry and sushi simultaneously, making them the first items in the order.
- Customer B's first order is curry.
- Customer C's first order is ramen.

I thought of using of `ROW_NUMBER()` instead of `DENSE_RANK()` for determining the "first order" in this question. 

But, since the `order_date` does not have a timestamp, it is impossible to determine the exact sequence of items ordered by the customer. 

Therefore, it would be inaccurate to conclude that curry is the customer's first order purely based on the alphabetical order of the product names. For this reason, I hold on to my solution of using `DENSE_RANK()` and consider both curry and sushi as Customer A's first order.

Also if you see the C ordered raman twice on the first order but still we consider it as a same order

***

**Q-4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
SELECT 
    product_name, COUNT(customer_id) AS times_cust_bought
FROM
    diner_res
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```

#### Steps:
- Perform a **COUNT** aggregation on the `product_id` column and **ORDER BY** the result in descending order using `times_cust_bought` column.
- Apply the **LIMIT** 1 clause to filter and retrieve the highest number of purchased items.

#### Output: 
<kbd>![Solution 4.png](https://github.com/sivapavithran93/Danny-s-Diner-Case-Study-Using-SQL/blob/main/Images/Solution%204.png)</kbd>

#### Insights:

- Most purchased item on the menu is ramen which is 8 times. Yummy! it seems 😋

***

**Q-5. Which item was the most popular for each customer?**

```sql
with popular_item as(
SELECT
    customer_id,product_name,
    dense_rank() over(partition by customer_id  order by count(product_name)desc) as rn
FROM
    diner_res
GROUP BY 1,2
)

SELECT
    customer_id,product_name
FROM
    popular_item
WHERE
    rn = 1;
```
*Each user may have more than 1 favourite item.*

#### Steps:
- Create a CTE named `popular_item`.
- Group results by `customer_id` and `product_name` and calculate the count of `product_id` occurrences for each group. 
- Utilize the **DENSE_RANK()** window function to calculate the ranking of each `customer_id` partition based on the count of orders **COUNT(`product_name`)** in descending order.
- In the outer query, select the appropriate columns and apply a filter in the **WHERE** clause to retrieve only the rows where the rank column equals 1, representing the rows with the highest order count for each customer.

#### Output: 
<kbd>![Solution 5.png](https://github.com/sivapavithran93/Danny-s-Diner-Case-Study-Using-SQL/blob/main/Images/Solution%205.png)</kbd>

#### Insights:

- Customer A and C's favourite item is ramen.🍜
- Customer B enjoys all items on the menu. A true foodie, i guess 🤣.

***

**Q-6. Which item was purchased first by the customer after they became a member?**

```sql
WITH first_purchase_cte AS
(
SELECT *,
    rank() over(partition by customer_id order by diff) as rn
FROM
(
SELECT *,
    ABS(order_date-join_date) as diff
FROM
    diner_res
WHERE
    order_date>=join_date
)AS c
)

SELECT
    customer_id,product_name,order_date,join_date
FROM
    first_purchase_cte
WHERE
    rn=1;

```

#### Steps:
- Create a CTE named `first_purchase_cte`.
- Use `abs` on date difference `order_date-join_date`. 
- Utilize the **RANK()** window function to calculate the ranking of each `customer_id` partition based on the date of orders **diff** in ascending order.
- In **WHERE** clause, we use `order_date` greater than or equal to `join_date`
- In the outer query, select the appropriate columns and apply a filter in the **WHERE** clause to retrieve only the rows where the rank column equals 1, representing the rows with the first ordered item for each customer.

#### Output: 
<kbd>![Solution 6.png](https://github.com/sivapavithran93/Danny-s-Diner-Case-Study-Using-SQL/blob/main/Images/Solution%206.png)</kbd>

#### Insights:

- Customer A bought `Curry` as the first item on the day of becoming a member.
- Customer B bought `Sushi` as the first item 2 days after becoming a member.
- Customer C didn't become the member.

***

### Q-7. Which item was purchased just before the customer became a member?

```sql
WITH purchase_cte AS(
SELECT *,
    rank() over(partition by customer_id order by diff) as rn
FROM
(
SELECT *,
    abs(order_date-join_date) as diff
FROM
    diner_res
WHERE
    order_date<join_date
)AS c
)
SELECT
    customer_id,
    group_concat(product_name) as items_bought,
    order_date,join_date
FROM
    purchase_cte 
WHERE
    rn=1
group by 1,3,4;
```

#### Steps:
- Create a CTE named `purchase_cte`.
- Use `abs` on date difference `order_date-join_date`. 
- Utilize the **RANK()** window function to calculate the ranking of each `customer_id` partition based on the date of orders **diff** in ascending order.
- In **WHERE** clause, we use `order_date` less than `join_date`
- In the outer query, select the appropriate columns and apply a filter in the **WHERE** clause to retrieve only the rows where the rank column equals 1, representing the rows with the ordered item before becoming a member for each customer.
- *Group_concat* is used to concat the item_bought together by a customer

#### Output: 
<kbd>![Solution 7.png](https://github.com/sivapavithran93/Danny-s-Diner-Case-Study-Using-SQL/blob/main/Images/Solution%207.png)</kbd>

#### Insights:

- Customer A bought `Curry & Sushi` as the *item_bought* before becoming a member.
- Customer B bought `Sushi` as the *item_bought* before becoming a member.
- Customer C didn't become the member.

***

### Q-8. What is the total items and amount spent for each member before they became a member?

```sql
WITH before_member_cte AS
(
SELECT *
FROM
    diner_res
WHERE
    order_date < join_date
)

SELECT
    customer_id,
    count(product_id) as total_items_bought,
    concat('$ ',sum(price)) as total_cost
FROM
    before_member_cte
GROUP BY 1
ORDER BY 1;
```
#### Steps:
- Create a CTE named `before_member_cte`.
- In **WHERE** clause, we use `order_date` less than `join_date`
- In the outer query, select the appropriate columns with **count**`product_id` with **sum**`price` as *total_cost* representing the rows with the ordered items count with total_cost spent before becoming a member for each customer.

#### Output: 
<kbd>![Solution 8.png](https://github.com/sivapavithran93/Danny-s-Diner-Case-Study-Using-SQL/blob/main/Images/Solution%208.png)</kbd>

#### Insights:

Before becoming members,
- Customer A spent $25 on 2 items.
- Customer B spent $40 on 3 items.

***

**Q-9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?**

```sql
SELECT 
    customer_id,
    SUM(CASE
        WHEN product_name IN ('curry' , 'ramen') THEN price * 10
        ELSE price * 20
    END) AS points
FROM
    diner_res
GROUP BY 1;
```

#### Steps:
Let's break down the question to understand the point calculation for each customer's purchases.
- Each $1 spent = 10 points. However, `product_id` 1 sushi gets 2x points, so each $1 spent = 20 points.
- Here's how the calculation is performed using a conditional CASE statement:
	- If product_id = 1, multiply every $1 by 20 points.
	- Otherwise, multiply $1 by 10 points.
- Then, calculate the total points for each customer.

#### Output: 
<kbd>![Solution 9.png](https://github.com/sivapavithran93/Danny-s-Diner-Case-Study-Using-SQL/blob/main/Images/Solution%209.png)</kbd>

#### Insights:

- Total points for Customer A is $860.
- Total points for Customer B is $940.
- Total points for Customer C is $360.

***

**Q-10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?**

```sql
SELECT 
    customer_id,
    SUM(CASE
        WHEN product_name = 'sushi' THEN price * 20
        WHEN order_date BETWEEN join_date AND DATE_ADD(join_date, INTERVAL 6 DAY) THEN price * 20
        ELSE price * 10
    END) AS points
FROM
    diner_res
WHERE
    order_date >= join_date
        AND EXTRACT(MONTH FROM order_date) = 1
GROUP BY 1
ORDER BY 1;
```

#### Assumptions:
- Before Day 1 (the day a customer becomes a member), each $1 spent earns 10 points. However, for sushi, each $1 spent earns 20 points.
- From Day 1 to Day 7 (the first week of membership), each $1 spent for any items earns 20 points.
- From Day 8 to the last day of January 2021, each $1 spent earns 10 points. However, sushi continues to earn double the points at 20 points per $1 spent.

#### Steps:

- In the query, calculate the points by using a `CASE` statement to determine the points based on our assumptions above. 
- If the `product_name` is 'sushi', multiply the price by 20. For orders placed between `join_date` and `interval 6 day` (i.e the first week of memebership) , also multiply the price by 20. 
- For all other products, multiply the price by 10.
- Calculate the sum of points for each customer.

#### Output: 
<kbd>![Solution 10.png](https://github.com/sivapavithran93/Danny-s-Diner-Case-Study-Using-SQL/blob/main/Images/Solution%2010.png)</kbd>

#### Insights:

- Total points for Customer A is 1,020.
- Total points for Customer B is 320.

***

## BONUS QUESTIONS

**Join All The Things**

**Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)**

```sql
SELECT 
    customer_id,
    order_date,
    product_name,
    price,
    IF(order_date >= join_date, 'Y', 'N') AS member
FROM
    diner_res
ORDER BY 1 , 2 , 3;
```

#### Steps:

- Since we created a **`view`** called *`Diner_res`* on this it is easy to identity if the customer is a member or not.
- Simple `IF` statement is suffice to identity those who are members.  

#### Output: 
<kbd>![Solution 11.png](https://github.com/sivapavithran93/Danny-s-Diner-Case-Study-Using-SQL/blob/main/Images/Solution%2011.png)</kbd>

***

**Rank All The Things**

**Danny also requires further information about the ```ranking``` of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ```ranking``` values for the records when customers are not yet part of the loyalty program.**

```sql
 WITH ranking_cte AS(
 SELECT 
    customer_id,
    order_date,
    product_name,
    price,
    IF(order_date >= join_date, 'Y', 'N') AS member
FROM
    diner_res
)

SELECT *,
    IF(member="N" , NULL , rank() over(partition by customer_id,member order by customer_id,order_date,product_name)) as ranking
FROM
    ranking_cte
ORDER BY 1,2,3;
```

#### Steps:

- Since we created a **`view`** called *`Diner_res`* on this it is easy to identity if the customer is a member or not.
- Simple `IF` statement is suffice to identity those who are members.
- Further `IF` statement is used and `Rank` window function is used for ranking customers based on customer_id and their membership.

#### Output: 

<kbd>![Solution 12.png](https://github.com/sivapavithran93/Danny-s-Diner-Case-Study-Using-SQL/blob/main/Images/Solution%2012.png)</kbd>

***
