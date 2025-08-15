The call stack consists of two OCL programs (`BI921P.ocl36.txt` and `BI921.ocl36.txt`) and two RPG programs (`BI921P.rpg36.txt` and `BI921.rpg36.txt`) that together implement the workflow for generating a "Product Tax Master List" on an IBM System/36 or AS/400 system. Below, I’ll identify the use cases implemented by this call stack and then provide a concise function requirement document for one of the use cases, reimagined as a large function that processes inputs directly rather than relying on interactive screen input.

### Use Cases Implemented by the Call Stack

The call stack implements the following use cases for generating and managing the Product Tax Master List:

1. **Generate Product Tax Master List for All Companies**:
   - **Description**: Produces a report listing tax details (state, product code, delivered code, and up to 10 tax codes) for all companies in the `BICONT` file, sorted by company number, state, product code, and delivered code.
   - **Components Involved**:
     - `BI921P.ocl36.txt`: Validates user input, allowing `KYALCO='ALL'` to select all companies and sets `KYJOBQ` to determine if the job runs interactively or in a job queue.
     - `BI921P.rpg36.txt`: Prompts for and validates input (`KYALCO='ALL'`, no company numbers entered, `KYJOBQ='Y' or 'N'`, `KYCOPY≥1`), retrieves company names from `BICONT` for display, and sets parameters for the OCL.
     - `BI921.ocl36.txt`: Sets a flag (`IAC` or `I*C`) based on `KYALCO`, sorts the `BIPRTX` file into `BI921` using `#GSORT` (no company filtering for 'ALL'), and runs `BI921` to generate the report.
     - `BI921.rpg36.txt`: Reads sorted `BIPRTX` data, retrieves company names from `BICONT` and product descriptions from `GSPROD`, and prints the report with headers (company, date, time, page) and details (state, product code, description, delivered code, tax codes).
   - **Inputs**: Selection of all companies (`KYALCO='ALL'`), job queue flag (`KYJOBQ`), number of copies (`KYCOPY`).
   - **Output**: A printed report listing tax details for all companies.

2. **Generate Product Tax Master List for Specific Companies**:
   - **Description**: Produces a report for up to three user-specified company numbers, validated against `BICONT`, sorted by company number, state, product code, and delivered code.
   - **Components Involved**:
     - `BI921P.ocl36.txt`: Validates input, ensuring `KYALCO='CO'` with valid company numbers (`KYCO1`, `KYCO2`, `KYCO3`), and sets job queue and copy parameters.
     - `BI921P.rpg36.txt`: Validates company numbers against `BICONT`, ensures at least one valid company is entered, and handles error messages for invalid inputs.
     - `BI921.ocl36.txt`: Filters `BIPRTX` records by specified company numbers (`KYCO1`, `KYCO2`, `KYCO3`) during the `#GSORT` operation and runs `BI921` for the report.
     - `BI921.rpg36.txt`: Processes filtered `BIPRTX` data, retrieves company names and product descriptions, and generates the report for the selected companies.
   - **Inputs**: Specific company numbers (`KYCO1`, `KYCO2`, `KYCO3`), `KYALCO='CO'`, job queue flag (`KYJOBQ`), number of copies (`KYCOPY`).
   - **Output**: A printed report listing tax details for the specified companies.

3. **Validate and Prepare Input Parameters for Report Generation**:
   - **Description**: Collects and validates user inputs (company selection, job queue option, number of copies) to ensure correct parameters are passed to the report generation process.
   - **Components Involved**:
     - `BI921P.ocl36.txt`: Checks for cancellation conditions and queues or runs `BI921` based on `KYJOBQ`.
     - `BI921P.rpg36.txt`: Prompts for and validates `KYALCO` ('ALL' or 'CO'), company numbers (valid in `BICONT` if 'CO'), `KYJOBQ` ('Y' or 'N'), and `KYCOPY` (≥1), displaying error messages for invalid inputs.
   - **Inputs**: `KYALCO`, `KYCO1`, `KYCO2`, `KYCO3`, `KYJOBQ`, `KYCOPY`.
   - **Output**: Validated parameters passed to the OCL for further processing.

4. **Sort Tax Data for Reporting**:
   - **Description**: Sorts the product tax data by company number, state, product code, and delivered code, filtering by specific companies if required.
   - **Components Involved**:
     - `BI921.ocl36.txt`: Uses `#GSORT` to sort `BIPRTX` into `BI921`, applying filters for company numbers if `KYALCO='CO'`.
   - **Inputs**: `BIPRTX` file, company numbers (`KYCO1`, `KYCO2`, `KYCO3`) if filtering.
   - **Output**: Sorted `BI921` file for use by the report program.

### Function Requirement Document

Below is a function requirement document for the primary use case: **Generate Product Tax Master List (covering both "All Companies" and "Specific Companies")**, reimagined as a single function that takes inputs directly rather than using interactive screen prompts. This consolidates the core functionality of the call stack, focusing on business requirements and necessary calculations.

<xaiArtifact artifact_id="c1137707-6e49-4bcb-a02e-06b2eb7ec447" artifact_version_id="2c8acbf4-a1e1-4c16-bd00-c88df042c1d4" title="ProductTaxMasterListFunction.md" contentType="text/markdown">

# Function Requirement Document: Generate Product Tax Master List

## Function Name
`GenerateProductTaxMasterList`

## Purpose
Generate a formatted Product Tax Master List report, listing tax details (state, product code, description, delivered code, and up to 10 tax codes) for all companies or specific user-selected companies, sorted by company number, state, product code, and delivered code.

