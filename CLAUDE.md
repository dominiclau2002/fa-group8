# IS453 Group Assignment - Dataset Documentation

## Overview

This document provides comprehensive documentation of the three key data sources used in the IS453 Group Assignment:
1. **Application Data.csv** - Loan application records
2. **Bureau Data.csv** - Credit bureau history
3. **IS453 Group Assignment - Data Dict.xlsx** - Data dictionary definitions

---

## 1. Application Data Dataset

### Dataset Structure
- **File**: `IS453 Group Assignment - Application Data.csv`
- **Size**: 307,511 rows × 120 columns
- **Primary Key**: `SK_ID_CURR` (unique, no duplicates)
- **Target Variable**: `TARGET` (binary: 0 = good, 1 = default)

### Target Variable Distribution
- **Good loans (TARGET=0)**: 282,686 records (91.93%)
- **Default loans (TARGET=1)**: 24,825 records (8.07%)
- **Default rate**: 8.07%

### Column Categories

#### 1. Identifiers & Target (2 columns)
- `SK_ID_CURR` - Customer ID (unique identifier)
- `TARGET` - Loan repayment status indicator

#### 2. Demographics (8 columns)
- `CODE_GENDER` - Gender of applicant (fair lending concern)
- `DAYS_BIRTH` - Days before application when applicant was born
  - Age range: 20.5 to 69.1 years
  - Range: -25,229 to -7,489 days
  - No missing values
- `CNT_CHILDREN` - Number of children
- `CNT_FAM_MEMBERS` - Number of family members
- `NAME_FAMILY_STATUS` - Family/marital status (fair lending concern)
- `NAME_EDUCATION_TYPE` - Education level
- `OWN_CAR_AGE` - Age of owned car (if applicable)
- `OCCUPATION_TYPE` - Type of occupation

#### 3. Financial Information (5 columns)
- `AMT_INCOME_TOTAL` - Annual income of applicant
  - Range: 25,650 to 117,000,000
  - Mean: 168,797.92
  - Median: 147,150
  - No zero/negative values
  - Contains extreme outliers
- `AMT_CREDIT` - Credit amount of the loan
- `AMT_ANNUITY` - Annuity of the loan (also appears in Bureau Data)
- `AMT_GOODS_PRICE` - Price of goods for which credit was given
- `NAME_INCOME_TYPE` - Type of income
  - Working: 158,774 records
  - Commercial associate: 71,617 records
  - Pensioner: 55,362 records
  - State servant: 21,703 records
  - Unemployed: 22 records
  - Student: 18 records
  - Businessman: 10 records
  - Maternity leave: 5 records

#### 4. Contract & Assets (6 columns)
- `NAME_CONTRACT_TYPE` - Type of loan contract
  - Cash loans: 278,232 records (90.48%)
  - Revolving loans: 29,279 records (9.52%)
- `FLAG_OWN_CAR` - Flag if applicant owns a car
- `FLAG_OWN_REALTY` - Flag if applicant owns real estate
- `NAME_TYPE_SUITE` - Who accompanied applicant during application
- `NAME_HOUSING_TYPE` - Type of housing applicant occupies
- `ORGANIZATION_TYPE` - Type of organization applicant works for

#### 5. Contact Flags (6 columns)
- `FLAG_MOBIL` - Had a mobile phone
- `FLAG_EMP_PHONE` - Had an employer phone
- `FLAG_WORK_PHONE` - Had a work phone
- `FLAG_CONT_MOBILE` - Provided contact mobile phone
- `FLAG_PHONE` - Provided phone
- `FLAG_EMAIL` - Provided email

#### 6. Dates & Time (6 columns)
- `DAYS_EMPLOYED` - Number of days employed
  - Range: -17,912 to 365,243
  - **Data Quality Issue**: 55,374 records have placeholder value (365243)
  - Recommend treating as missing or separate category
  - No missing values but invalid data present
- `DAYS_REGISTRATION` - Days since registration
- `DAYS_ID_PUBLISH` - Days since ID was published
- `DAYS_LAST_PHONE_CHANGE` - Days since last phone change
- `WEEKDAY_APPR_PROCESS_START` - Weekday of application
- `HOUR_APPR_PROCESS_START` - Hour of application

