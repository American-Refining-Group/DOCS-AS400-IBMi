### List of Use Cases Implemented by the AR930 Program

The `AR930` RPGLE program, invoked via the `AR930.ocl36.txt` OCL script, is designed for Accounts Receivable (A/R) control file maintenance. Based on the provided code, the program implements the following use cases, which focus on managing company records in the A/R control file (`ARCONT`):

1. **Maintain A/R Company Control Records**:
   - **Description**: Allows users to add, update, delete, or reactivate A/R company records, including configuration details such as company name, GL accounts, aging limits, and journal numbers. The program validates inputs against the customer file (`ARCUST`) and general ledger master file (`GLMAST`) to ensure data integrity.
   - **Operations**:
     - **Add**: Create a new company record in `ARCONT`.
     - **Update**: Modify existing company record details.
     - **Delete**: Mark a company record as deleted (`acdel = 'D'`) if no active customer records exist in `ARCUST`.
     - **Reactivate**: Restore a deleted company record (`acdel = 'A'`).
     - **Inquiry**: View existing company records.
   - **Context**: Interactive workstation interface using screens (`AR930S1` for inquiry/update, `AR930S2` for data entry), with navigation via function keys (F1, F4, F10, F11).

Given that the program is centered around a single, cohesive purpose (managing A/R company records), this is the primary use case. All operations (add, update, delete, reactivate, inquiry) are part of this use case, as they collectively address the maintenance of A/R control data.

---

### Function Requirement Document for A/R Company Control Maintenance

Below is a function requirement document for a non-interactive version of the A/R company control maintenance use case. This assumes the function accepts all necessary inputs programmatically (rather than via a screen) to perform add, update, delete, or reactivate operations on the `ARCONT` file. The document focuses on business requirements, process steps, and calculations, presented concisely.



# A/R Company Control Maintenance Function Requirements

## Overview
The `MaintainARCompanyControl` function manages Accounts Receivable (A/R) company records in the `ARCONT` file, supporting add, update, delete, and reactivate operations. It validates inputs against the `ARCUST` (A/R customer) and `GLMAST` (General Ledger master) files, ensuring data integrity for A/R configurations.

## Inputs
- **Operation Type**: String (`ADD`, `UPDATE`, `DELETE`, `REACTIVATE`).
- **Company Number** (`acco`): 2-digit numeric (non-zero for `ADD`, `UPDATE`).
- **Company Name** (`acname`): 30-character string.
- **A/R GL Number** (`acargl`): 8-digit numeric (optional).
- **Sales GL Number** (`acslgl`): 8-digit numeric (optional).
- **Discount GL Number** (`acdsgl`): 8-digit numeric (optional).
- **Cash GL Number** (`accsgl`): 8-digit numeric (optional).
- **Intercompany GL Number** (`actrgl`): 8-digit numeric (optional).
- **Finance Charge GL Number** (`acfngl`): 8-digit numeric (optional).
- **EFT Cash GL Number** (`acefcg`): 8-digit numeric (optional).
- **Next A/R Journal Number** (`acarj#`): 2-digit numeric.
- **Next Sales Journal Number** (`acslj#`): 2-digit numeric.
- **Finance Charge Percentage** (`acfinc`): 5-digit numeric (4.1 format, e.g., 12.345).
- **Security Code** (`acsecr`): 8-character string.
- **First Fiscal Month** (`acffmo`): 2-digit numeric (1-12).
- **Aging Limits**:
  - `aclmt1`: 2-digit numeric (0-30 days).
  - `aclmt2`: 2-digit numeric (31-60 days).
  - `aclmt3`: 2-digit numeric (61-90 days).
  - `aclmt4`: 2-digit numeric (91-120 days or over 120).
- **Last Close Date** (`acdtcl`): 6-digit numeric (YYMMDD).
- **Cash Receipts Date** (`acdtcr`): 6-digit numeric (YYMMDD).

## Outputs
- **Status**: Success or error code.
- **Error Message**: Description of validation or processing errors (if any).

## Process Steps
1. **Validate Operation and Company Number**:
   - Ensure `Operation Type` is one of `ADD`, `UPDATE`, `DELETE`, `REACTIVATE`.
   - For `ADD` or `UPDATE`, verify `acco` is non-zero.
   - For `ADD`, check that `acco` does not exist in `ARCONT`.
   - For `UPDATE`, `DELETE`, or `REACTIVATE`, check that `acco` exists in `ARCONT`.

