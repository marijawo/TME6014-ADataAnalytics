# Phase 2: Data Preprocessing and Cleaning Report

## 1. Data Source and Collection

The objective of this phase was to prepare a clean, analysis-ready dataset for predicting **Telco Customer Churn**.  
The data used is a public-domain dataset, commonly used in data-science tasks involving binary classification.

### Data Source Details

| Field | Description |
| :---- | :----------- |
| **Dataset** | Telco Customer Churn Dataset |
| **Size** | 7,043 customer records |
| **Features** | 21 features (demographics, services, account information) |
| **Target Variable** | `Churn` (Yes/No) |
| **Access Method** | Loaded from a local CSV file |

### Code Sample: Data Loading

The initial data-loading step established the baseline raw dataset.

```python
import pandas as pd
from sklearn.preprocessing import LabelEncoder, StandardScaler, OneHotEncoder

# Load the raw dataset
df = pd.read_csv('datasets/WA_Fn-UseC_-Telco-Customer-Churn.csv')

# Initial inspection (checking shape and data types)
print(f"Initial Dataset Shape: {df.shape}")
print(df.dtypes)
```

---

## 2. Data Preprocessing and Cleaning Steps (Task 2)

### 2.1 Missing Data Handling and Data Type Correction

A data-integrity check revealed that the `TotalCharges` column was incorrectly identified as an `object` (string) type, which typically indicates hidden, non-numeric values.

| Issue | Solution | Justification |
| :---- | :-------- | :------------- |
| **Hidden Missing Values** | 11 records in `TotalCharges` contained an empty string `' '`. These were coerced to `NaN` during type conversion. | The initial check (`df.isnull().sum()`) missed these string values. |
| **Imputation** | The 11 `NaN` values were imputed with `0.0`. | All missing records corresponded to customers with `tenure = 0` (new customers). They logically have 0 total charges. |
| **Data Type Correction** | `TotalCharges` was successfully converted from `object` to `float64`. | Necessary to enable mathematical operations and feature scaling. |

#### Code Sample: Imputation and Type Conversion

```python
# Create a copy for cleaning
df_clean = df.copy()

# Convert 'TotalCharges' to numeric, replacing ' ' with NaN
df_clean['TotalCharges'] = pd.to_numeric(df_clean['TotalCharges'], errors='coerce')

# Impute NaN values with 0 (for customers with tenure = 0)
df_clean['TotalCharges'].fillna(0, inplace=True)

print(f"Total missing values after imputation: {df_clean.isnull().sum().sum()}")
print(f"TotalCharges new type: {df_clean['TotalCharges'].dtype}")
```

---

### 2.2 Duplicate and Outlier Treatment

| Step | Outcome | Justification |
| :---- | :------- | :------------- |
| **Duplicate Removal** | No exact duplicate rows were found and therefore none were removed. | Ensures that the dataset does not contain redundant information. |
| **Outlier Treatment** | Outliers in numerical features (`tenure`, `MonthlyCharges`, `TotalCharges`) were retained. | The high values represent valid customer behavior (e.g., long-term, high-spending customers). Removing them could introduce bias and reduce the model's ability to predict churn in these valuable segments. |

---

### 2.3 Formatting and Standardization

Inconsistent text labels across multiple service-related columns were standardized.

```python
# Columns that contained inconsistent 'No service' labels
columns_to_standardize = ['MultipleLines', 'OnlineSecurity', 'OnlineBackup',
                          'DeviceProtection', 'TechSupport', 'StreamingTV', 'StreamingMovies']

for col in columns_to_standardize:
    # Replace both 'No internet service' and 'No phone service' with 'No'
    df_clean[col] = df_clean[col].replace({'No internet service': 'No', 'No phone service': 'No'})

print("Categorical values standardized for encoding.")
```

---

## 3. Preprocessing for Modeling (Feature Transformation)

The final steps involve converting all non-numerical data into a format suitable for machine-learning algorithms.

### 3.1 Categorical Data Encoding

We split the categorical features based on the number of unique values for appropriate encoding:

