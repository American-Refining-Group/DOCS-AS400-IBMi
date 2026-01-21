The provided CLP (Control Language Program) named `SA717C` is called by the RPGLE program `SA717P` to build a sales analysis work file and run a query (`SA717Q`). Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the SA717C Program

The `SA717C` program is designed to prepare a temporary work file (`SA717QF`) by joining multiple files, applying a query selection string, and running a predefined query to generate sales analysis data. Here’s a step-by-step breakdown of the process:

1. **Program Declaration and Parameters**:
   - **Purpose**: Defines the program and its input parameters.
   - **Steps**:
     - Declares the program with three parameters:
       - `&P$FGRP` (1 character): Group identifier ('G' or 'Z').
       - `&QRYSLT` (1024 characters): Query selection string passed from `SA717P` for filtering data.
       - `&P$TYPE` (1 character): Data type ('S' for sales, 'P' for product moves).
     - Declares variables for file names:
       - `&FILE01` (10 characters): Primary file (either `GSA5FILD`, `ZSA5FILD`, `GSA5MOVD`, or `ZSA5MOVD`).
       - `&FILE02` (10 characters): Secondary file (`GSA5SHP` or `ZSA5SHP`).
       - `&FILE03` (10 characters): Tertiary file (`GSHIPTO` or `ZSHIPTO`).

2. **Determine File Names Based on Parameters**:
   - **Purpose**: Assigns file names based on the input parameters `&P$TYPE` and `&P$FGRP`.
   - **Steps**:
     - If `&P$TYPE` = 'S' (sales):
       - Sets `&FILE01` to `&P$FGRP` concatenated with `SA5FILD` (e.g., `GSA5FILD` or `ZSA5FILD`).
     - If `&P$TYPE` = 'P' (product moves):
       - Sets `&FILE01` to `&P$FGRP` concatenated with `SA5MOVD` (e.g., `GSA5MOVD` or `ZSA5MOVD`).
     - Sets `&FILE02` to `&P$FGRP` concatenated with `SA5SHP` (e.g., `GSA5SHP` or `ZSA5SHP`).
     - Sets `&FILE03` to `&P$FGRP` concatenated with `SHIPTO` (e.g., `GSHIPTO` or `ZSHIPTO`).

3. **Copy Query Format File to QTEMP**:
   - **Purpose**: Creates a temporary copy of the query format file (`SA717QF`) in the `QTEMP` library.
   - **Steps**:
     - If `&P$FGRP` = 'G':
       - Copies `SA717QF` from library `DATA` to `QTEMP/SA717QF` using `CPYF` (Copy File) with `MBROPT(*REPLACE)` to overwrite any existing file and `CRTFILE(*YES)` to create the file if it doesn’t exist.
       - Monitors for `CPF2817` (copy failure, e.g., if the file doesn’t exist) to prevent program termination.
     - If `&P$FGRP` ≠ 'G' (assumed to be 'Z'):
       - Copies `SA717QF` from library `DATADEV` to `QTEMP/SA717QF` with the same options.
     - The temporary file in `QTEMP` is used to store the query results.

4. **Override and Open Query File**:
   - **Purpose**: Configures the database query environment by overriding the file and applying the query selection.
   - **Steps**:
     - Overrides the file `SA717QF` to point to `&FILE01` (e.g., `GSA5FILD` or `GSA5MOVD`) with `SHARE(*YES)` to allow shared access.
     - Executes `OPNQRYF` (Open Query File) to join three files:
       - `&FILE01` (primary file, e.g., `GSA5FILD` or `GSA5MOVD`).
       - `&FILE02` (e.g., `GSA5SHP` or `ZSA5SHP`).
       - `&FILE03` (e.g., `GSHIPTO` or `ZSHIPTO`).
     - Specifies the output format as `SA717QF`.
     - Applies the query selection string (`&QRYSLT`) passed from `SA717P`.
     - Defines key fields for sorting: `SACO` (company), `SACUST` (customer), `SASHIP` (ship-to).
     - Specifies join conditions:
       - `&FILE01/SACO` = `&FILE02/SHCO` (company match).
       - `&FILE01/SACUST` = `&FILE02/SHCUST` (customer match).
       - `&FILE01/SAINVN` = `&FILE02/SHINV` (invoice number match).
       - `&FILE01/SAORD` = `&FILE02/SHORD` (order number match).
       - `&FILE01/SACO` = `&FILE03/CSCO` (company match).
       - `&FILE01/SACUST` = `&FILE03/CSCUST` (customer match).
       - `&FILE01/SASHIP` = `&FILE03/CSSHIP` (ship-to match).
     - Uses `JDFTVAL(*YES)` to include records with missing join matches (left outer join).

5. **Copy Query Results to Output File**:
   - **Purpose**: Transfers the query results to a physical file for further processing.
   - **Steps**:
     - Uses `CPYFRMQRYF` (Copy From Query File) to copy the results of the open query (`&FILE01`) to `SA717QRYF` in the `*LIBL` library.
     - Specifies `MBROPT(*REPLACE)` to overwrite the target file and `CRTFILE(*NO)` to assume the target file exists.

6. **Run the Query**:
   - **Purpose**: Executes the predefined query `SA717Q` to generate the sales analysis report.
   - **Steps**:
     - Runs the query `GSSQRY/SA717Q` using `RUNQRY`, which processes the data in `SA717QRYF`.

7. **Cleanup and Termination**:
   - **Purpose**: Closes the query file and removes overrides.
   - **Steps**:
     - Closes the open query file (`&FILE01`) using `CLOF` (Close File).
     - Deletes all file overrides using `DLTOVR FILE(*ALL)`.
     - Ends the program with `ENDPGM`.

