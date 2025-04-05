# [SQL] Sales Performance Analysis

## I. Introduction
This project utilizes Google BigQuery to evaluate the sales performance of a bicycle manufacturer. By employing advanced SQL techniques, including Common Table Expressions (CTEs), aggregate functions, and table functions, it processes essential metrics such as sales volume, number of orders, customer retention rate, and year-over-year/month-over-month growth rates. These insights reveal underlying trends and support data-driven strategies to enhance business performance and boost sales.

## II. Dataset
This is a dataset available on Google BigQuery named "adventureworks2019". To access the dataset, follow these steps:
- Log in to your Google Cloud Platform account and create a new project.
- Navigate to the BigQuery console and select your newly created project.
- In the navigation panel, select "Add Data" and then "Search a project".

## III. Analysis
### **Query 01: Calculate Quantity of items, Sales value & Order quantity by each Subcategory in the last 12 months**
```sql
WITH FilteredOrders AS (
    -- Lọc đơn hàng trong 12 tháng trước ngay từ đầu
    SELECT ProductID, SalesOrderID, OrderQty, LineTotal, ModifiedDate
    FROM adventureworks2019.Sales.SalesOrderDetail
    WHERE DATE(ModifiedDate) >= DATE_SUB((SELECT MAX(DATE(ModifiedDate)) FROM adventureworks2019.Sales.SalesOrderHeader), INTERVAL 12 MONTH)
)
 SELECT 
        FORMAT_TIMESTAMP('%b %Y', F.ModifiedDate) AS Period,
        S.Name AS Subcategory,  
        SUM(F.OrderQty) AS ItemQuantity,  
        SUM(F.LineTotal) AS SalesValue,  
        COUNT(DISTINCT F.SalesOrderID) AS OrderQuantity  
    FROM FilteredOrders F  
    INNER JOIN adventureworks2019.Production.Product P ON F.ProductID = P.ProductID  
    INNER JOIN adventureworks2019.Production.ProductSubcategory S  
        ON CAST(P.ProductSubcategoryID AS INT64) = S.ProductSubcategoryID
    GROUP BY Period, S.Name
    ORDER BY Period DESC, S.Name;
```
| Period   | Subcategory        | ItemQuantity | SalesValue    | OrderQuantity |
|----------|-------------------|-------------|--------------|---------------|
| Sep 2013 | Bike Racks        | 312         | 22,828.51    | 71            |
| Sep 2013 | Bike Stands       | 26          | 4,134.00     | 26            |
| Sep 2013 | Bottles and Cages | 803         | 4,676.56     | 380           |
| Sep 2013 | Bottom Brackets   | 60          | 3,118.14     | 19            |
| Sep 2013 | Brakes            | 100         | 6,390.00     | 29            |
| Sep 2013 | Caps              | 440         | 2,879.48     | 203           |
| Sep 2013 | Chains            | 62          | 752.93       | 24            |
| Sep 2013 | Cleaners          | 296         | 1,611.83     | 108           |
| Sep 2013 | Cranksets         | 75          | 13,955.85    | 20            |
| Sep 2013 | Derailleurs       | 97          | 5,972.06     | 23            |
| Sep 2013 | Fenders           | 169         | 3,714.62     | 169           |
| Sep 2013 | Gloves            | 1044        | 75,524.59    | 467           |
| Sep 2013 | Handlebars        | 131         | 6,486.58     | 38            |
| Sep 2013 | Helmets           | 1016        | 27,534.85    | 304           |
| Sep 2013 | Hydration Packs   | 249         | 9,186.71     | 155           |
| Sep 2013 | Jerseys           | 1402        | 48,318.10    | 304           |
| Sep 2013 | Mountain Bikes    | 1194        | 1,244,716.36 | 263           |
| Sep 2013 | Road Frames       | 264         | 110,277.03   | 76            |
| Sep 2013 | Tires and Tubes   | 1420        | 18,561.38    | 787           |
| Oct 2013 | Bike Racks        | 284         | 21,181.20    | 70            |
| Oct 2013 | Bike Stands       | 24          | 3,816.00     | 24            |
| Oct 2013 | Bottles and Cages | 845         | 5,038.95     | 400           |
| Oct 2013 | Bottom Brackets   | 132         | 7,030.00     | 30            |
| Oct 2013 | Brakes            | 142         | 9,036.78     | 38            |
| Oct 2013 | Caps              | 406         | 2,738.84     | 208           |
| Oct 2013 | Chains            | 93          | 1,132.97     | 31            |
| Oct 2013 | Cleaners          | 294         | 1,612.75     | 108           |
| Oct 2013 | Cranksets         | 118         | 19,722.79    | 33            |
| Oct 2013 | Derailleurs       | 135         | 8,291.80     | 35            |
| Oct 2013 | Fenders           | 170         | 3,736.96     | 170           |
| Oct 2013 | Gloves            | 447         | 76,781.83    | 148           |
| Oct 2013 | Handlebars        | 188         | 9,196.00     | 56            |
| Oct 2013 | Helmets           | 1127        | 30,733.99    | 557           |
| Oct 2013 | Hydration Packs   | 243         | 9,207.61     | 154           |
| Oct 2013 | Jerseys           | 1372        | 47,761.26    | 319           |
| Oct 2013 | Mountain Bikes    | 1231        | 1,338,378.67 | 243           |
| Oct 2013 | Pedals            | 374         | 14,060.98    | 72            |

