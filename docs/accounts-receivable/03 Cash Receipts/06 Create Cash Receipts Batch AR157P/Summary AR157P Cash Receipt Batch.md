### List of Use Cases Implemented by the Program Suite

The program suite (`AR157P.ocl36`, `AR157P.rpgle`, `AR157.ocl36`, `AR157A.ocl36`, `AR157A.rpg36`) implements a single primary use case for creating an EFT (Electronic Funds Transfer) cash receipts batch file in an IBM midrange system (e.g., AS/400, iSeries). The use case is:

1. **Create EFT Cash Receipts Batch File**:
   - **Description**: This use case enables the creation of a cash receipts batch file for EFT transactions by validating user inputs (company number and bank upload date), adjusting transaction data, copying non-deleted records to the batch file, and cleaning up temporary files.
   - **Inputs**: Company number (`KYCO`), bank upload date (`KYUPDT`).
   - **Outputs**: A cash receipts batch file (`?9?CRIEGG`) containing validated and adjusted EFT transaction records, ready for further processing (e.g., transmission to a bank).
   - **Components**:
     - `AR157P.ocl36`: Orchestrates the process, validates environment, and calls other programs.
     - `AR157P.rpgle`: Validates user inputs (company number, upload date, file existence) via a workstation interface.
     - `AR157.ocl36`: Copies non-deleted records from a temporary file to the batch file and deletes the temporary file.
     - `AR157A.ocl36` and `AR157A.rpg36`: Adjust fields in the temporary file to ensure EFT compliance.

---

### Function Requirement Document: Create EFT Cash Receipts Batch File

<xaiArtifact artifact_id="f87dadc4-073d-4650-a723-d967d1bbc45b" artifact_version_id="196666ab-846d-40c9-90fe-a6b8d19ffc57" title="EFT_Cash_Receipts_Batch_Creation_Requirements.md" contentType="text/markdown">

# EFT Cash Receipts Batch Creation Function Requirements

## Purpose
The `CreateEFTCashReceiptsBatch` function creates a cash receipts batch file for EFT transactions by validating inputs, adjusting transaction data, and producing a batch file for banking or accounting purposes.

## Inputs
- **Company Number** (`KYCO`): 2-digit numeric identifier for the company (e.g., `01`).
- **Bank Upload Date** (`KYUPDT`): 6-digit numeric date in `YYYYMM` format (e.g., `202311` for November 2023).

## Outputs
- **Cash Receipts Batch File**: File named `QS36F/?9?CRIEGG` (e.g., `QS36F/CO123CRIEGG`), containing EFT transaction records.
- **Status Flag** (`STATUS`): `'Y'` (success) or `'N'` (failure).
- **Error Message** (`MSG`): Descriptive message for validation failures (e.g., "INVALID COMPANY #").

## Process Steps
1. **Validate Inputs**:
   - Check if `KYCO` exists in the `ARCONT` file (Accounts Receivable control data).
   - Ensure `KYUPDT` is non-zero.
   - Verify the existence of a temporary file (`QS36F/?9?EYYYYMM`, e.g., `QS36F/CO123E202311`) for the given `KYCO` and `KYUPDT`.

2. **Handle Existing Batch File**:
   - If a batch file (`QS36F/?9?CRIEGG`) exists, set `EXISTS` to `'Y'` and return an error: "A CASH RECEIPTS BATCH ALREADY EXISTS."
   - Allow deletion of the existing batch file if requested (via `KYDELT = 'Y'`).

3. **Adjust Transaction Fields**:
   - Process the temporary file (`CRTRAN`, labeled `QS36F/?9?EYYYYMM`).
   - Update fields (e.g., `KYUPDT`, transaction amount `ATAMT`, G/L accounts) to ensure EFT compliance.
   - Preserve non-deleted records (`ATDEL ≠ 'D'`).

4. **Copy Records to Batch File**:
   - Copy non-deleted records (`ATDEL ≠ 'D'`) from `QS36F/?9?EYYYYMM` to `QS36F/?9?CRIEGG`.
   - Create the batch file if it does not exist; replace its contents if it exists.

5. **Clean Up**:
   - Delete the temporary file (`QS36F/?9?EYYYYMM`) if it exists.
   - Clear all local variables to reset the environment.

## Business Rules
1. **Input Validation**:
   - `KYCO` must exist in `ARCONT` or return "INVALID COMPANY #".
   - `KYUPDT` must be non-zero or return "MUST ENTER BANK UPLOAD DATE".
   - Temporary file (`QS36F/?9?EYYYYMM`) must exist or return "FILE WITH DATE SELECTED DOES NOT EXIST".

2. **Batch File Uniqueness**:
   - Only one batch file (`QS36F/?9?CRIEGG`) can exist per company and run. If it exists, processing halts unless deletion is requested.

3. **Record Filtering**:
   - Only non-deleted records (`ATDEL ≠ 'D'`) are copied to the batch file.

