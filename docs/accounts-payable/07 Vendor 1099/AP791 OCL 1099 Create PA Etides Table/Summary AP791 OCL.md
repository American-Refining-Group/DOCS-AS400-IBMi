### List of Use Cases Implemented by the AP791 Program

The `AP791` program, driven by the OCL script (`AP791.ocl36.txt`) and the RPG program (`AP791.rpg36.txt`), implements a single primary use case:

1. **Generate and Print IRS 1099 Forms**:
   - **Description**: The program processes tax-related data to produce 1099 forms for IRS submission and a printed report for record-keeping. It reads input data from the `IRSTAX` file, processes payee records, accumulates payment totals, and outputs formatted data to the `PA1099` file and a printer report.
   - **Inputs**: A four-digit year (provided via user input in the OCL script) and tax data from the `IRSTAX` file.
   - **Outputs**: A formatted `PA1099` file for IRS submission and a printed report summarizing 1099 data.

### Function Requirement Document



# Function Requirement Document: Generate 1099 Forms

## Overview
The `Generate_1099_Forms` function processes tax data to produce IRS-compliant 1099 forms and a printed summary report for payees. It takes a four-digit year and a tax data file as inputs, generates a formatted output file for IRS submission, and produces a printed report with payment details.

## Inputs
- **Year**: A four-digit year (e.g., "2025") for the 1099 forms.
- **Input File (`IRSTAX`)**: A file containing tax records (750 bytes per record), including:
  - Record type (`T` for transmitter, `B` for payee).
  - Payment year, taxpayer ID number (TIN), payment amounts, payee details (name, address, city, state, zip), and withholding amounts.

## Outputs
- **Output File (`PA1099`)**: A file (875 bytes per record) containing IRS-compliant 1099 data for payees, including payer details, payee details, payment amounts, and withholding.
- **Printed Report (`PRINT`)**: A formatted report (164 bytes per line) with headers, payee details, payment amounts, and totals.

## Process Steps
1. **Validate Year Input**:
   - Accept a four-digit year to define the reporting period.
   - Construct a dynamic output file name (e.g., `P109925` for 2025).

2. **Clear and Prepare Files**:
   - Clear the `PATAX` file to store processed data.
   - Delete any existing file with the dynamic name (e.g., `P109925`) to avoid conflicts.
   - Create a new file with the dynamic name for output.

3. **Process Tax Records**:
   - Read records from the `IRSTAX` file.
   - Skip transmitter (`T`) records.
   - For payee (`B`) records:
     - Extract TIN (EIN or SSN), payment amounts (e.g., non-employee compensation, rents), payee details, and withholding.
     - Accumulate non-zero payment amounts into category totals (e.g., `TPAY1`, `TPAY2`, `TPAY3`, `TPAY7`) and a combined total (`DPAY`).

4. **Generate Output File**:
   - Write payee records to the `PA1099` file, including:
     - Fixed payer details (TIN: `222318612`, Name: "AMERICAN REFINING GROUP, INC.", Address: "77 NORTH KENDALL AVENUE, BRADFORD, PA 16701").
     - Payee details (TIN, name, address, city, state, zip).
     - Payment amounts and withholding (state and local).
     - Combined payment total (`DPAY`).

5. **Generate Printed Report**:
   - Print a header with company name, program name (`AP791`), report title, page number, and date.
   - For each payee record, print:
     - Record number, TIN type (EIN/SSN), name control, formatted TIN, payment amounts, name, address, city, state, and zip.
   - Print totals for payment categories at the end.

6. **Copy Output File**:
   - Copy the `PATAX` file to the dynamically named file (e.g., `P109925`) for storage or submission.

## Business Rules
1. **Record Processing**:
   - Process only payee (`B`) records; skip transmitter (`T`) records.
   - Include only non-zero payment amounts in totals.

2. **TIN Handling**:
   - Identify TIN as EIN (`A1TTIN = '1'`) or SSN (`A1TTIN = '2'`).
   - Format TINs for printing (e.g., `XXX-XXXXXX` for EIN, `XXX-XX-XXXX` for SSN).

3. **Payment Categories**:
   - Accumulate payments for specific 1099 categories (e.g., `A1PAY1`, `A1PAY2`, `A1PAY3`, `A1PAY7`) and compute a combined total (`DPAY`).

4. **IRS Compliance**:
   - Include mandatory fields in `PA1099` (payer/payee TIN, payment amounts, withholding, address).
   - Use fixed payer details for "AMERICAN REFINING GROUP, INC."

5. **Report Formatting**:
   - Align printed data in columns with headers for readability.
   - Include running totals for payment categories in the report.

## Calculations
- **Payment Totals**:
  - For each payee record, if `A1PAY1`, `A1PAY2`, `A1PAY3`, or `A1PAY7` is non-zero, add to respective totals (`TPAY1`, `TPAY2`, `TPAY3`, `TPAY7`) and combined total (`DPAY`).
  - Example: `TPAY1 += A1PAY1`, `DPAY += A1PAY1 + A1PAY2 + A1PAY3 + A1PAY7`.

- **Record Count**:
  - Increment `REC#` and `COUNT` for each processed payee record.

## Assumptions
- The input file (`IRSTAX`) is pre-populated with valid 1099 data.
- The printer is configured for 15 CPI (characters per inch) output.
- The program is executed in an IBM System/36 or AS/400 environment.

## Dependencies
- **Files**:
  - Input: `IRSTAX` (750 bytes).
  - Output: `PA1099` (875 bytes), `PRINT` (164 bytes).
  - Temporary: `PATAX` (cleared and used for processing).
- **External Program**: `GSY2K` (called by OCL for year-related processing).

