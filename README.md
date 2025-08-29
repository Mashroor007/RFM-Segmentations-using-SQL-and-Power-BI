# 📊 RFM Segmentation Project

## 🧠 Project Overview  
This project demonstrates a complete **RFM (Recency, Frequency, Monetary) Segmentation** workflow using **MySQL and Power BI**.  
The goal is to classify customers into meaningful segments and extract actionable insights for data-driven decision-making.  

---

## ✅ Workflow Steps  

- 📂 Created a dedicated MySQL database: `RFM_SALES`  
- 📥 Imported sample data into the `SAMPLE_SALES_DATA` table  
- 📊 Computed customer-level metrics: **CLV, Frequency, Quantity Ordered, Recency**  
- 🏷️ Built RFM Segmentation and created view: `RFM_SEGMENTATION_DATA`  
- 📈 Aggregated results to analyze segment-level metrics  

---

## 1️⃣ Create and Use Database  

```sql
CREATE DATABASE RFM_SALES;
USE RFM_SALES;
```
**Import Data**
## 1.Sample Data
```sql
SELECT * FROM sample_sales_data LIMIT 5;
```
## output
| OrderNo | QuantityOrdered | PriceEach | Sales   | OrderDate | Status  | Month | Year | ProductLine | CustomerName             | City          | Country | DealSize |
| ------- | --------------- | --------- | ------- | --------- | ------- | ----- | ---- | ----------- | ------------------------ | ------------- | ------- | -------- |
| 10107   | 30              | 95.7      | 2871    | 24/2/03   | Shipped | 2     | 2003 | Motorcycles | Land of Toys Inc.        | NYC           | USA     | Small    |
| 10121   | 34              | 81.35     | 2765.9  | 7/5/03    | Shipped | 5     | 2003 | Motorcycles | Reims Collectables       | Reims         | France  | Small    |
| 10134   | 41              | 94.74     | 3884.34 | 1/7/03    | Shipped | 7     | 2003 | Motorcycles | Lyon Souveniers          | Paris         | France  | Medium   |
| 10145   | 45              | 83.26     | 3746.7  | 25/8/03   | Shipped | 8     | 2003 | Motorcycles | Toys4GrownUps.com        | Pasadena      | USA     | Medium   |
| 10159   | 49              | 100       | 5205.27 | 10/10/03  | Shipped | 10    | 2003 | Motorcycles | Corporate Gift Ideas Co. | San Francisco | USA     | Medium   |

## 2.Customer-Level Metrics Query

```sql
SELECT 
    CUSTOMERNAME,
    ROUND(SUM(SALES),0) AS CLV,
    COUNT(DISTINCT ORDERNUMBER) AS FREQUENCY,
    SUM(QUANTITYORDERED) AS TOTAL_QTY_ORDERED,
    MAX(STR_TO_DATE(ORDERDATE,'%d/%m/%y')) AS LAST_TRANSACTION_DATE,
    DATEDIFF(
        (SELECT MAX(STR_TO_DATE(ORDERDATE, '%d/%m/%y')) FROM SAMPLE_SALES_DATA),
        MAX(STR_TO_DATE(ORDERDATE, '%d/%m/%y'))
    ) AS RECENCY
FROM SAMPLE_SALES_DATA
GROUP BY CUSTOMERNAME
LIMIT 5;
```

## Output:
| CustomerName                 | CLV     | Frequency | Total\_Qty\_Ordered | Last\_Transaction\_Date | Recency |
| ---------------------------- | ------- | --------- | ------------------- | ----------------------- | ------- |
| Alpha Cognac                 | 140,977 | 3         | 1374                | 2005-03-28              | 64      |
| Amica Models & Co.           | 188,235 | 2         | 1686                | 2004-09-09              | 264     |
| Anna's Decorations, Ltd      | 307,992 | 4         | 2938                | 2005-03-09              | 83      |
| Atelier graphique            | 48,360  | 3         | 540                 | 2004-11-25              | 187     |
| Australian Collectables, Ltd | 129,183 | 3         | 1410                | 2005-05-09              | 22      |

