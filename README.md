# ChicagoCrime-SQL-analysis

**Creator**: Tanuj Shankarwar

**Email**: tshankarwar@ucsd.edu

**Linkedin**: https://www.linkedin.com/in/tanuj-s-662785166/


### 1. How many total crimes were commited?


````sql
SELECT count(crime_id) AS "total reported crimes"
FROM crimes;
````
**Results**
Total Reported Crimes|
---------------------|
202536|


### 2. What were the top 5 crime types?

````sql

SELECT crime_type, COUNT(*) as frequency
FROM crimes 
GROUP BY crime_type 
ORDER BY COUNT(*) DESC 
LIMIT 5 
````
**Results**
| crime_type         | frequency |
|--------------------|-----------|
| battery            | 39988     |
| theft              | 39758     |
| criminal damage    | 24716     |
| assault            | 20086     |
| deceptive practice | 15710     |

#### Join data from three tables crimes,weather and community

````sql
CREATE  TABLE chicago_crimes as (
SELECT 
cr.id, 
cr.crime_type,
STR_TO_DATE(cr.crime_date, '%m/%d/%Y') as crime_date,
HOUR(STR_TO_DATE(cr.crime_date, '%m/%d/%Y %H:%i')) AS crime_hour,
cr.crime_description,
cr.crime_location,
cr.city_block,
co.name as community_name,
co.population ,
co.area_sq_mi as area_size,
co.density,
cr.arrest, 
cr.domestic,
wr.temp_high, 
wr.temp_low, 
wr.precipitation 
FROM crimes cr JOIN community co 
ON cr.community_id=co.community_area_id
JOIN weather wr 
ON STR_TO_DATE(wr.date, '%m/%d/%Y') = STR_TO_DATE(cr.crime_date, '%m/%d/%Y')
)
SELECT *
FROM chicago_crimes
LIMIT 2
````
**Results:**

| id     | crime_type        | crime_date | crime_hour | crime_description        | crime_location                        | city_block       | community_name | population | area_size | density   | arrest | domestic | temp_high | temp_low | precipitation |
|--------|-------------------|------------|------------|--------------------------|---------------------------------------|------------------|----------------|------------|-----------|-----------|--------|----------|-----------|----------|----------------|
| 168267 | assault           | 2021-12-16 | 12         | aggravated - handgun     | alley                                 | 47th st          | brighton park  | 45053      | 2.72      | 16563.6   | FALSE  | FALSE    | 66        | 32       |                |
| 168269 | criminal damage   | 2021-12-16 | 11         | to vehicle               | parking lot / garage (non residential) | spaulding ave    | avondale       | 36257      | 1.98      | 18311.62  | FALSE  | FALSE    | 66        | 32       |                |
