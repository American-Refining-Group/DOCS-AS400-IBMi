The RPGLE program `FR711B` is called by `FR711C` to print the Freight Out Reconciliation Report sorted by location. It reads data from the temporary work file `FR711W`, retrieves additional information from reference files, and generates a formatted report with totals at various levels. Below is an explanation of the process steps, business rules, tables used, and external programs called, based on the provided source code.

---

### Process Steps of the RPGLE Program (`FR711B`)

The program processes records from the `FR711W` work file, organizes them by company, location, and state, and prints a detailed report with subtotals and totals. It also supports an Excel-compatible output file. Here are the detailed steps:

1. **Initialization**:
   - **Purpose**: Sets up the program environment and parameters.
   - **Actions**:
     - Receives seven input parameters:
       - `P$CO`: Company code (2 characters).
       - `P$RDAT`: Report date (8 characters, CYMD format).
       - `P$LOC`: Location code (3 characters).
       - `P$CAID`: Carrier ID (6 characters).
       - `P$SORT`: Sort option (1 character, expected to be 'L' for location).
       - `P$SELMO`: Month selection (1 character, added in revision `jb02`).
       - `P$FGRP`: File group (1 character, 'G' or 'Z').
     - Defines data structures:
       - `W1FRGL`: Splits freight GL account into `W$ACCT` (account, 6 digits) and `W$SUB` (sub-account, 2 digits).
       - `D$RDAT`: Splits report date into `D$CCYY` (year) and `D$MM` (month).
       - `W1NAME`: Splits customer name into `W$NAME` (first 20 characters).
       - `D#CYMD`: Date conversion structure for parsing dates (CYMD, MDY, etc.).
       - `PSDS##`: Program status data structure for job, user, and timestamp information.
     - Defines arrays:
       - `STR` (82 elements): Likely for string formatting (not used in the provided code snippet).
       - `HYP` (66 elements): Likely for hyphenation or formatting (not used in the provided code).
       - `OVG` (2 elements): File overrides for 'G' group (`GINLOC`, `GGLCONT`).
       - `OVZ` (2 elements): File overrides for 'Z' group (`ZINLOC`, `ZGLCONT`).
       - `HDR` (5 elements, revision `jb02`/`jb03`): Descriptions for month selection options.
     - Applies file overrides based on `P$FGRP`:
       - For 'G': Overrides `INLOC` to `GINLOC` and `GLCONT` to `GGLCONT`.
       - For 'Z': Overrides `INLOC` to `ZINLOC` and `GLCONT` to `ZGLCONT`.
     - Opens the `INLOC` and `GLCONT` files.

2. **Processing Input Records**:
   - **Purpose**: Reads records from `FR711W` and organizes them for printing.
   - **Actions**:
     - Reads `FR711W` sequentially, with level-break indicators (`L7` for company, `L6` for location, `L5` for state, `L4` for city, `L3` for routing group, `L2` for customer, `L1` for order/sequence number).
     - At each level break:
       - **Company (L7)**:
         - Sets `PRTOVR` to `*ON` to indicate a new company section.
         - Resets company-level totals (`L7BFRT`, `L7EFRT`, `L7AFRT`, `L7DIFF`, `L7POST`, `L7NEGT`) and selected date range totals (`L7BFRTs`, `L7EFRTs`, `L7AFRTs`, `L7DIFFs`, `L7POSTs`, `L7NEGTs`, revision `jb01`).
         - Chains to `GLCONT` using `W1CO` to retrieve the company name (`GCNAME`) into `R$CNAM`. Clears `R$CNAM` if not found (`*IN99 = *ON`).
       - **Location (L6)**:
         - Sets `PRTOVR` to `*ON` to indicate a new location section.
         - Resets location-level totals (`L6BFRT`, `L6EFRT`, `L6AFRT`, `L6DIFF`, `L6POST`, `L6NEGT`) and selected date range totals (`L6BFRTs`, `L6EFRTs`, `L6AFRTs`, `L6DIFFs`, `L6POSTs`, `L6NEGTs`, revision `jb01`).
         - Chains to `INLOC` using `KEY` (likely company and location) to retrieve the location name (`ILNAME`) into `R$ILNM`. Clears `R$ILNM` if not found (`*IN99 = *ON`).
       - **State (L5)**:
         - Resets state-level totals (`L5BFRT`, `L5EFRT`, `L5AFRT`, `L5DIFF`, `L5POST`, `L5NEGT`) and selected date range totals (`L5BFRTs`, `L5EFRTs`, `L5AFRTs`, `L5DIFFs`, `L5POSTs`, `L5NEGTs`, revision `jb01`).
         - Note: The `PRTOVR` setting for state is commented out, suggesting it may not trigger a page break.

