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

| FullName | 	Customers_Country | 	Sales |  	Ranked |
|----------| ---------------------| ----------| ---------------|
| Xu, Jaclyn |	Australia |	9613.63 | 1 |
| Xu, Rafael |	Australia |	8786.19 | 	2 |
| Bryant, Isabella |	Canada |	6060.5 | 	1 |
| Miller, Chloe |	Canada |	6056.51 |	2 |
| Nara, Nichole |	France |	13295.38 |	1 |
| Henderson, Kaitlyn |	France	| 13294.27 |	2 |
| Xu, Franklin |	Germany |	11284.97 |  	1 |
| Vazquez, Ricky |	Germany | 10580.35 |	2 |
| Sanz, Marie |	United Kingdom |	8460.25 |	1 |
| Patel, Emmanuel |	United Kingdom | 	8451.26 |	2 |
| Hernandez, Jordan |	United States |	7234.26 |	1 |
| Richardson, Trinity |	United States |	6719.06 |	2 |

#### Purpose
The goal of this query is to enable senior stakeholders, as well as marketing and sales representatives, to identify the top-performing customers by total sales within each sales territory. This is meant to support data-driven decision-making for targeted marketing campaigns, relationship management, and customer loyalty strategies

____________________________________________________________________
### 2. Fetch the customers who made just one purchase
```sql

SELECT Purchases.* 
FROM (
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

**Approach for solving the questions** 
-	This subquery in combination with Row_Number() and LEAD?() Identifies customers based on their number of purchases per Order. 
-	Based on the Logic, "None" is return for customers who didn't make another purchase after the first or their Last purchase, 
-	The "SaleOrderNumbers" is return for customers who made further purchases after the first.
________________________________________
### 3. What is the total Distribution and Percentage of One time Buyers Per Country

```sql
WITH OneTimeBuyers AS (
SELECT 
		S.SalesOrderNumber,
		C.CustomerKey,
		C.FullName,
		C.CustomerCountry,
		COUNT(S.SalesOrderNumber) AS PurchaseCOunt
FROM Sales  AS s
INNER JOIN [Customers ] AS c
ON S.CustomerKey = C.CustomerKey
GROUP BY S.SalesOrderNumber,
		 C.CustomerKey,
		 C.FullName,
		 C.CustomerCountry
HAVING COUNT(S.SalesOrderNumber) = 1

), 
CountryCount AS (

	SELECT CustomerCountry,
	COUNT(*) As SinglePurchaseCount
	FROM OneTimeBuyers 
	GROUP BY CustomerCountry
)
SELECT 
		CC.CustomerCountry, 
		cc.SinglePurchaseCount,
		ROUND(CAST(cc.singlePurchaseCount * 100.0/ (SELECT COUNT(*) FROM OneTimebuyers) AS FLOAT),2) AS [% of Single Purchase]
FROM CountryCount AS CC
ORDER BY 2 DESC
```

### Result 

| CustomerCountry |	SinglePurchaseCount |	% of Single Purchase |
| ---------------- |------------------------| -----------------------|
| Australia |	3032 |	32.17 |
| United States |	2988 |	31.7 |
| United Kingdom |	922 |	9.78 |
| Canada |	894 |	9.49 |
| France |	796 |	8.45 |
| Germany |	793 |	8.41 |

#### Insight 🔍 
Australia has the highest proportion of One time buyers with over 32%. The United States follows closely behind with almost an equivalent margin of 31% of one time buyers. 

#### Purpose 
The purpose of this query is to help stakeholders identify the distribution of one-time customers across different countries, providing insight into where customer retention may be weakest. By revealing which countries have the highest proportion of customers making only a single purchase, stakeholders can pinpoint regions that may require targeted re-engagement strategies, localized marketing improvements, or service enhancements.

__________________
### 4. Segment Customers by their Purchase intervals and compare their spending habits or Purchase value
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

![5](https://github.com/user-attachments/assets/0c9859e5-7c25-4f65-9b25-301b5db4da7c)

| Customers_Purchase_Interval	| Purchase_Count |	Average_Purchase_Value|
|-------------------------------| ---------------| ---------------------------|
| 9-12 Months |	919 |	1676.23 |
| 6-9 Months |	841 |	881.67 |
| 0-1 Month | 	18355 |	766.81 |
| 1 Year and Above |	21386 |	594.65 |
| 3-6 Months | 1289 |	376.86 |
| 1-3 Months |	1091 |	157.22 |

#### Insight 🔍
Customers with 9-12 months interval have the highest average order value of $1,676. Followed by customers whose purchases happens after 6-9months with an average order value of $881.67. However Customers with purchase intervals 0-1month and 1 Year and above above have highest number of purchase frequency, both scoring 18,355 and 21,386 in purchase frequency respectively.
