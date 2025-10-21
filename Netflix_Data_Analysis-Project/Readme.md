# Netflix Movies and TV Shows Data Analysis using SQL

![](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows

```sql
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY 1;
```

**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the Most Common Rating for Movies and TV Shows
```sql
SELECT
    type,
    rating AS most_frequent_rating # Final result: Show the type and its most frequent rating.
FROM
    (
        # Subquery: Calculate counts and determine the rank for every rating within each type.
        SELECT
            type,
            rating,
            # Rank based on the count. Highest count gets rank 1.
            RANK() OVER (
                PARTITION BY type           # 1. Group the ranking by content type.
                ORDER BY COUNT(*) DESC      # 2. Order by frequency (count) descending.
            ) AS rnk                        # Alias the rank result as 'rnk'.
        FROM
            netflix
        GROUP BY
            type,
            rating # Count the occurrences of each rating/type combination.
    ) AS RankedRatings
WHERE
    rnk = 1; # Filter: Only keep the ratings that achieved Rank 1 (the most frequent).
```
### 3. List All Movies Released in a Specific Year (e.g., 2020)
```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```
### 4. Find the Top 5 Countries with the Most Content on Netflix 
```sql
SELECT
    country,                   # Select the country name.
    COUNT(*) AS total_content  # Count the number of rows (titles) for each country and label it.
FROM
    netflix
# Note: Filter out entries where the country field is NULL or empty for cleaner results.
WHERE
    country IS NOT NULL AND TRIM(country) <> ''
GROUP BY
    country                    # Group the counts by country.
ORDER BY
    total_content DESC         # Sort the results so the highest counts (most content) appear first.
LIMIT 5;                       # Restrict the final output to only the top 5 rows.
```
### 5. Identify the Longest Movie
```sql
SELECT
    title, # Select the movie title
    country,
    duration     # Select the original duration string (e.g., '150 min')
FROM
    netflix
WHERE
    type = 'Movie'  # Only consider entries where the type is 'Movie'
ORDER BY
    # Convert the duration string (e.g., '150 min') to a number for correct sorting, The REPLACE function removes the ' min' part so we can cast the remaining string to an integer.
    CAST(REPLACE(duration, ' min', '') AS SIGNED) DESC
LIMIT 1;            # Select only the top result (the movie with the longest duration)
```
### 6. Find Content Added in the Last 5 Years
```sql
SELECT title, date_added
FROM netflix
WHERE STR_TO_DATE(date_added, '%M %d, %Y') >= DATE_SUB((
    SELECT MAX(STR_TO_DATE(date_added, '%M %d, %Y')) FROM netflix
), INTERVAL 5 YEAR)
ORDER BY STR_TO_DATE(date_added, '%M %d, %Y') DESC;
```
### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'
```sql
SELECT
    title,
    type,
    release_year
FROM
    netflix
WHERE
    director = 'Rajiv Chilaka'
ORDER BY
    release_year DESC;

#unfortunately no direcor by that name fount
```
### 8. List All TV Shows with More Than 5 Seasons
```sql
SELECT
    title,
    duration,
    release_year
FROM
    netflix
WHERE
    type = 'TV Show'
    -- Extracts the number before 'Seasons'. SQL will implicitly compare this numeric string to the integer 5.
    AND SUBSTRING_INDEX(duration, ' ', 1) > 5
ORDER BY
    -- Adds 0 to the extracted number to explicitly force a numeric sort (without using CAST).
    2 DESC;
```
### 9. Count the Number of Content Items in Each Genre
```sql
SELECT
    listed_in AS Genre,
    COUNT(*) AS Content_Count
FROM
    netflix
GROUP BY 1
ORDER BY 2 DESC;
```
### 10.Find each year and the average numbers of content release in India on netflix.
```sql
SELECT
    release_year,
    COUNT(show_id) AS total_content_for_india
FROM netflix
WHERE
    country LIKE '%India%'
    AND release_year IS NOT NULL
GROUP BY
    release_year
ORDER BY
    release_year;
```
### 11. List All Movies that are Documentaries
```sql
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries';
```
### 12. Find All Content Without a Director
```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```
### 13 Find How Many Movies Actor 'vijay sethupathi' Appeared in the Last 10 Years
```sql
SELECT * 
FROM netflix
WHERE casts LIKE '%vijay sethupathi%'
AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```
### 14. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords
```sql
SELECT
    category,
    COUNT(*) AS content_count
FROM (
    SELECT
        CASE
            WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;
```
