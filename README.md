# Space Missions Data Cleaning, Analysis and Dashboard creation through Power BI


## Introduction:

This project is a comprehensive analysis of space missions data, focusing on various aspects such as mission success rates, rocket usage, launch locations, and financial insights.

Space exploration has always been a fascinating and challenging endeavor which I absolutely love, that pushes the boundaries of human knowledge and technology. This project aims to delve into the wealth of data available on space missions to gain insights into the history, trends, and patterns of space exploration.

<br/><br/>

## Project Overview

### Data Source

The data set includes all space missions from 1957 to August 2022, including details on the location, date, and result of the launch, the company responsible, and the name, price, and status of the rocket used for the mission. The file will be added as "space_missions.csv". 

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



 #### Page 1: Home
   
1. Mission Amount by Rocket/Country/Company Stacked Bar Chart:

 - Provides an overview of mission distribution across rockets, countries, companies, and years, allowing for comparison of mission volumes across different dimensions.

2. Missions by Year Area Chart:

 - Illustrates the trend of mission counts over time, highlighting fluctuations and patterns in mission launches across different years.

<br/><br/>

#### Page 2: Successful Missions and Mission Status

1. Successful Missions by Country/Launch Center/Company Stacked Bar Chart:

 - Showcases the distribution of successful missions across countries, launch centers, and companies, providing insights into the entities with the highest success rates.

2. Success Rate by Country Stacked Bar Chart:

 - Visualizes the success rates of missions by country, allowing for comparison of success rates among different nations.

3. Mission Status Percentage Pie Chart:

 - Presents the distribution of mission statuses (Success, Failure, Partial Failure, Prelaunch Failure) as a percentage, offering insights into overall mission outcomes.

<br/><br/>

#### Page 3: Price Related Charts

1. Total Cost by Year Area Chart:

 - Depicts the total cost of missions over time, enabling analysis of expenditure trends and budget allocations across various years.

2. Average Price by Country/Rocket/Company Stacked Bar Chart:

 - Presents the average prices of missions categorized by country, rocket, and company, allowing for in-depth analysis of pricing trends across various dimensions.

3. Mission by Rocket Status Pie Chart:

 - Represents the distribution of missions based on rocket status (Active or Inactive), offering insights into the utilization of rocket fleets.

#### Additional Insights:

 - Cards with Success Rate, Total Missions, Countries Involved, and Average Rocket Price:
Provides at-a-glance metrics summarizing key insights, including overall success rate, total missions, countries involved, and average rocket price.

####Navigation:
 - Date Slicer and Buttons for Easy Access:
Incorporates a date slicer on every page to facilitate filtering by specific time periods. Additionally, buttons are provided for seamless navigation between pages, enhancing user experience and accessibility.


<br/><br/>

<br/><br/>

<br/><br/>


