# E-Commerce Data Analysis
A comprehensive project to clean and analyze E-commerce transactional data using SQL (Microsoft SQL Server Management Studio - SSMS), addressing key business questions and deriving actionable insights.

## Dataset Overview

### Data Sources
The dataset used in this project was obtained from the DataDNA dataset challenges provided by Onyx Data. It consists of a single table with 8 columns and 541,909 rows. The dataset contains transactional data spanning from 2010 to 2011, tracking purchases made by customers.

To access the dataset, click [**here**](https://github.com/Rolakamin/E-Commerce-Data-Analysis/blob/main/Ecommerce%20Business.zip).




### Dataset Description
The table includes the following fields:
- InvoiceNo: Unique identifier for each transaction.
  
  **Note**: For this analysis, InvoiceNo was not treated as a unique identifier because each InvoiceNo represents multiple transactions. To resolve this, a TransactionID column was created to serve as the primary key.
- StockCode: Represents each product.
- Description: Describes the product.
- Quantity: Number of items purchased.
- UnitPrice: Price per unit of the product.
- InvoiceDate: Date and time of the transaction.
- CustomerID: Identifier for each customer.
- Country: Country where the purchase was made

## Data Cleaning
**Step 1: Removal of Duplicates**

The E-commerce dataset contained potential duplicate records, which could impact the accuracy and reliability of the analysis. These duplicates needed to be identified and removed to ensure data integrity.

**Approach**

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
- Row Count After Removing Duplicates: 531,223
- Duplicate Rows Removed: 10,686
  
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

The dataset was successfully cleaned by removing all duplicate rows, reducing the total row count from 541,909 to 531,223.

This step ensures that the dataset is now free of redundancy and ready for further analysis.

**Step 2: Handle Missing or Blank Values**

**a) CustomerID**

Missing values in the CustomerID column could represent data entry errors or incomplete data. These blanks need to be addressed to ensure that customer-based analysis is accurate.

**Approach:**

- The missing CustomerID values were first identified using a query.
- Missing values were replaced with 'Unknown' to indicate non-specific customer data.
  
**Queries:**

**Check for Missing CustomerID Values:**

```sql
SELECT COUNT(*) AS MissingCustomerIDs
FROM Ecommerce
WHERE CustomerID IS NULL;
```

**Results**

- Initial Row Count: 541,909
- Row Count After Removing Duplicates: 531,223
- Duplicate Rows Removed: 10,686
- Missing CustomerID Count (After Removing Duplicates): 134,542
  
**Interpretation:**
134,542 rows had missing CustomerID values. This could be due to transactions where customers did not provide an ID, incomplete records, or data entry errors.

To ensure accurate customer-based analysis, these missing values will be replaced with 'Unknown'

**Replace NULL CustomerID with 'Unknown':**

```sql
UPDATE Ecommerce
SET CustomerID = 'Unknown'
WHERE CustomerID IS NULL;
```

**b) Description**

The Description column contained missing, invalid, or placeholder values such as '???', '?', or 'Blanks', which needed to be addressed to ensure data consistency and accuracy in analysis.

To handle this, a query was used to detect descriptions that were either missing or contained placeholder values.

```sql
SELECT DISTINCT Description
FROM Ecommerce
WHERE Description LIKE '%?%' OR Description = 'Blanks';
```

This query identified **24 unique descriptions** that were flagged as problematic.

Thereafter, all identified invalid or missing descriptions were replaced with 'Unknown' to maintain consistency in the dataset.

```sql
UPDATE Ecommerce 
SET Description = 'Unknown'
WHERE Description LIKE '%?%' OR Description = 'Blanks';
```

After running this update query, **82 rows** were affected, meaning these descriptions were successfully replaced. To confirm the update, the following query was executed:

```sql
SELECT *  
FROM Ecommerce
WHERE Description = 'Unknown';
```

This returned **82 rows**, verifying that all invalid or missing descriptions were successfully updated.

**c) Country**

The Country column contained values labeled as **'Unspecified'**, which needed to be addressed for consistency in the dataset. There were no **NULL** values in this column, but some records had 'Unspecified' as the country value.

To identify how many rows were affected, the following query was used:

```sql
SELECT COUNT(*) AS UnspecifiedCountries
FROM Ecommerce
WHERE Country = 'Unspecified';
```
**Result**: **433 rows** were found with the 'Unspecified' value.

These rows were then updated with the value **'Unknown'** to maintain consistency across the dataset.

```sql
UPDATE Ecommerce
SET Country = 'Unknown'
WHERE Country = 'Unspecified';
```
















