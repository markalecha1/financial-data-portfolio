# Financial Accruals & Accounting Validation Framework

**Advanced SQL-based reconciliation system for financial accruals accounting across multiple banking systems**

## Business Context

Developed to validate and reconcile accrual accounting entries during SAP S/4HANA migration, ensuring accurate financial reporting for Interest Income, Interest Expense, Fee Income, and Fee Expense across 100+ banking clients.

## Validation Scope

### Financial Categories Validated:
- **Interest Accruals**: Debit/Credit Interest, Overdraft Interest
- **Fee Accruals**: Loan Commissions, Management Fees, Commitment Fees
- **Provision Accruals**: Liability Provisions, Sales Commissions
- **Expense Accruals**: Handling Fees, Operational Expenses

### Multi-System Integration:
- **Arctis**: Core banking transactions
- **BS (Bond/Securities)**: Derivatives and securities accounting
- **Murex**: Trading system integration
- **SAP FSDM**: Target system validation

  -- Simplified example of accrual categorization logic
WITH financial_accruals AS (
  SELECT 
    contract_id,
    accrual_type,
    amount,
    CASE 
      WHEN accrual_type = 'SOLLZINSEN' AND amount < 0 THEN 'InterestIncome'
      WHEN accrual_type = 'HABENZINSEN' AND amount > 0 THEN 'InterestExpense'
      WHEN accrual_type = 'KREDITPROVISION' AND amount > 0 THEN 'FeeIncome'
      -**- 20+ additional business rules...**
    END AS accounting_category,
    CASE 
      WHEN accounting_category = 'InterestIncome' THEN 'C_DebitInterest'
      WHEN accounting_category = 'InterestExpense' THEN 'C_CreditInterest'
      **-- Corresponding condition subtypes...**
    END AS condition_subtype
  FROM source_accruals
  WHERE reporting_date = :CUTOFF_DATE
)
SELECT * FROM financial_accruals;
