# Netflix database project using SQL

![Netflix_logo](https://github.com/SuriyaSureshkumar/Netflix_Database/blob/main/netflix_logo.png)

## **Overview**
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. 
The goal is to extract valuable insights and answer various business questions based on the dataset.

## **Objectives**
Analyze the distribution of content types (movies vs TV shows).
Identify the most common ratings for movies and TV shows.
List and analyze content based on release years, countries, and durations.
Explore and categorize content based on specific criteria and keywords.

## **Dataset**
The data for this project is sourced from the Kaggle dataset:

**Dataset Link:** [Netflix Database](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## **Schema**
```sql
create table Netflix
(
	show_id			varchar(10),
	type 			varchar(10),
	title 			varchar(150),
	director 		varchar(250),
	casts 			varchar(1000),
	country 		varchar(150),
	date_added 		varchar(50),
	release_year 	int,
	rating 			varchar(10),
	duration 		varchar(15) ,
	listed_in		varchar(150),
	description 	varchar(300)
);
```
## **Problem Statements**
### 1. Count the Number of Movies vs TV Shows
```sql
select
	type, count(*) as Total_content 
	from netflix
	group by type;
```

### 2. Find the Most Common Rating for Movies and TV Shows
```sql
select
	type, rating
	from
	(
		select
			type, rating, count(rating),
			rank() over (partition by type order by count(rating) desc) as Ranking
			from netflix
			group by 1,2
			-- order by 3 desc;
	) as t1
where Ranking = 1
```
### 3. List All Movies Released in a Specific Year (e.g., 2020)
```sql
select title,type,release_year from netflix
where 
	release_year = 2020 and 
	type = 'Movie';
```
### 4. Find the Top 5 Countries with the Most Content on Netflix
```sql
select 
	unnest(STRING_TO_ARRAY(country,',')) as new_country, 
	count(*) as total
from netflix
group by 1
order by 2 desc
limit 5;
```
### 5. Identify the Longest Movie
```sql
select title, duration
from netflix
where 	type = 'Movie' and 
		duration = (select max(duration) from netflix)
```
### 6. Find Content Added in the Last 5 Years
```sql
select 
		title, 
		to_date(date_added,'Month DD, YYYY'), 
		release_year
from netflix
where to_date(date_added,'Month DD, YYYY') >= current_date - interval '5 years'
order by to_date
```
### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'
```sql
select 
		title,
		type,
		director
from netflix
where director ilike '%Rajiv Chilaka%';
```
OR
```sql
select 
		*
	from
		(select title, type,
			unnest(string_to_array(director,',')) as director_name
			from netflix
		) as t
where director_name = 'Rajiv Chilaka';
```
### 8. List All TV Shows with More Than 5 Seasons
```sql
select
		title,
		type,
		duration
from netflix
where type = 'TV Show' and duration > '5%'
order by duration desc;
```
OR
```sql
select
		title,
		type,
		split_part(duration,' ',1) as seasons
from netflix
where type = 'TV Show' and split_part(duration,' ',1) > '5'
order by seasons desc;
```
### 9. Count the Number of Content Items in Each Genre
```sql
select 
	unnest(string_to_array(listed_in,',')) as Genre,
	count(*) as Total
from netflix
group by Genre
order by Total desc
```
### 10.Find each year and the average numbers of content release in India on netflix and return top 5 year with highest avg content release!
```sql
select 
		extract (year from(to_date(date_added,'month dd,yyyy'))) as added_year,
		count(*),
		round(count(*)::numeric/(select count(*) from netflix where country = 'India')::numeric * 100,2) as Average
from netflix
where country = 'India'
group by 1
order by 2 desc
limit 5;
```
### 11. List All Movies that are Documentaries
```sql
select
	title,
	type,
	listed_in
from netflix
where 
	type = 'Movie' 
	and
	listed_in like '%Documentaries%'
```
### 12. Find All Content Without a Director
```sql
select * from netflix
where director is null;
```
### 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years
```sql
select 
	title, 
	type,
	casts,
	release_year
	from netflix
	where 
		casts ilike '%salman khan%'
		and
		release_year >= extract(year from current_date) - 10
	order by release_year desc
```
### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India
```sql
select
	unnest(string_to_array(casts,',')) as Actors,
	count(title) as movie_count
	from netflix
	where type = 'Movie' and country ilike '%india%'
	group by Actors
	order by movie_count desc
	limit 10;
```
### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords. Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.
```sql
select
	case
		when description ilike '%kill%' or description ilike '%violence%'
		then 'Bad-Content'
		else 'Good-Content'
		end as movie_type,
	count(*) as total_count
	from netflix
	group by 1
	order by 2 desc;
```
