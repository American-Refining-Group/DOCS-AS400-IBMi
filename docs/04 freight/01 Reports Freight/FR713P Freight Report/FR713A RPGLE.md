The `FR713A` program is an RPGLE program called by `FR713C` to build the work file `FR713W` for the Freight Out Reconciliation Report sorted by location. It processes freight data from multiple input files, applies filtering based on input parameters, and writes the results to the work file. Below, I’ll explain the process steps, business rules, database tables used, and external programs called, based on the provided RPGLE source code.

### Process Steps of the FR713A Program

The `FR713A` program initializes the environment, opens database files, reads freight header and detail records, applies filters, calculates freight amounts, and writes the processed data to the work file. Here’s a detailed breakdown of the process steps:

1. **Program Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Sets up initial conditions and processes input parameters.
   - **Actions**:
     - Receives eight input parameters via the `*ENTRY PLIST`:
       - `p$co` (2): Company code.
       - `p$fdat` (8): From date (in `CYMD` format).
       - `p$tdat` (8): To date (in `CYMD` format).
       - `p$loc` (3): Location code.
       - `p$caid` (6): Carrier ID.
       - `p$cust` (6): Customer code.
       - `p$car$` (1): Flag to include zero-dollar freight records ('Y' or 'N').
       - `p$fgrp` (1): File group ('G' or 'Z', determining the database file set).
     - Moves input parameters to work fields:
       - `p$co` to `k$co` (key field for company).
       - `p$fdat` to `w$fdat` (from date).
       - `p$tdat` to `w$tdat` (to date).
       - `p$cust` to `w$cust` (customer).
     - Defines key lists:
       - `klkey1`: For accessing `frbinh` (header file) by company (`k$co`).
       - `klkey2`: For accessing `frbinf` and `frbind` (detail files) by company (`boco`), order number (`bordno`), and sequence number (`bosrn`).
     - Note: A commented-out `DSPLY` operation for `p$tdat` suggests debugging code that is no longer active.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Purpose**: Opens the necessary database files with appropriate overrides based on the file group (`p$fgrp`).
   - **Actions**:
     - Checks if `p$fgrp` is 'G' or 'Z'.
     - Applies file overrides using the `ovg` (for 'G') or `ovz` (for 'Z') arrays, which contain `OVRDBF` commands to redirect file access to the appropriate library (e.g., `gfrbinh` for 'G' or `zfrbinh` for 'Z').
     - Executes the override commands using the `QCMDEXC` system API for four files (`frbinh`, `frbinf`, `frcinhj1`, `frbind`).
     - Opens the input files `frbinh`, `frbinf`, `frcinhj1`, and `frbind`, and the output file `fr713w`.

3. **Read Header File (`readbinh` Subroutine)**:
   - **Purpose**: Reads the freight header file (`frbinh`) and applies filters to select relevant records.
   - **Actions**:
     - Positions the file pointer at the start of `frbinh` using `SETLL` with `klkey1` (company code).
     - Reads records sequentially using `READE` with `klkey1`, continuing until end-of-file (`*in90 = *on`).
     - Applies filters based on input parameters:
       - **Carrier ID (`p$caid`)**: If non-blank and not matching `bocaid`, skips the record (`ITER`).
       - **Location (`p$loc`)**: If non-blank and not matching `boloc`, skips the record.
       - **Customer (`w$cust`)**: If non-zero and not matching `bocust`, skips the record (added by revision `jb01`).
       - **Date Range**: Ensures the shipment date (`boshd8`) is within the range `[w$fdat, w$tdat]`. If not, skips the record.
     - For valid records:
       - Clears work fields for differences and freight amounts (`w$diffdtl`, `w$efrtdtl`, `w$cfrtdtl`, `w$bfrtdtl`, `w$bfrt`, `w$efrt`, `w$cfrt`, `w$diff`).
       - Sets `w$nodiff` to 'N' if the order is closed (`boclos = 'C'`) and the close date (`bocld8`) is within the to-date (`w$tdat`) (added by `jb01`).
       - Calls the `readbinf` subroutine to process related detail records.

