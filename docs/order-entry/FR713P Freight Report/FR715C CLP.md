The `FR715C` program is a CL (Control Language) program called by `FR713PC` to facilitate the generation of the Freight Out Reconciliation Report, specifically when sorted by product code. It prepares a temporary work file and invokes two programs: one to build the work file (`FR715A`) and another to print the report (`FR715B`). The structure is nearly identical to `FR713C` and `FR714C`, with differences in the work file and programs called to support sorting by product code. Below, I’ll explain the process steps, business rules, database tables used, and external programs called, based on the provided CLP source code.

### Process Steps of the FR715C Program

The `FR715C` program prepares a temporary work file (`FR715W`) in `QTEMP`, clears it and a secondary work file, applies file overrides, and calls `FR715A` to populate the work file and `FR715B` to print the report. Here’s a detailed breakdown of the process steps:

1. **Program Declaration and Parameter Definition**:
   - **Purpose**: Defines the program and its input parameters.
   - **Actions**:
     - Declares the program with nine input parameters, all of type `*CHAR`, matching those passed from `FR713PC`:
       - `&P$CO` (2): Company code.
       - `&P$FDAT` (8): From date (in `CYMD` format).
       - `&P$TDAT` (8): To date (in `CYMD` format).
       - `&P$LOC` (2): Location code (Note: Defined as LEN(2) here, unlike LEN(3) in `FR713C`, consistent with `FR714C`, which may indicate a system-specific requirement or a typo).
       - `&P$CAID` (6): Carrier ID.
       - `&P$SORT` (1): Sort option (expected to be 'P' for product code, as this program is called by `FR713PC` when `&P$SORT = 'P'`).
       - `&P$CUST` (6): Customer code.
       - `&P$CAR$` (1): Flag to include zero-dollar freight records ('Y' or 'N').
       - `&P$FGRP` (1): File group ('G' or 'Z', determining the database file set).
     - Declares a variable `&FR714W2` (8 characters) to construct the name of a secondary work file by concatenating `&P$FGRP` with the string 'FR714W2' (e.g., 'GFR714W2' or 'ZFR714W2'). (Note: The use of `FR714W2` across `FR713C`, `FR714C`, and `FR715C` suggests it may be a shared file or a naming convention for consistency.)

2. **Create Work File (`FR715W`)**:
   - **Purpose**: Ensures the temporary work file `FR715W` exists in the `QTEMP` library for storing intermediate data.
   - **Actions**:
     - Checks if `FR715W` exists in `QTEMP` using `CHKOBJ OBJ(QTEMP/FR715W) OBJTYPE(*FILE)`.
     - If the file does not exist (`CPF9801` message), creates it using `CRTDUPOBJ`:
       - If `&P$FGRP = 'G'`, duplicates `FR715W` from the `DATA` library to `QTEMP` with constraints (`CST(*NO)`) and triggers (`TRG(*NO)`) disabled.
       - If `&P$FGRP = 'Z'`, duplicates `FR715W` from the `DATADEV` library to `QTEMP` with the same settings.
     - This step ensures a clean, temporary file is available for the report generation process.

3. **Clear Work Files**:
   - **Purpose**: Clears data in the work files to prepare them for new data.
   - **Actions**:
     - Clears the `FR715W` file in `QTEMP` using `CLRPFM FILE(QTEMP/FR715W)` (updated by revision `JK01` to explicitly reference `QTEMP`).
     - Clears the secondary work file (e.g., `GFR714W2` or `ZFR714W2`) in the library list using `CLRPFM FILE(*LIBL/&FR714W2)`.

4. **Apply File Overrides**:
   - **Purpose**: Redirects file access to the temporary work file and ensures the correct secondary work file is used.
   - **Actions**:
     - Overrides the `FR715W` file to point to `QTEMP/FR715W` using `OVRDBF FILE(FR715W) TOFILE(QTEMP/FR715W)` (updated by `JK01` for explicit `QTEMP` reference).
     - Overrides the `FR714W2` file to point to the library list file (e.g., `GFR714W2` or `ZFR714W2`) using `OVRDBF FILE(FR714W2) TOFILE(*LIBL/&FR714W2)`.

5. **Build Work File**:
   - **Purpose**: Populates the `FR715W` work file with data for the report.
   - **Actions**:
     - Calls the `FR715A` program, passing eight parameters: `&P$CO`, `&P$FDAT`, `&P$TDAT`, `&P$LOC`, `&P$CAID`, `&P$CUST`, `&P$CAR$`, `&P$FGRP`.
     - The `FR715A` program is responsible for querying the necessary database tables and writing data to `FR715W` based on the provided parameters, sorted by product code.

6. **Call Print Program**:
   - **Purpose**: Generates the final report using the data in the work file.
   - **Actions**:
     - Calls the `FR715B` program, passing all nine parameters: `&P$CO`, `&P$FDAT`, `&P$TDAT`, `&P$LOC`, `&P$CAID`, `&P$SORT`, `&P$CUST`, `&P$CAR$`, `&P$FGRP`.
     - The `FR715B` program processes the data in `FR715W` and produces the Freight Out Reconciliation Report sorted by product code.

7. **Program Termination**:
   - **Purpose**: Ends the program after completing the report generation.
   - **Actions**:
     - Executes the `ENDPGM` command to terminate the program.

### Business Rules

The `FR715C` program enforces the following business rules:
1. **Temporary Work File Creation**:
   - Ensures a temporary work file (`FR715W`) is created in `QTEMP` to store intermediate data, specific to the user’s session, preventing conflicts in a multi-user environment.
   - The source library for `FR715W` depends on the file group (`DATA` for `&P$FGRP = 'G'`, `DATADEV` for `&P$FGRP = 'Z'`), aligning with the file overrides set by `FR713P`.
