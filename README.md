# Trucking Analysis

## Table of Contents
- [Data Cleaning](#data-cleaning)
- [Exploratory Analysis](#exploratory-data-analysis)
- [Data Analysis](#data-analysis)
- [Findings](#findings)
- [Recommendation](#recommendation)
- [Limitations](#limitations)


### Project Overview
---

The purpose of this project is to evaluate the state of the trucking industry to assist my father’s trucking business in making a key decision: whether to expand operations or maintain the current scale. This analysis aims to identify market trends, compare current industry metrics to historical downturns, and evaluate the trucking sector's performance in the broader economic context.

### Data Source(s)

Freight Analysis Framework: Sourced from the Department of Transportation via [here](https://catalog.data.gov/dataset/freight-analysis-framework).

### Tools

  - SQL - Data Analysis
  - Python - Cleaning and Visualization
  - Excel - Organizing the Original Dataset

### Data Cleaning

Loaded Raw CSV into Python

```Python
import pandas as pd
# Path to CSV file
file_path = 'Transportation_Services_Index_and_Seasonally-Adjusted_Transportation_Data.csv'
# Read the CSV file
data = pd.read_csv(file_path)
# Preview the first few rows
print(data.head())
```
Filtered for Relevant Columns
 
```python
# List of columns to keep
selected_columns = ['OBS_DATE', 'TRUCK_D11', 'TSI_Freight', 'TSI_Total', 'IND_PRO', 'MANUF', 'INV_TO_SALES']
# Filter the data
filtered_truck_data = data[selected_columns]
# Preview the filtered data
print(filtered_truck_data.head())
```
Saved file
```
# Save the filtered data to a new CSV
filtered_file_path = 'filtered_transportation_data.csv'
filtered_truck_data.to_csv(filtered_file_path, index=False)

print(f"Filtered data saved to {filtered_file_path}")
```
Formatted and Exported file
```
# Load the file (adjust delimiter if needed, e.g., ',' or '\t')
df = pd.read_csv("filtered_transportation_data.csv")
# Save the file with proper formatting
df.to_csv("cleaned_transportation_data.csv", index=False, sep=",")
```

### Exploratory Data Analysis

#### Key Areas

1. Correlation Analysis:
	- Assessed relationships between trucking activity, industrial production, and inventory-to-sales ratio.
	- Key correlations:
		- Truck Activity vs. Industrial Production
		- Truck Activity vs. Inventory-to-Sales Ratio

2. Rescaling and Trend Analysis:

	- Rescaled monthly data to identify seasonality trends in trucking activity post-2021 (post-COVID-19).

3. Historical Comparison:

	- Compared current averages of trucking activity and inventory ratios with historical economic downturns (2008, 2020).

Focus on metrics relevant to decision-making:

- Inventory-to-Sales Ratio: Highlights market health and demand fluctuations.
- Industrial Production: Indicates economic activity levels.
- Trucking Activity: Directly tied to your father's business.

### Data Analysis 

#### Correlation Analysis

<details open>
<summary>Click for Code</summary>
<br>

```sql
SELECT 
	corr(truck_activity_index, industrial_production) AS truck_activity_ind_production,
	corr(truck_activity_index, inventory_to_sales_ratio) AS truck_activity_sales_ratio,
	corr(truck_activity_index, manufacturing_index) AS truck_activity_manufacturing

FROM transportation_analysis;
```
</details>


Outputting 

#### 1.1
<img width="456" alt="Corr Trucking" src="https://github.com/user-attachments/assets/b2e2b77b-4342-487d-b83d-ebf0b6b2f06d">

#### Findings:
	- Positive Correlation between Truck Activity Industrial Production, and Manufacturing
 	- Negative Correlation between Truck Activity and Inventory to Sales Ratio


#### Correlation Visuals

<details open>
<summary>Click for Code</summary>
<br>
	
```Python
import pandas as pd
import matplotlib.pyplot as plt

file_path = 'Transportation_Services_Index_and_Seasonally-Adjusted_Transportation_Data.csv'
data = pd.read_csv(file_path)
df_subset = data[["TSI_Freight", "IND_PRO"]].dropna()

plt.figure(figsize=(8, 6))
plt.scatter(
    df_subset["TSI_Freight"],
    df_subset["IND_PRO"],
    color="blue", alpha=0.7
)
plt.title("Freight Transport Index vs Industrial Production", fontsize=14)
plt.xlabel("TSI_Freight", fontsize=12)
plt.ylabel("IND_PRO", fontsize=12)
plt.grid(alpha=0.5, linestyle="--")
plt.tight_layout()
plt.show()
```
</details>

#### 1.2
<img width="713" alt="Scatter Plot of Freight Index and Industrial Production" src="https://github.com/user-attachments/assets/4773b4ef-e279-406a-bb7e-f5e1a44fc1cf">

<details open>
<summary>Click for Code</summary>
<br>

```python
df_subset_2 = data[["TSI_Freight", "INV_TO_SALES"]].dropna()

plt.figure(figsize=(8, 6))
plt.scatter(
    df_subset_2["TSI_Freight"],
    df_subset_2["INV_TO_SALES"],
    color="blue", alpha=0.7
)
plt.title("Freight Transport Index vs Inventory To Sales Ratio", fontsize=14)
plt.xlabel("TSI_Freight", fontsize=12)
plt.ylabel("INV_TO_SALES", fontsize=12)
plt.grid(alpha=0.5, linestyle="--")
plt.tight_layout()
plt.show()

```
</details>

#### 1.3
<img width="667" alt="Scatter Plot of Freight Index and Inventory to Sales Ratio " src="https://github.com/user-attachments/assets/c12757b6-00cc-4366-be1b-e2c1ae2645ef">

<details open>
<summary>Click for Code</summary>
<br>

```python

df_subset_3 = data[["TSI_Freight", "MANUF"]].dropna()
plt.figure(figsize=(8, 6))
plt.scatter(
    df_subset_3["TSI_Freight"],
    df_subset_3["MANUF"],
    color="blue", alpha=0.7
)
plt.title("Freight Transport Index vs Inventory To Sales Ratio", fontsize=14)
plt.xlabel("TSI_Freight", fontsize=12)
plt.ylabel("MANUF", fontsize=12)
plt.grid(alpha=0.5, linestyle="--")
plt.tight_layout()
plt.show()

```
</details>

#### 1.4
<img width="680" alt="Freight to Manuf Correlation Scatter Plot" src="https://github.com/user-attachments/assets/fe523a1b-dd11-4825-9f3b-0c078d135fb2">

<details open>
<summary>Click for Code</summary>
<br>

#### Historical Averages Comparison

```sql
WITH historical_downturns AS (
    SELECT 
        AVG(INV_TO_SALES) AS avg_inventory_ratio_downturn,
        AVG(TRUCK_D11) AS avg_truck_activity_downturn
    FROM transportation_analysis
    WHERE EXTRACT(year FROM OBS_DATE) IN (2008, 2020)
),

recent_data AS (
    SELECT 
        AVG(INV_TO_SALES) AS avg_inventory_ratio_recent,
        AVG(TRUCK_D11) AS avg_truck_activity_recent
    FROM transportation_analysis
    WHERE OBS_DATE >= '2024-01-01'
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
</details>

#### 1.5
<img width="466" alt="Trucking Downturn" src="https://github.com/user-attachments/assets/ba7f543f-f2f1-42cb-ad6f-65171b4d7992">



<details open>
<summary>Click for Code</summary>
<br>

```python

df['OBS_DATE'] = pd.to_datetime(df['OBS_DATE'])
df = df.sort_values('OBS_DATE')
columns_to_plot = ['IND_PRO', 'TSI_Freight', 'TRUCK_D11']
plt.figure(figsize=(12, 6))
for column in columns_to_plot:
    plt.plot(df['OBS_DATE'], df[column], label=column.replace('_', ' ').title())
plt.xlabel('Observation Date', fontsize=12)
plt.ylabel('Scaled Index', fontsize=12)
plt.title('Industrial Production, Freight Activity, and Trucking Activity', fontsize=14)
plt.legend(fontsize=10)
plt.grid(alpha=0.5, linestyle='--')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```
</details>


#### Line Graph

#### 1.6

<img width="845" alt="Line Graph-Industry_Freight_Trucking" src="https://github.com/user-attachments/assets/20431499-7168-47fb-ba1b-64b3a0377b38">



Medians in bad, good, and recent economic periods 

<details open>
<summary>Click for Code</summary>
<br>



```sql
WITH historical_downturn_median_production AS (
SELECT percentile_cont(0.5) WITHIN GROUP (ORDER BY IND_PRO) AS downturn_median
FROM transportation_analysis
WHERE EXTRACT(year FROM OBS_DATE) IN (2008,2020)
),
historical_upturn_median_production AS (
SELECT percentile_cont(0.5) WITHIN GROUP (ORDER BY IND_PRO) AS upturn_median
FROM transportation_analysis
WHERE EXTRACT(year FROM observation_date) IN (2018)
),
recent_median_production AS (
SELECT percentile_cont(0.5) WITHIN GROUP (ORDER BY IND_PRO) AS recent_median
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

</details>


Results

#### 1.7

<img width="218" alt="Medians for Periods of the Economy" src="https://github.com/user-attachments/assets/1578ec8a-9b98-4380-a71c-8b2fcb7bcb8f">



### Findings 

#### Key Metrics Compared to Historical Downturns:

- Inventory ratios remain high, suggesting weaker demand and greater inventory in idle
- Figure [1.6](#1.6) demonstrates a high level of truck activity and a lower level of production meaning there is a greater supply of trucks compared to production 
- Trucking activity has not recovered to pre-COVID levels


### Recommendation

Given the current data:

	- Expanding operations is not recommended due to weak market demand and higher supply of trucks compared to production
	- Focus on optimizing costs and targeting high-demand periods in late Q4 and early Q4
 	- Wait for a decrease in truck activity and an increase in production

### Limitations

	- Economic Factors: The analysis does not account for external variables such as geopolitical events or elections.
	- Data Granularity: Industry-level data may not fully reflect regional construction trucking trends.

[Go Back Up](#trucking-analysis)