4. **Read Invoice File (`readbinf` Subroutine)**:
   - **Purpose**: Reads the freight invoice file (`frbinf`) to calculate freight amounts for the header record.
   - **Actions**:
     - Clears work fields for freight totals (`w$bfrt`, `w$efrt`).
     - Positions the file pointer at the start of `frbinf` using `SETLL` with `klkey2` (company, order number, sequence number).
     - Reads records matching `klkey2` using `READE`, continuing until no more matching records (`*in91 = *on`).
     - For each record:
       - Adds the freight amount (`bftfam`) to `w$bfrt` (total booked freight).
       - (Note: The truncated code likely includes additional calculations, such as for estimated freight `w$efrt`, based on context from later subroutines.)
     - After processing all matching records, calls the `readcinh` subroutine to process invoice header records.

5. **Read Invoice Header File (`readcinh` Subroutine, partially truncated)**:
   - **Purpose**: Reads the invoice header file (`frcinhj1`) to retrieve additional freight data.
   - **Actions**:
     - (Due to truncation, exact details are incomplete, but based on context and revision `jb02`):
       - Uses `frcinhj1` (a multi-file logical file replacing `frcinh1`) to include freight billed balance headers.
       - Likely calculates customer freight (`w$cfrt`) and adjusts invoice amounts by subtracting freight balancing order override totals (per `jb02`).
       - Calls the `readbind` subroutine to process detail records.

6. **Read Detail File (`readbind` Subroutine)**:
   - **Purpose**: Reads the freight detail file (`frbind`) to calculate prorated freight amounts and differences.
   - **Actions**:
     - Clears work fields for detail-level differences and freight amounts (`w$diffdtl`, `w$efrtdtl`, `w$cfrtdtl`, `w$bfrtdtl`).
     - Positions the file pointer at the start of `frbind` using `SETLL` with `klkey2`.
     - Reads records matching `klkey2` using `READE`, continuing until no more matching records (`*in93 = *on`).
     - For each record:
       - Calculates prorated amounts by multiplying total freight amounts (`w$bfrt`, `w$efrt`, `w$cfrt`) by the detail percentage (`bdpctt`):
         - `w$bfrtdtl = w$bfrt * bdpctt` (booked freight detail).
         - `w$efrtdtl = w$efrt * bdpctt` (estimated freight detail).
         - `w$cfrtdtl = w$cfrt * bdpctt` (actual/customer freight detail).
       - Calculates the difference (`w$diffdtl`):
         - If `w$efrtdtl ≠ 0`, `w$diffdtl = w$efrtdtl - w$cfrtdtl`.
         - Else, `w$diffdtl = w$bfrtdtl - w$cfrtdtl`.
       - If `w$nodiff = 'N'` (order closed in selected period, per `jb01`), sets `w$diffdtl` to zero.
     - Calls the `writewrk` subroutine to write the processed data to the work file.

7. **Write to Work File (`writewrk` Subroutine)**:
   - **Purpose**: Writes a record to the output work file `fr713w`.
   - **Actions**:
     - Clears the output record (`fr713wpf`).
     - Populates fields from header and detail data:
       - `w1co`: Company (`boco`).
       - `w1cust`: Customer (`bocust`).
       - `w1name`: Customer name (`boname`).
       - `w1frgl`: Freight GL account (`bdfrgl`).
       - `w1rtcd`: Routing code (`bortcd`).
       - `w1sdat`: Shipment date (`boshd8`).
       - `w1rdno`: Order number (`bordno`).
       - `w1srn`: Sequence number (`bosrn`).
       - `w1invn`: Invoice number (`boinvn`).
       - `w1type`: Order type (`botype`).
       - `w1um`: Unit of measure (`bdum`).
       - `w1prod`: Product code (`bdprod`).
       - `w1qty`: Quantity (`bdctqt`).
       - `w1clos`: Close status (`boclos`).
       - `w1bfrt`: Booked freight detail (`w$bfrtdtl`).
       - `w1efrt`: Estimated freight detail (`w$efrtdtl`).
       - `w1afrt`: Actual/customer freight detail (`w$cfrtdtl`).
       - `w1diff`: Freight difference (`w$diffdtl`).
       - `w1seq`: Sequence number (`bdseq`).
       - `tlbfrt`: Total booked freight (`w$bfrt`).
       - `tlefrt`: Total estimated freight (`w$efrt`).
       - `tlafrt`: Total actual/customer freight (`w$cfrt`).
       - `w1loc`: Location (`boloc`).
       - `w1sst`: Ship state (`bosst`).
       - `w1rtg1`: Routing group (`bortg1`).
       - `w1scit`: Ship city (`boscit`).
       - `w1ship`: Ship via (`boship`).
       - `w1frcd`: Freight code (`bofrcd`).
     - Writes the record to `fr713wpf` (the `fr713w` file).

