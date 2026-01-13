# 📊 Global Sales Performance Analysis (SQL • Power BI)

Analyzed multi-country sales data to uncover revenue drivers, profitability trends, and regional performance insights using SQL-based data modeling and Power BI dashboards.

---

## 1. Business Objective

The objective of this project is to analyze global sales performance across multiple countries to support data-driven decision-making in:

- Revenue growth strategy  
- Profitability optimization  
- Salesforce performance evaluation  
- Customer and payment behavior insights  

The analysis identifies top-performing regions, products, sales representatives, and customer segments, while evaluating discount impact and seasonal sales trends.

---

## 2. Dataset Overview

- **Source:** Dummy datasets (created for portfolio and learning purposes)
- **Tools Used:** Excel, MS SQL Server, Power BI
- **Countries Covered:**
  - Canada
  - China
  - India
  - Nigeria
  - United Kingdom
  - United States

### Dataset Characteristics
- Transaction-level sales data
- Identical schemas across all country datasets
- Key fields include:
  - Transaction_ID
  - Country
  - Sales_Representative
  - Product
  - Category
  - Quantity_Purchased
  - Cost_Price
  - Price_per_Unit
  - Discount_Applied
  - Customer_Age
  - Customer_Gender
  - Payment_Method
  - Transaction_Date

---

## 3. Data Preparation & Cleaning (SQL)

### Data Consolidation

All country-level tables were combined into a single analytical table for unified analysis.

```sql
SELECT * INTO Sales_Data
FROM (
    SELECT * FROM dbo.sales_Canada
    UNION ALL
    SELECT * FROM dbo.sales_China
    UNION ALL
    SELECT * FROM dbo.sales_India
    UNION ALL
    SELECT * FROM dbo.sales_Nigeria
    UNION ALL
    SELECT * FROM dbo.sales_UK
    UNION ALL
    SELECT * FROM dbo.sales_US
) AS CombinedSales;
```

---

## Handling Missing Values

**Quantity Purchased (Nigeria):** Missing value imputed using average quantity (6 units).
**Price per Unit (United States):** Filled using same-product price from another transaction (12.67).
**Discount Applied:**

```sql

SELECT AVG(Discount_Applied) AS "Average Discount"
FROM Sales_Data;,
```
Filled with average discount value $25.54

**Price per Unit:**

```sql
SELECT AVG(Price_per_Unit) AS "Avg Price"
FROM Sales_Data;
```
Filled with average price $255.63

## Duplicate Validation
```sql
SELECT Transaction_ID, Count(*) AS "Total IDs"
FROM Sales_Data
GROUP BY Transaction_ID
HAVING Count(*) > 1;
```
The output was none

## 4. Feature Engineering

Calculated fields were created directly in SQL:

- Total Sales | Total Cost | Profit

## 5. Analytical Focus Areas

- **Regional Performance**
  - Sales & Profit By Country
    ```sql
    SELECT Country,SUM(Total_Sales) AS "Total Sales",
               SUM(Profit) AS "Total Profit"
    FROM Sales_Data
    GROUP BY Country
    ORDER BY SUM(Profit) DESC;
    
- Product & Category Insights
  - Category Total Sales & contribution % by country
    ```sql
    SELECT Country, Category,
       SUM(Total_Sales) AS "Total Sales",
       SUM(Total_Sales) * 100.0 / SUM(SUM(Total_Sales)) 
          OVER (PARTITION BY Country) AS Contribution_Percent
    FROM Sales_Data
    GROUP BY Country, Category
    ORDER BY Country, Contribution_Percent DESC;
    
- Store Location Analysis
  - Regional hotspots (best-performing cities) Top 5
   ```sql
   SELECT TOP 5 Store_Location, 
       SUM(Total_Sales) AS "Total Sales",
       SUM(Profit) AS Profit
   FROM Sales_Data
   GROUP BY Store_Location
   ORDER BY Profit DESC, "Total Sales" DESC;
  
- Sales Representative Performance
  - Top 5 Sales Rep Globally
   ```sql
   SELECT TOP 5 Sales_Rep,
             SUM(Total_Sales) AS "Total Sales", 
             SUM(Profit) AS Profit
   FROM Sales_Data
   GROUP BY Sales_Rep
   ORDER BY "Total Sales" DESC, Profit DESC;
   ```
   
   - Top 2 Sales Reps per country (Total Sales Rev, Profit) CTE
     ```sql
     WITH RankedSales AS (
     SELECT 
        Country,
        Sales_Rep,
        SUM(Total_Sales) AS Total_Sales,
        SUM(Profit) AS Profit,
        ROW_NUMBER() OVER (
            PARTITION BY Country 
            ORDER BY SUM(Total_Sales) DESC
        ) AS RepRank
     FROM Sales_Data
     GROUP BY Country, Sales_Rep
     )
     SELECT 
        Country,
        Sales_Rep,
        Total_Sales,
        Profit,
        RepRank
     FROM RankedSales
     WHERE RepRank <= 2
     ORDER BY Country, RepRank;
     
