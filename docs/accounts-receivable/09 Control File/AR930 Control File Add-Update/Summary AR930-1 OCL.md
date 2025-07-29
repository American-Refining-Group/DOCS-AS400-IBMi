### List of Use Cases Implemented by the AR930 Program

The `AR930` RPGLE program, called from the `AR930.ocl36.txt` OCL program, implements the following use cases for Accounts Receivable (A/R) control file maintenance:

1. **Maintain A/R Control Records**:
   - **Description**: Allows users to add, update, delete, or reactivate company records in the A/R control file (`ARCONT`), with validations against the customer file (`ARCUST`) and general ledger master file (`GLMAST`). The program supports interactive entry and update modes, navigation through records, and error handling for invalid inputs.
   - **Sub-Use Cases**:
     - **Add New Company**: Create a new A/R control record with company details, G/L accounts, and aging limits.
     - **Update Existing Company**: Modify an existing A/R control record’s details, such as G/L accounts or aging limits.
     - **Delete Company**: Mark a company record as deleted, provided no active customer records exist.
     - **Reactivate Company**: Restore a previously deleted company record.
     - **Navigate Records**: Scroll forward or backward through company records for review or selection.

Since the program is designed for interactive screen-based maintenance, these sub-use cases are part of the overarching use case of maintaining A/R control records. For the purpose of the function requirement document, I will treat the primary use case as a single large function that processes inputs programmatically (without screen interaction) to manage A/R control records.

---



# Function Requirements Document: A/R Control Record Maintenance

## Overview
The `MaintainARControl` function manages Accounts Receivable (A/R) control records for a company, supporting add, update, delete, and reactivate operations. It processes input data, validates it against business rules, and updates the A/R control file (`ARCONT`), referencing the customer file (`ARCUST`) and general ledger master file (`GLMAST`).

## Function Signature
```plaintext
MaintainARControl(
  action: String,           // 'ADD', 'UPDATE', 'DELETE', 'REACTIVATE'
  company_number: Integer,  // 2-digit company number
  company_name: String,     // 30-character company name
  ar_gl: Integer,           // 8-digit A/R G/L account number
  sales_gl: Integer,        // 8-digit Sales G/L account number
  discount_gl: Integer,     // 8-digit Discount G/L account number
  cash_gl: Integer,         // 8-digit Cash G/L account number
  interco_gl: Integer,      // 8-digit Intercompany G/L account number
  finance_gl: Integer,      // 8-digit Finance Charge G/L account number
  eft_cash_gl: Integer,     // 8-digit EFT Cash G/L account number
  next_ar_journal: Integer, // 2-digit next A/R journal number
  next_sales_journal: Integer, // 2-digit next Sales journal number
  finance_charge_pct: Decimal(5,4), // Finance charge percentage
  security_code: String,    // 8-character security code
  first_fiscal_month: Integer, // 2-digit fiscal month (1-12)
  aging_limit_1: Integer,   // 2-digit aging limit (0-30 days)
  aging_limit_2: Integer,   // 2-digit aging limit (31-60 days)
  aging_limit_3: Integer,   // 2-digit aging limit (61-90 days)
  aging_limit_4: Integer,   // 3-digit aging limit (91-120 days)
  last_close_date: Integer, // 6-digit last year/month closed (YYMMDD)
  cash_receipts_date: Integer // 6-digit cash receipts date (YYMMDD)
) -> Result (status: String, message: String)
```

## Process Steps
1. **Input Validation**:
   - Validate the `action` is one of: 'ADD', 'UPDATE', 'DELETE', 'REACTIVATE'.
   - Check if the `company_number` exists in `ARCONT` (for UPDATE, DELETE, REACTIVATE) or does not exist (for ADD).
   - Validate all input fields against business rules (see below).

2. **Business Rule Enforcement**:
   - Apply business rules to ensure data integrity and compliance with A/R policies.
   - Return error messages if any rule is violated.

3. **Data Processing**:
   - **ADD**: Create a new `ARCONT` record with provided inputs and set `acdel = 'A'` (active).
   - **UPDATE**: Update an existing `ARCONT` record with provided inputs, retaining `acdel` status.
   - **DELETE**: Set `acdel = 'D'` in the `ARCONT` record if no active customer records exist in `ARCUST`.
   - **REACTIVATE**: Set `acdel = 'A'` in the `ARCONT` record if currently deleted.

4. **Output**:
   - Return a result with `status` ('SUCCESS' or 'ERROR') and a `message` (success confirmation or error description).

## Business Rules
1. **Company Number**:
   - Must be non-zero for ADD.
   - Must exist in `ARCONT` for UPDATE, DELETE, REACTIVATE.
   - Must not exist in `ARCONT` for ADD.

2. **Company Name**:
   - Must not be blank.

3. **General Ledger Accounts**:
   - `ar_gl`, `sales_gl`, `discount_gl`, `cash_gl`, `interco_gl`, `finance_gl`, `eft_cash_gl`:
     - If non-zero, must exist in `GLMAST` and not be deleted (`gldel ≠ 'D'`) or inactive (`gldel ≠ 'I'`).
     - Zero is allowed (bypasses validation).

4. **Security Code**:
   - Must not be blank.

5. **Aging Limits**:
   - `aging_limit_1`, `aging_limit_2`, `aging_limit_3`, `aging_limit_4`:
     - Must be non-zero.
     - Must be in ascending order: `aging_limit_1 < aging_limit_2 < aging_limit_3 < aging_limit_4`.
     - Represent aging buckets (0-30, 31-60, 61-90, 91-120 days from invoice date).

6. **Deletion**:
   - Cannot delete a company if active customer records (`ardel ≠ 'D'`) exist in `ARCUST`.

7. **Reactivation**:
   - Only allowed if the company record is marked deleted (`acdel = 'D'`).

## Calculations
- **Aging Limits**: No explicit calculations are performed; the function validates that `aging_limit_1` to `aging_limit_4` are in ascending order to define aging buckets for A/R reporting (e.g., 0-30, 31-60, 61-90, 91-120 days from invoice date).
- **Finance Charge Percentage**: Stored as a decimal (5,4) but no calculations are performed in this function; it is validated for presence and stored for use in other A/R processes.

## Inputs and Outputs
- **Inputs**: As defined in the function signature (company details, G/L accounts, aging limits, etc.).
- **Outputs**:
  - `status`: 'SUCCESS' if the operation completes successfully; 'ERROR' otherwise.
  - `message`: Descriptive message, e.g., "Record added successfully", "Invalid A/R G/L number", or "Company master records exist, deletion not allowed".

## Dependencies
- **Files**:
  - `ARCONT`: A/R control file (update, keyed on company number).
  - `ARCUST`: A/R customer file (input, for deletion validation).
  - `GLMAST`: General Ledger master file (input, for G/L account validation).
- **External Programs**: None.

## Error Handling
- Returns specific error messages for violations of business rules, such as:
  - "COMPANY CANNOT BE ZERO" (ADD with zero company number).
  - "RECORD NOT FOUND" (UPDATE/DELETE/REACTIVATE with non-existent company).
  - "CANNOT ADD - THIS RECORD EXISTS" (ADD with existing company).
  - "COMPANY NAME CANNOT BE BLANK".
  - "INVALID [A/R|SALES|DISCOUNT|CASH|INTER-CO|FINANCE|EFT CASH] GL NUMBER".
  - "SECURITY CODE CAN NOT BE BLANK".
  - "INVALID AGING LIMITS" (non-zero or non-ascending limits).
  - "COMPANY MASTER RECORDS EXIST" (DELETE with active customers).

