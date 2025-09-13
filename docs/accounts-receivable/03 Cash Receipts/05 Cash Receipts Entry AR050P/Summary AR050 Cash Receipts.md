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
### Conditional flows based on mode:
  - **Payment Statement Path**: AR050P.ocl36 → AR050P.rpgle (if GL needed) → AR050.ocl36 → AR050.rpgle → AR105.ocl36 → AR105.rpg36 → AR110.ocl36 → AR110.rpg36.
  - **Individual Entry Path**: AR050P.ocl36 → AR100.ocl36 → AR100.rpgle → AR110.ocl36 → AR110.rpg36.
  - **Implied Calls**: AR0501 (by AR050.rpgle for invoice display), AR1101 (by AR110.rpg36 for posting), AR111 (by AR105.ocl36, disabled).

| Program Name       | Main Purpose                                                                 | Tables (Files) Used                                                                 | Outputs (Files or Side Effects)                                   |
|--------------------|------------------------------------------------------------------------------|------------------------------------------------------------------------------------|------------------------------------------------------------------|
| AR050P.ocl36       | Orchestrates cash receipts mode selection and environment setup.              | ?9?CRSTGG, ?9?GLD1GG                                                               | Calls AR050P.rpgle, AR050.ocl36, or AR100.ocl36; deletes temporary files. |
| AR050P.rpgle       | Validates GL accounts for cash receipts and updates distribution file.        | ?9?GLD1GG, ?9?GLMAST, ?9?ARCONT                                                    | Updates ?9?GLD1GG; displays errors for correction.                |
| AR050.ocl36        | Sets up environment for batch payment statement entry.                        | ?9?SA5SHX, ?9?SA5FIND, ?9?CRCKGG, ?9?CRWKGG, ?9?ARCUST, ?9?ARDETL, ?9?ARCONT      | Builds/rebuilds ?9?SA5SHX, ?9?SA5FIND, ?9?CRWKGG, ?9?CRCKGG; calls AR050.rpgle. |
| AR050.rpgle        | Manages interactive batch payment entry and validation.                       | ?9?SA5SHX, ?9?SA5FIND, ?9?CRWKGG, ?9?CRCKGG, ?9?ARCUST, ?9?ARDETL, ?9?ARCONT      | Writes to ?9?CRWKGG, ?9?CRCKGG; displays errors; calls AR0501 (if F11 pressed). |
| AR100.ocl36        | Prepares environment for individual cash receipts entry.                      | ?9?CRIEGG, ?9?ARDETL, ?9?ARCUST, ?9?GLMAST, ?9?ARCONT                              | Builds ?9?CRIEGG; calls AR100.rpgle; deletes temporary files.     |
| AR100.rpgle        | Handles interactive entry of single cash receipts with validation.            | ?9?ar100d, ?9?CRIEGG, ?9?ARDETL, ?9?ARCUST, ?9?GLMAST, ?9?ARCONT                  | Writes to ?9?CRIEGG; displays errors and totals on screen.        |
| AR105.ocl36        | Edits payment statement entries by validating and sorting transactions.       | ?9?CRWKGG, ?9?CRCKGG, ?9?CRPSGG, ?9?CR10GG, ?9?AR11GG, ?9?ARCONT, ?9?ARCUST, ?9?ARDETL, ?9?GLMAST, ?9?GSTABL, ?9?SA5FIND, ?9?SA5FINM, ?9?SA5SHX, ?9?XGSTABL | Deletes/rebuilds temporary files; calls AR105.rpg36, AR110.ocl36, AR111. |
| AR105.rpg36        | Validates payment statements and generates transaction records.               | ?9?CRCHKS, ?9?CRWORK, ?9?ARCONT, ?9?ARDETL, ?9?CRTRAN, ?9?ARCUST                  | Writes to ?9?CRPSGG (CRTRAN); error flags for invalid data.       |
| AR110.ocl36        | Edits individual cash receipt entries by sorting and validating transactions. | ?9?CRIEGG, ?9?CR10GG, ?9?CRICGG, ?9?ARDETL, ?9?ARCUST, ?9?GLMAST, ?9?ARCONT, ?9?GSTABL, ?9?SA5FIND, ?9?SA5FINM, ?9?SA5SHX, ?9?XGSTABL, ?9?GSPROD | Deletes/rebuilds ?9?CR10GG, ?9?CRICGG; calls AR110.rpg36, AR1101 (implied). |
| AR110.rpg36        | Validates sorted individual transactions and generates reports.               | ?9?CR110S, ?9?CRTRAN, ?9?ARDETL, ?9?ARCUST, ?9?GLMAST, ?9?ARCONT, ?9?GSTABL, ?9?CRCHKS, PRINT, NODLIST | Updates ?9?CRIEGG (CRTRAN); generates edit (PRINT) and NOD (NODLIST) reports. |




