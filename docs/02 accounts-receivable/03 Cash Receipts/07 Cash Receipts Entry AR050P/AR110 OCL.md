The `AR110.ocl36` script is an OCL (Operation Control Language) procedure for the IBM System/36 environment, designed to handle the edit and validation process for individual cash receipts entries created by the `AR100` program. It is invoked from the main OCL script (`AR050P.ocl36`) when the user selects option `2` at memory location `100,1` and individual entries need validation before further processing. This script prepares the environment, sorts transaction records, and runs the `AR110` program to edit and validate entries, ensuring data integrity for posting. The script also incorporates an Atrium environment adaptation (replacing `?WS?` with `GG`, per JB01 comment dated 11/28/23). Below, I outline the process steps, business rules, tables (files) used, and external programs called.

### Process Steps of the AR110 OCL Program

1. **Year 2000 Compliance**:
   - `// GSY2K`: Executes the `GSY2K` utility to ensure Year 2000 compliance for date-related fields in the cash receipts process. This prevents errors in date validation or storage (e.g., ensuring 4-digit years).

2. **Check for Transaction File Existence**:
   - `// IF ?F'A,?9?CRIEGG'?/00000000 GOTO END`: Checks if the individual entry transaction file (`?9?CRIEGG`, created by `AR100`) exists and contains records (non-zero record count). If empty (`/00000000`), the script skips to the `END` tag, bypassing all processing, as there are no transactions to edit.

3. **Delete Invalid Records**:
   - `// LOAD $DELET`: Loads the `$DELET` system utility to remove invalid or marked-for-deletion records from the transaction file.
   - `// RUN`: Executes `$DELET` to clean up `?9?CRIEGG`.
   - `// IF DATAF1-?9?CR10GG SCRATCH UNIT-F1,LABEL-?9?CR10GG`: If the sorted transaction file (`?9?CR10GG`) exists, deletes it using `SCRATCH` to ensure a fresh file for sorting in the next step.
   - `// END`: Completes the deletion step.

4. **Sort Transaction File**:
   - `// LOAD #GSORT`: Loads the `#GSORT` system utility for sorting the transaction file.
   - `// FILE NAME-INPUT,LABEL-?9?CRIEGG`: Specifies `?9?CRIEGG` as the input file.
   - `// FILE NAME-OUTPUT,LABEL-?9?CR10GG,RECORDS-999000,EXTEND-999000`: Defines `?9?CR10GG` as the output file with a large capacity (999,000 records, extendable by another 999,000).
   - `// RUN`: Executes the sort with the following specifications:
     - `HSORTA 8A 3 N`: High-speed sort, ascending (`A`), using 8 sort fields, with 3 levels, no sequence checking (`N`).
     - `I C 1 1NECD`: Includes records where position 1 does not equal `'D'` (excludes deleted records, `atdel <> 'D'`).
     - `IAC 31 31EQCP`: Includes records where position 31 equals `'P'` (payment transactions only, `attype = 'P'`).
     - `FNC 7 8 CO NO`: Primary sort field: company number (`atco`, positions 7-8).
     - `FNC 2 6 SEQ NO`: Secondary sort field: sequence number (`seq`, positions 2-6).
   - `// END`: Completes the sort, producing a sorted file (`?9?CR10GG`) for editing.

5. **Check for Sorted File Existence**:
   - `// IF ?F'A,?9?CR10GG'?/00000000 GOTO END`: Skips to the `END` tag if the sorted file is empty, as no records are available for editing.

6. **Build Check Work File**:
   - `// IFF DATAF1-?9?CRICGG BLDFILE ?9?CRICGG,A,RECORDS,1000,96,,P,2,14`: If the check work file (`?9?CRICGG`) exists, rebuilds it:
     - `A`: Add mode.
     - `RECORDS,1000`: Allocates 1,000 records.
     - `96`: Record length of 96 bytes.
     - `,,P`: Packed decimal format.
     - `2,14`: Primary key starts at position 2, length 14 (likely customer + check number).
   - This ensures a clean work file for storing check-related data during editing.

