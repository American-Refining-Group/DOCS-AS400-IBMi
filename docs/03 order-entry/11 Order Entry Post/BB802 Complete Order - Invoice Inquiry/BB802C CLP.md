The **BB802C** CLP program is a control language program within the Customer Order and Invoice Inquiry system, designed to create and populate a work file (`BB802W`) in the `QTEMP` library based on user-specified query criteria. It is called from the main program **BB802** and processes data for open orders, canceled orders, and sales history, with support for product moves. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### **Process Steps of the BB802C CLP Program**

The program processes input parameters to determine the mode of operation, creates or clears work files, and executes queries to populate the work file with filtered data. The steps are driven by conditional logic based on the `P$MODE` parameter. Here’s a breakdown of the key process steps:

1. **Parameter Declaration**:
   - **Purpose**: Defines input parameters and variables for file handling.
   - **Actions**:
     - Declares parameters:
       - `&P$MODE` (3 characters): Mode of operation ('WRK', 'ORH', 'ORD', 'SAH', 'SAD', 'CNH', 'CND').
       - `&P$DSTC` (1 character): Destination city flag ('Q' to skip ship-to processing, otherwise process).
       - `&P$PMOV` (1 character): Product move flag ('Y' for product move, 'N' otherwise).
       - `&P$FGRP` (1 character): File group ('G' or 'Z' to determine library).
       - `&QRYSLT` (1024 characters): Query selection string for filtering data.
     - Declares variables for file names: `&FILE01`, `&FILE02`, `&FILE03` (10 characters each).

2. **Clear/Create Work Files**:
   - **Purpose**: Ensures the work file `BB802W` and related files are ready in `QTEMP`.
   - **Actions** (executed if `&P$MODE = 'WRK'`):
     - Attempts to clear the physical file `QTEMP/BB802W` using `CLRPFM`.
     - If the file is not found (`CPF3142`), creates it and related files by duplicating objects from the appropriate library:
       - If `&P$FGRP = 'G'`, duplicates from `DATA` library.
       - If `&P$FGRP ≠ 'G'`, duplicates from `DATADEV` library (per JK02).
     - Creates files: `BB802W`, `BB802WL1`, `BB802BQF`, `BB802FQF`, `BB802HQF`, `BB802IQF`, `BB802PQF`.
     - Handles errors (`CPF5813`, `CPF7302`) using `MONMSG` to continue if file creation fails.

3. **Process Open Orders - Header as Primary**:
   - **Purpose**: Queries open order header data (`BBORDH`) and populates `BB802W`.
   - **Actions** (executed if `&P$MODE = 'ORH'`):
     - Sets `&FILE01` to `&P$FGRP + 'BBORDH'` (e.g., `GBBORDH` or `ZBBORDH`).
     - Overrides `BBORDH` to `&FILE01` with `SHARE(*YES)` using `OVRDBF`.
     - Opens a query file (`OPNQRYF`) with:
       - File: `&FILE01`.
       - Query selection: `&QRYSLT`.
       - Key fields: `BOCO` (company), `BOCUST` (customer), `BORDNO` (order number), `BOINVN` (invoice number).
     - Overrides `BB802W` to `QTEMP/BB802W`.
     - Calls `BB802A` to process the query results.
     - Closes the query file (`CLOF`) and deletes overrides (`DLTOVR`).

4. **Process Open Orders - Detail as Primary**:
   - **Purpose**: Queries open order detail data (`BBORDD`) joined with header (`BBORDH`) and populates `BB802W`.
   - **Actions** (executed if `&P$MODE = 'ORD'`):
     - Sets `&FILE01` to `&P$FGRP + 'BBORDD'` and `&FILE02` to `&P$FGRP + 'BBORDH'`.
     - Overrides `BB802BQF` to `&FILE01` with `SHARE(*YES)`.
     - Opens a query file (`OPNQRYF`) with:
       - Files: `&FILE01`, `&FILE02`.
       - Format: `BB802BQF`.
       - Query selection: `&QRYSLT`.
       - Key fields: `BDCO` (company), `BOCUST` (customer), `BDORDN` (order number), `BOINVN` (invoice number).
       - Join fields: `BDCO` to `BOCO`, `BDORDN` to `BORDNO`, with `JDFTVAL(*YES)` for outer join.
     - Overrides `BB802W` to `QTEMP/BB802W`.
     - Calls `BB802B` to process the query results.
     - Closes the query file and deletes overrides.

