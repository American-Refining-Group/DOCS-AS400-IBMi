The provided call stack consists of four files: `AP300P.ocl36.txt`, `AP300P.rpg36.txt`, `AP300.ocl36.txt`, and `AP300.rpg36.txt`. These files work together to implement an Accounts Payable (A/P) vendor file update process on the IBM System/36, specifically for clearing Month-to-Date (MTD) and Year-to-Date (YTD) totals in the vendor master file. Below, I’ll identify the use cases implemented by this program suite and then provide a function requirement document for a non-interactive version of the primary use case, focusing on business requirements and calculations.

---

### Use Cases Implemented

The program suite implements a single primary use case with variations based on input parameters. The use case is:

1. **Clear Vendor MTD and YTD Totals**:
   - **Description**: Updates the A/P vendor master file by clearing MTD totals (discounts, purchases, payments) for selected vendors and, optionally, YTD totals (purchases, discounts, payments) while preserving last year’s data. The process includes validation of input parameters, sorting of vendor records, backup of the vendor file for IRS 1099 processing, and maintenance of a 12-month file history.
   - **Variations**:
     - **Company Selection**: Process all non-deleted vendors (`ALL`) or specific companies (`CO` with up to three company numbers).
     - **YTD Clearing**: Clear YTD totals (`KYYTDY = 'Y'`) or only MTD totals (`KYYTDY = ' '`).
     - **Execution Mode**: Run immediately or submit to a job queue (`KYJOBQ = 'Y'`, `'N'`, or `' '`).
   - **Components**:
     - `AP300P.ocl36.txt`: Controls the overall job flow, invoking `AP300P.rpg36.txt` and conditionally `AP300.ocl36.txt`.
     - `AP300P.rpg36.txt`: Validates user inputs (company selection, YTD flag, year, job queue) via a screen interface.
     - `AP300.ocl36.txt`: Sorts the vendor file, creates backups, and invokes `AP300.rpg36.txt`.
     - `AP300.rpg36.txt`: Performs the actual MTD/YTD clearing in the vendor file.

No additional use cases are implemented, as the suite is focused solely on this update process with configurable options.

---

### Function Requirement Document

The following document reimagines the primary use case as a non-interactive function that accepts inputs programmatically (instead of via a screen) and completes the A/P vendor file update process. The document is concise, focusing on business requirements and necessary calculations, and assumes a modern programming context while retaining the core logic from the System/36 programs.

<xaiArtifact artifact_id="1c352277-781e-4b7a-8a39-1556ef094b97" artifact_version_id="20849124-46eb-4809-b271-00b8d943c8b8" title="APVendorUpdateFunction.md" contentType="text/markdown">

# Function Requirement Document: AP Vendor File Update

## Overview
The `APVendorUpdate` function updates the Accounts Payable (A/P) vendor master file by clearing Month-to-Date (MTD) and optionally Year-to-Date (YTD) totals for selected vendors, ensuring data integrity and compliance with IRS 1099 requirements. It accepts input parameters programmatically, validates them, sorts vendor records, creates backups, and performs the updates.

## Inputs
- **CompanySelection** (string): `"ALL"` for all non-deleted vendors or `"CO"` for specific companies.
- **CompanyNumbers** (array of strings, max 3): Company codes (2 bytes each) if `CompanySelection = "CO"`. Empty if `"ALL"`.
- **ClearYTD** (boolean): `true` to clear YTD totals, `false` to clear only MTD totals.
- **Year** (integer, 4 digits): Year for 1099 backup (required if `ClearYTD = true`, e.g., 2025).
- **JobQueue** (boolean): `true` to queue the update, `false` to run immediately.
- **LibraryPrefix** (string): Prefix for file names (e.g., `"PROD"` for `PRODAPVEND`).

## Outputs
- **Status** (string): `"Success"`, `"ValidationError"`, or `"SystemError"`.
- **ErrorMessage** (string): Description of validation or system errors, if any.
- **UpdatedRecords** (integer): Number of vendor records updated.

## Process Steps
1. **Validate Inputs**:
   - Ensure `CompanySelection` is `"ALL"` or `"CO"`. Return `"ValidationError"` with message "Company selection must be 'ALL' or 'CO'" if invalid.
   - If `CompanySelection = "CO"`, verify `CompanyNumbers` contains 1–3 valid 2-byte company codes in the vendor file. Return `"ValidationError"` with message "Invalid company number" if any code is invalid or all are empty.
   - If `CompanySelection = "ALL"`, ensure `CompanyNumbers` is empty. Return `"ValidationError"` with message "Do not specify companies when selecting 'ALL'" if not empty.
   - If `ClearYTD = true`, ensure `Year` is a valid 4-digit year (> 0). Return `"ValidationError"` with message "A 4-digit year is required when clearing YTD" if invalid.
   - Ensure `JobQueue` is `true` or `false`. Return `"ValidationError"` with message "Job queue must be 'Y' or 'N'" if invalid.

2. **Check Exclusive Access**:
   - Verify no users are accessing the A/P system. Return `"SystemError"` with message "A/P system in use; ensure exclusive access" if locked.

3. **Sort Vendor Records**:
   - Filter the vendor master file (`<LibraryPrefix>APVEND`) to include only non-deleted records (`DeletionFlag ≠ 'D'`).
   - If `CompanySelection = "CO"`, include only records matching `CompanyNumbers`.
   - Sort by company (2 bytes) and vendor number (5 bytes) into a temporary file (`<LibraryPrefix>AP300S`).

