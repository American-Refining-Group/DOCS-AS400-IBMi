The RPGLE program `FR712A` is called by `FR712C` to build the temporary work file `FR712W` for the Freight Out Reconciliation Report, specifically for the carrier-sorted version of the report. It processes data from input files, applies filtering and calculations, and writes records to the work file. The program is nearly identical to `FR711A`, with the primary difference being the output file (`FR712W` instead of `FR711W`) and the implied sorting by carrier. Below is an explanation of the process steps, business rules, tables used, and external programs called, based on the provided source code.

---

### Process Steps of the RPGLE Program (`FR712A`)

The program initializes the environment, reads and filters data from input files, performs calculations, and writes the processed data to the output work file. Here are the detailed steps:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Sets up the program environment and parameters.
   - **Actions**:
     - Receives six input parameters:
       - `p$co`: Company code (2 characters).
       - `p$rdat`: Report date (8 digits, CYMD format).
       - `p$loc`: Location code (3 characters).
       - `p$caid`: Carrier ID (6 characters).
       - `p$selmo`: Month selection (1 character, added in revision `jb02`).
       - `p$fgrp`: File group (1 character, 'G' or 'Z').
     - Moves `p$co` to `k$co` (key field for company).
     - Moves `p$rdat` to `w$rdat` (report date) and `w$rdatfr` (from report date, with day set to '01').
     - Moves `p$selmo` to `w$selmo` (month selection).
     - Defines two key lists:
       - `klkey1`: For accessing `frbinh` (uses `k$co` for company).
       - `klkey2`: For accessing `frbind` (uses `boco`, `bordno`, and `bosrn` for company, order number, and sequence number).

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Purpose**: Opens the input files with appropriate overrides based on the file group.
   - **Actions**:
     - Checks the `p$fgrp` parameter ('G' or 'Z') to apply file overrides:
       - For 'G': Overrides `frbinh`, `frbinf`, `frcinhj1`, and `frbind` to `gfrbinh`, `gfrbinf`, `gfrcinhj1`, and `gfrbind` respectively.
       - For 'Z': Overrides to `zfrbinh`, `zfrbinf`, `zfrcinhj1`, and `zfrbind`.
     - Executes the overrides using the `QCMDEXC` system program, looping through the `ovg` or `ovz` arrays (four overrides).
     - Opens the input files: `frbinh`, `frbinf`, `frcinhj1` (revised from `frcinh1` in `jb06`), and `frbind`.

3. **Read Header File (`readbinh` Subroutine)**:
   - **Purpose**: Reads and filters records from the `frbinh` file, applying selection criteria.
   - **Actions**:
     - Sets the lower limit for `frbinh` using `klkey1` (company code) and reads records with `READE` until end-of-file (`*in90 = *ON`).
     - Applies filters to exclude records:
       - If `p$caid` is non-blank and `bocaid` (carrier ID) does not match, skips the record.
       - If `p$loc` is non-blank and `boloc` (location) does not match, skips the record.
       - If `bocoon = 'Y'` (customer-owned product, revision `jb05`), skips the record.
       - If `bocafr = 'N'` (calculate freight flag) and `bofrto = 0` (freight total, revision `jb05`), skips the record.
     - Applies date and status filters based on `w$selmo` (month selection, revisions `jb01`, `jb02`, `jb04`):
       - **Shipped before the selected month start (`boshd8 < w$rdatfr`) and not closed (`boclos ≠ 'C'`)**:
         - Included if `w$selmo = *blank` or `w$selmo = 'A'`.
       - **Shipped before the selected month start, closed (`boclos = 'C'`), but closed after the report date (`bocld8 > w$rdat`)**:
         - Included if `w$selmo = *blank` or `w$selmo = 'A'`.
       - **Shipped within the selected month (`boshd8 ≥ w$rdatfr` and `boshd8 ≤ w$rdat`)**:
         - If not closed (`boclos ≠ 'C'`) and `w$selmo = 'M'`, `'O'`, `'A'`, or `*blank`, includes the record.
         - If closed (`boclos = 'C'`) and `w$selmo = 'M'`, `'C'`, or `*blank`, includes the record.
     - For matching records, chains to `frcinhj1` (revised from `frcinh1` in `jb06`) to retrieve freight invoice amounts (`ficfam`).
     - Calculates freight amounts:
       - `w$bfrt`: Freight billed amount (`bofrto`).
       - `w$efrt`: Effective freight amount (`boefr`).
       - `w$cfrt`: Invoice amount (`ficfam`) minus freight balancing order override total (`bobfot`, revision `jb06`).
     - Determines if the difference should be zeroed (`w$nodiff`):
       - Set to 'N' if closed in the selected month (`boclos = 'C'` and `bocld8` matches the selected month).
     - Calculates the difference (`w$diff`):
       - If `w$efrt ≠ 0`, then `w$diff = w$efrt - w$cfrt`.
       - Otherwise, `w$diff = w$bfrt - w$cfrt`.
       - If `w$nodiff = 'N'`, sets `w$diff = 0` (revision `jb03`).
     - Calls the `readbind` subroutine to process detail records.

