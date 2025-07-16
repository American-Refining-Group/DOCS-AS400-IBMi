### Use Cases Implemented by the Program Call Stack

The call stack consists of three RPG36 programs: `AP765P`, `AP765`, and `AP765N`. Together, they implement a system for generating Vendor 1099 forms for tax reporting. The primary use case is:

1. **Generate Vendor 1099 Forms**:
   - **Description**: The system collects and validates input data for 1099 forms, processes vendor data from a file, and prints either two (`AP765`) or three (`AP765N`) 1099 forms per page on laser paper. The process includes validating user inputs, formatting vendor names and addresses, mapping payment amounts to specific 1099 form boxes, and printing forms with totals.
   - **Components**:
     - `AP765P`: Interactive screen program to collect and validate user inputs (e.g., headings, company ID, amount, type, current/last indicator).
     - `AP765`: Processes vendor data and prints two 1099 forms per page.
     - `AP765N`: Processes vendor data and prints three 1099 forms per page, likely an updated version for higher throughput.

### Function Requirement Document



# Vendor 1099 Form Generation Function Requirements

## Overview
This function processes vendor data to generate IRS 1099 forms, formatting and printing either two or three forms per page on laser paper. It validates input data, formats vendor/payee names and addresses, maps payment amounts to specific form boxes, and accumulates totals.

## Inputs
- **Company Information**:
  - `HEAD1`, `HEAD2`, `HEAD3` (30 chars each): Company headings (e.g., name, address).
  - `ID#` (10 chars): Company tax ID.
  - `ENTAMT` (8 chars, 2 decimals): Total amount for validation.
  - `TYPE` (1 char): Form type ('D' for Dividend, 'I' for Interest, 'M' for Miscellaneous, 'N' for Nonemployee Compensation).
  - `CURLST` (1 char): Current ('C') or Last ('L') year indicator.
  - `CNYEAR`/`YEAR` (4 chars): Calendar year for the form.
- **Vendor Data** (from file `AP766`):
  - `VNDEL` (1 char): Delete code.
  - `VN1099` (1 char): 1099 code ('M' or 'N').
  - `VNVEND` (5 chars): Vendor number.
  - `VNNAME` (30 chars): Vendor name.
  - `VNADD1–VNADD4` (30 chars each): Address lines.
  - `L1AMT1`, `L1AMT2` (9 chars, 2 decimals): Payment amounts for two boxes.
  - `BOX1`, `BOX2` (2 chars): Box numbers (1, 3, 6, or 7) for 1099 form fields.
  - `VNID#` (11 chars): Vendor tax ID.
  - `VNPYN1`, `VNPYN2` (40 chars each): Payee names (optional).
  - `VNNOVF` (1 char): Name overflow flag ('Y' or blank).
  - `STATID` (8 chars), `STATED` (9 chars, 2 decimals): State-related fields.

## Outputs
- **Printed 1099 Forms** (to printer file `AP1099`):
  - Two forms per page (`AP765`) or three forms per page (`AP765N`).
  - Fields: Headings, company ID, vendor/payee names, addresses, vendor number (as account number), amounts in boxes 1, 3, 6, or 7, year.
  - Totals: Count of forms, total amounts for boxes 1, 3, 6, 7.
- **Error Messages** (if validation fails):
  - Returned as a list of strings for invalid inputs.

## Process Steps
1. **Validate Input Data**:
   - Check `HEAD1`, `HEAD2`, `ID#` are not blank.
   - Verify `TYPE` is 'D', 'I', 'M', or 'N'.
   - Ensure `CURLST` is 'C' or 'L'.
   - Return error messages for any invalid inputs.
2. **Process Vendor Records**:
   - Read vendor data from `AP766`.
   - Validate that `L1AMT1` + `L1AMT2` equals `ENTAMT`. Skip record if invalid.
   - If `VN1099` is 'N', mark as Nonemployee Compensation; if 'M', mark as Miscellaneous (optional in `AP765N`).
   - If `VNVEND` is 9384 (`AP765`) or 32800 (`AP765N`), set a flag (`YES`).
3. **Format Names**:
   - If `VNPYN1` is blank, use `VNNAME`; otherwise, use `VNPYN1` and `VNPYN2` (if provided).
   - If `VNNOVF` is 'Y', format name to continue on a second line.
4. **Format Addresses**:
   - Select highest non-blank address line (`VNADD4` to `VNADD1`) for city/state/zip; shift lower lines to address fields.
5. **Map Amounts**:
   - Assign `L1AMT1` and `L1AMT2` to box 1, 3, 6, or 7 based on `BOX1` and `BOX2` values.
6. **Print Forms**:
   - Accumulate up to two (`AP765`) or three (`AP765N`) forms per page.
   - Print fields: headings, year, company ID, vendor ID, vendor number (as account), payee names, addresses, amounts.
   - If fewer than the maximum forms remain at end-of-file, print remaining form(s).
7. **Accumulate and Print Totals**:
   - Track count of forms and sum amounts for boxes 1, 3, 6, 7.
   - Print totals at job end.

## Business Rules
1. **Mandatory Fields**: `HEAD1`, `HEAD2`, `ID#` must not be blank.
2. **Valid Type**: `TYPE` must be 'D', 'I', 'M', or 'N'.
3. **Valid Year Indicator**: `CURLST` must be 'C' or 'L'.
4. **Amount Validation**: `L1AMT1` + `L1AMT2` must equal `ENTAMT`.
5. **Payee Names**: Use `VNPYN1` and `VNPYN2` if provided; otherwise, use `VNNAME`.
6. **Name Overflow**: If `VNNOVF` is 'Y', extend name to second line.
7. **Address Formatting**: Use highest non-blank address line for city/state/zip; shift others to address lines.
8. **Box Mapping**: Map amounts to boxes 1, 3, 6, or 7 based on `BOX1` and `BOX2`.
9. **Form Layout**: Print two (`AP765`) or three (`AP765N`) forms per page on laser paper.
10. **Vendor Number**: Print `VNVEND` as account number.
11. **Form Type**: Support Miscellaneous ('M') and Nonemployee Compensation ('N') forms; 'M' check is optional in `AP765N`.
12. **Totals**: Accumulate and print total forms and amounts for boxes 1, 3, 6, 7.

## Calculations
- **Total Amount Validation**: `TOTAMT = L1AMT1 + L1AMT2`. Compare with `ENTAMT`.
- **Form Counter**: Increment `COUNTP` (1 to 2 for `AP765`, 1 to 3 for `AP765N`) to track forms per page. Reset to 0 after printing.
- **Running Totals**:
  - `COUNT`: Increment by 1 per form.
  - `LRAMT1`, `LRAMT3`, `LRAMT6`, `LRAMT7`: Sum amounts for boxes 1, 3, 6, 7, respectively.

## Error Handling
- Return error messages for blank `HEAD1`, `HEAD2`, `ID#`, invalid `TYPE`, or invalid `CURLST`.
- Skip vendor records if `L1AMT1` + `L1AMT2` ≠ `ENTAMT`.

## Assumptions
- Input data is provided programmatically (not via screen).
- Output is formatted for a printer file (`AP1099`) compatible with laser paper.
- `AP766` file contains valid vendor data.

