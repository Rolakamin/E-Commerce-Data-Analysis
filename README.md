# E-Commerce Data Analysis
A comprehensive project to clean and analyze E-commerce transactional data using SQL (Microsoft SQL Server Management Studio - SSMS), addressing key business questions and deriving actionable insights.

## Project Overview

### Objectives

The goal of this project is to analyze an E-commerce dataset to uncover insights related to sales performance, customer behavior, and product profitability. This will help improve business decision-making in areas such as inventory management, marketing strategies, and customer segmentation.

## Business Problems & Key Questions

To achieve the above objective, the analysis will focus on answering these key business questions:

**1. Sales Performance Analysis**

- Which products generate the most revenue?
- Which countries contribute the highest revenue?
- What is the average order value (AOV)?

**2. Customer Segmentation**

- Which customers have the highest total spending? 
- Who are the top 5 highest-spending customers? 
- Which customers make purchases most frequently? 
- How can we categorize customers based on their spending levels?

**3. Product Performance Analysis**

- What are the best-selling products?
- Which products are the most profitable?
- What are the top 5 most expensive products?

 **4. Time Series Analysis**

- How does revenue change over time (monthly, quarterly, yearly)?
- Which time of day generates the most sales?
- What is the month-over-month revenue growth?

## Dataset Overview

### Data Source
The dataset was obtained from the DataDNA dataset challenges(Onyx Data) and contains 541,909 rows and 8 columns, tracking transactions from 2010 to 2011. 

To access the dataset, click [**here**](https://github.com/Rolakamin/E-Commerce-Data-Analysis/blob/main/Ecommerce%20Business.zip).

### Dataset Description

- InvoiceNo: Unique identifier for each transaction.
- StockCode: Product identifier
- Description: Name of the product. 
- Quantity: Number of items purchased.
- UnitPrice: Price per unit of the product.
- InvoiceDate: Date and time of the transaction.
- CustomerID: Unique identifier for each customer.
- Country: Country where the purchase was made
  
**Note**:
  
For this analysis, InvoiceNo was not unique, that is, InvoiceNo was not treated as a unique identifier because each InvoiceNo represents multiple transactions(multiple rows can belong to the same transaction). To resolve this, a TransactionID column was created to serve as the primary key.

Below is a screenshot of the e-commerce dataset queried in SQL Server after adding the `TransactionID` column:

![Screenshot of E-Commerce Data in SQL Server](https://github.com/Rolakamin/E-Commerce-Data-Analysis/blob/main/E-commerce%20Sample%20Data.png)

## Data Cleaning

Since the dataset contains errors, inconsistencies, and missing values, data cleaning techniques were applied using SQL to ensure accuracy.

**Step 1: Removal of Duplicates**

Some transactions were duplicated,which could impact the accuracy and reliability of the analysis.

The E-commerce dataset contained potential duplicate records, which could impact the accuracy and reliability of the analysis. These duplicates needed to be identified and removed to ensure data integrity.

**Approach:**

To address this issue:

1. **Identifying Duplicates:**

Duplicates were checked based on the following fields: InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, and Country.
This process excluded the TransactionID column, as it was treated as a unique identifier.

2. **Removing Duplicates:**

- Rows identified as duplicates were removed while retaining only the first occurrence of each duplicate group.
- A Common Table Expression (CTE) was used to rank duplicate rows and delete the redundant entries.
  
**Queries**

1. Identifying Duplicates
   
   The following SQL query was used to detect duplicate records:
   
```sql
SELECT 
    InvoiceNo, 
    StockCode, 
    Description, 
    Quantity, 
    InvoiceDate, 
    UnitPrice, 
    CustomerID, 
    Country, 
    COUNT(*) AS DuplictateCount
FROM Ecommerce
GROUP BY 
    InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country
HAVING COUNT(*) > 1;
```

2. Removing Duplicates
   
   The following SQL query was used to remove duplicate records:
   
```sql
WITH CTE AS (
    SELECT 
        *, 
        ROW_NUMBER() OVER (
            PARTITION BY InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country 
            ORDER BY TransactionID
        ) AS RowNumber
    FROM Ecommerce
)
DELETE FROM CTE
WHERE RowNumber > 1;
```

**Results**

- Initial Row Count: 541,909
- Duplicate Rows Removed: 5,268
- Final Row Count After Removing Duplicates: 536,641

  
**Verification**

To confirm the removal of duplicates, the following SQL query was executed, which returned no results, indicating that the dataset no longer contained duplicate rows:

```sql
SELECT 
    InvoiceNo, 
    StockCode, 
    Description, 
    Quantity, 
    InvoiceDate, 
    UnitPrice, 
    CustomerID, 
    Country, 
    COUNT(*) AS DuplicateCount
FROM Ecommerce
GROUP BY 
    InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country
HAVING COUNT(*) > 1;
```

**Observations**

The dataset was successfully cleaned by removing all duplicate rows, reducing the total row count from 541,909 to 536,641.

This step ensures that the dataset is now free of redundancy and ready for further analysis.

**Step 2: Handling Missing Values**

**a) CustomerID**
The CustomerID column contained missing values, which could indicate incomplete records or data entry errors. Addressing these missing values ensures accurate customer-based analysis.

**Approach:**

- The missing CustomerID (Nullvalues) were first identified using a query.
- NULLs were replaced with **'Unknown'** to indicate non-specific customer data and to maintain consistency
  
**Queries:**

**Check for Missing CustomerID Values:**

```sql
SELECT COUNT(*) AS MissingCustomerIDs
FROM Ecommerce
WHERE CustomerID IS NULL;
```

**Result:**
- Missing CustomerID Count (After Removing Duplicates): 135, 037
  
**Interpretation:**

135, 037 rows had missing CustomerID values. This could be due to transactions where customers did not provide an ID, incomplete records, or data entry errors.

To ensure accurate customer-based analysis, these missing values were replaced with 'Unknown' using the following query:

```sql
UPDATE Ecommerce
SET CustomerID = 'Unknown'
WHERE CustomerID IS NULL;
```

**b) Description**

