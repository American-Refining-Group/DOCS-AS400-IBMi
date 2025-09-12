### List of Use Cases Implemented by the Program Stack

The program stack (`AR050P.ocl36`, `AR050.rpgle`, `AR100.ocl36`, `AR100.rpgle`, `AR105.ocl36`, `AR105.rpg36`, `AR110.ocl36`, `AR110.rpg36`, `AR111`) implements the following use cases for the accounts receivable (AR) system on an IBM System/36 environment:

1. **Process Payment Statements**:
   - Handles batch entry of payment statements (multiple invoices per check) via `AR050`, followed by validation through `AR105` (initial edit), `AR110` (payment validation), and `AR111` (adjustment validation, though disabled). Produces transaction records, edit reports, and Notices of Difference (NODs) for discrepancies.

2. **Process Individual Cash Receipts**:
   - Allows single-transaction entry (e.g., one payment or miscellaneous cash) via `AR100`, followed by validation through `AR110`. Supports interactive entry, validation against master files, and NOD reporting for discrepancies.

### Function Requirement Document for Individual Cash Receipts Processing

<xaiArtifact artifact_id="a87fa812-7b95-41bd-8b86-fbb4db80a2a6" artifact_version_id="36a0b17d-e64e-4bfe-bbcf-9e091809f058" title="Individual_Cash_Receipts_Processing_Requirements.md" contentType="text/markdown">

# Function Requirement Document: Process Individual Cash Receipts

## Overview
The `ProcessIndividualCashReceipts` function processes a single cash receipt transaction (e.g., payment for an invoice or miscellaneous cash) in the AR system, validating inputs, generating transaction records, and producing edit reports or Notices of Difference (NODs). It replaces the interactive screen processing of `AR100.rpgle` and `AR110.rpg36` with programmatic input handling, maintaining Atrium compatibility (`GG` suffixes).

## Inputs
- **Company Number** (`co`: 2-digit numeric): Identifies the company.
- **Customer Number** (`cust`: 6-digit numeric): Identifies the customer.
- **Invoice Number** (`inv#`: 7-digit numeric, optional): Invoice to apply payment to; blank for miscellaneous cash.
- **Amount** (`amt`: 9,2 decimal): Payment amount (must be > 0).
- **Discount Amount** (`disc`: 7,2 decimal, optional): Discount taken (0 for miscellaneous cash).
- **Transaction Date** (`date`: 6-digit numeric, MMDDYY): Payment date.
- **Credit GL Account** (`glcr`: 8-digit numeric, optional): GL account for credit; defaults to AR GL.
- **Debit GL Account** (`gldr`: 8-digit numeric, optional): GL account for debit; defaults to cash GL.
- **Discount GL Account** (`gldi`: 8-digit numeric, optional): GL account for discount; defaults from control file.
- **Check Number** (`ckno`: 6-char): Check identifier.
- **Description** (`desc`: 25-char): Transaction description.
- **Due Date** (`dudt`: 6-digit numeric, MMDDYY, optional): Invoice due date.
- **Terms** (`term`: 2-char): Payment terms (from invoice or customer).
- **Salesman Number** (`slsm`: 2-digit numeric): Salesman code.
- **Notification of Difference** (`nod`: 1-char, 'Y' or blank): Indicates discrepancy requiring NOD.

## Outputs
- **Transaction Record**: Written to `?9?CRIEGG` (256-byte record) with validated data or error flag (`'E'`).
- **Sorted Transaction Record**: Written to `?9?CR10GG` (256-byte record) after sorting.
- **Check Record**: Written to `?9?CRICGG` (96-byte record) for check data.
- **Edit Report**: Lists transaction details, totals (amount, discount, miscellaneous), and errors.
- **NOD Report**: Generated for transactions with `nod = 'Y'`, detailing discrepancies, customer data, and approval requirements.
- **Return Code**: Success (0), Validation Error (1), or System Error (2).
- **Error Messages**: Array of validation errors (e.g., "Invalid Customer Number", "Invalid Invoice").