**Mountain Bikes** generated the highest revenue at **$1.24M**, significantly outperforming other products due to their high unit price and strong demand. In contrast, **Tires and Tubes** had the highest order quantity at **787**, followed by **Gloves** at **467**, as these are **essential replacement parts** that require **frequent repurchasing**. While Mountain Bikes drive profitability with premium pricing, consumables like Tires, Tubes, and Gloves sustain high transaction volumes. To **optimize sales**, businesses should leverage **high-margin products** like **Mountain Bikes** for targeted promotions while ensuring **steady inventory** and **discounts for consumables** to maintain customer retention and recurring revenue.

### **Query 02: Calculate % YoY growth rate by SubCategory & release top 3 subcategory with highest grow rate. Can use metric: quantity_item. Round results to 2 decimal**
```sql
WITH SalesData AS (
      SELECT 
         CAST(FORMAT_TIMESTAMP('%Y', TIMESTAMP(D.ModifiedDate)) AS INT64) AS Year,
         S.Name AS SubCategory,
         SUM (D.OrderQty) AS item_quantity
      FROM adventureworks2019.Sales.SalesOrderDetail D
      INNER JOIN adventureworks2019.Production.Product P ON D.ProductID = P.ProductID  
      INNER JOIN adventureworks2019.Production.ProductSubcategory S ON CAST(P.ProductSubcategoryID AS INT64) = S.ProductSubcategoryID
      GROUP BY S.Name, Year
   ),

     YoYData AS (
      SELECT 
         thisyear.Year,
         thisyear.SubCategory,
         thisyear.item_quantity AS Qty_item,
         previous.item_quantity AS Prv_qty,
         ROUND(((thisyear.item_quantity - previous.item_quantity)/previous.item_quantity),2) AS YoY_Rate
      FROM SalesData AS thisyear
      JOIN SalesData AS previous 
         ON thisyear.SubCategory = previous.SubCategory
         AND thisyear.Year = previous.Year + 1
     )

SELECT
   SubCategory,
   Qty_item,
   Prv_qty,
   YoY_Rate
FROM YoYData
ORDER BY YoY_Rate DESC
LIMIT 3;
```
| Row | SubCategory       | Qty_item | Prv_qty | YoY_Rate |
|-----|------------------|----------|---------|----------|
| 1   | Mountain Frames  | 3168     | 510     | 5.21     |
| 2   | Socks           | 2724     | 523     | 4.21     |
| 3   | Road Frames     | 5564     | 1137    | 3.89     |

