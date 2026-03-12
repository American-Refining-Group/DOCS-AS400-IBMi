The `FR714C` program is a CL (Control Language) program called by `FR713PC` to facilitate the generation of the Freight Out Reconciliation Report, specifically when sorted by carrier/routing. It prepares a temporary work file and invokes two programs: one to build the work file (`FR714A`) and another to print the report (`FR714B`). Below, I’ll explain the process steps, business rules, database tables used, and external programs called, based on the provided CLP source code.

### Process Steps of the FR714C Program

The `FR714C` program follows a structure similar to `FR713C`, with the primary difference being the focus on generating the report sorted by carrier/routing. Here’s a detailed breakdown of the process steps:

1. **Program Declaration and Parameter Definition**:
   - **Purpose**: Defines the program and its input parameters.
   - **Actions**:
     - Declares the program with nine input parameters, all of type `*CHAR`, matching those passed from `FR713PC`:
       - `&P$CO` (2): Company code.
       - `&P$FDAT` (8): From date (in `CYMD` format).
       - `&P$TDAT` (8): To date (in `CYMD` format).
       - `&P$LOC` (2): Location code (Note: Defined as LEN(2) here, unlike LEN(3) in `FR713C`, which may be a typo or intentional based on system requirements).
       - `&P$CAID` (6): Carrier ID.
       - `&P$SORT` (1): Sort option (expected to be 'C' for carrier/routing, as this program is called by `FR713PC` when `&P$SORT = 'C'`).
       - `&P$CUST` (6): Customer code.
       - `&P$CAR$` (1): Flag to include zero-dollar freight records ('Y' or 'N').
       - `&P$FGRP` (1): File group ('G' or 'Z', determining the database file set).
     - Declares a variable `&FR714W2` (8 characters) to construct the name of a secondary work file by concatenating `&P$FGRP` with the string 'FR714W2' (e.g., 'GFR714W2' or 'ZFR714W2').

2. **Create Work File (`FR714W`)**:
   - **Purpose**: Ensures the temporary work file `FR714W` exists in the `QTEMP` library for storing intermediate data.
   - **Actions**:
     - Checks if `FR714W` exists in `QTEMP` using `CHKOBJ OBJ(QTEMP/FR714W) OBJTYPE(*FILE)`.
     - If the file does not exist (`CPF9801` message), creates it using `CRTDUPOBJ`:
       - If `&P$FGRP = 'G'`, duplicates `FR714W` from the `DATA` library to `QTEMP` with constraints (`CST(*NO)`) and triggers (`TRG(*NO)`) disabled.
       - If `&P$FGRP = 'Z'`, duplicates `FR714W` from the `DATADEV` library to `QTEMP` with the same settings.
     - This step ensures a clean, temporary file is available for the report generation process.

3. **Clear Work Files**:
   - **Purpose**: Clears data in the work files to prepare them for new data.
   - **Actions**:
     - Clears the `FR714W` file in `QTEMP` using `CLRPFM FILE(QTEMP/FR714W)` (updated by revision `JK01` to explicitly reference `QTEMP`).
     - Clears the secondary work file (e.g., `GFR714W2` or `ZFR714W2`) in the library list using `CLRPFM FILE(*LIBL/&FR714W2)`.

4. **Apply File Overrides**:
   - **Purpose**: Redirects file access to the temporary work file and ensures the correct secondary work file is used.
   - **Actions**:
     - Overrides the `FR714W` file to point to `QTEMP/FR714W` using `OVRDBF FILE(FR714W) TOFILE(QTEMP/FR714W)` (updated by `JK01` for explicit `QTEMP` reference).
     - Overrides the `FR714W2` file to point to the library list file (e.g., `GFR714W2` or `ZFR714W2`) using `OVRDBF FILE(FR714W2) TOFILE(*LIBL/&FR714W2)`.

5. **Build Work File**:
   - **Purpose**: Populates the `FR714W` work file with data for the report.
   - **Actions**:
     - Calls the `FR714A` program, passing eight parameters: `&P$CO`, `&P$FDAT`, `&P$TDAT`, `&P$LOC`, `&P$CAID`, `&P$CUST`, `&P$CAR$`, `&P$FGRP`.
     - The `FR714A` program is responsible for querying the necessary database tables and writing data to `FR714W` based on the provided parameters, sorted by carrier/routing.

6. **Call Print Program**:
   - **Purpose**: Generates the final report using the data in the work file.
   - **Actions**:
     - Calls the `FR714B` program, passing all nine parameters: `&P$CO`, `&P$FDAT`, `&P$TDAT`, `&P$LOC`, `&P$CAID`, `&P$SORT`, `&P$CUST`, `&P$CAR$`, `&P$FGRP`.
     - The `FR714B` program processes the data in `FR714W` and produces the Freight Out Reconciliation Report sorted by carrier/routing.

7. **Program Termination**:
   - **Purpose**: Ends the program after completing the report generation.
   - **Actions**:
     - Executes the `ENDPGM` command to terminate the program.

### Business Rules

