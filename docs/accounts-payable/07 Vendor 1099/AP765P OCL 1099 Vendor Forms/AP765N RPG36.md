The RPG36 program `AP765N` is designed to generate Vendor 1099 forms, specifically tailored for printing three forms per page on laser paper, an evolution from the two-form-per-page design in `AP765`. It processes vendor data from an input file, validates amounts, formats addresses, and prints 1099 forms with fields like payee names, vendor IDs, and amounts. The program includes revisions for handling payee names, name overflow, vendor numbers as account numbers, and laser paper printing. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

### Process Steps

1. **File and Data Structure Definitions**:
   - **File Definitions (F-Spec)**:
     - `AP766` (Line 0008): Input file (`IP`), disk-based, with a record length of 300 bytes, containing vendor data.
     - `AP1099` (Line 0009): Output file (`O`), printer file with a record length of 80 bytes, used for printing 1099 forms.
     - `LAP1099` (Line 0011): Logical file for the printer, specifying 66 lines per page with overflow at line 66.
   - **Input Specifications (I-Spec)**:
     - **AP766 Record Format (NS 01)** (Lines 0012–0025):
       - Fields include `VNDEL` (delete code), `VN1099` (1099 code), `VNVEND` (vendor number), `VNNAME` (vendor name), `VNADD1–VNADD4` (address lines), `L1AMT1` and `L1AMT2` (amounts for two boxes), `BOX1` and `BOX2` (box numbers), `VNID#` (1099 ID), `VNPYN1` and `VNPYN2` (payee names 1 and 2, added in JB01), `VNNOVF` (name overflow flag, added in JB02), `STATID`, and `STATED` (state-related fields).
     - **Data Structure (UDS)** (Lines 0026–0032):
       - Fields include `HEAD1`, `HEAD2`, `HEAD3` (headings), `ID#` (company ID), `ENTAMT` (entered amount), `CURLST` (current/last indicator), `CNYEAR`, and `YEAR` (year fields).
   - **Output Specifications (O-Spec)** (Lines 0107–0141):
     - Defines output formats for printing three 1099 forms per page, controlled by indicators 31 (top form), 32 (middle form), and 40 (bottom form).
     - Outputs fields like headings, amounts, vendor/payee names, addresses, and totals, with specific positioning on the form.

2. **Calculation Specifications (C-Spec)**:
   - **Initial Setup and Vendor Checks**:
     - Line 0033: Turn off indicator 33 (Miscellaneous Income flag, commented out for 'M' check).
     - If `VN1099` is 'N' (Nonemployee Compensation), set indicator 34.
     - If `VNVEND` equals 32800, set `YES` to 'YES'.
   - **Amount Validation**:
     - Line 0034: Add `L1AMT1` and `L1AMT2` to `TOTAMT`.
     - Line 0035: Compare `TOTAMT` with `ENTAMT`. If they don’t match, set indicator 30 and branch to `END` (Line 0036).
   - **Form Counter Logic**:
     - Lines 0037: `COUNTP` tracks whether to print the top (31), middle (32), or bottom (40) form:
       - If `COUNTP` is 2, increment to 3, set indicator 40, turn off 32, and branch to `SKIP`.
       - If `COUNTP` is zero, set to 1, turn on 31, turn off 32 and 40.
       - Otherwise, increment `COUNTP`, turn off 31 and 40, and turn on 32.
   - **Clear Work Fields**:
     - Lines JB01: For each form (31, 32, 40), clear fields like `PYN11`, `PYN12`, `PYN13`, `PYN21`, `PYN22`, `PYN23`, `VNID#1`, `VNID#2`, `VNID#3`, `STATE1`, `STATE2`, `STATE3`, `STATI1`, `STATI2`, `STATI3`, `VNVEN1`, `VNVEN2`, `VNVEN3`, and amount fields (`AMT11`, `AMT31`, `AMT61`, `AMT71`, etc.).
   - **Payee and Address Processing for Form 1 (Indicator 31)**:
     - Lines 0038–0069: If `VNPYN1` is blank, use `VNNAME` for `PYN11`. If `VNNOVF` is 'Y', set indicators 35 and 71 for name overflow.
     - Address lines (`VNADD1–VNADD4`) are checked in descending order. The highest non-blank address line is moved to `CTSTZ1`, with lower lines shifted to `VNAD11`, `VNAD21`, or `PYN21` as needed.
     - If `VNPYN1` is not blank, use `VNPYN1` and `VNPYN2` for `PYN11` and `PYN21`, respectively, and set indicator 71.
   - **Payee and Address Processing for Form 2 (Indicator 32)**:
     - Similar logic, using fields like `PYN12`, `PYN22`, `CTSTZ2`, `VNAD12`, `VNAD22`, with indicators 36 and 72.
   - **Payee and Address Processing for Form 3 (Indicator 40)**:
     - Similar logic, using fields like `PYN13`, `PYN23`, `CTSTZ3`, `VNAD13`, `VNAD23`, with indicators 37 and 73.
   - **Additional Address Processing**:
     - Lines 0045–0069 (for 31, 32, 40): Additional checks for non-overflow cases (`N35`, `N36`, `N37`) to format addresses into `CTSTZ1`, `VNAD11`, `VNAD21`, etc.
   - **Amount Assignment**:
     - Lines 0071–0094: Assign `L1AMT1` and `L1AMT2` to specific amount fields (`AMT11`, `AMT31`, `AMT61`, `AMT71` for Form 1; `AMT12`, `AMT32`, `AMT62`, `AMT72` for Form 2; `AMT13`, `AMT33`, `AMT63`, `AMT73` for Form 3) based on `BOX1` and `BOX2` values (1, 3, 6, or 7).
   - **Accumulate Totals**:
     - Lines 0095–0098: For each form, increment `COUNT` and add amounts to running totals (`LRAMT1`, `LRAMT3`, `LRAMT6`, `LRAMT7`).
   - **Print Logic**:
     - Line 0100: If `COUNTP` equals 3, reset `COUNTP`, set indicators 31, 32, and 40, and execute the `EXCPT` operation to print all three forms.
   - **End-of-File Handling**:
     - If the end of file is reached (`LR` on) and indicator 31 is on, clear fields, set indicator 30, and print the last form with `EXCPT`.
   - **Output Execution**:
     - The `EXCPT` operation triggers printing of the forms based on indicators 31, 32, and 40, outputting fields to the printer file.

