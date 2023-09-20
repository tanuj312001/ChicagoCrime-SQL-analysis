# Chicago Crime SQL analysis

**Creator**: Tanuj Shankarwar

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

* With 202,536 reported crimes, businesses operating in Chicago might need to consider security measures, especially if these numbers are increasing year over year.

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

#### A high frequency of thefts (39,758 reported) suggests businesses might want to invest in anti-theft systems and technologies.
#### A significant number of assaults (20,086 reported) could indicate areas where nighttime business operations might be riskier.
#### Recommendation: Tailor security measures based on prevalent crime types. For example, increased surveillance for areas with high theft, fraud detection systems for deceptive practices, and physical security measures for areas with high battery or assault rates.

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

#### Crime counts reflect the raw number of recorded offenses. But when assessing crime risk across geographies, it is important to consider crime rates as opposed to crime counts.

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

### 4. What is the average offenses commited  across these neighbourhoods?
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

### 5. Arrest analysis

````sql
SELECT arrest, COUNT(*) AS total_crimes, 
       COUNT(*) / SUM(COUNT(*)) OVER() * 100 AS arrest_percentage
FROM chicago_crimes
GROUP BY arrest;
````

**Results:**
| Arrest | Total Crimes | Arrest Percentage |
|--------|--------------|------------------|
| FALSE  | 178,550      | 88.1572%         |
| TRUE   | 23,986       | 11.8428%         |


### 6. Finding out the arrest percentage for each crime type.

