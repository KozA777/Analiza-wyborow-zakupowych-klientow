# Customer Purchasing Habits Analysis

## Project Overview

This project aims to analyze customer purchasing habits to plan future sales and marketing strategies for the company. The analysis evaluates the impact of factors such as the number of discounts, customer gender, and annual income on the number of purchases made by customers.

## Data Description

The dataset used in this analysis includes information on customer purchasing behaviors, demographic details, and other relevant features:

* Age: Customer's age (in years)
* Gender: Customer's gender (0: Male, 1: Female)
* Annual Income: Customer's annual income (in dollars)
* Number of Purchases: Total number of purchases made by the customer (within one year)
* Product Category: Category of the purchased product (0: Electronics, 1: Clothing, 2: Household, 3: Beauty, 4: Sports)
* Time Spent on Website: Time spent by the customer on the website (in minutes, cumulatively over a year)
* Loyalty Program: Whether the customer is a member of the loyalty program (0: No, 1: Yes)
* Discounts Availed: Number of discounts availed by the customer (range: 0-5)
* Purchase Status (target variable): Probability of a purchase being made (0: No, 1: Yes)

## Analysis Goals

The primary objective of this analysis is to understand customer purchasing behaviors and identify factors that significantly influence their purchasing decisions.

## Data Preparation

1. Import necessary libraries:
```
from google.cloud import bigquery
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
```
2. Load the dataset:
```
df = pd.read_csv('/content/drive/MyDrive/Project1_Customer_preferences/customer_purchase_data.csv', sep=',')
```
3. Check the first 5 rows of the dataset:
```
df.head()
```
4. Descriptive statistics:
```
df.describe()
```
5. Check for duplicate rows:
```
df[df.duplicated()]
```
6. Remove duplicate rows:
```
df_cleaned = df.drop_duplicates()
```
7. Check for missing values:
```
df_cleaned.isnull().any()
```
8. Add a Client_ID column and move it to the beginning:
```
df_cleaned['Client_ID'] = df_cleaned.index
df_cleaned = df_cleaned[['Client_ID'] + [col for col in df_cleaned.columns if col != 'Client_ID']]
```
9. Split the dataset into two parts for normalization:
```
df_client = df_cleaned[['Client_ID', 'Age', 'Gender', 'AnnualIncome', 'LoyaltyProgram']]
df_purchases = df_cleaned[['Client_ID', 'NumberOfPurchases', 'ProductCategory', 'DiscountsAvailed', 'PurchaseStatus', 'TimeSpentOnWebsite']]
```
10. Create Google Cloud account, set up a Service Account, and generate a new JSON key for BigQuery access

