The provided OCL program, `AP300.ocl36.txt`, is an Operation Control Language (OCL) script for the IBM System/36 environment, called from the main OCL script (`AP300P.ocl36.txt`) to perform the Accounts Payable (A/P) vendor file update, specifically clearing Month-to-Date (MTD) and/or Year-to-Date (YTD) totals in the vendor master file. It also handles file backups for IRS 1099 processing. Below, I’ll explain the process steps, business rules, tables (files) used, and external programs called, ensuring a clear and concise analysis that ties back to the previously analyzed `AP300P.ocl36.txt` and `AP300P.rpg36.txt`.

---

### Process Steps of the OCL Program

The `AP300.ocl36.txt` script orchestrates the A/P vendor file update process, including sorting the vendor file, saving backups for 1099 processing, maintaining a rolling history of vendor files, and executing the update program. Here’s a step-by-step breakdown:

1. **User Prompt and Pause (`// PAUSE`)**:
   - The script displays a warning message to ensure no users are accessing the Accounts Payable system during the update, as concurrent access could cause data inconsistencies.
   - The message instructs users to exit the A/P system and press `0` to continue or `Shift/ES` and select option `2` to cancel.
   - The `// PAUSE` statement halts execution, waiting for user confirmation, ensuring exclusive access to the vendor file.

2. **Sort Vendor File (`// LOAD #GSORT`)**:
   - Loads the System/36 sort utility (`#GSORT`) to preprocess the vendor file.
   - **Input File**: `?9?APVEND` (vendor master file, shared mode `DISP-SHR`).
   - **Output File**: `?9?AP300S` (temporary sorted file, created with up to 999,000 records, extendable by 999,000, retained as a job file `RETAIN-J`).
   - **Sort Specifications**:
     - `HSORTA 7A 3X N`: Sorts in ascending order (`A`) on a 7-byte field, with additional specifications (`3X` likely indicates a control field or exclusion, `N` for no sequence checking).
     - **Selection Criteria**:
       - Records are included if the company number (positions 2–3) matches one of the user-specified companies (`?L'114,2'?`, `?L'116,2'?`, `?L'118,2'?` from the Local Data Area, corresponding to `KYCO1`, `KYCO2`, `KYCO3` from `AP300P.rpg36.txt`) and the record is not deleted (`NECD` at position 1).
       - If `KYALCO = 'ALL'` (from `AP300P`), all non-deleted records are likely included, as the OCL script relies on `AP300P` to validate company selection.
     - **Fields**:
       - `FNC 2 3 COMPANY`: Company number (2 bytes, positions 2–3).
       - `FNC 4 8 VENDOR`: Vendor number (5 bytes, positions 4–8).
   - The sorted output (`?9?AP300S`) contains only the records needed for the update, filtered by company and deletion status.

3. **Save Vendor File for 1099 Processing (`// IF ?L'135,1'?/Y ...`)**:
   - Checks if `?L'135,1'?` (corresponding to `KYYTDY` in `AP300P.rpg36.txt`) is `Y`, indicating YTD totals are to be cleared.
   - If true, copies the vendor file (`?9?APVEND`) to two backup files for 1099 processing:
     - `QS36F/APVN?L'136,4'?`: Named `APVN` followed by the 4-digit year (`KYCCYY` from `AP300P`, e.g., `APVN2025`).
     - `QS36F/?9?VN?L'136,4'?`: Named with library prefix `?9?` (e.g., `PRODVN2025`).
     - Both use `CPYF` with `MBROPT(*REPLACE)` to overwrite existing files and `CRTFILE(*YES)` to create the files if they don’t exist.
   - This step ensures a snapshot of the vendor file is preserved for IRS 1099 reporting, as YTD clearing would reset tax-related data.

4. **Maintain Rolling Vendor File History**:
   - Manages a rolling history of up to 12 months of vendor file backups (`?9?U1VEND` to `?9?UCVEND`).
   - **Steps**:
     - Deletes the oldest file (`?9?UCVEND`) if it exists (`DELETE ?9?UCVEND,F1`).
     - Renames files to shift the history:
       - `?9?UBVEND` to `?9?UCVEND`
       - `?9?UAVEND` to `?9?UBVEND`
       - `?9?U9VEND` to `?9?UAVEND`
       - ... down to `?9?U1VEND` to `?9?U2VEND`.
     - Copies the current vendor file (`?9?APVEND`) to `?9?U1VEND` using `CPYF` with `MBROPT(*REPLACE)` and `CRTFILE(*YES)`.
   - This creates a new backup (`?9?U1VEND`) and shifts older backups, maintaining a 12-month history for auditing or recovery.

5. **Set Inquiry Attribute (`// ATTR INQUIRY-YES,CANCEL-NO`)**:
   - Sets the job attribute to allow inquiry (`YES`) but disallow cancellation (`NO`) during the update, ensuring the process completes once started.

6. **Execute Update Program (`// LOAD AP300`)**:
   - Loads the `AP300` program, which performs the actual MTD/YTD clearing.
   - **Files**:
     - `APVEND`: Input vendor master file (`?9?APVEND`, shared mode).
     - `AP300S`: Sorted temporary file (`?9?AP300S`) from the sort step.
   - **Run**: Executes `AP300` to process the sorted records and update `APVEND` by clearing MTD and/or YTD totals based on parameters set by `AP300P.rpg36.txt` (e.g., `KYALCO`, `KYCO1`, `KYYTDY`).

