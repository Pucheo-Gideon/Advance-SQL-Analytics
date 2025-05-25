# Retail Intelligence Using Advanced SQL: Customer, Sales, and Revenue Analysis

## Project Overview

Major types of Data Analysis and roles in Data Analytics requires the application of SQL to get the job done and the mastery of SQL has an unopposable argument. 
The essence of this project is particularly focused on how Window Functions are a powerful feature within SQL and how their granular capabilities give more analytical 
power to Data Analyst, Scientist and an Engineers for solving critical business problems.

## Data Source
The used for this project is the popular AdventureWorks2021 culled from the Microsoft learning platform. The Data came with multiple tables but for the sake of this project, 
I reduce the number of tables to work with. The tables for this projects includes: Sales, Customers, Product and Territory. 

-  The Sales table has 13 columns and over 58,191 rows. 
-  Product table has 13 columns and over 600 rows of unique records
-  Customers table has 20 columns and over 18,000 Unique Cutomers
-  Sales Territory table comprises of 4 columns and 11 distinct
  
Micrososft SQL Server is the RDBMS I used for the entirety of this project to solve the business problems. [You can click here to watch how to download Ms SQL Server](https://youtu.be/ndNE3Z8kKjU?si=OolaSBBjRxnWxdrI)

## Data Cleaning and Preparation 
The Adventureworks tables were pretty much in a good shape and did not require extensive data cleaning. However due diligence of profiling and exploring the entire tables was observed to ensure all the tables were in the best condition possible for the analysis.

## Exploratory Data Analysis
The relevants questions that anchors this project and driving how we explore the dataset and answer critiacal business problems are listed below. Each is paired with its SQL query and results to map the journey from raw data to insight.

## Functions Use
Some of the Advance approaches used in this Project include 
1.  Aggregation function such as COUNT(),SUM, AVG(), Etc
2.  Subqueries
3.  Common Table Expressions - CTE's
4.  CASE Statement
5.  SQL Analytics Functions such as ROW(), DENSE_RANK(), LEAD(), LAG
6.  Value Functions Such as NTILE, FIRST_VALUE, LAST_VALUE ()

# Retail Transactions business Questions 
###1. Rank Top 2 customers by total sales within each territory
```sql 
SELECT * 
FROM [Customers ]

SELECT * --  Using the WHERE Clause, Fetch only the Top 5 Customers by Sales Per Territory
FROM (
		SELECT *, -- Ranked all The Customers based on their Total Transaction value 
				DENSE_RANK() OVER (PARTITION BY Sales.Customers_Country ORDER BY Sales.Sales DESC) AS Ranked
		FROM (
				SELECT  -- Get the Customers' Name, Country and Total Sales 
						C.FullName,
						C.CustomerCountry AS Customers_Country,
						ROUND(SUM(S.SalesAmount),2) AS Sales
				FROM [Customers ] AS C
				INNER JOIN Sales AS S
				ON S.CustomerKey = C.CustomerKey
				GROUP BY C.FullName, C.CustomerCountry
		) AS Sales
) AS Ranked_Customers
WHERE Ranked_Customers.Ranked <= 2

```

[![Result one - Customers](./image/Top 2 customers Per country.png)

