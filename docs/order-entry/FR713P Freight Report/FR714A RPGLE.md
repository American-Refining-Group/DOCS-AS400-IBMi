The `FR714A` program is an RPGLE program called by `FR714C` to build the work file `FR714W` for the Freight Out Reconciliation Report sorted by carrier/routing. It processes freight data from multiple input files, applies filtering based on input parameters, calculates freight amounts, and writes the results to the work file. The program is nearly identical to `FR713A`, with the primary difference being the output work file (`FR714W` instead of `FR713W`) and the focus on carrier/routing sorting. Below, I’ll explain the process steps, business rules, database tables used, and external programs called, based on the provided RPGLE source code.

### Process Steps of the FR714A Program

The `FR714A` program initializes the environment, opens database files, reads freight header and detail records, applies filters, calculates freight amounts, and writes the processed data to the work file `FR714W`. Here’s a detailed breakdown of the process steps:

1. **Program Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Sets up initial conditions and processes input parameters.
   - **Actions**:
     - Receives eight input parameters via the `*ENTRY PLIST`:
       - `p$co` (2): Company code.
       - `p$fdat` (8): From date (in `CYMD` format).
       - `p$tdat` (8): To date (in `CYMD` format).
       - `p$loc` (3): Location code (Note: Matches `FR713C` but differs from `FR714C`’s LEN(2), suggesting a potential discrepancy).
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

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Purpose**: Opens the necessary database files with appropriate overrides based on the file group (`p$fgrp`).
   - **Actions**:
     - Checks if `p$fgrp` is 'G' or 'Z'.
     - Applies file overrides using the `ovg` (for 'G') or `ovz` (for 'Z') arrays, which contain `OVRDBF` commands to redirect file access to the appropriate library (e.g., `gfrbinh` for 'G' or `zfrbinh` for 'Z').
     - Executes the override commands using the `QCMDEXC` system API for four files (`frbinh`, `frbinf`, `frcinhj1`, `frbind`).
     - Opens the input files `frbinh`, `frbinf`, `frcinhj1`, and `frbind`, and the output file `fr714w`.

3. **Read Header File (`readbinh` Subroutine)**:
   - **Purpose**: Reads the freight header file (`frbinh`) and applies filters to select relevant records.
   - **Actions**:
     - Positions the file pointer at the start of `frbinh` using `SETLL` with `klkey1` (company code).
     - Reads records sequentially using `READE` with `klkey1`, continuing until end-of-file (`*in90 = *on`).
     - Applies filters based on input parameters:
       - **Carrier ID (`p$caid`)**: If non-blank and not matching `bocaid`, skips the record (`ITER`).
       - **Location (`p$loc`)**: If non-blank and not matching `boloc`, skips the record.
       - **Customer (`w$cust`)**: If non-zero and not matching `bocust`, skips the record (added by revision `jb01`).
       - **Date Range**: Ensures the shipment date (`boshd8`) is within the range `[w$fdat, w$tdat]` (logic truncated but inferred from `FR713A`).
     - For valid records:
       - Clears work fields for differences and freight amounts (`w$diffdtl`, `w$efrtdtl`, `w$cfrtdtl`, `w$bfrtdtl`, `w$bfrt`, `w$efrt`, `w$cfrt`, `w$diff`).
       - Sets `w$nodiff` to 'N' if the order is closed (`boclos = 'C'`) and the close date (`bocld8`) is within the to-date (`w$tdat`) (per `jb01`, inferred from `FR713A`).
       - Calls the `readbinf` subroutine to process related detail records.

4. **Read Invoice File (`readbinf` Subroutine, partially truncated)**:
   - **Purpose**: Reads the freight invoice file (`frbinf`) to calculate freight amounts for the header record.
   - **Actions** (inferred from `FR713A` due to truncation):
     - Clears work fields for freight totals (`w$bfrt`, `w$efrt`).
     - Positions the file pointer at the start of `frbinf` using `SETLL` with `klkey2`.
     - Reads records matching `klkey2` using `READE`, accumulating the freight amount (`bftfam`) into `w$bfrt` (booked freight).
     - Calls the `readcinh` subroutine to process invoice header records.

5. **Read Invoice Header File (`readcinh` Subroutine, truncated)**:
   - **Purpose**: Reads the invoice header file (`frcinhj1`) to retrieve additional freight data.
   - **Actions** (inferred from `FR713A` and revision `jb02`):
     - Uses `frcinhj1` (a multi-file logical file replacing `frcinh1`) to include freight billed balance headers.
     - Calculates customer freight (`w$cfrt`) and adjusts invoice amounts by subtracting freight balancing order override totals.
     - Calls the `readbind` subroutine to process detail records.

6. **Read Detail File (`readbind` Subroutine, partially truncated)**:
   - **Purpose**: Reads the freight detail file (`frbind`) to calculate prorated freight amounts and differences.
   - **Actions** (inferred from `FR713A`):
     - Clears work fields for detail-level amounts (`w$diffdtl`, `w$efrtdtl`, `w$cfrtdtl`, `w$bfrtdtl`).
     - Reads `frbind` records matching `klkey2`, calculating prorated amounts using the detail percentage (`bdpctt`):
       - `w$bfrtdtl = w$bfrt * bdpctt` (booked freight).
       - `w$efrtdtl = w$efrt * bdpctt` (estimated freight).
       - `w$cfrtdtl = w$cfrt * bdpctt` (actual/customer freight).
       - `w$diffdtl = (w$efrtdtl ≠ 0 ? w$efrtdtl - w$cfrtdtl : w$bfrtdtl - w$cfrtdtl)`.
     - Sets `w$diffdtl` to zero if `w$nodiff = 'N'` (closed order in period).
     - Calls the `writewrk` subroutine.

