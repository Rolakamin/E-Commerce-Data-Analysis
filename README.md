# E-Commerce Data Analysis
A comprehensive project to clean and analyze E-commerce transactional data using SQL (Microsoft SQL Server Management Studio - SSMS), addressing key business questions and deriving actionable insights.

## Dataset Overview

### Data Sources
The dataset used in this project was obtained from the DataDNA dataset challenges provided by Onyx Data. It consists of a single table with 8 columns and 541,909 rows. The dataset contains transactional data spanning from 2010 to 2011, tracking purchases made by customers.

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

