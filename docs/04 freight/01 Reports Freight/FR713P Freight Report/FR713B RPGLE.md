The `FR713B` program is an RPGLE program called by `FR713C` to print the Freight Out Reconciliation Report sorted by location. It reads data from the work file `FR713W`, retrieves additional information from reference files, and generates a printed report with headers, detail lines, and totals. It also supports an additional output for Excel creation (per revision `mg01`). Below, I’ll explain the process steps, business rules, database tables used, and external programs called, based on the provided RPGLE source code (noting that the code is truncated, limiting visibility into some subroutines).

### Process Steps of the FR713B Program

The `FR713B` program processes the work file `FR713W`, organizes data hierarchically by company, location, state, city, and routing code, and produces a formatted report with totals at various levels. Here’s a detailed breakdown of the process steps based on the available code:

1. **Program Initialization (`*inzsr` Subroutine, partially truncated)**:
   - **Purpose**: Sets up initial conditions and processes input parameters.
   - **Actions** (inferred from context and file definitions):
     - Receives nine input parameters (as passed from `FR713C`):
       - `p$co` (2): Company code.
       - `p$fdat` (8): From date (in `CYMD` format).
       - `p$tdat` (8): To date (in `CYMD` format).
       - `p$loc` (3): Location code.
       - `p$caid` (6): Carrier ID.
       - `p$sort` (1): Sort option ('L' for location, as this program is called by `FR713C`).
       - `p$cust` (6): Customer code.
       - `p$car$` (1): Flag to include zero-dollar freight records ('Y' or 'N').
       - `p$fgrp` (1): File group ('G' or 'Z').
     - Moves input parameters to work fields (e.g., `p$fdat` to `w$fmdy`, `p$tdat` to `w$tmdy` for display, likely in `MMDDYY` format after conversion).
     - Initializes data structures for date conversion (`d#`), freight GL account (`w1frgl`, `w$acct`, `w$sub`), routing code (`w1rtg1`, `dsrtg1`), and customer name (`w1name`, `w$name`).
     - Opens input files `FR713W`, `INLOC`, `GLCONT`, and output files `FR714W2`, `LIST164`, and `LIST378` (per `mg01` for Excel output).

2. **Open Database Tables**:
   - **Purpose**: Applies file overrides and opens necessary files.
   - **Actions** (inferred from file definitions and `OVG`/`OVZ` arrays):
     - Applies overrides for `INLOC` and `GLCONT` based on `p$fgrp`:
       - For `p$fgrp = 'G'`: Overrides to `GINLOC` and `GGLCONT`.
       - For `p$fgrp = 'Z'`: Overrides to `ZINLOC` and `ZGLCONT`.
     - Executes overrides using `QCMDEXC` (likely in a subroutine like `opntbl`, not shown due to truncation).
     - Opens `INLOC` and `GLCONT` for reading, `FR713W` as the primary input file, `FR714W2` for output, and `LIST164`/`LIST378` as printer files.

3. **Process Work File (`FR713W`)**:
   - **Purpose**: Reads records from `FR713W` and organizes them hierarchically for reporting.
   - **Actions**:
     - Uses level-break indicators (`L1` to `L7`) defined in the input specification (`IFR713WPF`):
       - `L7`: Company (`W1CO`).
       - `L6`: Location (`W1LOC`).
       - `L5`: State (`W1SST`).
       - `L4`: City (`W1SCIT`).
       - `L3`: Routing code (`W1RTG1`).
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
       - **State Break (`*INL5 = *ON`)**:
         - Clears state-level totals (`L5BFRT`, `L5EFRT`, `L5AFRT`, `L5DIFF`, `L5POST`, `L5NEGT`).
         - (Commented `PRTOVR` suggests it may not trigger a new section, possibly intentional for formatting.)
       - **City Break (`*INL4 = *ON`)**:
         - Calls the `OVRFLO` subroutine to handle page overflow (not shown due to truncation).
         - Prints a city-level detail line (`EXCEPT L4DTL`).
         - Clears city-level totals (`L4BFRT`, `L4EFRT`, `L4AFRT`, `L4DIFF`, `L4POST`, `L4NEGT`).
       - **Routing Code Break (`*INL3 = *ON`)**:
         - Clears routing-level totals (`L3BFRT`, `L3EFRT`, `L3AFRT`, `L3DIFF`, `L3POST`, `L3NEGT`).
         - (Further processing likely in truncated code.)
     - Accumulates totals for booked freight (`W1BFRT`), estimated freight (`W1EFRT`), actual/customer freight (`W1AFRT`), and differences (`W1DIFF`) at each level.

