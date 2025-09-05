The `FR715B` program is an RPGLE program called by `FR715C` to print the Freight Out Reconciliation Report sorted by product code. It reads data from the work file `FR715W`, retrieves additional information from reference files, and generates a printed report with headers, detail lines, and totals. It also supports an additional output for Excel creation (per revision `mg01`). The program is nearly identical to `FR713B` and `FR714B`, with differences primarily in the input work file (`FR715W` vs. `FR713W` or `FR714W`), the sorting hierarchy (product code-focused), and a minor inconsistency in the output work file name (`FR714W2` instead of `FR715W2`). Below, I’ll explain the process steps, business rules, database tables used, and external programs called, based on the provided RPGLE source code (noting that the code is truncated, limiting visibility into some subroutines).

### Process Steps of the FR715B Program

The `FR715B` program processes the work file `FR715W`, organizes data hierarchically by company, location, product code, state, city, and customer, and produces a formatted report with totals at various levels. Here’s a detailed breakdown of the process steps based on the available code:

1. **Program Initialization (`*inzsr` Subroutine, partially truncated)**:
   - **Purpose**: Sets up initial conditions and processes input parameters.
   - **Actions** (inferred from context and file definitions):
     - Receives nine input parameters (as passed from `FR715C`):
       - `p$co` (2): Company code.
       - `p$fdat` (8): From date (in `CYMD` format).
       - `p$tdat` (8): To date (in `CYMD` format).
       - `p$loc` (3): Location code (Note: Matches `FR713C` and `FR714A` but differs from `FR715C`’s LEN(2), suggesting a potential discrepancy).
       - `p$caid` (6): Carrier ID.
       - `p$sort` (1): Sort option ('P' for product code, as called by `FR713PC`).
       - `p$cust` (6): Customer code.
       - `p$car$` (1): Flag to include zero-dollar freight records ('Y' or 'N').
       - `p$fgrp` (1): File group ('G' or 'Z').
     - Moves input parameters to work fields (e.g., `p$fdat` to `w$fmdy`, `p$tdat` to `w$tmdy` for display, likely in `MMDDYY` format after conversion).
     - Initializes data structures for date conversion (`d#`), freight GL account (`w1frgl`, `w$acct`, `w$sub`), routing code (`w1rtg1`, `dsrtg1`), customer name (`w1name`, `w$name`), and city (`w1scit`, `dsscit`).
     - Opens input files `FR715W`, `INLOC`, `GLCONT`, and output files `FR714W2` (likely a typo for `FR715W2`), `LIST164`, and `LIST378` (per `mg01` for Excel output).

2. **Open Database Tables**:
   - **Purpose**: Applies file overrides and opens necessary files.
   - **Actions** (inferred from `OVG`/`OVZ` arrays and file definitions):
     - Applies overrides for `INLOC` and `GLCONT` based on `p$fgrp`:
       - For `p$fgrp = 'G'`: Overrides to `GINLOC` and `GGLCONT`.
       - For `p$fgrp = 'Z'`: Overrides to `ZINLOC` and `ZGLCONT`.
     - Executes overrides using `QCMDEXC` (likely in a subroutine like `opntbl`, not shown due to truncation).
     - Opens `INLOC` and `GLCONT` for reading, `FR715W` as the primary input file, `FR714W2` (likely meant to be `FR715W2`) for output, and `LIST164`/`LIST378` as printer files.

