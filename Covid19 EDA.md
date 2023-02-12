# Covid19 EDA from Mar 2020 to Jan 2023

-- loading data into deaths table from .csv files
```
LOAD DATA LOCAL INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/latest_covid_deaths.csv'
INTO TABLE deaths
FIELDS TERMINATED BY ','
ESCAPED BY ''
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

-- loading data into vaccinations table from .csv file
```
LOAD DATA LOCAL INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/latest_covid_vaccinations.csv'
INTO TABLE vaccinations
FIELDS TERMINATED BY ','
ESCAPED BY ''
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

-- converting date column from text to DATE format
```
UPDATE deaths
SET date = STR_TO_DATE(date,'%d-%m-%Y');
UPDATE vaccinations1
SET date = STR_TO_DATE(date,'%d-%m-%Y');
```
-- Select Data that we are going to be starting with
```
SELECT 
		Location,
		date,
		total_cases,
		new_cases,
		total_deaths,
		population
FROM
		Deaths
WHERE
		continent IS NOT NULL
ORDER BY
		location,date;
```

-- Comparing total cases vs total deaths
```
SELECT  location,
	      date,
        total_cases,
        total_deaths,
        (total_deaths/total_cases) * 100 AS Death_percentage
FROM
		    deaths
WHERE
		    continent IS NOT NULL
ORDER BY
		    death_percentage DESC;
```

-- comparing countrywise latest records of total cases and total deaths
-- shows current total cases and death
```
SELECT  ds.location,
	      date,
        total_cases,
        total_deaths,
        ROUND((total_deaths/total_cases) * 100,2) AS Death_percentage
FROM
		    deaths ds
        INNER JOIN
        (SELECT location,
				        MAX(date) AS latest_date
		     FROM  
				        deaths
		     GROUP BY
				        location) AS d ON ds.location = d.location 
                AND ds.date = d.latest_date
WHERE
		     continent IS NOT NULL
ORDER BY 
		     location; 
```
-- comparing total cases vs population
```
SELECT 	location,
      	date,
        population,
        total_cases,
        ROUND((total_cases/population) * 100,3) AS percent_infected_population
FROM 
		    deaths
WHERE
		    continent IS NOT NULL
ORDER BY
		    location,date;
```        
-- finding percent of populaion infected for different countries
```
SELECT 	location,
    		population,
        MAX(total_cases) AS Highest_infection_count,
        MAX(total_cases/population) * 100 AS percent_population_infected
FROM
		    deaths
GROUP BY 
		    location,population
ORDER BY 
		    percent_population_infected DESC;
```        
-- finding total deaths for different countries till now
```
SELECT 	location,
    		MAX(total_deaths) AS total_death_count
FROM
		    deaths
WHERE
		    continent IS NOT NULL
GROUP BY
		    location
ORDER BY
		    total_death_count DESC;
```        
-- continentwise total death count
```
SELECT 	continent,
	    	MAX(total_deaths) AS total_death_count
FROM
		    deaths
WHERE 
		    continent IS NOT NULL
GROUP BY 
		    continent
ORDER BY 
		    total_death_count DESC;
```        
-- showing global numbers
```
SELECT 	date,
    		SUM(new_cases) AS total_case_count,
        SUM(new_deaths) AS total_death_count,
        (SUM(new_deaths)/SUM(new_cases)) * 100 AS death_percentage
FROM
		    deaths
WHERE
		    continent IS NOT NULL
GROUP BY 
		    date
ORDER BY 
		    date;
```        
-- calculating total people vaccinated for different country
```
SELECT
    	d.continent,
      d.location,
      d.date,
      d.population,
      v.new_vaccinations,
      SUM(v.new_vaccinations) OVER (PARTITION BY d.location ORDER BY d.location,d.date) AS total_people_vaccinated_till_date
FROM
		  deaths d
        INNER JOIN
	    vaccinations v 
        ON d.location = v.location AND
		       d.date = v.date
WHERE 
      d.continent IS NOT NULL;
```
-- finding % of people vaccinated using CTE
```
WITH cte_pop_vs_vac AS
(
	SELECT
      	d.continent,
       	d.location,
       	d.date,
       	d.population,
       	v.new_vaccinations,
      	SUM(v.new_vaccinations) OVER (PARTITION BY d.location ORDER BY d.location,d.date) AS total_people_vaccinated_till_date
	FROM
		    deaths d
	     	 JOIN
		    vaccinations v 
	        ON d.location = v.location AND
		         d.date = v.date
	WHERE d.continent IS NOT NULL
)
SELECT
		*,
    (total_people_vaccinated_till_date/population) * 100 AS percent_of_people_vaccinated
FROM
		cte_pop_vs_vac;
```        
-- finding % of people vaccinated by creating a temporary table
```
CREATE TEMPORARY TABLE t_popvsvac
SELECT
   		d.continent,
     	d.location,
     	d.date,
     	d.population,
     	v.new_vaccinations,
     	SUM(v.new_vaccinations) OVER (PARTITION BY d.location ORDER BY d.location,d.date) AS total_people_vaccinated_till_date
	FROM
	     deaths d
	       JOIN
	     vaccinations v 
	       ON d.location = v.location AND
	         d.date = v.date
  WHERE 
      d.continent IS NOT NULL
;
SELECT
		*,
    (total_people_vaccinated_till_date/population) * 100 AS percent_of_people_vaccinated
FROM
		popvsvac;
```        
-- Creating View to store data for later visualizations
```
CREATE VIEW v_percent_pop_infected AS
SELECT
     	d.continent,
      d.location,
      d.date,
      d.population,
      v.new_vaccinations,
      SUM(v.new_vaccinations) OVER (PARTITION BY d.location ORDER BY d.location,d.date) AS total_people_vaccinated_till_date
FROM
		  deaths d
		    JOIN
		  vaccinations v 
	     ON d.location = v.location AND
		      d.date = v.date
WHERE 
	    d.continent IS NOT NULL;
```        
	
-- To see my tableau Visualisations [Click HERE](https://public.tableau.com/views/Covid19EDADashboard/Dashboard1?:language=en-US&:display_count=n&:origin=viz_share_link)
