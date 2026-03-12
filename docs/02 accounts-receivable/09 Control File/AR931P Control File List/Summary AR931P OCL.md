### Use Cases Implemented by the Program

The call stack consists of the OCL script (`AR931P.ocl36.txt`), the RPGLE program (`AR931P.rpgle.txt`), and the RPG program (`AR931.rpg36.txt`). Together, they implement a single primary use case for generating an Accounts Receivable (A/R) Control File Listing report. Below is the identified use case:

1. **Generate Accounts Receivable Control File Listing Report**:
   - **Description**: This use case allows a user or system process to generate a formatted report listing details from the A/R control file (`ARCONT`) for selected companies, including general ledger numbers, journal numbers, finance charge percentages, fiscal month, and aging bucket limits. The process involves validating input parameters (company selection, job queue option, and number of copies) and producing a printed report with headers and detail lines.
   - **Components**:
     - `AR931P.ocl36.txt`: Orchestrates the job flow, calling setup programs and conditionally executing `AR931P` and `AR931`.
     - `AR931P.rpgle.txt`: Prompts for and validates user input (company selection, job queue, copies) via a display file, ensuring valid parameters are passed to `AR931`.
     - `AR931.rpg36.txt`: Reads `ARCONT` records and generates the report, applying filtering and formatting rules.
   - **Inputs**: Company selection (`ALL` or specific companies), company numbers (up to three), job queue flag (`Y` or `N`), number of copies.
   - **Outputs**: A printed report listing A/R control data for selected companies, with aging buckets based on invoice date (0-30, 31-60, 61-90, 91-120, over 120 days).

### Function Requirement Document



# Accounts Receivable Control File Listing Function Requirements

## Purpose
To generate a formatted Accounts Receivable (A/R) Control File Listing report based on specified company selection, job queue settings, and number of copies, using data from the A/R control file (`ARCONT`).

## Inputs
- **Company Selection**: `ALL` (all companies) or `CO` (specific companies).
- **Company Numbers**: Up to three company numbers (numeric, 2 digits each) if `CO` is selected.
- **Job Queue Flag**: `Y` (submit to job queue) or `N` (run interactively).
- **Number of Copies**: Integer (minimum 1) for report copies.
- **System Control Data**: Optional company number from `GSCONT` for default settings.

## Outputs
- **Printed Report**: Lists A/R control data for selected companies, including:
  - Company number and name.
  - A/R, cash, discount, intercompany, and finance G/L numbers.
  - Next A/R and sales journal numbers.
  - Finance charge percentage and security code.
  - First fiscal month.
  - Aging bucket upper limits (0-30, 31-60, 61-90, 91-120 days).

## Process Steps
1. **Validate Inputs**:
   - Verify company selection is `ALL` or `CO`.
   - If `CO`, ensure at least one valid company number exists in `ARCONT` and is not deleted (`ACDEL ≠ 'D'`).
   - If `ALL`, ensure no company numbers are provided.
   - Validate job queue flag is `Y` or `N`.
   - Ensure number of copies is at least 1 (default to 1 if 0).
   - Check system control file (`GSCONT`) for default company number; set `ALL` if none provided.

2. **Read A/R Control Data**:
   - Access `ARCONT` file (256 bytes/record).
   - Filter out records with `ACDEL = 'D'`.
   - Apply company selection:
     - If `ALL`, include all non-deleted records.
     - If `CO`, include only records matching specified company numbers.

3. **Generate Report**:
   - Print headers: report title, date, time, page number, and column labels (e.g., "CO#", "COMPANY NAME", "A/R G/L#").
   - For each selected `ARCONT` record, print:
     - `ACCO` (company number), `ACNAME` (company name).
     - `ACARGL`, `ACCSGL`, `ACDSGL`, `ACICGL`, `ACFNGL` (G/L numbers).
     - `ACARJ#`, `ACSLJ#` (journal numbers).
     - `ACFINC` (finance charge %), `ACSECR` (security code), `ACFFMO` (fiscal month).
     - `ACLMT1`, `ACLMT2`, `ACLMT3`, `ACLMT4` (aging bucket limits).
   - Use separator lines for readability.
   - Handle page overflow by reprinting headers.

4. **Job Execution**:
   - If job queue flag is `Y`, submit report generation to job queue.
   - If `N`, run interactively.
   - Produce specified number of report copies.

## Business Rules
- **Company Selection**:
  - Must be `ALL` or `CO`.
  - `CO` requires 1–3 valid company numbers from `ARCONT`.
  - `ALL` prohibits company number entries.
- **Aging Buckets**:
  - Based on invoice date (not due date), per revision (04/13/05):
    - 0-30 days (`ACLMT1`).
    - 31-60 days (`ACLMT2`).
    - 61-90 days (`ACLMT3`).
    - 91-120 days (`ACLMT4`).
    - Over 120 days (implicit).
- **Record Filtering**: Exclude deleted records (`ACDEL = 'D'`).
- **Job Queue**: Must be `Y` or `N`.
- **Copies**: Minimum 1; default to 1 if invalid.
- **Defaults**: Use `GSCONT` company number for `CO` selection if available; otherwise, default to `ALL`.

## Calculations
- **Aging Buckets**: No calculations in the function; bucket limits (`ACLMT1`–`ACLMT4`) are stored in `ARCONT` and printed as-is.
- **Page Numbering**: Increment `PAGE` for each new page on overflow.
- **Date/Time**: Capture system date and time for report header; format as MMDDYY and HHMMSS.

## Dependencies
- **Files**:
  - `ARCONT`: A/R control data (input).
  - `GSCONT`: System control data for default company (input).
  - `PRINT`: Printer file for report output.
- **System**: IBM System/36 or AS/400 environment with job queue support.

## Error Handling
- Return error if:
  - Company selection is not `ALL` or `CO`.
  - `CO` selected with invalid or missing company numbers.
  - `ALL` selected with company numbers provided.
  - Job queue flag is not `Y` or `N`.
- Cancel process if user or system signals cancellation (sets `KYCANC = 'CANCEL'`).