4. **Read Detail File (`readbind` Subroutine)**:
   - **Purpose**: Reads and processes detail records from `frbind` for the current header record.
   - **Actions**:
     - Clears detail-level work fields (`w$diffdtl`, `w$efrtdtl`, `w$cfrtdtl`, `w$bfrtdtl`).
     - Sets the lower limit for `frbind` using `klkey2` (company, order number, sequence number) and reads records with `READE` until end-of-file (`*in93 = *ON`).
     - For each detail record:
       - Calculates prorated amounts based on `bdpctt` (percentage total):
         - `w$bfrtdtl = w$bfrt * bdpctt`.
         - `w$efrtdtl = w$efrt * bdpctt`.
         - `w$cfrtdtl = w$cfrt * bdpctt`.
       - Calculates the detail-level difference (`w$diffdtl`):
         - If `w$efrtdtl ≠ 0`, then `w$diffdtl = w$efrtdtl - w$cfrtdtl`.
         - Otherwise, `w$diffdtl = w$bfrtdtl - w$cfrtdtl`.
         - If `w$nodiff = 'N'`, sets `w$diffdtl = 0` (revision `jb03`).
       - Calls the `writewrk` subroutine to write the detail record to `fr712w`.

5. **Write to Work File (`writewrk` Subroutine)**:
   - **Purpose**: Writes processed detail records to the `fr712w` work file.
   - **Actions**:
     - Clears the output record format (`fr712wpf`).
     - Populates fields from header and detail records and calculated values:
       - `w1co`: Company (`boco`).
       - `w1cust`: Customer number (`bocust`).
       - `w1name`: Customer name (`boname`).
       - `w1frgl`: Freight general ledger (`bdfrgl`).
       - `w1rtcd`: Routing code (`bortcd`).
       - `w1sdat`: Ship date (`boshd8`).
       - `w1rdno`: Order number (`bordno`).
       - `w1srn`: Sequence number (`bosrn`).
       - `w1invn`: Invoice number (`boinvn`).
       - `w1type`: Order type (`botype`).
       - `w1um`: Unit of measure (`bdum`).
       - `w1prod`: Product (`bdprod`).
       - `w1qty`: Quantity (`bdctqt`).
       - `w1clos`: Closed status (`boclos`).
       - `w1bfrt`: Billed freight (`w$bfrtdtl`).
       - `w1efrt`: Effective freight (`w$efrtdtl`).
       - `w1afrt`: Actual freight (`w$cfrtdtl`).
       - `w1diff`: Difference (`w$diffdtl`).
       - `w1seq`: Sequence (`bdseq`).
       - `tlbfrt`: Total billed freight (`w$bfrt`).
       - `tlefrt`: Total effective freight (`w$efrt`).
       - `tlafrt`: Total actual freight (`w$cfrt`).
       - `w1loc`: Location (`boloc`).
       - `w1sst`: Ship state (`bosst`).
       - `w1rtg1`: Routing group (`bortg1`).
       - `w1scit`: Ship city (`boscit`).
       - `w1ship`: Ship number (`boship`).
       - `w1frcd`: Freight code (`bofrcd`).
     - Writes the record to `fr712w`.

6. **Program Termination**:
   - Closes all open files (`frbinh`, `frbinf`, `frcinhj1`, `frbind`, `fr712w`).
   - Sets `*inlr = *ON` to indicate the program is ending.
   - Returns control to the caller (`FR712C`).

---

### Business Rules

