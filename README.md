# Space Missions Data Cleaning, Analysis and Dashboard creation through Power BI


## Introduction:

This project is a comprehensive analysis of space missions data, focusing on various aspects such as mission success rates, rocket usage, launch locations, and financial insights.

Space exploration has always been a fascinating and challenging endeavor which I absolutely love, that pushes the boundaries of human knowledge and technology. This project aims to delve into the wealth of data available on space missions to gain insights into the history, trends, and patterns of space exploration.

<br/><br/>

## Project Overview

### Data Source

The data set includes all space missions from 1957 to August 2022, including details on the location, date, and result of the launch, the company responsible, and the name, price, and status of the rocket used for the mission.. The file will be added as "space_missions.csv". 

### Objectives

 1. To clean, explore, and analyze space missions data sourced from various reliable sources.

 2. To identify trends and patterns in space missions, including mission success rates, rocket usage, and launch locations.

 3. To create informative visualizations and dashboards using Power BI to present the findings in a clear and engaging manner.

<br/><br/>


## Key Performance Indicators (KPIs)

We'll focus on the following KPIs to evaluate the performance and trends of space missions:



1. Total Missions

2. Mission Success Rate

3. Average Mission Cost

4. Countries Involved

<br/><br/>


## Charts and Visualizations:

We'll create a variety of charts and visualizations to present the data effectively:



1. Mission amount by Rocket/Country/Company/Year
2. Average Price by Company
3. Successful Missions by Country/Launch Center/Company
4. Success rate by Country
5. Mission status percentage
6. Average Price by Country/Rocket/Company
7. Total Cost by Year
8. Mission by Rocket Status




<br/><br/>




























1. How have rocket launches trended across time? Has mission success rate increased?
    
2. Which countries have had the most successful space missions? Has it always been that way?
    
3. Which rocket has been used for the most space missions? Is it still active?
    
4. Are there any patterns you can notice with the launch locations?

<br/><br/>

## Tools and Technologies:

 - Microsoft SQL Server Management Studio for data manipulation and analysis.
 - PowerBI for data visualization and statistical analysis.
 - Obsidian for documentation purposes.

<br/><br/>

## Data cleaning and preparation:
<br/><br/>



### Dividing the Location column:

Upon initial exploration of the dataset, it was evident that the "Location" column encapsulated multiple geographical attributes, including country, city/state, launch pad, and launch center. To enhance the granularity of the analysis, the following steps were taken to divide this column into distinct components:



 - A new column named "Country" was added to the SpaceMissions table using SQL's ALTER TABLE command. This column was intended to house the country information extracted from the "Location" column.

```sql
ALTER TABLE SpaceMissions
ADD Country VARCHAR(255);
```


 - Utilizing SQL functions like RIGHT and CHARINDEX, the country information was isolated from the "Location" column and populated into the newly created "Country" column. This involved locating the comma within the string from the rightmost side and extracting the substring preceding it, representing the country.

```sql
Update SpaceMissions
Set [Country] = RIGHT(Location,CHARINDEX(',',REVERSE(Location)) - 1)
```


 - A new column titled "LaunchSite" was introduced to capture the launch site details.

```sql
ALTER TABLE SpaceMissions
ADD LaunchSite VARCHAR(255);
```


 - SQL's LEFT and CHARINDEX functions were leveraged to identify the launch site information within the "Location" column and store it in the designated "LaunchSite" column. This process involved locating the first comma within the string and extracting the substring preceding it, which denoted the launch site.

```sql
SELECT LEFT(Location,CHARINDEX(',',Location) -1)
FROM SpaceMissions
```


 - Another new column named "LaunchCenter" was added to accommodate the launch center details.

```sql
ALTER TABLE SpaceMissions
ADD LaunchCenter VARCHAR(255);
```

 - Using SQL's SUBSTRING and CHARINDEX functions, the launch center information was extracted from the "Location" column and populated into the "LaunchCenter" column. 

   When it came to writing this specific script I wanted to, using a substring, get the part of the string within location which is after the first comma up until the second comma. Since The launch site column matches the first part of the location column up until the comma, I was using the LEN(LaunchSite) to show the substring where to start, and then using CHARINDEX starting AFTER the launch site part to locate the index of the next comma. Then I used - LEN(launchsite) so that it gives us the second part lenght for our substring script and not just the length from the beggining till the second comma.   After this, I noticed some entries in the location Column only have 1 comma and would give a null charIndex so we had to create the CASE in order to help us handle that.


```sql
UPDATE SpaceMissions
SET LaunchCenter = CASE WHEN CHARINDEX(',', Location, LEN(LaunchSite) + 2) > 0
         THEN SUBSTRING(Location, LEN(LaunchSite) + 2, (CHARINDEX(',', Location, LEN(LaunchSite) + 2) - (LEN(LaunchSite) + 1))-1)
         ELSE NULL 
    END
```



<br/><br/>



### FIxing Launch Site and Launch center column issues:


