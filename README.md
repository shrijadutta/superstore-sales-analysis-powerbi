# Superstore Sales Analysis
![Dashboard Screenshot](https://github.com/shrijadutta/superstore-sales-analysis-powerbi/blob/main/Superstore_Sales_Analysis_Dashboard.png?raw=true)
*Figure: Dashboard Overview*

## Project Overview
This project analyzes sales data from 2014 to 2017 to track revenue, profit, and quantity trends. It helps identify top products, customers, and regions while showing how performance improved each year.

## Data Source
The dataset used is `Sample - Superstore.csv`, sourced from Kaggle. This dataset provides detailed sales, profit, quantity, and customer information across products, regions, and time periods for a retail company.

## Tools Used
- Microsoft Power BI – Used for building interactive dashboards and data visualizations.
- Power Query – Used for data cleaning, normalization, and transformation.
- DAX (Data Analysis Expressions) – Used to create custom measures, calculated columns, and implement business logic within the data model.

## Data Preparation
- Loaded the superstore.csv dataset into Power BI.
- Performed data transformations in Power Query:
  - Changed data types of date columns using locale settings to ensure correct formatting.
  - Normalized the main Superstore table into:
       - Sales (Fact Table)
       - Orders, Products, and Customers (Dimension Tables)
  - Created a dedicated Date table.
  - Verified data integrity:
       - Removed duplicate values where necessary.
       - Ensured consistency in key fields used for relationships.
- Built a star schema by establishing relationships:
  - Connected the Sales fact table to all related dimension tables.
  - Built proper relationships between tables to ensure accurate data connections and filtering.
- Created a new table named *Slicer_Values*:
  - Contains a column with three entries: **Revenue**, **Profit**, and **Quantity**.
  - Used as a disconnected table to enable dynamic switching between YTD and PYTD metrics for the selected measure in reports.

## DAX Measures and Calculated Columns
  
### 1. Added a calculated column named `Inpast` in the Date table:
- Uses the last available order date to calculate the same period in the previous year.
- Flags each date as `TRUE` if it falls within or before that range.
- Used to support PYTD comparisons in visualizations and measures.
```
Inpast = 
var last_order_date = max(orders[Order Date])
var last_order_date_PY = EDATE(last_order_date,-12)
return
Date_Table[Date] <= last_order_date_PY
```
### 2. Created base measures:
``` 
Total_Revenue = SUM(Sales[Sales])
Total_Quantity = SUM(Sales[Quantity])
Total_Profit = SUM(Sales[Profit])
Profit_Margin = DIVIDE([Total_Profit],[Total_Revenue])
order_count = DISTINCTCOUNT(Sales[Order ID])
```
### 3. YTD and PYTD Measures:
- Year-To-Date (YTD) and Previous Year-To-Date (PYTD) measures were created to calculate revenue, profit, and quantity for the current and previous years, supporting year-over-year comparisons up to the last available date in the dataset

- A custom column InPast was used to ensure accurate alignment with the current period's date range.
```
YTD_Revenue  = TOTALYTD([Total_Revenue], Orders[Order Date])

YTD_Quantity = TOTALYTD([Total_Quantity], Orders[Order Date])

YTD_Profit   = TOTALYTD([Total_Profit], Orders[Order Date])

PYTD_Revenue =
CALCULATE(
    [Total_Revenue],
    SAMEPERIODLASTYEAR('Date_Table'[Date]),
    'Date_Table'[InPast] = TRUE
)

PYTD_Quantity =
CALCULATE(
    [Total_Quantity],
    SAMEPERIODLASTYEAR('Date_Table'[Date]),
    'Date_Table'[InPast] = TRUE
)

PYTD_Profit =
CALCULATE(
    [Total_Profit],
    SAMEPERIODLASTYEAR('Date_Table'[Date]),
    'Date_Table'[InPast] = TRUE
)
```
### 4. Dynamic YTD vs PYTD Comparison using SWITCH:
- To allow flexible, user-driven comparison between current year and previous year metrics, dynamic YTD and PYTD measures were created using SWITCH and SELECTEDVALUE.
```
YTD_values = 
VAR selected_value = SELECTEDVALUE(Slicer_Values[Values])
VAR result = 
    SWITCH(
        selected_value,
        "Revenue", [YTD_Revenue],
        "Quantity", [YTD_Quantity],
        "Profit", [YTD_Profit],
        BLANK()
    )
RETURN
result

PYTD_values = 
VAR selected_value = SELECTEDVALUE(Slicer_Values[Values])
VAR result = 
    SWITCH(
        selected_value,
        "Revenue", [PYTD_Revenue],
        "Quantity", [PYTD_Quantity],
        "Profit", [PYTD_Profit],
        BLANK()
    )
RETURN
result
```
- A measure YTD vs PYTD was created to calculate the difference between the current and previous year's performance for the selected metric.
```
YTD vs PYTD = [YTD_values] - [PYTD_values]
```
### 5. Customer Analysis Measures
- A measure was created to calculate the total number of unique customers based on distinct customer IDs in the Sales data.
```
Total_Customers = DISTINCTCOUNT(Sales[Customer ID])
```
- Another measure was created to calculate the number of customers with multiple orders, highlighting customer retention.
```
Retained_Customers = 
COUNTROWS(
    FILTER(
        GROUPBY(Sales, Sales[Customer ID]),
        [order_count] > 1
    )
)
```
- This measure calculates the customer retention rate, representing the proportion of customers who made repeat purchases.
```
Customer_Retention_Rate = 
DIVIDE([Retained_Customers], [Total_Customers])
```
### 6. Dynamic Chart Titles
- To improve report interactivity and clarity, dynamic chart titles were implemented using DAX measures that reflect slicer selections or current filters.
Example:
```
Waterfall_chart_title = SELECTEDVALUE(Slicer_Values[Values]) & " | YTD vs PYTD | Month-Product Category-Product Name | "
& SELECTEDVALUE(Date_Table[Date].[Year])
```
## Exploratory Data Analysis (EDA)
The EDA was performed on 2014–2017 sales data, analyzing Revenue, Quantity Sold, and Profit across customer segment, product categories, states, and individual customers. Key insights from this analysis include:
### 1. Revenue Analysis
- **Year-over-Year growth**: Revenue increased steadily from 2014 to 2017.
  - 2014: $484.25K
  - 2015: Slight decline to $470.53K (-$13.71K)
  - 2016: Significant growth to $609.21K (+$138.67K)
  - 2017: Revenue peaked at $733.22K (+$124.01K)
- **Product Categories**:
  - **Technology** consistently led in revenue contribution across all years.
  - **Office** Supplies held a stable mid-position.
  - **Furniture**, while often third in revenue, saw occasional spikes in Q3 and Q4.
- **Customer Segments**:
  - **Consumer segment** was the top revenue generator annually.
  - **Corporate followed**, with **Home Office** trailing, though all segments showed upward trends by 2017.
- **State-Level Revenue Analysis**:
  - Top revenue states shifted yearly, with California and New York consistently leading, joined by others like Texas, Pennsylvania, Indiana, Virginia, and Washington across 2014–2017.
  - States such as Missouri, Kansas, and South Dakota showed low revenue contributions across all four years.

### 2. Quantity Analysis
- **Year-over-Year Growth**: Steady increase in total quantity sold.
  - 2014: 7.58K units
  - 2015: 7.98K units
  - 2016: 9.84K units
  - 2017: 12.48K units
- **Product Categories**:
  - *Office Supplies* sold the highest number of units, despite not always being the top revenue driver.
  - *Technology* had higher revenue per unit, evident from fewer units sold but high revenue.
  - *Furniture* showed stable sales but inconsistent profit performance.
- **Customer Segments**:
  - All three customer segments — Consumer, Corporate, and Home Office — showed consistent growth in quantities purchased from 2014 to 2017.
  - The **Consumer segment** led in volume, while **Corporate** and **Home Office** segments grew steadily.
- **State-Level Quantity Analysis**:
  - Frequent leaders included California, Texas, New York, and Washington.
  - Underperformers were states like South Dakota, Montana, and Vermont.

### 3. Profit Analysis
- **Profit Growth**: Consistent growth year-over-year.
  - 2014: $49.54K
  - 2015: $61.62K (+24.4%)
  - 2016: $81.80K (+32.7%)
  - 2017: $93.44K (+14.2%)
- **Profit Margin**:
  - Profit margin increased consistently from 10.23% in 2014 to 13.10% in 2015, reaching a peak of 13.43% in 2016, before dipping slightly to 12.74% in 2017.
- **Product Categories**:
  - **Technology** was the highest profit contributor, mostly strong in Q1 and Q3.
  - **Office Supplies** showed stable profit across all quarters.
  - **Furniture** fluctuated, sometimes posting losses, especially in Q1 (e.g., 2015 and 2017).
- **Customer Segments**:
  - Profit rose steadily from 2014–2017, with the **Consumer** segment remaining the most profitable, followed by **Corporate** and **Home Office**.
- **State-level Profit analysis**:
  - California and Washington were the most consistently profitable states across 2014–2017, followed closely by New York, Virginia, Arizona, Indiana, Pennsylvania, Colorado, and Kentucky.
  - States such as Texas, Illinois, Ohio, and Tennessee consistently appeared among the lowest profit contributors.

### 4. Customer Analysis
- **Retention Rate**: Improved consistently
  - 2014: 44.2%
  - 2015: 54.28%
  - 2016: 59.09%
  - 2017: 71.14%
- **Top Customers**:
  - Top-performing customers varied by year. For example, Nathan Mautz and Mitch Webber were key contributors in 2014–2015, while Raymond Buch, Hunter Lopez, and Tom Ashbrook emerged as top revenue drivers in 2017.
  - These customers contributed significantly to both revenue and profit.

## Summary of Key Findings
- Steady growth in all core metrics (Revenue, Quantity, Profit) from 2014 to 2017.
- **Q3** and **Q4** are typically the strongest quarters for both revenue and profit.
- Clear dominance of the **Consumer Segment** in all areas.
- **Technology** outperformed other categories in profitability.
- Customer retention improved significantly, indicating stronger customer relationships.
  
## Recommendations
- Supporting strong-performing segments should be continued, as the Consumer segment and Technology category have shown consistent growth.
- High-performing states like California, New York, Texas, and Washington should be leveraged for further opportunities.
- Low-performing states can be explored further to identify potential areas for improvement.
- Customer retention efforts should be maintained, given the steady improvement observed.
- Exploring strategies to maintain steady sales throughout the year can help minimize dips in certain months.
- Growing segments like Home Office should be supported with tailored strategies to encourage further growth.
  
## Author
**SHRIJA DUTTA**  
• [GitHub](https://github.com/shrijadutta)  • [Email](mailto:shrijadutta19@gmail.com)

