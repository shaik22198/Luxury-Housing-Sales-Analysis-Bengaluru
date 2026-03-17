# Luxury Housing Data Analysis – Bangalore

## Overview

This project performs **Exploratory Data Analysis (EDA)** on a dataset of luxury residential properties in **Bangalore, India**. The objective is to clean, transform, and analyze the data to prepare it for **business insights and dashboard visualization**.

The analysis includes:

* Data cleaning
* Handling missing values
* Detecting and handling outliers
* Feature engineering
* Preparing the dataset for **SQL storage and further analytics**

---

# Project Workflow

The project follows a structured **data analysis pipeline**:

1. Data Loading
2. Data Understanding
3. Data Cleaning
4. Outlier Detection
5. Missing Value Imputation
6. Feature Engineering
7. Data Export to SQL

---

# Technologies Used

* **Python**
* **Pandas**
* **NumPy**
* **Matplotlib**
* **SQLAlchemy**
* **MySQL**
* **Jupyter Notebook**

---

# Dataset

The dataset contains luxury housing property transaction details for Bangalore.

### Important Columns

| Column           | Description                              |
| ---------------- | ---------------------------------------- |
| Property_ID      | Unique property identifier               |
| Developer_Name   | Name of the property developer           |
| Micro_Market     | Local real estate market area            |
| Configuration    | Property configuration (3BHK, 4BHK etc.) |
| Unit_Size_Sqft   | Size of the apartment                    |
| Ticket_Price_Cr  | Total property price in crores           |
| Transaction_Type | Primary / Secondary transaction          |
| Amenity_Score    | Score representing available amenities   |
| Buyer_Comments   | Buyer feedback                           |
| Purchase_Quarter | Date of purchase                         |

---

# 1. Data Loading

The dataset is loaded using **Pandas**.

```python
import pandas as pd

df = pd.read_csv("Luxury_Housing_Bangalore.csv")
```

Initial inspection:

```python
df.head()
df.info()
df.shape
```

---

# 2. Data Understanding

Each column's distribution is explored using:

```python
for col in df:
    print(df[col].value_counts())
```

This helps understand:

* Data distribution
* Category frequency
* Potential anomalies

---

# 3. Data Cleaning

## Price Column Cleaning

The price column contained currency symbols.

Example values:

```
₹12.5 Cr
```

Converted to numeric values:

```python
df['Ticket_Price_Cr'] = df['Ticket_Price_Cr']\
.str.replace("Cr","")\
.str.replace("₹","")\
.str.strip()\
.astype(float)
```

---

## Date Conversion

```python
df['Purchase_Quarter'] = pd.to_datetime(df['Purchase_Quarter'])
```

---

## Standardizing Categorical Variables

Configuration values were standardized.

```python
df['Configuration'] = df['Configuration'].replace({
'3bhk':'3BHK',
'3Bhk':'3BHK',
'4bhk':'4BHK',
'4Bhk':'4BHK',
'5bhk+':'5BHK+'
})
```

Micro market names were cleaned:

```python
df['Micro_Market'] = df['Micro_Market'].str.strip().str.title()
```

---

# 4. Removing Duplicate Records

Duplicate properties were identified using:

```python
df['Property_ID'].duplicated().sum()
```

Duplicates were removed:

```python
df = df.drop_duplicates(subset='Property_ID').reset_index(drop=True)
```

---

# 5. Outlier Detection

Outliers were detected using the **Interquartile Range (IQR) method**.

```python
def outlier(df):
    Q1 = df.quantile(0.25)
    Q3 = df.quantile(0.75)
    IQR = Q3 - Q1
    upper_limit = Q3 + IQR
    lower_limit = Q1 - IQR
    return upper_limit, lower_limit
```

Example:

```
Unit_Size_Sqft
Upper Limit : 10516
Lower Limit : 1459
```

Outliers were analyzed using filtering.

---

# 6. Handling Invalid Values

### Unit Size

Invalid values such as `-1` were detected.

```python
df[df['Unit_Size_Sqft'] == -1]
```

These values were replaced with **NaN**.

```python
df['Unit_Size_Sqft'] = df['Unit_Size_Sqft'].replace(-1.0, np.nan)
```

---

### Ticket Price

Unrealistic values were removed.

```python
df.loc[df['Ticket_Price_Cr'] < 0, 'Ticket_Price_Cr'] = np.nan
df.loc[df['Ticket_Price_Cr'] > 20, 'Ticket_Price_Cr'] = np.nan
```

---

# 7. Missing Value Imputation

### Unit Size

Missing values filled using developer and configuration average.

```python
df['Unit_Size_Sqft'] = df['Unit_Size_Sqft'].fillna(
df.groupby(['Developer_Name','Configuration'])['Unit_Size_Sqft'].transform('mean')
)
```

---

### Ticket Price

Filled using property size, configuration and transaction type.

```python
df['Ticket_Price_Cr'] = df['Ticket_Price_Cr'].fillna(
df.groupby(['Unit_Size_Sqft','Configuration','Transaction_Type'])['Ticket_Price_Cr'].transform('mean')
)
```

---

### Amenity Score

```python
df['Amenity_Score'] = df['Amenity_Score'].fillna(
df.groupby(['Developer_Name','Configuration'])['Amenity_Score'].transform('mean')
)
```

---

### Buyer Comments

Filled using mode.

```python
df['Buyer_Comments'] = df['Buyer_Comments'].fillna(df['Buyer_Comments'].mode()[0])
```

---

# 8. Feature Engineering

## Price per Square Foot

A new metric was created.

```python
df['Price_per_Sqft'] = df['Ticket_Price_Cr'] * 10000000 / df['Unit_Size_Sqft']
```

---

## Quarter Number

```python
df['Quarter_Number'] = df['Purchase_Quarter'].dt.quarter
```

---

## Booking Flag

```python
df['Booking_Flag'] = 1
```

---

# 9. Final Dataset Inspection

```python
df.info()
df.describe()
```

This ensures:

* No critical missing values
* Correct data types
* Reasonable value ranges

---

# 10. Exporting Data to SQL

The cleaned dataset is stored in **MySQL database**.

```python
from sqlalchemy import create_engine
from urllib.parse import quote_plus

engine = create_engine(
"mysql+pymysql://username:password@localhost:3306/luxury_housing_price_analysis"
)
```

Exporting data:

```python
df.to_sql("luxury_housing_data", engine, if_exists="replace", index=False)
```

---

# Project Outcome

After EDA and preprocessing:

* Dataset cleaned
* Outliers analyzed
* Missing values handled
* Features engineered
* Data stored in **SQL for analytics**

This dataset can now be used for:

* **Power BI dashboards**
* **Real estate price analysis**
* **Market trend analysis**
* **Predictive modeling**

---

# Future Improvements

* Build **interactive Power BI dashboards**
* Perform **location-based price analysis**
* Analyze **price trends over time**
* Build **machine learning models for price prediction**

---

# Author

**Shaik Abdul Kader**

Data Analysis Project – Luxury Housing Market