#### 7. Geographic Information (10 columns)
- `REGION_POPULATION_RELATIVE` - Relative population in region
- `REGION_RATING_CLIENT` - Rating of region for applicant
- `REGION_RATING_CLIENT_W_CITY` - Rating with city consideration
- `REG_REGION_NOT_LIVE_REGION` - Flag: registration region ≠ living region
- `REG_REGION_NOT_WORK_REGION` - Flag: registration region ≠ work region
- `LIVE_REGION_NOT_WORK_REGION` - Flag: living region ≠ work region
- `REG_CITY_NOT_LIVE_CITY` - Flag: registration city ≠ living city
- `REG_CITY_NOT_WORK_CITY` - Flag: registration city ≠ work city
- `LIVE_CITY_NOT_WORK_CITY` - Flag: living city ≠ work city

#### 8. Building/Property Information (54 columns!)
Three statistical versions (AVG, MODE, MEDIAN) for each feature:
- `APARTMENTS_*` - Number of apartments in building
- `BASEMENTAREA_*` - Basement area
- `YEARS_BEGINEXPLUATATION_*` - Years since building started operation
- `YEARS_BUILD_*` - Years since building was built
- `COMMONAREA_*` - Common area size
- `ELEVATORS_*` - Number of elevators
- `ENTRANCES_*` - Number of entrances
- `FLOORSMAX_*` - Maximum floors
- `FLOORSMIN_*` - Minimum floors
- `LANDAREA_*` - Land area
- `LIVINGAPARTMENTS_*` - Number of living apartments
- `LIVINGAREA_*` - Living area
- `NONLIVINGAPARTMENTS_*` - Non-living apartments
- `NONLIVINGAREA_*` - Non-living area
- `FONDKAPREMONT_MODE` - Fund for capital repair
- `HOUSETYPE_MODE` - Type of house
- `TOTALAREA_MODE` - Total area
- `WALLSMATERIAL_MODE` - Wall material
- `EMERGENCYSTATE_MODE` - Emergency state flag

**Note**: These 54 features may be redundant and candidates for dimensionality reduction.

#### 9. Social Circle Information (4 columns)
- `OBS_30_CNT_SOCIAL_CIRCLE` - Number of observations in 30-day social circle
- `DEF_30_CNT_SOCIAL_CIRCLE` - Number of defaults in 30-day social circle
- `OBS_60_CNT_SOCIAL_CIRCLE` - Number of observations in 60-day social circle
- `DEF_60_CNT_SOCIAL_CIRCLE` - Number of defaults in 60-day social circle

#### 10. Document Submission Flags (20 columns)
- `FLAG_DOCUMENT_2` through `FLAG_DOCUMENT_21`
- Indicates whether applicant submitted specific document types

#### 11. Credit Bureau Inquiry Activity (6 columns)
- `AMT_REQ_CREDIT_BUREAU_HOUR` - Credit inquiries in last hour
- `AMT_REQ_CREDIT_BUREAU_DAY` - Credit inquiries in last day
- `AMT_REQ_CREDIT_BUREAU_WEEK` - Credit inquiries in last week
- `AMT_REQ_CREDIT_BUREAU_MON` - Credit inquiries in last month
- `AMT_REQ_CREDIT_BUREAU_QRT` - Credit inquiries in last quarter
- `AMT_REQ_CREDIT_BUREAU_YEAR` - Credit inquiries in last year

#### 12. External Data (1 column)
- `EXT_SOURCE_1` - External data source (likely third-party score/rating)

### Data Quality Summary - Application Data
- **No missing values** in critical filtering columns (SK_ID_CURR, DAYS_BIRTH, DAYS_EMPLOYED, AMT_INCOME_TOTAL, NAME_INCOME_TYPE, NAME_CONTRACT_TYPE, TARGET)
- **Data quality issue**: 55,374 records (18%) have suspicious `DAYS_EMPLOYED` value of 365,243 (likely missing data placeholder)
- **No duplicates** in SK_ID_CURR
- **Valid age range**: 20.5 to 69.1 years (all valid)
- **Outliers in income**: Maximum of 117,000,000 (100+ times the median)

