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
 