8. **Program Termination**:
   - **Purpose**: Cleans up and exits the program.
   - **Actions**:
     - Closes all open files using `CLOSE *ALL`.
     - Sets `*INLR` to `*ON` to indicate the program is ending.
     - Returns control to the caller (`FR713C`).

### Business Rules

The `FR713A` program enforces the following business rules:
1. **File Group Selection**:
   - Uses `p$fgrp` ('G' or 'Z') to apply overrides to access the correct database files (e.g., `gfrbinh` or `zfrbinh`), ensuring data is retrieved from the appropriate library.
2. **Record Filtering**:
   - Filters `frbinh` records based on:
     - Carrier ID (`p$caid`): Must match `bocaid` if specified.
     - Location (`p$loc`): Must match `boloc` if specified.
     - Customer (`w$cust`): Must match `bocust` if specified (per `jb01`).
     - Shipment date (`boshd8`): Must be within `[w$fdat, w$tdat]`.
   - Skips records that fail these filters.
3. **Zero Difference for Closed Orders**:
   - If an order is closed (`boclos = 'C'`) and closed within the selected period (`bocld8 ≤ w$tdat`), the freight difference (`w$diffdtl`) is set to zero (per `jb01`).
4. **Freight Calculations**:
   - Accumulates booked freight (`w$bfrt`) from `frbinf` records.
   - Calculates prorated freight amounts at the detail level using `bdpctt` (percentage) from `frbind`.
   - Computes the freight difference (`w$diffdtl`) as:
     - Estimated freight (`w$efrtdtl`) minus actual/customer freight (`w$cfrtdtl`) if estimated freight is non-zero.
     - Otherwise, booked freight (`w$bfrtdtl`) minus actual/customer freight (`w$cfrtdtl`).
   - Adjusts invoice amounts by subtracting freight balancing order override totals (per `jb02`).
5. **Multi-File Logical File**:
   - Uses `frcinhj1` (replacing `frcinh1` per `jb02`) to include freight billed balance headers, enhancing data coverage without significant program changes.
6. **Zero-Dollar Freight Records**:
   - The `p$car$` parameter ('Y' or 'N') determines whether to include records with zero freight amounts, though its exact implementation is not visible in the truncated code.
7. **Data Integrity**:
   - Clears work fields before processing each header record and detail record to prevent data carryover.
   - Writes all relevant header and detail fields to `fr713w` for use by the print program (`FR713B`).

### Database Tables Used

The `FR713A` program directly accesses the following database files:
1. **frbinh** (Freight Billing Header, Input):
   - Used to read header records for freight orders.
   - Keyed by company (`boco`) via `klkey1`.
   - Overrides: `gfrbinh` (for `p$fgrp = 'G'`) or `zfrbinh` (for `p$fgrp = 'Z'`).
   - Fields used: `bocaid` (carrier ID), `boloc` (location), `bocust` (customer), `boshd8` (shipment date), `boclos` (close status), `bocld8` (close date), `boname` (customer name), `bortcd` (routing code), `bordno` (order number), `bosrn` (sequence number), `boinvn` (invoice number), `botype` (order type), `bosst` (ship state), `bortg1` (routing group), `boscit` (ship city), `boship` (ship via), `bofrcd` (freight code).
