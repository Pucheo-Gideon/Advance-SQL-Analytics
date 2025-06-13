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

#### 💡 Strategic Insights
- Close Competition: Marginal gaps between #1 and #2 in most markets (e.g., France: $1.11 difference).
- Revenue Powerhouses: Top customers drive 12-18% of territory sales (est.).
- Global Consistency: All territories have clear high-value customer leaders.

#### 🎯 Business Impact: Customer-Centric Growth
- Retention Priority: These customers are low-hanging fruit for revenue protection.
- Upsell Potential: France/Germany’s top spenders ($11K+) may respond to premium offers.
- Territory Strategy: Replicate success factors from top performers (e.g., Nichole Nara’s preferences).
____________________________________________________________________

### 2.Analyze customer behavior using Recency,Frequency, Monetary value modeling and report the RFM scores for all customers
```sql
--- Declare Variable for your benchmark Date
DECLARE @Today AS Date  = '2024-01-01';

WITH Base AS (
	SELECT	
			--- The Recency Test
			CustomerKey, 
			CAST(MAX(OrderDate) AS DATE) AS Most_recent_purchase_date, 
			-- Get the recency score
			DATEDIFF(DAY, CAST(MAX(OrderDate) AS DATE),  @Today) AS Recency_Score, 
			--Get the Frequency Score
			COUNT(SalesOrderNumber) AS Frequency_score,
			-- Get The Monetary_Value 
			SUM(SalesAmount) AS Monetary_Value
	FROM Sales
GROUP BY CustomerKey
),

RFM_Scores AS (
	SELECT 
		CustomerKey, 
		Recency_Score,
		NTILE(5) OVER (ORDER BY Recency_Score DESC) AS R,
		Frequency_Score,
		NTILE(5) OVER (ORDER BY Frequency_Score) AS F, 
		Monetary_Value,
		NTILE(5) OVER (ORDER BY Monetary_Value) AS M
	FROM Base
WHERE Monetary_Value IS NOT NULL
)

SELECT 
		(R  + F + M)/ 3 AS RFM_Grouping,
		COUNT(RFM_Scores.customerKey) as Total_Customers,
		CAST(SUM(Monetary_Value) AS DECIMAL(16,2)) AS Total_Revenue, 
		CAST (AVG(Monetary_Value) AS DECIMAL(16,2)) AS Avg_Revenue
FROM RFM_Scores
GROUP BY 	(R  + F + M)/ 3 
ORDER BY 1 DESC

```

| RFM_Grouping	| Total_Customers	| Total_Revenue	| Avg_Revenue |
| --------------| ----------------------| ------------- | ------------|
| 5 |	460 | 	2733982.11 | 	5943.44 |
| 4 |	3769 |	12600402.49 |	3343.17 |
| 3 |	5570 |	9793167.54 |	1758.20 |
| 2 |	5671 |	3889292.59 |	685.82 |
| 1 |	2405 |	169605.00 |	70.52 |

### Results 
Data shows that newest customers (RFM Group 5) — just 427 people — delivered:
💰 $5,993 average spend (88% higher than all other segments combined!)
🚀 $2.56M total revenue (outpacing groups 2x their size)

## Business Implication?
 This result from the data establishes the fact that the business's freshest leads are it's HIGHEST-VALUE customers. Losing them equates to losing 20%+ of revenue base overnight and their spending  behavior holds clues to replicating success

### 3. Fetch the customers who made just one purchase
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
### 4. What is the total Distribution and Percentage of One time Buyers Per Country

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

#### Results 🔍 
Australia has the highest share of one-time buyers at 32.17%, closely followed by the United States at 31.7%. This suggests that these two markets may have retention challenges, with a significant portion of customers not returning after their first purchase.

#### 🎯Business implication?
The purpose of this query is to help stakeholders identify the distribution of one-time customers across different countries, providing insight into where customer retention may be weakest. By revealing which countries have the highest proportion of customers making only a single purchase, stakeholders can pinpoint regions that may require targeted re-engagement strategies, localized marketing improvements, or service enhancements.

__________________

### 5. What is the total Distribution and Percentage of frequent Buyers Per Country


| CustomerCountry |	Return_Purchases |	% of Single Purchase |
| --------------- | ---------------------| -------------------------- |
| United States |	6213 | 	36 |
| Australia	| 3510	 | 20.34 |
| Canada |	2276 |	13.19 |
| United Kingdom |	2025 |	11.73 |
| Germany |	1619 |	9.38 |
| France |	1615 |	9.36 |

#### 🔍Result
The United States leads in terms of repeat purchases, accounting for 36% of all return purchases—16 percentage points higher than Australia, the next closest territory at 20.34%. Canada follows in third place with 13.19%. A particularly valuable insight is that although Canada ranked fourth in the number of one-time buyers, it ranks third in return purchases, indicating a solid base of loyal customers despite a relatively high drop-off rate. This contrast suggests opportunities to further nurture and retain Canadian customers, who show strong potential for loyalty once engaged.