## Process Steps
1. **Validate Inputs**:
   - Check `co` exists in `ARCONT`; return "Invalid Company Number" if not.
   - Check `cust` exists in `ARCUST`, not deleted (`ARDEL <> 'D'`); return "Invalid Customer Number" if not.
   - If `inv#` non-blank, check exists in `ARDETL`, not deleted (`ADDEL <> 'D'`); return "Invalid Invoice To Pay" or "Invoice Number May Not Be Blank" if invalid/blank.
   - Validate `amt > 0`; return "Invalid Amount, Must Not Be = To Zero" if not.
   - If `disc > 0`, ensure `inv#` is non-blank; return "Misc Cash cannot have Discount Amount" if blank.
   - Validate `date` and `dudt` (if provided): must be valid MMDDYY, Y2K-compliant; return "Invalid Transaction Date" if not.
   - Validate `glcr`, `gldr`, `gldi` (if provided) in `GLMAST`, not deleted (`GLDEL <> 'D'`) or inactive (`GLDEL <> 'I'`); return "Invalid Credit/Debit/Discount G/L Account" if not. Use defaults from `ARCONT` (`ACARGL`, `ACCSGL`, `ACDSGL`) if blank.
   - Check `nod` is `'Y'` or blank; return "Notif. of Diff. must be Blank or 'Y'" if not.

2. **Generate Transaction Record**:
   - Assign unique sequence number (`seq`, 5-digit, incremented by 10).
   - Write to `?9?CRIEGG` (`CRTRAN`): `seq`, `co`, `cust`, `inv#`, `amt`, `disc`, `'P'` (type), `date`, `glcr`, `gldr`, `ckno`, `desc`, `dudt`, `term`, `slsm`, `nod`, 8-digit dates (`date8`, `dudt8`).
   - If validation fails, set error flag (`'E'` in position 54); else set `' '`.

3. **Sort Transactions**:
   - Sort `?9?CRIEGG` into `?9?CR10GG` by company (`co`, positions 7-8) and sequence (`seq`, positions 2-6), including only non-deleted (`atdel <> 'D'`) and payment-type (`attype = 'P'`) records.

4. **Validate Sorted Transactions**:
   - Read `?9?CR10GG`, re-validate against `ARCUST`, `ARDETL`, `GLMAST`, `ARCONT`, `GSTABL`.
   - Check `ckno` in `?9?CRICGG`; retrieve check amount (`CRCKAM`).
   - Update `?9?CRIEGG` with error flag (`'E'`) or clear (`' '`) based on validation.
   - Calculate totals: total amount (`TOTAMT = SUM(amt)`), total discount (`TOTDIS = SUM(disc)`), total cash received (`TOTCAS = SUM(amt + disc)`), total miscellaneous cash (`TOTMIS`, for blank `inv#`), net AR change (`NETCHG`).

5. **Generate Reports**:
   - **Edit Report** (`PRINT`): Lists transactions (`co`, `cust`, `inv#`, `amt`, `disc`, `date`, `glcr`, `gldr`, `ckno`, `desc`, etc.), totals, and errors.
   - **NOD Report** (`NODLIST`, if `nod = 'Y'`): Includes customer details (`ARNAME`, `ARADR1`–`ARADR4`), salesman (`SLNAME`), invoice (`inv#`), check (`ckno`), check amount (`CRCKAM`), discrepancy amount (`ARBAL = amt - invoice balance`), "SHORTPAY"/"OVERPAY" status, approval instructions (sales manager, VP, president for >$2,500), and permanent credit file notice.

6. **Cleanup**:
   - Delete temporary files (`?9?CR10GG`, `?9?CRICGG`) after processing.
   - Return success code (0) or error code (1) with error messages.

## Business Rules
- **Data Validation**:
  - Company, customer, and invoice (if provided) must exist and not be deleted.
  - Amount must be positive; discounts require a valid invoice.
  - GL accounts must be valid, not deleted or inactive (per JB01, 08/24/2016).
  - Dates must be valid and Y2K-compliant.
  - `nod` must be `'Y'` or blank.
- **Transaction Types**:
  - Only payment transactions (`type = 'P'`) are processed; miscellaneous cash allowed with blank `inv#` but no discount.
