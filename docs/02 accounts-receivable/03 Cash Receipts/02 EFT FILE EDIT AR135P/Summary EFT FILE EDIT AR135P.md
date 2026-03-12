### Use Cases Implemented by the AR135P Program Call Stack

The call stack consists of several programs (`AR135P`, `AR135`, `AR135B`, `AR135A`, and `AR135TC`) that collectively implement the Accounts Receivable (AR) Electronic Funds Transfer (EFT) draft automation process. Based on the provided OCL and RPG source files, the primary use case is:

1. **EFT Draft Automation for Accounts Receivable**:
   - **Description**: This use case automates the processing of EFT transactions for AR, allowing users to enter and validate cash receipt transactions, update transaction records with relevant dates, and generate a pre-upload report for bank submission (e.g., to PNC). It includes input validation, transaction processing, and reporting to ensure accurate and compliant EFT draft processing.
   - **Components**:
     - `AR135P`: Captures and validates initial parameters (company number and bank upload date).
     - `AR135`: Provides an interactive interface for entering and updating EFT transaction details.
     - `AR135B`: Updates transaction records with select and update dates.
     - `AR135A`: Generates a detailed report of EFT transactions for verification before bank upload.
     - `AR135TC`: Validates the existence of the EFT transaction file.

This is the primary use case, as the programs are tightly integrated to support the EFT process from parameter input to final reporting. No additional distinct use cases are evident, as the programs focus on a single cohesive workflow.

---

### Function Requirement Document: EFT Draft Automation Function

<xaiArtifact artifact_id="114f337f-1e2d-4665-8fa4-3cc80eb2ae20" artifact_version_id="94e4fc4e-2643-4d9c-bbdb-1607ccb0b37c" title="EFT_Draft_Automation_Requirements.md" contentType="text/markdown">

# EFT Draft Automation Function Requirements

## Overview
The EFT Draft Automation function processes Accounts Receivable (AR) Electronic Funds Transfer (EFT) transactions for customer payments, validating inputs, updating transaction records, and generating a pre-upload report for bank submission. It replaces interactive screen inputs with programmatic inputs to streamline the process.

