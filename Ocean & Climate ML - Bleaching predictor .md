import pandas as pd 
#importing pandas library

df = pd.read_csv("cleaned_ocean_climate_data.csv")
display(df.head())# showing first 5 rows of dataset

Data Cleaning - updating formats, creating one new column and creating Bleaching score for easier analysis. 

df['Date'] = pd.to_datetime(df['Date'])
#convering date to datetime format

df['Year'] = df['Date'].dt.year
#creating Year as its own column

severity_map = {'None': 0, 'Low': 1, 'Medium': 2, 'High':3}
df['Bleaching_Score'] = df['Bleaching_Severity'].map(severity_map)
#converting Bleaching severity to numeric values 

df['Marine_Heatwave'] = df['Marine_Heatwave'].astype(int)
#converting marine_Heatwave to boolean/int

df = df.dropna(subset=['SST_C', 'pH_Level', 'Species_Observed', 'Bleaching_Score'])
#dropping missing values

df.head()

Bar Chart - Average species observed per reef

import seaborn as sns
import matplotlib.pyplot as plt

# importing seaborn and matplotlib

avg_species = (df.groupby("Location")["Species_Observed"].mean().sort_values(ascending=False)
)# grouping by location while using average species observed

#barchart creation
plt.figure(figsize=(10, 6))
sns.barplot(x=avg_species.values, y=avg_species.index, palette="Blues_d")
plt.title("Average Species Observed per Reef Location")
plt.xlabel("Avg Species Observed")
plt.ylabel("Reef Location")
plt.tight_layout()
plt.show()

Train Test Split

from sklearn.model_selection import train_test_split

# importing train/test from sklearn

features = ["SST_C", "pH_Level", "Species_Observed", "Marine_Heatwave"]
X = df[features]
y = df["Bleaching_Score"]
# Selecting features + target

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)