The **top 3 fastest-growing subcategories**—**Mountain Frames, Socks, and Road Frames** have all experienced significant YoY growth, with rates of **5.21, 4.21, and 3.89**, respectively. This suggests an overall surge in demand for both **cycling equipment and accessories**, possibly driven by **market trends, increased consumer interest, or effective promotional strategies**. The sharp increase in sales volume, particularly in frames, indicates a growing enthusiasm for cycling, while the rise in socks sales may reflect shifting consumer preferences or seasonal trends. Further analysis could examine external factors such as **pricing, marketing campaigns, or industry shifts** that contributed to this strong performance.  

### **Query 03: Ranking Top 3 TeritoryID with biggest Order quantity of every year. If there's TerritoryID with same quantity in a year, do not skip the rank number**
```sql
WITH SalesByYear AS (
  SELECT
    CAST(FORMAT_TIMESTAMP('%Y', TIMESTAMP(D.ModifiedDate)) AS INT64) AS Year,
    H.TerritoryID,
    SUM(D.OrderQty) AS TotalOrderQty
  FROM adventureworks2019.Sales.SalesOrderDetail D
  JOIN adventureworks2019.Sales.SalesOrderHeader H 
    ON D.SalesOrderID = H.SalesOrderID
  GROUP BY Year, H.TerritoryID
)

SELECT
  Year,
  TerritoryID,
  TotalOrderQty,
  DENSE_RANK() OVER (PARTITION BY Year ORDER BY TotalOrderQty DESC) AS TerritoryRank
FROM SalesByYear
QUALIFY TerritoryRank <= 3
ORDER BY Year DESC, TerritoryRank;
```
| Year | TerritoryID | TotalOrderQty | TerritoryRank |
|------|------------|--------------|--------------|
| 2014 | 4          | 11632        | 1            |
| 2014 | 6          | 9711         | 2            |
| 2014 | 1          | 8823         | 3            |
| 2013 | 4          | 26682        | 1            |
| 2013 | 6          | 22553        | 2            |
| 2013 | 1          | 17452        | 3            |
| 2012 | 4          | 17553        | 1            |
| 2012 | 6          | 14412        | 2            |
| 2012 | 1          | 8537         | 3            |
| 2011 | 4          | 3238         | 1            |
| 2011 | 6          | 2705         | 2            |
| 2011 | 1          | 1964         | 3            |

From **2011 to 2014**, **Territory 4 consistently led in total orders**, peaking in **2013 (26,682 orders) before a sharp decline in 2014**, while **Territory 6 and Territory 1 followed a similar trend**, indicating **a strong market boom in 2013 followed by a significant drop in 2014**, possibly due to **external economic factors, market saturation, or shifting customer demand**.

### **Query 04: Calculate Total Discount Cost belongs to Seasonal Discount for each SubCategory
```sql
SELECT
    CAST(FORMAT_TIMESTAMP('%Y', TIMESTAMP(D.ModifiedDate)) AS INT64) AS Year,
    S.Name,
    SUM(O.DiscountPct*D.UnitPrice*D.OrderQty) AS Total_Discount_Cost
FROM adventureworks2019.Sales.SalesOrderDetail D
INNER JOIN adventureworks2019.Sales.SpecialOffer O ON D.SpecialOfferID = O.SpecialOfferID 
INNER JOIN adventureworks2019.Production.Product P ON D.ProductID = P.ProductID  
INNER JOIN adventureworks2019.Production.ProductSubcategory S  
    ON CAST(P.ProductSubcategoryID AS INT64) = S.ProductSubcategoryID
WHERE O.Type = "Seasonal Discount"
GROUP BY S.Name, Year;
```
| Year | Name    | Total_Discount_Cost |
|------|--------|--------------------|
| 2012 | Helmets | 827.65             |
| 2013 | Helmets | 1606.04            |

