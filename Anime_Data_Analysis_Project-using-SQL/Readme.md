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
