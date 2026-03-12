The `AR105.ocl36` script is an OCL (Operation Control Language) procedure for the IBM System/36 environment, designed to handle the edit and validation of payment statement entries created by the `AR050` program in the accounts receivable (AR) system. It is conditionally invoked from the main OCL script (`AR050P.ocl36`) when the user selects option `1` at memory location `100,1` (indicating payment statement mode). The script ensures that payment and adjustment transactions are validated before posting, producing edit reports and Notices of Difference (NODs) as needed. It incorporates an Atrium environment adaptation by replacing `?WS?` with `GG` (per JB01 comment, 11/28/23). Below, I outline the process steps, business rules, tables (files) used, and external programs called, leveraging context from the provided documents and prior analyses.

### Process Steps of the AR105 OCL Program

1. **Year 2000 Compliance**:
   - `// GSY2K`: Executes the `GSY2K` utility to ensure Year 2000 compliance for date fields in payment and adjustment transactions, preventing errors in date validation or storage (e.g., ensuring 4-digit years).

2. **Check for Input Files**:
   - `// IF ?F'A,?9?CRCKGG'?/00000000 GOTO END`: Checks if the check work file (`?9?CRCKGG`, created by `AR050`) contains records. If empty, skips to the `END` tag.
   - `// IF ?F'A,?9?CRWKGG'?/00000000 GOTO END`: Checks if the invoice work file (`?9?CRWKGG`, also from `AR050`) contains records. If empty, skips to `END`. This ensures processing only occurs if both files have data.

3. **Delete Temporary Files**:
   - `// IF DATAF1-?9?CRPSGG DELETE ?9?CRPSGG,F1`: Deletes the payment statement transaction file if it exists to ensure a clean file for output.
   - `// IF DATAF1-?9?CR10GG DELETE ?9?CR10GG,F1`: Deletes the sorted payment transaction file.
   - `// IF DATAF1-?9?AR11GG DELETE ?9?AR11GG,F1`: Deletes the sorted adjustment transaction file. These deletions prevent stale data from prior runs.

4. **Edit Payment Statement Entries**:
   - `// LOAD AR105`: Loads the `AR105` program to perform initial validation of payment statement entries.
   - Defines files:
     - `CRWORK,LABEL-?9?CRWKGG`: Invoice work file (open invoices).
     - `CRCHKS,LABEL-?9?CRCKGG`: Check work file.
     - `ARCONT,LABEL-?9?ARCONT,DISP-SHR`: AR control file (shared).
     - `ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Customer master file (shared).
     - `ARDETL,LABEL-?9?ARDETL,DISP-SHR`: AR detail file (shared).
     - `CRTRAN,LABEL-?9?CRPSGG,RECORDS-999000,EXTEND-500`: Output transaction file with large capacity and 500-record extension.
   - `// RUN`: Executes `AR105` to validate payment statements, writing validated or errored transactions to `CRPSGG`.

5. **Sort Payment Transactions**:
   - `// LOAD #GSORT`: Loads the `#GSORT` system utility to sort payment transactions.
   - `// FILE NAME-INPUT,LABEL-?9?CRPSGG`: Input is the transaction file from `AR105`.
   - `// FILE NAME-OUTPUT,LABEL-?9?CR10GG,RECORDS-999000,EXTEND-999000`: Output is the sorted payment file.
   - `// RUN` with sort specifications:
     - `HSORTA 15A 3 N`: High-speed sort, ascending, 15 fields, 3 levels, no sequence checking.
     - `I C 1 1NECD`: Includes records where position 1 is not `'D'` (excludes deleted records).
     - `IAC 31 31EQCP`: Includes records where position 31 is `'P'` (payment transactions only).
     - `FNC 7 21 CO/CUST/INV`: Sorts by company/customer/invoice (positions 7-21).
   - `// END`: Produces sorted file `CR10GG` for payment validation.