5. **Process Sales History - Header as Primary**:
   - **Purpose**: Queries sales history header data (`SA5SHP`) and populates `BB802W`.
   - **Actions** (executed if `&P$MODE = 'SAH'`):
     - Sets `&FILE01` to `&P$FGRP + 'SA5SHP'` (e.g., `GSA5SHP` or `ZSA5SHP`).
     - Overrides `SA5SHP` to `&FILE01` with `SHARE(*YES)`.
     - Opens a query file (`OPNQRYF`) with:
       - File: `&FILE01`.
       - Query selection: `&QRYSLT`.
       - Key fields: `SHCO` (company), `SHCUST` (customer), `SHORD` (order number), `SHINV` (invoice number).
     - Overrides `BB802W` to `QTEMP/BB802W`.
     - Calls `BB802E` to process the query results.
     - Closes the query file and deletes overrides.

6. **Process Sales History - Detail as Primary (Standard)**:
   - **Purpose**: Queries sales history detail data (`SA5FILD`) joined with header (`SA5SHP`) and populates `BB802W`.
   - **Actions** (executed if `&P$MODE = 'SAD'` and `&P$PMOV ≠ 'Y'`):
     - Sets `&FILE01` to `&P$FGRP + 'SA5FILD'` and `&FILE02` to `&P$FGRP + 'SA5SHP'`.
     - Overrides `BB802FQF` to `&FILE01` with `SHARE(*YES)`.
     - Opens a query file (`OPNQRYF`) with:
       - Files: `&FILE01`, `&FILE02`.
       - Format: `BB802FQF`.
       - Query selection: `&QRYSLT`.
       - Key fields: `SACO` (company), `SACUST` (customer), `SAORD` (order number), `SAINVN` (invoice number), `SASEQ` (sequence).
       - Join fields: `SACO` to `SHCO`, `SACUST` to `SHCUST`, `SAORD` to `SHORD`, `SAINVN` to `SHINV`, with `JDFTVAL(*YES)`.
     - Overrides `BB802W` to `QTEMP/BB802W`.
     - Calls `BB802F` to process the query results.
     - Closes the query file and deletes overrides.

7. **Process Sales History - Detail as Primary (Product Move)**:
   - **Purpose**: Queries sales history move detail data (`SA5MOVD`) joined with header (`SA5SHP`) and populates `BB802W`.
   - **Actions** (executed if `&P$MODE = 'SAD'` and `&P$PMOV = 'Y'`):
     - Sets `&FILE01` to `&P$FGRP + 'SA5MOVD'` and `&FILE02` to `&P$FGRP + 'SA5SHP'`.
     - Overrides `BB802PQF` to `&FILE01` with `SHARE(*YES)`.
     - Opens a query file (`OPNQRYF`) with:
       - Files: `&FILE01`, `&FILE02`.
       - Format: `BB802PQF`.
       - Query selection: `&QRYSLT`.
       - Key fields: `SACO`, `SACUST`, `SAORD`, `SAINVN`, `SASEQ`.
       - Join fields: Same as standard sales history detail, with `JDFTVAL(*YES)`.
     - Overrides `BB802W` to `QTEMP/BB802W`.
     - Calls `BB802P` to process the query results.
     - Closes the query file and deletes overrides.

8. **Process Canceled Orders - Header as Primary**:
   - **Purpose**: Queries canceled order header data (`BBCNH`) joined with reason (`BBCNOR`) and populates `BB802W`.
   - **Actions** (executed if `&P$MODE = 'CNH'`):
     - Sets `&FILE01` to `&P$FGRP + 'BBCNH'` and `&FILE02` to `&P$FGRP + 'BBCNOR'`.
     - Overrides `BB802HQF` to `&FILE01` with `SHARE(*YES)`.
     - Opens a query file (`OPNQRYF`) with:
       - Files: `&FILE01`, `&FILE02`.
       - Format: `BB802HQF`.
       - Query selection: `&QRYSLT`.
       - Key fields: `BOCO`, `BOCUST`, `BORDNO`, `BOINVN`.
       - Join fields: `BOCO` to `BCCO`, `BORDNO` to `BCORDN`, with `JDFTVAL(*YES)`.
     - Overrides `BB802W` to `QTEMP/BB802W`.
     - Calls `BB802H` to process the query results.
     - Closes the query file and deletes overrides.