### Fair Lending Concerns
The following variables may conflict with fair lending principles:
- `CODE_GENDER` - Direct gender discrimination risk
- `CNT_CHILDREN` - Family structure discrimination
- `NAME_FAMILY_STATUS` - Marital status discrimination
- `DAYS_BIRTH` / Age derived from DAYS_BIRTH - Age discrimination
- `OWN_CAR_AGE` - Indirect age proxy

---

## 2. Bureau Data Dataset

### Dataset Structure
- **File**: `IS453 Group Assignment - Bureau Data.csv`
- **Size**: 1,716,428 rows × 17 columns
- **Primary Keys**:
  - `SK_ID_CURR` - Links to Application Data
  - `SK_ID_BUREAU` - Unique credit record identifier
- **Relationship**: One-to-Many (one applicant has multiple credit records)
- **Unique customers**: 305,811
- **Average records per customer**: 5.61

### Column Details

#### 1. Identifiers (2 columns)
- `SK_ID_CURR` - Customer ID (links to Application Data)
  - 305,811 unique values
  - 0 missing values
- `SK_ID_BUREAU` - Unique identifier for each credit bureau record

#### 2. Credit Status (2 columns)
- `CREDIT_ACTIVE` - Status of the credit bureau reported credit
  - Values: "Active", "Closed", etc.
  - 0 missing values
- `CREDIT_TYPE` - Type of credit reported to bureau
  - Examples: Consumer credit, Credit card, Mortgage, Car loan, Micro loan, etc.
  - 0 missing values

#### 3. Credit Currency (1 column)
- `CREDIT_CURRENCY` - Recoded currency of the credit
  - 0 missing values

#### 4. Temporal Information (4 columns)
- `DAYS_CREDIT` - Number of days before current application when the credit was reported to CB
  - Range: negative values indicating days in the past
  - 0 missing values
- `DAYS_CREDIT_ENDDATE` - Remaining duration of CB credit in days at time of application
  - 0 missing values
- `DAYS_ENDDATE_FACT` - Days since CB credit ended at time of application
  - May contain missing values (NaN) for active credits
- `DAYS_CREDIT_UPDATE` - How many days before loan application was the CB credit last updated
  - 0 missing values

#### 5. Amount Information (5 columns)
- `AMT_CREDIT_SUM` - Current credit amount for the CB credit
  - Range: varies by credit type
  - 13 missing values (0.00%)
- `AMT_CREDIT_SUM_DEBT` - Current debt on CB credit
  - May contain missing values for certain credit types
- `AMT_CREDIT_SUM_LIMIT` - Current credit limit of credit card reported to CB
  - May be missing for non-card credits
- `AMT_CREDIT_SUM_OVERDUE` - Current amount overdue on CB credit
  - Typically 0 for non-delinquent accounts
- `AMT_ANNUITY` - Annuity of the CB credit (also appears in Application Data)
  - Links conceptually to application annuity

#### 6. Overdue Information (2 columns)
- `CREDIT_DAY_OVERDUE` - Number of days past due on CB credit at time of application
  - 0 for accounts not in default
- `AMT_CREDIT_MAX_OVERDUE` - Maximal amount overdue on the CB credit
  - Indicates maximum delinquency experienced

#### 7. Prolongation (1 column)
- `CNT_CREDIT_PROLONG` - How many times the CB credit was prolonged/extended
  - 0 if never extended

### Data Quality Summary - Bureau Data
- **Minimal missing values**: Only 13 missing in AMT_CREDIT_SUM (0.00%)
- **No missing SK_ID_CURR**: All records can be linked to Application Data
- **One-to-Many relationship**: Creates need for aggregation before merging with Application Data
- **Complete temporal data**: All DAYS_* fields have values, enabling time-series analysis

---

## 3. Data Dictionary

### Source
- **File**: `IS453 Group Assignment - Data Dict.xlsx`
- **Structure**: Two sheets (Application Data, Bureau Data)
- **Format**: Row, Description, Special columns