11. Load data into Google Cloud tables:
```
client = bigquery.Client.from_service_account_json('/content/drive/MyDrive/Project1_Customer_preferences/nodal-subset-428419-s3-5eb70d85c5ba.json')
table_id_client = 'nodal-subset-428419-s3.dataset1.client'
table_id_purchases = 'nodal-subset-428419-s3.dataset1.purchases'
job_config_client = bigquery.LoadJobConfig(write_disposition=bigquery.WriteDisposition.WRITE_TRUNCATE)
job_config_purchases = bigquery.LoadJobConfig(write_disposition=bigquery.WriteDisposition.WRITE_TRUNCATE)
job_client = client.load_table_from_dataframe(df_client, table_id_client, job_config=job_config_client)
job_client.result()
job_purchases = client.load_table_from_dataframe(df_purchases, table_id_purchases, job_config=job_config_purchases)
job_purchases.result()
```
# Data Analysis
1. Correlation check:
```
sns.heatmap(df_cleaned.corr(), annot=True)
```
2. Impact of Loyalty Program on the number of purchases:
```
query = """
SELECT sum(p.NumberOfPurchases) as Suma_zamowien, c.LoyaltyProgram
FROM `nodal-subset-428419-s3.dataset1.client` c
JOIN `nodal-subset-428419-s3.dataset1.purchases` p
ON c.Client_ID = p.Client_ID
GROUP BY c.LoyaltyProgram
"""
df1 = client.query(query).to_dataframe()
df1.plot(kind='bar', x='LoyaltyProgram', y='Suma_zamowien')
plt.title('Number of purchases by loyalty program membership')
plt.xlabel('Loyalty Program')
plt.ylabel('Total Purchases')
plt.show()
```
3. Purchase choices by gender:
```
query = """
SELECT c.Gender, p.ProductCategory, SUM(p.NumberOfPurchases) AS TotalPurchases
FROM `nodal-subset-428419-s3.dataset1.client` c
JOIN `nodal-subset-428419-s3.dataset1.purchases` p
ON c.Client_ID = p.Client_ID
GROUP BY c.Gender, p.ProductCategory
ORDER BY c.Gender, TotalPurchases DESC
"""
df3 = client.query(query).to_dataframe()
plt.figure(figsize=(10, 6))
sns.barplot(x='ProductCategory', y='TotalPurchases', hue='Gender', data=df3)
plt.title('Total purchases by category and gender')
plt.xlabel('Product Category')
plt.ylabel('Total Purchases')
plt.show()
```
4. Impact of time spent on website on number of purchases:
```
query = """
SELECT ROUND(p.TimeSpentOnWebsite) as RoundedTimeSpend, SUM(p.NumberOfPurchases) AS NumberOfPurchases
FROM `nodal-subset-428419-s3.dataset1.purchases` p
GROUP BY RoundedTimeSpend
ORDER BY RoundedTimeSpend DESC, NumberOfPurchases DESC
"""
df5 = client.query(query).to_dataframe()
df5.plot(kind='scatter', x='RoundedTimeSpend', y='NumberOfPurchases')
plt.title('Scatter plot: Time spent on website vs. Number of purchases')
plt.xlabel('Time spent on website')
plt.ylabel('Number of purchases')
plt.show()
```
5. Impact of discounts on purchases by gender:
```
query = """
SELECT c.Gender, p.DiscountsAvailed, count(p.DiscountsAvailed) as DiscountAvailedCount
FROM `nodal-subset-428419-s3.dataset1.client` c
JOIN `nodal-subset-428419-s3.dataset1.purchases` p
ON c.Client_ID = p.Client_ID
GROUP BY c.Gender, p.DiscountsAvailed
ORDER BY p.DiscountsAvailed DESC
"""
df6 = client.query(query).to_dataframe()
plt.figure(figsize=(10, 6))
sns.barplot(x='DiscountsAvailed', y='DiscountAvailedCount', hue='Gender', data=df6)
plt.title('Number of purchases by number of discounts availed by gender')
plt.xlabel('Number of discounts availed')
plt.ylabel('Number of customers availing discounts')
plt.show()
```
6. Total purchases and income by gender:
```
query = """
SELECT c.Gender, SUM(c.AnnualIncome) as TotalIncome, SUM(p.NumberOfPurchases) as TotalPurchases
FROM `nodal-subset-428419-s3.dataset1.client` c
JOIN `nodal-subset-428419-s3.dataset1.purchases` p
ON c.Client_ID = p.Client_ID
GROUP BY c.Gender
"""
df9 = client.query(query).to_dataframe()
```
# Conclusions

1. The total number of orders is twice as high for those not in the loyalty program. However, this is indirectly correlated with twice the number of people not in the program. Membership in the loyalty program did not directly increase the number of purchased products.

2. The number of customers of both genders is very close (women constitute 50.2%, and men 49.8%). Women more often bought electronics and sports products, while men more often purchased household items. In other categories (clothing and beauty), the differences were marginal.

3. Time spent on the website did not affect the number of purchases.

4. Gender did not significantly influence discount usage. The number of customers using a given number of discounts was very similar for both genders. A larger difference was observed in those not using discounts, where women constituted 57.5%, 15% more than men.

5. The total annual income of women was slightly (1.4%) higher than that of men. A similar difference occurred in the number of purchases, with women ordering 1% more products than men.

6. Correlation analysis showed that none of the parameters significantly influenced others.

7. The observed uniformity of all analyzed variables likely results from the synthetic nature of the data, affecting the credibility of the conclusions and limiting their application in real sales and marketing strategies.

The synthetic nature of the data means that identified trends may result from artificial constructs rather than real customer behaviors. In reality, data is usually more diverse and unpredictable, allowing for a deeper analysis and better understanding of complex consumer behavior patterns.

Despite these limitations, this analysis serves as an exercise in analytical techniques and may be useful as a basis for more advanced and detailed research.

## Sources

The project was based on a dataset from the Kaggle website.
(https://www.kaggle.com/code/abdelruhmanessam/predict-customer-purchase-behavior-dataset)

## Authors
[Anna Kożuch]