9. **Process Canceled Orders - Detail as Primary**:
   - **Purpose**: Queries canceled order detail data (`BBCND`) joined with header (`BBCNH`) and reason (`BBCNOR`) and populates `BB802W`.
   - **Actions** (executed if `&P$MODE = 'CND'`):
     - Sets `&FILE01` to `&P$FGRP + 'BBCND'`, `&FILE02` to `&P$FGRP + 'BBCNH'`, and `&FILE03` to `&P$FGRP + 'BBCNOR'`.
     - Overrides `BB802IQF` to `&FILE01` with `SHARE(*YES)`.
     - Opens a query file (`OPNQRYF`) with:
       - Files: `&FILE01`, `&FILE02`, `&FILE03`.
       - Format: `BB802IQF`.
       - Query selection: `&QRYSLT`.
       - Key fields: `BDCO`, `BOCUST`, `BDORDN`, `BOINVN`.
       - Join fields: `BDCO` to `BOCO`, `BDORDN` to `BORDNO`, `BDCO` to `BCCO`, `BDORDN` to `BCORDN`, with `JDFTVAL(*YES)`.
     - Overrides `BB802W` to `QTEMP/BB802W`.
     - Calls `BB802I` to process the query results.
     - Closes the query file and deletes overrides.

10. **Retrieve Ship-to Destination City**:
    - **Purpose**: Updates `BB802W` with ship-to destination city from `SHIPTO`.
    - **Actions** (executed if `&P$DSTC ≠ 'Q'`):
      - Sets `&FILE01` to `&P$FGRP + 'SHIPTO'`.
      - Overrides `SHIPTO` to `&FILE01` with `SHARE(*YES)`.
      - Overrides `BB802W` to `QTEMP/BB802W`.
      - Calls `BB802G` to update the work file with destination city data.
      - Deletes overrides.

11. **Program Termination**:
    - Ends the program (`ENDPGM`).

---

### **Business Rules**

1. **Work File Creation**:
   - The work file `BB802W` and related logical files (`BB802WL1`, `BB802BQF`, `BB802FQF`, `BB802HQF`, `BB802IQF`, `BB802PQF`) are created in `QTEMP` to ensure isolation and temporary storage.
   - Files are duplicated from `DATA` (for `&P$FGRP = 'G'`) or `DATADEV` (for `&P$FGRP ≠ 'G'`, per JK02) to support different environments.
   - Files are cleared or created with constraints and triggers disabled (`CST(*NO)`, `TRG(*NO)`) to optimize performance.

2. **Mode-Based Processing**:
   - The program supports seven modes (`P$MODE`):
     - **WRK**: Clears or creates work files.
     - **ORH**: Processes open order headers (`BBORDH`).
     - **ORD**: Processes open order details (`BBORDD`) with header join.
     - **SAH**: Processes sales history headers (`SA5SHP`).
     - **SAD**: Processes sales history details (`SA5FILD` or `SA5MOVD` based on `P$PMOV`).
     - **CNH**: Processes canceled order headers (`BBCNH`) with reason join.
     - **CND**: Processes canceled order details (`BBCND`) with header and reason joins.
   - Each mode uses specific files and formats, with joins and key fields tailored to the data type.

3. **File Group Handling**:
   - The `P$FGRP` parameter ('G' or 'Z') determines the library prefix for files (e.g., `GBBORDH` vs. `ZBBORDH`).
   - Overrides ensure the correct library is accessed based on the environment.

4. **Product Move Support (JK01)**:
   - If `P$PMOV = 'Y'` and `P$MODE = 'SAD'`, uses `SA5MOVD` instead of `SA5FILD` for sales history details, calling `BB802P` instead of `BB802F`.
   - This supports queries involving product moves, ensuring compatibility with move-specific data.

5. **Query Selection**:
   - The `&QRYSLT` parameter contains a dynamic query selection string built by the calling program (BB802) to filter records based on user input (e.g., company, customer, order, invoice).
   - The query is applied consistently across modes, with joins ensuring related data is included.

