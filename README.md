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

#### 🔍 Insight
The query reveals the top two highest-spending customers in each country, with notable leaders such as Nichole Nara in France and Franklin Xu in Germany, both generating sales above $11,000. The results show a relatively close margin between the first and second-ranked customers in most territories, indicating a competitive customer base in terms of spending. Across all regions, these top customers significantly contribute to total sales performance.

#### 🎯 Why This Is Relevant
Identifying the top-performing customers by territory helps stakeholders recognize high-value individuals who drive substantial revenue. This insight supports strategic decisions around personalized relationship management, loyalty programs, and territory-specific marketing. It also helps sales teams prioritize customer retention efforts, ensuring that the business continues to nurture and reward its most valuable clients.

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
Australia has the highest share of one-time buyers at 32.17%, closely followed by the United States at 31.7%. This suggests that these two markets may have retention challenges, with a significant portion of customers not returning after their first purchase.

#### 🎯Why is this relevant?
The purpose of this query is to help stakeholders identify the distribution of one-time customers across different countries, providing insight into where customer retention may be weakest. By revealing which countries have the highest proportion of customers making only a single purchase, stakeholders can pinpoint regions that may require targeted re-engagement strategies, localized marketing improvements, or service enhancements.

__________________

### 3. What is the total Distribution and Percentage of frequent Buyers Per Country


| CustomerCountry |	Return_Purchases |	% of Single Purchase |
| --------------- | ---------------------| -------------------------- |
| United States |	6213 | 	36 |
| Australia	| 3510	 | 20.34 |
| Canada |	2276 |	13.19 |
| United Kingdom |	2025 |	11.73 |
| Germany |	1619 |	9.38 |
| France |	1615 |	9.36 |

#### 🔍Insight 
The United States leads in terms of repeat purchases, accounting for 36% of all return purchases—16 percentage points higher than Australia, the next closest territory at 20.34%. Canada follows in third place with 13.19%. A particularly valuable insight is that although Canada ranked fourth in the number of one-time buyers, it ranks third in return purchases, indicating a solid base of loyal customers despite a relatively high drop-off rate. This contrast suggests opportunities to further nurture and retain Canadian customers, who show strong potential for loyalty once engaged.

#### 🎯Why is this relevant ?
The intent of this query is to enable stakeholders have a clear picture of consumers reception and perception of their products by measure of frequency. By analyzing the percentage and total number of return purchases per country, the business can assess customer satisfaction, loyalty, and product-market fit in each territory. This is vital for informing regional retention strategies, allocating marketing resources effectively, and strengthening long-term customer relationships where the brand is gaining traction.


__________________
### 5. Segment Customers by their Purchase intervals and compare their spending habits or Purchase value
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

| Customers_Purchase_Interval	| Purchase_Count |	Average_Purchase_Value|
|-------------------------------| ---------------| ---------------------------|
| 9-12 Months |	919 |	1676.23 |
| 6-9 Months |	841 |	881.67 |
| 0-1 Month | 	18355 |	766.81 |
| 1 Year and Above |	21386 |	594.65 |
| 3-6 Months | 1289 |	376.86 |
| 1-3 Months |	1091 |	157.22 |

#### 🔍 Insight
Customers with purchase intervals of 9–12 months have the highest average order value at $1,676.23, followed by those purchasing every 6–9 months at $881.67. In contrast, customers purchasing most frequently—within 0–1 month and 1 year and above—record the highest purchase volumes at 18,355 and 21,386 respectively, but with lower average order values of $766.81 and $594.65

#### 🎯 Why This Is Relevant
This is meant to help stakeholders understand the relationship between purchase frequency and order value, revealing that less frequent buyers tend to spend more per transaction, while frequent buyers contribute more to overall sales volume. These patterns can inform segmentation strategies, such as nurturing high-value, infrequent buyers with personalized offers, while also reinforcing loyalty programs for frequent, lower-value buyers to sustain engagement and drive lifetime value.