# Function Requirement Document: Process Payment Statements

## Overview
The `ProcessPaymentStatements` function processes batch payment statements (multiple invoices per check) in the AR system, validating checks and invoices, generating transaction records, and producing edit reports and Notices of Difference (NODs) for discrepancies. It replaces the interactive screen processing of `AR050.rpgle`, `AR105.rpg36`, and `AR110.rpg36` with programmatic input handling, maintaining Atrium compatibility (`GG` suffixes). Adjustment transactions (`type = 'J'`) are disabled per `AR050.rpgle` notes (March 2024).

## Inputs
- **Check Records** (array of):
  - **Company Number** (`crco`: 2-digit numeric): Company identifier.
  - **Customer Number** (`crcust`: 6-digit numeric): Customer identifier.
  - **Check Number** (`crckno`: 6-char): Unique check identifier.
  - **Check Amount** (`crckam`: 9,2 decimal): Total check amount (> 0).
  - **Discount Amount** (`crdsam`: 7,2 decimal): Total discount taken.
  - **Payment Date** (`crpydt`: 6-digit numeric, MMDDYY): Check date.
  - **Applied Amount** (`crappl`: 9,2 decimal): Amount applied to invoices.
  - **Gross Amount** (`crgrss`: 9,2 decimal): Gross check amount.
  - **Foreign Currency Amount** (`crfcam`: 9,2 decimal): Finance charge amount (if any).
- **Invoice Records** (array per check):
  - **Company Number** (`awco`: 2-digit numeric): Matches `crco`.
  - **Customer Number** (`awcust`: 6-digit numeric): Matches `crcust`.
  - **Invoice Number** (`awinv#`: 7-digit numeric): Invoice to apply payment to.
  - **Due Date** (`awdudt`: 6-digit numeric, MMDDYY): Invoice due date.
  - **Amount Applied** (`awampd`: 9,2 decimal): Payment amount for invoice (> 0).
  - **Discount Amount** (`awdsam`: 7,2 decimal): Discount for invoice.
  - **Payment Code** (`awpaid`: 1-char, 'P'): Must be 'P' (adjustments disabled).
  - **Check Date** (`awpddt`: 6-digit numeric, MMDDYY): Matches `crpydt`.
  - **Check Number** (`awckno`: 6-char): Matches `crckno`.
  - **Override GL** (`awglno`: 8-digit numeric, optional): Credit GL override.
  - **Notification of Difference** (`awnod`: 1-char, 'Y' or blank): Discrepancy flag.
- **Default GL Accounts** (from `ARCONT` or input):
  - **AR GL** (`acargl`: 8-digit numeric): Default credit GL.
  - **Cash GL** (`accsgl`: 8-digit numeric): Default debit GL.
  - **Discount GL** (`acdsgl`: 8-digit numeric): Default discount GL.
  - **Finance GL** (`acfngl`: 8-digit numeric): Finance charge GL.

## Outputs
- **Transaction Records**: Written to `?9?CRPSGG` (256-byte records) for payments, unapplied cash, and finance charges.
- **Sorted Payment Records**: Written to `?9?CR10GG` (256-byte records) after sorting.
- **Edit Report**: Lists transactions, totals (amount, discount, cash received, miscellaneous, unapplied), and errors.
- **NOD Report**: Generated for transactions with `awnod = 'Y'`, detailing discrepancies, customer data, and approval requirements.
- **Return Code**: Success (0), Validation Error (1), or System Error (2).
- **Error Messages**: Array of validation errors (e.g., "Invalid Customer Number", "Invalid Invoice").