2. **frbinf** (Freight Billing Invoice, Input):
   - Used to calculate booked freight amounts for each header record.
   - Keyed by company (`boco`), order number (`bordno`), and sequence number (`bosrn`) via `klkey2`.
   - Overrides: `gfrbinf` (for `p$fgrp = 'G'`) or `zfrbinf` (for `p$fgrp = 'Z'`).
   - Fields used: `bftfam` (freight amount).
3. **frcinhj1** (Freight Invoice Header Join, Input):
   - A multi-file logical file (replacing `frcinh1` per `jb02`) that includes freight billed balance headers.
   - Keyed by company, order number, and sequence number (inferred from context).
   - Overrides: `gfrcinhj1` (for `p$fgrp = 'G'`) or `zfrcinhj1` (for `p$fgrp = 'Z'`).
   - Fields used: Not fully visible due to truncation, but likely used to calculate actual/customer freight (`w$cfrt`).
4. **frbind** (Freight Billing Detail, Input):
   - Used to calculate prorated freight amounts and differences.
   - Keyed by company (`boco`), order number (`bordno`), and sequence number (`bosrn`) via `klkey2`.
   - Overrides: `gfrbind` (for `p$fgrp = 'G'`) or `zfrbind` (for `p$fgrp = 'Z'`).
   - Fields used: `bdpctt` (percentage), `bdfrgl` (freight GL account), `bdum` (unit of measure), `bdprod` (product code), `bdctqt` (quantity), `bdseq` (sequence number).
5. **fr713w** (Freight Work File, Output):
   - The output work file where processed records are written.
   - Populated with fields from header and detail records, including calculated freight amounts and differences.
   - No overrides applied, as it is typically created in `QTEMP` by `FR713C`.

### External Programs Called

The `FR713A` program calls the following external program:
1. **QCMDEXC**:
   - System API used in the `opntbl` subroutine to execute file override commands (`OVRDBF`) for `frbinh`, `frbinf`, `frcinhj1`, and `frbind`.
   - Parameters: `dbov##` (override command string, 80 characters) and `dbol##` (command length, 15.5).

### Additional Notes
- **Revisions**:
  - **jb01 (04/29/2015)**: Added customer filtering (`w$cust`) and logic to zero out differences for closed orders within the selected period.
  - **jb02 (02/04/2019)**: Replaced `frcinh1` with `frcinhj1` to include freight billed balance headers and adjusted invoice amounts by subtracting freight balancing order override totals.
- **Truncated Code**:
  - The `readcinh` subroutine and parts of `readbinf` are truncated, limiting visibility into the full logic for calculating estimated and actual freight amounts. However, context suggests these involve processing `frcinhj1` to compute `w$cfrt` and possibly other freight-related calculations.
- **Data Structures**:
  - Uses time (`t#`) and date (`d#`) conversion data structures, though they are not explicitly used in the provided code, suggesting potential future or unused functionality.
  - Uses a program status data structure (`psds##`) to capture environment details like program name and user.
- **Indicator Usage**:
  - Indicators (e.g., 90-93 for file reads, 50-69 for errors) are used to control program flow, though no screen-related indicators (e.g., 19, 40-43) are used since this program is non-interactive.
- **Integration**:
  - `FR713A` is part of the chain (`FR713P` → `FR713PC` → `FR713C` → `FR713A` → `FR713B`) and focuses on data preparation for the location-sorted report.

### Summary

The `FR713A` program builds the `FR713W` work file by reading freight header (`frbinh`), invoice (`frbinf`), invoice header (`frcinhj1`), and detail (`frbind`) files, applying filters for carrier, location, customer, and date range, calculating prorated freight amounts and differences, and writing the results to the work file. It enforces business rules for filtering, zeroing differences for closed orders, and adjusting invoice amounts. The program directly accesses five database files and calls the `QCMDEXC` API for file overrides, with all data processing tailored for the location-sorted Freight Out Reconciliation Report.