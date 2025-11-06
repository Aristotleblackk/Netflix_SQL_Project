# Netflix Movies and Tv Shows data analysis using SQL
![Netflix Logo](https://github.com/Aristotleblackk/Netflix_SQL_Project/blob/main/Netflix%20logo.jpg)

## Objective 
- Analyze the distribution of content types (Movies vs TV Shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and duration.
- Explore and categorize content based on Specific criteria and keywords

## Data set

The data for this project is sourced from Kaggle

-**Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema 

```sql
--Netflix project--

CREATE TABLE netflix
(       show_id VARCHAR(6),
		TYPE VARCHAR(10),
		title VARCHAR(150),	
		director VARCHAR(208),	
		casts VARCHAR(1000),	
		country VARCHAR(150),
		date_added VARCHAR(50),	
		release_year INT,	
		rating VARCHAR (10),	
		duration VARCHAR (15),	
		listed_in VARCHAR (79),	
		description VARCHAR (250)
)
CREATE TABLE Netflix_mod 
(show_id VARCHAR(6),
		TYPE VARCHAR(10),
		title VARCHAR(150),	
		director VARCHAR(208),	
		casts VARCHAR(1000),	
		country VARCHAR(150),
		date_added VARCHAR(50),	
		release_year INT,	
		rating VARCHAR (10),	
		duration VARCHAR (15),	
		listed_in VARCHAR (79),	
		description VARCHAR (250)
);

INSERT INTO netflix_mod
SELECT *
FROM netflix;
```
## Business Problems and Solutions

### 1. Count the number of Movies vs TV Shows
SELECT type, 
COUNT(*)
FROM netflix 
GROUP BY 1;

### 2. Find the most common rating for movies and TV shows
SELECT
	type,
	rating
FROM
(SELECT 
		 type, 
		 rating, 
		 COUNT (*),
		 RANK () OVER(PARTITION BY type ORDER BY COUNT (*) DESC) as ranking	 
FROM netflix
	 GROUP BY 1,2
	 ORDER BY 1,3 desc) as T1
WHERE ranking = 1;

### 3. List all movies released in a specific year (2021)

SELECT * 
FROM netflix
WHERE TYPE = 'Movie' AND release_year = '2021';

--4. Find the top 5 countries with the most content on netflix

SELECT
		UNNEST (STRING_TO_ARRAY(country, ',')) as new_country, 
		COUNT(show_id) as total_content
	FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

### 5. Identify the longest movie
SELECT * FROM netflix
WHERE
	TYPE = 'Movie'
	AND 
	duration = (SELECT MAX(duration) FROM netflix);
	
### 6. Find the content added in the last 5 years

SELECT *
FROM netflix
WHERE 
TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'

### 7. Find all the movies and  TV shows by director 'Rajiv Chilaka'

SELECT * 
FROM netflix
WHERE director ILIKE '%Rajiv Chilaka%';

### 8. List all TV shows with more than 5 seasons

SELECT*
FROM netflix
WHERE 
TYPE = 'TV Show'
AND 
SPLIT_PART(duration, ' ', 1)::numeric > 5 

### 9. Count the number of content items in each genre
SELECT
 UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
 COUNT (show_id) as total_content
 FROM netflix
 GROUP BY 1
 ORDER BY 2 DESC

### 10. Top ten genres on Netflix
SELECT 
	 UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	 COUNT (show_id) as total_content
	 FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10

### (BONUS)11. The top 10 Directors who have produced the most content
SELECT
	 UNNEST(String_to_array(director, ',')) as director,
	 COUNT (show_id) as total_content 
	 FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10

### 12. Find each year and the average numbers of content released by United states on Netflix.
--return top 5 year with highest average content release

Select 
	EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) as date,
	COUNT(*) as yearly_content,
	ROUND(
	COUNT (*)::numeric/(SELECT COUNT(*)FROM netflix where country = 'United States')::numeric*100,2 
	)as avg_content_per_year 
FROM netflix
WHERE country = 'United States' AND date_added IS NOT NULL
GROUP BY 1
ORDER BY 3 desc 
### 13. List all movies that are documentaries

SELECT *, 
UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre
FROM netflix
WHERE listed_in= 'Documentaries'

--OR

SELECT *
FROM netflix
WHERE listed_in ILIKE '%documentaries%'

### 14.List all content without a director
SELECT *
FROM netflix
WHERE director IS NULL

### 15. Find how many movies actress 'Meryl Streep' appeared in the last 10 years

SELECT * 
FROM netflix
WHERE 
	casts ILIKE '%Meryl Streep%'
	AND
	release_year > EXTRACT(YEAR FROM CURRENT_DATE)-10

### 16. Find the top 10 actors who have appeared in the highest number of movies produced in India
SELECT
	 UNNEST(String_to_array(casts, ',')) AS actors,
	 COUNT (*) as total_content 
	 FROM netflix
WHERE country ILIKE '%United States%'
	GROUP BY 1
	ORDER BY 2 desc
	LIMIT 10
	
### 17. Categorize the content based on the prescence of the keywords 'kill' and 'violence' in the description field
-- Label content containing these keywords as and all other content as 'Good'
--Count how many items fall into each category 
WITH New_Table
AS
(
SELECT *,
CASE 
		WHEN 
		description ILIKE '%kill%' or
		description ILIKE '%violence%'
		THEN 'Bad_Content'
		ELSE 'Good Content'
	END category
FROM netflix
)
SELECT
	category,
	COUNT(*) as total_content
FROM new_table
GROUP BY 1
