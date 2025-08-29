# 📊 RFM Segmentation Project

## 🧠 Project Overview  
This project demonstrates an end-to-end **RFM (Recency, Frequency, Monetary) Segmentation** workflow using **MySQL and Power BI**. It classifies customers into actionable segments to improve marketing, retention, and profitability.  

---

## ✅ Workflow  
1. Created MySQL database `RFM_SALES`  
2. Imported data into `SAMPLE_SALES_DATA`  
3. Calculated metrics: CLV, Frequency, Quantity Ordered, Recency  
4. Built RFM Segmentation (`RFM_SEGMENTATION_DATA` view)  
5. Aggregated results and visualized in Power BI  

---

## ⚙️ SQL Code (Core Parts)  

```sql
-- Create Database
CREATE DATABASE RFM_SALES;
USE RFM_SALES;

-- Customer-Level Metrics
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
GROUP BY CUSTOMERNAME;


| Customer Name  | CLV    | Frequency | Total Qty Ordered | Last Transaction Date | Recency (days) |
| -------------- | ------ | --------- | ----------------- | --------------------- | -------------- |
| Alpha Stores   | 15,000 | 12        | 240               | 2023-08-01            | 5              |
| Beta Traders   | 8,500  | 7         | 125               | 2023-07-25            | 12             |
| Gamma Supplies | 22,000 | 15        | 310               | 2023-07-30            | 7              |