7. **Write to Work File (`writewrk` Subroutine)**:
   - **Purpose**: Writes a record to the output work file `fr714w`.
   - **Actions**:
     - Clears the output record (`fr714wpf`).
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
     - Writes the record to `fr714wpf` (the `fr714w` file).

8. **Program Termination**:
   - **Purpose**: Cleans up and exits.
   - **Actions**:
     - Closes all open files using `CLOSE *ALL`.
     - Sets `*INLR` to `*ON` and returns to `FR714C`.

### Business Rules

The `FR714A` program enforces the following business rules:
1. **File Group Selection**:
   - Uses `p$fgrp` ('G' or 'Z') to apply overrides to access the correct files (e.g., `gfrbinh` or `zfrbinh`).
2. **Record Filtering**:
   - Filters `frbinh` records based on:
     - Carrier ID (`p$caid`): Must match `bocaid` if specified.
     - Location (`p$loc`): Must match `boloc` if specified.
     - Customer (`w$cust`): Must match `bocust` if specified (per `jb01`).
     - Shipment date (`boshd8`): Must be within `[w$fdat, w$tdat]`.
3. **Zero Difference for Closed Orders**:
   - Sets `w$diffdtl` to zero for closed orders (`boclos = 'C'`) within the selected period (`bocld8 ≤ w$tdat`) (per `jb01`).
4. **Freight Calculations**:
   - Accumulates booked freight (`w$bfrt`) from `frbinf`.
   - Calculates prorated freight amounts using `bdpctt` from `frbind`.
   - Computes differences (`w$diffdtl`) based on estimated or booked freight vs. actual freight.
   - Adjusts invoice amounts by subtracting freight balancing order override totals (per `jb02`).
5. **Multi-File Logical File**:
   - Uses `frcinhj1` (per `jb02`) to include freight billed balance headers.
6. **Zero-Dollar Freight Records**:
   - The `p$car$` parameter determines inclusion of zero-dollar records (logic truncated).
7. **Data Integrity**:
   - Clears work fields to prevent data carryover.
   - Writes detailed records to `fr714w` for use by `FR714B`.

### Database Tables Used

The `FR714A` program directly accesses:
1. **frbinh** (Freight Billing Header, Input):
   - Keyed by company (`boco`) via `klkey1`.
   - Overrides: `gfrbinh` or `zfrbinh`.
   - Fields: `bocaid`, `boloc`, `bocust`, `boshd8`, `boclos`, `bocld8`, `boname`, `bortcd`, `bordno`, `bosrn`, `boinvn`, `botype`, `bosst`, `bortg1`, `boscit`, `boship`, `bofrcd`.
2. **frbinf** (Freight Billing Invoice, Input):
   - Keyed by company, order number, sequence number via `klkey2`.
   - Overrides: `gfrbinf` or `zfrbinf`.
   - Fields: `bftfam` (freight amount).
3. **frcinhj1** (Freight Invoice Header Join, Input):
   - Multi-file logical file (per `jb02`).
   - Overrides: `gfrcinhj1` or `zfrcinhj1`.
   - Fields: Used for actual/customer freight (not visible due to truncation).
4. **frbind** (Freight Billing Detail, Input):
   - Keyed by company, order number, sequence number via `klkey2`.
   - Overrides: `gfrbind` or `zfrbind`.
   - Fields: `bdpctt`, `bdfrgl`, `bdum`, `bdprod`, `bdctqt`, `bdseq`.
5. **fr714w** (Freight Work File, Output):
   - Stores processed records for the carrier-sorted report.
   - Fields: Same as `fr713w` (see `writewrk` subroutine).

### External Programs Called

The `FR714A` program calls:
1. **QCMDEXC**:
   - System API used in `opntbl` to execute `OVRDBF` commands.
   - Parameters: `dbov##` (80 characters), `dbol##` (15.5).

### Additional Notes
- **Revisions**:
  - **jb01 (04/29/2015)**: Added customer filtering and zeroed differences for closed orders.
  - **jb02 (02/04/2019)**: Replaced `frcinh1` with `frcinhj1` and adjusted invoice amounts.
- **Truncated Code**:
  - Missing `readbinf`, `readcinh`, and `readbind` subroutines limit full visibility, but `FR713A` provides a close template.
- **Location Code Length**:
  - `p$loc` is LEN(3) here but LEN(2) in `FR714C`, suggesting a potential inconsistency.
- **Integration**:
  - Part of the chain `FR713P` → `FR713PC` → `FR714C` → `FR714A` → `FR714B`.

### Summary

The `FR714A` program builds the `FR714W` work file by reading `frbinh`, `frbinf`, `frcinhj1`, and `frbind`, applying filters, calculating prorated freight amounts and differences, and writing records sorted by carrier/routing. It enforces rules for filtering, zeroing differences, and invoice adjustments, using five database files and the `QCMDEXC` API. The program mirrors `FR713A` but targets the carrier-sorted report.