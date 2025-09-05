The `FR713C` program is a CL (Control Language) program called by `FR713PC` to facilitate the generation of the Freight Out Reconciliation Report, specifically when sorted by location. It prepares a work file and invokes two programs: one to build the work file (`FR713A`) and another to print the report (`FR713B`). Below, I’ll explain the process steps, business rules, database tables used, and external programs called, based on the provided CLP source code.

### Process Steps of the FR713C Program

The `FR713C` program orchestrates the creation of a temporary work file, clears existing data, applies file overrides, builds the work file, and generates the report. Here’s a detailed breakdown of the process steps:

1. **Program Declaration and Parameter Definition**:
   - **Purpose**: Defines the program and its input parameters.
   - **Actions**:
     - Declares the program with nine input parameters, all of type `*CHAR`, matching those passed from `FR713PC`:
       - `&P$CO` (2): Company code.
       - `&P$FDAT` (8): From date (in `CYMD` format).
       - `&P$TDAT` (8): To date (in `CYMD` format).
       - `&P$LOC` (3): Location code.
       - `&P$CAID` (6): Carrier ID.
       - `&P$SORT` (1): Sort option (expected to be 'L' for location, as this program is called by `FR713PC` when `&P$SORT = 'L'`).
       - `&P$CUST` (6): Customer code.
       - `&P$CAR$` (1): Flag to include zero-dollar freight records ('Y' or 'N').
       - `&P$FGRP` (1): File group ('G' or 'Z', determining the database file set).
     - Declares a variable `&FR714W2` (8 characters) to construct the name of a secondary work file by concatenating `&P$FGRP` with the string 'FR714W2' (e.g., 'GFR714W2' or 'ZFR714W2').

2. **Create Work File (`FR713W`)**:
   - **Purpose**: Ensures the temporary work file `FR713W` exists in the `QTEMP` library for storing intermediate data.
   - **Actions**:
     - Checks if `FR713W` exists in `QTEMP` using `CHKOBJ OBJ(QTEMP/FR713W) OBJTYPE(*FILE)`.
     - If the file does not exist (`CPF9801` message), creates it using `CRTDUPOBJ`:
       - If `&P$FGRP = 'G'`, duplicates `FR713W` from the `DATA` library to `QTEMP` with constraints (`CST(*NO)`) and triggers (`TRG(*NO)`) disabled.
       - If `&P$FGRP = 'Z'`, duplicates `FR713W` from the `DATADEV` library to `QTEMP` with the same settings.
     - This step ensures a clean, temporary file is available for the report generation process.

3. **Clear Work Files**:
   - **Purpose**: Clears data in the work files to prepare them for new data.
   - **Actions**:
     - Clears the `FR713W` file in `QTEMP` using `CLRPFM FILE(QTEMP/FR713W)` (updated by revision `JK01` to explicitly reference `QTEMP`).
     - Clears the secondary work file (e.g., `GFR714W2` or `ZFR714W2`) in the library list using `CLRPFM FILE(*LIBL/&FR714W2)`.

4. **Apply File Overrides**:
   - **Purpose**: Redirects file access to the temporary work file and ensures the correct secondary work file is used.
   - **Actions**:
     - Overrides the `FR713W` file to point to `QTEMP/FR713W` using `OVRDBF FILE(FR713W) TOFILE(QTEMP/FR713W)` (updated by `JK01` for explicit `QTEMP` reference).
     - Overrides the `FR714W2` file to point to the library list file (e.g., `GFR714W2` or `ZFR714W2`) using `OVRDBF FILE(FR714W2) TOFILE(*LIBL/&FR714W2)`.

5. **Build Work File**:
   - **Purpose**: Populates the `FR713W` work file with data for the report.
   - **Actions**:
     - Calls the `FR713A` program, passing eight parameters: `&P$CO`, `&P$FDAT`, `&P$TDAT`, `&P$LOC`, `&P$CAID`, `&P$CUST`, `&P$CAR$`, `&P$FGRP`.
     - The `FR713A` program is responsible for querying the necessary database tables and writing data to `FR713W` based on the provided parameters.

6. **Call Print Program**:
   - **Purpose**: Generates the final report using the data in the work file.
   - **Actions**:
     - Calls the `FR713B` program, passing all nine parameters: `&P$CO`, `&P$FDAT`, `&P$TDAT`, `&P$LOC`, `&P$CAID`, `&P$SORT`, `&P$CUST`, `&P$CAR$`, `&P$FGRP`.
     - The `FR713B` program processes the data in `FR713W` and produces the Freight Out Reconciliation Report sorted by location.

7. **Program Termination**:
   - **Purpose**: Ends the program after completing the report generation.
   - **Actions**:
     - Executes the `ENDPGM` command to terminate the program.

### Business Rules

