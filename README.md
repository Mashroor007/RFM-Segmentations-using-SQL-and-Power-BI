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