## Process Steps
1. **Validate Check and Invoice Inputs**:
   - **Check Validation**:
     - Ensure `crco` exists in `ARCONT`; return "Invalid Company Number" if not.
     - Ensure `crcust` exists in `ARCUST`, not deleted (`ARDEL <> 'D'`); return "Invalid Customer Number" if not.
     - Verify `crckno` is unique, `crckam > 0`, `crpydt` is valid (MMDDYY, Y2K-compliant); return errors if invalid.
   - **Invoice Validation**:
     - Ensure `awco = crco`, `awcust = crcust`, `awckno = crckno`, `awpddt = crpydt`, `awpaid = 'P'`.
     - Verify `awinv#` exists in `ARDETL`, not deleted (`ADDEL <> 'D'`); return "Invalid Invoice To Pay".
     - Ensure `awampd > 0`, `awdudt` is valid; return "Invalid Amount" or "Invalid Transaction Date".
     - Validate `awglno` (if provided) in `GLMAST`, not deleted (`GLDEL <> 'D'`) or inactive (`GLDEL <> 'I'`); return "Invalid Credit G/L Account". Use `acargl` (AR GL) if blank.
     - Ensure `awnod` is `'Y'` or blank; return "Notif. of Diff. must be Blank or 'Y'".
     - Verify `crappl = SUM(awampd)` across invoices for the check.

2. **Generate Transaction Records (`AR105`)**:
   - For each check in `CRCHKS`:
     - Write payment transactions to `?9?CRPSGG` (`ARPAY`):
       - Fields: `'A'` (active), `seq` (increment by 10), `awco`, `awcust`, `awinv#`, `awampd`, `awdsam`, `'P'`, `awpddt`, `awglno` or `acargl` (credit GL), `accsgl` (debit GL), `awckno`, `awdudt`, `adterm` (terms), `adsls` (salesman), `acdsgl` (discount GL), `awpdd8`, `awdud8`, `awnod`.
     - If unapplied amount (`L2UNAP = crckam - SUM(awampd) > 0`), write unapplied cash (`ARUNAP`):
       - Fields: `'A'`, `seq`, `crco`, `crcust`, `'9999999'` (dummy invoice), `L2UNAP`, `0`, `'P'`, `crpydt`, `acargl`, `accsgl`, `crckno`, `crpydt`, `term`, `sls`, `0`, `crpyd8`, `crpyd8`, `' '`.
     - If `crfcam > 0`, write finance charge (`UPDFC`):
       - Fields: `'A'`, `seq`, `crco`, `crcust`, `0`, `crfcam`, `0`, `'P'`, `crpydt`, `acfngl`, `accsgl`, `'FIN CHG'`, `' '`, `crcust`, `crpydt`, `term`, `sls`, `0`, `crpyd8`, `crpyd8`, `' '`.
     - Mark invalid records with `'E'` in position 54.

3. **Sort Payment Transactions**:
   - Sort `?9?CRPSGG` into `?9?CR10GG` by company/customer/invoice (positions 7-21), including only non-deleted (`atdel <> 'D'`) and payment-type (`attype = 'P'`) records.

4. **Validate Sorted Payments (`AR110`)**:
   - Read `?9?CR10GG`, re-validate against `ARCUST`, `ARDETL`, `GLMAST`, `ARCONT`, `GSTABL`, `CRCHKS`.
   - Update `?9?CRPSGG` with error flag (`'E'`) or clear (`' '`) based on validation.
   - Calculate totals:
     - `TOTAMT = SUM(awampd)`: Total payment amount.
     - `TOTDIS = SUM(awdsam)`: Total discount.
     - `TOTCAS = SUM(awampd + awdsam)`: Total cash received per invoice (per MG02, 06/13/2022).
     - `TOTMIS = SUM(awampd)`: For dummy invoices (`'9999999'`).
     - `TOTARC = SUM(awampd - awdsam)`: AR impact.
     - `NETCHG = TOTAMT - TOTDIS`: Net AR change.