3. **Process Work File (`FR715W`)**:
   - **Purpose**: Reads records from `FR715W` and organizes them hierarchically for reporting.
   - **Actions**:
     - Uses level-break indicators (`L1` to `L7`) defined in the input specification (`IFR715WPF`):
       - `L7`: Company (`W1CO`).
       - `L6`: Location (`W1LOC`).
       - `L5`: Product code (`W1PROD`).
       - `L4`: State (`W1SST`).
       - `L3`: City (`W1SCIT`).
       - `L2`: Customer (`W1CUST`).
       - `L1`: Order number (`W1RDNO`) and sequence number (`W1SRN`).
     - For each record read:
       - **Company Break (`*INL7 = *ON`)**:
         - Sets `PRTOVR` to `*ON` to indicate a new company section.
         - Clears company-level totals (`L7BFRT`, `L7EFRT`, `L7AFRT`, `L7DIFF`, `L7POST`, `L7NEGT`).
         - Chains to `GLCONT` using `W1CO` to retrieve the company name (`GCNAME`) into `R$CNAM`. If not found (`*IN99 = *ON`), clears `R$CNAM`.
       - **Location Break (`*INL6 = *ON`)**:
         - Sets `PRTOVR` to `*ON`.
         - Clears location-level totals (`L6BFRT`, `L6EFRT`, `L6AFRT`, `L6DIFF`, `L6POST`, `L6NEGT`).
         - Chains to `INLOC` using `KEY` (likely a key list with `W1CO` and `W1LOC`) to retrieve the location name (`ILNAME`) into `R$ILNM`. If not found, clears `R$ILNM`.
       - **Product Code Break (`*INL5 = *ON`)**:
         - Clears product-level totals (`L5BFRT`, `L5EFRT`, `L5AFRT`, `L5DIFF`, `L5POST`, `L5NEGT`).
         - (Further processing likely in truncated code.)
       - **State, City, Customer Breaks (`*INL4`, `*INL3`, `*INL2`)**:
         - Likely handles page overflow and prints subtotals (via `OVRFLO` subroutine, not shown).
         - Accumulates totals for booked freight (`W1BFRT`), estimated freight (`W1EFRT`), actual/customer freight (`W1AFRT`), and differences (`W1DIFF`) at each level.

4. **Print Report**:
   - **Purpose**: Generates the printed report using `LIST164` (and `LIST378` for Excel per `mg01`).
   - **Actions**:
     - **Headers (`HDR01`)**:
       - Prints company information (`W1CO`, `R$CNAM`), report title ("FREIGHT OUT RECONCILIATION REPORT"), and page number (`PAGE`).
       - Includes job details (`PS#JOB`, `PS#PGM`, `PS#USR`), location (`W1LOC`, `R$ILNM`), and date/time (`PS#MDY`, `PS#HMS`).
       - Displays selection criteria: location (`P$LOC`), carrier (`P$CAID`), customer (`P$CUST`), zero-dollar flag (`P$CAR$`), and date range (`W$FMDY` to `W$TMDY`).
       - Prints column headers for shipping destination, product code, state, shipment date, customer, order number, sequence number, invoice, customer number, ship-to, type, quantity, unit of measure, freight code, billed freight, expected carrier, actual carrier, difference, and close status.
     - **Detail Lines (`DETL`)**:
       - Prints fields from `FR715W`: product code (`W1PROD`), city (`DSSCIT`), state (`W1SST`), shipment date (`W$SDAT`), customer name (`W$NAME`), order number (`W1RDNO`), sequence number (`W1SRN`), invoice number (`W1INVN`), customer number (`W1CUST`), ship-to (`W1SHIP`), order type (`W1TYPE`), quantity (`W1QTY`), unit of measure (`W1UM`), freight code (`W1FRCD`), billed freight (`W1BFRT`), estimated freight (`W1EFRT`), actual freight (`W1AFRT`), difference (`W1DIFF`), and close status (`W1CLOS`).
     - **Totals**:
       - **Location Totals (`TOTL6`)**: Prints location (`W1LOC`), totals for booked (`L6BFRT`), estimated (`L6EFRT`), actual (`L6AFRT`), and difference (`L6DIFF`), plus positive (`L6POST`) and negative (`L6NEGT`) carrier charges.
       - **Company Totals (`TOTL7`)**: Prints company totals for booked (`L7BFRT`), estimated (`L7EFRT`), actual (`L7AFRT`), difference (`L7DIFF`), positive (`L7POST`), and negative (`L7NEGT`) carrier charges.
     - **Excel Output (`LIST378`)**:
       - Added per `mg01` (11/27/2017) for Excel-compatible output (378 characters wide).
       - Specific formatting is not fully visible due to truncation.

5. **Write to Secondary Work File (`FR714W2`)**:
   - **Purpose**: Outputs data to `FR714W2`, likely intended to be `FR715W2`, possibly for Excel export or further processing.
   - **Actions** (inferred):
     - Writes records to `FR714W2` (not shown in output specifications but defined as an output file).
     - Likely mirrors detail or total data for external use.

6. **Program Termination** (inferred):
   - **Purpose**: Cleans up and exits.
   - **Actions**:
     - Closes all open files (`FR715W`, `INLOC`, `GLCONT`, `FR714W2`, `LIST164`, `LIST378`).
     - Sets `*INLR` to `*ON` and returns to `FR715C`.

### Business Rules

