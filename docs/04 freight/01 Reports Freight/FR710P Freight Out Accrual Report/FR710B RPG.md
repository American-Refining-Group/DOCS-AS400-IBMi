The RPG program `FR710B` is responsible for printing the Freight Out Accrual Report using data from the `FR710W` work file, which was populated by `FR710A`. Called by `FR710C`, it generates a detailed report with totals at the freight general ledger (GL) and company levels, and supports two output formats: a standard report (`LIST164`) and an Excel-compatible format (`LIST378`) added in revision `MG01`. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

### Process Steps of the FR710B Program

1. **Program Initialization (`*INZSR` Subroutine)**:
   - **Purpose**: Initializes the program environment and sets up variables.
   - **Steps**:
     - Receives three input parameters:
       - `P$CO`: Company number (2 characters).
       - `P$RDAT`: Report date (8 digits, `CCYYMMDD` format).
       - `P$FGRP`: File group ('G' or 'Z') for file overrides.
     - Moves `P$RDAT` to `D$RDAT` for use in report headers.
     - Defines a key list (`KEY`) for accessing `GLMAST` using `W1CO` (company), `W$ACCT` (account), and `W$SUB` (sub-account) from the `W1FRGL` field.
     - Initializes arrays `STR` ('* ') and `HYP` ('- ') for formatting report lines.
     - Sets `PRTOVR` to `*ON` to trigger header printing on the first page.
     - Sets `PRTHDR` to `*OFF` to control report header printing.
     - Calls the `OPNTBL` subroutine to open files.

2. **Open Database Tables (`OPNTBL` Subroutine)**:
   - **Purpose**: Opens input files with appropriate overrides based on the file group.
   - **Steps**:
     - Checks the `P$FGRP` parameter ('G' or 'Z').
     - For 'G', applies overrides from the `OVG` array to point to `GGLMAST` and `GGLCONT`.
     - For 'Z', applies overrides from the `OVZ` array to point to `ZGLMAST` and `ZGLCONT`.
     - Executes overrides using the `QCMDEXC` system API, passing the override command and length (80).
     - Opens the `GLMAST` and `GLCONT` files.

3. **Main Processing Loop**:
   - **Purpose**: Processes records from the `FR710W` file, which is defined with level breaks (`L1` for freight GL, `L2` for company).
   - **Steps**:
     - **Company Level Break (`*INL2`)**:
       - If a company-level break occurs (`*INL2` is `*ON`), initializes company-level totals (`L2BFRT`, `L2EFRT`, `L2AFRT`, `L2DIFF`) to zero.
       - Sets `PRTOVR` to `*ON` to print headers.
       - Chains to `GLCONT` using `W1CO` to retrieve the company name (`GCNAME`) into `R$CNAM`. If not found (`*IN99` is `*ON`), clears `R$CNAM`.
     - **Freight GL Level Break (`*INL1`)**:
       - If a freight GL-level break occurs (`*INL1` is `*ON`), initializes GL-level totals (`L1BFRT`, `L1EFRT`, `L1AFRT`, `L1DIFF`) to zero.
       - Chains to `GLMAST` using the `KEY` key list to retrieve the GL description (`GLDESC`) into `R$DESC`. If not found (`*IN99` is `*ON`), clears `R$DESC`.
     - Calls the `DTL` subroutine to process detail records.
     - At the freight GL level break (`L1`), calls `L1TOTL` to print GL totals.
     - At the company level break (`L2`), calls `L2TOTL` to print company totals.
     - Note: The `LRTOTL` subroutine for final totals is commented out, suggesting it may not be used in the current implementation.

4. **Process Detail Records (`DTL` Subroutine)**:
   - **Purpose**: Processes each record from `FR710W` and prints detail lines.
   - **Steps**:
     - Accumulates detail amounts into GL-level totals:
       - `W1BFRT` (billed freight) to `L1BFRT`.
       - `W1EFRT` (expected carrier) to `L1EFRT`.
       - `W1AFRT` (actual carrier) to `L1AFRT`.
       - `W1DIFF` (difference) to `L1DIFF`.
     - Tracks positive and negative differences:
       - If `W1DIFF` > 0, adds to `L2POST` (carrier charges greater).
       - If `W1DIFF` ≤ 0, adds to `L2NEGT` (carrier charges less).
     - Converts the ship date (`W1SDAT`) to `W$SDAT` and adjusts it by multiplying by 100.0001 (likely for formatting or sorting purposes).
     - Calls `OVRFLO` to handle page overflow.
     - Writes a detail line to the report using the `DETL` exception output.

5. **Print Freight GL Totals (`L1TOTL` Subroutine)**:
   - **Purpose**: Prints totals for each freight GL group.
   - **Steps**:
     - Calls `OVRFLO` to handle page overflow.
     - Writes the GL total line (`TOTL1`) with `W1FRGL`, `R$DESC`, `L1BFRT`, `L1EFRT`, `L1AFRT`, and `L1DIFF`.
     - Turns off indicator `*IN82` (likely used for formatting).
     - Accumulates GL totals into company totals (`L2BFRT`, `L2EFRT`, `L2AFRT`, `L2DIFF`).

