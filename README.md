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
### 1. Rank Top 2 customers by total sales within each territory
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

![1](https://github.com/user-attachments/assets/ec60e40d-f3ec-40c1-952a-a41e93050b36)


### 2. Fetch the customers who made just one purchase
```sql

SELECT Purchases.* 
FROM (
      -- This subquery in combination with Row_Number() and LEAD?() Identifies customers based on their number of purchases per Order. 
      -- Based on the Logic, "None" is return for customers who didn't make another purchase after the first or their Last purchase, 
      -- The "SaleOrderNumbers" is return for customers who made further purchases after the first.
      SELECT 
            S.SalesOrderNumber,
            C.FullName,
            S.ProductKey,
            P.ProductName,
            ROW_NUMBER () OVER(PARTITION BY S.SalesOrderNumber ORDER BY S.SalesOrderNumber ) AS No_of_Purchase,
            LEAD(S.SalesOrderNumber, 1, 'None') OVER(PARTITION BY S.SalesOrderNumber ORDER BY S.SalesOrderNumber DESC) AS "Next_Purchase?"
      FROM Sales  AS s
      INNER JOIN [Customers ] AS c
      ON S.CustomerKey = C.CustomerKey
      INNER JOIN Product AS  P 
      ON P.ProductKey = S.ProductKey
      GROUP BY S.SalesOrderNumber,
        C.FullName,
        S.ProductKey,
        P.ProductName

 ) AS Purchases 

WHERE Purchases.No_of_Purchase = 1 
AND Purchases.[Next_Purchase?] = 'None'
```

![3 - result set](https://github.com/user-attachments/assets/4b205846-ea6f-4c03-a2d3-7e3213507cdb)

### 3. Segment Customers by their Purchase intervals and compare their spending habits or Purchase value
```sql
WITH CTE_Purchase_Days AS (
SELECT
		
		S.CustomerKey, 
		C.FullName, 
		s.OrderDate AS OrderDate,
		S.SalesAmount,
		LEAD(s.OrderDate) OVER(PARTITION BY S.CustomerKey ORDER BY s.OrderDate) AS NextDay
FROM [Customers ]  AS C
INNER JOIN Sales AS S 
ON S.CustomerKey = C.CustomerKey
),

Calculated_Days_Bewtween_Purchases AS (
SELECT  pd.CustomerKey,
		pd.FullName,
		FORMAT(pd.OrderDate, 'dd-MMM-yyyy') AS OrderDate,
		FORMAT(pd.NextDay, 'dd-MMM-yyyy') AS Next_OrderDate,
		ROUND(SUM(pd.SalesAmount),2) as Avg_SalesAmount,
	    AVG(DATEDIFF(DAY, pd.OrderDate,pd.NextDay))  AS DaysBetween
FROM CTE_Purchase_Days AS pd
GROUP BY pd.CustomerKey,
		pd.FullName,
		FORMAT(pd.OrderDate, 'dd-MMM-yyyy') ,
		FORMAT(pd.NextDay, 'dd-MMM-yyyy')
)

SELECT 
		CASE 
            WHEN DaysBetween <= 30 THEN '0-1 Month'
            WHEN DaysBetween BETWEEN 31 AND 90 THEN '1-3 Months'
            WHEN DaysBetween BETWEEN 91 AND 180 THEN '3-6 Months'
            WHEN DaysBetween BETWEEN 181 AND 270 THEN '6-9 Months'
            WHEN DaysBetween BETWEEN 271 AND 365 THEN '9-12 Months'
            ELSE '1 Year and Above'
        END AS Customers_Purchase_Interval,
		COUNT(*) Purchase_Count,
		ROUND(AVG(cd.Avg_SalesAmount),2) AS Average_Purchase_Value
FROM Calculated_Days_Bewtween_Purchases AS cd
GROUP BY CASE 
            WHEN DaysBetween <= 30 THEN '0-1 Month'
            WHEN DaysBetween BETWEEN 31 AND 90 THEN '1-3 Months'
            WHEN DaysBetween BETWEEN 91 AND 180 THEN '3-6 Months'
            WHEN DaysBetween BETWEEN 181 AND 270 THEN '6-9 Months'
            WHEN DaysBetween BETWEEN 271 AND 365 THEN '9-12 Months'
            ELSE '1 Year and Above'
        END
ORDER BY 3 DESC
```

![5](https://github.com/user-attachments/assets/1c6fd84e-324b-45cf-957a-97a8204d5940)
