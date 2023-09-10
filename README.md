# UDACITY_MINI_PROJECT
Writing a sql query using JOIN,WINDOW FUNCTION and SUB_QUERY

##Question 1
Create a query that lists each movie,
the film category it is classified in,
and the number of times it has been rented out.

SELECT
   f.title,
   c.name,
   COUNT(r.rental_date) AS Rental_count 
FROM
   rental r 
   JOIN
      inventory i 
      ON r.inventory_id = i.inventory_id 
   JOIN
      film f 
      ON i.film_id = f.film_id 
   JOIN
      film_category fc 
      ON f.film_id = fc.film_id 
   JOIN
      category c 
      ON c.category_id = fc.category_id 
GROUP BY
   f.title,
   c.name 
ORDER BY
   c.name,
   f.title;
##Question 2 write a query to capture the customer name,
month 
and year of payment,
and total payment amount for each month by these top 10 paying customers ? 

SELECT
   CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
   EXTRACT(MONTH 
FROM
   p.payment_date) AS month,
   EXTRACT(YEAR 
FROM
   p.payment_date) AS year,
   SUM(p.amount) AS total_payment_amount 
FROM
   payment p 
   JOIN
      customer c 
      ON p.customer_id = c.customer_id 
WHERE
   c.customer_id IN 
   (
      SELECT
         customer_id 
      FROM
         payment 
      GROUP BY
         customer_id 
      ORDER BY
         SUM(amount) DESC LIMIT 10 
   )
GROUP BY
   customer_name,
   month,
   year 
ORDER BY
   customer_name,
   year,
   month;

##QUESTION 3 Write a query that returns the store ID for the store,
the year 
and month 
and the number of rental orders each store has fulfilled for that month. Your table should include a column for each of the following: year,
month,
store ID 
and count of rental orders fulfilled during that month. 

SELECT
   EXTRACT(YEAR 
FROM
   rental_date) AS year,
   EXTRACT(MONTH 
FROM
   rental_date) AS month,
   i.store_id,
   COUNT(*) AS rental_order_count 
FROM
   rental r 
   JOIN
      inventory i 
      ON i.inventory_id = r.inventory_id 
GROUP BY
   year,
   month,
   store_id 
ORDER BY
   year,
   month,
   store_id;

##question 4 Can you provide a table with the movie titles 
and divide them into 4 levels (first_quarter, second_quarter, third_quarter, 
and final_quarter) based 
on the quartiles (25 % , 50 % , 75 % ) of the average rental duration(in the number of days) for movies across all categories ? WITH avg_rental_duration AS 
(
   SELECT
      AVG(rental_duration) AS avg_duration 
   FROM
      film 
)
,
quartiles AS 
(
   SELECT
      PERCENTILE_CONT(0.25) WITHIN GROUP (
   ORDER BY
      avg_duration) AS first_quarter,
      PERCENTILE_CONT(0.50) WITHIN GROUP (
   ORDER BY
      avg_duration) AS second_quarter,
      PERCENTILE_CONT(0.75) WITHIN GROUP (
   ORDER BY
      avg_duration) AS third_quarter 
   FROM
      avg_rental_duration 
)
,
movie_levels AS 
(
   SELECT
      film.title,
      CASE
         WHEN
            film.rental_duration <= quartiles.first_quarter 
         THEN
            'first_quarter' 
         WHEN
            film.rental_duration <= quartiles.second_quarter 
         THEN
            'second_quarter' 
         WHEN
            film.rental_duration <= quartiles.third_quarter 
         THEN
            'third_quarter' 
         ELSE
            'final_quarter' 
      END
      AS level 
   FROM
      film, quartiles 
)
SELECT
   film_category.name AS category,
   movie_levels.title,
   movie_levels.level 
FROM
   film_category 
   JOIN
      film 
      ON film_category.film_id = film.film_id 
   JOIN
      movie_levels 
      ON film.title = movie_levels.title;