7. **Load and Run Edit Program**:
   - `// LOAD AR110`: Loads the `AR110` RPG program, which performs the edit/validation of individual cash receipts.
   - Defines files for `AR110`, most in shared mode (`DISP-SHR`):
     - `CR110S,LABEL-?9?CR10GG`: Sorted transaction file.
     - `CRTRAN,LABEL-?9?CRIEGG`: Original transaction file.
     - `ARDETL,LABEL-?9?ARDETL,DISP-SHR`: AR detail file (invoices).
     - `ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Customer master file.
     - `GLMAST,LABEL-?9?GLMAST,DISP-SHR`: General Ledger master file.
     - `ARCONT,LABEL-?9?ARCONT,DISP-SHR`: AR control file.
     - `GSTABL,LABEL-?9?GSTABL,DISP-SHR`: General system table.
     - `CRCHKS,LABEL-?9?CRICGG,DISP-SHRMM`: Check work file (shared with multiple members).
     - `SA5FIND,LABEL-?9?SA5FIND,DISP-SHRMM`: Sales file (index).
     - `SA5FINM,LABEL-?9?SA5FINM,DISP-SHRMM`: Sales master file.
     - `SA5SHX,LABEL-?9?SA5SHX,DISP-SHRMM`: Sales index file.
     - `XGSTABL,LABEL-?9?GSTABL,DISP-SHRMM`: Additional system table alias.
     - `GSPROD,LABEL-?9?GSPROD,DISP-SHR`: Product file (for called program `AR1101`).
   - Defines printer outputs:
     - `NODLIST,FORMSNO-NOD,PRIORITY-0,DEVICE-PJ`: For edit listing.
     - `NODPRINT,FORMSNO-NOD,CONTINUE-YES,PRIORITY-0,DEVICE-PJ`: For additional printing (used by `AR1101`).
   - Overrides printer output queue:
     - `// IF ?9?/G OVRPRTF FILE(PRINT) OUTQ(QUSRSYS/AREDIT)`: If `?9?` is `G`, redirects output to `AREDIT` queue.
     - `// IFF ?9?/G OVRPRTF FILE(PRINT) OUTQ(QUSRSYS/TESTOUTQ)`: Otherwise, uses `TESTOUTQ`.
   - `// RUN`: Executes `AR110` to validate transactions and produce edit reports.

8. **Cleanup**:
   - `// IF DATAF1-?9?CR10GG DELETE ?9?CR10GG,F1`: Deletes the sorted transaction file after processing.
   - `// IF DATAF1-?9?CRICGG DELETE ?9?CRICGG,F1`: Deletes the check work file after processing.
   - This ensures temporary files are removed to free disk space.

9. **Switch Reset**:
   - `// SWITCH 0XXXXXXX`: Resets the first bit of the switch register to `0`, leaving other bits unchanged, to restore the system state for subsequent processes.

10. **End Processing**:
    - `// TAG END`: Marks the end of the script, where processing jumps if no transactions or sorted records exist.

### Business Rules

The `AR110.ocl36` script enforces the following business rules for editing individual cash receipts:

1. **Conditional Execution**:
   - Only runs if the transaction file (`?9?CRIEGG`) contains records, skipping to `END` if empty to avoid unnecessary processing.
   - Invoked from `AR050P.ocl36` when the user selects option `2` (individual entry mode).

2. **Atrium Compatibility**:
   - Uses `GG` suffix (per JB01, 11/28/23) for file names (`CRIEGG`, `CR10GG`, `CRICGG`) to ensure compatibility with the Atrium environment, preventing file conflicts in multi-user or virtualized setups.

3. **Data Integrity and Cleanup**:
   - Deletes invalid or marked records (`atdel = 'D'`) using `$DELET` before sorting to ensure only valid transactions are processed.
   - Sorts transactions by company number (`atco`) and sequence number (`seq`), including only payment transactions (`attype = 'P'`) and non-deleted records (`atdel <> 'D'`).
   - Deletes temporary files (`CR10GG`, `CRICGG`) post-processing to manage disk resources.

4. **File and Printer Sharing**:
   - Most files use `DISP-SHR` or `DISP-SHRMM` (shared, multiple members) to support concurrent access in a multi-user environment, critical for AR and GL master files.
   - Printer outputs (`NODLIST`, `NODPRINT`) are configured with priority `0` and device `PJ`, with conditional output queue overrides (`AREDIT` or `TESTOUTQ`) based on the `?9?` value.