## Inputs
- **companySelection** (String): Either 'ALL' (include all companies) or 'CO' (specific companies).
- **companyNumbers** (Array of Strings, max 3): Company numbers (2 digits each, e.g., '01', '02', '03') when `companySelection='CO'`. Empty or null if 'ALL'.
- **jobQueue** (String): 'Y' (run in job queue) or 'N' (run interactively).
- **numCopies** (Integer): Number of report copies (minimum 1).
- **taxDataFile** (File): Input file containing tax data (equivalent to `BIPRTX`).
- **companyDataFile** (File): File containing company data (equivalent to `BICONT`).
- **productDataFile** (File): File containing product data (equivalent to `GSPROD`).

## Outputs
- **report** (File): A formatted report file (equivalent to `PRINT`) containing the Product Tax Master List, grouped by company with headers and detail lines.
- **errorMessage** (String, optional): Error message if validation fails (e.g., invalid company number).

## Process Steps
1. **Validate Inputs**:
   - Ensure `companySelection` is 'ALL' or 'CO'. If invalid, return error: "Company selection must be 'ALL' or 'CO'."
   - If `companySelection='CO'`, validate that at least one `companyNumbers` entry is non-zero and exists in `companyDataFile`. If invalid, return error: "Invalid company number" or "At least one valid company must be entered."
   - If `companySelection='ALL'`, ensure all `companyNumbers` are empty or zero. If not, return error: "No company numbers allowed when selecting 'ALL'."
   - Ensure `jobQueue` is 'Y' or 'N'. If invalid, return error: "Job queue must be 'Y' or 'N'."
   - If `numCopies < 1`, set to 1.

2. **Sort Tax Data**:
   - Read `taxDataFile` (64-byte records: delete code, company number, state, product code, delivered code, 10 tax codes).
   - Filter out records where delete code is 'D'.
   - If `companySelection='CO'`, filter records where company number matches any in `companyNumbers`.
   - Sort records by:
     - Company number (positions 2-3).
     - State (positions 4-5).
     - Product code (positions 6-9).
     - Delivered code (position 10).
   - Store sorted data in a temporary file.

3. **Generate Report**:
   - For each unique company number in the sorted data:
     - Retrieve company name from `companyDataFile` using company number. If not found, use blank.
     - Print header: company name, date (MMDDYY), time (HHMMSS), page number, and title "PRODUCT TAX MASTER LIST."
   - For each record:
     - Retrieve product description from `productDataFile` using company number and product code. If not found, use blank; if product code is 'MISC', use "MISCELLANEOUS INVOICE LINES."
     - Print detail line: state, product code, product description (first 20 characters), delivered code, tax codes 1-10.
   - Repeat for `numCopies` copies.
   - Output to a printer-compatible file.

4. **Execution Mode**:
   - If `jobQueue='Y'`, queue the report generation for asynchronous execution.
   - If `jobQueue='N'`, execute immediately.

## Business Rules
1. **Company Selection**:
   - 'ALL' includes all non-deleted companies in `companyDataFile`.
   - 'CO' requires 1-3 valid company numbers (2 digits, existing in `companyDataFile`).
   - If 'CO', at least one company number must be valid and non-zero.
   - If 'ALL', no company numbers may be specified.

2. **Data Filtering**:
   - Exclude tax records where delete code is 'D'.
   - For 'CO', include only records matching specified company numbers.

3. **Product Description**:
   - Use `productDataFile` for descriptions. If not found, use blank.
   - For product code 'MISC', use "MISCELLANEOUS INVOICE LINES" if no description is found.

4. **Tax Codes**:
   - Support up to 10 tax codes per product (4 bytes each).
   - Print all tax codes in the report, even if blank.

5. **Report Formatting**:
   - Group by company number, with headers on company change or page overflow.
   - Include date (MMDDYY), time (HHMMSS), and incremental page number.
   - Detail lines include state (2 chars), product code (4 chars), description (20 chars), delivered code (1 char), and tax codes (4 chars each).

6. **Copies**:
   - Minimum 1 copy; default to 1 if input is 0 or negative.

## Calculations
- **Date and Time**: Capture system date (MMDDYY) and time (HHMMSS) for report headers.
- **Page Number**: Increment per company change or page overflow (132 characters per line, assuming 66 lines per page).
- **Sorting**: Multi-level sort on company number (2 bytes), state (2 bytes), product code (4 bytes), delivered code (1 byte).
- **Key Construction**: Combine company number and product code for `productDataFile` lookup (6-byte key).

## Error Handling
- Return specific error messages for validation failures (e.g., invalid `companySelection`, `companyNumbers`, or `jobQueue`).
- Log errors for missing company or product data but continue processing with blanks.

## Assumptions
- Input files (`taxDataFile`, `companyDataFile`, `productDataFile`) are pre-populated and accessible.
- Output is a printer-compatible file (132 bytes per line).
- System supports queuing for `jobQueue='Y'`.

</xaiArtifact>

### Notes
- **Use Case Selection**: The function requirement document focuses on the primary use case (Generate Product Tax Master List) as it encompasses both "All Companies" and "Specific Companies," combining the core functionality of the call stack. The other use cases (validation and sorting) are subsumed as steps within this function.
- **Non-Interactive Design**: The function assumes direct input parameters, replacing the interactive screen input from `BI921P.rpg36.txt`. This aligns with the requirement to treat the use case as a large function.
- **Commented Files**: The `GSTABL` file (commented out in `BI921.rpg36.txt` and `BI921.ocl36.txt`) is excluded, assuming `GSPROD` is the active product file per modification `JK01`.
- **Error Messages**: The function retains the error message logic from `BI921P.rpg36.txt` but applies it to input validation.

If you need a function requirement document for the other use cases (e.g., validation or sorting as standalone functions) or further details, let me know! I can also refine the document or provide additional artifacts (e.g., pseudocode or sample report format).