- **NOD Handling**:
  - Generated for `nod = 'Y'` with discrepancy amount (`ARBAL = amt - invoice balance`).
  - Adjustments over $2,500 require president approval; NODs are permanent credit file records.
- **Calculations**:
  - `TOTAMT = SUM(amt)` across transactions.
  - `TOTDIS = SUM(disc)` across transactions.
  - `TOTCAS = SUM(amt + disc)` per invoice (per MG02, 06/13/2022).
  - `TOTMIS = SUM(amt)` for blank `inv#`.
  - `NETCHG = TOTAMT - TOTDIS` (net AR impact).
- **Atrium Compatibility**: Uses `GG` suffixes (`CRIEGG`, `CR10GG`, `CRICGG`) per JB01 (11/28/23).
- **Multi-User Support**: Shared file access (`DISP-SHR`, `DISP-SHRMM`) for concurrent processing.

## Dependencies
- **Files**:
  - `?9?CRIEGG` (transaction file, 256 bytes, 5-byte key).
  - `?9?CR10GG` (sorted transactions, 256 bytes).
  - `?9?CRICGG` (check file, 96 bytes, 1,000 records).
  - `?9?ARCUST` (customer master, 384 bytes).
  - `?9?ARDETL` (AR detail, 128 bytes).
  - `?9?GLMAST` (GL master, 256 bytes).
  - `?9?ARCONT` (AR control, 256 bytes).
  - `?9?GSTABL` (system table, 256 bytes).
  - `PRINT`, `NODLIST` (printer files, 164 bytes).
- **Utilities**: `GSY2K` (Y2K compliance), `#GSORT` (sorting).

## Notes
- Replaces interactive screens (`AR100FM`, `AR110`) with programmatic inputs.
- Assumes `AR1101` handles post-validation tasks (e.g., posting), not included in this function.
- Adjustment processing (`AR111`) is disabled per `AR050` notes.

</xaiArtifact>

## Call Stack Program Listing and Purpose

| Program Name       | Main Purpose                                                                 | Brief Description          |
|--------------------|------------------------------------------------------------------------------|----------------------------|
| AR050P.ocl36.txt  | Orchestrates the cash receipts prompt process, initializes environment, checks batch status, builds GL distribution files, and conditionally calls payment statement (AR050) or individual entry (AR100) programs. | Cash receipts prompt       |
| AR050P.rpgle.txt  | Handles cash receipts entry prompt for GL account validation (debit, credit, discount), defaults from AR control, and updates GL distribution file with user input. | GL account validation      |
| AR050.ocl36.txt   | Sets up payment statement entry environment by building indexes and work files for customers, invoices, and checks, then loads and runs AR050 for batch payment processing. | Payment statement setup    |
| AR050.rpgle.txt   | Manages interactive payment statement entry, customer/invoice selection, payment code assignment (P/pay, H/hold), amount application, check processing, and validation with error reporting. | Batch payment entry        |
| AR100.ocl36.txt   | Prepares individual cash receipts entry by initializing switches, building transaction work file, and loading AR100 with shared access to customer, GL, and AR files. | Individual entry setup     |
| AR100.rpgle.txt   | Provides interactive screen for single cash receipt entry, validates customer/invoice/GL accounts, handles add/update modes, and generates transactions with error messages. | Single receipt entry       |
| AR110.ocl36.txt   | Edits individual entries by deleting invalid records, sorting transactions, building check work file, and running AR110 for validation and NOD report generation. | Individual entry edit      |
| AR110.rpg36.txt   | Validates sorted individual transactions against master files (customer, invoice, GL), flags errors, calculates totals, and generates edit and NOD reports for discrepancies. | Transaction validation      |
| AR105.ocl36.txt   | Edits payment statement entries by validating payments via AR105, sorting and validating payments/adjustments via AR110/AR111, and producing reports with cleanup. | Payment statement edit     |
| AR105.rpg36.txt   | Processes payment statements from work files, validates checks/invoices, generates payment/unapplied cash/finance charge transactions, and writes to output file for further validation. | Payment processing         |