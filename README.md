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


### 3. Which communities had the most crime?

````sql
SELECT community_name,population, COUNT(*) as total_crimes
FROM chicago_crimes
GROUP BY community_name,population
ORDER BY total_crimes DESC
LIMIT 5
````
**Results:**

| community_name    | population | total_crimes |
|-------------------|------------|--------------|
| austin            | 96557      | 11341        |
| near north side   | 105481     | 8126         |
| south shore       | 53971      | 7272         |
| near west side    | 67881      | 6743         |
| north lawndale    | 34794      | 6161         |

### 4. Crime counts reflect the raw number of recorded offenses. But when assessing crime risk across geographies, it is important to consider crime rates as opposed to crime counts.

````sql
SELECT community_name,population, COUNT(*)*1000/population as crime_rate 
FROM chicago_crimes
GROUP BY community_name,population
ORDER BY COUNT(*)*1000/population desc
````
**Results:**
| community_name       | population | crime_rate |
|----------------------|------------|------------|
| west garfield park   | 17433      | 232.7769   |
| fuller park          | 2567       | 210.7519   |
| englewood            | 24369      | 189.8724   |
| east garfield park   | 19992      | 177.4710   |
| north lawndale       | 34794      | 177.0708   |

### 5. What is the average offenses commited  across these neighbourhoods?
````sql
with cte as (
SELECT community_name,population, COUNT(*)*1000/population as crime_rate 
FROM chicago_crimes
GROUP BY community_name,population
ORDER BY crime_rate desc
LIMIT 3 
)
SELECT community_name, AVG(total_crimes) AS average_crimes
FROM (
  SELECT community_name, crime_date, COUNT(*) as total_crimes
  FROM chicago_crimes
  WHERE community_name in (
  SELECT community_name FROM cte )
  GROUP BY community_name, crime_date
) AS subquery
GROUP BY community_name
````
**Results:**

| community_name       | average_crimes |
|----------------------|----------------|
| englewood            | 12.8172        |
| west garfield park   | 11.2410        |
| fuller park          | 1.9673         |

### 6. Arrest analysis

**Results:**
| Arrest | Total Crimes | Arrest Percentage |
|--------|--------------|------------------|
| FALSE  | 178,550      | 88.1572%         |
| TRUE   | 23,986       | 11.8428%         |

