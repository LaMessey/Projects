#Importing JSON File from kaggle 
import json
import pandas as pd

# Load JSON
with open("ikea_sample_file.json") as f:
    data = json.load(f)

#Flattening the structure
df = pd.json_normalize(data)

#Preview data and reviewing data info
df.head(3)
df.info()

#Checking data types
for col in df.columns:
    print(f"{col}: {df[col].apply(type).unique()}")

#Fixing data types for cleanliness
#Product_Price column - Removing any dollar signs and commas and convering to strings into real numbers (floats) for calculations
df['product_price'] = df['product_price'].replace('[\$,]', '', regex=True).astype(float)

#Average_rating - Converting column to numbers and setting rows as blank (NaN) to avoid issues. 
df['average_rating'] = pd.to_numeric(df['average_rating'], errors='coerce')

#Reviews_count - Converiting from string to number
df['reviews_count'] = pd.to_numeric(df['reviews_count'], errors='coerce')

#OverallRemoving leading/trailing spaces, making lowercase.
df.columns = df.columns.str.strip().str.lower().str.replace(' ', '_')

#Previewing to confirm updates
df.info()
df.head(5)


#Cleaning column names for ease of use
import re

# 1. Extract the product name (before the description)
def extract_product_name(title):
    # If there is a slash, take everything before the first English word
    if '/' in title:
        return title.split(' ', 1)[0] + ' / ' + title.split(' / ')[-1].split(' ')[0]
    else:
        return title.split(' ')[0]

# 2. Extract the product type (after name or slash, before dash)
def extract_product_type(title):
    if '/' in title:
        return title.split('/')[-1].strip().split('-')[0].strip()
    else:
        return ' '.join(title.split()[1:]).split('-')[0].strip()

# Apply functions to create new columns
df['product_name'] = df['product_title'].apply(extract_product_name)
df['product_type'] = df['product_title'].apply(extract_product_type)

# Optional: Move product_name and product_type to the front
cols = df.columns.tolist()
new_order = ['product_name', 'product_type'] + [col for col in cols if col not in ['product_name', 'product_type']]
df = df[new_order]

# Preview result
df[['product_title', 'product_name', 'product_type']].head(10)

#Reordering columns so the key ones come first: name, type, and original title
cols = df.columns.tolist()

#Moving product_name, product_type, and product_title at the front
new_order = ['product_name', 'product_type', 'product_title'] + [col for col in cols if col not in ['product_name', 'product_type', 'product_title']]

#Reordering the DataFrame
df = df[new_order]

#Reviews_count - currently a float changing to interger removing any .0 as it should always be a whole number 
df['reviews_count'] = df['reviews_count'].dropna().astype('Int64')

#Raw_product_details - full of HTML so removing HTML and keeping text using Beautiful Soup
#importing beautiful soup
from bs4 import BeautifulSoup

#Using BeautifulSoup to remove HTML tags
df['product_details_clean'] = df['raw_product_details'].apply(lambda x: BeautifulSoup(str(x), "html.parser").get_text(strip=True))

#Breadcrumbs - splitting into categories so split breadcrumb path into parts
breadcrumb_parts = df['breadcrumbs'].str.split('/', expand=True)

# Renaming and joining with original DataFrame
breadcrumb_parts.columns = [f'category_level_{i+1}' for i in range(breadcrumb_parts.shape[1])]
df = df.join(breadcrumb_parts)

#Creating a missing data indicator for ease of filtering in tableau
df = df.dropna(subset=['average_rating', 'reviews_count'])

#Creating in Price tiering
df['price_tier'] = pd.qcut(df['product_price'], 3, labels=['Low', 'Medium', 'High'])

#Creating review volumn tier
df['review_volume'] = pd.cut(df['reviews_count'], bins=[-1, 0, 20, 9999], labels=['None', 'Few', 'Popular'])

#Previewing new dataframe
df.head(3)

#Exporting to CSV for manipulation in Tableau
df.to_csv("ikea_cleaned.csv", index=False)