3. **Printing the Report**:
   - **Purpose**: Outputs the report to the printer files `LIST164` and `LIST378` (Excel-compatible, revision `mg01`).
   - **Actions**:
     - **Header (HDR01)**:
       - Prints company information (`W1CO`, `R$CNAM`), report title ("FREIGHT OUT RECONCILIATION REPORT"), and page number (`PAGE1`).
       - Prints job, program, and user information (`PS#JOB`, `PS#PGM`, `PS#USR`), location (`W1LOC`, `R$ILNM`), and month selection description (`D$SELMOHDR`, revision `jb02`).
       - Prints selected location (`P$LOC`), carrier (`P$CAID`), and date (`D$MM`, `D$CCYY`), along with the current date (`PS#MDY`) and time (`PS#HMS`).
       - Prints column headings (e.g., "SHIPPING DESTINATION", "STATE", "ROUTING", "SHP DATE", etc.).
     - **Detail Lines (DETL)**:
       - Prints detail fields from `FR711W`: city (`W1SCIT`), state (`W1SST`), routing group (`W1RTG1`), ship date (`W$SDAT`), customer name (`W$NAME`), order number (`W1RDNO`), sequence number (`W1SRN`), invoice number (`W1INVN`), customer number (`W1CUST`), ship-to (`W1SHIP`), order type (`W1TYPE`), quantity (`W1QTY`), unit of measure (`W1UM`), freight code (`W1FRCD`), billed freight (`W1BFRT`), expected carrier (`W1EFRT`), actual carrier (`W1AFRT`), difference (`W1DIFF`), and closed status (`W1CLOS`).
     - **Location Totals (TOTL6)**:
       - Prints location totals (`L6BFRT`, `L6EFRT`, `L6AFRT`, `L6DIFF`) and carrier charges greater/less (`L6POST`, `L6NEGT`).
     - **Company Totals (TOTL7)**:
       - Prints company totals for the selected month (`L7BFRTs`, `L7EFRTs`, `L7AFRTs`, `L7DIFFs`, `L7POSTs`, `L7NEGTs`, revision `jb01`) and all dates (`L7BFRT`, `L7EFRT`, `L7AFRT`, `L7DIFF`, `L7POST`, `L7NEGT`).
       - Includes labels for "SELECTED MONTH ONLY" and "ALL DATES".
     - **Excel Output (LIST378, revision `mg01`)**:
       - Outputs data to a separate printer file (`LIST378`) for Excel-compatible formatting, though the specific fields are not detailed in the provided code snippet.

4. **Program Termination**:
   - Closes all open files (`INLOC`, `GLCONT`, `FR711W`, `LIST164`, `LIST378`).
   - Sets `*INLR = *ON` to indicate the program is ending.
   - Returns control to the caller (`FR711C`).

---

### Business Rules