Seasonal discounts were exclusively applied to the Helmets subcategory in 2012 and 2013. The **Total Discount Cost for Helmets nearly doubled from 2012 (827.65) to 2013 (1606.04)**, indicating a **higher seasonal discount allocation in 2013**, which could be driven by **increased promotions, higher sales volume, or more aggressive discount strategies** to boost demand.  

### **Query 05: Retention rate of Customer in 2014 with status of Successfully Shipped (Cohort Analysis)**
```sql
WITH allorder AS (
        SELECT 
             DISTINCT H.CustomerID,
             EXTRACT(MONTH FROM H.ModifiedDate) AS month
        FROM adventureworks2019.Sales.SalesOrderDetail  AS D
        JOIN adventureworks2019.Sales.SalesOrderHeader AS H
        USING(SalesOrderID)
        WHERE H.Status = 5
              AND EXTRACT(YEAR FROM H.ModifiedDate) = 2014
),
     firstorder AS (
      SELECT 
            DISTINCT CustomerID,
            MIN(month) AS month_join
      FROM allorder  
      GROUP BY CustomerID
)

SELECT 
    F.month_join, 
    CONCAT('M-', CAST((A.Month - F.month_join) AS STRING)) AS month_diff,
    COUNT(A.CustomerID) AS customer_cnt
FROM allorder A
JOIN firstorder F ON F.CustomerID = A.CustomerID
GROUP BY month_join, month_diff
ORDER BY month_join, month_diff;
```
| month_join | month_diff | customer_count |
|------------|------------|----------------|
| 1          | M-0        | 2076           |
| 1          | M-1        | 78             |
| 1          | M-2        | 89             |
| 1          | M-3        | 252            |
| 1          | M-4        | 96             |
| 1          | M-5        | 61             |
| 1          | M-6        | 18             |
| 2          | M-0        | 1805           |
| 2          | M-1        | 51             |
| 2          | M-2        | 61             |
| 2          | M-3        | 234            |
| 2          | M-4        | 58             |
| 2          | M-5        | 8              |
| 3          | M-0        | 1918           |
| 3          | M-1        | 43             |
| 3          | M-2        | 58             |
| 3          | M-3        | 44             |
| 3          | M-4        | 11             |
| 4          | M-0        | 1906           |
| 4          | M-1        | 34             |
| 4          | M-2        | 44             |
| 4          | M-3        | 7              |
| 5          | M-0        | 1947           |
| 5          | M-1        | 40             |
| 5          | M-2        | 7              |
| 6          | M-0        | 909            |
| 6          | M-1        | 10             |
| 7          | M-0        | 148            |

Customer retention in 2014 shows a **steep decline** after the first month. While the initial number of customers (M-0) is high, the retention drops significantly within the first three months. For example, in January, only 78 customers remained after the first month out of 2076, and in February, only 51 out of 1805. The trend continues across other cohorts, indicating a common challenge in retaining customers beyond their initial purchase. This suggests the need for **better customer engagement strategies, loyalty programs, and post-purchase incentives** to improve long-term retention.

