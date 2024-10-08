# Music_store_Analysis
This project contains a series of SQL queries aimed at analyzing a music store database. The queries focus on extracting and analyzing key data, such as sales performance, customer demographics, and inventory details, providing insights into store operations and business trends.

1Q. Who is the senior most employee based on job title? 
    SELECT * FROM employee
    ORDER by levels DESC
    LIMIT 1

2Q: Which countries have the most Invoices?
    SELECT COUNT(*) as Total_invoices, billing_country 
    FROM invoice
    GROUP BY billing_country
    ORDER BY Total_invoices desc

3Q: What are top 3 values of total invoice and the countries? 
    SELECT billing_country, total FROM invoice 
    ORDER BY total desc
    LIMIT 3

4Q: Q4: Which city has the best customers? We would like to throw a promotional Music Festival in the city we made the most money. 
	Write a query that returns one city that has the highest sum of invoice totals. 
	Return both the city name & sum of all invoice totals
    SELECT billing_city, ROUND(SUM(total)) AS Total_Invoices
    FROM invoice
    GROUP BY billing_city
    ORDER BY Total_Invoices  DESC
    LIMIT 1

5Q: Who is the best customer? The customer who has spent the most money will be declared the best customer. 
    Write a query that returns the person who has spent the most money.
    SELECT c.customer_id, c.first_name, c.last_name, ROUND(SUM(i.total)) AS Total_money 
    FROM customer c
    INNER JOIN invoice i
    on c.customer_id = i.customer_id
    GROUP BY c.customer_id
    ORDER BY Total_money DESC
    LIMIT 1

6Q: Write query to return the email, first name, last name, & Genre of all Rock Music listeners. 
    Return your list ordered alphabetically by email starting with A.
Mtd1:   SELECT DISTINCT c.email, c.first_name, c.last_name FROM customer c
        INNER JOIN invoice i ON c.customer_id = i.customer_id
        INNER JOIN invoice_line il ON i.invoice_id = il.invoice_id
        WHERE track_id IN (
        	SELECT t.track_id FROM track t
        	JOIN genre g ON g.genre_id = t.genre_id
        	WHERE g.name = 'Rock')
        ORDER BY c.email 

Mtd2:   SELECT DISTINCT c.email, c.first_name, c.last_name FROM customer c
        INNER JOIN invoice i ON c.customer_id = i.customer_id
        INNER JOIN invoice_line il ON i.invoice_id = il.invoice_id
        INNER JOIN track t ON il.track_id = t.track_id
        INNER JOIN genre g ON t.genre_id = g.genre_id
        WHERE g.name = 'Rock'
        ORDER BY c.email 

 7Q: Let's invite the artists who have written the most rock music in our dataset. 
     Write a query that returns the Artist name and total track count of the top 10 rock bands. 
     	SELECT ar.name, COUNT(ar.artist_id) AS num_songs FROM track t
	JOIN album a ON a.album_id = t.album_id
	JOIN artist ar ON ar.artist_id = a.artist_id
	JOIN genre g ON t.genre_id = g.genre_id
	WHERE g.name = 'Rock'
	GROUP BY ar.artist_id
	ORDER BY num_songs DESC
	LIMIT 10;

 8Q: Return all the track names that have a song length longer than the average song length. 
     Return the Name and Milliseconds for each track. Order by the song length with the longest songs listed first. 
     SELECT name, milliseconds from track
     WHERE milliseconds >(
	SELECT AVG(milliseconds) as avg_song_len
	from track)
     ORDER BY milliseconds DESC:

9Q: Find how much amount spent by each customer on top selling artists? Write a query to return customer name, artist name and total spent?
    --Create a CTE to find top selling artist
 	with top_selling_artist AS(
		SELECT ar.artist_id as artist_id, ar.name as artist_name, sum(il.unit_price*il.quantity) as total_spent 
		FROM invoice_line il
		JOIN track t ON t.track_id = il.track_id
		JOIN album a ON a.album_id = t.album_id
		JOIN artist ar ON a.artist_id = ar.artist_id
		GROUP BY 1
		ORDER BY 3 DESC
		LIMIT 1
	    	)
	SELECT c.customer_id, c.first_name, c.last_name, tsa.artist_name,
	SUM(il.unit_price*il.quantity) AS total_spent
	FROM invoice i 
	JOIN customer c ON c.customer_id = i.customer_id
	JOIN invoice_line il ON il.invoice_id = i.invoice_id
	JOIN track t ON t.track_id = il.track_id
	JOIN album a ON a.album_id = t.album_id
	JOIN top_selling_artist tsa ON tsa.artist_id = a.artist_id
	GROUP BY 1,2,3,4 
	ORDER BY 5 DESC
 
 10Q: We want to find out the most popular music Genre for each country. We determine the most popular genre as the genre 
      with the highest amount of purchases. Write a query that returns each country along with the top Genre. For countries where 
      the maximum number of purchases is shared return all Genres

      WITH popular_genre AS
	(
	SELECT c.country, COUNT(il.quantity)AS num_of_purchases, g.name,g.genre_id,
		 ROW_NUMBER() OVER(PARTITION BY c.country ORDER BY COUNT(il.quantity) DESC) AS RoW_num
	From invoice_line il
	JOIN invoice i ON i.invoice_id = il.invoice_id
	JOIN customer c ON c.customer_id = i.customer_id
	JOIN track t ON t.traCk_id = il.track_id
	JOIN genre g ON g.genre_id = t.genre_id
	GROUP BY c.country, g.name,g.genre_id
	ORDER BY c.country ASC, num_of_purchases DESC 
	)
	SELECT * FROM popular_genre WHERE RoW_num <=1

 11Q. Write a query that determines the customer that has spent the most on music for each country. 
      Write a query that returns the country along with the top customer and how much they spent. 
      For countries where the top amount spent is shared, provide all customers who spent this amount

   MTD1-   	WITH recursive 
	      	customer_with_country AS(
			SELECT c.customer_id, c.first_name, c.last_name, i.billing_country, sum(total) as total_spent
			FROM customer c 
			JOIN invoice i ON i.customer_id = c.customer_id
			GROUP BY 1, 2, 3, 4 
			ORDER BY c.customer_id ASC, total_spent DESC, i.billing_country asc),
	
	    	country_most_spent AS(
			SELECT billing_country, MAX(total_spent) as max_spending
			FROM customer_with_country 
			GROUP BY billing_country)

		SELECT cc.billing_country, cc.total_spent, cc.first_name, cc.last_name, cc.customer_id
		FROM customer_with_country cc
		JOIN country_most_spent cs ON cc.billing_country = cs.billing_country
		WHERE cc.total_spent = cs.max_spending
		ORDER BY 1

       MTD2-  WITH customer_with_country AS(
			SELECT c.customer_id, c.first_name, c.last_name, i.billing_country, sum(total) as total_spent,
					ROW_NUMBER() OVER(PARTITION BY I.billing_country ORDER BY sum(total) DESC) AS Row_num
			FROM customer c
			JOIN invoice i ON i.customer_id = c.customer_id
			GROUP BY 1, 2, 3, 4 
			ORDER BY total_spent DESC, i.billing_country asc)
		SELECT * FROM customer_with_country WHERE Row_num <=1


	  



            

	