- **Report Structure**:
  - The report is organized hierarchically by company (`L7`), location (`L6`), and state (`L5`), with detail lines for each record and totals at the location and company levels.
  - Totals include:
    - Billed freight (`L7BFRT`, `L6BFRT`, `L5BFRT`).
    - Expected carrier charges (`L7EFRT`, `L6EFRT`, `L5EFRT`).
    - Actual carrier charges (`L7AFRT`, `L6AFRT`, `L5AFRT`).
    - Difference (`L7DIFF`, `L6DIFF`, `L5DIFF`).
    - Carrier charges greater (`L7POST`, `L6POST`, `L5POST`).
    - Carrier charges less (`L7NEGT`, `L6NEGT`, `L5NEGT`).
  - Selected date range totals (revision `jb01`) are included for the same fields with an 's' suffix (e.g., `L7BFRTs`) to reflect the filtered month selection.

- **Month Selection Options** (revisions `jb02`, `jb03`):
  - The `P$SELMO` parameter determines the data included, with descriptions printed in the header (`D$SELMOHDR`):
    - `M`: Selected month only, including all (open and closed) records ("Selected Mo Only All (Open/Closed)").
    - `O`: Selected month, open records only ("Selected Month Open Only").
    - `C`: Selected month, closed records only ("Selected Mo Closed Only").
    - `A`: All open records from previous and selected months ("All Open-Prev/Sel Mo").
    - Blank: Previous month open records and selected month all records ("Prev Mo Open/Sel Mo All").
  - The data is pre-filtered by `FR711A` based on these options.

- **File Group**:
  - The `P$FGRP` parameter ('G' or 'Z') determines which reference files are used:
    - 'G': Uses `GINLOC` and `GGLCONT`.
    - 'Z': Uses `ZINLOC` and `ZGLCONT`.

- **Excel Output** (revision `mg01`):
  - A separate printer file (`LIST378`) is used to generate an Excel-compatible output, likely with a wider format (378 characters vs. 164 for `LIST164`), though specific formatting details are not provided in the snippet.

- **Error Handling**:
  - Handles missing company or location records (`*IN99 = *ON`) by clearing the respective name fields (`R$CNAM`, `R$ILNM`).
  - No explicit error handling for file access or printing issues, relying on system-level error management.

- **Revision History**:
  - `jb01` (03/26/2015): Converted from RPG to RPGLE and added selected date range totals.
  - `jb02` (04/07/2015): Added month selection options (`M`, `O`, blank, `A`).
  - `jb03` (11/08/2017): Added `C` for selected month closed records.
  - `mg01` (11/27/2017): Added separate output for Excel creation (`LIST378`).

---

### Tables Used

- **Input Files**:
  - `FR711W`: Temporary work file in `QTEMP` (created by `FR711C`, populated by `FR711A`), containing freight reconciliation data. Read sequentially with level-break indicators (`L7` to `L1`).
  - `INLOC`: Location file, overridden to `GINLOC` or `ZINLOC` based on `P$FGRP`, used to retrieve location names (`ILNAME`).
  - `GLCONT`: Company file, overridden to `GGLCONT` or `ZGLCONT`, used to retrieve company names (`GCNAME`).

- **Output Files**:
  - `LIST164`: Printer file (164 characters wide) for the standard report output.
  - `LIST378`: Printer file (378 characters wide, revision `mg01`) for Excel-compatible output.

---

### External Programs Called

- **QCMDEXC**: System program used to execute file override commands for `INLOC` and `GLCONT` (implied, as overrides are defined in `OVG` and `OVZ` arrays, though the execution is not shown in the snippet).

---

### Additional Notes

- **Purpose**: `FR711B` generates the final Freight Out Reconciliation Report sorted by location, using data from `FR711W` and enriching it with company and location names from reference files.
- **Multi-User Support**: Relies on `FR711C` to create `FR711W` in `QTEMP`, ensuring job-specific data isolation.
- **Parameter Validation**: Assumes input parameters are valid, as they are validated by `FR711P` and passed through `FR711C`.
- **Execution Context**: Runs in the same job session as its caller, producing the report directly to the printer files.
- **Comparison with `FR712B`**: Likely similar in structure but tailored for location-sorted output, while `FR712B` handles carrier-sorted output.

This program produces a detailed, hierarchically organized report with totals, supporting both standard and Excel-compatible formats, based on pre-processed data from the work file.