### Application Data Dictionary
Contains descriptions for all 120 columns:
- **SK_ID_CURR**: ID of loan in our sample
- **TARGET**: Target variable (1 = client with payment difficulties, 0 = no payment difficulties)
- **NAME_CONTRACT_TYPE**: Identification if loan is cash or revolving
- **CODE_GENDER**: Gender of the client
- **FLAG_OWN_CAR**: Flag if the client owns a car
- **FLAG_OWN_REALTY**: Flag if the client owns real estate
- **CNT_CHILDREN**: Number of children the client has
- **AMT_INCOME_TOTAL**: Total income of the client
- **AMT_CREDIT**: Credit amount of the loan
- **AMT_ANNUITY**: Annuity of the loan
- **AMT_GOODS_PRICE**: Price of goods for which credit was given
- [... and 108 more columns with detailed descriptions ...]
- **AMT_REQ_CREDIT_BUREAU_YEAR**: Number of enquiries to Credit Bureau about the client in last year

### Bureau Data Dictionary
Contains descriptions for all 17 columns:
- **SK_ID_CURR**: ID of loan in our sample - one loan in our sample file can have multiple records in this file
- **SK_ID_BUREAU**: Recoded ID of previous Credit Bureau credit record
- **CREDIT_ACTIVE**: Status of the Credit Bureau (CB) reported credits
- **CREDIT_CURRENCY**: Recoded currency of the Credit Bureau credit
- **DAYS_CREDIT**: How many days before current application did credit appear in CB
- **CREDIT_DAY_OVERDUE**: Number of days past due on CB credit at time of application
- **DAYS_CREDIT_ENDDATE**: Remaining duration of CB credit (in days) at time of application
- **DAYS_ENDDATE_FACT**: Days since CB credit ended at time of application
- **AMT_CREDIT_MAX_OVERDUE**: Maximal amount overdue on the Credit Bureau credit
- **CNT_CREDIT_PROLONG**: How many times was the Credit Bureau credit prolonged
- **AMT_CREDIT_SUM**: Current credit amount for the Credit Bureau credit
- **AMT_CREDIT_SUM_DEBT**: Current debt on Credit Bureau credit
- **AMT_CREDIT_SUM_LIMIT**: Current credit limit of credit card reported in Credit Bureau credit
- **AMT_CREDIT_SUM_OVERDUE**: Current amount overdue on Credit Bureau credit
- **CREDIT_TYPE**: Type of Credit Bureau credit (Car, cash, mortgage, etc.)
- **DAYS_CREDIT_UPDATE**: How many days before loan application did last update to Credit Bureau credit occur
- **AMT_ANNUITY**: Annuity of the Credit Bureau credit

---

## 4. Dataset Relationships

### Linking Keys
- **Primary link**: `SK_ID_CURR` (Customer ID)
- **Secondary link**: `AMT_ANNUITY` (appears in both, but not unique/reliable as primary key)

### Relationship Type
- **One-to-Many**: One Application record can have multiple Bureau records
- **Cardinality**:
  - Application Data: 307,511 unique customers
  - Bureau Data: 1,716,428 records for 305,811 unique customers
  - Average: 5.61 bureau records per customer

### Coverage Analysis
- **IDs in both datasets**: 263,491 customers (85.7% of applications)
- **IDs only in Application Data**: 44,020 customers (14.3% - no prior credit history)
- **IDs only in Bureau Data**: 42,320 customers (13.9% - likely removed from lending)
- **Total unique customers**: 305,811

### Merging Considerations
1. **One-to-Many join** requires aggregation of Bureau Data
2. **Missing bureau data** for 44,020 applicants (new to credit market)
3. **Unmatched bureau records** for 42,320 IDs (data quality issue)
4. **Aggregation options**:
   - Sum of amounts
   - Count of active/closed credits
   - Maximum overdue amounts
   - Average credit age
   - Recent activity flags

---

## 5. Analysis Findings from Notebook

### Dataset Size & Structure Verification
✓ Application Data: 307,511 rows × 120 columns
✓ Bureau Data: 1,716,428 rows × 17 columns
✓ Data Dictionary: 120 Application definitions + 17 Bureau definitions

### Lifelong Learning Loan Segment Analysis

