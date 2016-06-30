# Stanford SQL Mini-Course Query Exercises
## SQL Movie-Rating Query Exercises

You've started a new movie-rating website, and you've been collecting data on reviewers' ratings of various movies. There's not much data yet, but you can still try out some interesting queries. Here's the schema: 

Movie ( mID, title, year, director ) 
English: There is a movie with ID number mID, a title, a release year, and a director. 

Reviewer ( rID, name ) 
English: The reviewer with ID number rID has a certain name. 

Rating ( rID, mID, stars, ratingDate ) 
English: The reviewer rID gave the movie mID a number of stars rating (1-5) on a certain ratingDate. 

Your queries will run over a small data set conforming to the schema. [View the database](https://lagunita.stanford.edu/c4x/DB/SQL/asset/moviedata.html). (You can also [download the schema and data](https://s3-us-west-2.amazonaws.com/prod-c2g/db/Winter2013/files/rating.sql).)

**Instructions:** Each problem asks you to write a query in SQL. When you click "Check Answer" our back-end runs your query against the sample database using SQLite. It displays the result and compares your answer against the correct one. When you're satisfied with your solution for a given problem, click the "Submit" button to check your answer.  

**Important Notes:**

* Your queries are executed using SQLite, so you must conform to the SQL constructs supported by SQLite.
* Unless a specific result ordering is asked for, you can return the result rows in any order.
* You are to translate the English into a SQL query that computes the desired result over all possible databases. All we actually check is that your query gets the right answer on the small sample database. Thus, even if your solution is marked as correct, it is possible that your query does not correctly reflect the problem at hand. (For example, if we ask for a complex condition that requires accessing all of the tables, but over our small data set in the end the condition is satisfied only by Star Wars, then the query "select title from Movie where title = 'Star Wars'" will be marked correct even though it doesn't reflect the actual question.) Circumventing the system in this fashion will get you a high score on the exercises, but it won't help you learn SQL. On the other hand, an incorrect attempt at a general solution is unlikely to produce the right answer, so you shouldn't be led astray by our checking system.

You may perform these exercises as many times as you like, so we strongly encourage you to keep working with them until you complete the exercises with full credit.

---
#####1. Find the titles of all movies directed by Steven Spielberg. 
```sql
select title
from movie
where director = "Steven Spielberg";
```
#####2. Find all years that have a movie that received a rating of 4 or 5, and sort them in increasing order. 
```sql
select distinct year
from rating left join movie using (mid)
where stars > 3
order by year;
```
#####3. Find the titles of all movies that have no ratings. 
```sql
select title
from movie left join rating using (mid)
where rating.mid is null;
```
#####4. Some reviewers didn't provide a date with their rating. Find the names of all reviewers who have ratings with a NULL value for the date. 
```sql
select name
from rating left join reviewer using (rid)
where ratingdate is null;
```
#####5. Write a query to return the ratings data in a more readable format: reviewer name, movie title, stars, and ratingDate. Also, sort the data, first by reviewer name, then by movie title, and lastly by number of stars. 
```sql
select name, title, stars, ratingDate
from rating
    left join reviewer using (rid)
    left join movie using (mid)
order by name, title, stars;
```
#####6. For all cases where the same reviewer rated the same movie twice and gave it a higher rating the second time, return the reviewer's name and the title of the movie. 
```sql
select name, title
from (
	select r2.rid, r2.mid
    from rating r1, rating r2
    where r1.rid = r2.rid and r1.mid = r2.mid and r1.ratingdate < r2.ratingdate and r1.stars < r2.stars
    ) subset
    left join movie using (mid)
    left join reviewer using (rid);
```
#####7. For each movie that has at least one rating, find the highest number of stars that movie received. Return the movie title and number of stars. Sort by movie title. 
```sql
select title, max(stars)
from rating left join movie using (mid)
group by title
order by title;
```
#####8. For each movie, return the title and the 'rating spread', that is, the difference between highest and lowest ratings given to that movie. Sort by rating spread from highest to lowest, then by movie title. 
```sql
select title, max(stars)-min(stars) as rating_spread
from rating left join movie using(mid)
group by title
order by rating_spread desc, title;
```
#####9. Find the difference between the average rating of movies released before 1980 and the average rating of movies released after 1980. (Make sure to calculate the average rating for each movie, then the average of those averages for movies before 1980 and movies after. Don't just calculate the overall average rating before and after 1980.) 
```sql
select avg(star1) - avg(star2)
from (
    select mid, avg(stars) as star1
    from rating left join movie using (mid)
    where year < 1980
    group by mid
    ) as before, (
    select mid,avg(stars) as star2
    from rating left join movie using (mid)
    where year > 1980
    group by mid
    ) as after;
```

## Extras
#####1. Find the names of all reviewers who rated Gone with the Wind. 
```sql
select distinct name
from rating left join reviewer using (rid) left join movie using(mid)
where title = 'Gone with the Wind';
```
#####2. For any rating where the reviewer is the same as the director of the movie, return the reviewer name, movie title, and number of stars. 
```sql
select name, title, stars
from rating r
    left join reviewer using (rid)
    left join movie using (mid)
where name = director;
```
#####3. Return all reviewer names and movie names together in a single list, alphabetized. (Sorting by the first name of the reviewer and first word in the title is fine; no need for special processing on last names or removing "The".) 
```sql
select name from reviewer
union
select title from movie
order by 1;
```
#####4. Find the titles of all movies not reviewed by Chris Jackson. 
```sql
select title
from movie
where mid not in (select mid 
                  from rating
                  where rid = (select rid 
                               from reviewer 
                               where name = 'Chris Jackson'))
```
#####5. For all pairs of reviewers such that both reviewers gave a rating to the same movie, return the names of both reviewers. Eliminate duplicates, don't pair reviewers with themselves, and include each pair only once. For each pair, return the names in the pair in alphabetical order. 
```sql
select distinct re1.name, re2.name 
from rating r1, rating r2, reviewer re1, reviewer re2
where re1.rid = r1.rid and re2.rid = r2.rid
and re1.name < re2.name and r1.mid = r2.mid;
```
#####6. For each rating that is the lowest (fewest stars) currently in the database, return the reviewer name, movie title, and number of stars. 
```sql
select name, title, stars
from rating
    left join movie using (mid)
    left join reviewer using (rid)
where stars = (select min(stars) from rating);
```
#####7. List movie titles and average ratings, from highest-rated to lowest-rated. If two or more movies have the same average rating, list them in alphabetical order. 
```sql
select title, avg(stars) as s
from rating left join movie using (mid)
group by title
order by s desc, title;
```
#####8. Find the names of all reviewers who have contributed three or more ratings. (As an extra challenge, try writing the query without HAVING or without COUNT.) 
```sql
select name
from rating left join reviewer using (rid)
group by name
having count(*) >= 3;
```
#####9. Some directors directed more than one movie. For all such directors, return the titles of all movies directed by them, along with the director name. Sort by director name, then movie title. (As an extra challenge, try writing the query both with and without COUNT.) 
```sql
select title, director
from movie
where director in (
    select director
    from movie
    group by director
    having count(*) > 1)
order by director, title;
```
#####10. Find the movie(s) with the highest average rating. Return the movie title(s) and average rating. (Hint: This query is more difficult to write in SQLite than other systems; you might think of it as finding the highest average rating and then choosing the movie(s) with that average rating.) 
```sql
select title, avg(stars)
from rating left join movie using (mid)
group by title
having avg(stars) = 
    (
    select max(avgStars)
    from 
        (
        select mid, avg(stars) as avgStars
        from rating
        group by mid
        )
    );
```
#####11. Find the movie(s) with the lowest average rating. Return the movie title(s) and average rating. (Hint: This query may be more difficult to write in SQLite than other systems; you might think of it as finding the lowest average rating and then choosing the movie(s) with that average rating.) 
```sql
select title, avg(stars)
from rating left join movie using (mid)
group by title
having avg(stars) = 
    (
    select min(avgs)
    from 
        (
        select mid, avg(stars) as avgs
        from rating
        group by mid
        )
    )
```
#####12. For each director, return the director's name together with the title(s) of the movie(s) they directed that received the highest rating among all of their movies, and the value of that rating. Ignore movies whose director is NULL. 
```sql
Select Movie.title,
       MAX(A.rate),
       Movie.director 
from (Select  rID, mID, AVG(stars) as rate, ratingDate 
                       From Rating Group By mID
                       ) A 
       Inner Join Movie on A.mID=Movie.mID  
where Movie.director is not NULL
group by Movie.director;
```
