# Bureau Data Flattening Documentation

## Executive Summary

This document details the process of transforming the **IS453 Group Assignment - Bureau Data.csv** from a one-to-many structure into a flat, aggregated dataset where each applicant (`SK_ID_CURR`) is represented by a single row. This flattening is necessary to enable merging with the Application Data dataset for credit scorecard development.

---

## 1. Context: Why Flattening is Necessary

### Current Data Structure Problem

**Bureau Data as delivered:**
- **File**: `IS453 Group Assignment - Bureau Data.csv`
- **Rows**: 1,716,428
- **Columns**: 17
- **Unique Customers**: 305,811
- **Average Records per Customer**: 5.61

Each applicant (`SK_ID_CURR`) has multiple rows in the Bureau Data, each representing a different credit account reported to the credit bureau. This one-to-many relationship reflects real-world credit history—a person may have multiple loans, credit cards, mortgages, etc.

**Application Data structure:**
- **File**: `IS453 Group Assignment - Application Data.csv`
- **Rows**: 307,511
- **Columns**: 120
- **Structure**: One row per loan application (unique `SK_ID_CURR`)
- **Primary Key**: `SK_ID_CURR` (unique, no duplicates)

**The Merge Problem:**
Cannot directly join 1,716,428 Bureau rows to 307,511 Application rows using a simple LEFT JOIN on `SK_ID_CURR`. A direct join would create many duplicate Application records (one for each Bureau record per applicant), which is not the desired analytical structure.

**The Solution:**
Flatten the Bureau Data by aggregating all records for each applicant into a single row, creating derived features that capture the applicant's credit history profile.

### Target Segment Context

For the **Lifelong Learning or Skill-Upgrade Loans** product:
- **Target applicants**: Ages 25-45, income ≤ $96,000, stable employment
- **Filtering reduces dataset** to ~31,125 applicants (10.12% of population)
- **Coverage**: ~85.7% of applications have corresponding Bureau history
- **New-to-credit applicants**: ~14.3% have no Bureau data (will have zero aggregates)

---

## 2. Bureau Data Dataset Overview

### All 17 Columns

| Column | Data Type | Description | Missing Values |
|--------|-----------|-------------|-----------------|
| `SK_ID_CURR` | Integer | Customer ID (links to Application Data) | 0 |
| `SK_ID_BUREAU` | Integer | **Unique identifier for each credit bureau record** | 0 |
| `CREDIT_ACTIVE` | Categorical | Status of the credit (Active, Closed, etc.) | 0 |
| `CREDIT_CURRENCY` | Integer/Categorical | Recoded currency of the credit | 0 |
| `DAYS_CREDIT` | Integer | Days before current application when credit appeared in CB | 0 |
| `CREDIT_DAY_OVERDUE` | Integer | Days past due on CB credit at time of application | 0 |
| `DAYS_CREDIT_ENDDATE` | Integer | Remaining duration of CB credit in days | 0 |
| `DAYS_ENDDATE_FACT` | Integer | Days since CB credit ended | May have NaN |
| `AMT_CREDIT_MAX_OVERDUE` | Float | Maximal amount overdue on CB credit | 0 |
| `CNT_CREDIT_PROLONG` | Integer | How many times CB credit was prolonged/extended | 0 |
| `AMT_CREDIT_SUM` | Float | Current credit amount for CB credit | 13 missing |
| `AMT_CREDIT_SUM_DEBT` | Float | Current debt on CB credit | May have NaN |
| `AMT_CREDIT_SUM_LIMIT` | Float | Current credit limit of credit card | May have NaN |
| `AMT_CREDIT_SUM_OVERDUE` | Float | Current amount overdue on CB credit | 0 |
| `CREDIT_TYPE` | Categorical | Type of credit (Consumer, Credit Card, Mortgage, etc.) | 0 |
| `DAYS_CREDIT_UPDATE` | Integer | Days since CB credit was last updated | 0 |
| `AMT_ANNUITY` | Float | Annuity of CB credit | May have NaN |

**Key Columns to Exclude:**
- `SK_ID_BUREAU` - Record-level unique identifier (not needed after flattening; each aggregated row represents all records for one applicant)

**Key Columns to Preserve:**
- `SK_ID_CURR` - Customer ID (becomes the primary key of flattened dataset, one per row)

---

## 3. Flattening Strategy

### Overview

