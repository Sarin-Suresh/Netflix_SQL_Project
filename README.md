# Netflix Movies and TV Shows Data Analysis Using SQL

![](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## Overview
This project presents an in-depth SQL-based analysis of Netflix's movies and TV shows data, aimed at uncovering valuable insights and addressing key business questions. This README outlines the project's objectives, challenges, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution between content types (Movies vs. TV Shows).
- Identify the most common content ratings.
- Explore content by release years, countries, and durations.
- Classify content based on specific criteria and keywords.

## Dataset

The data used in this project is sourced from Kaggle:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
-- Netflix Project

CREATE TABLE netflix (
	show_id VARCHAR(6),
	type VARCHAR(10),
	title VARCHAR(150),
	director VARCHAR(208),
	casts VARCHAR(1000),
	country VARCHAR(150),
	date_added VARCHAR(50),
	release_year INT,
	rating VARCHAR(10),
	duration VARCHAR(15),
	listed_in VARCHAR(150),
	description VARCHAR(250)
);

SELECT * FROM netflix;
```

## 15 Business Problems & Solutions

### 1. Count the Number of Movies vs. TV Shows
```sql
SELECT 
	type,
	COUNT(*) as total_content
FROM netflix
GROUP BY 1;
```

### 2. Identify the Most Common Rating for Movies and TV Shows
```sql
SELECT 
	type,
	rating
FROM
(
	SELECT 
		type,
		rating,
		COUNT(*),
		RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) as ranking
	FROM netflix
	GROUP BY 1, 2
) as t1
WHERE ranking = 1;
```

### 3. List All Movies Released in a Specific Year (e.g., 2020)
```sql
SELECT * 
FROM netflix
WHERE 
	type = 'Movie'
	AND release_year = 2020;
```

### 4. Top 5 Countries with the Most Content on Netflix
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(country, ',')) as new_country,
	COUNT(show_id) as total_content
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```

### 5. Identify the Longest Movie Duration
```sql
SELECT 
	title, 
	SUBSTRING(duration, 1, POSITION('m' IN duration)-1)::int duration
FROM netflix
WHERE 
	type = 'Movie' 
	AND duration IS NOT NULL
ORDER BY 2 DESC
LIMIT 1;
```

### 6. Find Content Added in the Last 5 Years
```sql
SELECT *
FROM netflix
WHERE 
	TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 Years';
```

### 7. Find All Movies/TV Shows Directed by 'Rajiv Chilaka'
```sql
SELECT 
	type,
	director,
	title
FROM netflix 
WHERE director ILIKE '%Rajiv Chilaka%';
```

### 8. List All TV Shows with More than 5 Seasons
```sql
SELECT 
	type,
	duration
FROM netflix
WHERE 
	type = 'TV Show'
	AND SPLIT_PART(duration, ' ', 1)::numeric > 5;
```

### 9. Count the Number of Content Items in Each Genre
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	COUNT(show_id) as total_content
FROM netflix
GROUP BY 1;
```

### 10. Find the Top 5 Years with the Highest Average Content Release in India
```sql
SELECT
	EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) as year, 
	COUNT(*),
	COUNT(*)::numeric / (SELECT COUNT(*) FROM netflix WHERE country = 'India') * 100 as avg_content_per_year 
FROM netflix
WHERE country = 'India'
GROUP BY 1;
```

### 11. List All Movies That Are Documentaries
```sql
SELECT * 
FROM netflix
WHERE listed_in ILIKE '%documentaries%';
```

### 12. Find All Content Without a Director
```sql
SELECT 
	title, 
	director
FROM netflix
WHERE director IS NULL;
```

### 13. Find the Number of Movies Actor 'Salman Khan' Appeared in Over the Last 10 Years
```sql
SELECT * 
FROM netflix
WHERE 
	casts ILIKE '%Salman Khan%'
	AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

### 14. Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India
```sql
SELECT
	UNNEST(STRING_TO_ARRAY(casts, ',')) as actors,
	COUNT(*) as total_content
FROM netflix
WHERE country ILIKE '%india%'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```

### 15. Categorize Content Based on Keywords in the Description
```sql
WITH new_table AS
(
	SELECT
		CASE
			WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad_content' 
			ELSE 'Good Content'
		END as category
	FROM netflix
)
SELECT
	category,
	COUNT(*) as total_content
FROM new_table
GROUP BY 1;
```

**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

## Findings and Conclusion

- **Content Distribution:** The dataset includes a wide variety of movies and TV shows across different ratings and genres.
- **Common Ratings:** Identifying common ratings provides insights into the target audience.
- **Geographical Insights:** Top countries and average content releases in India reveal regional content trends.
- **Content Categorization:** Classifying content by keywords aids in understanding Netflix’s content nature.

This analysis offers a detailed overview of Netflix’s content and supports informed decisions regarding content strategy.