6. **Ship-to Destination**:
   - If `P$DSTC ≠ 'Q'`, the program retrieves ship-to destination city data from `SHIPTO` to enhance the work file.
   - This step is skipped if `P$DSTC = 'Q'`, optimizing performance when destination data is not needed.

7. **Error Handling**:
   - Uses `MONMSG` to handle file creation errors (`CPF3142`, `CPF5813`, `CPF7302`), ensuring the program continues gracefully.
   - Commented-out debug messages suggest logging capabilities for troubleshooting (e.g., to users `CAPO` or `KRAJTEST`).

8. **Revision-Specific Rules**:
   - **JB01**: Changed work file creation to copy from `ARGDEV` or `ARGDEVTEST` (superseded by JK02).
   - **JK01**: Added support for product moves, processing `SA5MOVD` when `P$PMOV = 'Y'`.
   - **JK02**: Updated library references to `DATA` and `DATADEV`, overriding work files to `QTEMP` for consistency.

---

### **Tables Used**

The program accesses the following database files, with overrides applied based on `P$FGRP`:

1. **BB802W**: Work file in `QTEMP` (primary output file).
2. **BB802WL1**: Logical work file in `QTEMP`.
3. **BB802BQF**: Logical work file for open order details in `QTEMP`.
4. **BB802FQF**: Logical work file for sales history details in `QTEMP`.
5. **BB802HQF**: Logical work file for canceled order headers in `QTEMP`.
6. **BB802IQF**: Logical work file for canceled order details in `QTEMP`.
7. **BB802PQF**: Logical work file for sales history move details in `QTEMP`.
8. **BBORDH**: Open order header file (prefixed with `G` or `Z`).
9. **BBORDD**: Open order detail file (prefixed with `G` or `Z`).
10. **SA5SHP**: Sales history header file (prefixed with `G` or `Z`).
11. **SA5FILD**: Sales history detail file (prefixed with `G` or `Z`).
12. **SA5MOVD**: Sales history move detail file (prefixed with `G` or `Z`, used if `P$PMOV = 'Y'`).
13. **BBCNH**: Canceled order header file (prefixed with `G` or `Z`).
14. **BBCND**: Canceled order detail file (prefixed with `G` or `Z`).
15. **BBCNOR**: Canceled order reason file (prefixed with `G` or `Z`).
16. **SHIPTO**: Ship-to master file (prefixed with `G` or `Z`, used if `P$DSTC ≠ 'Q'`).

---

### **External Programs Called**

The program calls the following external programs based on the mode:

1. **BB802A**: Processes open order header data (`ORH` mode).
   - Parameters: None explicitly passed (relies on file overrides and open query).
2. **BB802B**: Processes open order detail data (`ORD` mode).
   - Parameters: None explicitly passed.
3. **BB802E**: Processes sales history header data (`SAH` mode).
   - Parameters: None explicitly passed.
4. **BB802F**: Processes sales history detail data (`SAD` mode, `P$PMOV ≠ 'Y'`).
   - Parameters: None explicitly passed.
5. **BB802P**: Processes sales history move detail data (`SAD` mode, `P$PMOV = 'Y'`).
   - Parameters: None explicitly passed.
6. **BB802H**: Processes canceled order header data (`CNH` mode).
   - Parameters: None explicitly passed.
7. **BB802I**: Processes canceled order detail data (`CND` mode).
   - Parameters: None explicitly passed.
8. **BB802G**: Updates `BB802W` with ship-to destination city data (when `P$DSTC ≠ 'Q'`).
   - Parameters: None explicitly passed.

---

### **Additional Notes**

- **File Overrides**: The program uses `OVRDBF` to dynamically map files to the correct library (`DATA` or `DATADEV`) and ensure shared access (`SHARE(*YES)`).
- **Query Flexibility**: The `OPNQRYF` command allows dynamic filtering via `&QRYSLT`, supporting complex user-defined criteria from BB802.
- **Temporary Files**: All work files are created in `QTEMP` to ensure session-specific data and cleanup.
- **Join Logic**: Uses outer joins (`JDFTVAL(*YES)`) to include records even if join conditions are not met, ensuring comprehensive results.
- **Commented Code**: Debug messages and `CPYFRMQRYF` commands are commented out, indicating they were used for testing or development.

This program is a critical component of the inquiry system, enabling dynamic data filtering and work file population to support the interactive interface of **BB802**.