4. **Print Report**:
   - **Purpose**: Generates the printed report using the `LIST164` printer file (and `LIST378` for Excel per `mg01`).
   - **Actions**:
     - **Headers (`HDR01`)**:
       - Prints company information (`W1CO`, `R$CNAM`), report title ("FREIGHT OUT RECONCILIATION REPORT"), and page number (`PAGE`).
       - Includes job details (`PS#JOB`, `PS#PGM`, `PS#USR`), location (`W1LOC`, `R$ILNM`), and date/time (`PS#MDY`, `PS#HMS`).
       - Displays selection criteria: location (`P$LOC`), carrier (`P$CAID`), customer (`P$CUST`), zero-dollar flag (`P$CAR$`), and date range (`W$FMDY` to `W$TMDY`).
       - Prints column headers for routing, product, destination, state, shipment date, customer, order number, sequence number, invoice, customer number, ship-to, type, quantity, unit of measure, freight code, billed freight, expected carrier, actual carrier, difference, and close status.
     - **Detail Lines (`DETL`)**:
       - Prints fields from `FR713W`: routing (`DSRTG1`), product (`W1PROD`), city (`W1SCIT`), state (`W1SST`), shipment date (`W$SDAT`), customer name (`W$NAME`), order number (`W1RDNO`), sequence number (`W1SRN`), invoice number (`W1INVN`), customer number (`W1CUST`), ship-to (`W1SHIP`), order type (`W1TYPE`), quantity (`W1QTY`), unit of measure (`W1UM`), freight code (`W1FRCD`), billed freight (`W1BFRT`), estimated freight (`W1EFRT`), actual freight (`W1AFRT`), difference (`W1DIFF`), and close status (`W1CLOS`).
     - **Totals**:
       - **Location Totals (`TOTL6`)**: Prints location (`W1LOC`), totals for booked (`L6BFRT`), estimated (`L6EFRT`), actual (`L6AFRT`), and difference (`L6DIFF`), plus positive (`L6POST`) and negative (`L6NEGT`) carrier charges.
       - **Company Totals (`TOTL7`)**: Prints company totals for booked (`L7BFRT`), estimated (`L7EFRT`), actual (`L7AFRT`), difference (`L7DIFF`), positive (`L7POST`), and negative (`L7NEGT`) carrier charges.
     - **Excel Output (`LIST378`)**:
       - Added per `mg01` (11/27/2017) to support Excel creation, likely with a wider format (378 characters vs. 164 for `LIST164`).
       - Specific formatting or output logic is not visible due to truncation.

5. **Write to Secondary Work File (`FR714W2`)**:
   - **Purpose**: Outputs data to `FR714W2`, possibly for further processing or Excel export.
   - **Actions** (inferred):
     - Writes records to `FR714W2` (not shown in the output specifications but defined as an output file).
     - Likely mirrors the detail or total data for compatibility with external tools.

6. **Program Termination** (inferred):
   - **Purpose**: Cleans up and exits.
   - **Actions**:
     - Closes all open files (`FR713W`, `INLOC`, `GLCONT`, `FR714W2`, `LIST164`, `LIST378`).
     - Sets `*INLR` to `*ON` and returns control to `FR713C`.

### Business Rules

The `FR713B` program enforces the following business rules:
1. **Hierarchical Reporting**:
   - Organizes data by company (`L7`), location (`L6`), state (`L5`), city (`L4`), routing code (`L3`), customer (`L2`), and order/sequence (`L1`), using level-break logic to print subtotals and reset accumulators.
2. **Reference Data Retrieval**:
   - Retrieves company name (`GCNAME`) from `GLCONT` and location name (`ILNAME`) from `INLOC` to enrich the report, defaulting to blank if not found.
3. **Totals Calculation**:
   - Accumulates booked, estimated, and actual freight amounts, and differences at location (`L6`) and company (`L7`) levels.
   - Tracks positive (`L6POST`, `L7POST`) and negative (`L6NEGT`, `L7NEGT`) carrier charge differences for reporting.
4. **Selection Criteria Display**:
   - Includes user-selected parameters (location, carrier, customer, date range, zero-dollar flag) in the report header for clarity.
5. **Excel Output**:
   - Supports a separate printer file (`LIST378`) for Excel-compatible output (per `mg01`), likely with a wider format to accommodate spreadsheet columns.
6. **Page Overflow Handling**:
   - Uses the `OVRFLO` subroutine (not shown) to manage page breaks, triggered at the city level (`L4DTL`).
