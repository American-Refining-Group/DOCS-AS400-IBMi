### List of Use Cases Implemented by the Program

The call stack consists of three components: `AP760P.ocl36.txt` (OCL control program), `AP760P.rpg36.txt` (RPG/36 screen validation program), and `AP760.rpg36.txt` (RPG/36 report generation program). Together, they implement a single primary use case for generating a Vendor 1099 Register. Below is the identified use case:

1. **Generate Vendor 1099 Register**:
   - **Description**: The system allows users to generate a printed report summarizing vendor payment data for IRS Form 1099, filtered by company, 1099 type, year (current or last), and job queue option. The report includes vendor details (number, 1099 ID, payee names) and payment amounts categorized by 1099 boxes (rent, medical, non-employee compensation/miscellaneous).
   - **Inputs**: Library/environment identifier, four-digit year, company selection (all or specific), 1099 type selection (all or specific), current/last year, job queue option (Y/N), and number of report copies.
   - **Outputs**: A printed report (`APPRINT`) with headers, vendor details, and totals grouped by 1099 type.
   - **Components Involved**:
     - `AP760P.ocl36.txt`: Orchestrates the process, validates the input file (`APVNYYYY`), and calls `AP760P` and `AP760`.
     - `AP760P.rpg36.txt`: Validates user inputs via a screen (`AP760PS1`) and sets parameters in the User Data Structure (UDS).
     - `AP760.rpg36.txt`: Processes vendor data from `AP761` and generates the report.

No additional distinct use cases are implemented, as the programs focus solely on the 1099 Register generation process with variations based on input parameters.

---

### Function Requirement Document: Generate Vendor 1099 Register



# Function Requirement: Generate Vendor 1099 Register

## Overview
This function generates a Vendor 1099 Register report summarizing vendor payment data for IRS Form 1099, filtered by company, 1099 type, year, and processing options. It processes input data, validates parameters, and produces a formatted report with vendor details and totals.

## Inputs
- **Library Identifier**: String identifying the environment/library (e.g., `LIB`).
- **Year**: Four-digit year for the 1099 data (e.g., `2025`).
- **Company Selection**: 
  - `'ALL'` for all companies or `'CO'` with up to three valid company numbers.
- **1099 Type Selection**: 
  - `'ALL'` for all 1099 types or `'TYP'` with up to three valid 1099 type codes.
- **Year Option**: `'C'` (current year) or `'L'` (last year, i.e., year - 1).
- **Job Queue Option**: `'Y'` (submit to job queue) or `'N'` (run interactively).
- **Number of Copies**: Integer (default 1 if 0).

## Outputs
- **Printed Report**: A 132-character-wide report (`APPRINT`) containing:
  - Header: Report title, date, time, page number.
  - Details: For each 1099 type, lists vendor number, 1099 ID, payee name(s), and amounts for boxes 1 (rent), 6 (medical), 3 (non-employee compensation/miscellaneous).
  - Totals: Vendor count and box amount totals per 1099 type.

## Process Steps
1. **Validate Input File**:
   - Construct file name (`APVNYYYY` or `[Library]VNYYYY`) using library and year.
   - Verify existence of the file (`APVNYYYY`, created by `AP300` period-end process).
   - If file is missing, return error: “Year not found ([FileName])”.

2. **Validate Parameters**:
   - **Company Selection**:
     - If `'ALL'`, ensure no specific company numbers are provided.
     - If `'CO'`, validate up to three company numbers against `APCONT` file.
     - Return error for invalid company selections: “ENTRY MUST BE 'CO' OR 'ALL'”, “INVALID COMPANY NUMBER”, or “IF ALL, DO NOT ENTER COMPANIES”.
   - **1099 Type Selection**:
     - If `'ALL'`, ensure no specific types are provided.
     - If `'TYP'`, validate up to three 1099 types against `GSTABL` (key `AP1099`).
     - Return error for invalid type selections: “ENTRY MUST BE 'TYP' OR 'ALL'”, “INVALID 1099 TYPE”, or “IF ALL, DO NOT ENTER TYPES”.
   - **Year Option**:
     - Validate `'C'` or `'L'`. Set report year to input year (`'C'`) or year - 1 (`'L'`).
     - Return error for invalid option: “CURR/LAST ENTRY MUST BE 'C' OR 'L'”.
   - **Job Queue Option**:
     - Validate `'Y'` or `'N'`. Return error for invalid option: “JOB QUEUE ENTRY MUST BE 'Y' OR 'N'”.
   - **Number of Copies**:
     - If 0, set to 1.