### **Query 06: Trend of Stock level & MoM diff % by all product in 2011. If %gr rate is null then 0. Round to 1 decimal
```sql
WITH StockData AS (
     SELECT
        P.Name,
        SUM(W.StockedQty) AS Stock_Qty,
        EXTRACT(MONTH FROM W.ModifiedDate) AS Month,
        EXTRACT(YEAR FROM W.ModifiedDate) AS Year
     FROM adventureworks2019.Production.WorkOrder W
     JOIN adventureworks2019.Production. Product P ON W.ProductID = P.ProductID
     WHERE EXTRACT(YEAR FROM W.ModifiedDate) = 2011
     GROUP BY EXTRACT(YEAR FROM W.ModifiedDate), EXTRACT(MONTH FROM W.ModifiedDate), P.Name
)

SELECT
  curr.Name,
  curr.Month,
  curr.Year,
  curr.Stock_Qty AS curr_stock,
  prev.Stock_Qty AS prev_stock,
  IF ((curr.Stock_Qty-prev.Stock_Qty)/prev.Stock_Qty*100 IS NULL, 0, ROUND((curr.Stock_Qty-prev.Stock_Qty)/prev.Stock_Qty*100,1)) AS diff
FROM StockData AS curr
JOIN StockData AS prev ON curr.Name = prev.Name AND curr.Month = prev.Month + 1
ORDER BY curr.name, curr.Month DESC;
```
| Product Name       | Month | Year | Current Stock | Previous Stock | MoM Diff (%) |
|--------------------|-------|------|--------------|---------------|-------------|
| BB Ball Bearing   | 12    | 2011 | 8475         | 14544         | -41.7       |
| BB Ball Bearing   | 11    | 2011 | 14544        | 19175         | -24.2       |
| BB Ball Bearing   | 10    | 2011 | 19175        | 8845          | 116.8       |
| BB Ball Bearing   | 9     | 2011 | 8845         | 9666          | -8.5        |
| BB Ball Bearing   | 8     | 2011 | 9666         | 12837         | -24.7       |
| BB Ball Bearing   | 7     | 2011 | 12837        | 5259          | 144.1       |
| Blade             | 12    | 2011 | 1842         | 3598          | -48.8       |
| Blade             | 11    | 2011 | 3598         | 4670          | -23.0       |
| Blade             | 10    | 2011 | 4670         | 2122          | 120.1       |
| Blade             | 9     | 2011 | 2122         | 2382          | -10.9       |
| Blade             | 8     | 2011 | 2382         | 3166          | -24.8       |
| Blade             | 7     | 2011 | 3166         | 1280          | 147.3       |
| Chain Stays       | 12    | 2011 | 1842         | 3598          | -48.8       |
| Chain Stays       | 11    | 2011 | 3598         | 4670          | -23.0       |
| Chain Stays       | 10    | 2011 | 4670         | 2122          | 120.1       |
| Down Tube         | 12    | 2011 | 921          | 1799          | -48.8       |
| Down Tube         | 11    | 2011 | 1799         | 2335          | -23.0       |
| Down Tube         | 10    | 2011 | 2335         | 1061          | 120.1       |
| Fork Crown       | 12    | 2011 | 921          | 1799          | -48.8       |
| Fork Crown       | 11    | 2011 | 1799         | 2335          | -23.0       |
| Fork Crown       | 10    | 2011 | 2335         | 1061          | 120.1       |
| Front Derailleur | 12    | 2011 | 861          | 1440          | -40.2       |
| Front Derailleur | 11    | 2011 | 1440         | 1918          | -24.9       |
| Front Derailleur | 10    | 2011 | 1918         | 874           | 119.5       |

The stock levels for most products in 2011 show **high month-to-month fluctuations**. Significant **declines** and **spikes** indicate irregular demand or supply chain inconsistencies.
- **Drastic changes**: Several products, such as *BB Ball Bearing, Blade, and Fork Crown*, have extreme changes exceeding **100%** in some months.
- **Common downtrend**: Many products see sharp stock declines towards the end of the year, possibly due to **seasonal demand or inventory clearing**.
- **High volatility**: Products like *BB Ball Bearing (July: +144.1%, August: -24.7%)* and *Blade (July: +147.3%, August: -24.8%)* indicate unstable stock management.

A more **consistent inventory strategy** is needed to balance supply and demand, reducing large fluctuations.

