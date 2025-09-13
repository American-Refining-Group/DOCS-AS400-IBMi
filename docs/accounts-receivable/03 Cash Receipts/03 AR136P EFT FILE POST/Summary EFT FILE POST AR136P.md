### List of Use Cases Implemented by the Program

Based on the provided programs (`AR136P.ocl36.txt`, `AR136P.rpgle.txt`, `AR136.ocl36.txt`, `AR136.rpg36.txt`) and their call stack, the primary use case implemented is:

1. **Generate EFT Customer Accounts Receivable Due Report**:
   - This use case involves prompting for and validating a company number and bank upload date, checking the existence of an EFT transaction file, and generating a report listing EFT transactions for customers, including invoice amounts, discounts, and EFT amounts due for a specific bank upload.

### Function Requirement Document

Below is a concise function requirement document for a non-interactive function that implements the above use case, assuming inputs are provided programmatically (e.g., via parameters) rather than through a screen. The document outlines the process steps, business rules, and calculations required to generate the EFT Customer Accounts Receivable Due Report.

<xaiArtifact artifact_id="e24d2856-1215-4660-8630-bba304a86f3c" artifact_version_id="ea31e3a9-3811-4fb0-9c2e-bf4ebfdc31ca" title="EFT_Report_Function_Requirements.md" contentType="text/markdown">

# Function Requirement Document: Generate EFT Customer Accounts Receivable Due Report

## Purpose
The function generates an Accounts Receivable (AR) Electronic Funds Transfer (EFT) report listing transactions due for EFT customers for a specified company and bank upload date, including invoice amounts, discounts, and EFT amounts.

## Inputs
- **Company Number (KYCO)**: 2-digit numeric (positions 101-102 in data area).
- **Bank Upload Date (KYUPDT)**: 6-digit numeric (positions 110-115 in data area, format YYMMDD).
- **Library/Prefix (?9?)**: String defining the library or prefix for file access.

## Outputs
- **EFT Report**: Printed report to the `LIST` printer file, detailing:
  - Header: Company name, bank upload date, run date/time, and column headers.
  - Detail Lines: Sequence number, customer number, customer name, settlement date, invoice number, invoice amount, discount amount, EFT amount.
  - Totals: Sum of invoice amounts, discount amounts, and EFT amounts.
- **Status Flag (STATUS)**: Single-character field (position 109 in data area) set to 'Y' (success) or 'N' (failure).

## Process Steps
1. **Initialize Environment**:
   - Execute setup or validation logic (emulating `GSGENIEC`).
   - Adjust date formats for Y2K compliance (emulating `GSY2K`).
2. **Validate Inputs**:
   - Check if `KYCO` exists in `ARCONT`.
   - Ensure `KYUPDT` is non-zero.
   - Verify the existence of the EFT transaction file (`GE` + `KYUPDT`) using logic similar to `AR135TC`.
3. **Generate Report**:
   - Retrieve company details from `ARCONT` using `KYCO`.
   - Read transactions from `CRTRAN` (file name: `?9?E` + `KYUPDT`).
   - For each transaction, retrieve customer details from `ARCUST` using the transaction’s company and customer number.
   - Calculate EFT amount as invoice amount minus discount.
   - Accumulate totals for invoice, discount, and EFT amounts.
   - Output the report to the `LIST` printer file.
4. **Set Status**:
   - Set `STATUS` to 'Y' if all validations pass and the report is generated; otherwise, set to 'N'.

## Business Rules
1. **Company Validation**:
   - The company number (`KYCO`) must exist in the `ARCONT` file.
   - Failure: Set `STATUS` to 'N' and return error "INVALID COMPANY #".
2. **Bank Upload Date Validation**:
   - The bank upload date (`KYUPDT`) must be non-zero.
   - Failure: Set `STATUS` to 'N' and return error "MUST ENTER BANK UPLOAD DATE".