#### 🎯Business implication ?
The intent of this query is to enable stakeholders have a clear picture of consumers reception and perception of their products by measure of frequency. By analyzing the percentage and total number of return purchases per country, the business can assess customer satisfaction, loyalty, and product-market fit in each territory. This is vital for informing regional retention strategies, allocating marketing resources effectively, and strengthening long-term customer relationships where the brand is gaining traction.

__________________

### 6.Analyze the year-over-year (YoY) sales growth trends across different countries to identify which markets experienced the most significant recoveries or declines?
```sql
	-- Step One: Calculate the total sales and group by territory(Country) and Year
WITH Territory_Sales AS (
SELECT  C.CustomerCountry, 
		YEAR(S.OrderDate) AS Year, 
		ROUND(CAST(SUM(S.SalesAmount) AS FLOAT), 2) Total_Sales
FROM Sales AS S
INNER JOIN [Customers ] AS C
ON S.CustomerKey = C.CustomerKey
GROUP BY C.CustomerCountry, 
		YEAR(S.OrderDate)

),

--- Step Two: Using the LAG Function, partititoned by each country and ordered by Year, fetch the previous Year Sales for each country
Prev_Year_Sales AS (
SELECT  *,
	LAG(TS.Total_Sales) OVER(PARTITION BY TS.CustomerCountry ORDER BY TS.Year) AS Prev_Sales 
FROM Territory_Sales AS TS
) ,

-- Step Three: Calculate the Year-over-Year Growth by subtracting Total Current Year Sales from Prev Year Sales and Dividing by Previous Years 
YoY AS (
SELECT *, 
		Py.Total_Sales - Py.Prev_Sales AS Variance,
		CAST(((Py.Total_Sales - Py.Prev_Sales)/Py.Prev_Sales) * 100.0  AS DECIMAL(16,2)) AS "YoY % ▼▲"
FROM Prev_Year_Sales AS Py 
)

-- Step Four: Get the Countries, Year and YoY
SELECT YoY.CustomerCountry,
		YoY.Year,
		YoY."YoY % ▼▲"
FROM YoY
```
#### Methodology 
- Aggregated Sales by Country and Year. Joined the Sales and Customers tables to link transactions to geographic territories
- Retrieve Prior-Year Sales Using LAG() Window Functions. his created a column (Prev_Sales) containing the previous year’s sales for each country, enabling YoY comparisons
- Compute Year-over-Year Growth by taing the variance of current year sales from the previous years and dividing by previous year sales and multiplied by 100 to get result as percentage  - [(Current_Year_Sales − Previous_Year_Sales) / Previous_Year_Sales] × 100

#### Result 
- 2022 Performance: France, Germany, and UK showed positive growth (52.90%, 15.56%, 26.54% respectively). Australia, Canada, and US experienced declines (-18.26%, -46.78%, -41.51%)
  
- 2023 Recovery/Expansion: All countries saw dramatic growth in 2023, with increases exceeding 100% YoY. The United States led with 278.79% growth. Canada and UK also showed particularly strong rebounds (250.30% and 205.83%)
  
- Also worth noting is that Countries that declined in 2022 saw the most dramatic rebounds in 2023 and France maintained consistent positive growth across both years.

|CustomerCountry | 	Year	| YoY % ▼▲ |
| --------------- | ----------- | -------- |
| Australia |	2021 |	NULL |
| Australia |	2022 |	-18.26 |
| Australia	| 2023 |	107.66 |
| Canada |	2021 |	NULL |
| Canada |	2022 |	-46.78 |
| Canada |	2023 |	250.30 |
| France |	2021 |	NULL |
| France |	2022 |	52.90 |
| France |	2023 |	150.20 |
| Germany |	2021 |	NULL |
| Germany |	2022 |	15.56 |
| Germany |	2023 |	199.32 |
| United Kingdom |	2021 |	NULL |
| United Kingdom |	2022 |	26.54 |
| United Kingdom |	2023 |	205.83 |
| United States |	2021 | 	NULL |
| United States |	2022 |	-41.51 |
| United States	| 2023 |	278.79 |


__________________
### 6. Segment Customers by their Purchase intervals and compare their spending habits or Purchase value
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

________________________________

