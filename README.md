                                                                     SQL_PROJECT-MUSIC STORE DATA ANALYSIS

#1. Who is the senior most employee based on job title?

select first_name, last_name, title from employee order by levels DESC limit 1

#2. Which countries have the most Invoices?

select billing_country, count(*) as total from invoice group by billing_country order by total DESC

#3. What are top 3 values of total invoice?

select total from invoice order by total DESC limit 3

#4. Which city has the best customers? We would like to throw a promotional Music 
Festival in the city we made the most money. Write a query that returns one city that 
has the highest sum of invoice totals. Return both the city name & sum of all invoice 
totals
 
select billing_city, sum(total) as total from invoice group by billing_city order by total DESC limit 1

#5. Who is the best customer? The customer who has spent the most money will be 
declared the best customer. Write a query that returns the person who has spent the 
most money 

select customer.customer_id, first_name, last_name, sum(total) as total
from customer 
join invoice on customer.customer_id=invoice.customer_id 
group by customer.customer_id, first_name, last_name
order by total DESC limit 1

#6. Write query to return the email, first name, last name, & Genre of all Rock Music 
listeners. Return your list ordered alphabetically by email starting with A 

select distinct email, first_name,last_name, g.name
from customer as c
join invoice as i
on c.customer_id=i.customer_id
join invoice_line as iline
on i.invoice_id=iline.invoice_id
join track as t
on iline.track_id=t.track_id
join genre as g 
on t.genre_id=g.genre_id
where g.name LIKE 'rock'
order by email

#7. Let's invite the artists who have written the most rock music in our dataset. Write a 
query that returns the Artist name and total track count of the top 10 rock bands 

select distinct artist.artist_id, artist.name, count(artist.artist_id) as count_track
from artist
join album2 
on artist.artist_id=album2.artist_id
join track 
on album2.album_id=track.album_id
join genre 
on track.genre_id=genre.genre_id
where genre.name LIKE 'Rock'
group by artist_id, artist.name
order by count_track DESC
limit 10

#8. Return all the track names that have a song length longer than the average song length. 
Return the Name and Milliseconds for each track. Order by the song length with the 
longest songs listed first 

select track.name, track.milliseconds
from track
where track.milliseconds > (

      select avg(track.milliseconds) as avg_mill
      from track)
order by track.milliseconds DESC

#9. Find how much amount spent by each customer on artists? Write a query to return 
customer name, artist name and total spent

WITH best_selling_artist AS (
    select artist.artist_id, artist.name as artist_name, sum(invoice_line.unit_price*invoice_line.quantity) as total_sales
	from invoice_line
    join track
    on invoice_line.track_id=track.track_id
	join album2
    on track.album_id=album2.album_id
    join artist
    on album2.artist_id=artist.artist_id
    Group by 1,2
    Order by 3 DESC
    limit 1 
)
select customer.customer_id, customer.first_name, customer.last_name, best_selling_artist.artist_name, sum(invoice_line.unit_price*invoice_line.quantity) as amount_spent
from customer
join invoice
on customer.customer_id=invoice.customer_id
join invoice_line
on invoice.invoice_id=invoice_line.invoice_id
join track 
on invoice_line.track_id=track.track_id
join album2
on track.album_id=album2.album_id
join best_selling_artist
on album2.artist_id=best_selling_artist.artist_id
group by 1,2,3,4
order by 5 desc

#10. We want to find out the most popular music Genre for each country. We determine the 
most popular genre as the genre with the highest amount of purchases. Write a query 
that returns each country along with the top Genre. For countries where the maximum 
number of purchases is shared return all Genres

WITH popular_genre AS (
     select customer.country, genre.name, genre.genre_id, count(invoice_line.quantity) as purchase, 
     ROW_NUMBER() over(partition by customer.country order by count(invoice_line.quantity) DESC) as RowNo
     from customer 
     join invoice
     on customer.customer_id=invoice.customer_id
     join invoice_line
     on invoice.invoice_id=invoice_line.invoice_id
     join track
     on invoice_line.track_id=track.track_id
     join genre
     on track.genre_id=genre.genre_id
     group by 1,2,3
     order by 1 ASC, 4 DESC 
)
select * from popular_genre where RowNo=1

#11. Write a query that determines the customer that has spent the most on music for each 
country. Write a query that returns the country along with the top customer and how 
much they spent. For countries where the top amount spent is shared, provide all 
customers who spent this amount

WITH customer_country_amount AS (
	select customer.customer_id, customer.first_name, customer.last_name, invoice.billing_country, sum(invoice.total) as total,
    ROW_NUMBER() OVER(partition by invoice.billing_country order by sum(invoice.total) DESC) as RowNo
    from customer
    join invoice 
    on customer.customer_id=invoice.customer_id
    group by 1,2,3,4
    order by 4 ASC, 5 DESC
)
