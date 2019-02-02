# MySQL_Sakila

# select sakila as active database
USE sakila;

# display actor table
SELECT * FROM actor;

# 1a- display only first name and last name of actors
SELECT first_name, last_name FROM actor;

# 1b- create 'Actor_Name' column with first and last name displayed in upper case letters in one column
SELECT CONCAT(first_name,' ', last_name) AS Actor_Name FROM actor;
 
# 2a- You need to find the ID number, first name, and last name of an actor, of whom you know only the first name, "Joe." 
# What is one query would you use to obtain this information?
SELECT * FROM actor
WHERE first_name= "JOE";

# 2b. Find all actors whose last name contain the letters GEN:
SELECT * FROM actor
WHERE last_name LIKE "%GEN%";

# 2c. Find all actors whose last names contain the letters LI. This time, order the rows by 
# last name and first name, in that order:
SELECT * FROM actor
WHERE last_name LIKE "%LI%" ORDER BY last_name, first_name;

# 2d. Using IN, display the country_id and country columns of the following countries: 
# Afghanistan, Bangladesh, and China:
SELECT country_id, country FROM country
WHERE country IN ('Afghanistan', 'Bangladesh', 'China');

# 3a. You want to keep a description of each actor. You don't think you will be performing queries on a description, 
# so create a column in the table actor named description and use the data type BLOB (Make sure to research 
# the type BLOB, as the difference between it and VARCHAR are significant).
ALTER TABLE actor ADD COLUMN description BLOB;
SELECT * FROM actor;

# 3b. Very quickly you realize that entering descriptions for each actor is too much effort. 
# Delete the description column.
ALTER TABLE actor
DROP COLUMN description;

# 4a. List the last names of actors, as well as how many actors have that last name.
SELECT last_name, COUNT(last_name) AS "Count"
FROM actor
GROUP BY last_name;

# 4b. List last names of actors and the number of actors who have that last name, 
# but only for names that are shared by at least two actors
SELECT last_name, COUNT(last_name) AS "Count"
FROM actor
GROUP BY last_name
HAVING COUNT(last_name) >= 2;

# 4c. The actor HARPO WILLIAMS was accidentally entered in the actor table as GROUCHO WILLIAMS. 
# Write a query to fix the record.
UPDATE actor
SET first_name = "HARPO"
WHERE first_name = "GROUCHO" AND last_name = "WILLIAMS";

SELECT first_name, last_name FROM actor WHERE first_name = "HARPO";

# 4d. Perhaps we were too hasty in changing GROUCHO to HARPO. It turns out that GROUCHO was the correct name 
# after all! In a single query, if the first name of the actor is currently HARPO, change it to GROUCHO.
UPDATE actor
SET first_name = "GROUCHO"
WHERE first_name = "HARPO" AND last_name = "WILLIAMS";

SELECT first_name, last_name FROM actor 
WHERE first_name = "GROUCHO" AND last_name = "WILLIAMS";

# 5a. You cannot locate the schema of the address table. Which query would you use to re-create it?
# Hint: https://dev.mysql.com/doc/refman/5.7/en/show-create-table.html
SHOW CREATE TABLE address;

# 6a. Use JOIN to display the first and last names, as well as the address, of each staff member. 
# Use the tables staff and address:
SELECT * FROM address;
SELECT * FROM staff;

SELECT address.address, staff.first_name, staff.last_name
FROM staff
JOIN address ON
address.address_id = staff.address_id;

# 6b. Use JOIN to display the total amount rung up by each staff member in August of 2005. 
# Use tables staff and payment.
SELECT * FROM payment;
SELECT staff.first_name, staff.last_name, SUM(payment.amount) AS "Total Sales"
FROM staff
JOIN payment ON
payment.staff_id = staff.staff_id
WHERE payment.payment_date LIKE "%2005-08%"
GROUP BY staff.staff_id;

# 6c. List each film and the number of actors who are listed for that film. Use tables film_actor and film. 
# Use inner join.
SELECT * FROM film;
SELECT film.title, COUNT(film_actor.actor_id) AS `# of Actors`
FROM film
JOIN film_actor ON
film_actor.film_id = film.film_id
GROUP BY film.title;

# 6d. How many copies of the film Hunchback Impossible exist in the inventory system?
SELECT title, count(title) AS "# of Versions"
FROM film
WHERE title = 'Hunchback Impossible';

# 6e. Using the tables payment and customer and the JOIN command, list the total paid by each customer. 
# List the customers alphabetically by last name:
SELECT customer.first_name, customer.last_name, sum(payment.amount) AS 'Customer Total'
FROM payment
JOIN customer
ON customer.customer_id = payment.customer_id
GROUP BY payment.customer_id
ORDER BY last_name;