6. **Validate Sorted Payments**:
   - `// IF ?F'A,?9?CR10GG'?/00000000 GOTO ADJUST`: Skips to adjustment processing if no payment transactions exist.
   - `// LOAD AR110`: Loads the `AR110` program to validate sorted payment transactions.
   - Defines files (mostly shared, `DISP-SHR` or `DISP-SHRMM`):
     - `CR110S,LABEL-?9?CR10GG`: Sorted payment transactions.
     - `CRTRAN,LABEL-?9?CRPSGG`: Original transaction file.
     - `ARDETL,LABEL-?9?ARDETL,DISP-SHR`: AR detail file.
     - `ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Customer master file.
     - `GLMAST,LABEL-?9?GLMAST,DISP-SHR`: General Ledger master file.
     - `ARCONT,LABEL-?9?ARCONT,DISP-SHR`: AR control file.
     - `GSTABL,LABEL-?9?GSTABL,DISP-SHR`: General system table.
     - `CRCHKS,LABEL-?9?CRCKGG,DISP-SHRMM`: Check work file.
     - `SA5FIND,LABEL-?9?SA5FIND,DISP-SHRMM`: Sales index file.
     - `SA5FINM,LABEL-?9?SA5FINM,DISP-SHRMM`: Sales master file.
     - `SA5SHX,LABEL-?9?SA5SHX,DISP-SHRMM`: Sales index file.
     - `XGSTABL,LABEL-?9?GSTABL,DISP-SHRMM`: System table alias.
     - Printer files: `NODLIST` and `NODPRINT` for edit and NOD reports.
   - Printer overrides:
     - `// IF ?9?/G OVRPRTF FILE(PRINT) OUTQ(QUSRSYS/AREDIT)`: Uses `AREDIT` queue if `?9?` is `G`.
     - `// IFF ?9?/G OVRPRTF FILE(PRINT) OUTQ(QUSRSYS/TESTOUTQ)`: Uses `TESTOUTQ` otherwise.
   - `// RUN`: Executes `AR110` to validate payments, flag errors, and produce NOD reports for discrepancies.

7. **Sort Adjustment Transactions**:
   - `// TAG ADJUST`: Marks the adjustment processing section.
   - `// LOAD #GSORT`: Loads `#GSORT` for sorting adjustment transactions.
   - `// FILE NAME-INPUT,LABEL-?9?CRPSGG`: Input is the transaction file.
   - `// FILE NAME-OUTPUT,LABEL-?9?AR11GG,RECORDS-999000,EXTEND-999000`: Output is the sorted adjustment file.
   - `// RUN` with sort specifications:
     - `HSORTA 15A 3 N`: Ascending sort, 15 fields, 3 levels, no sequence checking.
     - `I C 1 1NECD`: Excludes deleted records.
     - `IAC 31 31EQCJ`: Includes records where position 31 is `'J'` (adjustment transactions, though disabled in `AR050` per March 2024 note).
     - `FNC 7 21 CO/CUST/INV`: Sorts by company/customer/invoice.
   - `// END`: Produces sorted file `AR11GG`.

8. **Validate Sorted Adjustments**:
   - `// IF ?F'A,?9?AR11GG'?/00000000 GOTO END`: Skips to `END` if no adjustment transactions exist.
   - `// LOAD AR111`: Loads the `AR111` program to validate adjustments.
   - Defines files (all shared):
     - `AR111S,LABEL-?9?AR11GG`: Sorted adjustment transactions.
     - `ARTRAN,LABEL-?9?CRPSGG`: Original transaction file.
     - `ARDETL,LABEL-?9?ARDETL,DISP-SHR`: AR detail file.
     - `ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Customer master file.
     - `GLMAST,LABEL-?9?GLMAST,DISP-SHR`: General Ledger master file.
     - `ARCONT,LABEL-?9?ARCONT,DISP-SHR`: AR control file.
     - `GSTABL,LABEL-?9?GSTABL,DISP-SHR`: General system table.
   - `// RUN`: Executes `AR111` to validate adjustments and update `CRPSGG`.

9. **Cleanup and Termination**:
   - `// TAG END`: Marks the end of processing.
   - `// SWITCH 0XXXXXXX`: Resets the first bit of the switch register to `0`, leaving others unchanged, to restore system state.
   - `// IF DATAF1-?9?CR10GG DELETE ?9?CR10GG,F1`: Deletes the sorted payment file.
   - `// IF DATAF1-?9?AR11GG DELETE ?9?AR11GG,F1`: Deletes the sorted adjustment file.

### Business Rules

1. **Conditional Execution**:
   - Only runs if both `CRCKGG` (checks) and `CRWKGG` (invoices) contain records, ensuring valid input from `AR050`.
   - Invoked from `AR050P.ocl36` when option `1` is selected (payment statement mode).

2. **Atrium Compatibility**:
   - Uses `GG` suffix for files (`CRWKGG`, `CRCKGG`, `CRPSGG`, `CR10GG`, `AR11GG`) per JB01 (11/28/23) to align with the Atrium environment, avoiding file conflicts.

3. **Data Integrity and Cleanup**:
   - Deletes temporary files (`CRPSGG`, `CR10GG`, `AR11GG`) before processing to prevent stale data.
   - Deletes sorted files after processing to free disk space.
   - Sorts payments and adjustments separately, including only non-deleted records (`atdel <> 'D'`) and specific transaction types (`'P'` for payments, `'J'` for adjustments).

4. **Transaction Types**:
   - Processes payments (`ATTYPE = 'P'`) and adjustments (`ATTYPE = 'J'`), though adjustments are disabled in `AR050` (per March 2024 note in `AR050.rpgle`).
   - Sorts by company/customer/invoice to organize validation and reporting.