2. **Validate Input Data**:
   - **Company Name**: Must not be blank.
   - **Security Code**: Must not be blank.
   - **GL Accounts**: For `acargl`, `acslgl`, `acdsgl`, `accsgl`, `actrgl`, `acfngl`, `acefcg` (if non-zero):
     - Must exist in `GLMAST` and not be marked as deleted (`gldel = 'D'`) or inactive (`gldel = 'I'`).
   - **Aging Limits**:
     - Must be non-zero.
     - Must follow ascending order: `aclmt1 < aclmt2 < aclmt3 < aclmt4`.
   - **Finance Charge %**: No specific validation beyond format (4.1).

3. **Perform Operation**:
   - **ADD**:
     - Create new `ARCONT` record with `acdel = 'A'` (active) and provided inputs.
   - **UPDATE**:
     - Update existing `ARCONT` record with provided inputs, retaining `acdel` unless changed.
   - **DELETE**:
     - Check `ARCUST` for active records (non-deleted, matching `acco`).
     - If none exist, set `acdel = 'D'` in `ARCONT`.
   - **REACTIVATE**:
     - If `acdel = 'D'`, set `acdel = 'A'` in `ARCONT`.

4. **Write to File**:
   - Commit changes to `ARCONT` (add or update record).
   - Return success status or error message if validation fails.

## Business Rules
- **Company Number**: Must be unique for `ADD`; must exist for `UPDATE`, `DELETE`, `REACTIVATE`.
- **Company Name and Security Code**: Cannot be blank.
- **GL Accounts**: Non-zero accounts must be valid (exist in `GLMAST`, not deleted or inactive).
- **Aging Limits**: Non-zero, ascending order (0-30, 31-60, 61-90, 91-120/over 120 days from invoice date, per 04/13/05 revision).
- **Deletion**: Prohibited if active customer records exist in `ARCUST`.
- **Reactivation**: Only allowed if record is marked deleted (`acdel = 'D'`).
- **Inactive GL Accounts**: Treated as deleted (per 08/24/16 revision).
- **Numeric Fields**: Filler at positions 113-114 in `ARCONT` is numeric (2.0 format, per 09/27/22 revision).

## Calculations
- **Aging Limits**: No calculations; validated for non-zero and ascending order.
- **Finance Charge %**: Stored as provided (4.1 format), no calculations.
- **Journal Numbers**: Stored as provided, no calculations.
- **Dates**: `acdtcl` and `acdtcr` stored as provided (YYMMDD), no calculations.

## Error Messages
- "COMPANY CANNOT BE ZERO - ENTER OR" (zero `acco` in `ADD`/`UPDATE`).
- "CANNOT ADD - THIS RECORD EXISTS" (`acco` exists in `ADD`).
- "COMPANY NAME CANNOT BE BLANK".
- "SECURITY CODE CAN NOT BE BLANK".
- "INVALID ACCOUNTS RECEIVABLE GL NO" (invalid `acargl`).
- "INVALID SALES GL NUMBER" (invalid `acslgl`).
- "INVALID DISCOUNT GL NUMBER" (invalid `acdsgl`).
- "INVALID CASH GL NUMBER" (invalid `accsgl`).
- "INVALID INTER-CO G/L NUMBER" (invalid `actrgl`).
- "INVALID FINANCE CHARGE GL NUMBER" (invalid `acfngl`).
- "INVALID EFT CASH GL NUMBER" (invalid `acefcg`).
- "INVALID AGING LIMITS" (zero or non-ascending `aclmt1`–`aclmt4`).
- "COMPANY MASTER RECORDS EXIST" (active `ARCUST` records for `DELETE`).
- "DELETION OF COMPANY NOT ALLOWED" (active `ARCUST` records).
- "RECORD NOT FOUND" (invalid `acco` for `UPDATE`, `DELETE`, `REACTIVATE`).
- "PREVIOUS RECORD WAS REACTIVATED" (successful reactivation).

## Dependencies
- **Files**:
  - `ARCONT`: A/R control file (read/write).
  - `ARCUST`: A/R customer file (read-only, for deletion check).
  - `GLMAST`: General Ledger master file (read-only, for GL validation).
- **External Programs**: None (assumes inputs provided programmatically).

## Notes
- The function replaces interactive screen handling with direct input processing.
- Aging is based on invoice date (0-30, 31-60, 61-90, 91-120, over 120 days), per 04/13/05 revision.
- No external program calls are required, unlike the OCL setup (`GSGENIEC`, `GSY2K`).