The flattening process groups all Bureau records by `SK_ID_CURR`, then applies:
1. **Numerical aggregations** (MIN, MAX, MEAN, SUM) to 12 numeric columns
2. **Categorical aggregations** (counts) to 3 categorical columns

### 3.1 Numerical Column Aggregations

**Target Columns** (12 total):
```
DAYS_CREDIT
CREDIT_DAY_OVERDUE
DAYS_CREDIT_ENDDATE
DAYS_ENDDATE_FACT
AMT_CREDIT_MAX_OVERDUE
CNT_CREDIT_PROLONG
AMT_CREDIT_SUM
AMT_CREDIT_SUM_DEBT
AMT_CREDIT_SUM_LIMIT
AMT_CREDIT_SUM_OVERDUE
DAYS_CREDIT_UPDATE
AMT_ANNUITY
```

**Aggregation Functions:**
- **MIN**: Minimum value across all records for that applicant
- **MAX**: Maximum value across all records for that applicant
- **MEAN**: Average value across all records for that applicant
- **SUM**: Total value across all records for that applicant

**Output Naming Convention:**
```
<ORIGINAL_COLUMN>_MIN
<ORIGINAL_COLUMN>_MAX
<ORIGINAL_COLUMN>_MEAN
<ORIGINAL_COLUMN>_SUM
```

**Example:**
```
DAYS_CREDIT → DAYS_CREDIT_MIN, DAYS_CREDIT_MAX, DAYS_CREDIT_MEAN, DAYS_CREDIT_SUM
AMT_CREDIT_SUM → AMT_CREDIT_SUM_MIN, AMT_CREDIT_SUM_MAX, AMT_CREDIT_SUM_MEAN, AMT_CREDIT_SUM_SUM
```

**Result:**
- 12 columns × 4 functions = **48 numerical aggregate columns**

**Interpretation Examples:**
- `DAYS_CREDIT_MIN`: How long ago (in days) was the oldest credit account reported?
- `DAYS_CREDIT_MAX`: How long ago (in days) was the most recent credit account reported?
- `AMT_CREDIT_SUM_SUM`: What is the total credit amount across all accounts?
- `CNT_CREDIT_PROLONG_MEAN`: On average, how many times were credit accounts prolonged?

---

### 3.2 Categorical Column Aggregations

**Target Columns** (3 total):
```
CREDIT_ACTIVE
CREDIT_CURRENCY
CREDIT_TYPE
```

**Aggregation Method:** One-hot encoding with counts
- For each unique category value in a categorical column, create a new column
- The value in that new column = count of how many times that category appears for each applicant
- Missing categories default to 0

**Output Naming Convention:**
```
COUNT_<COLUMN>_<CATEGORY_VALUE>
```

**Examples:**

*CREDIT_ACTIVE aggregation:*
```
COUNT_CREDIT_ACTIVE_ACTIVE    → Number of active credits per applicant
COUNT_CREDIT_ACTIVE_CLOSED    → Number of closed credits per applicant
COUNT_CREDIT_ACTIVE_BAD_DEBT  → Number of bad debt accounts per applicant
COUNT_CREDIT_ACTIVE_SOLD      → Number of sold accounts per applicant
```

*CREDIT_TYPE aggregation:*
```
COUNT_CREDIT_TYPE_CONSUMER_CREDIT   → Count of consumer credit accounts
COUNT_CREDIT_TYPE_CREDIT_CARD       → Count of credit card accounts
COUNT_CREDIT_TYPE_MORTGAGE          → Count of mortgage accounts
COUNT_CREDIT_TYPE_CAR_LOAN          → Count of car loan accounts
COUNT_CREDIT_TYPE_MICRO_LOAN        → Count of micro loan accounts
... (one for each unique value in CREDIT_TYPE)
```

*CREDIT_CURRENCY aggregation:*
```
COUNT_CREDIT_CURRENCY_1    → Count of credits in currency code 1
COUNT_CREDIT_CURRENCY_2    → Count of credits in currency code 2
... (one for each unique currency code)
```

**Missing Value Handling:**
- If an applicant has no records for a category, the count = 0
- This ensures no NaN values in the output

**Result:**
- Number of categorical columns depends on unique values across all three categorical columns
- Estimated: 15-25+ additional columns (varies by data)

---

## 4. Flattened Dataset Output Structure

### Final Output File
- **Filename**: `IS453_Group_Assignment_Bureau_Flattened.csv`
- **Location**: `/mnt/user-data/outputs/IS453_Group_Assignment_Bureau_Flattened.csv`