# 7a. The music of Queen and Kris Kristofferson have seen an unlikely resurgence. As an unintended consequence, 
# films starting with the letters K and Q have also soared in popularity. 
# Use subqueries to display the titles of movies starting with the letters K and Q whose language is English.
SELECT title
FROM film
WHERE language_id IN
(
SELECT language_id
FROM language
WHERE name = "English"
)
AND title LIKE 'K%' or title LIKE 'Q%';

# 7b. Use subqueries to display all actors who appear in the film Alone Trip.
SELECT first_name, last_name
FROM actor
WHERE actor_id in
(
SELECT actor_id
FROM film_actor
WHERE film_id IN
(
SELECT film_id
FROM film
WHERE title = 'Alone Trip'
)
);

# 7c. You want to run an email marketing campaign in Canada, for which you will need the names and 
# email addresses of all Canadian customers. Use joins to retrieve this information.
SELECT first_name, last_name, email
FROM customer
WHERE address_id IN
(
SELECT address_id
FROM address
WHERE city_id IN
(
SELECT city_id
FROM city
WHERE country_id IN
(
SELECT country_id
FROM country
WHERE country = 'Canada'
)
)
);

# 7d. Sales have been lagging among young families, and you wish to target all family movies for a promotion. 
# Identify all movies categorized as family films.
SELECT title
FROM film
WHERE film_id in
(
SELECT film_id
FROM film_category
WHERE category_id IN
(
SELECT category_id
FROM category
WHERE name = "Family"
)
);

# 7e. Display the most frequently rented movies in descending order.
SELECT film.title, COUNT(rental.rental_id) AS `# of Rentals`
FROM film
JOIN inventory ON
film.film_id = inventory.film_id
JOIN rental ON
inventory.inventory_id = rental.inventory_id
GROUP BY film.title
ORDER BY `# of Rentals` DESC;

# 7f. Write a query to display how much business, in dollars, each store brought in.
# There are multiple ways to solve this problem. The start and end tables must be store and payment, in-between you can join
# on customer, store, or inventory and rental and connect store and payment. Each join pattern yields different sums for the two stores.
# In this case, the store_id and staff_id and the same and the first query below yields the correct totals. I matched this with the pattern that yielded
# the same results (payment-> staff -> store).


SELECT SUM(payment.amount) 
FROM payment 
JOIN staff 
WHERE payment.staff_id = staff.staff_id 
GROUP BY staff.staff_id;

SELECT store.store_id, SUM(payment.amount) AS 'Store Sales Total'
FROM store
JOIN staff ON
store.store_id = staff.store_id
JOIN payment ON
staff.staff_id = payment.staff_id
GROUP BY store.store_id;

# 7g. Write a query to display for each store its store ID, city, and country.
SELECT store.store_id, city.city, country.country
FROM store
JOIN address ON
store.address_id = address.address_id
JOIN city ON
address.city_id = city.city_id
JOIN country ON
city.country_id = country.country_id;

# 7h. List the top five genres in gross revenue in descending order. 
# (Hint: you may need to use the following tables: category, film_category, inventory, payment, and rental.)
SELECT category.name, SUM(payment.amount) AS `Gross Revenue`
FROM category
JOIN film_category ON
category.category_id = film_category.category_id
JOIN inventory ON
film_category.film_id = inventory.film_id
JOIN rental ON
inventory.inventory_id = rental.inventory_id
JOIN payment ON
rental.rental_id = payment.rental_id
GROUP BY category.name
ORDER BY `Gross Revenue` DESC LIMIT 5;

# 8a. In your new role as an executive, you would like to have an easy way of viewing the Top five 
# genres by gross revenue. Use the solution from the problem above to create a view.
# If you haven't solved 7h, you can substitute another query to create a view.
CREATE VIEW Top_5_Genres AS
SELECT category.name, SUM(payment.amount) AS `Gross Revenue`
FROM category
JOIN film_category ON
category.category_id = film_category.category_id
JOIN inventory ON
film_category.film_id = inventory.film_id
JOIN rental ON
inventory.inventory_id = rental.inventory_id
JOIN payment ON
rental.rental_id = payment.rental_id
GROUP BY category.name
ORDER BY `Gross Revenue` DESC LIMIT 5;

# 8b. How would you display the view that you created in 8a?
SELECT * FROM Top_5_Genres;

# 8c. You find that you no longer need the view top_five_genres. Write a query to delete it.
DROP VIEW Top_5_Genres;