### **Query 07: Calc Ratio of Stock / Sales in 2011 by product name, by month. Order results by month desc, ratio desc. Round Ratio to 1 decimal**
```sql
WITH SalesData AS (
      SELECT
         P.Name,
         D.ProductID,
         EXTRACT(MONTH FROM D.ModifiedDate) AS Month,
         EXTRACT(YEAR FROM D.ModifiedDate) AS Year,
         SUM(D.OrderQty) AS Sales
      FROM adventureworks2019.Sales.SalesOrderDetail D
      JOIN adventureworks2019.Production.Product P ON D.ProductID = P.ProductID
      WHERE EXTRACT(YEAR FROM D.ModifiedDate) = 2011
      GROUP BY EXTRACT(YEAR FROM D.ModifiedDate), EXTRACT(MONTH FROM D.ModifiedDate), P.Name, D.ProductID
),

     StockData AS (
      SELECT
         P.Name,
         W.ProductID,
         EXTRACT(MONTH FROM W.ModifiedDate) AS Month,
         EXTRACT(YEAR FROM W.ModifiedDate) AS Year,
         SUM(W.StockedQty) AS Stock
      FROM adventureworks2019.Production.WorkOrder W
      JOIN adventureworks2019.Production.Product P ON W.ProductID = P.ProductID
      WHERE EXTRACT(YEAR FROM W.ModifiedDate) = 2011
      GROUP BY EXTRACT(YEAR FROM W.ModifiedDate), EXTRACT(MONTH FROM W.ModifiedDate), P.Name, W.ProductID
)

SELECT
   S.Month,
   S.Year,
   S.ProductID,
   S.Name,
   S.Sales,
   T.Stock,
   ROUND(T.Stock/S.Sales,1) AS ratio
FROM SalesData S
JOIN StockData T ON S.ProductID = T.ProductID AND S.Month = T.Month
ORDER BY S.Month DESC, ratio DESC;
```

| Month | Year | ProductID | Name                   | Sales | Stock | Ratio |
|-------|------|-----------|------------------------|-------|-------|-------|
| 12    | 2011 | 745       | HL Mountain Frame      | 1     | 27    | 27.0  |
| 12    | 2011 | 743       | HL Mountain Frame      | 1     | 26    | 26.0  |
| 12    | 2011 | 748       | HL Mountain Frame      | 2     | 32    | 16.0  |
| 12    | 2011 | 722       | LL Road Frame          | 4     | 47    | 11.8  |
| 12    | 2011 | 741       | HL Mountain Frame      | 5     | 37    | 7.4   |
| 12    | 2011 | 727       | HL Mountain Frame      | 5     | 36    | 7.2   |
| 12    | 2011 | 738       | LL Road Frame          | 10    | 64    | 6.4   |
| 12    | 2011 | 730       | LL Road Frame          | 10    | 38    | 3.8   |
| 12    | 2011 | 729       | LL Road Frame          | 10    | 43    | 4.3   |
| 12    | 2011 | 732       | ML Road Frame          | 10    | 16    | 1.6   |
| 12    | 2011 | 751       | Road-150 Red           | 32    | 47    | 1.5   |
| 12    | 2011 | 750       | Road-150 Red           | 25    | 38    | 1.5   |
| 12    | 2011 | 775       | Mountain-100 Black     | 23    | 28    | 1.2   |
| 12    | 2011 | 773       | Mountain-100 Silver    | 32    | 36    | 1.1   |
| 12    | 2011 | 778       | Mountain-100 Black     | 25    | 27    | 1.1   |
| 12    | 2011 | 768       | Road-650 Black         | 19    | 21    | 1.1   |
| 12    | 2011 | 765       | Road-650 Black         | 39    | 43    | 1.1   |
| 12    | 2011 | 760       | Road-650 Red           | 31    | 33    | 1.1   |
| 12    | 2011 | 749       | Road-150 Red           | 45    | 51    | 1.1   |
| 12    | 2011 | 752       | Road-150 Red           | 32    | 35    | 1.1   |
| 12    | 2011 | 758       | Road-450 Red           | 37    | 37    | 1.0   |
| 12    | 2011 | 759       | Road-650 Red           | 12    | 12    | 1.0   |
| 12    | 2011 | 742       | HL Mountain Frame      | 3     | 3     | 1.0   |
| 12    | 2011 | 757       | Road-450 Red           | 6     | 6     | 1.0   |
| 12    | 2011 | 762       | Road-650 Red           | 41    | 40    | 1.0   |
| 12    | 2011 | 756       | Road-450 Red           | 23    | 23    | 1.0   |
| 12    | 2011 | 770       | Road-650 Black         | 52    | 53    | 1.0   |
| 12    | 2011 | 774       | Mountain-100 Silver    | 22    | 21    | 1.0   |
| 12    | 2011 | 755       | Road-450 Red           | 18    | 18    | 1.0   |
| 12    | 2011 | 761       | Road-650 Red           | 31    | 31    | 1.0   |
| 12    | 2011 | 777       | Mountain-100 Black     | 29    | 29    | 1.0   |
| 12    | 2011 | 764       | Road-650 Red           | 23    | 23    | 1.0   |
| 12    | 2011 | 754       | Road-450 Red           | 29    | 29    | 1.0   |
| 12    | 2011 | 767       | Road-650 Black         | 10    | 9     | 0.9   |
| 12    | 2011 | 753       | Road-150 Red           | 49    | 46    | 0.9   |
| 12    | 2011 | 769       | Road-650 Black         | 9     | 8     | 0.9   |
| 12    | 2011 | 766       | Road-650 Red           | 33    | 31    | 0.9   |
| 12    | 2011 | 772       | Mountain-100 Silver    | 18    | 17    | 0.9   |
| 12    | 2011 | 776       | Mountain-100 Black     | 24    | 22    | 0.9   |

