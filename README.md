    SQL_PROJECT-MUSIC STORE DATA ANALYSIS

#1. Who is the senior most employee based on job title?

   SELECT first_name, last_name, title 
   FROM employee 
   ORDER BY levels DESC limit 1

#2. Which countries have the most Invoices?

   SELECT billing_country, count(*) as total 
   FROM invoice 
   GROUP BY billing_country 
   ORDER BY total DESC

#3. What are top 3 values of total invoice?

   SELECT total 
   FROM invoice 
   ORDER BY total DESC limit 3

#4. Which city has the best customers? We would like to throw a promotional Music 
Festival in the city we made the most money. Write a query that returns one city that 
has the highest sum of invoice totals. Return both the city name & sum of all invoice 
totals
 
  SELECT billing_city, sum(total) as total 
  FROM invoice 
  GROUP BY billing_city 
  ORDER BY total DESC limit 1

#5. Who is the best customer? The customer who has spent the most money will be 
declared the best customer. Write a query that returns the person who has spent the 
most money 

  SELECT customer.customer_id, first_name, last_name, sum(total) as total
  FROM customer 
  JOIN invoice on customer.customer_id=invoice.customer_id 
  GROUP BYcustomer.customer_id, first_name, last_name
  ORDER BY total DESC limit 1

#6. Write query to return the email, first name, last name, & Genre of all Rock Music 
listeners. Return your list ordered alphabetically by email starting with A 

  SELECT distinct email, first_name,last_name, g.name
  FROM customer as c
  JOIN invoice as i
  ON c.customer_id=i.customer_id
  JOIN invoice_line as iline
  ON i.invoice_id=iline.invoice_id
  JOIN track as t
  ON iline.track_id=t.track_id
  JOIN genre as g 
  ON t.genre_id=g.genre_id
  WHERE g.name LIKE 'rock'
  ORDER BY email

#7. Let's invite the artists who have written the most rock music in our dataset. Write a 
query that returns the Artist name and total track count of the top 10 rock bands 

  SELECT distinct artist.artist_id, artist.name, count(artist.artist_id) as count_track
  FROM artist
  JOIN album2 
  ON artist.artist_id=album2.artist_id
  JOIN track 
  ON album2.album_id=track.album_id
  JOIN genre 
  ON track.genre_id=genre.genre_id
  WHERE genre.name LIKE 'Rock'
  GROUP BY artist_id, artist.name
  ORDER BY count_track DESC
  limit 10

#8. Return all the track names that have a song length longer than the average song length. 
Return the Name and Milliseconds for each track. Order by the song length with the 
longest songs listed first 

  SELECT track.name, track.milliseconds
  FROM track
  WHERE track.milliseconds > (

      SELECT avg(track.milliseconds) as avg_mill
      FROM track)
  ORDER BY track.milliseconds DESC

#9. Find how much amount spent by each customer on artists? Write a query to return 
customer name, artist name and total spent

  WITH best_selling_artist AS (
    SELECT artist.artist_id, artist.name as artist_name, sum(invoice_line.unit_price*invoice_line.quantity) as total_sales
    FROM invoice_line
    JOIN track
    ON invoice_line.track_id=track.track_id
    JOIN album2
    ON track.album_id=album2.album_id
    JOIN artist
    ON album2.artist_id=artist.artist_id
    GROUP BY 1,2
    ORDER BY 3 DESC
    limit 1 
   )
   SELECT customer.customer_id, customer.first_name, customer.last_name, best_selling_artist.artist_name, 
   SUM(invoice_line.unit_price*invoice_line.quantity) as amount_spent
   FROM customer
   JOIN invoice
   ON customer.customer_id=invoice.customer_id
   JOIN invoice_line 
   ON invoice.invoice_id=invoice_line.invoice_id
   JOIN track 
   ON invoice_line.track_id=track.track_id
   JOIN album2
   ON track.album_id=album2.album_id
   JOIN best_selling_artist
   ON album2.artist_id=best_selling_artist.artist_id
   GROUP BY 1,2,3,4
   ORDER BY 5 desc

#10. We want to find out the most popular music Genre for each country. We determine the 
most popular genre as the genre with the highest amount of purchases. Write a query 
that returns each country along with the top Genre. For countries where the maximum 
number of purchases is shared return all Genres

   WITH popular_genre AS (
     SELECT customer.country, genre.name, genre.genre_id, count(invoice_line.quantity) as purchase, 
     ROW_NUMBER() over(partition by customer.country order by count(invoice_line.quantity) DESC) as RowNo
     FROM customer 
     JOIN invoice
     ON customer.customer_id=invoice.customer_id
     JOIN invoice_line
     ON invoice.invoice_id=invoice_line.invoice_id
     JOIN track
     ON invoice_line.track_id=track.track_id
     JOIN genre
     ON track.genre_id=genre.genre_id
     GROUP BY 1,2,3
     ORDER BY 1 ASC, 4 DESC 
   )
   SELECT * from popular_genre where RowNo=1

#11. Write a query that determines the customer that has spent the most on music for each 
country. Write a query that returns the country along with the top customer and how 
much they spent. For countries where the top amount spent is shared, provide all 
customers who spent this amount

   WITH customer_country_amount AS (
    SELECT customer.customer_id, customer.first_name, customer.last_name, invoice.billing_country, sum(invoice.total) as 
    total,
    ROW_NUMBER() OVER(partition by invoice.billing_country order by sum(invoice.total) DESC) as RowNo
    FROM customer
    JOIN invoice 
    ON customer.customer_id=invoice.customer_id
    GROUP BY 1,2,3,4
    ORDER BY 4 ASC, 5 DESC
   )
