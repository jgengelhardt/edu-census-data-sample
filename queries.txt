-- How many public high schools are in each zip code?

SELECT 
	COUNT(school_id) AS "Number of High Schools", 
	zip_code AS "Zip Code"
FROM public_hs_data
GROUP BY zip_code;

-- How many public high schools are in each zip code? in each state?

SELECT 
	COUNT(school_id) AS "Number of High Schools", 
	state_code AS "State"
FROM public_hs_data
GROUP BY state_code;

-- What is the minimum, maximum, and average median_household_income of the nation?

SELECT 
	MIN(median_household_income) AS 'Min Median Income',
	ROUND(AVG(median_household_income), 2) AS 'Average Median Income',
	MAX(median_household_income) AS 'Max Median Income'
FROM census_data
WHERE median_household_income != 'NULL';

-- What is the minimum, maximum, and average median_household_income for each state?

SELECT 
	state_code,
	MIN(median_household_income) AS 'Min Median Income', 
	ROUND(AVG(median_household_income), 2) AS 'Average Median Income',
	MAX(median_household_income) AS 'Max Median Income'
FROM census_data
WHERE median_household_income != 'NULL'
GROUP BY state_code;

-- Do characteristics of the zip-code area, such as median household income, influence students’ performance in high school?

SELECT 
	CASE
		WHEN median_household_income < 25000 THEN '25000>'
		WHEN median_household_income BETWEEN 25000.01 AND 50000 THEN '25k-50k'
		WHEN median_household_income BETWEEN 50000.01 AND 100000 THEN '50K-100K'
		WHEN median_household_income BETWEEN 100000.01 AND 200000 THEN '100k-200k'
		WHEN median_household_income > 200000 THEN '200k+'
	END AS 'Median Household Income',
	COUNT(DISTINCT public_hs_data.school_id) AS '# of High Schools',
	ROUND(AVG(public_hs_data.pct_proficient_math),2) AS 'Math Prof.',
	ROUND(AVG(public_hs_data.pct_proficient_reading),2) AS 'Reading Prof.'
FROM public_hs_data
JOIN census_data ON census_data.zip_code = public_hs_data.zip_code
WHERE pct_proficient_math != 'NULL' AND pct_proficient_reading != 'NULL'
GROUP BY 1
ORDER BY 3 DESC;

-- On average, do students perform better on the math or reading exam?

SELECT 
	ROUND(AVG(public_hs_data.pct_proficient_math),2) AS 'Math Prof.',
	ROUND(AVG(public_hs_data.pct_proficient_reading),2) AS 'Reading Prof.'
FROM public_hs_data;

-- On average, do students in each state perform better on the math or reading exam? 

WITH math_scores AS (
	SELECT
		AVG(pct_proficient_math) AS math_score,
		state_code
	FROM public_hs_data
	WHERE pct_proficient_math IS NOT NULL
	GROUP BY state_code
),
reading_scores AS (
	SELECT
		AVG(pct_proficient_reading) AS reading_score,
		state_code
	FROM public_hs_data
	WHERE pct_proficient_reading IS NOT NULL
	GROUP BY state_code)
SELECT public_hs_data.state_code AS "State",
CASE
		WHEN reading_score > math_score THEN "Reading"
		WHEN reading_score < math_score THEN "Math"
END AS "Higher Scores"
FROM public_hs_data
	JOIN reading_scores ON reading_scores.state_code = public_hs_data.state_code
	JOIN math_scores ON math_scores.state_code = public_hs_data.state_code
GROUP BY public_hs_data.state_code;

-- What is the average proficiency on state assessment exams for each zip code in a given state?

SELECT 
	zip_code AS "Zip",
	COUNT(DISTINCT school_id) AS "# of Schools",
	ROUND(AVG(pct_proficient_math),2) AS "Math",
	ROUND(AVG(pct_proficient_reading),2) AS "Reading"
FROM public_hs_data
WHERE state_code = "IL"
GROUP BY zip_code
ORDER BY 2 DESC;
