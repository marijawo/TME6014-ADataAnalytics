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
# Create df_model copy and drop identifier
df_model = df_clean.copy()
df_model = df_model.drop('customerID', axis=1)

# Label Encoding for binary features (including Churn)
binary_cols_to_encode = ['Partner', 'Dependents', 'PhoneService', 'MultipleLines',
               'OnlineSecurity', 'OnlineBackup', 'DeviceProtection',
               'TechSupport', 'StreamingTV', 'StreamingMovies',
               'PaperlessBilling', 'Churn']

le = LabelEncoder()
for col in binary_cols_to_encode:
    df_model[col] = le.fit_transform(df_model[col])

# Encode 'gender' separately for clarity (0=Female, 1=Male)
df_model['gender'] = df_model['gender'].replace({'Female': 0, 'Male': 1})

# One-Hot Encoding for Nominal features
nominal_cols = ['InternetService', 'Contract', 'PaymentMethod']
df_model = pd.get_dummies(df_model, columns=nominal_cols, drop_first=True)
```

---

### 3.2 Feature Scaling / Normalization

Standard scaling (z-score normalization) was applied to the main numerical features to normalize their range and prevent any feature from disproportionately influencing model weights.

#### Code Sample: Feature Scaling

```python
from sklearn.preprocessing import StandardScaler

numerical_cols_to_scale = ['tenure', 'MonthlyCharges', 'TotalCharges']
scaler = StandardScaler()

# Apply standard scaling
df_model[numerical_cols_to_scale] = scaler.fit_transform(df_model[numerical_cols_to_scale])
```

---

## 4. Summary of Deliverables

The final deliverable of **Phase 2** is a fully preprocessed, numerical, and scaled dataset (`df_model`), prepared and validated for input into machine-learning models.
