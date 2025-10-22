# Mark Alecha - Data Engineering Portfolio

## Dashboard Completeness Validation  

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


### Sample Script Architecture:
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

### Sample Script Architecture:

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

# SQL Script Customizer

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

# Financial Transformation ETL

## Overview
This ETL script transforms financial data from source systems into a standardized FSDM (Financial Services Data Model) format for accounting and reporting purposes.

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

/*
Sample Financial Accrual ETL Transformation
Purpose: Transform banking transaction data into standardized accrual accounting records
Author: Your Name
Domain: Financial Services - Accounting & Regulatory Reporting
Dependencies: FSDM base template, Product Mapping table
*/
ID: /fsdm/transform/FSDM/Accrual_ARCTIS
TEMPLATE: |

        {% setIfNotDefined case_sensitive = true %}
        {% embed "/fsdm/transform/base" %}
        {% block statements %}
        INSERT INTO {{work.schema | format(case_sensitive ? '"%s"' : '%s')}}.{{work.table | format(case_sensitive ? '"%s"' : '%s')}} 
        (
        "FSDM_FILLING_SID", 
        "ACCOUNTING_SYSTEM_ACCOUNTING_SYSTEM_ID", 
        "FINANCIAL_CONTRACT_FINANCIAL_CONTRACT_ID", 
        "FINANCIAL_INSTRUMENT_FINANCIAL_INSTRUMENT_ID", 
        "FINANCIAL_INSTRUMENT_ACCOUNT_FINANCIAL_CONTRACT_ID", 
        "ACCOUNTING_CHANGE_REASON",  
        "ACCOUNTING_CHANGE_SEQUENCE_NUMBER", 
        "ACCRUAL_CATEGORY", 
        "ACCRUAL_TYPE", 
        "AMOUNT_IN_PAYMENT_CURRENCY", 
        "AMOUNT_IN_POSITION_CURRENCY", 
        "BANK", 
        "CLIENT", 
        "AUTH_OWNER", 
        "BUSINESS_VALID_FROM", 
        "BUSINESS_VALID_TO", 
        "CONDITION_SUBTYPE", 
        "CONDITION_TYPE", 
        "LOT_ID", 
        "MULTICURRENCY_CONTRACT_POSITION_CURRENCY", 
        "NOMINAL_AMOUNT_CURRENCY", 
        "ORIGINATING_SOURCE_SYSTEM", 
        "PAYMENT_CURRENCY", 
        "POSITION_CURRENCY", 
        "RECORD_SOURCE_ID", 
        "ROLE_OF_PAYER", 
        "RESULT_GROUP_RESULT_DATA_PROVIDER", 
        "RESULT_GROUP_RESULT_GROUP_ID"
        )
        (
        WITH 
        cte_Result AS (
        SELECT gb.WCD
              ,'{{defaultSpot}}'AS RecordSourceID
              ,g.SVC_QUELLSYSTEM AS OriginatingSourceSystem
              ,UPPER(ga.GESCHAEFT_SID) AS GESCHAEFT_SID
              ,CASE WHEN COALESCE(TRIM(gb.BILANZIERUNG_KZ), 'U')  = 'U' THEN 'UGB'
                    WHEN COALESCE(TRIM(gb.BILANZIERUNG_KZ), 'I') IN ('9', '3', 'I', '-') THEN 'IFRS'
                    END AS AccSystemID
              ,ga.BUCHUNGSDATUM
              ,ga.SOLLZINSEN_ABGEGRENZT
              ,ga.HABENZINSEN_ABGEGRENZT
              ,ga.BONUS_ABGEGRENZT
              ,ga.VERZUG_UEBERZIEHUNGSZINSEN_ABGEGRENZT
              ,ga.KREDITPROVISION_ABGEGRENZT
              ,ga.GESTIONSPROVISION_ABGEGRENZT
              ,ga.BEREITSTELLUNGSPROVISION_ABGEGRENZT
              ,ga.MANIPULATIONSGEBUEHR_ABGEGRENZT
              ,ga.UMSATZPROVISION_ABGEGRENZT
              ,ga.HAFTUNGSPROVISION_ABGEGRENZT
              ,ga.SPESEN_ABGEGRENZT
              ,ga.UEBERZIEHUNGSZINSEN_VERBRIEFTER_RAHMEN_ABGEGRENZT
        FROM {{cdwhSchema}}.GESCHAEFT g
        INNER JOIN {{cdwhSchema}}.GESCHAEFT_BASIS gb
        ON  gb.CDWH_FILLING_SID = g.CDWH_FILLING_SID
        AND gb.GESCHAEFT_SID    = g.GESCHAEFT_SID
        AND gb.SVC_KORREKTUR_ID = 0
        AND gb.SVC_AKTIV        = TRUE
        INNER JOIN {{cdwhSchema}}.GESCHAEFT_ABGRENZUNGSWERTE ga
        ON  ga.CDWH_FILLING_SID = g.CDWH_FILLING_SID
        AND ga.GESCHAEFT_SID    = g.GESCHAEFT_SID
        AND ga.SVC_KORREKTUR_ID = 0
        AND ga.SVC_AKTIV        = TRUE
        INNER JOIN {{fsdmSchema}}."PRODUCT_MAPPING" pm
        ON pm."GESCHAEFT_SID" = g.GESCHAEFT_SID
        AND pm."BANK" = g.BANK
        AND pm."FSDM_FILLING_SID" = '{{fsdm_filling.sid}}'
        WHERE g.CDWH_FILLING_SID = '{{cdwh_filling.sid}}'
        AND g.SVC_AKTIV = TRUE
        AND g.SVC_KORREKTUR_ID = 0
        )
        -- For SOLLZINSEN_ABGEGRENZT
        SELECT
        '{{fsdm_filling.sid}}' AS "FSDM_FILLING_SID",
        CAST(AccSystemID AS VARCHAR(128)) AS "ACCOUNTING_SYSTEM_ACCOUNTING_SYSTEM_ID",
        CAST(GESCHAEFT_SID AS VARCHAR(40)) AS "FINANCIAL_CONTRACT_FINANCIAL_CONTRACT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_FINANCIAL_INSTRUMENT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_ACCOUNT_FINANCIAL_CONTRACT_ID",
        CAST('TimeEffect' AS VARCHAR(100)) AS "ACCOUNTING_CHANGE_REASON",
        CAST(1 AS DECIMAL(10,0)) AS "ACCOUNTING_CHANGE_SEQUENCE_NUMBER",
        CAST(' ' AS VARCHAR(100)) AS "ACCRUAL_CATEGORY",
        CAST(CASE WHEN SOLLZINSEN_ABGEGRENZT * (-1) > 0 THEN 'InterestIncome'
                  WHEN SOLLZINSEN_ABGEGRENZT * (-1) < 0 THEN 'InterestExpense'
                  ELSE ' '
             END AS VARCHAR(128)) AS "ACCRUAL_TYPE",
        CAST(SOLLZINSEN_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_PAYMENT_CURRENCY",
        CAST(SOLLZINSEN_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_POSITION_CURRENCY",
        CAST('{{bank}}' AS DECIMAL(3,0)) AS "BANK",
        CAST('{{client}}' AS VARCHAR(3)) AS "CLIENT",
        CAST('{{authorizationGroup}}' AS VARCHAR(10)) AS "AUTH_OWNER",
        CAST(BUCHUNGSDATUM AS DATE) AS "BUSINESS_VALID_FROM",
        CAST('9999-12-31' AS DATE) AS "BUSINESS_VALID_TO",
        CAST(CASE WHEN SOLLZINSEN_ABGEGRENZT * (-1) > 0 THEN 'C_DebitInterest'
                  WHEN SOLLZINSEN_ABGEGRENZT * (-1) < 0 THEN 'C_NegativeDebitInterest'
                  ELSE ' '
             END AS VARCHAR(100)) AS "CONDITION_SUBTYPE",
        CAST('C_Interest' AS VARCHAR(40)) AS "CONDITION_TYPE",
        CAST(' ' AS VARCHAR(128)) AS "LOT_ID",
        CAST(' ' AS VARCHAR(3)) AS "MULTICURRENCY_CONTRACT_POSITION_CURRENCY",
        CAST(' ' AS VARCHAR(3)) AS "NOMINAL_AMOUNT_CURRENCY",
        CAST(OriginatingSourceSystem AS VARCHAR(10)) AS "ORIGINATING_SOURCE_SYSTEM",
        CAST(WCD AS VARCHAR(3)) AS "PAYMENT_CURRENCY",
        CAST(WCD AS VARCHAR(3)) AS "POSITION_CURRENCY",
        CAST(RecordSourceID AS VARCHAR(128)) AS "RECORD_SOURCE_ID",
        CAST(' ' AS VARCHAR(50)) AS "ROLE_OF_PAYER",
        CAST(' ' AS VARCHAR(256)) AS "RESULT_GROUP_RESULT_DATA_PROVIDER",
        CAST(' ' AS VARCHAR(128)) AS "RESULT_GROUP_RESULT_GROUP_ID"
        FROM cte_Result
        WHERE SOLLZINSEN_ABGEGRENZT NOT IN (NULL, 0)
        UNION ALL
        -- For HABENZINSEN_ABGEGRENZT
        SELECT
        '{{fsdm_filling.sid}}' AS "FSDM_FILLING_SID",
        CAST(AccSystemID AS VARCHAR(128)) AS "ACCOUNTING_SYSTEM_ACCOUNTING_SYSTEM_ID",
        CAST(GESCHAEFT_SID AS VARCHAR(40)) AS "FINANCIAL_CONTRACT_FINANCIAL_CONTRACT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_FINANCIAL_INSTRUMENT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_ACCOUNT_FINANCIAL_CONTRACT_ID",
        CAST('TimeEffect' AS VARCHAR(100)) AS "ACCOUNTING_CHANGE_REASON",
        CAST(1 AS DECIMAL(10,0)) AS "ACCOUNTING_CHANGE_SEQUENCE_NUMBER",
        CAST(' ' AS VARCHAR(100)) AS "ACCRUAL_CATEGORY",
        CAST(CASE WHEN HABENZINSEN_ABGEGRENZT * (-1) > 0 THEN 'InterestIncome'
                  WHEN HABENZINSEN_ABGEGRENZT * (-1) < 0 THEN 'InterestExpense'
                  ELSE ' '
             END AS VARCHAR(128)) AS "ACCRUAL_TYPE",
        CAST(HABENZINSEN_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_PAYMENT_CURRENCY",
        CAST(HABENZINSEN_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_POSITION_CURRENCY",
        CAST('{{bank}}' AS DECIMAL(3,0)) AS "BANK",
        CAST('{{client}}' AS VARCHAR(3)) AS "CLIENT",
        CAST('{{authorizationGroup}}' AS VARCHAR(10)) AS "AUTH_OWNER",
        CAST(BUCHUNGSDATUM AS DATE) AS "BUSINESS_VALID_FROM",
        CAST('9999-12-31' AS DATE) AS "BUSINESS_VALID_TO",
        CAST(CASE WHEN HABENZINSEN_ABGEGRENZT * (-1) > 0 THEN 'C_NegativeCreditInterest'
                  WHEN HABENZINSEN_ABGEGRENZT * (-1) < 0 THEN 'C_CreditInterest'
                  ELSE ' '
             END AS VARCHAR(100)) AS "CONDITION_SUBTYPE",
        CAST('C_Interest' AS VARCHAR(40)) AS "CONDITION_TYPE",
        CAST(' ' AS VARCHAR(128)) AS "LOT_ID",
        CAST(' ' AS VARCHAR(3)) AS "MULTICURRENCY_CONTRACT_POSITION_CURRENCY",
        CAST(' ' AS VARCHAR(3)) AS "NOMINAL_AMOUNT_CURRENCY",
        CAST(OriginatingSourceSystem AS VARCHAR(10)) AS "ORIGINATING_SOURCE_SYSTEM",
        CAST(WCD AS VARCHAR(3)) AS "PAYMENT_CURRENCY",
        CAST(WCD AS VARCHAR(3)) AS "POSITION_CURRENCY",
        CAST(RecordSourceID AS VARCHAR(128)) AS "RECORD_SOURCE_ID",
        CAST(' ' AS VARCHAR(50)) AS "ROLE_OF_PAYER",
        CAST(' ' AS VARCHAR(256)) AS "RESULT_GROUP_RESULT_DATA_PROVIDER",
        CAST(' ' AS VARCHAR(128)) AS "RESULT_GROUP_RESULT_GROUP_ID"
        FROM cte_Result
        WHERE HABENZINSEN_ABGEGRENZT NOT IN (NULL, 0)
        UNION ALL
        -- For BONUS_ABGEGRENZT
        SELECT
        '{{fsdm_filling.sid}}' AS "FSDM_FILLING_SID",
        CAST(AccSystemID AS VARCHAR(128)) AS "ACCOUNTING_SYSTEM_ACCOUNTING_SYSTEM_ID",
        CAST(GESCHAEFT_SID AS VARCHAR(40)) AS "FINANCIAL_CONTRACT_FINANCIAL_CONTRACT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_FINANCIAL_INSTRUMENT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_ACCOUNT_FINANCIAL_CONTRACT_ID",
        CAST('TimeEffect' AS VARCHAR(100)) AS "ACCOUNTING_CHANGE_REASON",
        CAST(1 AS DECIMAL(10,0)) AS "ACCOUNTING_CHANGE_SEQUENCE_NUMBER",
        CAST(' ' AS VARCHAR(100)) AS "ACCRUAL_CATEGORY",
        CAST(CASE WHEN BONUS_ABGEGRENZT * (-1) > 0 THEN 'FeeIncome'
                  WHEN BONUS_ABGEGRENZT * (-1) < 0 THEN 'FeeExpense'
                  ELSE ' '
             END AS VARCHAR(128)) AS "ACCRUAL_TYPE",
        CAST(BONUS_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_PAYMENT_CURRENCY",
        CAST(BONUS_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_POSITION_CURRENCY",
        CAST('{{bank}}' AS DECIMAL(3,0)) AS "BANK",
        CAST('{{client}}' AS VARCHAR(3)) AS "CLIENT",
        CAST('{{authorizationGroup}}' AS VARCHAR(10)) AS "AUTH_OWNER",
        CAST(BUCHUNGSDATUM AS DATE) AS "BUSINESS_VALID_FROM",
        CAST('9999-12-31' AS DATE) AS "BUSINESS_VALID_TO",
        CAST(CASE WHEN BONUS_ABGEGRENZT * (-1) > 0 THEN 'C_BonusRequested'
                  WHEN BONUS_ABGEGRENZT * (-1) < 0 THEN 'C_BonusToBePaid'
                  ELSE ' '
             END AS VARCHAR(100)) AS "CONDITION_SUBTYPE",
        CAST('C_Fee' AS VARCHAR(40)) AS "CONDITION_TYPE",
        CAST(' ' AS VARCHAR(128)) AS "LOT_ID",
        CAST(' ' AS VARCHAR(3)) AS "MULTICURRENCY_CONTRACT_POSITION_CURRENCY",
        CAST(' ' AS VARCHAR(3)) AS "NOMINAL_AMOUNT_CURRENCY",
        CAST(OriginatingSourceSystem AS VARCHAR(10)) AS "ORIGINATING_SOURCE_SYSTEM",
        CAST(WCD AS VARCHAR(3)) AS "PAYMENT_CURRENCY",
        CAST(WCD AS VARCHAR(3)) AS "POSITION_CURRENCY",
        CAST(RecordSourceID AS VARCHAR(128)) AS "RECORD_SOURCE_ID",
        CAST(' ' AS VARCHAR(50)) AS "ROLE_OF_PAYER",
        CAST(' ' AS VARCHAR(256)) AS "RESULT_GROUP_RESULT_DATA_PROVIDER",
        CAST(' ' AS VARCHAR(128)) AS "RESULT_GROUP_RESULT_GROUP_ID"
        FROM cte_Result
        WHERE BONUS_ABGEGRENZT NOT IN (NULL, 0)
        UNION ALL
        -- For VERZUG_UEBERZIEHUNGSZINSEN_ABGRENZT
        SELECT
        '{{fsdm_filling.sid}}' AS "FSDM_FILLING_SID",
        CAST(AccSystemID AS VARCHAR(128)) AS "ACCOUNTING_SYSTEM_ACCOUNTING_SYSTEM_ID",
        CAST(GESCHAEFT_SID AS VARCHAR(40)) AS "FINANCIAL_CONTRACT_FINANCIAL_CONTRACT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_FINANCIAL_INSTRUMENT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_ACCOUNT_FINANCIAL_CONTRACT_ID",
        CAST('TimeEffect' AS VARCHAR(100)) AS "ACCOUNTING_CHANGE_REASON",
        CAST(1 AS DECIMAL(10,0)) AS "ACCOUNTING_CHANGE_SEQUENCE_NUMBER",
        CAST(' ' AS VARCHAR(100)) AS "ACCRUAL_CATEGORY",
        CAST(CASE WHEN VERZUG_UEBERZIEHUNGSZINSEN_ABGEGRENZT * (-1) > 0 THEN 'InterestIncome'
                  WHEN VERZUG_UEBERZIEHUNGSZINSEN_ABGEGRENZT * (-1) < 0 THEN 'InterestExpense'
                  ELSE ' '
             END AS VARCHAR(128)) AS "ACCRUAL_TYPE",
        CAST(VERZUG_UEBERZIEHUNGSZINSEN_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_PAYMENT_CURRENCY",
        CAST(VERZUG_UEBERZIEHUNGSZINSEN_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_POSITION_CURRENCY",
        CAST('{{bank}}' AS DECIMAL(3,0)) AS "BANK",
        CAST('{{client}}' AS VARCHAR(3)) AS "CLIENT",
        CAST('{{authorizationGroup}}' AS VARCHAR(10)) AS "AUTH_OWNER",
        CAST(BUCHUNGSDATUM AS DATE) AS "BUSINESS_VALID_FROM",
        CAST('9999-12-31' AS DATE) AS "BUSINESS_VALID_TO",
        CAST(CASE WHEN VERZUG_UEBERZIEHUNGSZINSEN_ABGEGRENZT * (-1) > 0 THEN 'C_DefaultOverdraftInterestRequested'
                  WHEN VERZUG_UEBERZIEHUNGSZINSEN_ABGEGRENZT * (-1) < 0 THEN 'C_DefaultOverdraftInterestToBePaid'
                  ELSE ' '
             END AS VARCHAR(100)) AS "CONDITION_SUBTYPE",
        CAST('C_Interest' AS VARCHAR(40)) AS "CONDITION_TYPE",
        CAST(' ' AS VARCHAR(128)) AS "LOT_ID",
        CAST(' ' AS VARCHAR(3)) AS "MULTICURRENCY_CONTRACT_POSITION_CURRENCY",
        CAST(' ' AS VARCHAR(3)) AS "NOMINAL_AMOUNT_CURRENCY",
        CAST(OriginatingSourceSystem AS VARCHAR(10)) AS "ORIGINATING_SOURCE_SYSTEM",
        CAST(WCD AS VARCHAR(3)) AS "PAYMENT_CURRENCY",
        CAST(WCD AS VARCHAR(3)) AS "POSITION_CURRENCY",
        CAST(RecordSourceID AS VARCHAR(128)) AS "RECORD_SOURCE_ID",
        CAST(' ' AS VARCHAR(50)) AS "ROLE_OF_PAYER",
        CAST(' ' AS VARCHAR(256)) AS "RESULT_GROUP_RESULT_DATA_PROVIDER",
        CAST(' ' AS VARCHAR(128)) AS "RESULT_GROUP_RESULT_GROUP_ID"
        FROM cte_Result
        WHERE VERZUG_UEBERZIEHUNGSZINSEN_ABGEGRENZT NOT IN (NULL, 0)
        UNION ALL
        -- For KREDITPROVISION_ABGEGRENZT 
        SELECT
        '{{fsdm_filling.sid}}' AS "FSDM_FILLING_SID",
        CAST(AccSystemID AS VARCHAR(128)) AS "ACCOUNTING_SYSTEM_ACCOUNTING_SYSTEM_ID",
        CAST(GESCHAEFT_SID AS VARCHAR(40)) AS "FINANCIAL_CONTRACT_FINANCIAL_CONTRACT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_FINANCIAL_INSTRUMENT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_ACCOUNT_FINANCIAL_CONTRACT_ID",
        CAST('TimeEffect' AS VARCHAR(100)) AS "ACCOUNTING_CHANGE_REASON",
        CAST(1 AS DECIMAL(10,0)) AS "ACCOUNTING_CHANGE_SEQUENCE_NUMBER",
        CAST(' ' AS VARCHAR(100)) AS "ACCRUAL_CATEGORY",
        CAST(CASE WHEN KREDITPROVISION_ABGEGRENZT > 0 THEN 'FeeIncome'
                  WHEN KREDITPROVISION_ABGEGRENZT < 0 THEN 'FeeExpense'
                  ELSE ' '
             END AS VARCHAR(128)) AS "ACCRUAL_TYPE",
        CAST(KREDITPROVISION_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_PAYMENT_CURRENCY",
        CAST(KREDITPROVISION_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_POSITION_CURRENCY",
        CAST('{{bank}}' AS DECIMAL(3,0)) AS "BANK",
        CAST('{{client}}' AS VARCHAR(3)) AS "CLIENT",
        CAST('{{authorizationGroup}}' AS VARCHAR(10)) AS "AUTH_OWNER",
        CAST(BUCHUNGSDATUM AS DATE) AS "BUSINESS_VALID_FROM",
        CAST('9999-12-31' AS DATE) AS "BUSINESS_VALID_TO",
        CAST(CASE WHEN KREDITPROVISION_ABGEGRENZT > 0 THEN 'C_LoanCommissionRequested'
                  WHEN KREDITPROVISION_ABGEGRENZT < 0 THEN 'C_LoanCommissionToBePaid'
                  ELSE ' '
             END AS VARCHAR(100)) AS "CONDITION_SUBTYPE",
        CAST('C_Fee' AS VARCHAR(40)) AS "CONDITION_TYPE",
        CAST(' ' AS VARCHAR(128)) AS "LOT_ID",
        CAST(' ' AS VARCHAR(3)) AS "MULTICURRENCY_CONTRACT_POSITION_CURRENCY",
        CAST(' ' AS VARCHAR(3)) AS "NOMINAL_AMOUNT_CURRENCY",
        CAST(OriginatingSourceSystem AS VARCHAR(10)) AS "ORIGINATING_SOURCE_SYSTEM",
        CAST(WCD AS VARCHAR(3)) AS "PAYMENT_CURRENCY",
        CAST(WCD AS VARCHAR(3)) AS "POSITION_CURRENCY",
        CAST(RecordSourceID AS VARCHAR(128)) AS "RECORD_SOURCE_ID",
        CAST(' ' AS VARCHAR(50)) AS "ROLE_OF_PAYER",
        CAST(' ' AS VARCHAR(256)) AS "RESULT_GROUP_RESULT_DATA_PROVIDER",
        CAST(' ' AS VARCHAR(128)) AS "RESULT_GROUP_RESULT_GROUP_ID"
        FROM cte_Result
        WHERE KREDITPROVISION_ABGEGRENZT NOT IN (NULL, 0)
        UNION ALL
        -- For GESTIONSPROVISION_ABGEGRENZT
        SELECT
        '{{fsdm_filling.sid}}' AS "FSDM_FILLING_SID",
        CAST(AccSystemID AS VARCHAR(128)) AS "ACCOUNTING_SYSTEM_ACCOUNTING_SYSTEM_ID",
        CAST(GESCHAEFT_SID AS VARCHAR(40)) AS "FINANCIAL_CONTRACT_FINANCIAL_CONTRACT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_FINANCIAL_INSTRUMENT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_ACCOUNT_FINANCIAL_CONTRACT_ID",
        CAST('TimeEffect' AS VARCHAR(100)) AS "ACCOUNTING_CHANGE_REASON",
        CAST(1 AS DECIMAL(10,0)) AS "ACCOUNTING_CHANGE_SEQUENCE_NUMBER",
        CAST(' ' AS VARCHAR(100)) AS "ACCRUAL_CATEGORY",
        CAST(CASE WHEN GESTIONSPROVISION_ABGEGRENZT > 0 THEN 'FeeIncome'
                  WHEN GESTIONSPROVISION_ABGEGRENZT < 0 THEN 'FeeExpense'
                  ELSE ' '
             END AS VARCHAR(128)) AS "ACCRUAL_TYPE",
        CAST(GESTIONSPROVISION_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_PAYMENT_CURRENCY",
        CAST(GESTIONSPROVISION_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_POSITION_CURRENCY",
        CAST('{{bank}}' AS DECIMAL(3,0)) AS "BANK",
        CAST('{{client}}' AS VARCHAR(3)) AS "CLIENT",
        CAST('{{authorizationGroup}}' AS VARCHAR(10)) AS "AUTH_OWNER",
        CAST(BUCHUNGSDATUM AS DATE) AS "BUSINESS_VALID_FROM",
        CAST('9999-12-31' AS DATE) AS "BUSINESS_VALID_TO",
        CAST(CASE WHEN GESTIONSPROVISION_ABGEGRENZT > 0 THEN 'C_ManagementCommissionRequested'
                  WHEN GESTIONSPROVISION_ABGEGRENZT < 0 THEN 'C_ManagementCommissionToBePaid'
                  ELSE ' '
             END AS VARCHAR(100)) AS "CONDITION_SUBTYPE",
        CAST('C_Fee' AS VARCHAR(40)) AS "CONDITION_TYPE",
        CAST(' ' AS VARCHAR(128)) AS "LOT_ID",
        CAST(' ' AS VARCHAR(3)) AS "MULTICURRENCY_CONTRACT_POSITION_CURRENCY",
        CAST(' ' AS VARCHAR(3)) AS "NOMINAL_AMOUNT_CURRENCY",
        CAST(OriginatingSourceSystem AS VARCHAR(10)) AS "ORIGINATING_SOURCE_SYSTEM",
        CAST(WCD AS VARCHAR(3)) AS "PAYMENT_CURRENCY",
        CAST(WCD AS VARCHAR(3)) AS "POSITION_CURRENCY",
        CAST(RecordSourceID AS VARCHAR(128)) AS "RECORD_SOURCE_ID",
        CAST(' ' AS VARCHAR(50)) AS "ROLE_OF_PAYER",
        CAST(' ' AS VARCHAR(256)) AS "RESULT_GROUP_RESULT_DATA_PROVIDER",
        CAST(' ' AS VARCHAR(128)) AS "RESULT_GROUP_RESULT_GROUP_ID"
        FROM cte_Result
        WHERE GESTIONSPROVISION_ABGEGRENZT NOT IN (NULL, 0)
        UNION ALL
        -- For BEREITSTELLUNGSPROVISION_ABGRENZT
        SELECT
        '{{fsdm_filling.sid}}' AS "FSDM_FILLING_SID",
        CAST(AccSystemID AS VARCHAR(128)) AS "ACCOUNTING_SYSTEM_ACCOUNTING_SYSTEM_ID",
        CAST(GESCHAEFT_SID AS VARCHAR(40)) AS "FINANCIAL_CONTRACT_FINANCIAL_CONTRACT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_FINANCIAL_INSTRUMENT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_ACCOUNT_FINANCIAL_CONTRACT_ID",
        CAST('TimeEffect' AS VARCHAR(100)) AS "ACCOUNTING_CHANGE_REASON",
        CAST(1 AS DECIMAL(10,0)) AS "ACCOUNTING_CHANGE_SEQUENCE_NUMBER",
        CAST(' ' AS VARCHAR(100)) AS "ACCRUAL_CATEGORY",
        CAST(CASE WHEN BEREITSTELLUNGSPROVISION_ABGEGRENZT > 0 THEN 'FeeIncome'
                  WHEN BEREITSTELLUNGSPROVISION_ABGEGRENZT < 0 THEN 'FeeExpense'
                  ELSE ' '
             END AS VARCHAR(128)) AS "ACCRUAL_TYPE",
        CAST(BEREITSTELLUNGSPROVISION_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_PAYMENT_CURRENCY",
        CAST(BEREITSTELLUNGSPROVISION_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_POSITION_CURRENCY",
        CAST('{{bank}}' AS DECIMAL(3,0)) AS "BANK",
        CAST('{{client}}' AS VARCHAR(3)) AS "CLIENT",
        CAST('{{authorizationGroup}}' AS VARCHAR(10)) AS "AUTH_OWNER",
        CAST(BUCHUNGSDATUM AS DATE) AS "BUSINESS_VALID_FROM",
        CAST('9999-12-31' AS DATE) AS "BUSINESS_VALID_TO",
        CAST(CASE WHEN BEREITSTELLUNGSPROVISION_ABGEGRENZT > 0 THEN 'C_CommitmentFeeRequested'
                  WHEN BEREITSTELLUNGSPROVISION_ABGEGRENZT < 0 THEN 'C_CommitmentFeeToBePaid'
                  ELSE ' '
             END AS VARCHAR(100)) AS "CONDITION_SUBTYPE",
        CAST('C_Fee' AS VARCHAR(40)) AS "CONDITION_TYPE",
        CAST(' ' AS VARCHAR(128)) AS "LOT_ID",
        CAST(' ' AS VARCHAR(3)) AS "MULTICURRENCY_CONTRACT_POSITION_CURRENCY",
        CAST(' ' AS VARCHAR(3)) AS "NOMINAL_AMOUNT_CURRENCY",
        CAST(OriginatingSourceSystem AS VARCHAR(10)) AS "ORIGINATING_SOURCE_SYSTEM",
        CAST(WCD AS VARCHAR(3)) AS "PAYMENT_CURRENCY",
        CAST(WCD AS VARCHAR(3)) AS "POSITION_CURRENCY",
        CAST(RecordSourceID AS VARCHAR(128)) AS "RECORD_SOURCE_ID",
        CAST(' ' AS VARCHAR(50)) AS "ROLE_OF_PAYER",
        CAST(' ' AS VARCHAR(256)) AS "RESULT_GROUP_RESULT_DATA_PROVIDER",
        CAST(' ' AS VARCHAR(128)) AS "RESULT_GROUP_RESULT_GROUP_ID"
        FROM cte_Result
        WHERE BEREITSTELLUNGSPROVISION_ABGEGRENZT NOT IN (NULL, 0)
        UNION ALL
        -- For MANIPULATIONSGEBUEHR_ABGEGRENZT
        SELECT
        '{{fsdm_filling.sid}}' AS "FSDM_FILLING_SID",
        CAST(AccSystemID AS VARCHAR(128)) AS "ACCOUNTING_SYSTEM_ACCOUNTING_SYSTEM_ID",
        CAST(GESCHAEFT_SID AS VARCHAR(40)) AS "FINANCIAL_CONTRACT_FINANCIAL_CONTRACT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_FINANCIAL_INSTRUMENT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_ACCOUNT_FINANCIAL_CONTRACT_ID",
        CAST('TimeEffect' AS VARCHAR(100)) AS "ACCOUNTING_CHANGE_REASON",
        CAST(1 AS DECIMAL(10,0)) AS "ACCOUNTING_CHANGE_SEQUENCE_NUMBER",
        CAST(' ' AS VARCHAR(100)) AS "ACCRUAL_CATEGORY",
        CAST(CASE WHEN MANIPULATIONSGEBUEHR_ABGEGRENZT > 0 THEN 'FeeIncome'
                  WHEN MANIPULATIONSGEBUEHR_ABGEGRENZT < 0 THEN 'FeeExpense'
                  ELSE ' '
             END AS VARCHAR(128)) AS "ACCRUAL_TYPE",
        CAST(MANIPULATIONSGEBUEHR_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_PAYMENT_CURRENCY",
        CAST(MANIPULATIONSGEBUEHR_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_POSITION_CURRENCY",
        CAST('{{bank}}' AS DECIMAL(3,0)) AS "BANK",
        CAST('{{client}}' AS VARCHAR(3)) AS "CLIENT",
        CAST('{{authorizationGroup}}' AS VARCHAR(10)) AS "AUTH_OWNER",
        CAST(BUCHUNGSDATUM AS DATE) AS "BUSINESS_VALID_FROM",
        CAST('9999-12-31' AS DATE) AS "BUSINESS_VALID_TO",
        CAST(CASE WHEN MANIPULATIONSGEBUEHR_ABGEGRENZT > 0 THEN 'C_HandlingFeeRequested'
                  WHEN MANIPULATIONSGEBUEHR_ABGEGRENZT < 0 THEN 'C_HandlingFeeToBePaid'
                  ELSE ' '
             END AS VARCHAR(100)) AS "CONDITION_SUBTYPE",
        CAST('C_Fee' AS VARCHAR(40)) AS "CONDITION_TYPE",
        CAST(' ' AS VARCHAR(128)) AS "LOT_ID",
        CAST(' ' AS VARCHAR(3)) AS "MULTICURRENCY_CONTRACT_POSITION_CURRENCY",
        CAST(' ' AS VARCHAR(3)) AS "NOMINAL_AMOUNT_CURRENCY",
        CAST(OriginatingSourceSystem AS VARCHAR(10)) AS "ORIGINATING_SOURCE_SYSTEM",
        CAST(WCD AS VARCHAR(3)) AS "PAYMENT_CURRENCY",
        CAST(WCD AS VARCHAR(3)) AS "POSITION_CURRENCY",
        CAST(RecordSourceID AS VARCHAR(128)) AS "RECORD_SOURCE_ID",
        CAST(' ' AS VARCHAR(50)) AS "ROLE_OF_PAYER",
        CAST(' ' AS VARCHAR(256)) AS "RESULT_GROUP_RESULT_DATA_PROVIDER",
        CAST(' ' AS VARCHAR(128)) AS "RESULT_GROUP_RESULT_GROUP_ID"
        FROM cte_Result
        WHERE MANIPULATIONSGEBUEHR_ABGEGRENZT NOT IN (NULL, 0)
        UNION ALL
        -- For UMSATZPROVISION_ABGEGRENZT
        SELECT
        '{{fsdm_filling.sid}}' AS "FSDM_FILLING_SID",
        CAST(AccSystemID AS VARCHAR(128)) AS "ACCOUNTING_SYSTEM_ACCOUNTING_SYSTEM_ID",
        CAST(GESCHAEFT_SID AS VARCHAR(40)) AS "FINANCIAL_CONTRACT_FINANCIAL_CONTRACT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_FINANCIAL_INSTRUMENT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_ACCOUNT_FINANCIAL_CONTRACT_ID",
        CAST('TimeEffect' AS VARCHAR(100)) AS "ACCOUNTING_CHANGE_REASON",
        CAST(1 AS DECIMAL(10,0)) AS "ACCOUNTING_CHANGE_SEQUENCE_NUMBER",
        CAST(' ' AS VARCHAR(100)) AS "ACCRUAL_CATEGORY",
        CAST(CASE WHEN UMSATZPROVISION_ABGEGRENZT > 0 THEN 'FeeIncome'
                  WHEN UMSATZPROVISION_ABGEGRENZT < 0 THEN 'FeeExpense'
                  ELSE ' '
             END AS VARCHAR(128)) AS "ACCRUAL_TYPE",
        CAST(UMSATZPROVISION_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_PAYMENT_CURRENCY",
        CAST(UMSATZPROVISION_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_POSITION_CURRENCY",
        CAST('{{bank}}' AS DECIMAL(3,0)) AS "BANK",
        CAST('{{client}}' AS VARCHAR(3)) AS "CLIENT",
        CAST('{{authorizationGroup}}' AS VARCHAR(10)) AS "AUTH_OWNER",
        CAST(BUCHUNGSDATUM AS DATE) AS "BUSINESS_VALID_FROM",
        CAST('9999-12-31' AS DATE) AS "BUSINESS_VALID_TO",
        CAST(CASE WHEN UMSATZPROVISION_ABGEGRENZT > 0 THEN 'C_SalesCommissionRequested'
                  WHEN UMSATZPROVISION_ABGEGRENZT < 0 THEN 'C_SalesCommissionToBePaid'
                  ELSE ' '
             END AS VARCHAR(100)) AS "CONDITION_SUBTYPE",
        CAST('C_Fee' AS VARCHAR(40)) AS "CONDITION_TYPE",
        CAST(' ' AS VARCHAR(128)) AS "LOT_ID",
        CAST(' ' AS VARCHAR(3)) AS "MULTICURRENCY_CONTRACT_POSITION_CURRENCY",
        CAST(' ' AS VARCHAR(3)) AS "NOMINAL_AMOUNT_CURRENCY",
        CAST(OriginatingSourceSystem AS VARCHAR(10)) AS "ORIGINATING_SOURCE_SYSTEM",
        CAST(WCD AS VARCHAR(3)) AS "PAYMENT_CURRENCY",
        CAST(WCD AS VARCHAR(3)) AS "POSITION_CURRENCY",
        CAST(RecordSourceID AS VARCHAR(128)) AS "RECORD_SOURCE_ID",
        CAST(' ' AS VARCHAR(50)) AS "ROLE_OF_PAYER",
        CAST(' ' AS VARCHAR(256)) AS "RESULT_GROUP_RESULT_DATA_PROVIDER",
        CAST(' ' AS VARCHAR(128)) AS "RESULT_GROUP_RESULT_GROUP_ID"
        FROM cte_Result
        WHERE UMSATZPROVISION_ABGEGRENZT NOT IN (NULL, 0)
        UNION ALL
        -- For HAFTUNGSPROVISION_ABGEGRENZT
        SELECT
        '{{fsdm_filling.sid}}' AS "FSDM_FILLING_SID",
        CAST(AccSystemID AS VARCHAR(128)) AS "ACCOUNTING_SYSTEM_ACCOUNTING_SYSTEM_ID",
        CAST(GESCHAEFT_SID AS VARCHAR(40)) AS "FINANCIAL_CONTRACT_FINANCIAL_CONTRACT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_FINANCIAL_INSTRUMENT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_ACCOUNT_FINANCIAL_CONTRACT_ID",
        CAST('TimeEffect' AS VARCHAR(100)) AS "ACCOUNTING_CHANGE_REASON",
        CAST(1 AS DECIMAL(10,0)) AS "ACCOUNTING_CHANGE_SEQUENCE_NUMBER",
        CAST(' ' AS VARCHAR(100)) AS "ACCRUAL_CATEGORY",
        CAST(CASE WHEN HAFTUNGSPROVISION_ABGEGRENZT > 0 THEN 'FeeIncome'
                  WHEN HAFTUNGSPROVISION_ABGEGRENZT < 0 THEN 'FeeExpense'
                  ELSE ' '
             END AS VARCHAR(128)) AS "ACCRUAL_TYPE",
        CAST(HAFTUNGSPROVISION_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_PAYMENT_CURRENCY",
        CAST(HAFTUNGSPROVISION_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_POSITION_CURRENCY",
        CAST('{{bank}}' AS DECIMAL(3,0)) AS "BANK",
        CAST('{{client}}' AS VARCHAR(3)) AS "CLIENT",
        CAST('{{authorizationGroup}}' AS VARCHAR(10)) AS "AUTH_OWNER",
        CAST(BUCHUNGSDATUM AS DATE) AS "BUSINESS_VALID_FROM",
        CAST('9999-12-31' AS DATE) AS "BUSINESS_VALID_TO",
        CAST(CASE WHEN HAFTUNGSPROVISION_ABGEGRENZT > 0 THEN 'C_LiabilityCommissionRequested'
                  WHEN HAFTUNGSPROVISION_ABGEGRENZT < 0 THEN 'C_LiabilityCommissionToBePaid'
                  ELSE ' '
             END AS VARCHAR(100)) AS "CONDITION_SUBTYPE",
        CAST('C_Fee' AS VARCHAR(40)) AS "CONDITION_TYPE",
        CAST(' ' AS VARCHAR(128)) AS "LOT_ID",
        CAST(' ' AS VARCHAR(3)) AS "MULTICURRENCY_CONTRACT_POSITION_CURRENCY",
        CAST(' ' AS VARCHAR(3)) AS "NOMINAL_AMOUNT_CURRENCY",
        CAST(OriginatingSourceSystem AS VARCHAR(10)) AS "ORIGINATING_SOURCE_SYSTEM",
        CAST(WCD AS VARCHAR(3)) AS "PAYMENT_CURRENCY",
        CAST(WCD AS VARCHAR(3)) AS "POSITION_CURRENCY",
        CAST(RecordSourceID AS VARCHAR(128)) AS "RECORD_SOURCE_ID",
        CAST(' ' AS VARCHAR(50)) AS "ROLE_OF_PAYER",
        CAST(' ' AS VARCHAR(256)) AS "RESULT_GROUP_RESULT_DATA_PROVIDER",
        CAST(' ' AS VARCHAR(128)) AS "RESULT_GROUP_RESULT_GROUP_ID"
        FROM cte_Result
        WHERE HAFTUNGSPROVISION_ABGEGRENZT NOT IN (NULL, 0)
        UNION ALL
        -- For SPESEN_ABGEGRENZT
        SELECT
        '{{fsdm_filling.sid}}' AS "FSDM_FILLING_SID",
        CAST(AccSystemID AS VARCHAR(128)) AS "ACCOUNTING_SYSTEM_ACCOUNTING_SYSTEM_ID",
        CAST(GESCHAEFT_SID AS VARCHAR(40)) AS "FINANCIAL_CONTRACT_FINANCIAL_CONTRACT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_FINANCIAL_INSTRUMENT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_ACCOUNT_FINANCIAL_CONTRACT_ID",
        CAST('TimeEffect' AS VARCHAR(100)) AS "ACCOUNTING_CHANGE_REASON",
        CAST(1 AS DECIMAL(10,0)) AS "ACCOUNTING_CHANGE_SEQUENCE_NUMBER",
        CAST(' ' AS VARCHAR(100)) AS "ACCRUAL_CATEGORY",
        CAST(CASE WHEN SPESEN_ABGEGRENZT > 0 THEN 'FeeIncome'
                  WHEN SPESEN_ABGEGRENZT < 0 THEN 'FeeExpense'
                  ELSE ' '
             END AS VARCHAR(128)) AS "ACCRUAL_TYPE",
        CAST(SPESEN_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_PAYMENT_CURRENCY",
        CAST(SPESEN_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_POSITION_CURRENCY",
        CAST('{{bank}}' AS DECIMAL(3,0)) AS "BANK",
        CAST('{{client}}' AS VARCHAR(3)) AS "CLIENT",
        CAST('{{authorizationGroup}}' AS VARCHAR(10)) AS "AUTH_OWNER",
        CAST(BUCHUNGSDATUM AS DATE) AS "BUSINESS_VALID_FROM",
        CAST('9999-12-31' AS DATE) AS "BUSINESS_VALID_TO",
        CAST(CASE WHEN SPESEN_ABGEGRENZT > 0 THEN 'C_ExpensesRequested'
                  WHEN SPESEN_ABGEGRENZT < 0 THEN 'C_ExpensesToBePaid'
                  ELSE ' '
             END AS VARCHAR(100)) AS "CONDITION_SUBTYPE",
        CAST('C_Fee' AS VARCHAR(40)) AS "CONDITION_TYPE",
        CAST(' ' AS VARCHAR(128)) AS "LOT_ID",
        CAST(' ' AS VARCHAR(3)) AS "MULTICURRENCY_CONTRACT_POSITION_CURRENCY",
        CAST(' ' AS VARCHAR(3)) AS "NOMINAL_AMOUNT_CURRENCY",
        CAST(OriginatingSourceSystem AS VARCHAR(10)) AS "ORIGINATING_SOURCE_SYSTEM",
        CAST(WCD AS VARCHAR(3)) AS "PAYMENT_CURRENCY",
        CAST(WCD AS VARCHAR(3)) AS "POSITION_CURRENCY",
        CAST(RecordSourceID AS VARCHAR(128)) AS "RECORD_SOURCE_ID",
        CAST(' ' AS VARCHAR(50)) AS "ROLE_OF_PAYER",
        CAST(' ' AS VARCHAR(256)) AS "RESULT_GROUP_RESULT_DATA_PROVIDER",
        CAST(' ' AS VARCHAR(128)) AS "RESULT_GROUP_RESULT_GROUP_ID"
        FROM cte_Result
        WHERE SPESEN_ABGEGRENZT NOT IN (NULL, 0)
        UNION ALL
        -- For UEBERZIEHUNGSZINSEN_VERBRIEFTER_RAHMEN_ABGEGRENZT
        SELECT
        '{{fsdm_filling.sid}}' AS "FSDM_FILLING_SID",
        CAST(AccSystemID AS VARCHAR(128)) AS "ACCOUNTING_SYSTEM_ACCOUNTING_SYSTEM_ID",
        CAST(GESCHAEFT_SID AS VARCHAR(40)) AS "FINANCIAL_CONTRACT_FINANCIAL_CONTRACT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_FINANCIAL_INSTRUMENT_ID",
        CAST(' ' AS VARCHAR(40)) AS "FINANCIAL_INSTRUMENT_ACCOUNT_FINANCIAL_CONTRACT_ID",
        CAST('TimeEffect' AS VARCHAR(100)) AS "ACCOUNTING_CHANGE_REASON",
        CAST(1 AS DECIMAL(10,0)) AS "ACCOUNTING_CHANGE_SEQUENCE_NUMBER",
        CAST(' ' AS VARCHAR(100)) AS "ACCRUAL_CATEGORY",
        CAST(CASE WHEN UEBERZIEHUNGSZINSEN_VERBRIEFTER_RAHMEN_ABGEGRENZT > 0 THEN 'InterestIncome'
                  WHEN UEBERZIEHUNGSZINSEN_VERBRIEFTER_RAHMEN_ABGEGRENZT < 0 THEN 'InterestExpense'
                  ELSE ' '
             END AS VARCHAR(128)) AS "ACCRUAL_TYPE",
        CAST(UEBERZIEHUNGSZINSEN_VERBRIEFTER_RAHMEN_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_PAYMENT_CURRENCY",
        CAST(UEBERZIEHUNGSZINSEN_VERBRIEFTER_RAHMEN_ABGEGRENZT * (-1) AS DECIMAL(31,6)) AS "AMOUNT_IN_POSITION_CURRENCY",
        CAST('{{bank}}' AS DECIMAL(3,0)) AS "BANK",
        CAST('{{client}}' AS VARCHAR(3)) AS "CLIENT",
        CAST('{{authorizationGroup}}' AS VARCHAR(10)) AS "AUTH_OWNER",
        CAST(BUCHUNGSDATUM AS DATE) AS "BUSINESS_VALID_FROM",
        CAST('9999-12-31' AS DATE) AS "BUSINESS_VALID_TO",
        CAST(CASE WHEN UEBERZIEHUNGSZINSEN_VERBRIEFTER_RAHMEN_ABGEGRENZT < 0 THEN 'C_OverdraftInterestSecuredLimitToBePaid'
                  WHEN UEBERZIEHUNGSZINSEN_VERBRIEFTER_RAHMEN_ABGEGRENZT > 0 THEN 'C_OverdraftInterestSecuredLimitRequested'
                  ELSE ' '
             END AS VARCHAR(100)) AS "CONDITION_SUBTYPE",
        CAST('C_Interest' AS VARCHAR(40)) AS "CONDITION_TYPE",
        CAST(' ' AS VARCHAR(128)) AS "LOT_ID",
        CAST(' ' AS VARCHAR(3)) AS "MULTICURRENCY_CONTRACT_POSITION_CURRENCY",
        CAST(' ' AS VARCHAR(3)) AS "NOMINAL_AMOUNT_CURRENCY",
        CAST(OriginatingSourceSystem AS VARCHAR(10)) AS "ORIGINATING_SOURCE_SYSTEM",
        CAST(WCD AS VARCHAR(3)) AS "PAYMENT_CURRENCY",
        CAST(WCD AS VARCHAR(3)) AS "POSITION_CURRENCY",
        CAST(RecordSourceID AS VARCHAR(128)) AS "RECORD_SOURCE_ID",
        CAST(' ' AS VARCHAR(50)) AS "ROLE_OF_PAYER",
        CAST(' ' AS VARCHAR(256)) AS "RESULT_GROUP_RESULT_DATA_PROVIDER",
        CAST(' ' AS VARCHAR(128)) AS "RESULT_GROUP_RESULT_GROUP_ID"
        FROM cte_Result
        WHERE UEBERZIEHUNGSZINSEN_VERBRIEFTER_RAHMEN_ABGEGRENZT NOT IN (NULL, 0)
        )
        {% endblock %} 
        {% endembed %}


## 📫 Contact
- Email: nomerzonalecha1@gmail.com
- Phone: 09760180051

*Note: Actual table names, proprietary business rules, and client-specific logic have been replaced with generic examples to protect confidentiality.*
