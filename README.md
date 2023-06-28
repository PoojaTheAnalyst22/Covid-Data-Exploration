# Covid-Data-Exploration


 create database covid_19

 use covid_19

 select * from covid_deaths
 select * from covid_vaccinations

 select * from covid_vaccinations
 order by 3,4


-- this is done to check the data is entered right

-- Select data we are going to be using

 Select Location, Date, total_cases, new_cases, total_deaths, population
 from covid_deaths
 order by 1,2

-- looking at percentage of death in India

 Select Location, Date, total_cases, new_cases,total_deaths,
 (total_deaths/total_cases)*100 as Deathpercentage
 from covid_deaths
 where location = 'India'
 order by 1,2


-- looking at percentage of infection in India

 Select Location, Date, Population, total_cases, new_cases,total_deaths,
 (total_cases/population)*100 as Infectedpercentage
 from covid_deaths
 where location='India'
 order by 1,2


--looking at countries with highest infection rate 

 Select Location, population, Max(total_cases) as Highest_infection_count,
 Max((total_cases/population))*100 as percentage_population_infected
 from covid_deaths
 group by population, location
 order by percentage_population_infected desc


--looking at countries with the highest death rate.

 Select Location, Max(cast(total_deaths as int))  as Total_death_count
 from covid_deaths
 where continent is not null -- the location column had columns and other unrelated info
 group by location
 order by Total_death_count desc


--Seeing the same contintent vise

 Select continent, max(total_deaths) as Total_death_count
 from covid_deaths
 where continent is not null-- the location column had columns and other unrelated info
 group by continent
 order by Total_death_count desc


--Global numbers

 Select Sum(new_cases) as total_cases, sum(cast(new_deaths as int)) as total_deaths, 
 sum(cast(new_deaths as int))/Sum(new_cases)*100 as Deathpercentage
 from covid_deaths
 where continent is not null
 order by 1,2


--joining the two tables

 Select * from  covid_deaths d
 Join covid_vaccinations v
 on d.location=v.location
 and d.date=v.date


--Looking at total population vs vaccination

 Select d.continent, d.location, d.date, d.population, v.new_vaccinations
 From  covid_deaths d, covid_vaccinations v
 where d.location=v.location
 and d.date=v.date
 and d.continent is not null
 order by 2,3


--Looking at the total number of population which has been vaccinated.

 Select d.continent, d.location, d.date, d.population, v.new_vaccinations
 From covid_deaths d
 Join covid_vaccinations v
 on d.location=v.location
 and d.date=v.date
 where d.continent is not null
 order by d.location, d.date


--now to have a column to display the rolling  count of vaccinations by location

 Select d.continent, d.location, d.date, d.population, v.new_vaccinations,
 Sum(CONVERT(bigint,v.new_vaccinations)) OVER (Partition by d.location Order by d.location, d.Date) as Rolling_vacination_count
 From  covid_deaths d
 Join covid_vaccinations v
 on d.location=v.location
 and d.date=v.date
 where d.continent is not null
 order by 2,3


--Using CTE

 with pop_vs_vac (Continent, Location, Date, population, new_vaccinations, Rolling_vacination_number)
 as
 (
 Select d.continent, d.location, d.date, d.population, v.new_vaccinations,
 Sum(CONVERT(bigint,v.new_vaccinations)) OVER (Partition by d.location Order by d.location, d.Date) as Rolling_vacination_number
 From covid_deaths d
 Join covid_vaccinations v
 on d.location=v.location
 and d.date=v.date
 where d.continent is not null
 )
 select *, (Rolling_vacination_number/population)*100 as Rolling_vaccination_percentage
 From pop_vs_vac


--Creating a view 

 Create view Population_vs_vaccination
 as 
 Select d.continent, d.location, d.date, d.population, v.new_vaccinations,
 Sum(CONVERT(bigint,v.new_vaccinations)) OVER (Partition by d.location Order by d.location, d.Date) as Rolling_vacination_number
 From  covid_deaths d
 Join covid_vaccinations v
 on d.location=v.location
 and d.date=v.date
 where d.continent is not null


--opening a view

 select * from Population_vs_vaccination