7. **No Input Validation**:
   - Relies on `FR713P` for input validation, assuming all data in `FR713W` is correct.

### Database Tables Used

The `FR713B` program directly accesses the following files:
1. **FR713W** (Freight Work File, Input, Primary):
   - Primary input file containing processed freight data from `FR713A`.
   - Keyed by company (`W1CO`), location (`W1LOC`), state (`W1SST`), city (`W1SCIT`), routing code (`W1RTG1`), customer (`W1CUST`), order number (`W1RDNO`), and sequence number (`W1SRN`).
   - Fields used: `W1CO`, `W1LOC`, `W1SST`, `W1SCIT`, `W1RTG1`, `W1CUST`, `W1RDNO`, `W1SRN`, `W1INVN`, `W1SHIP`, `W1TYPE`, `W1QTY`, `W1UM`, `W1FRCD`, `W1BFRT`, `W1EFRT`, `W1AFRT`, `W1DIFF`, `W1CLOS`, `W1PROD`.
2. **INLOC** (Inventory Location, Input):
   - Used to retrieve location name (`ILNAME`) for `W1LOC`.
   - Keyed by company and location (via `KEY`, not shown but inferred).
   - Overrides: `GINLOC` (for `p$fgrp = 'G'`) or `ZINLOC` (for `p$fgrp = 'Z'`).
3. **GLCONT** (General Ledger Control, Input):
   - Used to retrieve company name (`GCNAME`) for `W1CO`.
   - Keyed by company (`W1CO`).
   - Overrides: `GGLCONT` (for `p$fgrp = 'G'`) or `ZGLCONT` (for `p$fgrp = 'Z'`).
4. **FR714W2** (Secondary Work File, Output):
   - Used to write data, possibly for Excel export or further processing.
   - Specific fields written are not visible due to truncation.
5. **LIST164** (Printer File, Output):
   - Primary printer file (164 characters wide) for the standard report.
   - Outputs headers (`HDR01`), detail lines (`DETL`), and totals (`TOTL6`, `TOTL7`).
6. **LIST378** (Printer File, Output):
   - Added per `mg01` for Excel-compatible output (378 characters wide).
   - Specific output format is not visible due to truncation.

### External Programs Called

The `FR713B` program likely calls the following external program (inferred from context):
1. **QCMDEXC**:
   - System API used to execute file override commands (`OVRDBF`) for `INLOC` and `GLCONT` in the `opntbl` subroutine (not shown due to truncation).
   - Parameters: `dbov##` (override command string, 80 characters) and `dbol##` (command length, 15.5).

### Additional Notes
- **Revisions**:
  - **mg01 (11/27/2017)**: Added `LIST378` printer file for Excel-compatible output, expanding the report’s utility.
- **Truncated Code**:
  - The code is truncated (18,656 characters missing), omitting key subroutines like `OVRFLO`, the main processing loop, and parts of the output logic for `FR714W2` and `LIST378`. However, the provided output specifications and level-break logic provide insight into the report structure.
- **Data Structures**:
  - Uses a date conversion data structure (`d#`) to format dates (`W$FMDY`, `W$TMDY`, `W$SDAT`) in `MMDDYY` format.
  - Uses data structures to split `W1FRGL` (freight GL account) into account (`W$ACCT`) and subaccount (`W$SUB`), and to trim `W1RTG1` (routing code) and `W1NAME` (customer name) for display.
- **Printer Files**:
  - `LIST164` is the standard report output (164 characters wide).
  - `LIST378` supports a wider format for Excel, suggesting a more detailed or column-rich output.
- **Integration**:
  - Fits into the chain (`FR713P` → `FR713PC` → `FR713C` → `FR713A` → `FR713B`) to produce the location-sorted report.
- **Location Code Length**:
  - The `p$loc` parameter is defined as LEN(3), consistent with `FR713C`, but differs from `FR714C` and `FR715C` (LEN(2)), suggesting a potential system-specific discrepancy.

### Summary

The `FR713B` program reads the `FR713W` work file, retrieves company and location names from `GLCONT` and `INLOC`, and generates a Freight Out Reconciliation Report sorted by location using `LIST164` (and `LIST378` for Excel). It organizes data hierarchically, prints headers with selection criteria, detail lines, and totals at location and company levels, and writes to `FR714W2` for additional processing. The program enforces rules for hierarchical reporting, totals calculation, and reference data retrieval, relying on `FR713P` for input validation. It uses five files and likely calls `QCMDEXC` for file overrides.