/* Query 1 - What movie category are families watching?
The following categories are considered family movies: Animation, Children, Classics, Comedy, Family and Music. */

SELECT c.name AS category_name, COUNT(rental) AS rental_count
FROM film AS f
JOIN film_category AS fc
      ON f.film_id = fc.film_id
JOIN category AS c   
      ON fc.category_id = c.category_id
JOIN inventory AS i
      ON f.film_id = i.film_id
JOIN rental
ON i.inventory_id = rental.inventory_id
WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
GROUP BY 1
ORDER BY 1



/* QUERY 2-  What is the number of rental orders for each store per month?
*/

SELECT  Date_TRUNC('MONTH', rental_date) Rental_month, store_id, COUNT(*) AS COUNT
FROM (SELECT store.store_id, rental.rental_date
      FROM store 
      JOIN staff 
      ON store.store_id = staff.store_id
      JOIN rental
      ON rental.staff_id = staff.staff_id) AS t1
GROUP BY 1, 2
ORDER BY 1




/* QUERY 3- Who were the top 10 paying customers and the total payments made each month for the year 2007? 
*/

WITH t1 AS (
  SELECT C.customer_id,
	     SUM(p.amount) AS total_payment
   FROM customer c
	  JOIN payment p
   ON p.customer_id = c.customer_id
   GROUP BY 1
   ORDER BY 2 DESC
   LIMIT 10 )

SELECT c.first_name || ' ' || c.last_name AS customer_name,
		     DATE_TRUNC('month', p.payment_date) AS payment_month,
		     COUNT(p.payment_id) AS pay_countpermonth,
		     SUM(p.amount) AS total_payment
FROM customer c
JOIN payment p
ON c.customer_id = p.customer_id
JOIN t1
ON c.customer_id = t1.customer_id
GROUP BY 1, 2
ORDER BY 1, 2;






/* Query 4 - What's the difference across monthly payments for the top 10 paying customers during 2007?. */

WITH t1 AS (
             SELECT C.customer_id,
	         SUM(p.amount) AS total_payment
             FROM customer c
	         JOIN payment p
             ON p.customer_id = c.customer_id
             GROUP BY 1
             ORDER BY 2 DESC
              LIMIT 10 
),

t2 AS (
        SELECT c.first_name || ' ' || c.last_name AS customer_name,
		       DATE_TRUNC('month', p.payment_date) AS payment_month,
		       COUNT(p.payment_id) AS pay_countpermonth,
		       SUM(p.amount) AS total_payment
        FROM customer c
        JOIN payment p
        ON c.customer_id = p.customer_id
        JOIN t1
        ON c.customer_id = t1.customer_id
        GROUP BY 1, 2
        ORDER BY 1, 2)

SELECT customer_name, 
       payment_month, 
       total_payment,
       LAG(total_payment) OVER(ORDER BY customer_name) AS prev_payment,
       total_payment - LAG(total_payment) OVER(ORDER BY customer_name) AS payment_difference
FROM t2
ORDER BY 5 DESC


