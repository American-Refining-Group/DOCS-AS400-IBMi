The RPG program `FR710A` is designed to build the `FR710W` work file for the Freight Out Accrual Report. It is called by the `FR710C` program and processes data from multiple input files to generate records for the report based on specific business rules. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

### Process Steps of the FR710A Program

1. **Program Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Sets up the program environment and initializes variables.
   - **Steps**:
     - Receives three input parameters:
       - `p$co`: Company number (2 characters).
       - `p$rdat`: Report date (8 digits, `CCYYMMDD` format).
       - `p$fgrp`: File group ('G' or 'Z') to determine file overrides.
     - Moves `p$co` to `k$co` (key field for file access) and `p$rdat` to `w$rdat` (work field for report date).
     - Defines key lists:
       - `klkey1`: Uses `k$co` to access the `frbinh` file by company.
       - `klkey2`: Uses `boco` (company), `bordno` (order number), and `bosrn` (serial number) to access `frbinf`, `frcinhj1`, and `frbind` files.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Purpose**: Opens input files with appropriate overrides based on the file group.
   - **Steps**:
     - Checks the `p$fgrp` parameter ('G' or 'Z').
     - For 'G', applies overrides from the `ovg` array to point to files `gfrbinh`, `gfrbinf`, `gfrcinhj1`, and `gfrbind`.
     - For 'Z', applies overrides from the `ovz` array to point to files `zfrbinh`, `zfrbinf`, `zfrcinhj1`, and `zfrbind`.
     - Executes overrides using the `QCMDEXC` system API, passing the override command and length.
     - Opens the input files: `frbinh`, `frbinf`, `frcinhj1` (replacing `frcinh1` per revision `jb03`), and `frbind`.

3. **Read Header File (`readbinh` Subroutine)**:
   - **Purpose**: Reads the `frbinh` file (freight billing header) and applies selection criteria.
   - **Steps**:
     - Sets the lower limit (`setll`) for `frbinh` using `klkey1` (company number).
     - Reads records (`reade`) until end-of-file (`*in90` is `*on`).
     - Applies filtering rules (see Business Rules below):
       - Skips records where `bocoon` (customer-owned product) is 'Y' (revision `jb02`).
       - Skips records where `bocafr` (calculate freight) is 'N' and `bofrto` (freight total) is zero (revision `jb02`).
       - Includes records where:
         - Shipment date (`boshd8`) is on or before the report date (`w$rdat`) and the order is not closed (`boclos` ≠ 'C').
         - Or, shipment date is on or before the report date, the order is closed (`boclos` = 'C'), and the close date (`bocld8`) is after the report date.
     - Clears work fields (`w$diffdtl`, `w$efrtdtl`, `w$cfrtdtl`, `w$bfrtdtl`, `w$bfrt`, `w$efrt`, `w$cfrt`, `w$diff`) for each header record.
     - Sets `w$nodiff` to 'N' if the order is closed in the selected month (`boclos` = 'C' and `bocld8` ≤ `w$rdat`) to zero out differences (revision `jb01`).
     - Calls `readbinf` to process related detail records.

4. **Read Freight Billing Detail File (`readbinf` Subroutine)**:
   - **Purpose**: Aggregates freight amounts from the `frbinf` file (freight billing detail).
   - **Steps**:
     - Clears work fields `w$bfrt` (booked freight) and `w$efrt` (estimated freight).
     - Sets the lower limit for `frbinf` using `klkey2` (company, order number, serial number).
     - Reads matching records (`reade`) until end-of-file (`*in91`).
     - Accumulates `bftfam` (booked freight amount) into `w$bfrt` and `cftfam` (estimated freight amount) into `w$efrt`.
     - Calls `readcinh` to process related invoice records.

5. **Read Freight Invoice File (`readcinh` Subroutine)**:
   - **Purpose**: Aggregates invoice amounts from the `frcinhj1` file (freight invoice header, replaced `frcinh1` per revision `jb03`).
   - **Steps**:
     - Clears work fields `w$cfrt` (invoiced freight) and `w$diff` (difference).
     - Sets the lower limit for `frcinhj1` using `klkey2`.
     - Reads matching records (`reade`) until end-of-file (`*in92`).
     - Accumulates `frinam` (invoice amount) into `w$cfrt`, subtracting `frfboa` (freight balancing order override total) per revision `jb03`.
     - Calculates the difference (`w$diff`):
       - If `w$efrt` (estimated freight) is non-zero, subtracts `w$efrt` from `w$cfrt`.
       - Otherwise, subtracts `w$bfrt` (booked freight) from `w$cfrt`.
     - If `w$nodiff` is 'N' (order closed in selected month), sets `w$diff` to zero (revision `jb01`).
     - Calls `readbind` to process detail records.

6. **Read Freight Billing Detail File (`readbind` Subroutine)**:
   - **Purpose**: Processes detail records from `frbind` and calculates prorated amounts.
   - **Steps**:
     - Clears work fields `w$diffdtl`, `w$efrtdtl`, `w$cfrtdtl`, and `w$bfrtdtl`.
     - Sets the lower limit for `frbind` using `klkey2`.
     - Reads matching records (`reade`) until end-of-file (`*in93`).
     - Calculates prorated amounts using `bdpctt` (percentage):
       - `w$bfrtdtl` = `w$bfrt` × `bdpctt` (booked freight detail).
       - `w$efrtdtl` = `w$efrt` × `bdpctt` (estimated freight detail).
       - `w$cfrtdtl` = `w$cfrt` × `bdpctt` (invoiced freight detail).
     - Calculates detail-level difference (`w$diffdtl`):
       - If `w$efrtdtl` is non-zero, subtracts `w$cfrtdtl` from `w$efrtdtl`.
       - Otherwise, subtracts `w$cfrtdtl` from `w$bfrtdtl`.
     - If `w$nodiff` is 'N', sets `w$diffdtl` to zero (revision `jb01`).
     - If the order is not closed (`boclos` ≠ 'C') or is closed but `w$diffdtl` is non-zero, calls `writewrk` to write to the work file (revision `jb01`).

