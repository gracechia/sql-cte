# Common Table Expressions (CTE)

## What is a CTE?
You can think of it as a temporary set of rows that you define on your own and then use in the same query. In principle, CTEs are similar to subqueries.

Let's take a look at the syntax of a CTE:
```
WITH project_revenue AS ( -- name your CTE here
  SELECT
    project.id,
    SUM(amount) AS sum_amount
  FROM project 
  LEFT JOIN donation 
         ON donation.project_id = project.id
  GROUP BY project.id
)

SELECT
  id,
  sum_amount
FROM project_revenue -- reference your CTE here
```
- Every CTE query starts with `WITH`
- You need to give your CTE a name (`project_revenue`), and define it within the parentheses. 
- Then, once you close the bracket, you can select columns from this CTE as if it were a table.
- Note that you need to define your CTE first, i.e. before the SELECT clause of the outer query.

## Why use CTEs?
CTEs are more readable than subqueries

## Example of a Subquery vs CTE

This uses a subquery:
```
SELECT
  customers.id,
  customers.email,
  customers.name,
  customer_orders.orders_count
FROM ( -- subqueries are hard to read
    SELECT
        orders.customer_id,
        COUNT(orders.id) AS orders_count
    FROM orders
    GROUP BY orders.customer_id
) customer_orders
LEFT JOIN customers
       ON customers.id = customer_orders.customer_id
```

This uses a CTE instead:
```
WITH customer_orders AS (
  SELECT
      orders.customer_id,
      COUNT(orders.id) AS orders_count
  FROM orders
  GROUP BY orders.customer_id
)
SELECT
    customers.id,
    customers.email,
    customers.name,
    customer_orders.orders_count
FROM customer_orders
LEFT JOIN customers
       ON customers.id = customer_orders.customer_id
```

## Let's try using CTEs in BigQuery
Log in to Google BigQuery: https://cloud.google.com/bigquery
We will be using the public dataset for stackoverflow

```
WITH user_posts AS (
    SELECT
        users.id,
        users.display_name,
        COUNT(DISTINCT posts.id) AS no_of_posts
    FROM `bigquery-public-data.stackoverflow.users` AS users
    LEFT JOIN `bigquery-public-data.stackoverflow.post_history` AS posts
           ON users.id = posts.user_id
    GROUP BY
        users.id,
        users.display_name
)

SELECT
    id,
    display_name,
    no_of_posts
FROM user_posts
```

## Many CTEs in one query
You can actually have as many CTEs in a single query as you need. Each of them should be separated with a comma, and the `WITH` keyword should only appear once, at the beginning.

Do remember that WITH appears only once, at the beginning. The other CTEs are separated with commas. DO NOT put a comma after the last CTE.

Introducing multiple CTEs usually makes sense when they refer to each other. For now, we may think of other usages: for instance, you can use set operations like UNION to show results from both CTEs.

Now, in BigQuery, try:
```
WITH favourites_min AS (
    SELECT
        users.id,
        users.display_name,
        SUM(favorite_count) AS no_of_favorites
    FROM `bigquery-public-data.stackoverflow.users` AS users
    LEFT JOIN `bigquery-public-data.stackoverflow.posts_questions` AS posts_questions
           ON users.id = posts_questions.owner_user_id
    GROUP BY
        users.id,
        users.display_name
    HAVING
        no_of_favorites > 50 -- users who received more than 50 favourites
),

comment_min AS (
    SELECT
        users.id,
        users.display_name,
        SUM(comment_count) AS no_of_comments
    FROM `bigquery-public-data.stackoverflow.users` AS users
    LEFT JOIN `bigquery-public-data.stackoverflow.posts_questions` AS posts_questions
           ON users.id = posts_questions.owner_user_id
    GROUP BY
        users.id,
        users.display_name
    HAVING
        no_of_comments > 20 -- users who received more than 20 comments
)

SELECT
    id,
    display_name
FROM favourites_min
UNION ALL -- union to get users who received more than 50 favourites OR 20 comments
SELECT
    id,
    display_name
FROM comment_min
```

## Credits
https://academy.vertabelo.com/course/common-table-expressions
