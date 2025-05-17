# ![CI logo](https://codeinstitute.s3.amazonaws.com/fullstack/ci_logo_small.png)

# Project Idea 4: Online Retail Transaction Analysis



## Project Overview

This project analyses customer transactions from an online retail dataset to uncover purchasing patterns, understand customer behaviour,  customer segments, and product performance. 

The goal is to generate actionable insights that help optimise pricing, product bundling, and marketing strategies.

Deliverables include clear documentation, structured code, insightful visualisations using Matplotlib, Seaborn and Plotly, and a well-managed workflow using GitHub and Kanban board for version control and task tracking.



## Dataset Content

Source: [Kaggle – Online Retail Transaction Dataset](https://www.kaggle.com/datasets/abhishekrp1517/online-retail-transactions-dataset)


### ETL (Data Extraction, Transformation, and Loading) PROCESS:

  **Step 1: Check what needs to be cleaned**

  - **Initial `df.info()` Output:**

```plaintext
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 541909 entries, 0 to 541908
Data columns (total 8 columns):
 #   Column       Non-Null Count  Dtype  
---  ------       --------------  -----  
 0   InvoiceNo    541909 non-null  object
 1   StockCode    541909 non-null  object
 2   Description  540455 non-null  object
 3   Quantity     541909 non-null  int64  
 4   InvoiceDate  541909 non-null  object
 5   UnitPrice    541909 non-null  float64
 6   CustomerID   541909 non-null  int64  
 7   Country      541909 non-null  object
```
The above table shows that there are 541909 rows in the dataset, the dataset contains 8 columns - INvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID and Country

Description Column total count is **540455** vs the RangeIndex total count is **541909**, ie. 1454 rows are missing
**`InvoiceDate` dtype is an object. It should be converted to a datetime format**


**Step2:**  Initial `df.describe()` Output:

```plaintext
       Quantity      UnitPrice     CustomerID
count  541909.000000  541909.000000  541909.000000
mean       9.552250       4.611114   15287.518434
std      218.081158      96.759853    1484.746041
min   -80995.000000  -11062.060000   12346.000000
25%        1.000000       1.250000   14367.000000
50%        3.000000       2.080000   15287.000000
75%       10.000000       4.130000   16255.000000
max    80995.000000   38970.000000   18287.000000
```
This table shows that the `Quanity`, `UnitPrice` and `CustomerID` have the same count 541909 which means all the 3 columns have no missing values

`mean` - average value of each column Qty = 9.55 per trnx, UnitPrice 4.61

`std` - standard deviation shows that quantity has a large spread 218 vs 9.55, which suggests soeme outliers in the dataset

`UnitPrice` also has a large spread 96.76 vs 4.61, which suggests some high prices in some trnx
Qty wise, 25% of trnx had <=1 unit, 50% had <=3 units and 75% had <=10 units.  This shows that most orders are small.

- Negative values present in the `min` row of `Quantity` and `UnitPrice`. These records should be removed to ensure the integrity of sales analysis.**

- The maximum `Quantity` of **80,995** and `UnitPrice` of **£38,970** indicate possible outliers or rare bulk transactions. These may represent wholesale purchases, manual entry errors, or exceptional product values, and should be reviewed before drawing conclusions from aggregated data.


**Step 3: Missing Values Check:**
check total missing values per column
using `df.isnull().sum()`

```plaintext
InvoiceNo         0
StockCode         0
Description    1454
Quantity          0
InvoiceDate       0
UnitPrice         0
CustomerID        0
Country           0
dtype: int64
```
This table shows that there are **1454** missing values for Description column only


**Step 4: Remove missing values:**
`df_cleaned = df.dropna(subset=['CustomerID', 'Description'])`

This code removes rows with NaN values in the CustomerID or Description columns.
The cleaned data is stored in df_cleaned to avoid overwriting the original dataset (df), so there's a backup in case something goes wrong.

Although CustomerID shows no missing values at this stage, it is included in the dropna call to ensure any previously missing or misread entries are excluded. This guarantees the integrity of customer-level analysis.


**Step 5: Remove negative Quantity and UnitPrice:**
`df_cleaned = df_cleaned[(df_cleaned['Quantity'] > 0) & (df_cleaned['UnitPrice'] > 0)]`


**Step 6: Drop duplicate rows:**
`df_cleaned = df_cleaned.drop_duplicates()`


**Step 7: Convert `InvoiceDate` to datetime**
`df_cleaned['InvoiceDate'] = pd.to_datetime(df_cleaned['InvoiceDate'])`

```plaintext
<class 'pandas.core.frame.DataFrame'>
Index: 540455 entries, 0 to 541908
Data columns (total 8 columns):
 #   Column       Non-Null Count   Dtype         
---  ------       --------------   -----         
 0   InvoiceNo    540455 non-null  object        
 1   StockCode    540455 non-null  object        
 2   Description  540455 non-null  object        
 3   Quantity     540455 non-null  int64         
 4   InvoiceDate  540455 non-null  datetime64[ns]
 5   UnitPrice    540455 non-null  float64       
 6   CustomerID   540455 non-null  int64         
 7   Country      540455 non-null  object        
dtypes: datetime64[ns](1), float64(1), int64(2), object(4)
memory usage: 37.1+ MB
```

**Step 8: check descriptive stats to verify the changes made:**
`df_cleaned.describe()` output:

```plaintext
```plaintext
       Quantity         InvoiceDate          UnitPrice     CustomerID
count  524878.000     524878                524878.000    524878.000
mean       10.617     2011-07-04 15:30:16   3.923         15287.631
min         1.000     2010-12-01 08:26:00   0.001          12346.000
25%         1.000     2011-03-28 12:13:00   1.250          14375.000
50%         4.000     2011-07-20 11:22:00   2.080          15287.000
75%        11.000     2011-10-19 11:41:00   4.130          16245.000
max     80995.000     2011-12-09 12:50:00   13541.330      18287.000
std       156.280     NaN                   36.093         1482.145
```
Now the total count for Quantity, InvoiceDate, UnitPrice and CustomerID all have the same count **524878**
`mean` Qty = 10.617 and UnitPrice = 3.923
`std`  Qty = 156.28 and UnitPrice = 36.093
`min`  Qty <= 1     and UnitPrice = 0.001
`max`  Qty <= 80995 and UnitPrice = 13541.330


**Step 9: Add a new column `TotalPrice` to support Revenue Analysis**
  `df_cleaned['TotalPrice'] = df_cleaned['Quantity'] * df_cleaned['UnitPrice']`

```Plaintext
InvoiceNo StockCode                          Description  Quantity        InvoiceDate  UnitPrice  CustomerID         Country  TotalPrice
0    536365    85123A   WHITE HANGING HEART T-LIGHT HOLDER         6  2010-12-01 08:26:00       2.55       17850  United Kingdom       15.30
1    536365     71053                  WHITE METAL LANTERN         6  2010-12-01 08:26:00       3.39       17850  United Kingdom       20.34
2    536365    84406B       CREAM CUPID HEARTS COAT HANGER         8  2010-12-01 08:26:00       2.75       17850  United Kingdom       22.00
3    536365    84029G  KNITTED UNION FLAG HOT WATER BOTTLE         6  2010-12-01 08:26:00       3.39       17850  United Kingdom       20.34
4    536365    84029E       RED WOOLLY HOTTIE WHITE HEART.         6  2010-12-01 08:26:00       3.39       17850  United Kingdom       20.34
```

**Step 10 check the final cleaned data output
`df_cleaned.info()`

```plaintext
<class 'pandas.core.frame.DataFrame'>
Index: 524878 entries, 0 to 541908
Data columns (total 9 columns):
 #   Column       Non-Null Count   Dtype         
---  ------       --------------   -----         
 0   InvoiceNo    524878 non-null  object        
 1   StockCode    524878 non-null  object        
 2   Description  524878 non-null  object        
 3   Quantity     524878 non-null  int64         
 4   InvoiceDate  524878 non-null  datetime64[ns]
 5   UnitPrice    524878 non-null  float64       
 6   CustomerID   524878 non-null  int64         
 7   Country      524878 non-null  object        
 8   TotalPrice   524878 non-null  float64       
dtypes: datetime64[ns](1), float64(2), int64(2), object(4)
memory usage: 40.0+ MB
```

`df_cleaned.describe()`

```plaintext

        Quantity	   InvoiceDate	                    UnitPrice	   CustomerID	    TotalPrice
count	524878.000000	524878	                        524878.000000	524878.000000	524878.000000
mean	10.616600	    2011-07-04 15:30:16.317049088	3.922573	    15287.631345	20.275399
min	    1.000000	    2010-12-01 08:26:00	            0.001000	    12346.000000	0.001000
25%	    1.000000	    2011-03-28 12:13:00	            1.250000	    14375.000000	3.900000
50%	    4.000000	    2011-07-20 11:22:00	            2.080000	    15287.000000	9.920000
75%	   11.000000	    2011-10-19 11:41:00	            4.130000	    16245.000000	17.700000
max	80995.000000	    2011-12-09 12:50:00	        13541.330000	    18287.000000	168469.600000
std	  156.280031	    NaN	                           36.093028	    1482.145530	    271.693566
```
`df_cleaned.shape`
```plaintext
(524878, 9)
```



## Business Requirements
- Identify high-value and at-risk customers
- Discover popular and premium-priced products
- Detect seasonal trends and sales patterns
- Propose pricing, promotion, and customer reactivation strategies


## Project Plan
### High-Level Steps:
1. Data extraction and loading
2. Cleaning and transformation
3. Exploratory analysis and visualisation
4. Recency, Frequency, Monetary (RFM) segmentation
   - Recency:  how recently a customer made a purchase
   - Frequency: how often they purchase
   - Monetary: how much money they spend
5. Strategic recommendations
6. Reflection and documentation

