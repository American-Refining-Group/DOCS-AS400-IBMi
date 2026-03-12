### List of Use Cases Implemented by the Program Stack

The program stack, consisting of `AR300P.ocl36.txt`, `AR300P.rpgle.txt`, and `AR300.rpg36.txt`, implements a single primary use case:

1. **Generate Monthly Accounts Receivable (A/R) Statements**:
   - This use case involves prompting the user for input parameters (via `AR300P.rpgle`), validating those inputs, and generating A/R statements for selected companies or all companies, including transaction details, aging information, and finance charges (via `AR300.rpg36`). It also prepares the `ARDETL` file for purging by marking paid transactions.

### Function Requirement Document



# A/R Statement Generation Function Requirements

## Overview
The `generateARStatements` function generates monthly accounts receivable (A/R) statements for specified companies or all companies, calculates finance charges, and prepares the A/R detail file for purging. It processes input parameters, validates them, and produces formatted statement outputs with transaction details, aging buckets, and auditor instructions.

## Inputs
- **Security Code (`kysec`, 8 chars)**: Validates access to company data.
- **Statement Date (`kydate`, 6 digits, MMDDYY)**: Date for statements.
- **Company Selection (`kyalco`, 3 chars)**: 'ALL' for all companies or 'CO ' for specific companies.
- **Company Numbers (`kyco1`, `kyco2`, `kyco3`, 2 digits each)**: Specific company numbers if `kyalco = 'CO '`.
- **Month-to-Date Flag (`kycmtd`, 1 char)**: 'Y', 'N', or blank for month-to-date reporting.
- **Year-to-Date Flag (`kycytd`, 1 char)**: 'Y', 'N', or blank for year-to-date reporting.
- **Number of Copies (`kycopy`, 2 digits)**: Number of statement copies (defaults to 1 if 0).

## Outputs
- **Statement Reports**:
  - Primary statement (`REPORT`): Customer details, transactions, aging, finance charges.
  - Remittance stub (`REPRT2`): Payment submission details.
  - Saved copy (`REPRT3`): For archival to `PITT-FS/SHARED/AR STATEMENTS`.
  - Alternate format (`REPRT4`): Minimal remittance info.
  - Wire transfer format (`REPRT5`): Includes bank details for wire transfer customers.
- **Updated `ARDETL` File**: Marks transactions as paid ('P') or statement ('S') for purging.
- **Error Messages**: Returned if validation fails (e.g., "INVALID STATEMENT DATE").

## Process Steps
1. **Initialize Environment**:
   - Call `GSGENIEC` to set up the job environment.
   - Check job variable (`L'506,3'`) to determine if processing should proceed; exit if condition met.
   - Execute `SCPROCP` with library parameter (`?9?`) for setup.
   - Run `GSY2K` for date format handling.
   - Reset local data areas and job switches.

2. **Validate Inputs**:
   - Verify security code (`kysec`) against `acsecr` in `ARCONT` for the company.
   - Validate `kydate` (MMDDYY) for correct month (1–12), day (1–31, or 28/29 for February with leap year check), and year (using `y2kcen`).
   - Ensure `kyalco` is 'ALL' or 'CO '.
   - If `kyalco = 'CO '`, validate `kyco1`, `kyco2`, `kyco3` exist in `ARCONT`, are not deleted (`acdel ≠ 'D'`), and match `acco`.
   - If `kyalco = 'ALL'`, ensure `kyco1`, `kyco2`, `kyco3` are 0.
   - Validate `kycmtd` and `kycytd` are 'Y', 'N', or blank.
   - Set `kycopy` to 1 if 0.
   - Return error messages for any validation failure and halt processing.

3. **Load Company Data**:
   - Retrieve up to 10 companies from `ARCONT` (non-deleted records) for display or processing.
   - If `GSCONT` has a valid company number (`gxcono`), set `kyalco = 'CO '` and `kyco1 = gxcono`; otherwise, set `kyalco = 'ALL'`.
   - Set defaults: `kycmtd = 'Y'`, `kycytd = 'N'`, `kycopy = 1`.