The Description column contained missing, invalid, or unclear values, such as '???', '?', 'Blanks', and NULL values. Additionally, there were inconsistent formatting issues, extra spaces, and vague descriptions that needed to be addressed to ensure data consistency and improve the accuracy of analysis.

1. Identifying and Handling Missing or Placeholder Values
   
To detect missing or placeholder descriptions, a query was executed to identify values such as '???', '?', 'Blanks', and NULL entries. These were replaced with 'UNKNOWN' to maintain uniformity and avoid ambiguity.

```sql
SELECT DISTINCT Description 
FROM Ecommerce 
WHERE Description IS NULL 
   OR Description LIKE '%?%' 
   OR Description = 'Blanks';
```

```sql
UPDATE Ecommerce 
SET Description = 'UNKNOWN' 
WHERE Description IS NULL 
   OR Description LIKE '%?%' 
   OR Description = 'Blanks';
```

2. Removing Extra Spaces and Hidden Characters
   
Some descriptions had leading/trailing spaces and non-printable characters (e.g., CHAR(160), a non-breaking space). They were removed to ensure consistency.

```sql
-- Query to Trim spaces and Remove Hidden Characters
UPDATE Ecommerce
SET Description = TRIM(REPLACE(Description, CHAR(160), ''));
```

3. Identifying and Grouping Irrelevant Descriptions
   
To find descriptions that were too vague or unhelpful, occurences of each unique description were counted.

```sql
-- Query to Count Unique Descriptions
SELECT Description, COUNT(*) 
FROM Ecommerce
GROUP BY Description
ORDER BY COUNT(*) DESC;
```

This query helped to identify frequent but unclear descriptions which were categorized as irrelevant:

- Miscellaneous
- Damaged Item
- E-commerce Issue
- Stock Adjustment
- Returned Item
- Adjust bad debt

 4. Validating the Categorized Descriptions
 After grouping these irrelevant descriptions, their total counts were verified to confirm their significance.

```sql
--Query to Count Occurrences of Unwanted Descriptions
SELECT Description, COUNT(*) 
FROM Ecommerce
WHERE Description IN ('Miscellaneous', 'Damaged Item', 'E-commerce Issue', 
                      'Stock Adjustment', 'Returned Item', 'Adjust bad debt')
GROUP BY Description;
```

5. Filtering Out Unwanted Descriptions
   
```sql
-- Query to Exclude Unwanted Descriptions
SELECT * 
FROM Ecommerce
WHERE Description NOT IN ('Miscellaneous', 'Damaged Item', 'E-commerce Issue, 'Stock Adjustment', 'Returned Item', 'Adjust bad debt');
```
This query returned 534, 168 rows. 

6. Standardizing and Formatting Descriptions
   
To ensure consistency in the dataset, all descriptions were converted to uppercase. Since most relevant descriptions were already in uppercase, this step ensured uniform formatting across all records.

```sql
UPDATE Ecommerce
SET Description = UPPER(TRIM(Description));
```

**c) Country**

The Country column contained values labeled as 'Unspecified', which needed to be addressed for consistency in the dataset. There were no NULL values in this column, but some records had 'Unspecified' as the country value.