Following the division of the "Location" column, discrepancies were observed in the "LaunchSite" and "LaunchCenter" columns, necessitating corrective actions:


 - Addressing Null Launch Center Values:

Entries in the "Location" column lacking launch sites resulted in the launch centers being erroneously recorded as launch sites. Additionally, several entries contained null values in the "LaunchCenter" column. To rectify these inconsistencies, null launch center values were reassigned to the "LaunchCenter" column to ensure data accuracy.

First checked the changes through a select script:

```sql
SELECT CASE 
			WHEN LaunchCenter IS NULL THEN LaunchSite
			ELSE LaunchCenter 
		END AS UPDATEDLAUNCHCENTER,
		LaunchCenter,
		LaunchSite,
		[Date]
FROM SpaceMissions
ORDER BY [Date] DESC
```


Then applied it through the set script:

```sql
UPDATE SpaceMissions
SET LaunchCenter = CASE 
			WHEN LaunchCenter IS NULL THEN LaunchSite
			ELSE LaunchCenter 
		END;
```


 - Removing Incorrect Launch Site Entries:
   
Entries where the launch site matched the launch center were identified and nullified in the "LaunchSite" column to mitigate data inaccuracies. This involved verifying each entry and updating the respective column accordingly.

First checking the actions:
 
```sql
Select CASE 
		WHEN LaunchSite = LaunchCenter THEN NULL
		ELSE LaunchSite
		END AS NewLaunchSite ,LaunchCenter,
		LaunchSite,
		[Date]
from SpaceMissions
ORDER BY [Date] DESC
```

Then applying them:

```sql
UPDATE SpaceMissions
SET LaunchSite = CASE 
		WHEN LaunchSite = LaunchCenter THEN NULL
		ELSE LaunchSite
		END;
```


<br/><br/>

### Remove useless Column:

Having successfully divided the "Location" column and rectified associated discrepancies, the redundant "Location" column was removed from the SpaceMissions table using the SQL ALTER TABLE command.

```sql
ALTER TABLE SpaceMissions
DROP COLUMN [Location]
```


<br/><br/>

With the completion of the data cleaning process, our table is now primed for visualization. Despite encountering numerous entries with null values in the "Price" and "Time" columns, we opted to retain them for accurate mission status representation in our graphical analyses. This decision ensures that our visualizations provide a comprehensive view of mission success rates, accounting for all relevant data points. Particularly, since most of the null Price entries will be old entries, then we would be specifically removing many old values and it would greatly affect our accuracy in a negative way when it came to evaluating other dynamics not related to Time and Costs.

<br/><br/>

## Power BI Enhancements:

Upon importing the dataset into Power BI and embarking on visualization creation, an issue surfaced regarding the "Price" column format. Initially set as a string due to CSV import limitations in the SQL program, converting it to decimal numbers presented challenges. This was primarily due to the presence of dots instead of commas as decimal separators.

To address this, a meticulous process ensued, involving the duplication, trimming, and transformation of the "Price" column. Every dot was replaced with a comma, enabling the conversion to decimal numbers. This step was indispensable, as measures and SUM functions require numerical data for accurate calculations and visualizations.

Furthermore, to enhance interpretability, a new column was created to express prices in millions, achieved by multiplying the price by 1,000,000. This transformation facilitates a clearer understanding of the price scale within our visualizations.

In addition, two essential measures were devised to augment our analytical capabilities:

Total Missions Measure: This measure calculates the total number of missions, encompassing all entries within the dataset.

Total Missions with Price Measure: Recognizing the prevalence of numerous missions without recorded prices, particularly attributed to the USSR, a specialized measure was crafted. It counts the total missions with non-null prices, thereby mitigating skewness in analyses, especially evident in average price calculations by company. To ensure robustness, a filter was applied, restricting analysis to companies with at least five missions featuring recorded prices.

Here's the DAX formula for the Total Missions with Price measure:

```DAX
TotMissionsPrice =COUNTROWS(FILTER('SpaceMissions',NOT(ISBLANK('SpaceMissions'[Price]))))
```

Along with these I also created a SuccessfulMission count Measure and A Success Rate Percentage Measure.

```DAX
SuccessRatePercentage = DIVIDE([SuccessfulMissions],[TotalMissions],0)
```



<br/><br/>

## Data visualization:

### Summary of Dashboard Insights








<br/><br/>



<br/><br/>

## Dashboard image:





<br/><br/>
<br/><br/>
## Results/findings
<br/><br/>


### Trends in Sale Prices Over Time:


<br/><br/>

### Correlation Between Property Size (Acreage) and Sale Price:


<br/><br/>

### Distribution of Housing Types Based on Land Use:

<br/><br/>

### Total Property Value Over the Years:

<br/><br/>

### Patterns in Housing Characteristics:


<br/><br/>

### Average Sale Price of Properties in Nashville:

<br/><br/>

### Properties Sold as Vacant:


<br/><br/>

### Other Trends in the Data:

<br/><br/>

## Conclusion:
