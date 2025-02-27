# Superstore Sales Data Analysis & RFM Segmentation

## Project Overview
This project involves creating a MySQL database, importing Superstore sales data, performing data cleaning, and conducting an exploratory data analysis (EDA). Additionally, it segments customers using RFM (Recency, Frequency, Monetary) analysis to categorize them based on purchasing behavior.

## Files Used
- **Superstore Sales Data.xlsx** - Raw sales data.
- **superstore_sales.sql** - SQL script for database creation, data insertion and analysis.

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
SELECT COUNT(*) AS TOTAL_INSTANCES FROM SUPERSTORE_SALES;
```
|TOTAL_INSTANCES  |
|-----------------|
|9426             |

### 8. Checking Last Order Date
```sql
SELECT MONTHNAME(MAX(ORDERDATE)) AS LAST_ORDER_MONTH, MAX(ORDERDATE) AS LAST_ORDER_DATE FROM SUPERSTORE_SALES;
```
| LAST_ORDER_MONTH | LAST_ORDER_DATE |
|------------------|-----------------|
|December          |2013-12-31       |

### 9. Checking for Duplicate Rows
```sql
SELECT ROWID, COUNT(*) AS ROW_COUNT FROM superstore.SUPERSTORE_SALES GROUP BY ROWID ORDER BY 2 DESC LIMIT 5;
```
| ROWID | ROW_COUNT |
|-------|-------|
| 18606 | 1     |
| 20847 | 1     |
| 23086 | 1     |
| 23087 | 1     |
| 23088 | 1     |

### 10. Checking for Invalid Order Quantity
```sql
SELECT ROWID, QUANTITYORDERED FROM SUPERSTORE_SALES ORDER BY QUANTITYORDERED LIMIT 5;
```
| ROWID  | QUANTITYORDERED |
|--------|-------|
| 25323  | 1     |
| 21253  | 1     |
| 20631  | 1     |
| 24987  | 1     |
| 21254  | 1     |

### 11. Checking for Invalid Unit Price
```sql
SELECT ROWID, ROUND(UNITPRICE, 2) AS UNITPRICE FROM SUPERSTORE_SALES ORDER BY UNITPRICE LIMIT 5;
```
| ROWID  | UNITPRICE |
|--------|-----------|
| 26347  | 0.99      |
| 25309  | 0.99      |
| 20321  | 1.14      |
| 22380  | 1.14      |
| 20356  | 1.14      |

## RFM Segmentation
### 12. Creating RFM Scores
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
### 13. Analyzing RFM Score Combinations
```sql
SELECT * FROM RFM_SCORE_DATA WHERE RFM_SCORE_COMBINATION = '111';
SELECT * FROM RFM_SCORE_DATA WHERE RFM_SCORE_COMBINATION = '222';
SELECT * FROM RFM_SCORE_DATA WHERE RFM_SCORE_COMBINATION = '333';
SELECT * FROM RFM_SCORE_DATA WHERE RFM_SCORE_COMBINATION = '444';
```
### 14. Creating Customer Segments
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
### 15. Segment-wise Customer Analysis
```sql
SELECT
    CUSTOMER_SEGMENT,
    COUNT(*) AS NUMBER_OF_CUSTOMERS,
    ROUND(AVG(MONETARY_VALUE),0) AS AVERAGE_MONETARY_VALUE
FROM RFM_ANALYSIS
GROUP BY CUSTOMER_SEGMENT;
```

## Results & Findings
- There were **no duplicate rows**.
- There were **no invalid order quantity**.
- There were **no invalid unit price**.
- The **last order date was 2013-12-31**.
- Customers are segmented into different categories like Loyal, Active, Churned, etc. to helps in targeted marketing strategies and better customer relationship management
- The insight suggests that while **loyal customers generate the highest revenue**, there are **significant numbers of churned customers and potential churners** that may require retention strategies.

## Customer Segments

| Category                          | NUMBER_OF_CUSTOMERS | AVERAGE_MONETARY_VALUE |
|-----------------------------------|---------|---------|
| CHURNED CUSTOMER                 | 622     | 486     |
| POTENTIAL CHURNERS               | 349     | 1067    |
| Other                             | 565     | 3231    |
| ACTIVE                            | 366     | 952     |
| NEW CUSTOMERS                     | 17      | 185     |
| LOYAL                             | 434     | 8502    |
| SLIPPING AWAY, CANNOT LOSE        | 349     | 6907    |


## Technologies Used
- **Database**: MySQL
- **Data Analysis**: SQL Queries

## How to Use
1. Clone the repository.
2. Import the data and SQL scripts into MySQL.
3. Run the queries sequentially to clean, analyze, and segment the data.

## Author
Zulkarnain Saurav

