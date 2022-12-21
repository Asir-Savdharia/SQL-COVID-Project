# SQL-COVID-Project

SELECT*
FROM `sql-covid-project.covid_data.covid_worldwide`

## New Cases vs. New Deaths (Deadliness Check)

SELECT location, date, population,total_cases, new_cases, total_deaths, new_deaths, SAFE_DIVIDE(new_cases, new_deaths)*100 AS deathbynewcases 
FROM `sql-covid-project.covid_data.covid_worldwide`
order by 1,2

## Looking at Total Cases vs. Populations 

SELECT Location, date, total_cases, population, (total_cases/population)*100 as CasesPopulation
FROM `sql-covid-project.covid_data.covid_worldwide`
order by 1,2 

## Looking at Countries with Highest Infection Rate compared to Population 

SELECT Location, population, MAX(total_cases) AS highestinfectioncount , MAX((total_cases/population))*100 AS PercentPopulationInfected
FROM `sql-covid-project.covid_data.covid_worldwide`
Group by Location, population
order by PercentPopulationInfected desc

## Showing Countries with Lowest Death Count per Population 

SELECT Location, MIN(cast(total_deaths AS integer)) AS lowestdeathcount
FROM `sql-covid-project.covid_data.covid_worldwide`
WHERE continent is not null
Group by location 
order by lowestdeathcount desc

## Show the breakdown by continent 

SELECT continent, MAX(cast(total_deaths AS integer)) as TotalDeathCount 
FROM `sql-covid-project.covid_data.covid_worldwide`
Where continent is not null
Group by continent 
order by TotalDeathCount desc

## Global Numbers 

SELECT date, SUM(new_cases),SUM(cast(new_deaths AS integer)), SUM(cast(new_deaths AS integer))/SUM(new_cases)*100 AS DeathPercentage
FROM `sql-covid-project.covid_data.covid_worldwide` 
Where continent is not null 
Group By date
order by 1,2 

## Total population vs. vaccinations 

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM(CAST(vac.new_vaccinations AS int)) OVER (Partition by dea.location Order by dea.location, dea.date) AS RollingVaxCount
FROM `sql-covid-project.covid_data.covid_worldwide` dea 
JOIN  `sql-covid-project.covid_data.covid_worldwide` vac 
  On dea.location = vac.location
  and dea.date = vac.date 
Where dea.continent is not null 
order by 2,3 

## CTE to look at percentages of population that are vaccinated 

With PopvsVac 
as ( 
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM(CAST(vac.new_vaccinations AS int)) OVER (Partition by dea.location Order by dea.location, dea.date) AS RollingVaxCount
FROM `sql-covid-project.covid_data.covid_worldwide` dea 
JOIN  `sql-covid-project.covid_data.covid_worldwide` vac 
  On dea.location = vac.location
  and dea.date = vac.date 
Where dea.continent is not null 
)
## order by 2,3 
Select *, (RollingVaxCount/population)*100
FROM PopvsVac


## Creating a View of different stats 

Create View covid_data.DeathCountPerCountry
AS
SELECT Location, MIN(cast(total_deaths AS integer)) AS lowestdeathcount
FROM `sql-covid-project.covid_data.covid_worldwide`
WHERE continent is not null
Group by location 
## order by lowestdeathcount desc

SELECT *
FROM `sql-covid-project.covid_data.DeathCountPerCountry`

## Countries with Highest Positive Rates by Tests Administered 

SELECT Location, Date, positive_rate, CAST(total_tests AS int), positive_rate/CAST(total_tests AS int)*100 AS positivebypop
FROM `sql-covid-project.covid_data.covid_worldwide`
WHERE total_tests is not null


Create View covid_data.PositivePop 
AS
(
SELECT Location, Date, positive_rate, CAST(total_tests AS int) AS t_tests, positive_rate/CAST(total_tests AS int)*100 AS positivebypop
FROM `sql-covid-project.covid_data.covid_worldwide`
WHERE total_tests is not null
)

SELECT* 
FROM `sql-covid-project.covid_data.PositivePop`

## Booster Rate in total population (data limitation is that single, double, and triple are all in one) 

SELECT Location, Date, total_boosters, new_deaths, population, total_boosters/population*100 AS PopBoosted
FROM `sql-covid-project.covid_data.covid_worldwide`
WHERE total_boosters is not null AND total_boosters > 0 
Order by 1,2 

Create View covid_data.BoostedPop
AS  
(
SELECT Location, Date, total_boosters, new_deaths, population, total_boosters/population*100 AS PopBoosted
FROM `sql-covid-project.covid_data.covid_worldwide`
WHERE total_boosters is not null AND total_boosters > 0 
Order by 1,2 
)

SELECT* 
FROM `sql-covid-project.covid_data.BoostedPop`

## Death Rate as populations become more boosted 

SELECT Location, Date, total_boosters, new_deaths, new_deaths/total_boosters*100 AS BoostedDeathRate 
FROM `sql-covid-project.covid_data.covid_worldwide`
WHERE total_boosters is not null 

Create View covid_data.DeathRateBoostedPop
AS
(
SELECT Location, Date, total_boosters, new_deaths, new_deaths/total_boosters*100 AS BoostedDeathRate 
FROM `sql-covid-project.covid_data.covid_worldwide`
WHERE total_boosters is not null 
) 

SELECT *
FROM `sql-covid-project.covid_data.DeathRateBoostedPop`


## New Deaths and ICU patients as Vaccination Rate Increases

SELECT Location, Date, new_deaths, icu_patients, total_vaccinations, icu_patients/total_vaccinations*100 AS CovidSeverityWithVax 
FROM `sql-covid-project.covid_data.covid_worldwide`
WHERE total_vaccinations is not null and total_vaccinations > 0 

Create view covid_data.CovidSeverityWithVax
AS
(
SELECT Location, Date, new_deaths, icu_patients, total_vaccinations, icu_patients/total_vaccinations*100 AS CovidSeverityWithVax 
FROM `sql-covid-project.covid_data.covid_worldwide`
WHERE total_vaccinations is not null and total_vaccinations > 0 
)

SELECT*
FROM `sql-covid-project.covid_data.CovidSeverityWithVax`


## Looking at death rate by total cases 

SELECT Location, Date, CAST(total_deaths AS numeric) AS all_deaths, total_cases, total_cases/total_deaths*100 AS CovidDeathRate 
FROM `sql-covid-project.covid_data.covid_worldwide`
WHERE total_deaths is not null AND total_deaths > 0 

Create View covid_data.CovidDeathRate
AS
(
SELECT Location, Date, CAST(total_deaths AS numeric) AS all_deaths, total_cases, total_cases/total_deaths*100 AS CovidDeathRate 
FROM `sql-covid-project.covid_data.covid_worldwide`
WHERE total_deaths is not null AND total_deaths > 0 
)

SELECT*
FROM `sql-covid-project.covid_data.CovidDeathRate`