3. **Output Processing**:
   - **Form 1 (Indicator 31)**:
     - Outputs headings (`HEAD1`, `HEAD2`, `HEAD3`), year (`CNYEAR`), amounts (`AMT11`, `AMT31`), IDs (`ID#`, `VNID#1`), payee names (`PYN11`, `PYN21`), addresses (`VNAD11`, `VNAD21`, `CTSTZ1`), and vendor number (`VNVEN1`).
   - **Form 2 (Indicator 32)**:
     - Similar to Form 1 but uses fields like `AMT12`, `AMT32`, `PYN12`, `PYN22`, `VNAD12`, `VNAD22`, `CTSTZ2`, `VNVEN2`.
   - **Form 3 (Indicator 40)**:
     - Similar to Form 1 but uses fields like `AMT13`, `AMT33`, `PYN13`, `PYN23`, `VNAD13`, `VNAD23`, `CTSTZ3`, `VNVEN3`.
   - **Totals (LR Indicator)**:
     - Prints totals (`LRAMT1`, `LRAMT6`, `LRAMT7`, `COUNT`) and headings at the end of the job.

### Business Rules

1. **Amount Validation**:
   - The sum of `L1AMT1` and `L1AMT2` must equal `ENTAMT`. If not, the program skips to the `END` tag, bypassing further processing for that record.
2. **Form Layout**:
   - Three forms are printed per page (top, middle, bottom), controlled by indicators 31, 32, and 40, respectively. `COUNTP` cycles through 1 (top), 2 (middle), and 3 (bottom).
   - If fewer than three forms remain at the end, the last form is printed with indicator 30.
3. **Payee Name Handling** (JB01, 2013):
   - If `VNPYN1` is blank, use `VNNAME`. If `VNPYN1` and `VNPYN2` are provided, use them for printing payee names.
4. **Name Overflow** (JB02, 2014):
   - If `VNNOVF` is 'Y', set indicators 35/71 (Form 1), 36/72 (Form 2), or 37/73 (Form 3) to handle name continuation on the second line.
5. **Address Formatting**:
   - Address lines are processed from `VNADD4` to `VNADD1`, using the highest non-blank line for `CTSTZ1`/`CTSTZ2`/`CTSTZ3`, with lower lines shifted to `VNAD11`/`VNAD21`, etc.
6. **Amount Mapping**:
   - Amounts (`L1AMT1`, `L1AMT2`) are assigned to specific boxes (1, 3, 6, or 7) based on `BOX1` and `BOX2` values, corresponding to IRS 1099 form fields.
7. **Vendor Number as Account** (JB02, 2014):
   - Vendor number (`VNVEND`) is printed as the account number (`VNVEN1`, `VNVEN2`, `VNVEN3`).
8. **Laser Paper Printing** (MG03, 2016):
   - Designed to print on laser paper with three forms per page.
9. **Three Different Forms** (MG04, 2021):
   - Supports printing three distinct 1099 forms per page, with separate fields for each.
10. **1099 Code Handling**:
    - If `VN1099` is 'N', set indicator 34 (Nonemployee Compensation). The check for 'M' (Miscellaneous) is commented out, suggesting a focus on Nonemployee Compensation forms.

### Tables Used

- **AP766**:
  - Input disk file containing vendor data: `VNDEL`, `VN1099`, `VNVEND`, `VNNAME`, `VNADD1–VNADD4`, `L1AMT1`, `L1AMT2`, `BOX1`, `BOX2`, `VNID#`, `VNPYN1`, `VNPYN2`, `VNNOVF`, `STATID`, `STATED`.
- **No explicit arrays** are defined, but work fields like `AMT11`, `AMT31`, `LRAMT1`, etc., act as temporary storage for calculations and printing.

### External Programs Called

- **None**:
  - The program does not explicitly call any external programs (no `CALL` statements).
  - It is invoked from a main OCL script, but no details about the OCL or other programs are provided.

### Summary

The `AP765N` program:
- Reads vendor data from the `AP766` file and prints 1099 forms to the `AP1099` printer file, formatted for laser paper with three forms per page.
- Validates that the sum of `L1AMT1` and `L1AMT2` matches `ENTAMT`.
- Formats payee names, using `VNPYN1` and `VNPYN2` if provided, or `VNNAME` otherwise, with overflow handling.
- Processes addresses by selecting the highest non-blank line and shifting others accordingly.
- Maps amounts to specific 1099 form boxes (1, 3, 6, 7) based on `BOX1` and `BOX2`.
- Accumulates totals and prints a summary at the end.
- Uses indicators to control printing of top (31), middle (32), and bottom (40) forms, with special handling for the last form.
- Does not call external programs but is part of a larger OCL-driven process.