7. **Write to Work File (`writewrk` Subroutine)**:
   - **Purpose**: Writes processed data to the `FR710W` work file.
   - **Steps**:
     - Clears the `fr710wpf` record format.
     - Populates fields from input files and calculations:
       - `w1co` (company), `w1cust` (customer), `w1name` (name), `w1frgl` (freight GL), `w1rtcd` (route code), `w1sdat` (ship date), `w1rdno` (order number), `w1srn` (serial number), `w1invn` (invoice number), `w1type` (type), `w1um` (unit of measure), `w1prod` (product), `w1qty` (quantity), `w1clos` (close status), `w1bfrt` (`w$bfrtdtl`), `w1efrt` (`w$efrtdtl`), `w1afrt` (`w$cfrtdtl`), `w1diff` (`w$diffdtl`), `w1seq` (sequence).
       - Totals: `tlbfrt` (`w$bfrt`), `tlefrt` (`w$efrt`), `tlafrt` (`w$cfrt`).
     - Writes the record to `fr710wpf`.

8. **Program Termination**:
   - Closes all open files (`close *all`).
   - Sets `*inlr` to `*on` to signal program completion.
   - Returns control to the caller (`FR710C`).

### Business Rules
- **Record Selection (Header)**:
  - Excludes records where the product is customer-owned (`bocoon` = 'Y') (revision `jb02`).
  - Excludes records where freight calculation is not required (`bocafr` = 'N') and the freight total is zero (`bofrto` = 0) (revision `jb02`).
  - Includes records based on shipment and close status:
    - Shipped on or before the report date and not closed.
    - Shipped on or before the report date, closed, but closed after the report date.
  - Zeroes out differences (`w$diff` and `w$diffdtl`) if the order is closed in the selected month (`boclos` = 'C' and `bocld8` ≤ `w$rdat`) (revision `jb01`).
- **Freight Calculation**:
  - Aggregates booked (`bftfam`) and estimated (`cftfam`) freight from `frbinf`.
  - Aggregates invoiced freight (`frinam`) from `frcinhj1`, subtracting the freight balancing order override (`frfboa`) (revision `jb03`).
  - Calculates differences at the header level (`w$diff`) and detail level (`w$diffdtl`) based on estimated or booked freight versus invoiced freight.
  - Prorates freight amounts at the detail level using `bdpctt` (percentage).
- **File Overrides**:
  - Uses `p$fgrp` to select between 'G' (production) and 'Z' (development) file sets, ensuring flexibility across environments.
- **Work File Output**:
  - Writes only records where the order is not closed or has a non-zero difference when closed (revision `jb01`).
  - Includes detailed fields and totals for use by the report printing program (`FR710B`).
- **Multi-File Support**:
  - Replaced `frcinh1` with `frcinhj1` (a multi-file logical file) to include freight billed balance headers without significant code changes (revision `jb03`).

### Tables Used
- **Input Files**:
  - `frbinh`: Freight billing header file, used to select orders based on company, shipment date, and close status.
  - `frbinf`: Freight billing detail file, used to aggregate booked (`bftfam`) and estimated (`cftfam`) freight amounts.
  - `frcinhj1`: Freight invoice header file (replaced `frcinh1` per `jb03`), used to aggregate invoiced freight (`frinam`) and subtract balancing overrides (`frfboa`).
  - `frbind`: Freight billing detail file, used to calculate prorated freight amounts based on percentages (`bdpctt`).
- **Output File**:
  - `fr710w`: Work file (`fr710wpf` record format) in `QTEMP`, populated with processed data for the Freight Out Accrual Report.
- **File Overrides**:
  - For `p$fgrp` = 'G': `gfrbinh`, `gfrbinf`, `gfrcinhj1`, `gfrbind`.
  - For `p$fgrp` = 'Z': `zfrbinh`, `zfrbinf`, `zfrcinhj1`, `zfrbind`.

### External Programs Called
- **QCMDEXC**: System API used to execute file override commands for `frbinh`, `frbinf`, `frcinhj1`, and `frbind`.

### Additional Notes
- **Revisions**:
  - **jb01 (4/27/2015)**: Fixed record selection and added logic to zero out differences for orders closed in the selected month.
  - **jb02 (5/30/2018)**: Excluded customer-owned products (`bocoon` = 'Y') and records with no freight calculation and zero freight total.
  - **jb03 (2/4/2019)**: Replaced `frcinh1` with `frcinhj1` to include freight billed balance headers and subtracted `frfboa` from invoice amounts.
- **Error Handling**: Uses indicators (`*in90` to `*in93`) to detect end-of-file conditions but lacks explicit error handling for file access or processing failures.
- **Purpose**: The program processes complex freight data, applying business rules to filter and calculate accruals, and prepares a work file for the final report generation by `FR710B`.

This program is critical for building the data foundation for the Freight Out Accrual Report, ensuring accurate freight calculations and proper record selection based on the specified criteria.