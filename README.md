# App Store Data Analysis for Optimal Ad Placement

### Overview
A comprehensive analysis of the Apple App Store dataset aimed at assisting developers in determining optimal ad strategies based on app categories, pricing, and user ratings.

### Data Source
The datasets used in this project are from Kaggle and include detailed information about apps on the Apple App Store, including descriptions, genres, ratings, and more.

### Objective
To explore which app categories are most popular, set optimal pricing strategies, and identify factors that maximize user ratings.

### Methodology
**Data Importing:** Imported datasets using SQL Lite Online.\
**Data Preprocessing:** Combined multiple data chunks into a single dataset.\
**Exploratory Data Analysis (EDA):** Conducted initial analysis to understand the dataset's structure, unique app counts, and missing values.\
**In-Depth Analysis:** Performed detailed SQL queries to analyze app genres, ratings, pricing strategies, and description lengths.
### Key Findings
**Paid vs Free Apps:** Paid apps tend to have higher ratings compared to free apps.\
**Language Support:** Apps supporting 10-30 languages have higher user ratings.\
**Genre Analysis:** Finance and Book apps present opportunities for quality app development.\
**Description Length:** A positive correlation between the length of the app description and user ratings was observed.\
**Competition in Genres:** Games and Entertainment apps face high competition.
### Final Recommendations
#### Based on the analysis, the following recommendations are proposed for developers to optimize ad strategies:
- Consider developing paid apps for higher user engagement.
- Focus on language support, particularly in the 10-30 language range.
- Explore opportunities in Finance and Book genres.
- Emphasize detailed app descriptions.
- Note the competitive landscape in Games and Entertainment categories.
  
### SQL Queries
#### Importing dataset
```sql
-- combined small chuncks of data into one whole data set

CREATE TABLE applestore_description_combined AS

SELECT * FROM appleStore_description1

UNION ALL

SELECT * FROM appleStore_description2

UNION ALL

SELECT * FROM appleStore_description3

UNION ALL

SELECT * FROM  appleStore_description4;
```
#### Exploratory Data analysis (EDA)
Check the number of unique apps in both tablesAppleStore
```sql
SELECT COUNT(DISTINCT id) AS UNIQUEAPPIDs
FROM AppleStore

SELECT COUNT(DISTINCT id) AS UNIQUEAPPIDs
FROM applestore_description_combined
```
check for any missing values in key fieldsAppleStore
```sql
SELECT COUNT(*) as MissingValues
FROM AppleStore
WHERE track_name is NULL OR user_rating IS NULL OR prime_genre IS NULL

SELECT COUNT(*) as MissingValues
FROM applestore_description_combined
WHERE app_desc is NULL
```
Find out the number of apps per genre 
```sql
SELECT prime_genre, COUNT(*) as NumApps
FROM AppleStore
GROUP BY prime_genre
ORDER BY NumApps DESC
```
Get an overview of the apps' ratings 
```sql
SELECT min(user_rating) AS  MinRating,
			 max(user_rating) AS  MaxRating,
             avg(user_rating) AS  AvgRating
FROM AppleStore
```
#### Data Analysis
Determine whether paid apps have higher ratings than free apps 
```sql
SELECT CASE
						WHEN price > 0 THEN 'Paid'
                        ELSE 'Free'
               END as App_Type,
               avg(user_rating) as Avg_Rating
FROM AppleStore
GROUP BY App_Type
```
Check if apps with more supported languages have higher ratingsAppleStore
```sql
SELECT CASE
						WHEN lang_num < 10 THEN '<10 languages'
                        WHEN lang_num BETWEEN 10 AND 30 THEN '10-30 languages'
                        ELSE '>30 languages'
              END AS language_bucket,
              avg(user_rating) AS Avg_Rating
FROM AppleStore
GROUP BY language_bucket
ORDER BY Avg_Rating DESC
```
check genres with low ratingsAppleStore
```sql
SELECT prime_genre,
			  avg(user_rating) as Avg_Rating
FROM AppleStore
GROUP BY prime_genre
ORDER BY Avg_Rating ASC
LIMIT 10
```
check if there is correlation between the length of the app description and the user rating
```sql
SELECT CASE
						WHEN length(b.app_desc) < 500 THEN 'Short'
                        WHEN length(b.app_desc)  BETWEEN 500 AND 1000 THEN 'Medium'
                        ELSE 'Long'
              END as description_length_bucket,
              avg(a.user_rating) AS average_rating

FROM 
			AppleStore as a 
JOIN 
			applestore_description_combined as b
ON 
			a.id = b.id
            
GROUP BY description_length_bucket
ORDER BY average_rating DESC
```
check the top-rated apps for each genre
```sql
SELECT 
			prime_genre,
            track_name,
            user_rating
FROM (
			SELECT
  			prime_genre,
  			track_name,
  			user_rating,
  			RANK() OVER(PARTITION BY prime_genre ORDER BY user_rating DESC, rating_count_tot DESC) AS rank
  			FROM
  			AppleStore
		) as a 
WHERE 
a.rank = 1
```