3. **EFT File Existence**:
   - The transaction file (`GE` + `KYUPDT`) must exist.
   - Failure: Set `STATUS` to 'N' and return error "FILE WITH DATE SELECTED DOES NOT EXIST".
4. **EFT Customer Eligibility**:
   - Only transactions for customers with `AREFT` = 'Y' in `ARCUST` are included in the report.
5. **Report Content**:
   - Include only transactions from `CRTRAN` matching the validated `KYCO` and `KYUPDT`.
   - Display customer name from `ARCUST` and company name from `ARCONT`.
   - Include contact information (e.g., "LISA AT AMERICAN REFINING GROUP") in the report header.
6. **Error Handling**:
   - If any validation fails, set `STATUS` to 'N' and return without generating the report.

## Calculations
- **EFT Amount**:
  - For each transaction in `CRTRAN`:
    - `EFTAMT` = `ATAMT` (invoice amount) - `ATDISC` (discount amount).
- **Report Totals**:
  - Accumulate:
    - `LRIAMT` = Sum of `ATAMT` (total invoice amount).
    - `LRDAMT` = Sum of `ATDISC` (total discount amount).
    - `LREAMT` = Sum of `EFTAMT` (total EFT amount).

## Files Used
- **ARCONT**: AR control file for company details (e.g., `ACNAME`).
- **ARCUST**: Customer master file for customer details (e.g., `ARNAME`, `AREFT`).
- **CRTRAN**: EFT transaction file (`?9?E` + `KYUPDT`) for transaction data (e.g., `ATAMT`, `ATDISC`, `ATINV#`).
- **LIST**: Printer file for report output.

## Side Effects
- Updates `STATUS` field in the data area (position 109) to 'Y' or 'N'.
- No direct updates to `ARCONT`, `ARCUST`, or `CRTRAN` files (read-only access).
- Generates printed report to `LIST`.

## Notes
- The function assumes non-interactive input (e.g., via parameters) instead of screen prompts (replacing `ar136pd` from `AR136P.rpgle.txt`).
- External programs (`GSGENIEC`, `GSY2K`, `AR135TC`) are emulated within the function for setup, date handling, and file existence checks.
- The report format matches `AR136.rpg36.txt`, including headers, detail lines, and totals.

</xaiArtifact>


# Program Summary Table

| Program Name | Purpose | Tables/Files Used | Outputs/Side Effects |
|--------------|---------|-------------------|----------------------|
| **AR136P.ocl36.txt** | Controls the execution of the AR EFT process, validating conditions and calling the RPG program for user input. | ARCONT, GSCONT (via AR136P.rpgle.txt) | Sets data area fields (e.g., STATUS, KYCO, KYUPDT); calls AR136P and AR136; clears local storage. |
| **AR136P.rpgle.txt** | Prompts for and validates company number and bank upload date for EFT processing. | ar136pd (workstation), arcont, gscont | Updates data area STATUS field to 'Y' or 'N'; displays error messages on ar136pd; calls AR135TC. |
| **AR136.ocl36.txt** | Loads and runs the AR136 program to generate the EFT Customer Accounts Receivable Due Report. | CRTRAN, ARCUST, ARCONT | Generates EFT report via AR136; no direct table updates. |
| **AR136.rpg36.txt** | Generates the EFT Customer Accounts Receivable Due Report, listing transactions, discounts, and EFT amounts. | CRTRAN, ARCUST, ARCONT, LIST (printer) | Prints EFT report to LIST printer file; no table updates. |
| **GSGENIEC** | Performs initial setup or validation for the EFT process (not provided). | Unknown (source not provided) | Unknown (likely sets up environment or data area fields); source needed for details. |
| **GSY2K** | Handles date-related processing or Y2K compliance (not provided). | Unknown (source not provided) | Unknown (likely adjusts date formats); source needed for details. |
| **AR135TC** | Checks if the EFT transaction file for the selected date exists (not provided). | Unknown (receives filenm, returns status) | Sets STATUS field in data area to 'Y' or 'N'; source needed for details. |