4. **Dynamic File Naming**:
   - Files use `?9?` (company code, e.g., `CO123`) and `YYYYMM` (bank upload date, e.g., `202311`) for naming (e.g., `QS36F/CO123E202311`, `QS36F/CO123CRIEGG`).
   - The suffix `GG` (replacing `WS`) aligns with the Atrium environment.

5. **Shared Access**:
   - Files (`ARCONT`, `GSCONT`, `CRTRAN`) are accessed in shared mode (`DISP-SHR`) to support multi-user environments.

6. **Environment Reset**:
   - Local variables are cleared after processing to prevent data leakage.

## Calculations
- **File Name Construction**:
  - Temporary file: Concatenate `?9?` (company code) + `'E'` + `KYUPDT` (e.g., `CO123E202311`).
  - Batch file: Concatenate `?9?` + `'CRIEGG'` (e.g., `CO123CRIEGG`).
- **Field Adjustments**:
  - Update `KYUPDT` in `CRTRAN` to match input `KYUPDT` at position 37.
  - Other fields (e.g., `ATAMT`, `ATDISC`, G/L accounts) may be reformatted or recalculated for EFT compliance (specific calculations not visible in provided code).

## Error Handling
- Return `STATUS = 'N'` and an appropriate error message for validation failures.
- If `EXISTS = 'Y'`, return "A CASH RECEIPTS BATCH ALREADY EXISTS." and await deletion confirmation (`KYDELT = 'Y'`).

## Dependencies
- **Files**:
  - `ARCONT`: Validates company number (`KYCO`).
  - `GSCONT`: Provides default company number for initialization.
  - `CRTRAN` (`QS36F/?9?EYYYYMM`): Temporary file for EFT transaction data.
  - `QS36F/?9?CRIEGG`: Target batch file.
- **External Program**:
  - `AR135TC`: Checks existence of the temporary file.

## Assumptions
- Inputs (`KYCO`, `KYUPDT`) are provided programmatically, not via a screen.
- The temporary file (`CRTRAN`) contains valid EFT transaction records before processing.
- The `AR135TC` program returns `STATUS = 'Y'` (file exists) or `'N'` (file does not exist).

</xaiArtifact>


## EFT Cash Receipts Batch File Creation Program Suite Summary

| **Program** | **Call Order** | **Main Purpose** | **Tables/Files Used** | **Outputs (Files or Side Effects)** |
|-------------|----------------|------------------|-----------------------|-------------------------------------|
| **AR157P.ocl36** | 3 | Orchestrates the EFT batch creation process, calling validation and processing programs. | - `ARCONT` (label: `?9?ARCONT`, shared)<br>- `?9?CRIEGG` (library: `DATAF1`) | - Sets local variables (e.g., `EXISTS = 'Y'`).<br>- Deletes `?9?CRIEGG` if requested.<br>- Error message if batch file exists.<br>- Calls `AR157P.rpgle` and `AR157.ocl36`. |
| **AR157P.rpgle** | 4 | Validates user inputs (company number, bank upload date) and checks file existence. | - `AR157PFM` (workstation file, combined)<br>- `ARCONT` (input, 256 bytes, keyed)<br>- `GSCONT` (input, 512 bytes, keyed) | - Sets `STATUS` (`'Y'` or `'N'`), `KYDELT` (`'Y'` for delete), `EXISTS` (`'Y'` if batch file exists).<br>- Displays error messages (e.g., "INVALID COMPANY #").<br>- Calls `AR135TC`. |
| **AR135TC** | 5 | Checks if a temporary file exists for the given upload date. | - `FILENM` (constructed as `'GE' + KYUPDT`) | - Sets `STATUS` (`'Y'` or `'N'`) to indicate file existence.<br>- Returns error message if file does not exist. |
| **AR157.ocl36** | 6 | Copies non-deleted records from a temporary file to the batch file and cleans up. | - `?9?E?L'110,6'?` (source, e.g., `QS36F/CO123E202311`)<br>- `?9?CRIEGG` (target, e.g., `QS36F/CO123CRIEGG`) | - Creates/replaces `?9?CRIEGG` with non-deleted records.<br>- Deletes `?9?E?L'110,6'?`.<br>- Clears local variables.<br>- Calls `AR157A.ocl36`. |
| **AR157A.ocl36** | 7 | Loads and runs the field adjustment program for the temporary file. | - `CRTRAN` (label: `?9?E?L'110,6'?`, shared) | - Executes `AR157A.rpg36` to adjust fields in `CRTRAN`.<br>- No direct output files or side effects. |
| **AR157A.rpg36** | 8 | Adjusts fields in the temporary file for EFT compliance. | - `CRTRAN` (update, 256 bytes, e.g., `QS36F/CO123E202311`) | - Updates `CRTRAN` fields (e.g., `KYUPDT` at position 37).<br>- Ensures data is EFT-compliant for copying to `?9?CRIEGG`. |
