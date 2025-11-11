# Phase Objective

The goal of this phase is to gather the required data for your project and transform it into a clean, usable format ready for analysis.

---

## Tasks

### 1. Data Collection
- Gather data from the chosen sources (internal datasets, open data portals, APIs, sensors, web scraping, etc.).
- Document each source clearly:
  - URL  
  - Access method  
  - Dataset description  
  - Retrieval date  

---

### 2. Data Preprocessing

#### **Data Acquisition Validation**
- Confirm dataset integrity (readable, correct format, no corrupted records)

#### **Missing Data Handling**
- Identify missing values  
- Apply a reasonable handling method (remove or impute)

#### **Duplicate Removal**
- Detect and remove duplicate rows or records

#### **Outlier Detection & Treatment**
- Identify unusual values and decide whether to keep, fix, or remove them

#### **Data Type Correction**
- Ensure all features have correct data types (numeric, categorical, dates, etc.)

#### **Formatting & Standardization**
- Clean up inconsistent formatting (capitalization, date formats, units, etc.)

#### **Categorical Data Encoding**
- Convert categorical features into numerical formats suitable for analysis

#### **Feature Scaling / Normalization**
- Apply appropriate scaling if numeric features are on different ranges

#### **Data Integration (if multiple sources used)**
- Merge datasets and handle alignment inconsistencies

---

### 3. Documentation
- Record any problems encountered (e.g., limited access, inconsistent formats)
- Provide a detailed list of preprocessing steps, including justification for any decisions made

---

## Deliverables

A packaged dataset including:

- **Raw data** (as originally collected)  
- **Cleaned/processed dataset**  
- **Source code**  
- **Preprocessing documentation** (1–2 pages or well-structured README), including:
  - Data source descriptions  
  - Cleaning and transformation steps  
  - Summary of challenges and solutions
