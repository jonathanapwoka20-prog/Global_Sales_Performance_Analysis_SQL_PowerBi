# 📊 Global Sales Performance Analysis (SQL • Power BI)

Analyzed multi-country sales data to uncover revenue drivers, profitability trends, and regional performance insights using SQL-based data modeling and Power BI dashboards.

---

## 1️⃣ Business Objective

The objective of this project is to analyze global sales performance across multiple countries to support data-driven decision-making in:

- Revenue growth strategy  
- Profitability optimization  
- Salesforce performance evaluation  
- Customer and payment behavior insights  

The analysis identifies top-performing regions, products, sales representatives, and customer segments, while evaluating discount impact and seasonal sales trends.

---

## 2️⃣ Dataset Overview

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

## 3️⃣ Data Preparation & Cleaning (SQL)

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
