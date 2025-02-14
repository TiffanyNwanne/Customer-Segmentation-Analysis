# Data Analysis Methodology Report

## **1. Introduction**

This report outlines the methodologies used for exploring, cleaning, and analyzing banking customer data using **Python (Pandas & Matplotlib) and SQL**. The steps include **data exploration, data cleaning, segmentation, visualization, and machine learning-based clustering** to generate insights and recommendations.

---

## **2. Data Exploration in Python (Pandas & Matplotlib)**

### **A. View Data Sample**

To gain an overview of the dataset, we displayed the first 20 rows:

```python
python
Copy
df.head(20)  # Displays the first 20 rows

```

### **B. Checking Unique Values in Categorical Columns**

To understand the variety of customer locations:

```python
python
Copy
for col in ["Location (City/State)"]:
    print(f"{col} unique values: {df[col].unique()}")

```

### **C. Handling Missing Values**

To check for missing data across all columns:

```python
python
Copy
df.isnull().sum()  # Count missing values per column

```

### **D. Data Visualization**

To understand customer demographics and behavioral trends, various visualizations were created using Matplotlib and Seaborn.

- **Age Distribution**

```python
python
Copy
import matplotlib.pyplot as plt
import seaborn as sns

plt.figure(figsize=(8,5))
sns.histplot(df["Age"], bins=20, kde=True)
plt.title("Age Distribution")
plt.show()

```

- **Transaction Frequency by Gender**

```python
python
Copy
plt.figure(figsize=(8,5))
sns.boxplot(x=df["Gender"], y=df["Transaction Frequency (per month)"])
plt.title("Transaction Frequency by Gender")
plt.show()

```

- **Loan Status Count**

```python
python
Copy
sns.countplot(x=df["Loan Status"])
plt.show()

```

---

## **3. Data Cleaning in Python**

### **A. Removing Duplicate Records**

To ensure data integrity:

```python
python
Copy
df = df.drop_duplicates()
df = df.drop_duplicates(subset=["Customer ID"])  # Based on specific columns
df = df.reset_index(drop=True)  # Reset index after removing duplicates

```

### **B. Mapping States to Regions**

Extracting state information:

```python
python
Copy
df["State"] = df["Location (City/State)"].apply(lambda x: x.split()[-1])

```

Mapping states to regions:

```python
python
Copy
state_to_region = {
    "Lagos": "South-West", "Ibadan": "South-West",
    "Enugu": "South-East", "Kano": "North-West",
    "Abuja": "North-Central", "Port Harcourt": "South-South"
}
df["Region"] = df["State"].map(state_to_region)

```

### **C. Categorizing Age Ranges**

Creating age groups:

```python
python
Copy
bins = [0, 18, 25, 35, 45, 55, 65, 100]
labels = ["Under 18", "18-25", "26-35", "36-45", "46-55", "56-65", "65+"]
df["Age Range"] = pd.cut(df["Age"], bins=bins, labels=labels, right=False)

```

### **D. Customer Segmentation Based on Account Behavior**

Classifying customers based on their average monthly balance:

```python
python
Copy
df["Value Segment"] = pd.qcut(df["Average Monthly Balance"], q=3, labels=["Low", "Medium", "High"])

```

---

## **4. Storing and Querying Data with SQLite & SQL**

### **A. Uploading Data to SQLite**

```python
python
Copy
import sqlite3

conn = sqlite3.connect("bank_data.db")
df.to_sql("customers", conn, if_exists="replace", index=False)
conn.close()

```

### **B. SQL Queries for Analysis**

**Total Number of Customers**

```sql
sql
Copy
SELECT COUNT(DISTINCT "Customer ID") AS Total_Customers FROM customers;

```

**Average Transaction Frequency**

```sql
sql
Copy
SELECT AVG("Transaction Frequency (per month)") AS Avg_Transaction_Frequency FROM customers;

```

**Most Preferred Transaction Channel**

```sql
sql
Copy
SELECT "Preferred Channels", COUNT(*) AS Count
FROM customers
GROUP BY "Preferred Channels"
ORDER BY Count DESC
LIMIT 1;

```