5. **Y2K Compliance**:
   - Mandatory `GSY2K` execution ensures date fields (e.g., transaction dates) are correctly processed, avoiding millennium-related errors.

6. **File Capacity and Structure**:
   - `CRICGG` work file: 1,000 records, 96 bytes, 14-byte key (positions 2-15), packed decimal format.
   - `CR10GG` sorted file: Large capacity (999,000 records, extendable) to handle high transaction volumes.
   - `CRIEGG` transaction file: Assumed pre-built by `AR100` with 256-byte records.

7. **Support for Called Program**:
   - Includes files (`GSPROD`, `NODPRINT`) specifically for `AR1101`, a program called by `AR110`, indicating additional processing (e.g., posting or reporting).

### Tables (Files) Used

The script defines or interacts with the following files, all prefixed with `?9?` for session-specific libraries:
1. **?9?CRIEGG** (aliased as `CRTRAN`):
   - Type: Input transaction file (from `AR100`).
   - Usage: Contains raw individual cash receipts for editing.
2. **?9?CR10GG** (aliased as `CR110S`):
   - Type: Sorted transaction file (output of `#GSORT`).
   - Record Capacity: 999,000, extendable by 999,000.
   - Usage: Provides sorted records (by company, sequence) for validation.
3. **?9?CRICGG** (aliased as `CRCHKS`):
   - Type: Check work file.
   - Record Length: 96 bytes, 1,000 records.
   - Key: 14-byte key (positions 2-15).
   - Usage: Stores check-related data during editing.
4. **?9?ARDETL** (aliased as `ARDETL`):
   - Type: AR detail file (shared).
   - Usage: Validates invoice details.
5. **?9?ARCUST** (aliased as `ARCUST`):
   - Type: Customer master file (shared).
   - Usage: Validates customer data.
6. **?9?GLMAST** (aliased as `GLMAST`):
   - Type: General Ledger master file (shared).
   - Usage: Validates GL accounts.
7. **?9?ARCONT** (aliased as `ARCONT`):
   - Type: AR control file (shared).
   - Usage: Provides defaults and control data.
8. **?9?GSTABL** (aliased as `GSTABL`, `XGSTABL`):
   - Type: General system table (shared).
   - Usage: Configuration/reference data.
9. **?9?SA5FIND** (aliased as `SA5FIND`):
   - Type: Sales index file (shared, multiple members).
   - Usage: Likely for sales data lookup.
10. **?9?SA5FINM** (aliased as `SA5FINM`):
    - Type: Sales master file (shared, multiple members).
    - Usage: Sales transaction details.
11. **?9?SA5SHX** (aliased as `SA5SHX`):
    - Type: Sales index file (shared, multiple members).
    - Usage: Additional sales indexing.
12. **?9?GSPROD** (aliased as `GSPROD`):
    - Type: Product file (shared).
    - Usage: Used by called program `AR1101` for product-related data.

### External Programs Called

The script explicitly calls or loads:
1. **GSY2K**:
   - System utility for Y2K date compliance.
   - Purpose: Ensures correct date handling.
2. **$DELET**:
   - System utility to remove invalid/deleted records from `CRIEGG`.
   - Purpose: Cleans transaction file before sorting.
3. **#GSORT**:
   - System utility for sorting `CRIEGG` into `CR10GG`.
   - Purpose: Orders transactions by company and sequence number.
4. **AR110**:
   - Main RPG program for editing/validating cash receipts.
   - Purpose: Processes sorted transactions, validates against master files, and produces edit reports.
5. **AR1101**:
   - Called by `AR110` (implied by `GSPROD` and `NODPRINT` definitions).
   - Purpose: Likely handles posting or additional reporting (e.g., to general ledger or sales).

### Summary

The `AR110.ocl36` script is a critical step in the individual cash receipts workflow, ensuring transactions entered via `AR100` are validated before posting. It:
- Executes Y2K compliance and cleans up invalid records.
- Sorts transactions by company and sequence, filtering out deleted or non-payment records.
- Builds a check work file and runs `AR110` with access to AR, GL, and sales files.
- Produces edit reports and cleans up temporary files post-processing.
- Adapts to the Atrium environment with `GG` suffixes and supports multi-user access via shared files.
