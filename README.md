# Netflix Movies and TV Shows Data Analysis using SQL
![Netflix Logo](https://github.com/mina407/Netflix_Sql_Project/blob/main/logo.png)
## Overview 
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.
## Objectives
* Analyze the distribution of content types (movies vs TV shows).
* Identify the most common ratings for movies and TV shows.
* List and analyze content based on release years, countries, and durations.
* Explore and categorize content based on specific criteria and keywords.
* ## Business Problems and Solutions

```sql
-- Count the number of movies vs tv shows
select 
	type , count(*) as total_number 
from netflix 
group by type ;
```

```sql
-- Find the most commen rating for movies and tv show 
with cte as 
		(select 
			type ,
			rating ,
			count(*) ,
			rank() over(partition by type order by count(*) desc) as ranking
		from netflix 
		group by 1,2) 
select type , ranking 
from cte 
where ranking = 1 ;
```
```sql
-- list all the movies released in aspecific year -->2020
select * 
from netflix 
where release_year = 2020 
	and type = 'movie' ;
```

```sql 
-- find the top 5 countries with the most content on netflix 
select unnest(string_to_array(country , ',' )) as new_country , 
	count(*) as total_number
from netflix 
group by 1 
order by total_number desc 
limit 5;
```

```sql
-- Identify the longest movie 

select title , duration 
from netflix 
where duration = (select max(duration) from netflix)
	and type = 'Movie' ;
```

```sql
-- find the content added in the last 5 years 
SELECT * 
FROM netflix
WHERE to_date(date_added, 'mm dd YYYY') >= current_date - interval '5 years' 
order by date_added desc;
```

```sql
-- find all the movies / tv show by director 'rajiv chliaka'
select title , type ,director
from netflix 
where director ilike '%Rajiv Chilaka%' -- take into acount there is more than one director

```

```sql
-- list all tv show more than 5 season 

select type ,
	split_part(duration , ' ' , 1) as season_num
from netflix
where  split_part(duration , ' ' , 1)::numeric > 5
	and type = 'TV Show'
order by 2;
```

```sql
-- Count the number of content items in each gener 

select unnest(string_to_array(listed_in , ','))as gener , count(*) as num_gener 
from netflix 
group by  gener
order by num_gener desc ;
```

```sql
-- find each year and the average number of content release by india on netflix 
-- top five 
select 

	extract(year from to_date(date_added , 'MM DD YYYY')) as year,
	count(*) as yearly_content ,
	round(count(*)::numeric / (select count(*) from netflix where country = 'India')::numeric *100  , 1) ||'%' as Avg_yearly
	
from netflix 
where country = 'India'
group by year
order by yearly_content desc
limit 5;
```

```sql
-- list all movies are documentaries
select type ,title ,listed_in
from netflix 
where 
	listed_in ilike '%documentaries%' ; 
```

```sql
-- fina all content without a director 
select * 
from netflix 
where 
	director is null ;
```

```sql
-- find how many movies actor 'Salman Khan' appeared in last 10 years 

select * 
from netflix 
where  release_year > extract (year from current_date) -10
and casts ilike '%Salman Khan%' ; 
```

```sql
-- find top 10 actors who have appeared in the highest number of movie product in india 
select 
	unnest(string_to_array(casts , ',')) as actor_name , 
	count(*) as Num_of_appearing 
from netflix 
WHERE country ilike '%India%'
group by actor_name
order by Num_of_appearing desc
limit 10 ;

```sql
Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords
with cte as( 
			select * , 
				case 
					when description ilike '%kill%' 
					or description ilike '%Violence%' then 'Bad Content' 
					else 'Good Content'
				end as category 
			from netflix ) 
select category , count(*)
from cte 
group by category 
```
## Findings and Conclusion
* Content Distribution: The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
* Common Ratings: Insights into the most common ratings provide an understanding of the content's target audience.
* Content Categorization: Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.
