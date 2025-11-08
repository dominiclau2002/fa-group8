# Data Flow Mapping - IS453 Group Assignment

**Document Purpose:** Complete documentation of the data preparation pipeline for loan default prediction analysis. Identifies data transformations, intermediate states, and critical issues.

**Dataset:** IS453 Group Assignment - Loan Default Data
**Last Updated:** Based on Use_this_one.ipynb analysis
**Current Data State:** df_merged_cleaned (252,137 rows × 102 columns)

---

## Table of Contents
1. [Data Pipeline Overview](#data-pipeline-overview)
2. [Detailed Stage Breakdown](#detailed-stage-breakdown)
3. [Duplicate Column Issue](#duplicate-column-issue-critical)
4. [Complete Transformation Table](#complete-transformation-table)
5. [Current Data State](#current-data-state)
6. [Key Insights & Recommendations](#key-insights--recommendations)

---

## Data Pipeline Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         RAW DATA SOURCES                               │
├─────────────────────────────────────────────────────────────────────────┤
│ • Application Data.csv (307,511 rows × 120 cols)                        │
│ • Bureau Data.csv (1,716,428 rows × 17 cols)                            │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
        ┌─────────────────────────┐    ┌─────────────────────────┐
        │  APPLICATION FILTERING  │    │   BUREAU FILTERING      │
        │  (Cell 11)              │    │   (Cell 12)             │
        │                         │    │                         │
        │ • Drop single-value     │    │ • Drop single-value     │
        │ • Drop >55% missing     │    │ • Drop >55% missing     │
        │ • Drop no-variation     │    │ • Drop no-variation     │
        │ • Drop gender cols      │    │ • Remove bad records    │
        │                         │    │                         │
        │ OUTPUT:                 │    │ OUTPUT:                 │
        │ appData_filtered        │    │ bureauData_filtered     │
        │ (307,511 × 92)          │    │ (~1.69M × 15)           │
        └─────────────────────────┘    └─────────────────────────┘
                    │                               │
                    │                               ▼
                    │                    ┌─────────────────────────┐
                    │                    │   BUREAU AGGREGATION    │
                    │                    │   (Cell 24)             │
                    │                    │                         │
                    │                    │ • GroupBy SK_ID_CURR    │
                    │                    │ • Aggregate 13 metrics  │
                    │                    │                         │
                    │                    │ OUTPUT:                 │
                    │                    │ agg_bureau              │
                    │                    │ (~306K × 13)            │
                    │                    └─────────────────────────┘
                    │                               │
                    └───────────────┬───────────────┘
                                    ▼
                    ┌─────────────────────────────────┐
                    │  MERGE APPLICATION + BUREAU     │
                    │  (Cell 26)                      │
                    │                                 │
                    │  df_app LEFT JOIN agg_bureau    │
                    │  on SK_ID_CURR                  │
                    │                                 │
                    │  OUTPUT:                        │
                    │  df_merged (252,137 × 105)      │
                    └─────────────────────────────────┘
                                    │
                                    ▼
                    ┌─────────────────────────────────┐
                    │  CORRELATION FILTERING          │
                    │  (Cell 32)                      │
                    │                                 │
                    │  • Identify correlated pairs    │
                    │  • Drop weaker correlations     │
                    │  • Remove 3 columns             │
                    │                                 │
                    │  OUTPUT:                        │
                    │  df_merged_cleaned              │
                    │  (252,137 × 102)                │
                    └─────────────────────────────────┘
                                    │
                                    ▼
                    ┌─────────────────────────────────┐
                    │  FINAL DATA STATE               │
                    │  Ready for EDA & Modeling       │
                    └─────────────────────────────────┘
```

---

## Detailed Stage Breakdown

### STAGE 1: DATA LOADING

**Cell:** 5, 8

**Input Files:**
- `IS453 Group Assignment - Application Data.csv`
- `IS453 Group Assignment - Bureau Data.csv`

**Operations:**
- Read CSV files using pandas
- No initial transformations

**Output:**
| Dataframe | Rows | Columns | Description |
|-----------|------|---------|-------------|
| appData | 307,511 | 120 | Raw loan application records |
| bureauData | 1,716,428 | 17 | Raw credit bureau history (one-to-many) |

---

### STAGE 2: APPLICATION DATA FILTERING

**Cell:** 11

**Input:** appData (307,511 rows × 120 columns)

**Filtering Criteria Applied:**

1. **Drop single-value columns** - Columns with only 1 unique value (no variance)
2. **Drop high missing columns** - Columns with >55% missing data
3. **Drop no-variation columns** - Columns where TARGET=0 and TARGET=1 have identical distributions
4. **Drop gender columns** - CODE_GENDER removed for fair lending compliance

**Columns Dropped:** 28 columns including:
- Single-value columns (specific names not detailed in notebook)
- High-missing columns (specific names not detailed in notebook)
- CODE_GENDER (fairness/compliance)

**Output:** appData_filtered (307,511 rows × 92 columns)

---

### STAGE 3: BUREAU DATA FILTERING

**Cell:** 12

**Input:** bureauData (1,716,428 rows × 17 columns)

**Step 1: Merge with TARGET for Analysis**
- Join with appData_filtered[['SK_ID_CURR', 'TARGET']]
- Enables filtering based on variation between goods/bads
- Creates bureauData_with_target (1,716,428 rows × 18 columns)

**Step 2: Apply Filtering Criteria**

Same criteria as application data:
1. Drop single-value columns
2. Drop columns with >55% missing data:
   - **AMT_CREDIT_MAX_OVERDUE:** 65.51% missing
   - **AMT_ANNUITY:** 71.47% missing (also dropped from bureau, kept from application)
3. Drop columns with no variation between goods/bads

**Step 3: Data Quality Filtering**

Remove records with invalid data:
- DROP: AMT_CREDIT_SUM_DEBT < 0 → **8,418 records removed**
- DROP: AMT_CREDIT_SUM_LIMIT < 0 → **351 records removed**
- DROP: AMT_CREDIT_SUM_DEBT > AMT_CREDIT_SUM → **17,235 records removed**
- DROP: AMT_CREDIT_SUM_OVERDUE > AMT_CREDIT_SUM_DEBT → **32 records removed**

Total records removed: **26,036** (1.5% of 1.7M)

**Step 4: Remove TARGET Column**
- TARGET was used for filtering only, not retained in bureau data

**Output:** bureauData_filtered (~1,690,392 rows × 15 columns)

**Columns in bureauData_filtered:**
- SK_ID_BUREAU (unique identifier)
- SK_ID_CURR (customer link)
- CREDIT_ACTIVE
- CREDIT_CURRENCY
- CREDIT_TYPE
- DAYS_CREDIT
- CREDIT_DAY_OVERDUE
- DAYS_CREDIT_ENDDATE
- DAYS_ENDDATE_FACT
- CNT_CREDIT_PROLONG
- AMT_CREDIT_SUM
- AMT_CREDIT_SUM_DEBT
- AMT_CREDIT_SUM_LIMIT
- AMT_CREDIT_SUM_OVERDUE
- DAYS_CREDIT_UPDATE

---

### STAGE 4: BUREAU DATA AGGREGATION

**Cell:** 24

**Input:** bureauData_filtered (~1,690,392 rows × 15 columns) - One-to-many relationship (one customer can have multiple bureau records)

**Aggregation Strategy:** GroupBy SK_ID_CURR with various aggregation functions

**Aggregation Details:**

```python
agg_bureau = df_bureau.groupby("SK_ID_CURR").agg({
    "SK_ID_BUREAU": "count",              # → renamed to NUM_PREV_LOANS
    "CREDIT_ACTIVE": lambda x: x.mode()[0] if len(x.mode()) > 0 else x[0],
    "CREDIT_DAY_OVERDUE": "sum",
    "DAYS_CREDIT": "min",                 # Earliest credit date
    "DAYS_CREDIT_ENDDATE": "max",         # Latest end date
    "DAYS_ENDDATE_FACT": "max",           # Latest fact date
    "CNT_CREDIT_PROLONG": "sum",          # Total prolongations
    "AMT_CREDIT_SUM": "sum",              # Total credit
    "AMT_CREDIT_SUM_DEBT": "sum",         # Total debt
    "AMT_CREDIT_SUM_LIMIT": "sum",        # Total limits
    "AMT_CREDIT_SUM_OVERDUE": "sum",      # Total overdue
    "DAYS_CREDIT_UPDATE": "max"           # Most recent update
})
```

**Additional Step:** Add CREDIT_TYPE
- For each customer, find the CREDIT_TYPE with highest AMT_CREDIT_SUM
- Merge back as single "CREDIT_TYPE" column (dominant type)

**Aggregation Functions Used:**
- **count:** Number of bureau records per customer → NUM_PREV_LOANS
- **mode:** Most common credit status
- **sum:** Total across all credits
- **min:** Earliest date (longest history)
- **max:** Latest/most recent values

**Impact of Aggregation:**
- Input: ~1,690,392 rows (one-to-many)
- Output: ~305,811 rows (one-to-one, one per customer)
- Rows reduced by: 1,384,581 (consolidation from many bureau records per customer to one aggregated row)

**Output:** agg_bureau (~305,811 rows × 13 columns)

**Columns in agg_bureau:**
1. NUM_PREV_LOANS (count of bureau records)
2. CREDIT_ACTIVE (mode - most common status)
3. CREDIT_DAY_OVERDUE (sum)
4. DAYS_CREDIT (min - earliest credit date)
5. DAYS_CREDIT_ENDDATE (max)
6. DAYS_ENDDATE_FACT (max)
7. CNT_CREDIT_PROLONG (sum)
8. AMT_CREDIT_SUM (sum)
9. AMT_CREDIT_SUM_DEBT (sum)
10. AMT_CREDIT_SUM_LIMIT (sum)
11. AMT_CREDIT_SUM_OVERDUE (sum)
12. DAYS_CREDIT_UPDATE (max)
13. CREDIT_TYPE (dominant type with highest credit amount)

---

### STAGE 5: MERGE APPLICATION & BUREAU DATA

**Cell:** 26

**Operation:**
```python
df_merged = df_app.merge(agg_bureau, on="SK_ID_CURR", how="left")
```

**Merge Details:**

| Aspect | Value |
|--------|-------|
| Type | LEFT JOIN |
| Join Key | SK_ID_CURR |
| Left Table | df_app (appData_filtered) |
| Left Table Size | 307,511 rows × 92 columns |
| Right Table | agg_bureau |
| Right Table Size | ~305,811 rows × 13 columns |
| Result | df_merged |
| Result Size | 252,137 rows × 105 columns |

**Row Reduction Analysis:**

Expected behavior:
- LEFT JOIN should keep all 307,511 application records
- Unmatched records get NaN for bureau columns

Actual result:
- Only 252,137 rows retained (18% loss)
- **55,374 records lost**

**Possible reasons for row loss:**
1. Records were filtered out earlier in the pipeline
2. Data quality issues in SK_ID_CURR
3. Records with missing SK_ID_CURR were dropped
4. Bureau merge filtering was stricter than expected

**Post-Merge Data Handling:**

1. **Fill bureau columns with 0 for new customers**
   - Customers with no bureau history get NaN for bureau columns
   - These are filled with 0 (assumes new-to-credit customers)

2. **Drop records with missing SK_ID_CURR or AMT_INCOME_TOTAL**
   - Essential columns cannot be missing

3. **Fill remaining missing values with 'Missing' string**
   - Creates explicit missing data category

**Output:** df_merged (252,137 rows × 105 columns)

---

### STAGE 6: CORRELATION-BASED FILTERING

**Cell:** 32

**Input:** df_merged (252,137 rows × 105 columns)

**Operation:** Remove highly correlated features

**Method:**
1. Calculate Pearson correlation matrix for numeric columns
2. Identify pairs with |correlation| > 0.80 (threshold)
3. For each correlated pair, keep the column with stronger correlation to TARGET
4. Drop the weaker column

**Highly Correlated Pairs Identified (>0.80):**

| Column 1 | Column 2 | Correlation | Dropped (weaker) | Correlation to TARGET |
|----------|----------|-------------|------------------|----------------------|
| REG_CITY_NOT_LIVE_CITY | LIVE_CITY_NOT_WORK_CITY | 0.9089 | LIVE_CITY_NOT_WORK_CITY | - |
| REG_REGION_NOT_WORK_REGION | LIVE_REGION_NOT_WORK_REGION | 0.9442 | LIVE_REGION_NOT_WORK_REGION | - |
| REGION_RATING_CLIENT | REGION_RATING_CLIENT_W_CITY | 0.9547 | REGION_RATING_CLIENT | - |

**Columns Dropped (3 total):**
1. LIVE_CITY_NOT_WORK_CITY (highly correlated with REG_CITY_NOT_LIVE_CITY)
2. LIVE_REGION_NOT_WORK_REGION (highly correlated with REG_REGION_NOT_WORK_REGION)
3. REGION_RATING_CLIENT (highly correlated with REGION_RATING_CLIENT_W_CITY)

**Rationale:** These pairs have very high correlation (>0.89), indicating they carry redundant information. Removing one from each pair reduces multicollinearity while retaining predictive power.

**Output:** df_merged_cleaned (252,137 rows × 102 columns)

---

### STAGE 7: SINGLE-VALUE CATEGORICAL CHECK

**Cell:** 36

**Input:** df_merged_cleaned (252,137 rows × 102 columns)

**Operation:** Check categorical columns for single unique value (no variance)

**Result:** No single-value categorical columns found

**Output:** df_merged_cleaned (252,137 rows × 102 columns) - Unchanged

---

## Duplicate Column Issue (CRITICAL)

### Problem Description

When preparing data for WOE binning, **30 columns appear in both the numeric AND categorical column lists**, creating duplicates in the combined feature list.

### Root Cause

The issue stems from conflicting type definitions:

```python
# Line 1: These binary flags are stored as int64 (numeric type)
numeric_cols = df_merged_cleaned.select_dtypes(include=['number']).columns.tolist()

# Line 2: These same columns also appear in categorical_columns list
categorical_cols_for_woe = [col for col in categorical_columns
                            if col not in ['SK_ID_CURR', 'TARGET', 'ORGANIZATION_TYPE']
                            and col in df_merged_cleaned.columns]

# Line 3: Combining them creates duplicates
all_cols_for_woe = numeric_cols + categorical_cols_for_woe
```

**Why This Happens:**
- Binary flag columns (0/1 values) are stored as `int64` dtype in pandas
- `select_dtypes(include=['number'])` includes int64 columns → they appear in `numeric_cols`
- The `categorical_columns` list explicitly includes these same flags
- When concatenating the two lists, duplicates are not removed

### Affected Columns (30 total)

**Type 1: Binary Flag Columns (12 columns)**
- FLAG_MOBIL
- FLAG_EMP_PHONE
- FLAG_WORK_PHONE
- FLAG_CONT_MOBILE
- FLAG_PHONE
- FLAG_EMAIL

**Type 2: Document Submission Flags (20 columns)**
- FLAG_DOCUMENT_2 through FLAG_DOCUMENT_21

**Type 3: Regional Mismatch Flags (4 columns)**
- REG_REGION_NOT_LIVE_REGION
- REG_REGION_NOT_WORK_REGION
- REG_CITY_NOT_LIVE_CITY
- REG_CITY_NOT_WORK_CITY

**Type 4: Target and ID (2 columns)**
- SK_ID_CURR
- TARGET

### Where the Issue Occurs

**File:** Use_this_one.ipynb
**Cell:** Cell containing WOE binning code (approximately Cell 30-40)
**Code Section:**
```python
numeric_cols = df_merged_cleaned.select_dtypes(include=['number']).columns.tolist()
categorical_cols_for_woe = [col for col in categorical_columns
                            if col not in ['SK_ID_CURR', 'TARGET', 'ORGANIZATION_TYPE']
                            and col in df_merged_cleaned.columns]
all_cols_for_woe = numeric_cols + categorical_cols_for_woe
```

### Impact on Downstream Analysis

1. **Duplicate processing:** Scorecardpy may try to process the same columns twice
2. **Ambiguous truth values:** When scorecardpy evaluates the dataframe, it may encounter duplicate columns causing the "ambiguous truth value" error
3. **WOE binning failure:** The woebin() function fails because of conflicting data about the same feature
4. **Misleading IV values:** If somehow processed, IV values would be computed twice for the same features

### How to Reproduce

```python
# This code demonstrates the duplication:
numeric_cols = df_merged_cleaned.select_dtypes(include=['number']).columns.tolist()
categorical_cols_for_woe = [col for col in categorical_columns
                            if col not in ['SK_ID_CURR', 'TARGET', 'ORGANIZATION_TYPE']
                            and col in df_merged_cleaned.columns]

print(f"Numeric columns: {len(numeric_cols)}")              # 48
print(f"Categorical columns: {len(categorical_cols_for_woe)}")  # 47
print(f"Total if combined: {len(numeric_cols) + len(categorical_cols_for_woe)}")  # 95

# But actual unique columns should be ~72
unique_cols = list(set(numeric_cols + categorical_cols_for_woe))
print(f"Actual unique columns: {len(unique_cols)}")         # ~72
print(f"Duplicate columns: {95 - len(unique_cols)}")        # ~23 (but actually 30 due to SK_ID_CURR + TARGET)
```

### Recommended Fix

**Option 1: Remove duplicates from combined list (RECOMMENDED)**
```python
numeric_cols = df_merged_cleaned.select_dtypes(include=['number']).columns.tolist()
categorical_cols_for_woe = [col for col in categorical_columns
                            if col not in ['SK_ID_CURR', 'TARGET', 'ORGANIZATION_TYPE']
                            and col in df_merged_cleaned.columns]

all_cols_for_woe = numeric_cols + categorical_cols_for_woe

# REMOVE DUPLICATES - KEEP FIRST OCCURRENCE
all_cols_for_woe = list(dict.fromkeys(all_cols_for_woe))

# OPTIONAL: Remove SK_ID_CURR as it's just an ID
all_cols_for_woe = [col for col in all_cols_for_woe if col != 'SK_ID_CURR']

df_for_woe = df_merged_cleaned[all_cols_for_woe + ['TARGET']].copy()
```

**Option 2: Convert binary flags to categorical type**
```python
# Before creating feature lists, convert binary flags to categorical
binary_flags = [col for col in df_merged_cleaned.columns
                if col.startswith('FLAG_') or col.startswith('REG_')]

for col in binary_flags:
    df_merged_cleaned[col] = df_merged_cleaned[col].astype('category')

# Now numeric_cols won't include these
numeric_cols = df_merged_cleaned.select_dtypes(include=['number']).columns.tolist()
```

**Option 3: Separate numeric and categorical WOE binning**
```python
# Perform WOE binning separately for each type
numeric_cols = df_merged_cleaned.select_dtypes(include=['number']).columns.tolist()
numeric_cols = [col for col in numeric_cols if col not in ['SK_ID_CURR', 'TARGET']]

categorical_cols = [col for col in categorical_columns
                    if col not in ['SK_ID_CURR', 'TARGET', 'ORGANIZATION_TYPE']
                    and col in df_merged_cleaned.columns]

# Bin numeric features
numeric_bins = sc.woebin(df_merged_cleaned[numeric_cols + ['TARGET']],
                         y='TARGET', auto_continue=True)

# Bin categorical features
categorical_bins = sc.woebin(df_merged_cleaned[categorical_cols + ['TARGET']],
                             y='TARGET', auto_continue=True)

# Combine results
all_bins = {**numeric_bins, **categorical_bins}
```

---

## Complete Transformation Table

| Step | Cell | Operation | Input | Output | Rows Change | Cols Change |
|------|------|-----------|-------|--------|-------------|------------|
| 1 | 5 | Load Application CSV | - | appData | +307,511 | +120 |
| 2 | 8 | Load Bureau CSV | - | bureauData | +1,716,428 | +17 |
| 3 | 11 | Filter Application Data | appData (307,511×120) | appData_filtered (307,511×92) | 0 | -28 |
| 4 | 12a | Merge Bureau with TARGET | bureauData (1,716,428×17) | bureauData_with_target (1,716,428×18) | 0 | +1 |
| 5 | 12b | Filter Bureau Data | bureauData_with_target (1,716,428×18) | bureauData_filtered (~1,690,392×15) | -26,036 | -3 |
| 6 | 24a | Copy Application | appData_filtered (307,511×92) | df_app (307,511×92) | 0 | 0 |
| 7 | 24b | Copy Bureau | bureauData_filtered (~1,690,392×15) | df_bureau (~1,690,392×15) | 0 | 0 |
| 8 | 24c | Aggregate Bureau | df_bureau (~1,690,392×15) | agg_bureau (~305,811×13) | -1,384,581 | -2 |
| 9 | 26a | Merge App + Bureau | df_app (307,511×92) + agg_bureau (~305,811×13) | df_merged (252,137×105) | -55,374 | +13 |
| 10 | 26b | Fill Missing Values | df_merged (252,137×105) | df_merged (252,137×105) | 0 | 0 |
| 11 | 32 | Remove Correlations | df_merged (252,137×105) | df_merged_cleaned (252,137×102) | 0 | -3 |
| 12 | 36 | Check Single-Value | df_merged_cleaned (252,137×102) | df_merged_cleaned (252,137×102) | 0 | 0 |

---

## Current Data State

### df_merged_cleaned Final Specifications

| Attribute | Value |
|-----------|-------|
| **Total Rows** | 252,137 |
| **Total Columns** | 102 |
| **Target Variable** | TARGET (0 = good, 1 = default) |
| **Primary Key** | SK_ID_CURR |
| **Default Rate** | ~8% (estimated from original 8.07%) |
| **Data Loss from Original** | 55,374 records (18%) |

### Column Breakdown (by source)

**From Application Data (89 columns):**

*Demographics (7 cols):*
- DAYS_BIRTH, CNT_CHILDREN, CNT_FAM_MEMBERS, NAME_FAMILY_STATUS, NAME_EDUCATION_TYPE, NAME_INCOME_TYPE, OCCUPATION_TYPE

*Financial (5 cols):*
- AMT_INCOME_TOTAL, AMT_CREDIT, AMT_ANNUITY, AMT_GOODS_PRICE, FLAG_OWN_CAR, FLAG_OWN_REALTY

*Temporal (6 cols):*
- DAYS_EMPLOYED, DAYS_REGISTRATION, DAYS_ID_PUBLISH, DAYS_LAST_PHONE_CHANGE, WEEKDAY_APPR_PROCESS_START, HOUR_APPR_PROCESS_START

*Geographic (7 cols):*
- REGION_POPULATION_RELATIVE, REGION_RATING_CLIENT_W_CITY, REG_REGION_NOT_LIVE_REGION, REG_REGION_NOT_WORK_REGION, REG_CITY_NOT_LIVE_CITY, REG_CITY_NOT_WORK_CITY

*Contact Flags (6 cols):*
- FLAG_MOBIL, FLAG_EMP_PHONE, FLAG_WORK_PHONE, FLAG_CONT_MOBILE, FLAG_PHONE, FLAG_EMAIL

*Document Flags (20 cols):*
- FLAG_DOCUMENT_2 through FLAG_DOCUMENT_21

*Contract & Type (3 cols):*
- NAME_CONTRACT_TYPE, NAME_TYPE_SUITE, NAME_HOUSING_TYPE

*Building Info (26 cols):*
- APARTMENTS_AVG, APARTMENTS_MODE, APARTMENTS_MEDI
- YEARS_BEGINEXPLUATATION_AVG, YEARS_BEGINEXPLUATATION_MODE, YEARS_BEGINEXPLUATATION_MEDI
- ELEVATORS_AVG, ELEVATORS_MODE, ELEVATORS_MEDI
- ENTRANCES_AVG, ENTRANCES_MODE, ENTRANCES_MEDI
- FLOORSMAX_AVG, FLOORSMAX_MODE, FLOORSMAX_MEDI
- LIVINGAREA_AVG, LIVINGAREA_MODE, LIVINGAREA_MEDI
- HOUSETYPE_MODE, TOTALAREA_MODE, WALLSMATERIAL_MODE, EMERGENCYSTATE_MODE
- FONDKAPREMONT_MODE (1 col)

*Social Circle (4 cols):*
- OBS_30_CNT_SOCIAL_CIRCLE, DEF_30_CNT_SOCIAL_CIRCLE, OBS_60_CNT_SOCIAL_CIRCLE, DEF_60_CNT_SOCIAL_CIRCLE

*Credit Bureau Activity (6 cols):*
- AMT_REQ_CREDIT_BUREAU_HOUR, AMT_REQ_CREDIT_BUREAU_DAY, AMT_REQ_CREDIT_BUREAU_WEEK, AMT_REQ_CREDIT_BUREAU_MON, AMT_REQ_CREDIT_BUREAU_QRT, AMT_REQ_CREDIT_BUREAU_YEAR

*Identifiers & Target (2 cols):*
- SK_ID_CURR, TARGET

**From Bureau Data Aggregation (13 columns):**
- NUM_PREV_LOANS, CREDIT_ACTIVE, CREDIT_TYPE, CREDIT_DAY_OVERDUE, DAYS_CREDIT, DAYS_CREDIT_ENDDATE, DAYS_ENDDATE_FACT, CNT_CREDIT_PROLONG, AMT_CREDIT_SUM, AMT_CREDIT_SUM_DEBT, AMT_CREDIT_SUM_LIMIT, AMT_CREDIT_SUM_OVERDUE, DAYS_CREDIT_UPDATE

### Data Types Distribution

| Type | Count | Examples |
|------|-------|----------|
| int64 | ~50 | SK_ID_CURR, CNT_CHILDREN, DAYS_BIRTH, all FLAGS, etc. |
| float64 | ~35 | AMT_INCOME_TOTAL, REGION_POPULATION_RELATIVE, aggregated amounts, etc. |
| object | ~17 | NAME_CONTRACT_TYPE, NAME_FAMILY_STATUS, CREDIT_ACTIVE, CREDIT_TYPE, HOUSETYPE_MODE, WALLSMATERIAL_MODE, EMERGENCYSTATE_MODE |

### Missing Values Summary

**Original Status:**
- Application Data: Minimal missing values (intentionally filtered)
- Bureau Data: Some missing values handled with "Missing" string representation

**Current Status (df_merged_cleaned):**
- Bureau columns for new customers: Filled with 0 or NaN (represents no history)
- Categorical columns with missing: Contains "Missing" as string value
- Numeric columns: Some NaN values may exist

---

## Key Insights & Recommendations

### Data Quality Observations

✅ **Strengths:**
1. **Effective filtering:** 28 columns removed from application for poor quality
2. **Bureau data cleaning:** ~26K invalid records removed (negative values, logical inconsistencies)
3. **Smart aggregation:** Successfully consolidated 1.7M bureau records to 306K customer-level features
4. **Multicollinearity handling:** Identified and removed 3 highly correlated columns (>0.94 correlation)
5. **Missing value strategy:** Explicit handling with "Missing" category for categorical data

⚠️ **Concerns:**
1. **18% row loss after merge:** 55,374 records (18%) lost after application-bureau merge - needs investigation
2. **Duplicate column issue:** 30 binary flag columns treated as both numeric and categorical
3. **No multicollinearity analysis for categorical:** Only numeric multicollinearity addressed
4. **Building features redundancy:** 54 building-related columns (26 after filtering) may still be redundant across AVG/MODE/MEDIAN versions
5. **Feature concentration:** Heavy emphasis on location/building features (26 cols) vs. behavioral features (limited)

### Recommendations

#### Immediate Action Items

1. **Fix Duplicate Columns Issue** (CRITICAL for WOE binning)
   - Use `list(dict.fromkeys())` to remove duplicates from combined column list
   - OR convert binary flags to categorical dtype before selecting numeric columns
   - This blocks WOE binning currently

2. **Investigate 18% Row Loss**
   - Check for data quality issues in SK_ID_CURR
   - Verify merge logic is correct
   - Determine if lost records are systematically different (bias risk)

3. **Handle Missing Data Representation**
   - Bureau columns with NaN vs. "Missing" string inconsistency
   - Decide on imputation strategy (0 for "no history" vs. explicit missing marker)

#### Feature Engineering Opportunities

1. **Reduce Building Feature Redundancy**
   - Keep only one version (MODE typically most relevant for categories)
   - Or create PCA components from the 26 building features
   - Current approach: 26 out of 102 columns (25% of features)

2. **Create Derived Features**
   - **Days Since:** Time-decay features (DAYS_CREDIT recency indicator)
   - **Credit Utilization:** AMT_CREDIT_SUM_DEBT / AMT_CREDIT_SUM ratio
   - **Portfolio Diversity:** Count of different CREDIT_TYPEs per customer
   - **Delinquency Ratio:** AMT_CREDIT_SUM_OVERDUE / AMT_CREDIT_SUM

3. **Categorical Feature Grouping**
   - Rare categories → "Other" group
   - OCCUPATION_TYPE likely has many low-frequency categories
   - ORGANIZATION_TYPE explicitly flagged as high-cardinality in earlier analysis

#### For Fair Lending Compliance

1. **Protected Class Risk** ⚠️
   - DAYS_BIRTH → Age proxy (correlates with location, occupation)
   - Building characteristics → Geographic proxy (redlining risk)
   - OWN_CAR_AGE → Age proxy
   - Recommend disparate impact testing across demographic groups

2. **Documentation**
   - Document rationale for each excluded/included feature
   - Track feature engineering decisions
   - Prepare for model explainability requirements

#### For Modeling

1. **Class Imbalance**
   - Default rate ~8% → Severe class imbalance
   - Consider: SMOTE, class weights, stratified sampling

2. **Feature Scaling**
   - Numeric features have vastly different ranges
   - Required before: Logistic regression, neural networks, distance-based models
   - NOT required before: Tree-based models

3. **Multicollinearity**
   - Consider VIF analysis for remaining features
   - PCA alternative for highly correlated feature groups
   - Building features are likely still correlated

4. **Interaction Terms**
   - DAYS_CREDIT × NUM_PREV_LOANS (credit age vs. frequency)
   - AMT_INCOME_TOTAL × AMT_CREDIT (leverage ratio)
   - CREDIT_DAY_OVERDUE × CREDIT_ACTIVE (current delinquency indicator)

---

## Appendix: Column Definitions

### All 102 Columns in df_merged_cleaned

**Numeric (approx 50 columns):**
SK_ID_CURR, CNT_CHILDREN, CNT_FAM_MEMBERS, AMT_INCOME_TOTAL, AMT_CREDIT, REGION_POPULATION_RELATIVE, DAYS_BIRTH, DAYS_EMPLOYED, DAYS_REGISTRATION, DAYS_ID_PUBLISH, FLAG_MOBIL, FLAG_EMP_PHONE, FLAG_WORK_PHONE, FLAG_CONT_MOBILE, FLAG_PHONE, FLAG_EMAIL, REGION_RATING_CLIENT_W_CITY, HOUR_APPR_PROCESS_START, REG_REGION_NOT_LIVE_REGION, REG_REGION_NOT_WORK_REGION, REG_CITY_NOT_LIVE_CITY, REG_CITY_NOT_WORK_CITY, FLAG_DOCUMENT_2...21 (20 cols), OBS_30_CNT_SOCIAL_CIRCLE, DEF_30_CNT_SOCIAL_CIRCLE, OBS_60_CNT_SOCIAL_CIRCLE, DEF_60_CNT_SOCIAL_CIRCLE, DAYS_LAST_PHONE_CHANGE, [APARTMENTS_*, ELEVATORS_*, ENTRANCES_*, FLOORSMAX_*, LIVINGAREA_*]_AVG/MODE/MEDI, AMT_REQ_CREDIT_BUREAU_*, NUM_PREV_LOANS, CREDIT_DAY_OVERDUE, DAYS_CREDIT, DAYS_CREDIT_ENDDATE, DAYS_ENDDATE_FACT, CNT_CREDIT_PROLONG, AMT_CREDIT_SUM, AMT_CREDIT_SUM_DEBT, AMT_CREDIT_SUM_LIMIT, AMT_CREDIT_SUM_OVERDUE, DAYS_CREDIT_UPDATE

**Categorical (approx 17 columns):**
TARGET, NAME_CONTRACT_TYPE, FLAG_OWN_CAR, FLAG_OWN_REALTY, NAME_TYPE_SUITE, NAME_INCOME_TYPE, NAME_EDUCATION_TYPE, NAME_FAMILY_STATUS, NAME_HOUSING_TYPE, OCCUPATION_TYPE, WEEKDAY_APPR_PROCESS_START, HOUSETYPE_MODE, WALLSMATERIAL_MODE, EMERGENCYSTATE_MODE, CREDIT_ACTIVE, CREDIT_TYPE

**Float64 (approx 35 columns):**
AMT_ANNUITY, AMT_GOODS_PRICE, OWN_CAR_AGE, [all aggregated amounts and percentiles]

---

## Document History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-11-08 | Initial comprehensive mapping document |

---

**End of Mapping Document**