The `FR715B` program enforces the following business rules:
1. **Hierarchical Reporting**:
   - Organizes data by company (`L7`), location (`L6`), product code (`L5`), state (`L4`), city (`L3`), customer (`L2`), and order/sequence (`L1`), with product code as the primary sort (`L5`).
2. **Reference Data Retrieval**:
   - Retrieves company name (`GCNAME`) from `GLCONT` and location name (`ILNAME`) from `INLOC`, defaulting to blank if not found.
3. **Totals Calculation**:
   - Accumulates booked, estimated, and actual freight amounts, and differences at location (`L6`) and company (`L7`) levels.
   - Tracks positive (`L6POST`, `L7POST`) and negative (`L6NEGT`, `L7NEGT`) carrier charge differences.
4. **Selection Criteria Display**:
   - Includes user-selected parameters (location, carrier, customer, date range, zero-dollar flag) in the report header.
5. **Excel Output**:
   - Supports `LIST378` for Excel-compatible output (per `mg01`), with a wider format for spreadsheet compatibility.
6. **Page Overflow Handling**:
   - Uses the `OVRFLO` subroutine (not shown) to manage page breaks.
7. **No Input Validation**:
   - Relies on `FR713P` for input validation, assuming `FR715W` data is correct.

### Database Tables Used

The `FR715B` program directly accesses:
1. **FR715W** (Freight Work File, Input, Primary):
   - Keyed by company (`W1CO`), location (`W1LOC`), product code (`W1PROD`), state (`W1SST`), city (`W1SCIT`), customer (`W1CUST`), order number (`W1RDNO`), and sequence number (`W1SRN`).
   - Fields: `W1CO`, `W1LOC`, `W1PROD`, `W1SST`, `W1SCIT`, `W1CUST`, `W1RDNO`, `W1SRN`, `W1INVN`, `W1SHIP`, `W1TYPE`, `W1QTY`, `W1UM`, `W1FRCD`, `W1BFRT`, `W1EFRT`, `W1AFRT`, `W1DIFF`, `W1CLOS`.
2. **INLOC** (Inventory Location, Input):
   - Keyed by company and location (via `KEY`).
   - Overrides: `GINLOC` or `ZINLOC`.
   - Field: `ILNAME` (location name).
3. **GLCONT** (General Ledger Control, Input):
   - Keyed by company (`W1CO`).
   - Overrides: `GGLCONT` or `ZGLCONT`.
   - Field: `GCNAME` (company name).
4. **FR714W2** (Secondary Work File, Output):
   - Likely a typo for `FR715W2`, used for additional output, possibly for Excel.
5. **LIST164** (Printer File, Output):
   - Standard report output (164 characters wide).
6. **LIST378** (Printer File, Output):
   - Excel-compatible output (378 characters wide, per `mg01`).

### External Programs Called

The `FR715B` program likely calls:
1. **QCMDEXC**:
   - System API for executing `OVRDBF` commands in the `opntbl` subroutine (not shown).
   - Parameters: `dbov##` (80 characters), `dbol##` (15.5).

### Additional Notes
- **Revisions**:
  - **mg01 (11/27/2017)**: Added `LIST378` for Excel output.
- **Truncated Code**:
  - Missing 23,439 characters, including the main processing loop and `OVRFLO` subroutine, but output specifications clarify the report structure.
- **File Naming Issue**:
  - The output file is defined as `FR714W2`, which is likely a typo for `FR715W2`, as it should align with the program’s purpose and input file (`FR715W`).
- **Data Structures**:
  - Uses `d#` for date formatting, and structures for splitting `W1FRGL`, `W1RTG1`, `W1NAME`, and `W1SCIT`.
- **Integration**:
  - Part of the chain `FR713P` → `FR713PC` → `FR715C` → `FR715A` → `FR715B`.
- **Location Code Length**:
  - `P$LOC` is LEN(3) in the output specs but LEN(2) in `FR715C`, indicating a potential inconsistency.

### Summary

The `FR715B` program reads `FR715W`, retrieves names from `INLOC` and `GLCONT`, and generates a product code-sorted Freight Out Reconciliation Report using `LIST164` (and `LIST378` for Excel). It organizes data hierarchically, prints headers, detail lines, and totals, and writes to `FR714W2` (likely `FR715W2`). It uses six files and likely calls `QCMDEXC` for overrides, enforcing rules for hierarchical reporting and totals calculation.