The **HL Mountain Frame** product line showed the highest **Stock-to-Sales ratio**, with some models having a ratio as high as **27.0**. This suggests either overstocking or low sales demand, potentially indicating a need for **discounting strategies** or **marketing efforts** to increase sales.  

Meanwhile, products like the **Road-650 Black** and **Mountain-100 Black** maintained a **balanced stock-to-sales ratio** (around **1.0-1.1**), which may indicate **efficient inventory management** and consistent sales trends.  

Products with a ratio below **1.0**, such as **Mountain-100 Black (ratio: 0.9)**, might be in high demand and could risk **stockouts** if not replenished in time.

### **Query 08: No of order and value at Pending status in 2014
```sql
SELECT 
       EXTRACT( YEAR FROM ModifiedDate) AS Year,
       Status,
       COUNT(DISTINCT PurchaseOrderID) AS Order_cnt,
       SUM(TotalDue) AS Value
FROM adventureworks2019.Purchasing.PurchaseOrderHeader
WHERE EXTRACT( YEAR FROM ModifiedDate) = 2014
      AND Status = 1
GROUP BY year, Status;
```
| Year | Status | Order Count | Value            |
|------|--------|------------|------------------|
| 2014 | 1      | 224        | 3,873,579.01     |

In **2014**, there were **224 pending orders** with a total value of **$3,873,579.01**. This indicates a significant volume of transactions still awaiting processing. If these orders remain unfulfilled, they may lead to potential revenue loss or customer dissatisfaction. Monitoring the status and expediting the processing of these orders should be a priority to ensure smooth operations and cash flow management.

## IV. Conclusion
By utilizing Google BigQuery, this project conducted an in-depth analysis of a bicycle manufacturer’s dataset, uncovering valuable insights into sales performance, product demand, and operational effectiveness. The findings highlight the strong contribution of high-value bike sales, supported by steady accessory demand, as well as notable growth in specific product segments and consistently high-performing sales regions. Additionally, the analysis identified key areas for improvement, including inventory optimization, customer retention efforts, and order processing efficiency. Leveraging big data analytics, this study offers a data-driven framework for strategic decision-making, enabling the company to refine its product portfolio, enhance marketing initiatives, and streamline operations. These insights provide a pathway for sustainable growth and a competitive edge in the evolving bicycle industry.




