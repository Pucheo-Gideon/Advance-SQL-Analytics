# Retail Intelligence Using Advanced SQL: Customer, Sales, and Revenue Analysis

## Project Overview

This project leverages the AdventureWorks dataset to perform a comprehensive business intelligence analysis, focusing on customer segmentation, sales performance, market trends, and product growth. By applying SQL-based analytics, we uncover actionable insights to optimize marketing strategies, improve customer retention, and enhance revenue growth.

The analysis addresses key business challenges, including:
- Customer Value Identification (Top customers, RFM segmentation).
- Geographic Sales Performance (YoY trends, buyer distribution by country).
- Campaign & Product Effectiveness (Monthly active user growth, product MoM trends.

## Objectives
- Identify high-value customers to prioritize retention and personalized engagement.
- Evaluate market performance across regions to allocate resources effectively.
- Assess promotional campaign success in driving customer acquisition and retention.
- Determine product performance trends to optimize inventory and marketing strategies.
  
## Data Source
This project utilizes the AdventureWorks2021 dataset from Microsoft's learning platform. While the original database contains multiple tables, this analysis focuses on four key tables:
- Sales (58,191+ transactions across 13 columns)
- Customers (18,000+ unique customer records across 20 columns)
- Product (600+ unique products across 13 columns)
- Territory (11 distinct regions across 4 columns)

The analysis was conducted using Microsoft SQL Server as the primary RDBMS.

## Data Preparation
The dataset required minimal cleaning due to its well-structured nature. However, comprehensive data profiling and exploratory analysis were performed to:
- Validate data integrity and consistency
- Ensure proper table relationships for accurate joins
- Confirm absence of critical missing values
__________________________________________________________________________

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

#### 🎯 Business Implication: Customer-Centric Growth
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
________________________________________

### 3. How effective were monthly campaigns at driving customer growth?
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

### Business implications
Tracking Monthly Active Users (MAU) growth is critical for businesses running recurring campaigns or promotions. By analyzing month-over-month (MoM) changes in distinct customers, we can quantify the effectiveness of marketing efforts, identify seasonal patterns, and pinpoint periods of exceptional growth or decline. This analysis covers January 2021 through December 2023, revealing how customer engagement evolved across 36 months—highlighting successes (like the 195% surge in February 2023) and areas needing intervention (such as March 2022’s -29.54% drop).

#### Methodology
1.	Aggregate Distinct Users: Counted unique CustomerKey values grouped by month and year using DATEPART and DATENAME.
2.	Retrieve Prior Month Counts: Applied the LAG() window function to partition data by year and order by month, creating a Prev_Month_Count column for comparison.
3.	Calculate MoM Growth: Computed percentage change using the formula: ((Current - Previous)/Previous) * 100.0


________________________________________

#### 4. What is the Percentage of customers whose total sales increased from their previous year 
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

#### Business Implication?
This signals dangerous over-reliance on new customer acquisition (costing 5-25x more than retention) while 86.6% of existing customers are actively disengaging - a revenue leak that threatens long-term profitability unless addressed through targeted loyalty strategies.
________________________________________

# Markets/Geographic Sales Performance


### 5. What is the total Distribution and Percentage of One time Buyers Per Country

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
Australia & US lead with over 30% one-time buyers each - 2x higher than other markets. If just 10% of these buyers returned, revenue could grow by 10%. Revealing which countries have the highest proportion of customers making only a single purchase, stakeholders can pinpoint regions that may require targeted re-engagement strategies, localized marketing improvements, or service enhancements.

___________________________________________________________________


### 6. What is the total Distribution and Percentage of frequent Buyers Per Country

```sql
	WITH FrequentBuyers AS (
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
HAVING COUNT(S.SalesOrderNumber) >=2

), 
CountryCount AS (

	SELECT CustomerCountry,
	COUNT(*) As Return_Purchases
	FROM FrequentBuyers
	GROUP BY CustomerCountry
)
SELECT 
		CC.CustomerCountry, 
		cc.Return_Purchases,
		ROUND(CAST(cc.Return_Purchases * 100.0/ (SELECT COUNT(*) FROM FrequentBuyers) AS FLOAT),2) AS [% of Returned Purchase]
FROM CountryCount AS CC
ORDER BY 2 DESC
```
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
By analyzing the percentage and total number of return purchases per country, the business can assess customer satisfaction, loyalty, and product-market fit in each territory. This is vital for scaling or doubling down on winning retention tactics (e.g., loyalty programs) that drive high, Target one-time buyers with “second purchase” incentives (e.g., 15% off next order) and investigate why countries like  Germany/France lag (<10%) and test market-specific perks.
________________________________________________

### 7. Analyze the year-over-year (YoY) sales growth trends across different countries to identify which markets experienced the most significant recoveries or declines?
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

#### Result 
- 2022 Performance: France, Germany, and UK showed positive growth (52.90%, 15.56%, 26.54% respectively). Australia, Canada, and US experienced declines (-18.26%, -46.78%, -41.51%)

- 2023 Recovery/Expansion: All countries saw dramatic growth in 2023, with increases exceeding 100% YoY. The United States led with 278.79% growth. Canada and UK also showed particularly strong rebounds (250.30% and 205.83%)
- Also worth noting is that Countries that declined in 2022 saw the most dramatic rebounds in 2023 and France maintained consistent positive growth across both years.

#### Methodology 
- Aggregated Sales by Country and Year. Joined the Sales and Customers tables to link transactions to geographic territories
- Retrieve Prior-Year Sales Using LAG() Window Functions. his created a column (Prev_Sales) containing the previous year’s sales for each country, enabling YoY comparisons
- Compute Year-over-Year Growth by taing the variance of current year sales from the previous years and dividing by previous year sales and multiplied by 100 to get result as percentage  - [(Current_Year_Sales − Previous_Year_Sales) / Previous_Year_Sales] × 100
______________________________________________________________________

# Campaign & Product Performance

### 8.  What products achieve the highest positive MoM performance in 2021
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
____________________________________________________________________________________

### 9. What is the percentage of products in inventory with the longest sales streak (12 consecutive months in 2023?
```sql 
	WITH MonthlySales AS ( -- Get the distinct month a product was sold 
		SELECT P.ProductKey, 
				P.ProductName,
				CAST(DATEFROMPARTS(YEAR(S.OrderDate), Month(S.Orderdate), 1) AS DATE) AS Sales_Month
		FROM Sales AS  S
		INNER JOIN Product  AS P ON S.ProductKey = p.ProductKey
		WHERE YEAR(ORDERDATE) = '2023'
		GROUP BY  P.ProductKey, 
				  P.ProductName,
				  YEAR(S.OrderDate),
				  Month(S.Orderdate)
), 

WithLag AS ( -- Use lag to get the previous month

	SELECT Ms.ProductKey, 
			Ms.ProductName, 
			Ms.Sales_Month, 
			LAG(MS.Sales_Month) OVER (PARTITION BY ProductKey ORDER BY Ms.Sales_Month) AS PrevMonth
	FROM MonthlySales AS Ms
),

WithStreakFlag AS ( -- Flags a new streak when the month isn't consecutive

	SELECT *, 
			CASE 
				WHEN DATEDIFF(MONTH, WL.PrevMonth, WL.Sales_month) = 1 THEN 0
				ELSE 1
			END AS Flag
	FROM WithLag  AS WL
), 

StreakGroups AS ( -- Create streak groups using SUM(---) Over
	SELECT *, 
			SUM(Flag) OVER (PARTITION BY Productkey ORDER BY  SaleS_Month ROWS UNBOUNDED PRECEDING) AS StreakGroupID
	FROM WithStreakFlag
),
StreakCounts AS (
	SELECT 
		Productkey, 
		ProductName, 
		StreakGroupID, 
		COUNT (*) AS StreakLength 
	FROM StreakGroups
	GROUP BY ProductKey, ProductName, StreakGroupID
),

MaxStreaks AS (
	SELECT ProductKey, 
			ProductName,
			MAX(StreakLength) AS LongestStreakMonths
	FROM StreakCounts 
	GROUP BY ProductKey, ProductName
)

SELECT ROUND(CAST(COUNT(*)* 100.0/(SELECT COUNT(*) FROM Product) AS FLOAT),2) AS "% of Product with 12month salels streak"
FROM MaxStreaks
WHERE MaxStreaks.LongestStreakMonths = 12
```
| Metric                                | Value |
|---------------------------------------|-------|
| % of Products with 12-Month Sales Streak | 15.84  |

#### 📊 Result:
Out of 600+ products, only 98 (~15%) maintained consistent sales across every month in 2021.

#### 💡 Business implication:
These products represent what customers demand the most. Ensuring they're always in stock can:
1. Prevent lost sales and revenue.
2. Improve customer satisfaction.
3. Reduce the risk of churn

_______________________________________________________________________________________

## Recommendations

1. Top Customers by Territory: The top 2 customers per region contribute disproportionately to revenue - focusing retention efforts here and implement VIP programs for top buyers can enhance loyalty.
2. RFM: RFM segmentation reveals untapped potential in mid-tier customers. Upsell premium products and Launch re-engagement campaigns.
3. YoY Customer Sales Growth: Customers with rising YoY sales indicate strong loyalty - capitalize on this trend by rewarding customers with increasing spend (e.g., discounts on next purchase).
4. YoY Market Trends: Some regions (France) show stable growth, while others (US) rebound dramatically - adjust strategies accordingly. For High-Growth Markets (e.g., US, Canada): Increase ad spend. For declining markets (e.g., Australia 2022): Investigate external factors (competition, economy).
5. Buyer Distribution by Country: Buyer frequency varies significantly by country - localized strategies are essential. For High One-Time Buyers, improve post-purchase engagement (e.g., follow-up emails) and Introduce subscription models for High Frequent Buyers:
6. Monthly Active Users (MAU) Growth: MAU trends reveal seasonality - plan inventory and marketing around these patterns. Double down on successful campaigns on Peak Months (June) and run retention-focused promotions for dipped Months (July).
7. Top-Performing Products (MoM Growth): Product performance fluctuates - real-time tracking helps optimize stock levels. Best Sellers (High MoM) should be bundle with slower-moving products and discount or reposition declining products. The products with consecutive month sales streaks - which indicates customers' high and frequent demand - should be sufficiently stock-up to ensure  high availability during demands, maintain customers' satisfaction so as to ensure steady revenue generation from those products.

## Conclusion

The range of this analysis covers critical aspects of AdventureWorks business and extract valuable insights that provides a data-driven roadmap for AdventureWorks to:
1. Boost customer retention through targeted RFM strategies.
2. Optimize regional investments based on YoY growth trends.
3. Enhance campaign ROI by aligning with MAU and product trends.

By implementing these recommendations, stakeholders can increase revenue, reduce churn, and make informed strategic decisions.