2. **Data Isolation**:
   - Clearing `FR715W` and `FR714W2` ensures no residual data from previous runs affects the current report.
   - Using `QTEMP` for `FR715W` isolates the data to the current job, enhancing security and performance.
3. **File Overrides**:
   - Explicit overrides ensure that `FR715A` and `FR715B` access the correct temporary work file (`QTEMP/FR715W`) and the appropriate secondary work file (`GFR714W2` or `ZFR714W2`).
4. **Parameter Consistency**:
   - All parameters received from `FR713PC` are passed unchanged to `FR715A` and `FR715B`, ensuring consistency with user inputs validated by `FR713P`.
5. **Sort by Product Code**:
   - The program is designed to handle the report sorted by product code (`&P$SORT = 'P'`), as it is called by `FR713PC` for this specific case.
6. **No Direct Validation**:
   - The program assumes all input parameters are valid, relying on `FR713P` for validation (e.g., company, dates, location, customer).
7. **Secondary Work File**:
   - The use of `FR714W2` (despite the program being `FR715C`) suggests a shared work file across the report variants (`FR713C`, `FR714C`, `FR715C`), likely for common data or cross-referencing, which may be a design choice or require verification for correctness.

### Database Tables Used

The `FR715C` program does not directly access database tables but manages two work files and relies on `FR715A` and `FR715B` for data processing. The files involved are:
1. **FR715W**:
   - A temporary work file created in `QTEMP`.
   - Used to store intermediate data for the Freight Out Reconciliation Report sorted by product code.
   - Source: Duplicated from `DATA/FR715W` (if `&P$FGRP = 'G'`) or `DATADEV/FR715W` (if `&P$FGRP = 'Z'`).
   - Cleared and overridden to `QTEMP/FR715W` for use by `FR715A` and `FR715B`.
2. **FR714W2**:
   - A secondary work file (e.g., `GFR714W2` or `ZFR714W2`) in the library list.
   - Cleared and overridden to ensure the correct file is used based on `&P$FGRP`.
   - Likely used by `FR715A` or `FR715B` for additional data processing or cross-referencing. (Note: The naming `FR714W2` in `FR715C` is unusual and may indicate a shared file or a potential oversight in naming.)

The actual database tables (e.g., `glcont`, `inloc`, `arcust`, and likely freight transaction tables) are accessed by `FR715A` to build the work file, but these are not explicitly mentioned in the `FR715C` code. Based on the context from `FR713P`, the following tables are likely involved indirectly:
- **glcont** (General Ledger Control): For company data (overridden to `gglcont` or `zglcont`).
- **inloc** (Inventory Location): For location data (overridden to `ginloc` or `zinloc`).
- **arcust** (Accounts Receivable Customer): For customer data (overridden to `garcust` or `zarcust`).
- **Other Freight Tables**: Likely accessed by `FR715A` to retrieve freight transaction data, possibly including product code details specific to this report variant.

### External Programs Called

The `FR715C` program calls the following external programs:
1. **FR715A**:
   - **Purpose**: Builds the work file `FR715W` by querying relevant database tables and applying the input parameters, sorted by product code.
   - **Parameters**: `&P$CO`, `&P$FDAT`, `&P$TDAT`, `&P$LOC`, `&P$CAID`, `&P$CUST`, `&P$CAR$`, `&P$FGRP`.
2. **FR715B**:
   - **Purpose**: Generates the Freight Out Reconciliation Report sorted by product code, using the data in `FR715W`.
   - **Parameters**: `&P$CO`, `&P$FDAT`, `&P$TDAT`, `&P$LOC`, `&P$CAID`, `&P$SORT`, `&P$CUST`, `&P$CAR$`, `&P$FGRP`.

### Additional Notes
- **Revisions (JK01)**:
  - On 08/30/21, the program was updated to:
    - Explicitly reference `QTEMP/FR715W` in `CLRPFM` and `OVRDBF` commands for clarity and reliability.
    - Replace library `ARGDEVTEST` with `DATADEV` and `ARGDEV` with `DATA` in the `CRTDUPOBJ` command to align with the correct production and development environments.
- **Temporary Files**:
  - Using `QTEMP` for `FR715W` ensures that the file is job-specific and automatically cleaned up when the job ends, reducing resource conflicts.
- **Error Handling**:
  - The program handles the case where `FR715W` does not exist in `QTEMP` by creating it. No additional error handling is implemented, relying on `FR713P` for input validation.
- **Integration**:
  - `FR715C` is part of the program chain (`FR713P` → `FR713PC` → `FR715C` → `FR715A`/`FR715B`) designed to modularize the report generation process, separating input validation, dispatching, data preparation, and printing.
- **Location Code Length**:
  - The `&P$LOC` length (LEN(2) in `FR715C` and `FR714C`, vs. LEN(3) in `FR713C`) may indicate a system-specific requirement or a potential error that should be verified.
- **Secondary Work File Naming**:
  - The use of `FR714W2` in `FR715C` (instead of, e.g., `FR715W2`) is consistent with `FR713C` and `FR714C`, suggesting a shared secondary work file across all three report variants. This may be intentional for cross-referencing data or could indicate a naming inconsistency requiring verification.

### Summary

The `FR715C` program prepares a temporary work file (`FR715W`) in `QTEMP`, clears it and a secondary work file (`FR714W2`), applies file overrides, and calls `FR715A` to populate the work file and `FR715B` to print the Freight Out Reconciliation Report sorted by product code. It enforces business rules for temporary file management and parameter consistency, relying on `FR715A` and `FR715B` for data access and report generation. The program does not directly access database tables but manages work files, with actual data retrieval handled by the called programs.