## Dashboard image:
<br/><br/>
![image](https://github.com/DiogoGravanita/Space-Missions-SQL-PowerBI-Project/assets/163042130/2aa3a974-736e-4b86-b0b8-c1f659eac659)
<br/><br/>
![image](https://github.com/DiogoGravanita/Space-Missions-SQL-PowerBI-Project/assets/163042130/520d234a-11f2-43f4-8b3b-cd4f69478b23)
<br/><br/>
![image](https://github.com/DiogoGravanita/Space-Missions-SQL-PowerBI-Project/assets/163042130/56303984-46d6-493b-b721-83742149028e)


<br/><br/>
<br/><br/>
# Results/findings
<br/><br/>


## Key Performance Indicators (KPIs)




1. Total Missions: 5000
2. Mission Success Rate: 89,89%
3. Average Mission Cost: 128.3 Million €
4. Countries Involved: 22


<br/><br/>


## Charts and Visualizations Findings:

<br/><br/>
### Mission Amount by Rocket/Country/Company/Year:

The analysis reveals that space missions are predominantly led by the United States and Russia, with Kazakhstan being a significant contributor as well. Notably, the Cosmos-3M rocket, often associated with RVSN USSR, stands out as the most frequently used rocket in these missions.

<br/><br/>

### Average Price by Company:

Among the companies involved in space missions, NASA emerges as the top spender, investing significantly higher than its counterparts like Boeing, ULA, and Arianespace. This underscores NASA's commitment to pioneering space exploration efforts.

<br/><br/>

### Missions by Year:

The data illustrates a distinct trend in space mission activity over the years. From a surge in missions during the mid-60s to late 70s, a decline until the mid-2010s, and a subsequent resurgence, it's evident that space exploration has witnessed dynamic shifts in mission frequency over time.

<br/><br/>

### Successful Missions by Country/Launch Center/Company:

The analysis highlights the success rates of space missions across various countries, launch centers, and companies. Russia and the USA lead in successful missions, with the Plesetsk Cosmodrome serving as a pivotal launch center. RVSN USSR emerges as the company with the highest success rate, indicating robust mission planning and execution.

<br/><br/>

### Success Rate by Country:

France, surprisingly, boasts the highest success rate among countries involved in space missions, closely followed by Russia, China, and Japan. This suggests meticulous planning and stringent quality control measures in the space programs of these nations.

<br/><br/>

### Mission Status Percentage:

The distribution of mission statuses provides valuable insights into the outcomes of space missions. While successes dominate the majority, a notable percentage represents failures, partial failures, and prelaunch failures, emphasizing the inherent risks and complexities associated with space exploration endeavors.

<br/><br/>

### Average Price by Country/Rocket/Company:

Analysis of average mission costs by country, rocket type, and company sheds light on the financial investments in space missions. The USA and Kazakhstan lead in average mission costs, with certain rocket types such as Energiya/Buran commanding substantial expenses. Additionally, NASA emerges as the top spender among companies, indicating its significant financial contributions to space exploration.

<br/><br/>

### Total Cost by Year:

The total costs incurred in space missions exhibit an upward trajectory over the years, indicating a growing investment in space exploration endeavors. Significant spikes in costs during specific years underscore major milestones or ambitious projects undertaken during those periods.
<br/><br/>

### Mission by Rocket Status:

The distribution of rockets by status reveals that a majority are retired, reflecting the evolution and turnover of space exploration technologies. However, a notable portion remains active, indicating ongoing missions and the utilization of modern rocket systems in contemporary space endeavors.

<br/><br/>

## Conclusion:

The analysis of space mission data provides invaluable insights into the evolution, trends, and dynamics of space exploration efforts over the years. Through comprehensive data processing, visualization, and interpretation, several key findings have emerged, shaping our understanding of the space industry landscape.

The exploration of mission trends reveals distinct patterns in mission frequency, success rates, and financial investments. Notably, the dominance of the United States and Russia in space missions underscores their longstanding contributions and leadership in the field. Furthermore, the resurgence of space missions in recent years highlights renewed interest and investments in space exploration, driving innovation and technological advancements.

The analysis also sheds light on the financial aspects of space missions, with NASA emerging as a major financial contributor, reflecting its commitment to pioneering space exploration endeavors. Additionally, the distribution of mission costs by country, rocket type, and company underscores the significant investments required to execute successful space missions, highlighting the collaborative efforts of nations and organizations worldwide.

Moreover, the exploration of mission success rates by country and company underscores the importance of meticulous planning, rigorous quality control measures, and innovative technologies in ensuring mission success. While successes dominate the majority of missions, the presence of failures and partial failures underscores the inherent risks and challenges associated with space exploration, emphasizing the need for continuous improvement and resilience in the face of adversity.

Overall, the comprehensive analysis of space mission data provides valuable insights for policymakers, researchers, and space enthusiasts alike, guiding future endeavors and shaping the trajectory of space exploration in the years to come. As humanity continues to push the boundaries of exploration and discovery, the data-driven insights gleaned from this project serve as a foundation for informed decision-making and strategic planning in the pursuit of knowledge and discovery beyond Earth's atmosphere.



Notes: The following graphs were filtered given that there were atleast 5 entries in which we had rocket prices for them so that we could get a more accurate estimation of the values and not have spiked values due to too little data to create an accurate average:

Average price by company
Average price by Country


Also the Success Rate by country graph has a minimum of 10 Total missions completed per country rule so that, once again we can accurately estimate the actual best performers.