- Customer Demographics
  - Total Sales/Profitability/Average Sales/Average by demographic segment
   ```sql
   SELECT Customer_Gender, 
       SUM(Total_Sales) AS "Total Sales",
       AVG(Total_Sales) AS "Average Sales",
       SUM(Profit) AS "Total Profit",
       AVG(Profit) AS "Average Profit"
   FROM Sales_Data
   GROUP BY Customer_Gender
   ORDER BY "Total Sales" DESC, "Average Sales" DESC, "Total Profit" DESC, "Average Profit" DESC;
   
- Payment Method Trends
   - Sales/Profit By Payment method
   ```sql
   SELECT Payment_Method,
       SUM(Total_Sales) AS "Total Sales",
       AVG(Total_Sales) AS "Average Sales",
       SUM(Profit) AS "Total Profit",
       AVG(Profit) AS "Average Profit"
   FROM Sales_Data
   GROUP BY Payment_Method
   ORDER BY "Total Sales" DESC, "Average Sales" DESC, "Total Profit" DESC, "Average Profit";
   
- Time-Based Trends (Monthly, Quarterly, Daily)
  - Monthly sales/profit trend Using a CTE to return month names as well and determine seasonal peaks.
    ```sql
    WITH MonthCTE AS (
    SELECT 
        MONTH(Date) AS MonthNumber,
        CASE 
            WHEN MONTH(Date) = 1 THEN 'January'
            WHEN MONTH(Date) = 2 THEN 'February'
            WHEN MONTH(Date) = 3 THEN 'March'
            WHEN MONTH(Date) = 4 THEN 'April'
            WHEN MONTH(Date) = 5 THEN 'May'
            WHEN MONTH(Date) = 6 THEN 'June'
            WHEN MONTH(Date) = 7 THEN 'July'
            WHEN MONTH(Date) = 8 THEN 'August'
            WHEN MONTH(Date) = 9 THEN 'September'
            WHEN MONTH(Date) = 10 THEN 'October'
            WHEN MONTH(Date) = 11 THEN 'November'
            WHEN MONTH(Date) = 12 THEN 'December'
        END AS MonthName,
        Total_Sales, Profit
    FROM Sales_Data
    )
    SELECT 
        MonthNumber AS [Month],
        MonthName,
        SUM(Total_Sales) AS [Total Sales],
        SUM(Profit) AS Profit
    FROM MonthCTE
    GROUP BY MonthNumber, MonthName
    ORDER BY MonthNumber ASC;
 - Daily Totals Sales and Profit Trend. Using a CTE to asign day numbers to weekdays from 1 -7
   ```sql
   WITH DayNumberCTE AS (
    SELECT 
        DATENAME(WEEKDAY, Date) AS Day_Name,
        CASE
            WHEN DATENAME(WEEKDAY, Date) = 'Monday' THEN 1
            WHEN DATENAME(WEEKDAY, Date) = 'Tuesday' THEN 2
            WHEN DATENAME(WEEKDAY, Date) = 'Wednesday' THEN 3
            WHEN DATENAME(WEEKDAY, Date) = 'Thursday' THEN 4
            WHEN DATENAME(WEEKDAY, Date) = 'Friday' THEN 5
            WHEN DATENAME(WEEKDAY, Date) = 'Saturday' THEN 6
            WHEN DATENAME(WEEKDAY, Date) = 'Sunday' THEN 7
        END AS Day_Number,
        Total_Sales,
        Profit
    FROM Sales_Data
   )
   SELECT 
        Day_Number,
        Day_Name,
        SUM(Total_Sales) AS Total_Sales,
        SUM(Profit) AS Profit
   FROM DayNumberCTE
   GROUP BY Day_Number, Day_Name
   ORDER BY Day_Number ASC;

## 6. Power BI Modeling
**Key Measures (DAX)**
- Total Sales Revenue
  ```powerbi
  Total Sales Rev = SUM(Sales_Data[Total_Sales])
- Total Profit
  ```powerbi
  Total Profit = SUM(Sales_Data[Profit])
- Profit Margin %
  ```powerbi
  Profit Margin % = DIVIDE([Total Profit], [Total Sales Rev], 0)
- Average Order Value
  ```powerbi
  Average Order Value = DIVIDE([Total Sales Rev], DISTINCTCOUNT(Sales_Data[Transaction_ID]), 0)
- Average Deal Size per Sales Rep
  ```powerbi
  Average Deal Size = AVERAGE(Sales_Data[Total_Sales])
  ```
- Discount Impact (Revenue lost due to discount given to customers)
  ```powerbi
  Discount Impact (Tot Rev Lost) = SUMX(
    Sales_Data,
    (Sales_Data[Price_per_Unit] * Sales_Data[Quantity_Purchased]) * (Sales_Data[Discount_Applied] / 100)
  )
Time intelligence implemented using Power Query for month and day extraction.

7️⃣ Key Insights

Profit concentration varies significantly by country

Discount-heavy regions experience margin erosion

A small group of sales reps drives disproportionate revenue

Payment preferences vary strongly by geography

Clear seasonal sales peaks are observable

8️⃣ Business Recommendations

Optimize discount strategies in low-margin regions

Scale best practices from top-performing sales reps

Align campaigns with seasonal demand spikes

Prioritize high-margin product categories

Customize payment offerings by country

9️⃣ Dashboard Overview

The Power BI dashboard provides an executive-level overview of:

Revenue & profit performance

Regional trends

Sales rep rankings

Customer & payment insights

Time-based sales behavior

Dashboard screenshots are available in the /visuals folder.

🛠️ Tools & Skills Demonstrated

SQL (Data Cleaning, Aggregations, Feature Engineering)

Excel (Initial validation)

Power BI (DAX, Power Query, Visualization)

Business Analytics

Dashboard Storytelling

