## Covid Data Exploration with SQL (MS SQL Server)

**Project description:** 

1. The data was pulled from [ourworldindata.org](https://ourworldindata.org/covid-deaths)
2. The data covers a wide range of information from death count to test, vaccination, hospital admissions, financial and medical conditions, GDP, etc of all countries from the beginning of the pandemic till 24th Jan 2022.
3. I did some exploration with some of the columns (attributes) of the dataset, namely daily new_case, death and vaccination counts and filtered and did some calcualtion to make quantitative observations and get a sense for the data.
4. With the brief exploration one could rank the countries based on the Mortality rate, Infection rate, daily vaccination count, or break them down to regions or select certain countries and obtain insights on how well different countries dealed and managed the spread of virus.
5. Complete query [file](https://github.com/Tea-pott/DataAnalysis/blob/main/SQL%20Project%20Covid.sql).

<br>
<img src="images/coronavirus-data-explorer.png?raw=true"/>
<br>

### Summary of quesries to explore the data

**1. Mortaity rate in Europe:** 
Percent of confirmed cases that passed away in European countries in Desending order (as of 2022-01-24).
It shows the possibility of death in case of infection for each country and for more precise outlook the analysis could be broken down to provinces and cities combined with socio-demographic and health conditions. 
For example, it could be the case that amount of daily sunlight in a region could have an effect on mortality rate of men over 60 with certain medical condition.  

```sql

SELECT 
  location, 
  date, 
  total_cases, 
  total_deaths, 
  round((total_deaths/total_cases)*100, 2) AS death_percent
FROM 
  SQL_Project..[covid-death]
WHERE 
  continent = 'Europe' and date = '2022-01-24 00:00:00.000'
ORDER BY  
  5 DESC

```

**2. Infection rate in Asia:**
Percent of population infected in Asian countries based on confirmed cases (as of 2022-01-24). 
(Of course someone could ask if people who are infected many times are counted again in the total_cases or not.)

```sql

SELECT 
  location, 
  date, 
  MAX(total_cases) as Highest_case_count, 
  population, 
  MAX((total_cases/population)*100) AS Infected_population_perc
FROM  
  SQL_Project..[covid-death]
WHERE 
  continent = 'Asia'
GROUP BY 
  location, population, date
ORDER BY 
  Infected_population_perc DESC

```

**3. Global Vaccination:**  
Total vaccination count in countries around the world (till 2022-01-24) 
(Inner Join)

```sql

SELECT 
  dea.location, 
  population, 
  Max(cast (total_vaccinations as bigint)) as Total_vaccination
FROM 
  SQL_Project..[covid-death] dea
INNER JOIN 
  SQL_Project..[covid-vaccinations] vac
ON 
  dea.location = vac.location 
AND 
  dea.date = vac.date
WHERE 
  dea.continent is not null
GROUP BY 
  dea.location, population
ORDER BY 
  Total_vaccination DESC

```

**4. Global Rollong Vaccination Count:**
Rolling Count of daily administered vaccines in countries around the world (till 2022-01-24) 

```sql

SELECT 
  dea.continent, 
  dea.location, 
  dea.date, 
  dea.population, 
  vac.new_vaccinations,
  Sum(Convert(bigint, vac.new_vaccinations)) OVER (Partition by dea.location Order by dea.location,
  dea.date) as Rolling_vac_count
FROM 
  SQL_Project..[covid-death] dea
JOIN 
  SQL_Project..[covid-vaccinations] vac
ON 
  dea.location = vac.location 
AND 
  dea.date = vac.date
WHERE 
  dea.continent is not null
ORDER BY 
	2,3

```