- **Record Selection Filters** (revisions `jb01`, `jb02`, `jb04`, `jb05`):
  - Excludes records where:
    - Carrier ID (`bocaid`) does not match `p$caid` if specified.
    - Location (`boloc`) does not match `p$loc` if specified.
    - Customer-owned product is 'Y' (`bocoon`, revision `jb05`).
    - Calculate freight is 'N' (`bocafr`) and freight total is zero (`bofrto`, revision `jb05`).
  - Includes records based on ship date (`boshd8`), closed status (`boclos`), and month selection (`w$selmo`):
    - **Previous Month (Open)**: Includes records shipped before the selected month start (`w$rdatfr`) and not closed, if `w$selmo = *blank` or `w$selmo = 'A'`.
    - **Previous Month (Closed After Report Date)**: Includes records shipped before the selected month start, closed, but closed after the report date (`bocld8 > w$rdat`), if `w$selmo = *blank` or `w$selmo = 'A'`.
    - **Selected Month**:
      - Includes open records (`boclos ≠ 'C'`) if `w$selmo = 'M'`, `'O'`, `'A'`, or `*blank`.
      - Includes closed records (`boclos = 'C'`) if `w$selmo = 'M'`, `'C'`, or `*blank`.
  - The `w$rdatfr` is set to the first day of the selected month, and `w$rdat` is the report date (typically the last day).

- **Month Selection Options** (revisions `jb02`, `jb04`):
  - `M`: Selected month only, including all (open and closed) records.
  - `O`: Selected month, open records only.
  - `C`: Selected month, closed records only.
  - `A`: All open records from previous and selected months.
  - Blank: Previous month open records and selected month all records.

- **Freight Calculations** (revision `jb06`):
  - Freight invoice amount (`w$cfrt`) is calculated as `ficfam` (from `frcinhj1`) minus the freight balancing order override total (`bobfot`).
  - Difference calculations:
    - Header level: `w$diff = w$efrt - w$cfrt` if `w$efrt ≠ 0`, otherwise `w$diff = w$bfrt - w$cfrt`.
    - Detail level: `w$diffdtl = w$efrtdtl - w$cfrtdtl` if `w$efrtdtl ≠ 0`, otherwise `w$diffdtl = w$bfrtdtl - w$cfrtdtl`.
  - If a record is closed in the selected month (`w$nodiff = 'N'`), the difference (`w$diff` and `w$diffdtl`) is set to zero (revision `jb03`).

- **File Group**:
  - The `p$fgrp` parameter ('G' or 'Z') determines which files are accessed:
    - 'G': Uses `gfrbinh`, `gfrbinf`, `gfrcinhj1`, `gfrbind`.
    - 'Z': Uses `zfrbinh`, `zfrbinf`, `zfrcinhj1`, `zfrbind`.

- **Multi-File Logical File** (revision `jb06`):
  - Replaced `frcinh1` with `frcinhj1`, a multi-file logical file, to include freight billed balance headers, enhancing data coverage without significant program changes.

- **Sorting by Carrier**:
  - The program is designed for carrier-sorted output, likely reflected in the structure or indexing of `fr712w`, though the provided code does not explicitly show carrier-specific sorting logic (possibly in the truncated `readbinh` section).

- **Error Handling**:
  - The program does not explicitly handle errors beyond file read indicators (`*in90`, `*in93`). Errors are likely managed by the calling program or system.

---

### Tables Used

- **Input Files**:
  - `frbinh`: Header file for freight billing, overridden to `gfrbinh` or `zfrbinh` based on `p$fgrp`.
  - `frbinf`: Freight billing file, overridden to `gfrbinf` or `zfrbinf`.
  - `frcinhj1`: Multi-file logical file for freight invoice data (revised from `frcinh1` in `jb06`), overridden to `gfrcinhj1` or `zfrcinhj1`.
  - `frbind`: Detail file for freight billing, overridden to `gfrbind` or `zfrbind`.

- **Output File**:
  - `fr712w`: Temporary work file in `QTEMP` (created by `FR712C`), used to store processed records for the carrier-sorted report.

---

### External Programs Called

- **QCMDEXC**: System program used to execute file override commands in the `opntbl` subroutine.

---

### Additional Notes

- **Purpose**: `FR712A` extracts, filters, and calculates freight data to populate the `fr712w` work file, which is then used by `FR712B` to generate the carrier-sorted report.
- **Comparison with `FR711A`**: The program is nearly identical to `FR711A`, with the primary difference being the output file (`fr712w` vs. `fr711w`) and the implied sorting by carrier (likely in the file structure or indexing). The logic for filtering, calculations, and writing is the same.
- **Multi-User Support**: The use of `QTEMP/fr712w` (set up by `FR712C`) ensures multi-user capability by providing a job-specific work file.
- **Parameter Validation**: Assumes input parameters are valid, as they are validated by `FR711P` and passed through `FR712C`.
- **Execution Context**: Runs in the same job session as its caller, processing data sequentially and writing to the work file.

This program efficiently processes freight data, applying complex filtering and calculations to prepare the work file for the carrier-sorted Freight Out Reconciliation Report.