The `FR713C` program enforces the following business rules:
1. **Temporary Work File Creation**:
   - The program ensures a temporary work file (`FR713W`) is created in `QTEMP` to store intermediate data, specific to the user’s session, preventing conflicts in a multi-user environment.
   - The source library for `FR713W` depends on the file group (`DATA` for `&P$FGRP = 'G'`, `DATADEV` for `&P$FGRP = 'Z'`), aligning with the file overrides set by `FR713P`.
2. **Data Isolation**:
   - Clearing `FR713W` and `FR714W2` ensures no residual data from previous runs affects the current report.
   - Using `QTEMP` for `FR713W` isolates the data to the current job, enhancing security and performance.
3. **File Overrides**:
   - Explicit overrides ensure that `FR713A` and `FR713B` access the correct temporary work file (`QTEMP/FR713W`) and the appropriate secondary work file (`GFR714W2` or `ZFR714W2`).
4. **Parameter Consistency**:
   - All parameters received from `FR713PC` are passed unchanged to `FR713A` and `FR713B`, ensuring consistency with user inputs validated by `FR713P`.
5. **Sort by Location**:
   - The program is designed to handle the report sorted by location (`&P$SORT = 'L'`), as it is called by `FR713PC` for this specific case.
6. **No Direct Validation**:
   - The program assumes all input parameters are valid, relying on `FR713P` for validation (e.g., company, dates, location, customer).

### Database Tables Used

The `FR713C` program does not directly access database tables but manages two work files and relies on `FR713A` and `FR713B` for data processing. The files involved are:
1. **FR713W**:
   - A temporary work file created in `QTEMP`.
   - Used to store intermediate data for the Freight Out Reconciliation Report.
   - Source: Duplicated from `DATA/FR713W` (if `&P$FGRP = 'G'`) or `DATADEV/FR713W` (if `&P$FGRP = 'Z'`).
   - Cleared and overridden to `QTEMP/FR713W` for use by `FR713A` and `FR713B`.
2. **FR714W2**:
   - A secondary work file (e.g., `GFR714W2` or `ZFR714W2`) in the library list.
   - Cleared and overridden to ensure the correct file is used based on `&P$FGRP`.
   - Likely used by `FR713A` or `FR713B` for additional data processing or cross-referencing.

The actual database tables (e.g., `glcont`, `inloc`, `arcust`, and likely freight transaction tables) are accessed by `FR713A` to build the work file, but these are not explicitly mentioned in the `FR713C` code. Based on the context from `FR713P`, the following tables are likely involved indirectly:
- **glcont** (General Ledger Control): For company data (overridden to `gglcont` or `zglcont`).
- **inloc** (Inventory Location): For location data (overridden to `ginloc` or `zinloc`).
- **arcust** (Accounts Receivable Customer): For customer data (overridden to `garcust` or `zarcust`).
- **Other Freight Tables**: Likely accessed by `FR713A` to retrieve freight transaction data.

### External Programs Called

The `FR713C` program calls the following external programs:
1. **FR713A**:
   - **Purpose**: Builds the work file `FR713W` by querying relevant database tables and applying the input parameters (e.g., company, date range, location, customer).
   - **Parameters**: `&P$CO`, `&P$FDAT`, `&P$TDAT`, `&P$LOC`, `&P$CAID`, `&P$CUST`, `&P$CAR$`, `&P$FGRP`.
2. **FR713B**:
   - **Purpose**: Generates the Freight Out Reconciliation Report sorted by location, using the data in `FR713W`.
   - **Parameters**: `&P$CO`, `&P$FDAT`, `&P$TDAT`, `&P$LOC`, `&P$CAID`, `&P$SORT`, `&P$CUST`, `&P$CAR$`, `&P$FGRP`.

### Additional Notes
- **Revisions (JK01)**:
  - On 07/23/21, the program was updated to:
    - Explicitly reference `QTEMP/FR713W` in `CLRPFM` and `OVRDBF` commands for clarity and reliability.
    - Replace library `ARGDEVTEST` with `DATADEV` and `ARGDEV` with `DATA` in the `CRTDUPOBJ` command to align with the correct production and development environments.
- **Temporary Files**:
  - Using `QTEMP` ensures that `FR713W` is job-specific and automatically cleaned up when the job ends, reducing resource conflicts.
- **Error Handling**:
  - The program handles the case where `FR713W` does not exist in `QTEMP` by creating it. No additional error handling is implemented, relying on `FR713P` for input validation.
- **Integration**:
  - `FR713C` is part of a program chain (`FR713P` → `FR713PC` → `FR713C` → `FR713A`/`FR713B`) designed to modularize the report generation process, separating input validation, dispatching, data preparation, and printing.

### Summary

The `FR713C` program prepares a temporary work file (`FR713W`) in `QTEMP`, clears it and a secondary work file (`FR714W2`), applies file overrides, and calls `FR713A` to populate the work file and `FR713B` to print the Freight Out Reconciliation Report sorted by location. It enforces business rules for temporary file management and parameter consistency, relying on `FR713A` and `FR713B` for data access and report generation. The program does not directly access database tables but manages work files, with actual data retrieval handled by the called programs.