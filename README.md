# From Excel to SQL: Building a Data Pipeline for Online Retail with Azure and Python

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

I will be using Azure and Before starting on the pipeline, I have created a data factory, a SQL database and a blob storage for this project. After uploading the Online Retail.csv file I created on Python to the blob storage, I create a pipeline on Azure Data Factory called "Blob to SQL". By linking the  