3. **Process Vendor Data**:
   - Read vendor records from `AP761` (derived from `APVNYYYY`).
   - Skip records with delete code (`VNDEL` set).
   - Group records by 1099 type (`VN1099`).
   - For each record:
     - Retrieve 1099 type description from `GSTABL` using key `AP1099` + `VN1099`.
     - Map payment amounts (`VNAMT1`, `VNAMT2`) to boxes based on `VNBOX1` and `VNBOX2`:
       - Box 1: Rent.
       - Box 6: Medical and health care payments.
       - Box 3: Non-employee compensation (also used for box 7, miscellaneous).
     - Handle payee names:
       - If `VNPYN1` is blank, use `VNNAME`.
       - If `VNNOVF = 'Y'`, use `VNADD1` as second payee name.
       - If `VNPYN1` and `VNPYN2` are provided, use them.
     - Accumulate amounts per box and vendor count per 1099 type.

4. **Generate Report**:
   - Output to `APPRINT` (132 characters):
     - **Header**: Print “VENDOR 1099 REGISTER”, date (MMDDYY), time (HH.MM.SS), page number.
     - **Group Header**: For each 1099 type, print type code, description, and column headers (“NUMBER”, “1099 ID #”, “PAYEE NAME”, “RENT (BOX=1)”, “MEDICAL (BOX=6)”, “MISC (BOX=3)”).
     - **Detail Lines**: For each vendor, print vendor number, 1099 ID, payee name(s), and box amounts.
     - **Second Line**: Print second payee name if applicable.
     - **Totals**: Print vendor count and box amount totals per 1099 type.
   - If job queue is `'Y'`, submit report generation to queue; otherwise, run interactively.

## Business Rules
1. **Input Validation**:
   - Company selection must be `'ALL'` or `'CO'` with valid company numbers in `APCONT`.
   - 1099 type selection must be `'ALL'` or `'TYP'` with valid types in `GSTABL`.
   - Year option must be `'C'` or `'L'`, mapping to input year or year - 1.
   - Job queue option must be `'Y'` or `'N'`.
   - Number of copies must be non-zero (default to 1).

2. **Data Processing**:
   - Process only non-deleted vendor records (`VNDEL` not set).
   - Group records by 1099 type (`VN1099`) for reporting.
   - Map payment amounts to boxes 1, 3, 6, or 7 based on `VNBOX1` and `VNBOX2`.
   - Use payee names (`VNPYN1`, `VNPYN2`) if provided; otherwise, use vendor name (`VNNAME`). Use address (`VNADD1`) as second line if `VNNOVF = 'Y'`.

3. **Calculations**:
   - Accumulate amounts for each box per vendor:
     - `BX1AMT` += `VNAMT1` or `VNAMT2` if `VNBOX1` or `VNBOX2` = 1.
     - `BX3AMT` += `VNAMT1` or `VNAMT2` if `VNBOX1` or `VNBOX2` = 3 or 7.
     - `BX6AMT` += `VNAMT1` or `VNAMT2` if `VNBOX1` or `VNBOX2` = 6.
   - Sum box amounts and vendor count per 1099 type (`L21AMT`, `L23AMT`, `L26AMT`, `L2CNT`).

4. **Report Formatting**:
   - Print headers, group details, and totals in a 132-character format.
   - Include second payee name line only if `VNPYN2` or `VNNOVF = 'Y'`.
   - Adjust formatting based on 1099 type (`KYTY1` = `'M'` or `'N'` for miscellaneous or non-employee compensation).

## Dependencies
- **Files**:
  - `APVNYYYY`: Input vendor data file (from `AP300`).
  - `APCONT`: Accounts payable control file (company numbers and names).
  - `GSTABL`: General system table (1099 type descriptions).
  - `AP761`: Processed vendor 1099 data.
  - `APPRINT`: Printer output file.
- **Programs**: None called directly; invoked by `AP760P.ocl36.txt`.

## Error Handling
- Return descriptive errors for invalid inputs (company, type, year, job queue).
- Terminate if `APVNYYYY` file is missing.
- Ensure report includes only valid, non-deleted vendor records.

