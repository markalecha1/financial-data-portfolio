# Financial Accruals Reconciliation Engine

## 🎯 Project Overview
Advanced SQL-based validation framework developed during SAP S/4HANA migration to ensure accurate financial accruals accounting across multiple banking systems.

## 📊 Business Challenge
- Multiple legacy systems with different accounting treatments
- Manual reconciliation processes causing regulatory risks
- Inconsistent accrual categorization across 100+ banking clients

## 🛠 Solution Architecture
```mermaid
graph TB
    A[Arctis System] --> C[Accruals Engine]
    B[BS System] --> C
    D[Murex System] --> C
    E[SAP FSDM] --> F[Validation Layer]
    C --> F
    F --> G[Reconciliation Reports]
    F --> H[Data Quality Metrics]

-- Example of accrual categorization logic
WITH financial_accruals AS (
  SELECT 
    contract_id,
    accrual_type,
    amount,
    CASE 
      WHEN accrual_type = 'Interest' AND amount < 0 THEN 'InterestIncome'
      WHEN accrual_type = 'Interest' AND amount > 0 THEN 'InterestExpense'
      WHEN accrual_type = 'Fee' AND amount > 0 THEN 'FeeIncome'
      -- 20+ additional business rules implemented...
    END AS accounting_category
  FROM source_accruals
)
SELECT * FROM financial_accruals;