---

### Business Rules

The OCL script enforces the following business rules for the A/P vendor file update:

1. **Exclusive Access**:
   - No users can access the A/P system during the update to prevent data corruption. The `PAUSE` ensures users confirm they’ve exited the system.

2. **Company Filtering**:
   - Updates are applied to either all non-deleted vendors (`KYALCO = 'ALL'`) or specific companies (`KYCO1`, `KYCO2`, `KYCO3`) as validated by `AP300P`.
   - Only non-deleted records (`NECD ≠ 'D'`) are processed, as specified in the sort criteria.

3. **YTD Backup for 1099**:
   - If YTD totals are cleared (`KYYTDY = 'Y'`), the vendor file is backed up to two files (`APVN<year>` and `?9?VN<year>`) for IRS 1099 processing.
   - The year (`KYCCYY`) must be provided, as validated by `AP300P`.

4. **Rolling File History**:
   - Maintains a 12-month history of vendor file snapshots (`?9?U1VEND` to `?9?UCVEND`) for auditing or recovery.
   - The oldest file is deleted, and others are renamed to accommodate a new backup each time the process runs.

5. **File Management**:
   - The sorted file (`?9?AP300S`) is temporary (`RETAIN-J`) and used only for the current job.
   - The vendor file (`?9?APVEND`) is updated in shared mode to allow read access but ensure safe updates.

6. **No Cancellation During Update**:
   - Once the update starts (`AP300` is loaded), cancellation is disabled (`CANCEL-NO`) to ensure completion.

---

### Tables (Files) Used

The OCL script uses the following files:

1. **?9?APVEND**:
   - Type: Disk file (vendor master file).
   - Purpose: Primary input file containing vendor data, updated by `AP300`.
   - Access: Shared mode (`DISP-SHR`) for sorting and updating.
   - Label: `?9?APVEND` (e.g., `PRODAPVEND` with library prefix).

2. **?9?AP300S**:
   - Type: Temporary disk file.
   - Purpose: Stores sorted vendor records filtered by company and deletion status for processing by `AP300`.
   - Attributes: Up to 999,000 records, extendable, retained for the job (`RETAIN-J`).
   - Label: `?9?AP300S`.

3. **APVN?L'136,4'?**:
   - Type: Disk file in library `QS36F`.
   - Purpose: Backup of vendor file for 1099 processing when YTD is cleared (e.g., `APVN2025` for year 2025).
   - Attributes: Created/overwritten with `CPYF`, `MBROPT(*REPLACE)`, `CRTFILE(*YES)`.

4. **?9?VN?L'136,4'?**:
   - Type: Disk file in library `QS36F`.
   - Purpose: Additional 1099 backup with library prefix (e.g., `PRODVN2025`).
   - Attributes: Same as `APVN<year>`.

5. **?9?U1VEND to ?9?UCVEND**:
   - Type: Disk files (12 files).
   - Purpose: Rolling history of vendor file backups for up to 12 months.
   - Operations: Renamed in sequence, with the oldest (`?9?UCVEND`) deleted and a new `?9?U1VEND` created.

---

### External Programs Called

The OCL script calls the following programs:

1. **#GSORT**:
   - System/36 sort utility.
   - Purpose: Sorts the `?9?APVEND` file into `?9?AP300S` based on company and vendor fields, filtering by user-specified companies and non-deleted records.

2. **AP300**:
   - Main update program.
   - Purpose: Processes the sorted file (`?9?AP300S`) to clear MTD and/or YTD totals in `?9?APVEND` based on parameters from `AP300P` (e.g., `KYALCO`, `KYYTDY`).

---

### Summary

**Process Overview**:
- The `AP300.ocl36.txt` script manages the A/P vendor file update process:
  - Prompts users to ensure exclusive access with a `PAUSE`.
  - Sorts the vendor file (`?9?APVEND`) into a temporary file (`?9?AP300S`) based on company selection.
  - If YTD is cleared, saves the vendor file to two 1099 backup files (`APVN<year>`, `?9?VN<year>`).
  - Maintains a 12-month rolling history of vendor file backups (`?9?U1VEND` to `?9?UCVEND`).
  - Executes the `AP300` program to clear MTD/YTD totals in `?9?APVEND`.

**Business Rules**:
- Requires exclusive A/P system access during execution.
- Filters updates by company (`ALL` or specific) and excludes deleted records.
- Saves YTD data for 1099 processing when clearing YTD totals.
- Maintains a 12-month vendor file history.
- Disables cancellation during the update to ensure completion.

**Tables Used**:
- `?9?APVEND`: Vendor master file (input/output).
- `?9?AP300S`: Temporary sorted file.
- `APVN<year>`, `?9?VN<year>`: 1099 backup files.
- `?9?U1VEND` to `?9?UCVEND`: Rolling history backups.

**External Programs**:
- `#GSORT`: Sorts the vendor file.
- `AP300`: Performs the MTD/YTD update.

This script integrates with `AP300P.rpg36.txt` (for input validation) and `AP300P.ocl36.txt` (for job control), ensuring a robust A/P update process with proper data backup and validation. If you need details on the `AP300` program’s RPG logic or further clarification, please provide additional files or context!