# Trucking Analysis

## Table of Contents

-[Recommendation](#Recommendation)


### Project Overview
---

The project is intended to help my father's trucking business by answering the question "Should I expand my business or hold off?". The question at hand is answered by highlighting the trends within the trucking market, comparing the current numbers to the numbers of economic downturns, and assesing where the current market is in relation to prior markets.

### Data Source(s)

The Dataset is called Freight Analysis Framwork and is sourced via the Department of Transportation is sourced via [here](https://catalog.data.gov/dataset/freight-analysis-framework).
### Tools

This is where I would list the tools used 

  - SQL - Data Analysis
  - Python - Cleaning and Visualization
  - Excel - Organizing the Original Dataset

### Data Cleaning/Preparation

After the Data was downloaded I loaded data into Jupyter Notebook via

```Python
import pandas as pd
# Path to CSV file
file_path = 'Transportation_Services_Index_and_Seasonally-Adjusted_Transportation_Data.csv'
# Read the CSV file
data = pd.read_csv(file_path)
# Preview the first few rows
print(data.head())
```
Then I filtered the CSV for only the columns I needed via 
```python
# List of columns to keep
selected_columns = ['OBS_DATE', 'TRUCK_D11', 'TSI_Freight', 'TSI_Total', 'IND_PRO', 'MANUF', 'INV_TO_SALES']
# Filter the data
filtered_truck_data = data[selected_columns]
# Preview the filtered data
print(filtered_truck_data.head())
```
Then Saved file
```
# Save the filtered data to a new CSV
filtered_file_path = 'filtered_transportation_data.csv'
filtered_truck_data.to_csv(filtered_file_path, index=False)

print(f"Filtered data saved to {filtered_file_path}")
```
Then formatted and exported the file 
```
# Load the file (adjust delimiter if needed, e.g., ',' or '\t')
df = pd.read_csv("filtered_transportation_data.csv")
# Save the file with proper formatting
df.to_csv("cleaned_transportation_data.csv", index=False, sep=",")
```
Then Opened it in PostgresSQL

### Exploratory Data Analysis

To analyze this project I first looked at the correlation between Trucking Production Column and the columns of manufacturing and inventory to sales ratio. Then I rescaled the Data, grouped by year and analyzed trucking production by month since 2021. Years past 2021 indicate data post covid and more recent data. Next step was analyzing where the current market is in relation to previous markets. I did this by analyzing historic downturns and recent data in comparing production and inventory to sales ratio.

### Data Analysis 

Correlation Between Truck Activity and Inventory Sales Ratio, Manufacturing
```sql
SELECT 
	corr(truck_activity_index, industrial_production) AS truck_activity_ind_production,
	corr(truck_activity_index, inventory_to_sales_ratio) AS truck_activity_sales_ratio,
	corr(truck_activity_index, manufacturing_index) AS truck_activity_manufacturing

FROM transportation_analysis;
```
Outputting 


<img width="456" alt="Corr Trucking" src="https://github.com/user-attachments/assets/b2e2b77b-4342-487d-b83d-ebf0b6b2f06d">

Graph the Scatter Plots of the Correlations with Python

```Python




```sql
WITH historical_downturns AS (
    SELECT 
        AVG(inventory_to_sales_ratio) AS avg_inventory_ratio_downturn,
        AVG(truck_activity_index) AS avg_truck_activity_downturn
    FROM transportation_analysis
    WHERE EXTRACT(year FROM observation_date) IN (2008, 2020)
),

recent_data AS (
    SELECT 
        AVG(inventory_to_sales_ratio) AS avg_inventory_ratio_recent,
        AVG(truck_activity_index) AS avg_truck_activity_recent
    FROM transportation_analysis
    WHERE observation_date >= '2024-01-01'
),
comparison AS (
    SELECT 
        ROUND(historical_downturns.avg_inventory_ratio_downturn, 4) AS downturn_inv_ratio,
        ROUND(recent_data.avg_inventory_ratio_recent, 4) AS recent_avg_inv_ratio,
        ROUND(historical_downturns.avg_truck_activity_downturn,4) AS downturn_avg_truck_act,
        ROUND(recent_data.avg_truck_activity_recent,4) AS recent_truck_act
    FROM historical_downturns, recent_data)
SELECT * FROM comparison;
```
Output

<img width="466" alt="Trucking Downturn" src="https://github.com/user-attachments/assets/ba7f543f-f2f1-42cb-ad6f-65171b4d7992">


Compare Max Production year to Max Trucking Year

```sql
WITH max_production_year AS 
(
SELECT EXTRACT(year FROM observation_date) AS year, MAX(industrial_production) AS peak_industry_production
FROM transportation_analysis
GROUP BY year
),

max_trucking_year AS (
SELECT EXTRACT(year FROM observation_date) AS year, MAX(truck_activity_index) AS peak_trucking_activity
FROM transportation_analysis
GROUP BY year
)

SELECT pro.year, pro.peak_industry_production, truck.peak_trucking_activity
FROM max_production_year AS pro
LEFT JOIN max_trucking_year AS truck
ON pro.year = truck.year
ORDER BY truck.peak_trucking_activity DESC;
```

Comparing medians between bad, good, and recent economic periods 


```sql
WITH historical_downturn_median_production AS (

SELECT percentile_cont(0.5) WITHIN GROUP (ORDER BY industrial_production) AS downturn_median

FROM transportation_analysis

WHERE EXTRACT(year FROM observation_date) IN (2008,2020)

),

historical_upturn_median_production AS (

SELECT percentile_cont(0.5) WITHIN GROUP (ORDER BY industrial_production) AS upturn_median

FROM transportation_analysis

WHERE EXTRACT(year FROM observation_date) IN (2018)

),

recent_median_production AS (

SELECT percentile_cont(0.5) WITHIN GROUP (ORDER BY industrial_production) AS recent_median

FROM transportation_analysis

WHERE EXTRACT(year FROM observation_date) IN (2023,2024)

),



comparison_analysis AS (

SELECT ROUND(CAST(downturn_median AS NUMERIC),2) AS downturn, ROUND(CAST(upturn_median AS NUMERIC),2) AS upturn, ROUND(CAST(recent_median AS NUMERIC),2) AS recent

FROM historical_downturn_median_production, historical_upturn_median_production, recent_median_production

)

SELECT *

FROM comparison_analysis;


```


### Results/Findings 

This section would go over the results/findings that I found

### Recommendation

Given the current average inventory ratio being close to the inventory ratio during economic downturns of 2008 and 2020, expanding the trucking industry would not be advised. Looking at average truck activity by month, the industry benefits the most when it was around late December and early next year.


### Limitations

The data itself, while from the US government, fails to factor in external variables such as the affect that elections have on the economy and the geopolotics at play.

### References

Where I found the code and what or who helped indirectly 