#### Filtering Criteria Applied
1. **Age**: 25-55 years (DAYS_BIRTH: -20,075 to -9,125 days)
2. **Income**: ≤ $96,000 (AMT_INCOME_TOTAL)
3. **Income Type**: Working, Commercial associate, State servant, Unemployed, Maternity leave
4. **Contract Type**: Cash loans only
5. (Excludes employment duration filter in final analysis)

#### Filtering Results
- **Original dataset**: 307,511 applicants (8.07% default rate)
- **Filtered segment**: 31,125 applicants (10.12% retention rate)
- **Segment default rate**: 9.99% (higher risk than overall portfolio)
- **Risk difference**: +1.92 percentage points worse than population average

#### Individual Filter Impact
- **Age filter (25-55 years)**: 226,662 applicants (73.71% retention)
  - Bad rate: 8.71%
- **Income Type filter**: 252,116 applicants (81.99% retention)
  - Bad rate: 8.66%
- **Cash loans filter**: 278,232 applicants (90.48% retention)
  - Bad rate: 8.35%

### Data Quality Assessment
- **Critical**: 55,374 placeholder DAYS_EMPLOYED values (365,243 days)
- **Minor**: Income outliers (max: $117M vs median: $147K)
- **Positive**: No missing values in key filtering columns
- **Positive**: No duplicates in SK_ID_CURR

### Key Insights
1. **Lifelong learning target market is riskier** than general population by 1.92 percentage points
2. **Income cap of $96K** filters out 68% of applicants, suggesting this is a middle-income product
3. **Cash loans dominate** (90.48% of portfolio, 8.35% default rate vs 10.07% for revolving)
4. **Age diversity**: Valid range 20-69 years, but target segment filters to 25-55
5. **Prior credit**: 86% of applicants have bureau history; 14% are credit new-to-bank

---

## 6. Recommendations

### For Data Preparation
1. Handle DAYS_EMPLOYED placeholder values (55,374 records)
   - Option A: Treat as missing, impute using similar customers
   - Option B: Create separate "employment unknown" category
   - Option C: Exclude from employment-based filtering

2. Address income outliers
   - Cap at 99th percentile or use log transformation
   - Investigate records with >$1M annual income

3. Aggregate Bureau Data before merging
   - Sum credit amounts by status (active/closed)
   - Count delinquency instances
   - Calculate average days-since-credit
   - Flag recent credit inquiries

### For Fair Lending Compliance
1. Remove or carefully handle: `CODE_GENDER`, `DAYS_BIRTH` (if age-based)
2. Audit impact of: Family status, children count, occupation
3. Document any proxy variables (e.g., building characteristics correlating with protected classes)
4. Test for disparate impact across demographic groups

### For Model Development
1. Consider dimensionality reduction for 54 building features
2. Feature engineering opportunities:
   - Bureau aggregation metrics
   - Recency indicators from DAYS_* columns
   - Ratio features (debt-to-income, credit utilization)
3. Handle class imbalance (8.07% default rate)
4. Validate on lifelong learning segment separately

---

## 7. Quick Reference

### File Paths (Relative)
- Application data: `IS453 Group Assignment - Application Data.csv`
- Bureau data: `IS453 Group Assignment - Bureau Data.csv`
- Data dictionary: `IS453 Group Assignment - Data Dict.xlsx`

### Key Statistics
| Metric | Application | Bureau |
|--------|-------------|--------|
| Rows | 307,511 | 1,716,428 |
| Columns | 120 | 17 |
| Primary Key | SK_ID_CURR | SK_ID_CURR + SK_ID_BUREAU |
| Unique Customers | 307,511 | 305,811 |
| Default Rate | 8.07% | N/A (credit history) |
| Missing Values | 0 (critical) | 0 (critical) |

### Linking Column
- **SK_ID_CURR**: Present in both datasets, unique in Application Data, duplicated in Bureau Data (1-to-many)

### Target Segment (Lifelong Learning Loans)
- Size: 31,125 applicants (10.12% of population)
- Default rate: 9.99% (1.92 pp worse than overall)
- Profile: Ages 25-55, income ≤$96K, stable employment, cash loan preference

---

*Document generated from analysis of IS453 Group Assignment datasets*
*Last updated: Based on notebook execution results*