To identify affected rows, the following query was used:

```sql
SELECT COUNT(*) 
FROM Ecommerce
WHERE Country = 'Unspecified';
```

To maintain consistency, all occurrences of 'Unspecified' were replaced with 'Unknown' using the following update query:

```sql
UPDATE Ecommerce
SET Country = 'Unknown'
WHERE Country = 'Unspecified';
```

**Step 3: Correcting Inconsistent Data**

Inconsistent data can arise from errors in data collection or entry. In this dataset, negative values were found in the Quantity and UnitPrice columns, which indicated incorrect records.

**Cleaning Negative Values in the Quantity Column**

The goal of this process was to investigate and handle negative values in the Quantity column. Negative quantities often indicate returns, stock adjustments, or errors. This data was systematically analyzed and cleaned to ensure data integrity.

**Identifying Negative Quantities**

The records that contained negative quantities were checked. 

```sql
SELECT COUNT(*)
FROM Ecommerce
WHERE Quantity < 0;
```
**Initial Findings:**
**10,587** rows had negative values in the Quantity column

To further understand these records, the first 10 rows were retrieved.

```sql
SELECT TOP 10 * 
FROM Ecommerce
WHERE Quantity < 0;
```

**Investigating Negative Quantity Patterns**

Furthermore, InvoiceNo column was checked to analyze whether negative quantities were associated with returns or other transaction types.

```sql
SELECT * 
FROM Ecommerce 
WHERE InvoiceNo LIKE '%C%';
```

**Findings:**

- **9,251** rows had InvoiceNo values containing the letter **'C'**, suggesting these were canceled transactions or returns.
- However, 1,336 rows had negative quantities but did not have 'C%' in InvoiceNo.

To isolate these non-return-related records:

```sql
SELECT * 
FROM Ecommerce 
WHERE Quantity < 0 
AND InvoiceNo NOT LIKE 'C%';
```

**Findings:**

The **1,336** rows likely represented **stock adjustments**, **damaged items**, and other **discrepancies**.

**Creating a Backup for Returned Transactions**

Before deletion, all transactions with negative quantities and an InvoiceNo starting with 'C' (returns) were backed up.

```sql
SELECT * INTO Ecommerce_Returns 
FROM Ecommerce 
WHERE InvoiceNo LIKE 'C%';
```

**Deleting Returns from the Main Table**

After saving the backup,transactions from the main Ecommerce table were removed.

```sql
DELETE FROM Ecommerce 
WHERE InvoiceNo LIKE 'C%';
```

**Results:**

**9,251** rows were deleted from the main table.

**Investigating Remaining Negative Quantities**

After removing returns, the records were checked to know if there was still negative quantities or not. 

```sql
SELECT COUNT(*)
FROM Ecommerce
WHERE Quantity < 0;
```

 **Findings:**

1,336 rows still had negative quantities.

To understand these records, the Description column was examined.

**Findings:**

Most of these records had the following descriptions:

- Miscellaneous
- E-commerce Issue
- Damaged Item
- Stock Adjustment 
- Returned Item
- Adjust Bad Debt
- 
Since these descriptions were grouped as **irrelevant**, they were targeted for deletion.

**Deleting Irrelevant Descriptions**

To remove records with negative quantities belonging to these irrelevant categories, this query was executed:

```sql
DELETE FROM Ecommerce 
WHERE Quantity < 0 
AND Description IN ('Miscellaneous', 'E-commerce Issue', 'Damaged Item', 'Stock Adjustment', 'Returned Item', 'Adjust bad debt');
```

**Results:**

**1,324** rows were deleted.

To confirm only necessary records remained, the query below was executed again.

```sql
SELECT COUNT(*)
FROM Ecommerce
WHERE Quantity < 0;
```

**Final Result:**

0 rows had negative quantities, meaning the column was successfully cleaned.

**Handling Negative Values in the UnitPrice Column**

**Step 1: Identifying Negative Unit Prices**

```sql
SELECT * 
FROM Ecommerce 
WHERE UnitPrice < 0;
```

**Findings:**

-**2 records** had a UnitPrice of **-11062.06.**
-The **Description** for these transactions was **"Adjust bad debt"**, suggesting they were accounting adjustments rather than actual product sales.
-The **CustomerID** was marked as **"Unknown"**, further indicating that these were not standard customer transactions.

**Step 2: Investigating the Frequency of "Adjust bad debt"**

```sql
SELECT COUNT(*) 
FROM Ecommerce 
WHERE Description = 'Adjust bad debt';
```

**Findings:**

**3 records** had the Description **"Adjust bad debt"** in total.

