### List of Use Cases Implemented by the Program

The call stack, consisting of `AP760P.ocl36.txt`, `AP760P.rpg36.txt`, `AP760.ocl36.txt`, `AP761.rpg36.txt`, and `AP760.rpg36.txt`, implements a single primary use case:

1. **Generate Vendor 1099 Register Report**:
   - This use case involves collecting user inputs for filtering vendor 1099 data, preprocessing the data to consolidate vendor records, and producing a formatted report summarizing payments by 1099 type and box numbers (e.g., rent, medical, miscellaneous) for tax reporting purposes. The process supports filtering by company, 1099 type, current or last year, and job queue execution, ensuring accurate aggregation and formatting for IRS compliance.

---

### Function Requirement Document: Generate Vendor 1099 Register Report



# Function Requirement Document: Generate Vendor 1099 Register Report

## Purpose
Generate a consolidated 1099 register report summarizing vendor payments by 1099 type and box numbers (rent, medical, miscellaneous) for tax reporting, filtered by user-specified criteria.

## Inputs
- **Year**: Four-digit year (e.g., 2025) for the 1099 data file.
- **Company Selection**:
  - `'ALL'`: Include all companies.
  - `'CO'`: List of up to three valid company numbers.
- **1099 Type Selection**:
  - `'ALL'`: Include all 1099 types.
  - `'TYP'`: List of up to three valid 1099 types.
- **Year Scope**: `'C'` (current year) or `'L'` (last year).
- **Job Queue**: `'Y'` (submit to job queue) or `'N'` (run interactively).
- **Copies**: Number of report copies (minimum 1).
- **1099 Data File**: File name (e.g., `APVN2025`) containing vendor payment data.
- **Control File**: Company data file (e.g., `APCONT`) for validation.
- **Table File**: Lookup table (e.g., `GSTABL`) for 1099 type descriptions.

## Outputs
- **1099 Register Report**: Printed report listing:
  - Vendor number, 1099 ID, payee or vendor name (with overflow support).
  - Payment amounts by 1099 box (1=rent, 3/7=miscellaneous, 6=medical).
  - Totals by 1099 type, including vendor count and box amounts.
- **Temporary Files**: Intermediate sorted and processed files (e.g., `AP760S`, `AP761`), deleted after processing.

## Process Steps
1. **Validate Inputs**:
   - Ensure year is a valid four-digit value and 1099 data file (e.g., `APVN2025`) exists.
   - Validate company numbers against control file if `'CO'` selected; reject if invalid or none provided.
   - Validate 1099 types against table file if `'TYP'` selected; reject if invalid or none provided.
   - Ensure year scope is `'C'` or `'L'`, job queue is `'Y'` or `'N'`, and copies is at least 1.

2. **Sort 1099 Data**:
   - Filter 1099 data file by:
     - Excluding deleted records (delete code ≠ `'D'`).
     - Matching specified companies (if `'CO'`) or all companies (if `'ALL'`).
     - Matching specified 1099 types (if `'TYP'`) or all types (if `'ALL'`).
   - Sort by 1099 type, vendor number, and company number, producing a temporary sorted file.

3. **Preprocess Data**:
   - Consolidate records by vendor, creating one record per vendor:
     - Aggregate current year YTD paid amount (or last year if `'L'` selected).
     - If a second 1099 box amount exists, subtract it from the primary amount.
     - Set first box number to 7 (miscellaneous) if unspecified.
     - Include vendor number, 1099 ID, name, payee names, name overflow, address, and box numbers/amounts.
   - Write to an intermediate work file.

4. **Generate Report**:
   - Group records by 1099 type, retrieving descriptions from table file.
   - For each vendor:
     - Print vendor number, 1099 ID, and name (payee names if provided, else vendor name; use address for overflow if flagged).
     - Print amounts for boxes 1 (rent), 3/7 (miscellaneous), and 6 (medical).
   - At each 1099 type break:
     - Print vendor count and total amounts for boxes 1, 3/7, and 6.
   - Include report headers with date, time, page number, and column labels.

5. **Clean Up**:
   - Delete temporary files after report generation.

## Business Rules
1. **Data Filtering**:
   - Include only non-deleted vendor records.
   - If `'CO'` selected, require at least one valid company; if `'ALL'`, exclude specific companies.
   - If `'TYP'` selected, require at least one valid 1099 type; if `'ALL'`, exclude specific types.

2. **Amount Calculation**:
   - Use current year YTD paid for `'C'`; last year YTD paid for `'L'` (pending future year-specific file support).
   - If a second 1099 box amount exists, subtract it from the primary amount to split total payment.

3. **Box Mapping**:
   - Box 1: Rent payments.
   - Box 3 or 7: Miscellaneous payments (7 defaults if first box unspecified).
   - Box 6: Medical payments.

4. **Name Formatting**:
   - Use payee name 1 and 2 if provided; otherwise, use vendor name.
   - If name overflow flag is `'Y'`, use address line 1 as secondary name.

5. **Report Requirements**:
   - Group by 1099 type with subtotals.
   - Include vendor count per 1099 type.
   - Support multiple report copies (default 1 if zero specified).

## Calculations
- **Primary Amount (`L1AMT1`)**: Sum of YTD paid (`VNYTDP` or `VNLYDP`) across all records for a vendor.
- **Secondary Amount (`L1AMT2`)**: Sum of second box amount (`VNB2AM`) if applicable.
- **Adjusted Primary Amount**: `L1AMT1` - `L1AMT2` if `L1AMT2` > 0 and second box exists.
- **Box Totals**: Accumulate amounts for boxes 1, 3, 6, and 7 by 1099 type, mapping box 7 to box 3 for reporting.

## Error Handling
- Reject invalid inputs (e.g., non-existent companies, types, or file).
- Display error messages for invalid selections and halt processing until corrected.