#### 7. What is the Percentage of customers whose total sales increased from their previous year 
``` sql

WITH Customer_Value AS (
SELECT -- Step One: Create a CTE and aggregate each Customer total purchase value, grouped by Customer Name, Key and year 
		C.CustomerKey,
		c.FullName,
		YEAR(S.OrderDate) AS Year,
		SUM(s.SalesAmount) As Sales_Value
FROM [Customers ] AS C
INNER JOIN Sales AS S 
ON C.CustomerKey = S.CustomerKey
GROUP BY  C.FullName,
		 YEAR(S.OrderDate),
		 C.CustomerKey
),

Yearly_Value AS (
SELECT	-- Step Two: Create a second CTE. Using SUM Window Function, Partition aggregated sales value per customer by year and customer key 
		Cv.CustomerKey,
		Cv.FullName, 
		CV.Year,
		ROUND(CAST(SUM(Cv.Sales_value) OVER (PARTITION BY   CV.Year, Cv.CustomerKey ORDER BY CV.Year, Cv.Sales_value) AS FLOAT),2) AS Customer_Yearly_Value,
		DENSE_RANK() OVER (PARTITION BY  Cv.CustomerKey ORDER BY CV.Year,  Cv.Sales_Value) AS Rank
FROM Customer_Value AS Cv
) ,

Current_Year_vs_Previous AS (
SELECT *,			-- Step Three: Create a third CTE. Using LAG function, fetch each customers previous year total purchase value. Using LAST_VALUE function 
					-- within each customer partition,grab the last total purchase value within that window, representing customer's latest year total purchase value
		LAG(Yv.Customer_Yearly_Value) OVER (PARTITION BY Yv.CustomerKey ORDER BY   Yv.CustomerKey) AS Prev_Year_Value,
		LAST_VALUE(Yv.Customer_Yearly_Value) OVER (PARTITION BY Yv.CustomerKey ORDER BY   Yv.CustomerKey) AS Current_Year_Value
FROM Yearly_Value  AS Yv
),

--- Step Four: Fetch from the third chained CTE the Customers'Name, Previous and Latest Year Sales Value. Using the WHERE Clause filter and return only the Customers whose 
--- Current Year total purchase value is greater than the previous total Purchase value
Current_Greater_Than_Previous AS (
SELECT  Cp.FullName, 
		Cp.Prev_Year_value, 
		Cp.Current_Year_Value
FROM Current_Year_vs_Previous AS Cp
WHERE Cp.Prev_Year_value IS NOT NULL
	  AND Cp.Current_Year_Value > Cp.Prev_Year_value
	  )

SELECT COUNT(*) AS High_YoY_Customers,
		ROUND(CAST(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM [Customers ]) AS FLOAT),2) AS Prct_of_High_YoY_Customers
		--(SELECT COUNT(*) FROM [Customers ]) As Total_Customers
FROM Current_Greater_Than_Previous 
```

##### Result 
| High_YoY_Customers |	Prct_of_High_YoY_Customers |
| ------------------- | ----------------------------|
| 2478	| 13.4 |

#### Insight 🔍
Out of 18,000+ customers, only 2,478 (13.4%) showed increased year-over-year spending - meaning 9 out of 10 customers spent less than the previous period or don't even returned for a purchase.

#### Why is this critical?
This signals dangerous over-reliance on new customer acquisition (costing 5-25x more than retention) while 86.6% of existing customers are actively disengaging - a revenue leak that threatens long-term profitability unless addressed through targeted loyalty strategies.

### 8. How effective were monthly campaigns at driving customer growth?
```sql
WITH Distinct_Customers AS  (

	SELECT	
		DATEPART(YEAR, OrderDate) AS Year, 
		DATEPART(MONTH, OrderDate) AS Month_Number,
		DATENAME(MONTH, OrderDate) AS Month_Name,
		COUNT(DISTINCT Customerkey) AS Distinct_Customers
	FROM Sales 
	GROUP BY DATEPART(YEAR, OrderDate),
		 DATEPART(MONTH, OrderDate), 
		 DATENAME(MONTH, OrderDate) 
	

),

--Step Two: In a CTE and using the LAG() function, get the previous month distribution of distinct customers. 
Prev_Month_users AS (
SELECT *, 
	LAG( DC.Distinct_Customers) OVER( PARTITION BY DC.Year ORDER BY DC.Month_Number, DC.Year)  AS Prev_Month_Count
FROM Distinct_Customers AS DC
)

-- Get the MoM % ▲▼ using the formula: (Current - Previous)/Previous
SELECT
      PM.Year, 
      PM.Month_Name,
      PM.Distinct_Customers,
      ROUND((CAST((PM.Distinct_Customers - PM.Prev_Month_Count) AS FLOAT)/PM.Prev_Month_Count) * 100.0,2) AS "MoM % ▲▼"
FROM Prev_Month_users AS PM
```
### 🔍 Key Insights
#### 🚀 Explosive Growth in 2023
- February 2023 saw a 195.3% MoM increase in MAU – a massive leap!
- Sustained momentum: June 2023 (+24.66%), November 2023 (+8.28%).
#### 📈📉 Seasonality & Volatility
- Peaks: June was consistently strong:
- 2021: +39.05% | 2022: +70.98% (🎯 Prime campaign months?)
- Troughs: Mid-year slumps:
- July 2021: -20% | July 2023: -12.76% (🌧️ Seasonal dip?)
#### 🛠️ Stabilization Trend
- 2023 had fewer wild swings vs. 2021–2022, suggesting:
- Better user retention 📊
- Optimized marketing spend 💰

