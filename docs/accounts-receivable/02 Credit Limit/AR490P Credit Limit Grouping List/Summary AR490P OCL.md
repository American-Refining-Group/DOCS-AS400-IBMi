### List of Use Cases Implemented by the Program

Based on the provided call stack (`AR490P.ocl36.txt`, `AR490P.rpgle.txt`, `AR490.ocl36.txt`, `AR490.rpg36.txt`), the program implements a single primary use case:

1. **Generate Credit Limit Grouping List**:
   - This use case allows users to generate a printed report listing customers grouped by company with their associated credit limit group numbers. The user can select either all companies or specific companies, choose whether to run the report interactively or in a job queue, and specify the number of report copies. The report includes company names, customer numbers, customer names, and up to 15 credit limit group numbers per customer.

---

### Function Requirement Document



# Credit Limit Grouping List Function Requirements

## Overview
The `GenerateCreditLimitGroupingList` function generates a report listing customers by company, including customer names and up to 15 credit limit group numbers per customer. It accepts inputs for company selection, job queue preference, and number of copies, processes data from company and customer files, and produces a sorted, formatted report.

## Inputs
- **Company Selection** (`companySelection`): String, either `'ALL'` (all companies) or `'CO'` (specific companies).
- **Company Numbers** (`companyNumbers`): Array of up to 3 numeric company numbers (2 digits each). Required if `companySelection = 'CO'`, otherwise empty.
- **Job Queue** (`jobQueue`): String, `'Y'` (run in job queue) or `'N'` (run interactively).
- **Copies** (`copies`): Numeric, number of report copies (default 1 if 0).

## Outputs
- **Report**: A formatted text file or printer output containing:
  - Header: Company name, report title ("CREDIT LIMIT GROUPING LIST"), date, time, page number.
  - Detail: Customer number, customer name, up to 15 credit limit group numbers per customer.
- **Status**: Success or error message (e.g., "Invalid company number").

## Process Steps
1. **Validate Inputs**:
   - Ensure `companySelection` is `'ALL'` or `'CO'`. If invalid, return error: "Company selection must be 'CO' or 'ALL'".
   - If `companySelection = 'CO'`, validate `companyNumbers`:
     - At least one company number must be non-zero.
     - Each company number must exist in `ARCONT` and not be marked deleted (`ACDEL ≠ 'D'`). If invalid, return error: "Invalid company number".
   - If `companySelection = 'ALL'`, ensure `companyNumbers` is empty. If not, return error: "If ALL, do not enter companies".
   - Ensure `jobQueue` is `'Y'` or `'N'`. If invalid, return error: "Job queue entry must be 'Y' or 'N'".
   - If `copies = 0`, set to 1.

2. **Retrieve Default Company**:
   - Query `GSCONT` for a default company number (`GXCONO`).
   - If non-zero, set `companySelection = 'CO'` and include `GXCONO` in `companyNumbers`. Otherwise, use `companySelection = 'ALL'`.

3. **Sort Input Data**:
   - Read `ARCLGR` file (credit limit grouping data).
   - Filter records where `CGDEL ≠ 'D'` (non-deleted).
   - If `companySelection = 'CO'`, include only records where `CGCONO` matches a value in `companyNumbers`.
   - Sort by `CGCONO` (company number, positions 2-3) and `CGCUST` (customer number, positions 4-9).
   - Output sorted data to temporary file `AR490`.

4. **Generate Report**:
   - Read sorted `AR490` file.
   - For each record:
     - Retrieve company name (`ACNAME`) from `ARCONT` using `CGCONO`. If not found, use blank.
     - Retrieve customer name (`ARNAME`) from `ARCUST` using `ARKEY` (company + customer number). If not found, use blank.
     - Extract credit limit group numbers (`CGCL01` to `CGCL15`, positions 10-99).
     - Write to report:
       - Header (per company): `ACNAME`, "CREDIT LIMIT GROUPING LIST", date, time, page number, separator lines.
       - Detail (per customer): `CGCUST` (zero-suppressed), `ARNAME`, `CGCL01` to `CGCL15` (zero-suppressed, 9-character spacing).
   - Repeat for `copies` iterations.

5. **Execution Mode**:
   - If `jobQueue = 'Y'`, submit report generation to job queue.
   - If `jobQueue = 'N'`, run interactively.

## Business Rules
- **Company Selection**:
  - `'ALL'` includes all non-deleted companies; `'CO'` requires 1-3 valid company numbers.
  - Company numbers must exist in `ARCONT` and be active (`ACDEL ≠ 'D'`).
- **Customer Data**:
  - Only non-deleted customers (`CGDEL ≠ 'D'`) from `ARCLGR` are included.
  - Customer names are retrieved from `ARCUST`; missing names are blank.
- **Report Content**:
  - Lists up to 15 credit limit group numbers per customer (out of 25 possible in `ARCLGR`).
  - Groups customers by company, with headers per company.
- **Copies**:
  - Minimum 1 copy; 0 is corrected to 1.
- **Error Handling**:
  - Invalid inputs halt processing with specific error messages.
- **Date Handling**:
  - Uses system date/time, formatted as MMDDYY and HHMMSS, with Y2K compliance.

## Calculations
- **Sorting**: Performed on `CGCONO` (numeric, 2 digits) and `CGCUST` (numeric, 6 digits).
- **Page Numbering**: Increments per page, zero-suppressed.
- **Field Formatting**:
  - `CGCUST`, `CGCL01` to `CGCL15`: Zero-suppressed numeric output.
  - `ARNAME`, `ACNAME`: Left-justified text.
- **Separator Lines**: Generated as repeating `'* '` for 164 characters.

## Dependencies
- **Files**:
  - `ARCLGR`: Input data (company, customer, credit limit groups).
  - `ARCONT`: Company data (company number, name).
  - `ARCUST`: Customer data (company number, customer number, name).
  - `GSCONT`: Default company configuration.
  - `GSTABL`: Configuration table (purpose unspecified, likely report formatting).
- **Utilities**:
  - Year 2000 date handling (GSY2K).
  - Sort utility (#GSORT).

## Execution Context
- Runs on IBM midrange system (System/36 or AS/400).
- Supports batch (`jobQueue = 'Y'`) or interactive (`jobQueue = 'N'`) execution.