### Flattened Dataset Shape
- **Rows**: 305,811 (one unique `SK_ID_CURR`)
- **Columns**: ~65-75 total
  - 1 column: `SK_ID_CURR`
  - 48 columns: Numerical aggregates (12 columns × 4 functions)
  - ~15-25 columns: Categorical counts (varies by unique values)

### Column Organization
```
SK_ID_CURR
├── Numerical Aggregates (48 columns)
│   ├── DAYS_CREDIT_MIN, DAYS_CREDIT_MAX, DAYS_CREDIT_MEAN, DAYS_CREDIT_SUM
│   ├── CREDIT_DAY_OVERDUE_MIN, CREDIT_DAY_OVERDUE_MAX, ...
│   ├── DAYS_CREDIT_ENDDATE_MIN, DAYS_CREDIT_ENDDATE_MAX, ...
│   ├── DAYS_ENDDATE_FACT_MIN, DAYS_ENDDATE_FACT_MAX, ...
│   ├── AMT_CREDIT_MAX_OVERDUE_MIN, AMT_CREDIT_MAX_OVERDUE_MAX, ...
│   ├── CNT_CREDIT_PROLONG_MIN, CNT_CREDIT_PROLONG_MAX, ...
│   ├── AMT_CREDIT_SUM_MIN, AMT_CREDIT_SUM_MAX, ...
│   ├── AMT_CREDIT_SUM_DEBT_MIN, AMT_CREDIT_SUM_DEBT_MAX, ...
│   ├── AMT_CREDIT_SUM_LIMIT_MIN, AMT_CREDIT_SUM_LIMIT_MAX, ...
│   ├── AMT_CREDIT_SUM_OVERDUE_MIN, AMT_CREDIT_SUM_OVERDUE_MAX, ...
│   ├── DAYS_CREDIT_UPDATE_MIN, DAYS_CREDIT_UPDATE_MAX, ...
│   └── AMT_ANNUITY_MIN, AMT_ANNUITY_MAX, ...
└── Categorical Counts (~15-25 columns)
    ├── COUNT_CREDIT_ACTIVE_ACTIVE, COUNT_CREDIT_ACTIVE_CLOSED, ...
    ├── COUNT_CREDIT_TYPE_CONSUMER_CREDIT, COUNT_CREDIT_TYPE_CREDIT_CARD, ...
    └── COUNT_CREDIT_CURRENCY_1, COUNT_CREDIT_CURRENCY_2, ...
```

---

## 5. Data Quality Considerations

### Missing Values in Source Data

**Minimal Missing Values in Bureau Data:**
- `AMT_CREDIT_SUM`: 13 missing values (0.00%)
- `DAYS_ENDDATE_FACT`: May have NaN for active credits
- `AMT_CREDIT_SUM_DEBT`: May have NaN for certain credit types
- `AMT_CREDIT_SUM_LIMIT`: May be NaN for non-card credits
- `AMT_ANNUITY`: May have NaN

**Handling in Aggregation:**
- Pandas aggregation functions (`min()`, `max()`, `mean()`, `sum()`) automatically skip NaN values
- Missing values will **not propagate** to flattened output
- For applicants with no valid data for a column, result will be NaN (then filled with 0 as needed)

### Linking Integrity

**SK_ID_CURR Coverage:**
- Bureau Data has 305,811 unique `SK_ID_CURR` values
- Application Data has 307,511 unique `SK_ID_CURR` values
- **Gap**: 44,020 applicants in Application Data with no Bureau history (new-to-credit customers)
- These applicants will have all Bureau aggregates = 0 when left-joined

### Data Validation After Flattening

**Expected Validation Results:**
✓ 305,811 unique rows (one per unique `SK_ID_CURR`)
✓ No duplicate `SK_ID_CURR` values
✓ 48 numerical aggregate columns with `_MIN`, `_MAX`, `_MEAN`, `_SUM` suffixes
✓ ~15-25 categorical count columns with `COUNT_*` naming pattern
✓ No NaN values in categorical count columns (all default to 0)
✓ Numerical aggregates may contain 0 or positive values only

---

## 6. Integration with Application Data

### Merge Process

After flattening, the Bureau data can be merged with Application data:

```python
# Flattened Bureau data: 305,811 rows × ~65-75 columns
bureau_flat = pd.read_csv('IS453_Group_Assignment_Bureau_Flattened.csv')

# Application data: 307,511 rows × 120 columns
app_df = pd.read_csv('IS453 Group Assignment - Application Data.csv')

# Left join (keep all applications, add Bureau features where available)
merged_df = app_df.merge(bureau_flat, on='SK_ID_CURR', how='left')

# Result: 307,511 rows × ~185-195 columns
# - All 120 original Application columns
# - ~65-75 Bureau aggregate columns (0 for applicants without Bureau history)
```

### Feature Engineering Opportunities

Once merged, additional features can be created from the flattened Bureau aggregates:

**Derived Features:**
```
Total_Credits = COUNT_CREDIT_ACTIVE_ACTIVE + COUNT_CREDIT_ACTIVE_CLOSED
Active_Credits = COUNT_CREDIT_ACTIVE_ACTIVE
Closed_Credits = COUNT_CREDIT_ACTIVE_CLOSED

Credit_Utilization = AMT_CREDIT_SUM_DEBT_SUM / AMT_CREDIT_SUM_SUM
Average_Credit_Age = DAYS_CREDIT_MEAN

Max_Historical_Overdue = AMT_CREDIT_MAX_OVERDUE_MAX
Total_Overdue = AMT_CREDIT_SUM_OVERDUE_SUM

Recent_Credit_Activity = (DAYS_CREDIT_UPDATE_MIN < -90)  # Updated in last 90 days

Credit_Type_Diversity = COUNT(non-zero COUNT_CREDIT_TYPE_*)

Has_Bureau_History = (DAYS_CREDIT_MIN is not NaN)
```

---

## 7. Lifelong Learning Segment Specifics

### Segment Filtering Criteria (reminder)

When filtering the merged dataset for the Lifelong Learning segment:

```
Age (DAYS_BIRTH):           -16,425 to -9,125 days (25-45 years)
Income (AMT_INCOME_TOTAL):  36,000 to 72,000
Income Type:                Working, Commercial associate, State servant
Contract Type:              Cash loans
Employment (DAYS_EMPLOYED): ≤ -365 (1+ years employed)
```

**Expected segment size:** ~31,125 applicants
**Expected default rate:** 9.99% (higher risk than population)

### Bureau Features for Segment

For the Lifelong Learning segment, the flattened Bureau features will reveal:

- **Credit History Depth**: How many prior credits does the typical applicant have?
- **Credit Mix**: Ratio of credit cards to consumer loans to mortgages
- **Delinquency Risk**: Historical overdue amounts and prolongation counts
- **Recent Activity**: How recently has the applicant used credit?
- **Credit Limits**: Average and total credit limits available to applicant

These features are valuable predictors for a scorecard because:
- Established credit history (multiple prior accounts) → more predictable repayment
- Active credit usage → demonstrates ongoing financial engagement
- Low historical delinquency → lower default risk
- Recent activity → applicant still active in credit market

---

## 8. Processing Implementation Steps

### Pseudocode Overview

```
1. READ Bureau Data CSV
   └─ 1,716,428 rows × 17 columns

2. GROUP BY SK_ID_CURR
   └─ Create 305,811 groups

3. FOR EACH NUMERICAL COLUMN (12 columns):
   ├─ Compute: MIN, MAX, MEAN, SUM
   └─ Create 4 output columns per source column (48 total)

4. FOR EACH CATEGORICAL COLUMN (3 columns):
   ├─ GET unique values in column
   ├─ FOR EACH unique value:
   │  └─ COUNT occurrences per SK_ID_CURR
   └─ Create output column: COUNT_<COLUMN>_<VALUE>

5. COMBINE all aggregated columns
   └─ Result: 305,811 rows × (~65-75 columns)

6. FILL missing categorical counts with 0
   └─ Ensure no NaN in COUNT_* columns

7. EXPORT to CSV
   └─ File: IS453_Group_Assignment_Bureau_Flattened.csv

8. VALIDATE output
   ├─ Check unique SK_ID_CURR count = 305,811
   ├─ Verify no duplicate rows
   ├─ Confirm all _MIN, _MAX, _MEAN, _SUM suffixes present
   ├─ Confirm all COUNT_* columns present
   └─ Verify no NaN in categorical count columns
```

---

## 9. Key Metrics After Flattening

### Size Reduction
| Metric | Before | After |
|--------|--------|-------|
| Total Rows | 1,716,428 | 305,811 |
| Unique Customers | 305,811 | 305,811 |
| Data Points | 29,178,276 | ~20,399,535 |
| Compression Ratio | 100% | 5.6% (per customer) |

