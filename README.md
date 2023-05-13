# From Excel to SQL: Building a Data Pipeline for Online Retail Analysis with Azure and Python

## Information about the Data
  [Online Retail Data Set](https://archive.ics.uci.edu/ml/datasets/Online+Retail) captures transactional data from 01/12/2010 to 09/12/2011 for a UK-based non-store online retailer that specializes in selling unique gifts for all occasions. The dataset includes details of purchases made by both retail and wholesale customers.

### Here are the attributes of this dataset:
**InvoiceNo:** Invoice number. Nominal, a 6-digit integral number uniquely assigned to each transaction. If this code starts with letter 'c', it indicates a cancellation. <br>
**StockCode:** Product (item) code. Nominal, a 5-digit integral number uniquely assigned to each distinct product. <br>
**Description:** Product (item) name. Nominal. <br>
**Quantity:** The quantities of each product (item) per transaction. Numeric. <br>
**InvoiceDate:** Invice Date and time. Numeric, the day and time when each transaction was generated. <br>
**UnitPrice:** Unit price. Numeric, Product price per unit in sterling. <br>
**CustomerID:** Customer number. Nominal, a 5-digit integral number uniquely assigned to each customer. <br>
**Country:** Country name. Nominal, the name of the country where each customer resides. <br>

## Initial Steps on Python
  The dataset comes in a xlsx file format, I will first read this file on Python and turn it into a dataframe for easier manipulation.
```python
import pandas as pd
import datetime
import numpy as np
df = pd.DataFrame(pd.read_excel("Online Retail.xlsx"))
```

To check if missing values are present I wrote a program that shows the percentage of missing values per each column.
```python
for col in df.columns:
    missing_percentage = np.mean(df[col].isnull())
    print(col, missing_percentage)
```
```
InvoiceNo 0.0
StockCode 0.0
Description 0.002683107311375157
Quantity 0.0
InvoiceDate 0.0
UnitPrice 0.0
CustomerID 0.249266943342886
Country 0.0
```
Based on the above output, Description and CustomerID columns have missing values. I can easily get rid of them at this stage, before they can mess up the process while copying the data to a SQL table. 
```python
df = df.dropna()
```
Checking the data types is also a good idea to help me create the template table on SQL later on. 

```python
df.dtypes
```
```
InvoiceNo              object
StockCode              object
Description            object
Quantity                int64
InvoiceDate    datetime64[ns]
UnitPrice             float64
CustomerID            float64
Country                object
```
I will change the data type for CustomerID column as it seems to be a float right now, but it is more suitable to be an integer data type.

```python
df['CustomerID'] = df['CustomerID'].astype(int)
```
With that, the only thing left for me to do on Python is to get an output file with csv file format.

```python
df.to_csv("Online Retail.csv",index=None,header=True) 
```
Now I am ready to go on Azure and continue the rest of my project on there.

## Creating the Pipeline on Azure

As part of this project, I have created a data factory, a SQL database and a blob storage for this project. After uploading the Online Retail.csv file I created on Python to the blob storage, I created a pipeline on Azure Data Factory called "Blob to SQL". By linking the SQL database and blob storage to the pipeline, I have access to both of them to use in my pipeline. My goal with this pipeline was to load the Online Retail.csv file into a SQL table. To achieve this, I created an empty table named Retail_data in the SQL database with the following schema:

```sql
CREATE TABLE Retail_data (
InvoiceNo varchar(50),	
StockCode varchar(50),	
Description varchar(100),	
Quantity int,	
InvoiceDate datetime,	
UnitPrice decimal(18,2),
CustomerID int,	
Country varchar(50)
)
```
With this table in place, I created datasets on Azure Data Factory for both Online Retail.csv and the Retail_data table. The Online Retail.csv dataset served as the source, and the Retail_data dataset as the sink for the copy data activity in the "Blob to SQL" pipeline. The copy data activity was configured to move data from the source dataset to the sink dataset, effectively loading the contents of Online Retail.csv into the Retail_data table in the SQL database. I named the copy data activity "Online Retail dataset csv to SQL".  I also debugged the pipeline to ensure everything was working correctly before publishing it.

<p align= "center">
  <img src ="https://github.com/tubako/retail-analytics/assets/70372302/08b6203d-8feb-40f8-996c-70f1175d0453">
<p/>

## Exploring and Analyzing Online Retail Data on Azure SQL Database

Below is the Retail_data table with its first few lines.

<p align = "center">
  <img src = "https://github.com/tubako/retail-analytics/assets/70372302/a75e3d11-fb89-4d8b-b490-6c2f2b4d7449">
<p/>

Before starting the analysis, to undeerstand the data more I summarize some aspects of the data. 

- Total revenue: 16600131.62 GBP
  ```sql
  SELECT sum(Quantity*UnitPrice) AS TotalRevenue
  FROM Retail_data
  ```  
- Top-selling products:
  ```sql
  SELECT Description, SUM(Quantity) as TotalSold
  FROM Retail_data
  GROUP BY Description
  ORDER BY TotalSold DESC
  ```
  <P>
    <img src= "https://github.com/tubako/retail-analytics/assets/70372302/2cc8f4f4-a89d-4453-bac0-312045d22a7f">
  <p/>  
- Countries with the most customers:
  ```sql
  SELECT Country,  COUNT(DISTINCT CustomerID) AS NumCustomers
  FROM Retail_data
  GROUP BY Country
  ORDER BY NumCustomers DESC
  ```
  <p>
    <img src="https://github.com/tubako/retail-analytics/assets/70372302/55b05ab4-48f0-41a3-b96e-6bccc995b137">
  <p/>
  
After these warm-up queries, I have some questions I want to answer about this data.  
1. What are the top-selling products in each country?  
2. Which customers have the highest purchase frequency and total spending?  
3. What is the daily, weekly, or monthly sales trend?  
4. Which days of the week or times of the day have the highest sales?
