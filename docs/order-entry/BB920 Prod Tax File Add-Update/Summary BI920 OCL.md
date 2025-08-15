### Use Cases Implemented
Based on the OCL script and RPG program BI920, which together form a call stack for maintaining the Product Tax Master file, the following use cases are implemented. These are derived from the program's interactive maintenance functionality, validations, and auxiliary processes:

1. **Inquiry and Selection of Product Tax Records**: Allows users to specify criteria (company, state, starting product code, delivered status) to retrieve and display batches of product tax records for viewing and editing, including paging through results and handling the special 'MISC' product.

2. **Addition of New Product Tax Records**: Enables adding new entries to the Product Tax Master file for valid products, with up to 10 tax codes per record, ensuring all validations pass before committing.

3. **Update of Existing Product Tax Records**: Permits modification of tax codes in existing Product Tax Master records, with re-validation and commit only if changes are non-blank and valid.

4. **Deletion of Product Tax Records**: Supports marking records for deletion (via 'D' flag) or implicit deletion (by blanking all tax codes), physically removing them from the file.

5. **Validation of Tax Configurations**: Enforces business rules during editing, such as checking tax codes against the Sales Tax Master, restricting sales tax based on company invoice style, and ensuring 'MISC' products use only 'T'-prefixed tax codes.

6. **Cleanup of Orphaned Product Tax Records**: On program exit (via F3 key), automatically scans and deletes Product Tax Master records without matching sellable products in the Product Master file.

These use cases are interconnected within the single interactive program flow but can be considered distinct based on user actions and program subroutines.

### Function Requirement Document: Maintain Product Tax Configurations
This document assumes the primary use case (encompassing inquiry, add, update, delete, and validation) is implemented as a non-interactive function `MaintainProductTax`. The function receives all inputs upfront (e.g., via parameters or a data structure) and processes them in batch, returning success/failure with logs or updated records. It focuses on business requirements for tax maintenance in a billing system, with concise steps and rules. No UI interaction; all operations are file-based.

#### Function Overview
- **Purpose**: Automate maintenance of product tax configurations by company and state, ensuring accurate tax application in billing/invoicing. Handles add/update/delete for tax codes linked to products, with validation against master data.
- **Inputs**:
  - Company Code (CO: 2-char, required; must exist in BICONT).
  - State Code (STAT: 2-char, required).
  - Delivered Flag (DELV: 1-char; 'Y' or blank).
  - Starting Product Code (SPRD: 4-char; optional, defaults to '  01').
  - List of Product Tax Changes: Array of structures, each with:
    - Product Code (PTPROD: 4-char; 'MISC' or from GSPROD).
    - Tax Codes (up to 10: 4-char each; blank if unchanged).
    - Delete Flag ('D' or blank).
- **Outputs**:
  - Success Flag (boolean).
  - Error Log (array of messages with details, e.g., invalid tax codes).
  - Updated Records (optional: list of affected BIPRTX entries).
- **Dependencies**: Read access to GSPROD, BICONT, BISLTX; Update access to BIPRTX.

#### Process Steps
1. **Validate Input Criteria**:
   - Retrieve company details from BICONT using CO as key.
   - If company not found, return error: "INVALID COMPANY NUMBER ENTERED".
   - Validate DELV: Must be 'Y' or blank; else error: "DELIVERED MUST BE 'Y' OR BLANK".
   - Retrieve invoice style (BCINST) from BICONT for later rules.

2. **Retrieve and Prepare Product List**:
   - Position on GSPROD using key (CO + SPRD or '  01').
   - Include 'MISC' as first pseudo-product if starting from beginning.
   - Fetch sellable products (TPSELL = 'Y') sequentially, up to batch size (e.g., all matching inputs).
   - For each product in input changes list, match against fetched products or 'MISC'; ignore non-matches.

3. **Validate Tax Changes**:
   - For each product change:
     - If all tax codes blank and not delete, skip (no action).
     - If BCINST = '2', prohibit any 'T'-prefixed tax codes; error: "SALES TAX NOT ALLOWED IF INVOICE STYLE 2" with offending codes.
     - Validate each tax code exists in BISLTX (key: STTXCD); error: "TAX CODES NOT IN 'SALES TAX MASTER FILE'" with invalid codes.
     - If product = 'MISC', ensure non-blank tax codes start with 'T'; error: "TAX CODE MUST BEGIN WITH 'T'" with offending codes.

4. **Apply Changes to BIPRTX**:
   - Build key (PRTXKY: CO + STAT + PTPROD + DELV).
   - For each valid change:
     - If record exists:
       - If delete flag = 'D' or all tax codes blank: Delete record.
       - Else: Update with new tax codes (PTTXC1 to PTTXC0).
     - If record not exists and tax codes non-blank: Add new record with 'A' audit flag.
   - Commit all changes atomically if no errors.

5. **Cleanup Orphaned Records**:
   - Scan all BIPRTX records for the company/state.
   - For non-'MISC' products, verify existence in GSPROD (sellable).
   - Delete any unmatched records.

6. **Return Results**:
   - If errors, rollback changes and return error log.
   - Else, return success with any updated records.

#### Business Rules
- **Tax Applicability**: Tax codes determine billing taxes per product/state; up to 10 per record for multi-tax scenarios (e.g., federal, state, local).
- **'MISC' Handling**: Special entry for miscellaneous invoice lines; always processed first; restricts to transaction ('T') taxes only.
- **Sellable Products Only**: Products must have TPSELL = 'Y' in GSPROD; non-sellable skipped during retrieval.
- **Blank Tax Codes**: All blanks imply no tax; do not add/update record (effective delete if existing).
- **Invoice Style Restriction**: BCINST = '2' blocks sales taxes to comply with company billing policies.
- **Key Uniqueness**: Records unique by company + state + product + delivered; no duplicates allowed.
- **Audit/Deletion**: Added records flagged 'A'; deleted flagged 'D' (but physically removed).
- **No Calculations**: No arithmetic involved; purely validation and data maintenance.
- **Error Prioritization**: Halt on first validation failure per record; collect all errors for logging.

This function encapsulates the program's logic as a batch process, suitable for API or automated integration.