4. **Process Companies and Customers**:
   - For each company (`ADCO` from `ARDETL`, filtered by `kyalco` and `kyco1–3`):
     - Validate fiscal month (`ACFFMO` from `ARCONT`) against statement month (`STMO`).
     - Determine invoicing style (`BCINST` from `BICONT`): '1' (style 1), '2' (style 2), '5' (wire transfer), or default.
     - For each customer (`ADCUST` from `ARDETL`):
       - Retrieve customer data from `ARCUST` (`ARNAME`, `ARADR1–4`, `ARTOTD`, `ARCURD`, `AR0110–AROV30`, `ARSTMT`, `ARFINC`, `ARWIRE`).
       - Skip if `ARSTMT = 'N'` or total due is 0.

5. **Process Transactions**:
   - For each transaction in `ARDETL`:
     - Identify type (`ADTYPE`): 'I' (invoice), 'J' (journal), 'P' (payment).
     - Calculate remaining amount (`ADLEFT = ADAMT - ADPART`).
     - Add current month payment (`ADPAY`) to `ADPART`.
     - Flag as paid (`P`) if `ADAMT = ADPART`, else statement (`S`).
     - Mark miscellaneous cash if `PPINV# = 9`.
     - Note past due invoices (`ADAGE = 1`).

6. **Calculate Finance Charges**:
   - If `ARFINC = 'Y'` in `ARCUST`, calculate `FCAMT = (AR0110 + AR1120 + AR2130 + AROV30) * ACFINC` (from `ARCONT`).
   - Set `FCAMT = 0.50` if less than 0.50.
   - Add `FCAMT` to `ARFIN$` and `ARCURD`.

7. **Generate Statements**:
   - Produce statements based on invoicing style:
     - Style 1 (`61`): Standard format (`REPORT`).
     - Style 2 (`62`): Remittance stub (`REPRT2`).
     - Style 5 (`63`): Wire transfer with bank details (`REPRT3`, `REPRT5`).
     - Default (`64`): Minimal format (`REPRT4`).
   - Include customer details, transaction lines (date, invoice, charge, payment, balance), aging totals, finance charges, and auditor’s messages.
   - Save a copy to `PITT-FS/SHARED/AR STATEMENTS` (`REPRT3`).

8. **Prepare for Purge**:
   - Update `ARDETL` with 'P' (paid) or 'S' (statement) to mark records for purging.

## Business Rules
1. **Statement Eligibility**: Print only if `ARSTMT ≠ 'N'` and `ARTOTD ≠ 0`.
2. **Company Selection**: Process all companies (`kyalco = 'ALL'`) or specific valid companies (`kyalco = 'CO '`, `kyco1–3` exist in `ARCONT`).
3. **Date Validation**: `kydate` must be a valid MMDDYY (1–12 months, 1–31 days, 28/29 for February with leap year logic).
4. **Finance Charges**: Apply if `ARFINC = 'Y'`, minimum $0.50, based on overdue amounts (`AR0110 + AR1120 + AR2130 + AROV30`).
5. **Aging Accuracy**: `ARTOTD` must equal sum of aging buckets (`ARCURD + AR0110 + AR1120 + AR2130 + AROV30`).
6. **Transaction Status**: Mark `ARDETL` records as paid (`P`) if fully paid, else statement (`S`).
7. **Output Formats**: Support multiple formats (standard, remittance, wire transfer, archived copy) based on `BCINST`.
8. **Security**: Validate `kysec` against `acsecr` in `ARCONT`.
9. **Defaults**: `kycmtd = 'Y'`, `kycytd = 'N'`, `kycopy = 1` if not specified.

## Error Handling
- Return specific messages for validation failures:
  - Invalid security code, date, company selection, company numbers, or flags.
- Halt processing and return errors to caller for correction.

## Files Used
- **ARCONT**: Company control (security code, finance charge %, fiscal month).
- **BICONT**: Billing control (invoicing style).
- **ARCUST**: Customer master (name, address, aging, statement flag).
- **ARDETL**: A/R details (transactions, amounts, aging code).
- **AR300S**: Statement parameters.
- **GSCONT**: Global system control (default company).
- **REPORT, REPRT2–5**: Printer files for statement outputs.