5. **Generate Reports**:
   - **Edit Report** (`PRINT`): Lists transactions (`awco`, `awcust`, `awinv#`, `awampd`, `awdsam`, `awpddt`, GLs, `awckno`, etc.), totals, and errors.
   - **NOD Report** (`NODLIST`, if `awnod = 'Y'`): Includes customer details (`ARNAME`, `ARADR1`–`ARADR4`), salesman (`SLNAME`), invoice (`awinv#`), check (`crckno`), check amount (`crckam`), discrepancy amount (`ARBAL = awampd - invoice balance`), "SHORTPAY"/"OVERPAY" status, approval instructions (sales manager, VP, president for >$2,500), and permanent credit file notice.

6. **Cleanup**:
   - Delete temporary files (`?9?CR10GG`) after processing.
   - Return success code (0) or error code (1) with error messages.

## Business Rules
- **Data Validation**:
  - Company, customer, and invoice must exist and not be deleted.
  - Check amount (`crckam`) must be positive; payment date (`crpydt`) and due date (`awdudt`) must be valid and Y2K-compliant.
  - Amount applied (`awampd`) must be positive; discount (`awdsam`) requires valid invoice.
  - GL accounts (`awglno`, `acargl`, `accsgl`, `acdsgl`, `acfngl`) must be valid, not deleted or inactive (per JB01, 08/24/2016).
  - `awnod` must be `'Y'` or blank.
- **Transaction Types**:
  - Only payment transactions (`awpaid = 'P'`) are processed; adjustments (`'J'`) are disabled.
  - Unapplied cash uses dummy invoice (`'9999999'`); finance charges use `acfngl` and description `'FIN CHG'`.
- **NOD Handling**:
  - Generated for `awnod = 'Y'` with discrepancy amount (`ARBAL`).
  - Adjustments over $2,500 require president approval; NODs are permanent credit file records.
- **Calculations**:
  - `L2UNAP = crckam - SUM(awampd)`: Unapplied cash per check.
  - `TOTAMT = SUM(awampd)`, `TOTDIS = SUM(awdsam)`, `TOTCAS = SUM(awampd + awdsam)`, `TOTMIS = SUM(awampd for dummy invoices)`, `TOTARC = SUM(awampd - awdsam)`, `NETCHG = TOTAMT - TOTDIS`.
- **Atrium Compatibility**: Uses `GG` suffixes (`CRWKGG`, `CRCKGG`, `CRPSGG`, `CR10GG`) per JB01 (11/28/23).
- **Multi-User Support**: Shared file access (`DISP-SHR`, `DISP-SHRMM`) for concurrent processing.

## Dependencies
- **Files**:
  - `?9?CRWKGG` (invoice work file, 96 bytes).
  - `?9?CRCKGG` (check work file, 96 bytes).
  - `?9?CRPSGG` (transaction file, 256 bytes).
  - `?9?CR10GG` (sorted payments, 256 bytes).
  - `?9?ARCUST` (customer master, 384 bytes).
  - `?9?ARDETL` (AR detail, 128 bytes).
  - `?9?GLMAST` (GL master, 256 bytes).
  - `?9?ARCONT` (AR control, 256 bytes).
  - `?9?GSTABL` (system table, 256 bytes).
  - `PRINT`, `NODLIST` (printer files, 164 bytes).
- **Utilities**: `GSY2K` (Y2K compliance), `#GSORT` (sorting).

## Notes
- Replaces interactive screens (`AR050FM`, `AR110`) with programmatic inputs.
- Assumes `AR1101` handles post-validation tasks (e.g., posting), not included in this function.
- Adjustment processing (`AR111`) is disabled per `AR050` notes.

---

This document concisely captures the business requirements and calculations for the **Process Payment Statements** use case, aligning with the program stack's functionality while adapting to a programmatic input model. Let me know if further clarification is needed!