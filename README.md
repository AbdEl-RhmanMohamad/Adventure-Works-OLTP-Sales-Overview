# AdventureWorks-OLTP-Sales-Overview

## Project Overview
This repository contains a Power BI project using the Adventure Works Database. The project aims to extract, model, and analyze sales data to provide actionable insights for business decision-making.

Data Source
The data source for this project is the Adventure Works OLTP database, accessed through Direct Query. For more details on the Adventure Works database and how to install and configure it, visit the [official documentation](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver15&tabs=ssms).

## Data Extraction
The following tables and views were extracted to form the basis of the analysis:

- Sales.SalesOrderHeader
- Sales.SalesOrderDetail
- Sales.vSalesPerson (view)
- Sales.SalesTerritory
- Purchasing.ShipMethod
- Production.Product
- Production.ProductSubcategory
- Production.ProductCategory
- Status (added based on the ufnGetSalesOrderStatusText function)
- Date (created using Power Query)

  ### AdventureWorks-OLTP ERD
  ![OLTP Schema](https://github.com/user-attachments/assets/aea0e6dc-f608-4032-a02b-56887b077945)

  ### M language Code used to create the Date table
```
let
    // Load  fact table
    FactTable = Fact_SalesOrder,

    // List of date columns in the fact table
    DateColumns = {"OrderDate", "DueDate", "ShipDate"},

    // Extract minimum and maximum dates from each date column and convert to date type
    MinDates = List.Transform(DateColumns, each Date.From(List.Min(Table.Column(FactTable, _)))),
    MaxDates = List.Transform(DateColumns, each Date.From(List.Max(Table.Column(FactTable, _)))),

    // Find overall minimum and maximum dates
    OverallMinDate = List.Min(MinDates),
    OverallMaxDate = List.Max(MaxDates),

    // Generate a list of dates from the overall minimum date to the overall maximum date
    DateList = List.Dates(OverallMinDate, Duration.Days(OverallMaxDate - OverallMinDate) + 1, #duration(1, 0, 0, 0)),

    // Convert the list to a table and add the necessary columns
    DateTable = Table.FromList(DateList, Splitter.SplitByNothing(), {"Dates"}),
    #"Changed Type" = Table.TransformColumnTypes(DateTable,{{"Dates", type date, "ddd/MMM/YYYY"}}),

    // Add the Year column
    Year = Table.AddColumn(#"Changed Type", "Year", each Date.Year([Dates]), Int64.Type),

    // Add the Quarter Number column
    QuarterNumber = Table.AddColumn(Year, "Quarter", each Date.QuarterOfYear([Dates]), Int64.Type),

    // Add the Quarter column as "Q" + Quarter Number
    Quarter = Table.AddColumn(QuarterNumber, "Quarter Name", each "Q" & Text.From([Quarter]), type text),

    // Add the Month column
    Month = Table.AddColumn(Quarter, "Month", each Date.Month([Dates]), Int64.Type),

    // Add the Month Name column
    MonthName = Table.AddColumn(Month, "Month Name", each Date.ToText([Dates], "MMM"), type text),

    // Add the Day of Week column
    DayOfWeek = Table.AddColumn(MonthName, "Day", each Number.Mod(Date.DayOfWeek([Dates], Day.Sunday) + 1, 7), Int64.Type),

    // Add the Day Name column
    DayName = Table.AddColumn(DayOfWeek, "Day Name", each Date.ToText([Dates], "ddd"), type text)

in
    DayName
```

## Data Modeling
The data was modeled into a star schema to ensure efficient querying and analysis. The star schema structure included:

**Fact Table**: Containing transactional data (e.g., sales orders).
**Dimension Tables**: Containing descriptive data (e.g., products, sales territories, ship methods).

![Power BI Star Schema Model](https://github.com/user-attachments/assets/46990e52-b810-46ee-a420-036d3d22e094)

## Data Transformation
Several transformations were applied to clean and prepare the data for analysis:

1- Renamed tables and columns for clarity.
2- Removed unused columns to streamline the dataset.

## Measures
The following measures were created to facilitate comprehensive analysis:

- Total Orders ***$31.47K***.
- Total SubTotal Measure ***$109.85M***.
- Total Tax Measure ***$10.19M***.
- Total Freight Measure ***$3.18M***.
- Total Due Measure ***$123.22M***.
- Total Quantity Measure ***275K***.
- Total Orders by Order Date vs. Ship Date vs. Due Date (using **USERELATIONSHIP** function on DAX)

## Visualization and Insights
The Power BI dashboard created from this project provides insights into various aspects of the sales data, including:

- Order trends over time.
- Sales performance by territory, product category, and subcategory.
- Shipping methods and their usage.
- Sales performance of top salespersons.
- Total Orders by Status.
- Total Orders by Ship Method.
- Total Orders by Category, SubCategory, Product.
- Total Orders by Flag Online/Offline.
- Total Orders and Total Due by Territory.
- Top 10 Sales Persons by Total Orders.

![Dashboard 1](https://github.com/user-attachments/assets/fed4b2d6-7cc4-428a-b114-ffda4f7fd6c3)

![Dashboard 2](https://github.com/user-attachments/assets/d712fe4d-ca89-4d81-b3c4-042a9f418a68)

![Dashboard 3](https://github.com/user-attachments/assets/2ae7f408-2117-4859-9978-e04d1572267f)

![Dashboard 4](https://github.com/user-attachments/assets/f87191be-95ae-461f-ae90-b678fe40eaf3)


## Conclusion
This Power BI project demonstrates the comprehensive process of extracting, modeling, and analyzing sales data from the Adventure Works database. Key points include:

1. **Data Source**: Utilized the Adventure Works OLTP database with Direct Query.
2. **Data Extraction**: Retrieved data from essential tables, views and function.
3. **Date Table Creation**: Implemented an **M language** script to generate a comprehensive date dimension table for enhanced time-based analysis.
4. **Data Transformation**: Cleaned and prepared the dataset by renaming tables and columns and removing unused data.
5. **Data Modeling**: Structured data into a star schema for efficient querying and analysis.
6. **Measures**: Created a set of critical measures for analysis, including the use of USERELATIONSHIP function to manage Inactive Relationships in a Role-Playing Dimension.
7. **Visualization**: Developed insightful dashboards to highlight trends, sales performance, and key business metrics.