---

### Business Rules

The program enforces the following business rules:
1. **File Selection Based on Data Type**:
   - If `&P$TYPE` = 'S', the primary file is `GSA5FILD` or `ZSA5FILD` (sales data).
   - If `&P$TYPE` = 'P', the primary file is `GSA5MOVD` or `ZSA5MOVD` (product movement data).
   - This ensures the correct dataset is used based on whether the user wants sales or product movement analysis.

2. **Group-Based Library Selection**:
   - If `&P$FGRP` = 'G', the source library for `SA717QF` is `DATA`.
   - If `&P$FGRP` = 'Z', the source library is `DATADEV`.
   - This supports different environments or datasets (e.g., production vs. development).

3. **Query Filtering**:
   - The query selection string (`&QRYSLT`) from `SA717P` filters records based on conditions like non-deleted records (`SADEL *NE "D"`), specific freight codes, carrier codes, and date ranges.
   - The join conditions ensure data consistency across the primary file (`&FILE01`), shipment file (`&FILE02`), and ship-to file (`&FILE03`) by matching company, customer, invoice, order, and ship-to fields.

4. **Error Handling**:
   - Monitors for `CPF2817` during the `CPYF` operation to handle cases where the source file may not exist or other copy errors occur, ensuring the program continues execution.

5. **Data Output**:
   - The query results are stored in `SA717QRYF`, which is overwritten each time the program runs.
   - The `SA717Q` query processes this file to produce the final sales analysis report.

---

### Tables Used

The program interacts with the following files (tables):

1. **SA717QF**:
   - **Type**: Physical file (PF).
   - **Library**: `DATA` (for `&P$FGRP` = 'G') or `DATADEV` (for `&P$FGRP` = 'Z'), copied to `QTEMP/SA717QF`.
   - **Usage**: Temporary query format file that defines the structure for the joined query output.
   - **Access**: Copied to `QTEMP` and used as the format for `OPNQRYF`.

2. **GSA5FILD** or **ZSA5FILD**:
   - **Type**: Physical file (PF).
   - **Library**: `DATA` or `DATADEV`.
   - **Usage**: Primary file for sales data when `&P$TYPE` = 'S'.
   - **Fields**: Includes `SACO` (company), `SACUST` (customer), `SAINV` (invoice), `SAORD` (order), `SASHIP` (ship-to), `SADEL` (delete flag), `SAIND8` (date field).
   - **Access**: Joined in `OPNQRYF`.

3. **GSA5MOVD** or **ZSA5MOVD**:
   - **Type**: Physical file (PF).
   - **Library**: `DATA` or `DATADEV`.
   - **Usage**: Primary file for product movement data when `&P$TYPE` = 'P'.
   - **Fields**: Similar to `SA5FILD`, with fields like `SACO`, `SACUST`, `SAINV`, `SAORD`, `SASHIP`.
   - **Access**: Joined in `OPNQRYF`.

4. **GSA5SHP** or **ZSA5SHP**:
   - **Type**: Physical file (PF).
   - **Library**: `DATA` or `DATADEV`.
   - **Usage**: Secondary file containing shipment data.
   - **Fields**: Includes `SHCO` (company), `SHCUST` (customer), `SHINV` (invoice), `SHORD` (order).
   - **Access**: Joined in `OPNQRYF`.

5. **GSHIPTO** or **ZSHIPTO**:
   - **Type**: Physical file (PF).
   - **Library**: `DATA` or `DATADEV`.
   - **Usage**: Tertiary file containing ship-to data.
   - **Fields**: Includes `CSCO` (company), `CSCUST` (customer), `CSSHIP` (ship-to).
   - **Access**: Joined in `OPNQRYF`.

6. **SA717QRYF**:
   - **Type**: Physical file (PF).
   - **Library**: `*LIBL` (library list).
   - **Usage**: Output file for the query results from `OPNQRYF`.
   - **Access**: Populated by `CPYFRMQRYF` and used by the `SA717Q` query.

---

### External Programs Called

The program does not directly call external programs but executes the following CL commands that invoke IBM i system utilities:

1. **CPYF** (Copy File):
   - Copies the `SA717QF` file from `DATA` or `DATADEV` to `QTEMP`.
   - Monitors for `CPF2817` to handle copy errors.

2. **OPNQRYF** (Open Query File):
   - Performs a dynamic join and query on `&FILE01`, `&FILE02`, and `&FILE03` using the `&QRYSLT` selection string.

3. **CPYFRMQRYF** (Copy From Query File):
   - Copies the query results to `SA717QRYF`.

4. **RUNQRY** (Run Query):
   - Executes the predefined query `GSSQRY/SA717Q` to process the data in `SA717QRYF`.

5. **CLOF** (Close File):
   - Closes the open query file (`&FILE01`).

6. **DLTOVR** (Delete Override):
   - Removes file overrides.

---

### Summary

The `SA717C` CLP program is a critical component of the sales analysis process initiated by `SA717P`. It:
- Dynamically selects input files based on the group (`&P$FGRP`) and data type (`&P$TYPE`).
- Copies a query format file to `QTEMP` and joins three files (`SA5FILD`/`SA5MOVD`, `SA5SHP`, `SHIPTO`) using `OPNQRYF`.
- Applies the query selection string to filter data, stores results in `SA717QRYF`, and runs the `SA717Q` query to generate the sales analysis report.
- Cleans up by closing files and removing overrides.

This program supports flexible data processing for sales or product movement analysis across different datasets, with robust error handling and environment-specific library usage. If you need further clarification or analysis, let me know!