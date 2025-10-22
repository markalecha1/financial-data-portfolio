# Financial Accruals Reconciliation Engine

##  Project Overview
Advanced SQL-based validation framework developed during legacy to SAP S/4HANA migration to ensure accurate financial accruals accounting across multiple banking systems.


## Solution Architecture
```mermaid
graph TB
    A[Arctis System] --> C[SPOT Layer]
    B[BS System] --> C
    D[Murex System] --> C
    C --> F[SAP FSDM Layer]
    F --> G[SAP FPSL Layer]
    G --> H[Dashboard Validation Layer]

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