### **C. Segmentation Analysis in SQL**

**Preferred Channels by Age Range**

```sql
sql
Copy
SELECT "Age Range", "Preferred Channels", COUNT(*) AS Count
FROM customers
GROUP BY "Age Range"
ORDER BY Count DESC;

```

**Transaction Frequency by Gender**

```sql
sql
Copy
SELECT "Gender", AVG("Transaction Frequency (per month)") AS Avg_Frequency
FROM customers
GROUP BY "Gender"
ORDER BY Avg_Frequency DESC;

```

**Loan Status by Region**

```sql
sql
Copy
SELECT "Region", "Loan Status"
FROM customers
GROUP BY "Region";

```

### **D. Rule-Based Recommendations in SQL**

```sql
sql
Copy
SELECT "Customer ID",
       "Preferred Channels",
       "Age Range",
       "Loan Status",
       "Value Segment",
       CASE
           WHEN "Preferred Channels" = 'ATM' AND "Value Segment" = 'Low' AND "Loan Status" = 'No Loans' THEN 'Basic Savings Account + ATM Fee Discounts'
           WHEN "Preferred Channels" = 'Mobile App' AND "Value Segment" = 'High' THEN 'Premium Banking + Investment Opportunities'
           WHEN "Age Range" = '26-35' AND "Loan Status" = 'Active' THEN 'Loan Top-Up or Refinancing Offer'
           WHEN "Region" = 'South-West' AND "Value Segment" = 'High' AND "Preferred Channels" = 'Online Banking' THEN 'Wealth Management Services'
           WHEN "Age Range" = '56-65' AND "Value Segment" = 'Low' THEN 'Pension & Retirement Plans'
           ELSE 'General Banking Products'
       END AS "Recommended Product"
FROM customers;

```

---

## **5. Machine Learning-Based Segmentation in Python**

### **A. One-Hot Encoding for Categorical Features**

```python
python
Copy
from sklearn.preprocessing import OneHotEncoder
import pandas as pd

df = pd.read_sql("SELECT * FROM customers", conn)
features = ["Preferred Channels", "Age Range", "Loan Status", "Value Segment"]

encoder = OneHotEncoder(sparse=False, drop="first")
encoded_features = encoder.fit_transform(df[features])
feature_names = encoder.get_feature_names_out(features)
df_encoded = pd.DataFrame(encoded_features, columns=feature_names)
df_encoded["Customer ID"] = df["Customer ID"]

```

### **B. K-Means Clustering for Customer Segmentation**

```python
python
Copy
from sklearn.cluster import KMeans

kmeans = KMeans(n_clusters=5, random_state=42)
df_encoded["Cluster"] = kmeans.fit_predict(df_encoded.drop(columns=["Customer ID"]))

df["Customer Segment"] = df_encoded["Cluster"]

```

### **C. Data-Driven Product Recommendations**

```python
python
Copy
recommended_products = df.groupby("Customer Segment")["Most Used Product"].agg(lambda x: x.value_counts().idxmax())
df["Recommended Product"] = df["Customer Segment"].map(recommended_products)

```

---

## **6. Conclusion**

This report details the methods used to explore, clean, and analyze banking customer data. Using **Python (Pandas, Matplotlib, and Seaborn)** for data visualization and **SQL queries** for structured analysis, we identified key insights on customer behaviors. Finally, **machine learning-based segmentation** (K-Means) provided a more dynamic approach to categorizing customers, enhancing product recommendations and business strategy development.

### **Key Takeaways:**

1. **Data exploration** revealed transaction trends, preferred channels, and customer demographics.
2. **Data cleaning** ensured accurate analysis by handling missing values and duplicates.
3. **SQL-based queries** allowed structured retrieval of customer insights.
4. **Clustering models (K-Means)** enabled segmentation for personalized banking recommendations.

By leveraging these analytical techniques, banks can **optimize their services, improve customer satisfaction, and drive business growth**.