### Feature Expansion
| Category | Count |
|----------|-------|
| Original Bureau Columns | 17 |
| Removed (SK_ID_BUREAU) | -1 |
| Numerical Aggregates | +48 |
| Categorical Counts | +~15-25 |
| **Total Output Columns** | **~65-75** |

---

## 10. Next Steps After Flattening

1. **Load flattened Bureau data**
   ```python
   bureau_flat = pd.read_csv('IS453_Group_Assignment_Bureau_Flattened.csv')
   ```

2. **Merge with Application data**
   ```python
   merged_df = app_df.merge(bureau_flat, on='SK_ID_CURR', how='left')
   ```

3. **Filter for Lifelong Learning segment**
   ```python
   segment = merged_df[
       (-16425 <= merged_df['DAYS_BIRTH']) & (merged_df['DAYS_BIRTH'] <= -9125) &
       (36000 <= merged_df['AMT_INCOME_TOTAL']) & (merged_df['AMT_INCOME_TOTAL'] <= 72000) &
       (merged_df['NAME_INCOME_TYPE'].isin(['Working', 'Commercial associate', 'State servant'])) &
       (merged_df['NAME_CONTRACT_TYPE'] == 'Cash loans')
   ]
   ```

4. **Feature engineering**
   - Create derived features from Bureau aggregates
   - Handle outliers and missing values
   - Prepare for credit scorecard model

5. **Exploratory Data Analysis**
   - Analyze correlations between Bureau features and TARGET
   - Identify predictive Bureau features for scorecard

6. **Logistic Regression Modeling**
   - Build credit scorecard using course techniques
   - Apply Weight of Evidence (WOE) transformation
   - Calculate scorecard points

---

## 11. File References

### Input Files
- **Bureau Data**: `/mnt/project/IS453 Group Assignment - Bureau Data.csv`
- **Application Data**: `/mnt/project/IS453 Group Assignment - Application Data.csv`
- **Data Dictionary**: `/mnt/project/IS453 Group Assignment - Data Dict.xlsx`

### Output Files
- **Flattened Bureau**: `/mnt/user-data/outputs/IS453_Group_Assignment_Bureau_Flattened.csv`

### Documentation Files
- **CLAUDE.md**: Original dataset documentation
- **FLATTEN_DATA.md**: This file (flattening process documentation)
- **INSTRUCTION.md**: Detailed flattening specifications (source requirements)

---

## Appendix: Column Reference

### Numerical Columns (to be aggregated with MIN, MAX, MEAN, SUM)

| # | Column | Description | Data Type |
|---|--------|-------------|-----------|
| 1 | DAYS_CREDIT | Days before application when credit appeared in CB | Integer |
| 2 | CREDIT_DAY_OVERDUE | Days past due on CB credit | Integer |
| 3 | DAYS_CREDIT_ENDDATE | Remaining duration of CB credit | Integer |
| 4 | DAYS_ENDDATE_FACT | Days since CB credit ended | Integer |
| 5 | AMT_CREDIT_MAX_OVERDUE | Maximal amount overdue | Float |
| 6 | CNT_CREDIT_PROLONG | Times CB credit was prolonged | Integer |
| 7 | AMT_CREDIT_SUM | Current credit amount | Float |
| 8 | AMT_CREDIT_SUM_DEBT | Current debt | Float |
| 9 | AMT_CREDIT_SUM_LIMIT | Credit limit | Float |
| 10 | AMT_CREDIT_SUM_OVERDUE | Current amount overdue | Float |
| 11 | DAYS_CREDIT_UPDATE | Days since last update | Integer |
| 12 | AMT_ANNUITY | Annuity of CB credit | Float |

### Categorical Columns (to be one-hot encoded with counts)

| # | Column | Examples of Unique Values |
|---|--------|---------------------------|
| 1 | CREDIT_ACTIVE | Active, Closed, Bad Debt, Sold |
| 2 | CREDIT_CURRENCY | 1, 2, 3, ... (currency codes) |
| 3 | CREDIT_TYPE | Consumer Credit, Credit Card, Mortgage, Car Loan, Micro Loan, ... |

### Identifier & Excluded Column

| Column | Action | Reason |
|--------|--------|--------|
| SK_ID_CURR | **KEEP** | Primary key for output (one row per unique value) |
| SK_ID_BUREAU | **EXCLUDE** | Record-level unique ID; not needed after aggregation |

---

*Last Updated: November 1, 2025*
*Part of IS453 Group Assignment - Credit Scorecard Development Project*