The `FR714C` program enforces the following business rules:
1. **Temporary Work File Creation**:
   - Ensures a temporary work file (`FR714W`) is created in `QTEMP` to store intermediate data, specific to the user’s session, preventing conflicts in a multi-user environment.
   - The source library for `FR714W` depends on the file group (`DATA` for `&P$FGRP = 'G'`, `DATADEV` for `&P$FGRP = 'Z'`), aligning with the file overrides set by `FR713P`.
2. **Data Isolation**:
   - Clearing `FR714W` and `FR714W2` ensures no residual data from previous runs affects the current report.
   - Using `QTEMP` for `FR714W` isolates the data to the current job, enhancing security and performance.
3. **File Overrides**:
   - Explicit overrides ensure that `FR714A` and `FR714B` access the correct temporary work file (`QTEMP/FR714W`) and the appropriate secondary work file (`GFR714W2` or `ZFR714W2`).
4. **Parameter Consistency**:
   - All parameters received from `FR713PC` are passed unchanged to `FR714A` and `FR714B`, ensuring consistency with user inputs validated by `FR713P`.
5. **Sort by Carrier/Routing**:
   - The program is designed to handle the report sorted by carrier/routing (`&P$SORT = 'C'`), as it is called by `FR713PC` for this specific case.
6. **No Direct Validation**:
   - The program assumes all input parameters are valid, relying on `FR713P` for validation (e.g., company, dates, location, customer).

### Database Tables Used

The `FR714C` program does not directly access database tables but manages two work files and relies on `FR714A` and `FR714B` for data processing. The files involved are:
1. **FR714W**:
   - A temporary work file created in `QTEMP`.
   - Used to store intermediate data for the Freight Out Reconciliation Report sorted by carrier/routing.
   - Source: Duplicated from `DATA/FR714W` (if `&P$FGRP = 'G'`) or `DATADEV/FR714W` (if `&P$FGRP = 'Z'`).
   - Cleared and overridden to `QTEMP/FR714W` for use by `FR714A` and `FR714B`.
2. **FR714W2**:
   - A secondary work file (e.g., `GFR714W2` or `ZFR714W2`) in the library list.
   - Cleared and overridden to ensure the correct file is used based on `&P$FGRP`.
   - Likely used by `FR714A` or `FR714B` for additional data processing or cross-referencing.

The actual database tables (e.g., `glcont`, `inloc`, `arcust`, and likely freight transaction tables) are accessed by `FR714A` to build the work file, but these are not explicitly mentioned in the `FR714C` code. Based on the context from `FR713P`, the following tables are likely involved indirectly:
- **glcont** (General Ledger Control): For company data (overridden to `gglcont` or `zglcont`).
- **inloc** (Inventory Location): For location data (overridden to `ginloc` or `zinloc`).
- **arcust** (Accounts Receivable Customer): For customer data (overridden to `garcust` or `zarcust`).
- **Other Freight Tables**: Likely accessed by `FR714A` to retrieve freight transaction data.

### External Programs Called

The `FR714C` program calls the following external programs:
1. **FR714A**:
   - **Purpose**: Builds the work file `FR714W` by querying relevant database tables and applying the input parameters, sorted by carrier/routing.
   - **Parameters**: `&P$CO`, `&P$FDAT`, `&P$TDAT`, `&P$LOC`, `&P$CAID`, `&P$CUST`, `&P$CAR$`, `&P$FGRP`.
2. **FR714B**:
   - **Purpose**: Generates the Freight Out Reconciliation Report sorted by carrier/routing, using the data in `FR714W`.
   - **Parameters**: `&P$CO`, `&P$FDAT`, `&P$TDAT`, `&P$LOC`, `&P$CAID`, `&P$SORT`, `&P$CUST`, `&P$CAR$`, `&P$FGRP`.

### Additional Notes
- **Revisions (JK01)**:
  - On 08/30/21, the program was updated to:
    - Explicitly reference `QTEMP/FR714W` in `CLRPFM` and `OVRDBF` commands for clarity and reliability.
    - Replace library `ARGDEVTEST` with `DATADEV` and `ARGDEV` with `DATA` in the `CRTDUPOBJ` command to align with the correct production and development environments.
- **Temporary Files**:
  - Using `QTEMP` for `FR714W` ensures that the file is job-specific and automatically cleaned up when the job ends, reducing resource conflicts.
- **Error Handling**:
  - The program handles the case where `FR714W` does not exist in `QTEMP` by creating it. No additional error handling is implemented, relying on `FR713P` for input validation.
- **Integration**:
  - `FR714C` is part of the program chain (`FR713P` → `FR713PC` → `FR714C` → `FR714A`/`FR714B`) designed to modularize the report generation process, separating input validation, dispatching, data preparation, and printing.
- **Location Code Length**:
  - The discrepancy in `&P$LOC` length (LEN(2) in `FR714C` vs. LEN(3) in `FR713C`) may indicate a system-specific requirement or a potential error in the code that should be verified.

### Summary

The `FR714C` program prepares a temporary work file (`FR714W`) in `QTEMP`, clears it and a secondary work file (`FR714W2`), applies file overrides, and calls `FR714A` to populate the work file and `FR714B` to print the Freight Out Reconciliation Report sorted by carrier/routing. It enforces business rules for temporary file management and parameter consistency, relying on `FR714A` and `FR714B` for data access and report generation. The program does not directly access database tables but manages work files, with actual data retrieval handled by the called programs.