### Why this is Relevant 
Tracking Monthly Active Users (MAU) growth is critical for businesses running recurring campaigns or promotions. By analyzing month-over-month (MoM) changes in distinct customers, we can quantify the effectiveness of marketing efforts, identify seasonal patterns, and pinpoint periods of exceptional growth or decline. This analysis covers January 2021 through December 2023, revealing how customer engagement evolved across 36 months—highlighting successes (like the 195% surge in February 2023) and areas needing intervention (such as March 2022’s -29.54% drop).

#### Methodology
1.	Aggregate Distinct Users: Counted unique CustomerKey values grouped by month and year using DATEPART and DATENAME.
2.	Retrieve Prior Month Counts: Applied the LAG() window function to partition data by year and order by month, creating a Prev_Month_Count column for comparison.
3.	Calculate MoM Growth: Computed percentage change using the formula: ((Current - Previous)/Previous) * 100.0

### 9.  What products achieve the highest positive MoM performance in 2021
```sql 
	
 WITH Territory_Sales AS ( -- Join Sales table to the territory to retrieve Territory name. Aggregate sales for each territry and group by Country and Months
 SELECT 
		T.Country, 
		MONTH(S.OrderDate) AS Month_Number,
		DATENAME(MONTH, S.OrderDate) AS Month_Name,
		ROUND(CAST(SUM(S.SalesAmount) AS FLOAT),2) AS Sales
 FROM Sales AS S 
 JOIN Territory AS T 
 ON T.SalesTerritoryKey = S.SalesTerritoryKey
 WHERE YEAR(S.OrderDate) = '2021'
 GROUP	BY T.Country, 
		MONTH(S.OrderDate),
		DATENAME(MONTH, S.OrderDate)
),

Month_over_Month AS ( -- Get the previous month using the Lag Function. Get the Difference between current and previous month and divide by Previous Month to see the Month-over-Month in revenue performance 
 SELECT Ts.Country,
		Ts.Month_number,
		Ts.Month_Name,
		Ts.Sales,
		LAG(Ts.Sales) OVER ( PARTITION BY  Ts.Country ORDER BY MONTH(TS.Month_Number) )AS Prev_Sales,
		ROUND((Ts.Sales - LAG(Ts.Sales) OVER( PARTITION BY  Ts.Country ORDER BY MONTH(TS.Month_Number)))/ LAG(Ts.Sales) OVER ( PARTITION BY  Ts.Country ORDER BY MONTH(TS.Month_Number)) * 100.0,2) AS "MoM % ▲▼"
 FROM Territory_Sales AS Ts
 )
 SELECT MoM.Country,
		COUNT(*) AS "No_of_+ve_MoM"
 FROM Month_over_Month  AS MoM
 WHERE MoM."MoM % ▲▼" > 0
 GROUP BY MoM.Country
 ORDER BY 2 DESC
```

#### Result 
| ProductName |	No of +ve MoM |
|------------|----------------|
| Road-150 Red, 52 | 	7 |
| Road-150 Red, 56 | 	7 |
| Road-150 Red, 62 |	6 |
| Mountain-100 Black 38 |	6 |
| Mountain-100 Black, 42 |	6 |
| Road-650 Red, 48 |	6 |
| Mountain-100 Black, 44 |	5 |
| Mountain-100 Black, 48 |	5 |
| Mountain-100 Silver, 42 |	5 |
| Road-150 Red, 44 |	5 |
| Road-150 Red, 48 |	5 |
| Road-650 Black, 60 |	5 |


#### 🚀 MoM Sales Performance: Key Insights
📈 Top Performers (+ve MoM)
🔥 Road Bikes Dominate: Road-150 Red (52, 56, 62) & Road-650 Red (48) lead with 6-7 months of growth in 2021. Mountain-100 Black (38, 42) also show strong consistency (6 +ve MoM).

#### 💡 Business Implication:
These products are the company's cash cows and scaling marketing & inventory for these models should sustain the positive Month-over-month streak for the subsequent fiscal year!

#### 🔎 Methodology: Solving MoM Sales Variance
- Data Preparation: oined the Sales table to Products to retrieve product names. Aggregated sales by product and month.

- Month-over-Month Calculation: Used the LAG() function to compare each month’s revenue to the previous month. Calculated the percentage change:
(Current Month − Previous Month) / Previous Month.

- Filtered for positive MoM changes to identify growth trends. Counted occurrences per product to find consistent top performers. Repeated for negative MoM to flag declining products.
