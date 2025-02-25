# Superstore Sales Data Analysis & RFM Segmentation

## Project Overview
This project involves creating a MySQL database, importing Superstore sales data, performing data cleaning, and conducting an exploratory data analysis (EDA). Additionally, it segments customers using RFM (Recency, Frequency, Monetary) analysis to categorize them based on purchasing behavior.

## Database and Table Creation
### 1. Create Database
```sql
CREATE DATABASE SUPERSTORE;
USE SUPERSTORE;
```
### 2. Create Table & Bulk Insert Data
- A table `SUPERSTORE_SALES` is created.
- Bulk data is inserted into the table.

## Data Cleaning and Formatting
### 3. Checking Data
```sql
SELECT * FROM SUPERSTORE_SALES LIMIT 10;
```
### 4. Renaming Columns for Convenience
```sql
ALTER TABLE SUPERSTORE_SALES  
CHANGE COLUMN `Row ID` RowID INT,
CHANGE COLUMN `Unit Price` UnitPrice DOUBLE,
CHANGE COLUMN `Shipping Cost` ShippingCost DOUBLE,
CHANGE COLUMN `Customer ID` CustomerID TEXT,
CHANGE COLUMN `Product Base Margin` ProductBaseMargin DOUBLE,
CHANGE COLUMN `Order Date` OrderDate TEXT,
CHANGE COLUMN `Ship Date` ShipDate TEXT,
CHANGE COLUMN `Order ID` OrderID TEXT;
```
### 5. Converting Excel Date Format
```sql
UPDATE SUPERSTORE_SALES
SET OrderDate = DATE_ADD('1899-12-30', INTERVAL OrderDate DAY);

UPDATE SUPERSTORE_SALES
SET ShipDate = DATE_ADD('1899-12-30', INTERVAL ShipDate DAY);
```
### 6. Formatting Dates
```sql
UPDATE SUPERSTORE_SALES
SET ORDERDATE = STR_TO_DATE(ORDERDATE, '%d/%m/%y');

UPDATE SUPERSTORE_SALES
SET SHIPDATE = STR_TO_DATE(SHIPDATE, '%d/%m/%y');
```

## Exploratory Data Analysis (EDA)
### 7. Checking Total Rows
```sql
SELECT COUNT(*) FROM SUPERSTORE_SALES;
```
### 8. Checking Last Order Date
```sql
SELECT MONTHNAME(MAX(ORDERDATE)) AS LAST_ORDER_MONTH, MAX(ORDERDATE) AS LAST_ORDER_DATE FROM SUPERSTORE_SALES;
```
### 9. Checking for Duplicate Customers
```sql
SELECT CUSTOMERID, COUNT(*) FROM SUPERSTORE_SALES GROUP BY CUSTOMERID ORDER BY 2 DESC;
```

## RFM Segmentation
### 10. Creating RFM Scores
```sql
CREATE OR REPLACE VIEW RFM_SCORE_DATA AS
WITH CUSTOMER_AGGREGATED_DATA AS
(SELECT
    CUSTOMERNAME,
    DATEDIFF((SELECT MAX(ORDERDATE) FROM SUPERSTORE_SALES), MAX(ORDERDATE)) AS RECENCY_VALUE,
    COUNT(DISTINCT ORDERID) AS FREQUENCY_VALUE,
    ROUND(SUM(SALES),0) AS MONETARY_VALUE
FROM SUPERSTORE_SALES
GROUP BY CUSTOMERNAME),

RFM_SCORE AS
(SELECT 
    C.*,
    NTILE(4) OVER (ORDER BY RECENCY_VALUE DESC) AS R_SCORE,
    NTILE(4) OVER (ORDER BY FREQUENCY_VALUE ASC) AS F_SCORE,
    NTILE(4) OVER (ORDER BY MONETARY_VALUE ASC) AS M_SCORE
FROM CUSTOMER_AGGREGATED_DATA AS C)

SELECT
    R.CUSTOMERNAME,
    R.RECENCY_VALUE,
    R_SCORE,
    R.FREQUENCY_VALUE,
    F_SCORE,
    R.MONETARY_VALUE,
    M_SCORE,
    (R_SCORE + F_SCORE + M_SCORE) AS TOTAL_RFM_SCORE,
    CONCAT_WS('', R_SCORE, F_SCORE, M_SCORE) AS RFM_SCORE_COMBINATION
FROM RFM_SCORE AS R;
```
### 11. Analyzing RFM Score Combinations
```sql
SELECT * FROM RFM_SCORE_DATA WHERE RFM_SCORE_COMBINATION = '111';
SELECT * FROM RFM_SCORE_DATA WHERE RFM_SCORE_COMBINATION = '222';
SELECT * FROM RFM_SCORE_DATA WHERE RFM_SCORE_COMBINATION = '333';
SELECT * FROM RFM_SCORE_DATA WHERE RFM_SCORE_COMBINATION = '444';
```
### 12. Creating Customer Segments
```sql
CREATE OR REPLACE VIEW RFM_ANALYSIS AS
SELECT 
    RFM_SCORE_DATA.*,
    CASE
        WHEN RFM_SCORE_COMBINATION IN (111, 112, 121, 132, 211, 211, 212, 114, 141) THEN 'CHURNED CUSTOMER'
        WHEN RFM_SCORE_COMBINATION IN (133, 134, 143, 224, 334, 343, 344, 144) THEN 'SLIPPING AWAY, CANNOT LOSE'
        WHEN RFM_SCORE_COMBINATION IN (311, 411, 331) THEN 'NEW CUSTOMERS'
        WHEN RFM_SCORE_COMBINATION IN (222, 231, 221,  223, 233, 322) THEN 'POTENTIAL CHURNERS'
        WHEN RFM_SCORE_COMBINATION IN (323, 333,321, 341, 422, 332, 432) THEN 'ACTIVE'
        WHEN RFM_SCORE_COMBINATION IN (433, 434, 443, 444) THEN 'LOYAL'
    ELSE 'Other'
    END AS CUSTOMER_SEGMENT
FROM RFM_SCORE_DATA;
```
### 13. Segment-wise Customer Analysis
```sql
SELECT
    CUSTOMER_SEGMENT,
    COUNT(*) AS NUMBER_OF_CUSTOMERS,
    ROUND(AVG(MONETARY_VALUE),0) AS AVERAGE_MONETARY_VALUE
FROM RFM_ANALYSIS
GROUP BY CUSTOMER_SEGMENT;
```

## Results & Insights
- Customers are segmented into different categories like Loyal, Active, Churned, etc.
- This segmentation helps in targeted marketing strategies and better customer relationship management.

## Technologies Used
- **Database**: MySQL
- **Data Analysis**: SQL Queries

## How to Use
1. Clone the repository.
2. Import the SQL scripts into MySQL.
3. Run the queries sequentially to clean, analyze, and segment the data.

## Future Improvements
- Automate RFM analysis using Python and visualization tools.
- Implement machine learning models for predictive analysis.
- Create a dashboard for dynamic data visualization.

## Author
Zulkarnain Saurav



