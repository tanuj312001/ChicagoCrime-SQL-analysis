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







