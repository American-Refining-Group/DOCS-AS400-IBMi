### List of Use Cases Implemented by the AP790 Program

Based on the provided OCL (`AP790.ocl36.txt`) and RPG/36 (`AP790.rpg36.txt`) programs, the primary use case implemented is:

1. **Generate 1099 File Edit Report**:
   - **Description**: The program processes an Accounts Payable (A/P) 1099 file (`AP1099`) to produce a printed edit report for verifying 1099 data (e.g., payee details, payment amounts, and totals) before submission to the IRS. It ensures the data is accurate and compliant with IRS 1099 reporting requirements, supporting forms such as 1099-MISC, 1099-DIV, or 1099-INT.
   - **Scope**: The program reads structured records (Transmitter, Employer, Payee, End of Payer, State Totals, Company Final), validates key fields (e.g., TINs), accumulates payment totals, and formats a report with payee details and aggregated amounts.
   - **Inputs**: The `AP1099` file containing 1099 data and a substitution variable (`?9?`) for dynamic file labeling.
   - **Outputs**: A printed report listing payee details (TIN, name, address, payment amounts) and totals, formatted for verification.

No additional use cases are evident from the provided code, as the program focuses solely on generating the edit report for the 1099 file.

---

### Function Requirement Document



# Function Requirement Document: Generate 1099 File Edit Report

## Overview
The `Generate_1099_File_Edit_Report` function processes an Accounts Payable (A/P) 1099 file to produce a printed edit report for verifying 1099 data before IRS submission. It supports IRS 1099 forms (e.g., 1099-MISC, 1099-DIV, 1099-INT) by validating and formatting payee details, payment amounts, and totals.

## Inputs
- **1099 Data File** (`AP1099`):
  - Format: Fixed-length (750 bytes) disk file.
  - Record Types:
    - Transmitter (`T`): TIN, name, address, contact details.
    - Employer (`A`): Payer TIN, name, address, phone.
    - Payee (`B`): Payee TIN (EIN/SSN), name, address, payment amounts (1, 3, 6, 7).
    - End of Payer (`C`): Total payee count and payment amounts.
    - State Totals (`K`): State-level payment totals.
    - Company Final (`F`): Total count of Employer records.
- **File Label Parameter** (`label_prefix`): String (e.g., year or identifier) to construct file label (`label_prefix + "AP1099"` or `"AP1099"`).
- **Printer Settings**:
  - Device: `PRINT`.
  - Characters Per Inch (CPI): 15.

## Outputs
- **Printed Edit Report**:
  - **Header**:
    - Company: "AMERICAN REFINING GROUP".
    - Title: "EDIT OF A/P 1099 FILE".
    - Program: "AP790".
    - Page number and date (MM/DD/YY).
    - Column headings: Record #, Type, Abbreviation, SSN/EIN, Amount 1, Amount 6, Amount 3, Name, Address, City, State, ZIP.
  - **Detail Lines** (Payee records):
    - Record number, TIN type (EIN/SSN), name control, formatted TIN (XX-XXXXXXX for EIN, XXX-XX-XXXX for SSN), payment amounts (1, 3, 6, 7), payee name, address, city, state, ZIP.
  - **Totals** (End of Payer records):
    - Payee count and total payment amounts (1, 3, 6, 7).
  - Format: 164-character lines, 15 CPI.

## Process Steps
1. **Initialize**:
   - Set record counter (`REC#`) to 0.
   - Set payee counter (`COUNT`) and payment totals (`TAMT1`, `TAMT3`, `TAMT6`, `TAMT7`) to 0.
2. **Assign File**:
   - If `label_prefix = "G"`, use file label `"AP1099"`.
   - Else, use file label `label_prefix + "AP1099"`.
   - Open file in shared mode (`DISP-SHR`).
3. **Read 1099 File**:
   - Sequentially read records from `AP1099`.
   - Identify record type using `A1REC` (position 1): `T`, `A`, `B`, `C`, `K`, `F`.
4. **Process Payee Records (`A1REC = 'B'`)**:
   - Increment `REC#` and `COUNT`.
   - If deletion flag (`A1DEL`) is `'D'`, skip record.
   - Validate TIN type (`A1TTIN`): `'1'` (EIN) or `'2'` (SSN).
   - Accumulate payment amounts:
     - `TAMT1 += A1PAY1` (Payment Amount 1).
     - `TAMT3 += A1PAY3` (Payment Amount 3).
     - `TAMT6 += A1PAY6` (Payment Amount 6).
     - `TAMT7 += A1PAY7` (Payment Amount 7).
   - Output payee details to report (if not deleted):
     - Record number, TIN type, name control, formatted TIN, payment amounts, name, address, city, state, ZIP.
5. **Process End of Payer Records (`A1REC = 'C'`)**:
   - Output totals: `COUNT`, `TAMT1`, `TAMT3`, `TAMT6`, `TAMT7`.
6. **Process Other Records**:
   - Read but do not process Transmitter (`T`), Employer (`A`), State Totals (`K`), or Company Final (`F`) records for calculations.
   - Include relevant fields (e.g., counts, totals) in report if specified.
7. **Generate Report**:
   - Print header at page start.
   - Print detail lines for each valid Payee record.
   - Print totals for End of Payer records.
   - Format output at 15 CPI to printer device `PRINT`.

## Business Rules
1. **IRS Compliance**:
   - Process 1099 file per IRS specifications (e.g., 1099-MISC, 1099-DIV, 1099-INT).
   - Validate TINs: EIN (`'1'`, formatted as `XX-XXXXXXX`) or SSN (`'2'`, formatted as `XXX-XX-XXXX`).
   - Include required fields (e.g., payment amounts, payee details, state withholding).
2. **Deletion Handling**:
   - Skip records with `A1DEL = 'D'`.
3. **Payment Accumulation**:
   - Sum payment amounts (1, 3, 6, 7) for Payee records into `TAMT1`, `TAMT3`, `TAMT6`, `TAMT7`.
   - Report totals for verification against End of Payer records.
4. **Report Formatting**:
   - Ensure payee details (name, address, TIN, payments) are printed clearly for manual review.
   - Suppress leading zeros in counts and format monetary amounts appropriately.
5. **File Flexibility**:
   - Support dynamic file labeling based on `label_prefix` for different 1099 file versions (e.g., by year).

## Calculations
- **Record Counter** (`REC#`): Incremented by 1 for each Payee record processed (`REC# += 1`).
- **Payee Counter** (`COUNT`): Incremented by 1 for each non-deleted Payee record (`COUNT += 1`).
- **Payment Totals**:
  - `TAMT1 += A1PAY1` (sum of Payment Amount 1).
  - `TAMT3 += A1PAY3` (sum of Payment Amount 3).
  - `TAMT6 += A1PAY6` (sum of Payment Amount 6).
  - `TAMT7 += A1PAY7` (sum of Payment Amount 7).
- **TIN Formatting**:
  - If `A1TTIN = '1'`: Format `A1TIN` as `XX-XXXXXXX` (using `A1ID21`, `A1ID22`).
  - If `A1TTIN = '2'`: Format `A1TIN` as `XXX-XX-XXXX` (using `A1ID11`, `A1ID12`, `A1ID13`).

## Assumptions
- The function processes a single 1099 file at a time.
- Only payment amounts 1, 3, 6, and 7 are relevant for the report (others ignored).
- The printer is pre-configured and available as `PRINT`.

## Constraints
- File format must adhere to IRS 1099 specifications (750-byte records).
- Program runs on IBM System/36 with RPG/36 and OCL support.
- No external programs called; all processing is self-contained.

