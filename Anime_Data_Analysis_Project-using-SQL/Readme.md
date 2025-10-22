# ⛩️Anime Data Analysis Project using SQL

![]( https://github.com/AdhavanHero/MY-SQL-Projects/blob/main/Anime_Data_Analysis_Project-using-SQL/wp5104277.jpg )

## Project Overview
This project involves a detailed analysis of a comprehensive anime dataset. The primary goal is to leverage MySQL to uncover key performance indicators, audience trends, and the impact of production factors (like source material and studios) on anime success.

## 💾 Dataset and Schema

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Anime Dataset](https://www.kaggle.com/datasets/duongtruongbinh/manga-and-anime-dataset)

### Schema
```sql
CREATE TABLE Animedata(
    -- Primary Descriptive Fields
    AnimeID INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    Title VARCHAR(500),
    -- Performance Metrics
    Score FLOAT,          -- Use FLOAT for decimal scores (e.g., 9.14)
    Vote INT,             -- Total votes
    Ranked INT,           -- Integer rank (1, 2, 3...)
    Popularity INT,       -- Integer popularity rank (1, 2, 3...)
    -- Timing and Status
    Episodes VARCHAR(50), -- Kept as VARCHAR to handle 'Unknown' or 'N/A'
    Status VARCHAR(50),
    Aired VARCHAR(200),
    Premiered VARCHAR(50),
    -- Production and Licensing (These fields contain complex lists/strings, so use large VARCHAR)
    Producers VARCHAR(2000),
    Licensors VARCHAR(2000),
    Studios VARCHAR(1000),
    Source VARCHAR(50),     -- e.g., 'Manga', 'Original', 'Light novel'
    Duration VARCHAR(100),
    -- Rating
    Rating VARCHAR(100)     -- e.g., 'R - 17+'
);
```

## 🎯 Anime Performance and Trend Analysis – SQL Solutions

The analysis is broken down into three logical parts, addressing performance, production factors, and time-based trends.

Part 1: Performance and Aggregations
```sql
### 1.Find the top 5 highest-rated anime with over 100,000 votes.

SELECT Vote, AnimeID, Title
FROM animedata where vote > 100000
order by 1 desc limit 5;
```
### # 5 Find how many titles have a score below the overall average score.
```sql
seLECT COUNT(*) FROM animedata WHERE Score < (SELECT AVG(Score) FROM animedata WHERE Score IS NOT NULL);
```
### 9 List anime with high scores (8.5+) but low votes (<5,000) to identify underrated gems.
```sql
SELECT Title, Score, Vote, Premiered FROM animedata WHERE Score >= 8.5 AND Vote < 5000 ORDER BY Vote;
```
### 15 Find the top 5 highest-ranked anime with fewer than 10,000 votes (critically acclaimed but niche titles).
  ```sql
  SELECT
  Title,
  ranked,
  Vote,
  Score
FROM
  animedata
WHERE Vote < 10000        -- Filter 1: Low popularity by vote count
AND Ranked IS NOT NULL -- Filter 2: Must have a critical Rank
ORDER BY  Ranked ASC            -- Orders by the best (lowest) Rank number
LIMIT 5;
```
## Part 2: Production, Source, and Audience Impact

### 3 Most Common Source: What is the most frequent source material
```sql
select source, count(*) as Title_Count from Animedata group by 1 order by 2 desc limit 1
```
### 4 Compare the average score between the most restrictive and most general rating categories.
```sql
SELECT
  Rating,
  AVG(Score) AS Avg_Score
FROM
  animedata
WHERE
  Rating IN ('R - 17+ (violence & profanity)', 'PG-13 - Teens 13 or older')
  AND Score IS NOT NULL
GROUP BY
  Rating;
```
### 6 Identify the top 3 studios with the highest average scores (only include studios with 10+ titles).
```sql
SELECT
  Studios,
  COUNT(*) AS Title_Count,
  AVG(Score) AS Avg_Score
FROM
  animedata
WHERE
  Studios NOT IN ('None found, add some', 'None')
  AND Score IS NOT NULL
GROUP BY
  Studios
HAVING
  Title_Count >= 10
ORDER BY
  Avg_Score DESC
LIMIT 3;
```
### 8 Find which Source material has the highest total vote count (popularity indicator)
```sql
SELECT Source, SUM(Vote) AS Total_Votes FROM animedata GROUP BY Source ORDER BY Total_Votes DESC LIMIT 3;
```
### 10 Find the most common Source material used by the top-rated studio identified in Task 6.
```sql
SELECT
  Source,
  COUNT(*) AS Source_Count
FROM
  animedata
WHERE
  -- 1. Finds the specific studio name determined as the highest-rated in a subquery
  Studios = (
    SELECT
      Studios
    FROM
      animedata
    WHERE
      Studios NOT IN ('None found, add some', 'None')
      AND Score IS NOT NULL
    GROUP BY
      Studios
    HAVING
      COUNT(*) >= 10 -- Only studios with 10+ titles
    ORDER BY
      AVG(Score) DESC
    LIMIT 1 -- Selects only the single best studio
  )
GROUP BY
  Source
ORDER BY
  Source_Count DESC
LIMIT 1;
```
### 11 Find which Source material generates the highest overall popularity (lowest average rank number).
```sql
SELECT Source, AVG(Popularity) AS Avg_Popularity_Rank, COUNT(*) AS Title_Count FROM animedata 
WHERE Source IS NOT NULL 
GROUP BY Source HAVING 
Title_Count > 50 
ORDER BY Avg_Popularity_Rank ASC LIMIT 5;
```
## Part 3: Trend and Format Comparison

####2 Compare the average score of short-form anime (1–12 episodes) vs long-form anime (50+ episodes).
```sql
SELECT
  CASE
    WHEN CAST(Episodes AS UNSIGNED) BETWEEN 1 AND 12 THEN 'Short Anime'
    WHEN CAST(Episodes AS UNSIGNED) >= 50 THEN 'Long Anime' #cast used seperate txt like 12 from te varchar
    ELSE 'Other'
  END AS Anime_Type,
  AVG(Score) AS Average_Score
FROM animedata
WHERE Score IS NOT NULL
  AND Episodes != 'Unknown'
  AND Episodes != '-'
GROUP BY Anime_Type; 
```
### 7 Analyze annual release trends and average scores for years with 50+ releases.
```sql
SELECT
  RIGHT(Premiered, 4) AS Premier_Year,
  COUNT(*) AS Annual_Releases,
  AVG(Score) AS Annual_Avg_Score
FROM
  animedata
SELECT
  RIGHT(Premiered, 4) AS Premier_Year,
  COUNT(*) AS Annual_Releases,
  AVG(Score) AS Annual_Avg_Score
FROM
  animedata
WHERE
  Premiered IS NOT NULL
  AND Score IS NOT NULL
GROUP BY
  Premier_Year
HAVING
  Annual_Releases >= 5 
ORDER BY
  Premier_Year DESC;
```

### 12 Compare the average score of anime with exactly 1 episode (typically movies/OVAs) versus the average score of all other anime titles.
```sql
SELECT
  CASE
    WHEN Episodes = '1' THEN 'Movie/OVA'
    ELSE 'Series/Other'
  END AS Format,
  AVG(Score) AS Avg_Score
FROM
  animedata
WHERE
  Score IS NOT NULL
  AND Episodes REGEXP '^[0-9]+$' -- Ensures 'Episodes' contains only numeric values
GROUP BY
  Format;
  ```
### 13 Calculate the average score for long-running anime (100+ episodes).
  ```sql
  SELECT
  COUNT(Title) AS Long_Running_Count,
  AVG(Score) AS Avg_Score_Over_100_Eps
FROM
  animedata
WHERE
  Score IS NOT NULL
  AND Episodes REGEXP '^[0-9]+$' -- Ensures 'Episodes' contains only numbers
  AND CAST(Episodes AS UNSIGNED) >= 100; -- Converts Episodes to a number for comparison
```  
### 14 Segment the data by decade (e.g., 2000s, 2010s, 2020s) and find the decade with the highest average score.
```sql
SELECT
  FLOOR(RIGHT(Premiered, 4) / 10) * 10 AS Decade_Start_Year,
  AVG(Score) AS Avg_Score
FROM
  animedata
WHERE
  Premiered IS NOT NULL
  AND LENGTH(Premiered) = 9  -- Still needed to ensure the RIGHT() pulls a clean year
  AND Score IS NOT NULL
GROUP BY
  Decade_Start_Year
ORDER BY
  Avg_Score DESC;
```
