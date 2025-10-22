# Mark Alecha - Data Engineering Projects

## Project 1: Dashboard Completeness Validation  

Financial accounting reconciliation dashboard for SPOT/FPSL/FSDM systems.

<img width="1268" height="697" alt="image" src="https://github.com/user-attachments/assets/f570f4b8-2d33-4694-9edc-3d49a04a2057" />


## Files
- `DEMO_Completeness_Validation_Dashboard_DummyDATA.xlsx` - Main dashboard file

## Purpose
Reconciliation and validation between SPOT, FSDM, and FPSL systems for various financial products including loans, derivatives, guarantees, and account products.

## Systems Integrated
- SPOT
- FSDM (Financial Services Data Model)
- FPSL (Financial Product Subledger)
- Arctis, Murex, SPEC source systems


### Script Architecture:
WITH multi_system_data AS (
-- Consolidate data from 4+ source systems
SELECT data FROM arctis_system
UNION ALL
SELECT data FROM bs_system
UNION ALL
SELECT data FROM murex_system
),
categorized_accruals AS (
-- Apply 50+ business rules for accounting treatment
SELECT *,
CASE
WHEN accrual_type = 'Interest' AND amount < 0 THEN 'InterestIncome'
WHEN accrual_type = 'Interest' AND amount > 0 THEN 'InterestExpense'
WHEN accrual_type = 'Fee' AND amount > 0 THEN 'FeeIncome'
-- 20+ additional accounting rules...
END AS accounting_category
FROM multi_system_data
),
validation_results AS (
-- Compare against target system and identify gaps
SELECT
source_data,
target_data,
CASE WHEN source_data IS NULL THEN 'Missing in Target'
WHEN target_data IS NULL THEN 'Missing in Source'
ELSE 'Matched' END AS validation_status
FROM categorized_accruals
FULL OUTER JOIN target_system ON key_fields
)
SELECT * FROM validation_results;

### Script Architecture:

WITH contract_data AS (
-- Extract and standardize contract information
SELECT
contract_id,
product_type,
account_attributes,
customer_segment,
regulatory_flags
FROM source_systems
WHERE reporting_date = :CUTOFF_DATE
),
product_categorization AS (
-- Implement hierarchical classification logic
SELECT *,
CASE
WHEN product_type = 'Loans' AND has_collateral = true
THEN 'Collateralized Loans'
WHEN product_type = 'Deposits' AND account_type = 'Term'
THEN 'Term Deposits'
WHEN product_type = 'Guarantees' AND guarantee_type = 'Performance'
THEN 'Performance Guarantees'
-- 25+ additional categorization rules...
END AS product_bucket,
CASE
WHEN product_bucket IN ('Loans', 'Guarantees') THEN 'Credit Risk'
WHEN product_bucket IN ('Deposits') THEN 'Liquidity Risk'
ELSE 'Market Risk'
END AS risk_category
FROM contract_data
),
final_mapping AS (
-- Map to target system and validate
SELECT
source.*,
target.product_catalog_code,
CASE WHEN target.product_catalog_code IS NULL
THEN 'Requires Mapping' ELSE 'Mapped' END AS mapping_status
FROM product_categorization source
LEFT JOIN target_catalog target ON business_keys
)
SELECT * FROM final_mapping;

### Technical Highlights:
- **Complex CTEs**: Multi-layer data transformation
- **Business Logic**: 50+ accounting rules implemented in SQL
- **Data Reconciliation**: Cross-system validation with gap analysis
- **Performance**: Handled 50,000+ monthly accounting entries

# Project 2: SQL Script Customizer

A powerful web-based tool for customizing SQL scripts with replacement rules and generating multiple script versions with meaningful filenames.

<img width="925" height="881" alt="image" src="https://github.com/user-attachments/assets/09d0bd43-943a-43e8-be9a-67daacf8802a" />


## Features

- 📁 Upload multiple SQL files
- 🔧 Define custom replacement rules
- 🎯 Generate all value combinations
- 📝 Preview generated scripts
- 💾 Bulk download functionality
- 🎨 Clean, responsive interface

## How to Use

1. Upload your SQL scripts
2. Define replacement rules (placeholder → values)
3. Set filename patterns
4. Process and download customized scripts

## Technologies Used

- HTML5
- CSS3
- JavaScript (ES6)
- No external dependencies - works offline!

# Project 2: Financial Accrual Data Transformation ETL

## Overview
This ETL script transforms financial accrual data from source systems into a standardized FSDM (Financial Services Data Model) format for accounting and reporting purposes.

## Key Features
- **Multi-currency accrual processing** for various financial instruments
- **Accounting system differentiation** (UGB vs IFRS)
- **Comprehensive fee and interest categorization**
- **Temporal data handling** with business validity periods
- **Data quality enforcement** with null/zero value filtering

## Technical Highlights
- **Template-based architecture** using embedded base templates
- **CTE (Common Table Expression)** for optimized data processing
- **Union-based multi-category processing** for different accrual types
- **Parameterized configuration** for environment flexibility

## Accrual Types Processed
- Interest (Debit/Credit/Overdraft)
- Various fee types (Bonus, Loan Commission, Management Fees, etc.)
- Provision calculations
- Expense accruals

## Database Schema
- **Source**: `GESCHAEFT`, `GESCHAEFT_BASIS`, `GESCHAEFT_ABGRENZUNGSWERTE`
- **Target**: FSDM accrual model with financial contract mapping
## Data Flow
1. **Extract**: Join multiple source tables with active record filtering
2. **Transform**: 
   - Categorize accounting systems (UGB/IFRS)
   - Apply business logic multipliers (-1 for correct accounting)
   - Map to standardized accrual types and categories
3. **Load**: Insert into target FSDM table with proper typing

## Key Transformations
- Amount sign correction for accounting standards
- Conditional accrual type mapping
- Currency and temporal handling
- Source system tracking

| Field | Type | Description | Business Logic |
|-------|------|-------------|----------------|
| ACCRUAL_TYPE | VARCHAR(128) | InterestIncome/InterestExpense/FeeIncome/FeeExpense | Conditional mapping based on amount sign |
| AMOUNT_IN_PAYMENT_CURRENCY | DECIMAL(31,6) | Transformed accrual amount | Multiplied by -1 for correct accounting |
| BUSINESS_VALID_FROM | DATE | Accrual effective date | From BUCHUNGSDATUM |
| CONDITION_SUBTYPE | VARCHAR(100) | Detailed fee/interest category | Business-specific mapping logic |



## 📫 Contact
- Email: nomerzonalecha1@gmail.com
- Phone: 09760180051

*Note: Actual table names, proprietary business rules, and client-specific logic have been replaced with generic examples to protect confidentiality.*