````sql
SELECT crime_type, 
       COUNT(*) AS total_crimes,
       SUM(CASE WHEN arrest = 'TRUE' THEN 1 ELSE 0 END) AS total_arrests,
       (SUM(CASE WHEN arrest = 'TRUE' THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS arrest_percentage
FROM chicago_crimes
GROUP BY crime_type
````
**Results:**

| Crime Type                      | Total Crimes | Total Arrests | Arrest Percentage |
|--------------------------------|--------------|---------------|------------------|
| Assault                        | 20,086       | 1,767         | 8.7972           |
| Criminal Damage                | 24,716       | 861           | 3.4836           |
| Theft                          | 39,758       | 1,491         | 3.7502           |
| Motor Vehicle Theft            | 10,410       | 333           | 3.1988           |
| Other Offense                   | 13,588       | 1,439         | 10.5902          |
| Robbery                        | 7,813        | 376           | 4.8125           |
| Battery                        | 39,988       | 5,462         | 13.6591          |
| Deceptive Practice              | 15,710       | 176           | 1.1203           |
| Homicide                       | 803          | 182           | 22.6650          |
| Narcotics                      | 4,072        | 3,935         | 96.6356          |
| Criminal Trespass               | 3,367        | 988           | 29.3436          |
| Burglary                       | 6,546        | 231           | 3.5289           |
| Offense Involving Children     | 1,839        | 106           | 5.7640           |
| Weapons Violation              | 8,865        | 5,485         | 61.8725          |

### 7. What weekday were most crimes committed? Which month has the most crimes commited and the avg temp?


````sql
SELECT
	MONTHNAME(crime_date) as month,
	COUNT(*) AS n_homicides,
	round(avg(temp_high), 1) AS avg_high_temp
FROM
	chicago_crimes
WHERE crime_type = 'homicide'
GROUP BY
	month
ORDER BY
	n_homicides DESC;
````
**Results:**
| month     | n_homicides | avg_high_temp |
|-----------|-------------|---------------|
| July      | 112         | 82.6          |
| September | 89          | 80.8          |
| June      | 85          | 83.5          |
| August    | 81          | 85.3          |
| May       | 66          | 73.9          |
| October   | 64          | 67.9          |
| November  | 62          | 50.6          |
| January   | 55          | 34.1          |
| April     | 54          | 65.1          |
| December  | 52          | 48.6          |
| March     | 45          | 54.7          |
| February  | 38          | 27.0          |

#### The highest amount of crimes occured in the mild weather months.

### 8. What weekday were most crimes committed? 

````sql
SELECT
	DAYNAME(crime_date) as day ,
	COUNT(*) as total_crimes
FROM
	chicago_crimes
GROUP BY
	DAYNAME(crime_date)
ORDER BY total_crimes desc
````
**Results:**

| Day       | Total Crimes |
|-----------|--------------|
| Saturday  | 29841        |
| Friday    | 29829        |
| Sunday    | 29569        |
| Monday    | 29194        |
| Wednesday | 28143        |
| Tuesday   | 28135        |
| Thursday  | 27825        |

####  Crimes increases upto 7% during the weekends 

### 9. What are the top  city blocks streets that have had the most homicides including ties?


SELECT city_block, n_homicides
FROM (
SELECT city_block,
COUNT(*) as n_homicides,
RANK() OVER (ORDER BY COUNT(*) desc ) as ranking 
FROM chicago_crimes
WHERE crime_type='homicide'
GROUP BY city_block
) as subquery 
WHERE ranking<=10
`
**Results:**
| City Block                    | N Homicides |
|------------------------------|-------------|
| Madison St                   | 14          |
| 79th St                      | 14          |
| Morgan St                    | 10          |
| 71st St                      | 10          |
| Cottage Grove Ave            | 9           |
| Michigan Ave                 | 9           |
| Van Buren St                 | 8           |
| Emerald Ave                  | 7           |
| Polk St                      | 7           |
| Cicero Ave                   | 7           |
| Pulaski Rd                   | 7           |
| Dr Martin Luther King Jr Dr  | 7           |
| State St                     | 7           |


### 10. What was the number of reported crimes on the hottest day of the year vs the coldest?

````sql
with hottest as (
SELECT temp_high, COUNT(*) AS total_crimes
FROM chicago_crimes
WHERE temp_high=(SELECT MAX(temp_high) FROM chicago_crimes)
GROUP BY temp_high
),
coldest as (
SELECT temp_high, COUNT(*) AS total_crimes
FROM chicago_crimes
WHERE temp_high=(SELECT MIN(temp_high) FROM chicago_crimes)
GROUP BY temp_high
)
SELECT *
FROM hottest 
UNION 
SELECT *
FROM coldest
````
**Results:**

| Temp High | Total Crimes |
|-----------|--------------|
| 95        | 552          |
| 4         | 402          |

### There was a 37% increase in crime on the Hottest day of the year vs Coldest day of the year.

#### 11. What are the top 5 least reported crime, how many arrests were made and the percentage of arrests made?

````sql
SELECT crime_type, low_crimes, arrest_count,
ROUND(arrest_count*100/low_crimes,2) as arrest_percentage
FROM(
SELECT crime_type, COUNT(*) AS low_crimes,
SUM(CASE WHEN arrest='TRUE' THEN 1 ELSE 0 END) AS arrest_count
FROM chicago_crimes
GROUP BY crime_type
ORDER BY low_crimes 
LIMIT 5
) as subquery 
````

**Results:**

| Crime Type                | Low Crimes | Arrest Count | Arrest Percentage |
|---------------------------|------------|--------------|------------------|
| Other Narcotic Violation  | 2          | 1            | 50.00            |
| Public Indecency          | 4          | 4            | 100.00           |
| Non-Criminal              | 4          | 1            | 25.00            |
| Human Trafficking         | 12         | 0            | 0.00             |
| Gambling                  | 13         | 11           | 84.62            |



#### 12. How many crimes were reported on a monthly basis in chronological order. What is the month to month percentage change of crimes reported?


````sql
SELECT month,total_crimes,
ROUND(100*(total_crimes-LAG(total_crimes) OVER())/LAG(total_crimes) OVER(),2)  as month_to_month 
FROM (
SELECT MONTHNAME(crime_date) as month, COUNT(*) as total_crimes
FROM chicago_crimes
GROUP BY MONTHNAME(crime_date)
ORDER BY MONTH(crime_date)
) AS subquery 
````

**Results:**

| month      | total_crimes | month_to_month |
|------------|--------------|----------------|
| January    | 16038        | NULL           |
| February   | 12888        | -19.64         |
| March      | 15742        | 22.14          |
| April      | 15305        | -2.78          |
| May        | 17539        | 14.60          |
| June       | 18566        | 5.86           |
| July       | 18966        | 2.15           |
| August     | 18255        | -3.75          |
| September  | 18987        | 4.01           |
| October    | 19018        | 0.16           |
| November   | 16974        | -10.75         |
| December   | 14258        | -16.00         |


#### 13. How many types of theft took place and their count?

````sql
SELECT crime_type ,crime_description,COUNT(*) as total_crimes
FROM chicago_crimes
WHERE crime_description LIKE '%theft%'
GROUP BY crime_type, crime_description

````

**Results:**

| crime_type           | crime_description                          | total_crimes |
|----------------------|--------------------------------------------|--------------|
| theft                | retail theft                               | 6085         |
| deceptive practice   | theft of labor / services                  | 354          |
| deceptive practice   | financial identity theft over $ 300        | 2731         |
| deceptive practice   | financial identity theft $300 and under    | 5104         |
| motor vehicle theft  | theft / recovery - automobile              | 509          |
| deceptive practice   | attempt - financial identity theft         | 453          |
| theft                | attempt theft                              | 236          |
| deceptive practice   | theft of lost / mislaid property           | 275          |
| theft                | delivery container theft                   | 20           |
| deceptive practice   | aggravated financial identity theft        | 61           |
| deceptive practice   | theft by lessee, motor vehicle             | 211          |
| motor vehicle theft  | theft / recovery - truck, bus, mobile home | 17           |
| motor vehicle theft  | theft / recovery - cycle, scooter, bike with vin | 2    |
| deceptive practice   | theft by lessee, non-motor vehicle         | 9            |


#### 14. Most consecutive days where a homicide occured and the timeframe?

````sql
WITH get_all_dates AS (
	SELECT DISTINCT DATE(crime_date) AS c_date
	FROM chicago_crimes
	WHERE crime_type = 'homicide'
),
get_diff AS (
	SELECT 
		c_date,
		@rn := @rn + 1 AS rn,
		DATEDIFF(c_date, @rn) AS diff
	FROM get_all_dates, (SELECT @rn := 0) r
),
get_diff_count AS (
	SELECT
		c_date,
		COUNT(*) OVER (PARTITION BY diff) AS diff_count
	FROM get_diff
	GROUP BY c_date, diff
)
SELECT
	MAX(diff_count) AS most_consecutive_days,
	CONCAT(MIN(c_date), ' to ', MAX(c_date)) AS time_frame
FROM get_diff_count
WHERE diff_count > 40;
````

**Results:**
| most_consecutive_days | time_frame                |
|----------------------|---------------------------|
| 316                  | 2021-01-01 to 2021-12-27 |