5. **File Sharing**:
   - Uses `DISP-SHR` or `DISP-SHRMM` for master files (`ARCUST`, `ARDETL`, `GLMAST`, `ARCONT`, `GSTABL`) and sales files, supporting multi-user access.
   - `CRTRAN` (`CRPSGG`) has a large capacity (999,000 records, extendable by 500) to handle high volumes.

6. **Y2K Compliance**:
   - `GSY2K` ensures date fields (e.g., payment dates) are processed correctly.

7. **Printer Output**:
   - `AR110` generates edit and NOD reports (`NODLIST`, `NODPRINT`) with conditional queue overrides (`AREDIT` or `TESTOUTQ`) based on `?9?`.

### Tables (Files) Used

1. **?9?CRWKGG** (aliased as `CRWORK`):
   - Type: Input work file (invoices from `AR050`).
   - Usage: Provides open invoice data for validation.

2. **?9?CRCKGG** (aliased as `CRCHKS`):
   - Type: Input work file (checks from `AR050`).
   - Usage: Provides check data.

3. **?9?CRPSGG** (aliased as `CRTRAN`, `ARTRAN`):
   - Type: Output transaction file.
   - Record Capacity: 999,000, extendable by 500.
   - Usage: Stores validated payment/adjustment transactions.

4. **?9?CR10GG** (aliased as `CR110S`):
   - Type: Sorted payment transaction file.
   - Record Capacity: 999,000, extendable by 999,000.
   - Usage: Input for `AR110` validation.

5. **?9?AR11GG** (aliased as `AR111S`):
   - Type: Sorted adjustment transaction file.
   - Record Capacity: 999,000, extendable by 999,000.
   - Usage: Input for `AR111` validation.

6. **?9?ARCONT** (aliased as `ARCONT`):
   - Type: AR control file (shared).
   - Usage: Provides defaults (e.g., GL accounts).

7. **?9?ARCUST** (aliased as `ARCUST`):
   - Type: Customer master file (shared).
   - Usage: Validates customer data.

8. **?9?ARDETL** (aliased as `ARDETL`):
   - Type: AR detail file (shared).
   - Usage: Validates invoice details.

9. **?9?GLMAST** (aliased as `GLMAST`):
   - Type: General Ledger master file (shared).
   - Usage: Validates GL accounts.

10. **?9?GSTABL** (aliased as `GSTABL`, `XGSTABL`):
    - Type: General system table (shared).
    - Usage: Configuration data.

11. **?9?SA5FIND** (aliased as `SA5FIND`):
    - Type: Sales index file (shared, multiple members).
    - Usage: Sales data lookup.

12. **?9?SA5FINM** (aliased as `SA5FINM`):
    - Type: Sales master file (shared, multiple members).
    - Usage: Sales transaction details.

13. **?9?SA5SHX** (aliased as `SA5SHX`):
    - Type: Sales index file (shared, multiple members).
    - Usage: Additional sales indexing.

### External Programs Called

1. **GSY2K**:
   - System utility for Y2K date compliance.
   - Purpose: Ensures correct date handling.

2. **#GSORT**:
   - System utility for sorting transactions into `CR10GG` (payments) and `AR11GG` (adjustments).
   - Purpose: Organizes transactions for validation.

3. **AR105**:
   - RPG program for initial payment statement validation.
   - Purpose: Validates `CRWKGG` and `CRCKGG`, writes to `CRPSGG`.

4. **AR110**:
   - RPG program for validating sorted payment transactions.
   - Purpose: Validates `CR10GG`, updates `CRPSGG`, produces NOD reports.

5. **AR111**:
   - RPG program for validating sorted adjustment transactions.
   - Purpose: Validates `AR11GG`, updates `CRPSGG`.

6. **AR1101** (implied by OCL):
   - Called by `AR110` for additional processing (e.g., posting or reporting).
   - Purpose: Likely handles GL updates or detailed reporting (uses `NODPRINT`).

### Summary

The `AR105.ocl36` script orchestrates the validation of payment statement entries from `AR050` by:
- Ensuring Y2K compliance and checking input files (`CRWKGG`, `CRCKGG`).
- Running `AR105` for initial validation, sorting payments (`AR110`) and adjustments (`AR111`) separately.
- Validating against master files (`ARCUST`, `ARDETL`, `GLMAST`, `ARCONT`) and producing NOD reports.
- Cleaning up temporary files and resetting switches.
- Supporting Atrium with `GG` suffixes and multi-user access via shared files.

It enforces strict data integrity, processes payments and (disabled) adjustments separately, and prepares validated transactions for further processing (e.g., posting via `AR1101`).