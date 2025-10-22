# Mark Alecha - Data Engineering Projects

## Project 1: Financial Accruals Validation 
**Advanced SQL framework for financial accounting reconciliation**

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


### Technical Highlights:
- **Complex CTEs**: Multi-layer data transformation
- **Business Logic**: 50+ accounting rules implemented in SQL
- **Data Reconciliation**: Cross-system validation with gap analysis
- **Performance**: Handled 50,000+ monthly accounting entries

### Business Impact:
- Validated $XX billion in financial accruals
- 100% accurate regulatory reporting post-migration

## Project 2: Financial Product Master Data Engine
**Automated classification system for banking products**

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

text

### Technical Highlights:
- **Hierarchical Logic**: Multi-level product classification
- **Risk Categorization**: Automated regulatory treatment assignment
- **Cross-System Mapping**: Validation against target taxonomies
- **Data Quality**: Comprehensive validation rules

### Business Impact:
- Standardized 20+ product categories bank-wide
- Enabled accurate Basel III capital calculation
- 75% reduction in manual categorization effort

## 🔧 Technical Skills Demonstrated
- **Advanced SQL**: CTEs, Complex Joins, Window Functions, Business Logic
- **Data Architecture**: Multi-system integration, Data modeling
- **Financial Domain**: Accounting rules, Regulatory requirements
- **Tools**: EXASOL, SAP HANA, DBeaver, JIRA

## 📫 Contact
- Email: nomerzonalecha1@gmail.com
- Phone: 09760180051

*Note: Actual table names, proprietary business rules, and client-specific logic have been replaced with generic examples to protect confidentiality.*