## 3.Creating View For Segmentation 
```sql
CREATE VIEW RFM_SEGMENTATION_DATA AS
WITH CLV AS 
(
    SELECT
        CUSTOMERNAME,
        MAX(STR_TO_DATE(ORDERDATE, '%d/%m/%y')) AS CUSTOMER_LAST_TRANSACTION_DATE,
        DATEDIFF(
            (SELECT MAX(STR_TO_DATE(ORDERDATE, '%d/%m/%y')) FROM SAMPLE_SALES_DATA),
            MAX(STR_TO_DATE(ORDERDATE, '%d/%m/%y'))
        ) AS RECENCY_VALUE,
        COUNT(DISTINCT ORDERNUMBER) AS FREQUENCY_VALUE,
        SUM(QUANTITYORDERED) AS TOTAL_QTY_ORDERED,
        ROUND(SUM(SALES),0) AS MONETARY_VALUE
    FROM SAMPLE_SALES_DATA
    GROUP BY CUSTOMERNAME
),

RFM_SCORE AS
(
    SELECT 
        C.*,
        NTILE(5) OVER(ORDER BY RECENCY_VALUE DESC) AS R_SCORE,
        NTILE(5) OVER(ORDER BY FREQUENCY_VALUE ASC) AS F_SCORE,
        NTILE(5) OVER(ORDER BY MONETARY_VALUE ASC) AS M_SCORE
    FROM CLV AS C
),

RFM_COMBINATION AS
(
    SELECT
        R.*,
        R_SCORE + F_SCORE + M_SCORE AS TOTAL_RFM_SCORE,
        CONCAT_WS('', R_SCORE, F_SCORE, M_SCORE) AS RFM_COMBINATION
    FROM RFM_SCORE AS R
)

SELECT
    RC.*,
    CASE
        WHEN RFM_COMBINATION IN (455, 515, 542, 544, 552, 553, 452, 545, 554, 555) 
            THEN "Champions"
        WHEN RFM_COMBINATION IN (344, 345, 353, 354, 355, 443, 451, 342, 351, 352, 441, 442, 444, 445, 453, 454, 541, 543, 515, 551) 
            THEN 'Loyal Customers'
        WHEN RFM_COMBINATION IN (513, 413, 511, 411, 512, 341, 412, 343, 514) 
            THEN 'Potential Loyalists'
        WHEN RFM_COMBINATION IN (414, 415, 214, 211, 212, 213, 241, 251, 312, 314, 311, 313, 315, 243, 245, 252, 253, 255, 242, 244, 254) 
            THEN 'Promising Customers'
        WHEN RFM_COMBINATION IN (141, 142, 143, 144, 151, 152, 155, 145, 153, 154, 215) 
            THEN 'Needs Attention'
        WHEN RFM_COMBINATION IN (113, 111, 112, 114, 115) 
            THEN 'About to Sleep'
        ELSE "Other"
    END AS CUSTOMER_SEGMENT
FROM RFM_COMBINATION RC;
```
## 4.Final Segmentation
```sql
SELECT * FROM RFM_SEGMENTATION_DATA LIMIT 5;
```

| CustomerName            | Recency | Frequency | MonetaryValue | R\_Score | F\_Score | M\_Score | Segment   |
| ----------------------- | ------- | --------- | ------------- | -------- | -------- | -------- | --------- |
| Boards & Toys Co.       | 112     | 2         | 18,259        | 4        | 2        | 1        | Champions |
| Atelier graphique       | 187     | 3         | 48,360        | 3        | 3        | 1        | Champions |
| Auto-Moto Classics Inc. | 179     | 3         | 52,959        | 3        | 3        | 1        | Champions |
| Microscale Inc.         | 209     | 2         | 66,290        | 2        | 1        | 1        | Champions |
| Royale Belge            | 141     | 4         | 66,880        | 4        | 5        | 1        | Champions |

## 5.Final RFM

```sql

SELECT
    CUSTOMER_SEGMENT,
    SUM(MONETARY_VALUE) AS TOTAL_SPENDING,
    ROUND(AVG(MONETARY_VALUE),0) AS AVG_SPENDING,
    SUM(FREQUENCY_VALUE) AS TOTAL_ORDERS,
    SUM(TOTAL_QTY_ORDERED) AS TOTAL_QTY
FROM RFM_SEGMENTATION_DATA
GROUP BY CUSTOMER_SEGMENT;
```

## Output:

| Customer Segment    | Total Spending | Avg Spending | Total Orders | Total Qty Ordered |
| ------------------- | -------------- | ------------ | ------------ | ----------------- |
| Champions           | 367,337        | 61,223       | 17           | 3,968             |
| About to Sleep      | 716,736        | 89,592       | 15           | 7,360             |
| Loyal Customers     | 577,609        | 115,522      | 13           | 5,722             |
| Potential Loyalists | 3,139,858      | 149,517      | 54           | 31,252            |
| Promising Customers | 3,329,075      | 184,949      | 50           | 32,178            |
| Needs Attention     | 3,473,776      | 231,585      | 47           | 33,536            |
| At Risk             | 2,312,250      | 289,031      | 31           | 22,820            |
| Lost/Inactive       | 6,148,617      | 558,965      | 80           | 61,298            |

## 🔎 Key Insights
- **Champions & Loyal Customers** → Most engaged and profitable customers.  
- **Needs Attention & About to Sleep** → Customers at risk of churn, require re-engagement.  
- **Lost/Inactive Customers** → Historically high spenders but need reactivation efforts.  

---

## 🎯 Why RFM Segmentation?
RFM (Recency, Frequency, Monetary) analysis helps businesses to:  
- 🎯 **Target marketing campaigns** more effectively  
- 🔄 **Improve customer retention** by identifying loyalty drivers  
- 💰 **Maximize profitability** by focusing on high-value customers  

---

## 📌 Tools & Technologies
- 🗄️ **Database:** MySQL  
- 📊 **Visualization:** Power BI
-  🐍 **Python:** Bulk data insertion, preprocessing, and automation

---