6. **Print Company Totals (`L2TOTL` Subroutine)**:
   - **Purpose**: Prints totals for each company group.
   - **Steps**:
     - Calls `OVRFLO` to handle page overflow.
     - Writes the company total line (`TOTL2`) with `L2BFRT`, `L2EFRT`, `L2AFRT`, `L2DIFF`, `L2POST`, and `L2NEGT`.
     - Accumulates company totals into report-level totals (`LRBFRT`, `LREFRT`, `LRAFRT`, `LRDIFF`).

7. **Handle Page Overflow (`OVRFLO` Subroutine)**:
   - **Purpose**: Manages page breaks and prints report headers when needed.
   - **Steps**:
     - If overflow indicator `*INOF` is `*ON`, sets `PRTOVR` and `*IN82` to `*ON`.
     - If `PRTOVR` is `*ON`, writes the report header (`HDR01`) and resets `PRTOVR` to `*OFF`.

8. **Program Termination**:
   - **Purpose**: Completes processing and closes files.
   - **Steps**:
     - The program implicitly closes files at the end (via `*INLR` in the RPG cycle, though not explicitly shown).
     - Returns control to the caller (`FR710C`).

### Business Rules
- **Report Structure**:
  - The report is organized by company (`W1CO`, level `L2`) and freight GL (`W1FRGL`, level `L1`), with detail lines for each record in `FR710W`.
  - Detail lines include company, freight GL, routing, ship date, customer number, customer name, order number, serial number, invoice number, type, quantity, unit of measure, billed freight (`W1BFRT`), expected carrier (`W1EFRT`), actual carrier (`W1AFRT`), difference (`W1DIFF`), close status, and sequence number.
  - Totals are printed at the freight GL level (`TOTL1`) and company level (`TOTL2`), including billed, expected, and actual freight amounts, and differences.
  - Company totals include a breakdown of positive (`L2POST`) and negative (`L2NEGT`) differences to highlight carrier charge discrepancies.
- **Output Formats**:
  - `LIST164`: Standard report format with a width of 164 characters, designed for traditional printing.
  - `LIST378`: Wider format (378 characters) added for Excel creation (revision `MG01`), with adjusted column positions for better spreadsheet compatibility.
- **File Overrides**:
  - Uses `P$FGRP` to select between 'G' (production: `GGLMAST`, `GGLCONT`) and 'Z' (development: `ZGLMAST`, `ZGLCONT`) file sets.
- **Data Enrichment**:
  - Retrieves company name (`GCNAME`) from `GLCONT` and GL description (`GLDESC`) from `GLMAST` to enhance report readability.
  - If lookups fail (`*IN99`), fields are cleared to avoid errors.
- **Page Management**:
  - Headers are printed on page overflow or when triggered by `PRTOVR`, including company, report title, page number, job details, date, time, and selected report date.
- **Date Formatting**:
  - The report date (`D$RDAT`) is split into `D$MM` (month) and `D$CCYY` (year) for header display.
  - Ship date (`W1SDAT`) is adjusted for formatting purposes.

### Tables Used
- **Input Files**:
  - `FR710W`: Primary input file (record format `FR710WPF`) containing freight accrual data from `FR710A`. Processed with level breaks on `W1CO` (`L2`) and `W1FRGL` (`L1`).
  - `GLMAST`: General ledger master file, used to retrieve the freight GL description (`GLDESC`) for `W1FRGL`.
  - `GLCONT`: Company master file, used to retrieve the company name (`GCNAME`) for `W1CO`.
- **Output Files**:
  - `LIST164`: Printer file for the standard report (164 characters wide).
  - `LIST378`: Printer file for the Excel-compatible report (378 characters wide, revision `MG01`).
- **File Overrides**:
  - For `P$FGRP` = 'G': `GGLMAST`, `GGLCONT`.
  - For `P$FGRP` = 'Z': `ZGLMAST`, `ZGLCONT`.

### External Programs Called
- **QCMDEXC**: System API used to execute file override commands for `GLMAST` and `GLCONT`.

### Additional Notes
- **Revision MG01 (11/27/2017)**:
  - Added the `LIST378` printer file to support Excel output, with wider column spacing to accommodate spreadsheet formatting.
- **Error Handling**:
  - Uses indicator `*IN99` to handle failed lookups in `GLMAST` and `GLCONT`, clearing output fields to prevent errors.
  - Lacks explicit error handling for file I/O or printing issues.
- **Commented Code**:
  - The `LRTOTL` subroutine is commented out, suggesting that report-level totals are not currently printed, possibly due to requirements changes.
- **Purpose**:
  - The program formats and prints the Freight Out Accrual Report, providing a detailed breakdown of freight charges and discrepancies, with totals to aid financial analysis.

This program completes the report generation process by producing a formatted output from the `FR710W` work file, supporting both traditional and Excel-compatible formats for flexibility in reporting.