## Inputs
- **Company Number (KYCO)**: 2-digit numeric, identifies the company.
- **Bank Upload Date (KYUPDT)**: 6-digit numeric (MMDDYY), date for EFT submission.
- **Test/Production Flag (TSTPRD)**: 1-character, indicates environment (e.g., 'P' for production).
- **Transaction Data**:
  - Sequence Number (ATSEQ#): 5-digit numeric, unique transaction identifier.
  - Customer Number (ATCUST): 6-digit numeric, identifies the customer.
  - Invoice Number (ATINV#): 7-digit numeric, identifies the invoice (blank for misc cash).
  - Amount (ATAMT): 10-digit packed decimal (2 decimals), transaction amount.
  - Discount (ATDISC): 7-digit packed decimal (2 decimals), discount amount.
  - Type (ATTYPE): 1-character, 'P' for cash receipt or misc cash.
  - Transaction Date (ATDATE): 6-digit numeric (MMDDYY), date of transaction.
  - Due Date (ATDUDT): 6-digit numeric (MMDDYY), invoice due date.
  - Terms (ATTERM): 2-digit numeric, payment terms code.
  - Credit G/L Account (ATGLCR): 8-digit numeric, G/L account for credit.
  - Debit G/L Account (ATGLDR): 8-digit numeric, G/L account for debit.
  - Discount Company (ATCODI): 2-digit numeric, company for discount.
  - Discount G/L Account (ATGLDI): 8-digit numeric, G/L account for discount.
  - Credit Company (ATCOCR): 2-digit numeric, intercompany credit.
  - Debit Company (ATCODR): 2-digit numeric, intercompany debit.
  - Notification of Difference (ATNOD): 1-character, 'Y' or blank.
- **Select Date (KYSEDT)**: 6-digit numeric (MMDDYY), date transactions are selected for EFT.

## Outputs
- **Updated Transaction File (CRTRAN)**: Updated with transaction details and dates (KYUPDT, KYSEDT).
- **EFT Pre-Upload Report (LIST3)**: Printed report with headers, transaction details, and totals (invoice amount, discount, EFT amount).
- **Status Code**: Indicates success or error with error messages (e.g., "INVALID COMPANY #").

## Process Steps
1. **Validate Inputs**:
   - Verify `KYCO` exists in `ARCONT` (company master).
   - Ensure `KYUPDT` is non-zero and valid (MMDDYY).
   - Check EFT transaction file (`GE + TSTPRD + E + KYUPDT`) exists via `AR135TC`.
   - Validate `ATCUST` exists in `ARCUST` and `AREFT = 'Y'` (EFT participant).
   - Ensure `ATINV#` is non-blank for non-misc cash (`ATTYPE = 'P'`).
   - Verify `ATAMT > 0`, `ATDISC = 0` for misc cash.
   - Validate `ATGLCR`, `ATGLDR`, `ATGLDI` exist in `GLMAST`.
   - Confirm `ATCOCR`, `ATCODR`, `ATCODI` are valid companies in `ARCONT`.
   - Check `ATDATE` and `ATDUDT` are valid (MMDDYY, accounts for leap years using `Y2KCEN`, `Y2KCMP`).
   - Ensure `ATNOD` is 'Y' or blank.
   - Verify `ATSEQ#` is unique in `CRTRAN`.

2. **Process Transactions**:
   - Write validated transaction data to `CRTRAN`, including `ATDEL` (delete flag, blank unless marked 'D').
   - Update `ARDETL` with payment amounts (`ADPAY`, `ADPART`) and terms (`ADTERM`).
   - Update `CRTRAN` with `KYUPDT` and `KYSEDT` for selected transactions.

3. **Calculate Totals**:
   - For each transaction: `EFTAMT = ATAMT - ATDISC`.
   - Accumulate totals:
     - `LRIAMT += ATAMT` (total invoice amount).
     - `LRDAMT += ATDISC` (total discount).
     - `LREAMT += EFTAMT` (total EFT amount).

4. **Generate Report**:
   - Output to `LIST3`:
     - **Header**: Company name (`ACNAME` from `ARCONT`), report title ("ELECTRONIC FUNDS WORKFILE PRE-UPLOAD TO PNC EDIT REPORT"), contact info, run date/time (`SYSDTE`, `SYSTME`), bank upload date (`KYUPDT`).
     - **Detail Lines**: For each `CRTRAN` record (where `ATDEL ≠ 'D'` and `AREFT = 'Y'`), print `ATSEQ#`, `ATCUST`, `ARNAME` (from `ARCUST`), `KYSEDT`, `ATINV#`, `ATAMT`, `ATDISC`, `EFTAMT`.
     - **Totals**: Print `LRIAMT`, `LRDAMT`, `LREAMT`.

5. **Return Status**:
   - Return success or error with messages (e.g., "INVALID CUSTOMER NUMBER", "FILE WITH DATE SELECTED DOES NOT EXIST").

## Business Rules
- **Validation**:
  - Company (`KYCO`), customer (`ATCUST`), and G/L accounts must exist in respective files (`ARCONT`, `ARCUST`, `GLMAST`).
  - Bank upload date (`KYUPDT`) and transaction file must be valid.
  - Invoice number (`ATINV#`) required for non-misc cash; misc cash cannot have discounts.
  - Dates (`ATDATE`, `ATDUDT`, `KYUPDT`, `KYSEDT`) must be valid MMDDYY, with Y2K compliance.
  - `ATNOD` restricted to 'Y' or blank.
  - Sequence numbers (`ATSEQ#`) must be unique in `CRTRAN`.

- **EFT Eligibility**:
  - Only customers with `AREFT = 'Y'` in `ARCUST` are processed.
  - Transactions marked deleted (`ATDEL = 'D'`) are excluded from reporting.

- **Calculations**:
  - EFT amount: `EFTAMT = ATAMT - ATDISC`.
  - Totals: Sum `ATAMT`, `ATDISC`, and `EFTAMT` across transactions.

- **Environment Flexibility**:
  - Supports multiple environments via `TSTPRD` (e.g., production/test file labels).
  - File labels use dynamic prefixes (e.g., `?9?E?L'110,6'?`).

- **Data Integrity**:
  - Updates `CRTRAN` and `ARDETL` only after validation.
  - Shared file access (`DISP-SHR`) ensures concurrent processing.

- **Reporting**:
  - Report includes customer details, transaction data, and totals for bank verification.
  - Excludes deleted transactions and non-EFT customers.

## Files Used
- **ARCONT**: Validates company number, provides company name (`ACNAME`), G/L accounts.
- **GSCONT**: Provides default company number (`GXCONO`).
- **CRTRAN**: Stores EFT transactions; updated with dates and details.
- **CRTRANR**: Reference transaction file for reads/comparisons.
- **ARCUST**: Validates customers, provides name (`ARNAME`), EFT flag (`AREFT`).
- **ARCUSTX**: Customer index for name/address lookup.
- **GLMAST**: Validates G/L accounts (`ATGLCR`, `ATGLDR`, `ATGLDI`).
- **ARDETL**: Updates payment details (`ADPAY`, `ADPART`, `ADTERM`).
- **LIST3**: Output printer file for EFT report.

## External Dependencies
- **AR135TC**: Validates EFT transaction file existence, returns status ('N' for not found).

</xaiArtifact>





## EFT Program Summary Table

| Program Name | Purpose | Tables/Files Used | Outputs/Side Effects |
|--------------|---------|-------------------|---------------------|
| **AR135P** | Validates initial EFT parameters (company and bank upload date) via interactive screen. | `ARCONT`, `GSCONT`, `AR135PD` (workstation) | Displays validation errors on `AR135PD`; no direct table updates; calls `AR135TC` for file validation. |
| **AR135** | Provides an interactive interface for entering and updating EFT transaction details. | `AR135FM` (workstation), `CRTRAN`, `CRTRANR`, `ARCUST`, `ARCUSTX`, `ARCONT`, `GLMAST`, `ARDETL` | Updates `CRTRAN` with new/updated transactions; updates `ARDETL` with payment details (`ADPAY`, `ADPART`); displays transaction data and errors on `AR135FM`. |
| **AR135B** | Updates EFT transaction records with select and update dates. | `CRTRAN` | Updates `CRTRAN` with `KYUPDT` (positions 228-233) and `KYSEDT` (positions 234-239). |
| **AR135A** | Generates a pre-upload EFT report for bank submission verification. | `CRTRAN`, `ARCUST`, `ARCONT`, `LIST3` (printer) | Outputs EFT report to `LIST3` with headers, transaction details, and totals (`LRIAMT`, `LRDAMT`, `LREAMT`); no table updates. |