4. **Create Backups for 1099** (if `ClearYTD = true`):
   - Copy `<LibraryPrefix>APVEND` to:
     - `APVN<Year>` (e.g., `APVN2025`).
     - `<LibraryPrefix>VN<Year>` (e.g., `PRODVN2025`).
   - Overwrite existing files and create if they don’t exist.

5. **Maintain 12-Month File History**:
   - Delete the oldest backup file (`<LibraryPrefix>UCVEND`) if it exists.
   - Rename files to shift history:
     - `<LibraryPrefix>UBVEND` to `<LibraryPrefix>UCVEND`.
     - `<LibraryPrefix>UAVEND` to `<LibraryPrefix>UBVEND`.
     - ... down to `<LibraryPrefix>U1VEND` to `<LibraryPrefix>U2VEND`.
   - Copy `<LibraryPrefix>APVEND` to `<LibraryPrefix>U1VEND`, overwriting if exists.

6. **Update Vendor Records**:
   - For each record in `<LibraryPrefix>AP300S`, update the corresponding record in `<LibraryPrefix>APVEND`:
     - **MTD Updates** (always):
       - Set `MTDDiscounts` to 0.
       - Set `MTDPurchases` to 0.
       - Set `MTDPayments` to 0.
       - Set `PreviousBalance` to `CurrentBalance`.
     - **YTD Updates** (if `ClearYTD = true`):
       - Move `YTDPurchases` to `LastYearPurchases`.
       - Move `YTDPaid` to `LastYearPaid`.
       - Set `YTDPurchases` to 0.
       - Set `YTDDiscounts` to 0.
       - Set `YTDPaid` to 0.
   - Track the number of updated records.

7. **Execute or Queue**:
   - If `JobQueue = true`, submit the update process to a job queue.
   - If `JobQueue = false`, execute immediately.
   - Return `"Success"` with the count of updated records or `"SystemError"` with an error message if execution fails.

## Business Rules
1. **Exclusive Access**: The A/P system must not be in use during the update to prevent data corruption.
2. **Company Selection**:
   - `"ALL"` processes all non-deleted vendors.
   - `"CO"` processes only specified, valid company codes (1–3).
   - Invalid or conflicting company selections are rejected.
3. **MTD Clearing**: Always clear MTD discounts, purchases, and payments, preserving the current balance as the previous balance.
4. **YTD Clearing**:
   - Clear YTD purchases, discounts, and payments only if `ClearYTD = true`.
   - Preserve YTD data in last-year fields (`LastYearPurchases`, `LastYearPaid`) for auditing.
   - Require a valid 4-digit year for 1099 backups.
5. **Backups for 1099**: Save two copies of the vendor file (`APVN<Year>`, `<LibraryPrefix>VN<Year>`) before clearing YTD totals.
6. **File History**: Maintain a rolling 12-month history of vendor file backups (`<LibraryPrefix>U1VEND` to `<LibraryPrefix>UCVEND`).
7. **Data Integrity**:
   - Process only non-deleted records.
   - Preserve the current balance (`CurrentBalance`) during updates.
   - Ensure backups are created before modifying the vendor file.

## Calculations
- **MTD Updates**:
  - `MTDDiscounts = 0`
  - `MTDPurchases = 0`
  - `MTDPayments = 0`
  - `PreviousBalance = CurrentBalance`
- **YTD Updates** (if `ClearYTD = true`):
  - `LastYearPurchases = YTDPurchases`
  - `LastYearPaid = YTDPaid`
  - `YTDPurchases = 0`
  - `YTDDiscounts = 0`
  - `YTDPaid = 0`

## Assumptions
- The vendor master file (`<LibraryPrefix>APVEND`) contains fields for MTD/YTD totals, deletion flag, company, and vendor number.
- The system supports file operations (copy, rename, delete) and job queuing.
- Input validation ensures only valid data is processed, eliminating the need for interactive error correction.

</xaiArtifact>

---

### Explanation

- **Use Case Identification**: The suite implements one core use case—clearing MTD/YTD totals—with variations for company selection, YTD clearing, and execution mode. All components (`AP300P.ocl36.txt`, `AP300P.rpg36.txt`, `AP300.ocl36.txt`, `AP300.rpg36.txt`) contribute to this single process, with no additional use cases identified.
- **Function Requirement Document**:
  - **Non-Interactive Design**: The document reimagines the process as a function that accepts inputs directly, bypassing the screen interface (`AP300PS1` from `AP300P.rpg36.txt`).
  - **Business Focus**: Emphasizes business rules (e.g., exclusive access, company filtering, 1099 backups) and calculations derived from `AP300.rpg36.txt` (e.g., MTD/YTD field updates).
  - **Integration**: Incorporates logic from all four files:
    - Validation from `AP300P.rpg36.txt` (company selection, YTD flag, year).
    - Sorting and backups from `AP300.ocl36.txt` (`#GSORT`, `CPYF`).
    - Updates from `AP300.rpg36.txt` (MTD/YTD clearing).
    - Job control from `AP300P.ocl36.txt` (queue or immediate execution).
  - **Conciseness**: Streamlines the process into clear steps and rules, avoiding technical System/36 specifics (e.g., OCL syntax, RPG indicators) while preserving functionality.

This document provides a modernized, non-interactive specification for the A/P update process, suitable for implementation in a contemporary system while retaining the original business logic. If you need further refinements or additional details, let me know!