- **Label Encoding:** Applied to the target variable (`Churn`) and other binary columns (`Yes/No`, `Male/Female`).  
- **One-Hot Encoding:** Applied to nominal, multi-category features (`InternetService`, `Contract`, `PaymentMethod`).

#### Code Sample: Categorical Encoding

```python
# --- Defensive fix for missing 'churn' key ---

# Identify the exact column name that represents churn
print("Available columns:", df_clean.columns.tolist())

# Find the actual churn column (case-insensitive match)
churn_col = None
for c in df_clean.columns:
    if c.strip().lower() == "churn":
        churn_col = c
        break

if churn_col is None:
    raise KeyError("No 'Churn' column found in dataset. Check dataset headers.")

# Standardize the column name
df_clean.rename(columns={churn_col: "churn"}, inplace=True)

# Now proceed as before
target_col = "churn"
y = df_clean[target_col].map({"yes": 1, "no": 0})
print("Target distribution:")
print(y.value_counts())

# Drop non-feature columns and target
feature_df = df_clean.drop(columns=["customerID", target_col])

# One-hot encode
X = pd.get_dummies(feature_df, drop_first=True)

print("\nFeature matrix shape after encoding:", X.shape)
X.head()
```

---

### 3.2 Feature Scaling / Normalization

Standard scaling (z-score normalization) was applied to the main numerical features to normalize their range and prevent any feature from disproportionately influencing model weights.

#### Code Sample: Feature Scaling

```python
# Identify numeric feature columns
numeric_feature_cols = X.select_dtypes(include=[np.number]).columns.tolist()
print("Number of numeric feature columns:", len(numeric_feature_cols))

# Initialize the scaler
scaler = StandardScaler()

# Fit and transform numeric columns
X_scaled_array = scaler.fit_transform(X[numeric_feature_cols])

# Create a DataFrame with scaled values
X_scaled = pd.DataFrame(X_scaled_array, columns=numeric_feature_cols, index=X.index)

# Final model-ready DataFrame: target + scaled features
model_df = pd.concat([y.rename("churn"), X_scaled], axis=1)
print("Model-ready dataframe shape:", model_df.shape)
model_df.head()

```

---


## 🧩 Summary of Challenges and Solutions

| **Challenge** | **Description** | **Solution Implemented** |
|----------------|-----------------|---------------------------|
| **Inconsistent Column Names** | The churn label was not recognized due to capitalization ("Churn" vs "churn") and whitespace variations. | Added a defensive case-insensitive column detection that standardizes "Churn" to "churn". |
| **Missing / Invalid Numeric Values** | The `TotalCharges` column contained blank strings, leading to conversion errors when casting to numeric. | Used `pd.to_numeric(errors='coerce')` to turn blanks into NaN, then imputed missing values with the median. |
| **Duplicate Records** | Potential duplicate customer records could bias analysis. | Used `df.duplicated()` to detect and drop duplicates safely. |
| **Outliers in Numeric Fields** | Extreme values in `MonthlyCharges` and `TotalCharges` distorted the distribution. | Applied IQR-based filtering to identify and remove rows with outliers. |
| **Mixed Data Types** | Columns like `SeniorCitizen` were numeric but conceptually categorical (0/1). | Explicitly cast to integer and documented as categorical. |
| **Categorical Encoding** | Non-numeric features (like `InternetService` or `Contract`) were incompatible with ML models. | Applied `pd.get_dummies()` for one-hot encoding, ensuring `drop_first=True` to avoid multicollinearity. |
| **Scaling Differences** | Features had varying numeric ranges (`tenure`, `MonthlyCharges`, etc.). | Standardized with `StandardScaler` for consistent model training. |
| **Formatting & Case Inconsistencies** | Categories like "Yes", " yes", and "YES" existed. | Normalized all string columns to lowercase and stripped whitespace. |


### > Note the cleaned dataset is also created from the last code block under ## 11. Saving Raw, Cleaned, and Model-Ready Datasets


## End of Report