**Step 3: Backing Up Affected Records**

Before making any changes, the affected rows were backed up  into a separate table. This allows a record of these adjustments to be retained in case they are needed later.

```sql
SELECT * INTO Ecommerce_BadDebt
FROM Ecommerce
WHERE Description = 'Adjust bad debt';
```

**Step 4: Cross-Checking the Backup Table**

```sql
SELECT * FROM Ecommerce_BadDebt;
```

**Findings:**

**1 record** had a **positive UnitPrice (11062.06).**
**2 records** had **negative UnitPrice values (-11062.06 each).**

This confirms that the negative values were likely adjustments reversing the original transaction, that is, the negative values were not actual refunds or sales—they were adjustments to remove or correct an earlier transaction, ensuring accurate financial reporting.

**Step 5: Removing "Adjust Bad Debt" Entries from the Sales Data**

Since these records represent financial adjustments rather than actual sales, they should be removed from sales analysis to prevent distortions in revenue calculations.

```sql
DELETE FROM Ecommerce
WHERE Description = 'Adjust bad debt';
```

## Data Analysis & SQL Queries

### Sales Performance Analysis

**Objective:** This analysis aims to understand how products are selling across different countries and customers based on transactional data.

1. Which products generate the most revenue?
   
```sql
  SELECT TOP 10 
    Description, 
    ROUND(SUM(UnitPrice * Quantity), 2) AS TotalRevenue
FROM Ecommerce
GROUP BY Description
ORDER BY TotalRevenue DESC;
```































1. Total Sales by Product
   
```sql
SELECT 
    StockCode, 
    Description, 
    SUM(Quantity * UnitPrice) AS TotalSales
FROM Ecommerce
GROUP BY StockCode, Description
ORDER BY TotalSales DESC;
```

2. Total Revenue by Country
   
```sql
SELECT 
    Country, 
    SUM(Quantity * UnitPrice) AS TotalRevenue
FROM Ecommerce
GROUP BY Country
ORDER BY TotalRevenue DESC;
```

3. Average Order Value (AOV)
   
 ```sql  
SELECT 
    CustomerID, 
    COUNT(DISTINCT InvoiceNo) AS TotalOrders,
    SUM(Quantity * UnitPrice) AS TotalSpending,
    SUM(Quantity * UnitPrice) / COUNT(DISTINCT InvoiceNo) AS AvgOrderValue
FROM Ecommerce
WHERE CustomerID <> 'Unknown'
GROUP BY CustomerID
ORDER BY AvgOrderValue DESC;
```
**Customer 16446 had the highest Average Order Value (AOV) in the dataset, meaning they spent significantly more per order than any other customer**

### Customer Segmentation

**Objective:** Identify and analyze customer purchasing behavior to classify customers based on their total spending, purchase frequency, and overall value to the business.

4. Which customers have the highest total spending?
   
```sql 
SELECT 
    CustomerID, 
    SUM(Quantity * UnitPrice) AS TotalSpending
FROM Ecommerce
WHERE CustomerID <> 'Unknown'
GROUP BY CustomerID
ORDER BY TotalSpending DESC;
```

5. Who are the top 5 highest-spending customers?

```sql 
WITH CustomerRevenue AS (
    SELECT 
        CustomerID, 
        SUM(Quantity * UnitPrice) AS TotalRevenue,
        RANK() OVER (ORDER BY SUM(Quantity * UnitPrice) DESC) AS Rank
    FROM Ecommerce
    WHERE CustomerID <> 'Unknown'
    GROUP BY CustomerID
)
SELECT *
FROM CustomerRevenue
WHERE Rank <= 5;
```

6. Which customers make purchases most frequently?

```sql 
SELECT 
    CustomerID, 
    COUNT(DISTINCT InvoiceNo) AS PurchaseFrequency
FROM Ecommerce
WHERE CustomerID <> 'Unknown'
GROUP BY CustomerID
ORDER BY PurchaseFrequency DESC;
```

7. How can we categorize customers based on their spending levels?
   
```sql 
SELECT 
    CustomerID, 
    SUM(Quantity * UnitPrice) AS TotalSpending,
    CASE 
        WHEN SUM(Quantity * UnitPrice) > 10000 THEN 'High-Value Customer'
        WHEN SUM(Quantity * UnitPrice) BETWEEN 1000 AND 10000 THEN 'Medium Spender'
        ELSE 'Low-Value Customer'
    END AS CustomerSegment
FROM Ecommerce
WHERE CustomerID <> 'Unknown'
GROUP BY CustomerID
ORDER BY TotalSpending DESC;
```
























