# Stanford SQL Course

You've started a new movie-rating website, and you've been collecting data on reviewers' ratings of various movies. There's not much data yet, but you can still try out some interesting queries. Here's the schema:    

Movie ( mID, title, year, director )    
English: There is a movie with ID number mID, a title, a release year, and a director. 
   
Reviewer ( rID, name )    
English: The reviewer with ID number rID has a certain name.    

Rating ( rID, mID, stars, ratingDate )    
English: The reviewer rID gave the movie mID a number of stars rating (1-5) on a certain ratingDate.    

Your queries will run over a small data set conforming to the schema. [View the database](https://lagunita.stanford.edu/c4x/DB/SQL/asset/moviedata.htmlv). (You can also [download the schema and data](https://s3-us-west-2.amazonaws.com/prod-c2g/db/Winter2013/files/rating.sql).)    


**Movie**

|mID	|title	|year	|director|
| --- | --- | --- | --- |
|101|	Gone with the Wind	|1939	|Victor Fleming|
|102|	Star Wars	|1977	|George Lucas|
|103|	The Sound of Music	|1965	|Robert Wise|
|104|	E.T.	|1982|	Steven Spielberg|
|105|	Titanic	|1997	|James Cameron|
|106|	Snow White|	1937	|\<null>|
|107|	Avatar	|2009|	James Cameron|
|108|	Raiders of the Lost Ark	|1981	|Steven Spielberg|

**Reviewer**

rID|	name
--- | ---
201	|Sarah Martinez
202|	Daniel Lewis
203	|Brittany Harris
204	|Mike Anderson
205	|Chris Jackson
206	|Elizabeth Thomas
207	|James Cameron
208	|Ashley White

**Rating**


rID	|mID|	stars|	ratingDate
--- | --- |--- | ---
201	|101|	2|	2011-01-22
201	|101|	4|	2011-01-27
202	|106|	4|	\<null>|
203|103|	2|	2011-01-20
203	|108|	4|	2011-01-12
203	|108|	2|	2011-01-30
204	|101|	3|	2011-01-09
205	|103|	3|	2011-01-27
205	|104|	2|	2011-01-22
205	|108|	4|	\<null>
206	|107|	3|	2011-01-15
206	|106|	5|	2011-01-19
207	|107|	5|	2011-01-20
208	|104|	3|	2011-01-02
#### Instructions: 
Each problem asks you to write a query in SQL. When you click "Check Answer" our back-end runs your query against the sample database using SQLite. It displays the result and compares your answer against the correct one. When you're satisfied with your solution for a given problem, click the "Submit" button to check your answer.    

#### Important Notes:   

* Your queries are executed using SQLite, so you must conform to the SQL constructs supported by SQLite.   
* Unless a specific result ordering is asked for, you can return the result rows in any order.   
* You are to translate the English into a SQL query that computes the desired result over all possible databases. All we actually check is that your query gets the right answer on the small sample database. Thus, even if your solution is marked as correct, it is possible that your query does not correctly reflect the problem at hand. (For example, if we ask for a complex condition that requires accessing all of the tables, but over our small data set in the end the condition is satisfied only by Star Wars, then the query "select title from Movie where title = 'Star Wars'" will be marked correct even though it doesn't reflect the actual question.) Circumventing the system in this fashion will get you a high score on the exercises, but it won't help you learn SQL. On the other hand, an incorrect attempt at a general solution is unlikely to produce the right answer, so you shouldn't be led astray by our checking system.
   
You may perform these exercises as many times as you like, so we strongly encourage you to keep working with them until you complete the exercises with full credit.

#### Q1  (1/1 point)
Find the titles of all movies directed by Steven Spielberg. 

	select title from Movie
	where director='Steven Spielberg'
	
#### Q2  (1/1 point)
Find all years that have a movie that received a rating of 4 or 5, and sort them in increasing order. 

	select distinct year
	from rating,movie
	where movie.mid=rating.mid
	and (stars=4 or stars=5)
	order by year ASC;
	
#### Q3  (1/1 point)
Find the titles of all movies that have no ratings. 

	select title
	from movie left join rating
	on rating.mid=movie.mid
	where stars is null

#### Q4  (1/1 point)
Some reviewers didn't provide a date with their rating. Find the names of all reviewers who have ratings with a NULL value for the date. 

	select distinct name
	from rating ,reviewer
	where rating.rid=reviewer.rid
	and ratingDate is null


#### Q5  (1/1 point)
Write a query to return the ratings data in a more readable format: reviewer name, movie title, stars, and ratingDate. Also, sort the data, first by reviewer name, then by movie title, and lastly by number of stars. 

	select name,title,stars,ratingDate
	from movie,rating,reviewer
	where movie.mid=rating.mid 
	and rating.rid=reviewer.rid
	order by name,title,stars;


#### Q6  (1/1 point)
For all cases where the same reviewer rated the same movie twice and gave it a higher rating the second time, return the reviewer's name and the title of the movie. 

	select r.name,m.title
	from rating r1,rating r2,movie m,reviewer r
	where r1.mid=r2.mid and r1.rid=r2.rid
	and m.mid=r1.mid and r.rid=r1.rid
	and r2.ratingDate>r1.ratingDate
	and r2.stars>r1.stars

#### Q7  (1/1 point)
For each movie that has at least one rating, find the highest number of stars that movie received. Return the movie title and number of stars. Sort by movie title. 

	select m.title,max(r.stars)
	from movie m
	left join rating r
	on m.mid=r.mid
	where r.rid is not null
	group by m.mid
	order by m.title


#### Q8  (1/1 point)
For each movie, return the title and the 'rating spread', that is, the difference between highest and lowest ratings given to that movie. Sort by rating spread from highest to lowest, then by movie title. 

	select m.title, (max(stars)-min(stars))as rating_spread
	from movie m
	join rating r using (mid)
	group by m.mid
	order by rating_spread desc, m.title


#### Q9  (1/1 point)
Find the difference between the average rating of movies released before 1980 and the average rating of movies released after 1980. (Make sure to calculate the average rating for each movie, then the average of those averages for movies before 1980 and movies after. Don't just calculate the overall average rating before and after 1980.) 

	select avg(before.stars) - avg(after.stars)
	from (select avg(r1.stars) as stars
	from rating r1
	left join movie m1 on r1.mid=m1.mid
	where m1.year<'1980'
	group by r1.mid) as before,
	(select avg(r1.stars) as stars
	from rating r1
	left join movie m1 on r1.mid=m1.mid
	where m1.year>'1980'
	group by r1.mid) as after
	
	
### Link of My [Statement of Accomplish](https://prod-cert-bucket.s3.amazonaws.com/downloads/1234116783874e7fa6ef055c9f5dc91e/Statement.pdf)