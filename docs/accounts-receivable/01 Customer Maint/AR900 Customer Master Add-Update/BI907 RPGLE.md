The provided document is an RPGLE program named `BI907.rpgle.txt`, called from the main OCL script `AR900.ocl36.txt` for **Customer & Ship To File Maintenance/Inquiry** within the Bradford Order Entry/Invoices system on an IBM midrange system (likely AS/400, now IBM i). This program manages customer and shipto-specific product data, including alternate descriptions, freight codes, and container types, with support for add, update, delete, and reactivate operations. Below, I explain the **process steps**, **business rules**, **tables used**, and **external programs called**.

---

### **Process Steps of the RPGLE Program**

The `BI907` program provides a subfile-based interface (`BI907D`) for maintaining or inquiring about customer and shipto product data stored in the `ARCUPR` file. It supports operations such as adding new records, updating existing records, deleting records (marking as inactive), reactivating records, and copying alternate descriptions from other customers or shiptos. The program operates in three modes: Update Mode, All Mode, and Add Mode, controlled by function keys and user input.

1. **Program Initialization (`*INZSR` Subroutine)**:
   - **Purpose**: Sets up initial parameters, opens files, and initializes the program state.
   - **Steps**:
     - Receives input parameters: company (`p$co`), customer (`p$cst`), shipto (`p$shp`), mode (`p$mode`, `MNT` for maintenance or `INQ` for inquiry), and file group (`p$fgrp`, `Z` or `G`).
     - Applies file overrides (`ovg` for `G` files, `ovz` for `Z` files) based on `p$fgrp` using `QCMDEXC`.
     - Opens database files: `GSTABL`, `ARCUPR`, `BICONT`, `GSPROD`, `ARCUST`, `SHIPTO`, `ARCUPHS`, and `BI907W`.
     - Sets the screen header (`c$hdr1`) based on mode (`MNT` or `INQ`).
     - Initializes subfile control fields (`c1cono`, `c1cust`, `c1ship`) from input parameters and sets default container type (`c1cnty = 'A'`).
     - Checks if records exist in `ARCUPR` for the company, customer, and shipto:
       - If records exist, sets Update Mode (`c1mode = 'Update Mode'`, `s1updt = *ON`, `s1f10d = 'F10=Add Mode'`).
       - If no records exist and inquiry mode is off, sets All Mode (`c1mode = 'All Mode'`, `s1f10d = 'F10=Add Mode'`).
       - If no records exist and inquiry mode is on, sets All Mode with protected fields.
     - Captures the current date and time (`t#time`) and formats it as `t#cymd` (YYYYMMDD) for history records.

2. **Main Subfile Processing (`srsfl1` Subroutine)**:
   - **Purpose**: Manages the subfile (`SFL1`) display and user interactions in a loop.
   - **Steps**:
     - Clears the message subfile and initializes the company number (`c1cono = 10`).
     - Positions the work file (`BI907W`) based on user input (`sf1rep`).
     - Suppresses errors on the first display (`w$frst`).
     - Enters a loop (`sf1agn`) to display the subfile control format (`SFLCTL1`) and process user inputs:
       - Writes the command line (`SFLCMD1`) and message subfile if needed.
       - Sets cursor position for Add Mode (`row1 = 10`, `col1 = 02`).
       - Displays the subfile if records exist (`*IN41`) and control format (`*IN40`).
       - Processes function keys and user actions (see below).
       - Updates cursor location (`csrloc`) and subfile record number (`rcdnb1`) for redisplay.

3. **Function Key Processing**:
   - **F03 (Exit)**: Exits the program by clearing flags (`sf1agn`, `fmtagn`) and iterating.
   - **F04 (Field Prompting)**:
     - For `SFLCTL1`: Prompts for product code (`C1PROD`) or container type (`C1CNTY`) using `LGSPROD` or `LGSTABL`.
     - For `SFL1`: Prompts for product code (`S1PROD`) or container type (`S1CNTY`) and updates subfile fields.
     - For `SFLCCPY`: Prompts for customer (`S3CUST`) or shipto (`S3SHIP`) using `LARCUST` or `LCSTSHP`.
   - **F05 (Refresh)**: Clears container type (`r$cnty`) and repositions the subfile (`repsfl`).
   - **F08 (Copy Alternate Description)**:
     - Opens a window (`SFLCCPY`) to input source customer (`s3cust`) and shipto (`s3ship`).
     - Validates inputs against `ARCUST` and `SHIPTO`.
     - Calls `BI9078` to copy alternate descriptions, updates the message subfile, and repositions the subfile.
   - **F09 (History Inquiry)**:
     - Calls `GB730P` to display history for the selected subfile record (`SFL1`) using parameters (`o$file = 'ARCUPR'`, `o$fgrp`, `c1cono`, `c1cust`, `c1ship`, `s1prod`, `s1cnty`).
   - **F10 (Toggle Mode)**:
     - Toggles between Update Mode, All Mode, and Add Mode:
       - Update Mode: Shows existing `ARCUPR` records, allows updates (`s1updt = *ON`, `s1f10d = 'F10=Add Mode'`).
       - All Mode: Shows all products from `GSPROD`, allows adding new records (`s1updt = *OFF`, `s1f10d = 'F10=Update Mode'`).
       - Add Mode: Allows adding new records (`s1updt = *ON`, `s1f10d = 'F10=All Mode'`).
     - Repositions the subfile after mode change.
   - **F12 (Cancel)**: Exits the subfile loop.
   - **F22 (Reactivate)**:
     - For a selected subfile record (`SFL1`) marked as deleted (`s1del = 'D'`, `s1exis = 'Y'`), opens a window (`SFLRST1`).
     - Sets `cpdel = 'A'` in `ARCUPR`, updates the record, writes to history (`ARCUPHS`), and clears the subfile record.
   - **F23 (Delete)**:
     - For a selected subfile record (`SFL1`, `s1exis = 'Y'`, `s1del ≠ 'D'`), opens a window (`SFLDEL1`).
     - Sets `cpdel = 'D'` in `ARCUPR`, updates the record, writes to history, and clears the subfile record.
   - **Page Down**: Loads additional subfile records (`sf1lod`).
   - **Enter**: Processes subfile changes (`sf1prc`) or repositions the subfile if control fields (`c1prod`, `c1cnty`, `c$dlyn`) change.

4. **Subfile Processing (`sf1prc` Subroutine)**:
   - Reads changed subfile records (`readc SFL1`) and processes them (`sf1chg`).
   - Sets `s1chng` if changes are detected.

5. **Subfile Change Processing (`sf1chg` Subroutine)**:
   - Validates subfile input (`sf1edt`).
   - If no errors (`*IN50 = *OFF`) and not in inquiry mode, updates or adds records to `ARCUPR` (`sf1upd`).
   - Updates the subfile record (`sf1pro`) and sets `SFLNXTCHG` (`*IN44`) if errors occur.

6. **Subfile Input Validation (`sf1edt` Subroutine)**:
   - Validates subfile fields:
     - **Product Code (`s1prod`)**: Required in Add Mode; must exist in `GSPROD`.
     - **Existing Record (`s1exis`)**: In Add Mode, checks if the record already exists in `ARCUPR`.
     - **Container Type (`s1cnty`)**: Must exist in `GSTABL` (`CNTRTY` table) if not blank.
     - **Gallons Billed Code (`s1glcd`)**: Must be `'G'` or blank.
     - **Freight Code (`s1frcd`)**: Must exist in `GSTABL` (`BBFRCD` table) if not blank.
     - **Separate Freight Code (`s1sfrt`)**: Must be `'Y'`, `'N'`, or blank.
     - **Calculate Freight Code (`s1cafr`)**: Must be `'Y'`, `'N'`, or blank.
     - **Freight Code Rules**:
       - If `s1frcd = 'C'` (collect), `s1sfrt` and `s1cafr` must be `'Y'`, `'N'`, or blank (per `JB01`, `JB02`).
       - If `s1frcd = 'P'` (prepaid), defaults `s1sfrt = 'N'`, `s1cafr = 'Y'` if blank.
       - If `s1frcd = 'A'` (prepaid & add), `s1sfrt` must be `'Y'`, defaults `s1cafr = 'Y'` if blank.
   - Sets error indicators (`*IN50`–`*IN57`, `*IN61`, `*IN62`) and adds error messages to the message subfile if validations fail.

7. **Update/Add to Database (`sf1upd` Subroutine)**:
   - If `s1prod` is not blank:
     - Checks if the record exists in `ARCUPR` (`klsfl1`).
     - If it does not exist (`*IN99 = *ON`) and fields (`s1glcd`, `s1cpds`, `s1frcd`, `s1sfrt`, `s1cafr`) are not blank:
       - Clears `ARCUPR` record, sets `cpdel = 'A'`, and populates fields from subfile (`sf1mov`).
       - Writes a new record to `ARCUPR`.
       - Writes a history record to `ARCUPHS`.
       - Sets `s1exis = 'Y'`.
     - If it exists:
       - Updates `ARCUPR` with subfile values (`sf1mov`).
       - Writes a history record to `ARCUPHS`.
       - Sets `s1exis = 'Y'`.

8. **Move Subfile Values to File (`sf1mov` Subroutine)**:
   - Moves subfile fields (`s1cpds`, `s1glcd`, `s1frcd`, `s1sfrt`, `s1cafr`) to `ARCUPR` fields (`cpcpds`, `cpglcd`, `cpfrcd`, `cpsfrt`, `cpcafr`).

9. **Reactivate Record (`sf1rst` Subroutine)**:
   - Displays a window (`SFLRST1`) to confirm reactivation.
   - If `F22` is pressed and the record exists in `ARCUPR`:
     - Sets `cpdel = 'A'`, updates `ARCUPR`, writes to `ARCUPHS`, and displays a message (`err(11)`).
     - Clears the subfile record.

10. **Delete Record (`sf1del` Subroutine)**:
    - Displays a window (`SFLDEL1`) to confirm deletion.
    - If `F23` is pressed and the record exists in `ARCUPR`:
      - Sets `cpdel = 'D'`, updates `ARCUPR`, writes to `ARCUPHS`, and displays a message (`err(10)`).
      - Clears the subfile record.

11. **Copy Alternate Description (`sf1cpy` Subroutine)**:
    - Displays a window (`SFLCCPY`) to input source customer (`s3cust`) and shipto (`s3ship`).
    - Validates inputs against `ARCUST` and `SHIPTO`.
    - Calls `BI9078` to copy alternate descriptions, passing `c1cono`, `c1cust`, `c1ship`, `s3cust`, `s3ship`, and `p$fgrp`.
    - Displays a success message (`err(12)`).

12. **Load Subfile (`sf1lod` Subroutine)**:
    - Loads up to 12 records (`pagsz1`) from `BI907W` into the subfile (`SFL1`).
    - Filters records based on mode (`s1updt`), container type (`c1cnty`), and include deleted flag (`c$dlyn`).
    - Formats each record (`sf1fmt`) and writes to the subfile.

13. **Build Work File (`ArcuprSrBld` and `GsprodSrBld` Subroutines)**:
    - **ArcuprSrBld**: Builds `BI907W` from `ARCUPR` for Update Mode, adding missing container types from `GSTABL`.
    - **GsprodSrBld**: Builds `BI907W` from `GSPROD` for All Mode, including sellable products (`tpsell = 'Y'`) and container types, merging with `ARCUPR` data if available.

14. **Clear Work File (`ClrWrkFile` Subroutine)**:
    - Closes `BI907W`, calls `BI907C2` to clear it, and reopens it.

15. **Write History (`writehist` Subroutine)**:
    - Writes a record to `ARCUPHS` with fields from `ARCUPR`, current date (`t#cymd`), time (`t#hms`), and user ID (`ps#usr8`).

16. **Program Termination**:
    - Closes all files, sets `*INLR = *ON`, and returns.

---

### **Business Rules**

The program enforces the following business rules to ensure data integrity:

1. **Mode-Based Operations**:
   - **Update Mode**: Displays existing `ARCUPR` records for the specified company, customer, and shipto; allows updates, deletions, and reactivations.
   - **All Mode**: Displays all sellable products from `GSPROD`, allowing new record additions.
   - **Add Mode**: Enables adding new records to `ARCUPR` with validated fields.
   - Inquiry mode (`p$mode = 'INQ'`) protects all input fields (`*IN71`).

2. **Field Validations**:
   - **Product Code (`s1prod`)**: Mandatory in Add Mode; must exist in `GSPROD`.
   - **Container Type (`s1cnty`)**: Must exist in `GSTABL` (`CNTRTY`) if not blank.
   - **Gallons Billed Code (`s1glcd`)**: Must be `'G'` or blank.
   - **Freight Code (`s1frcd`)**: Must exist in `GSTABL` (`BBFRCD`) if not blank (e.g., `C` for collect, `P` for prepaid, `A` for prepaid & add).
   - **Separate Freight Code (`s1sfrt`)**: Must be `'Y'`, `'N'`, or blank.
   - **Calculate Freight Code (`s1cafr`)**: Must be `'Y'`, `'N'`, or blank.
   - **Freight Code Rules**:
     - For `s1frcd = 'C'` (collect):
       - `s1sfrt` and `s1cafr` must be `'Y'`, `'N'`, or blank (per `JB01`, `JB02`).
       - Defaults: `s1sfrt = 'N'`, `s1cafr = 'N'` if blank.
     - For `s1frcd = 'P'` (prepaid):
       - Defaults: `s1sfrt = 'N'`, `s1cafr = 'Y'` if blank.
     - For `s1frcd = 'A'` (prepaid & add):
       - `s1sfrt` must be `'Y'`.
       - Defaults: `s1cafr = 'Y'` if blank.
     - `JB01`: Allows `s1cafr = 'Y'` for collect (non-Bradford locations, e.g., Anchor).
     - `JB02`: Allows `s1sfrt = 'Y'` for collect with a $100 service fee when ARG arranges shipping.

3. **Record Existence**:
   - Prevents adding a record if it already exists in `ARCUPR` or is marked as deleted.
   - Deletion marks records as inactive (`cpdel = 'D'`) rather than physically deleting.
   - Reactivation changes `cpdel` from `'D'` to `'A'`.

4. **Copy Alternate Descriptions**:
   - Source customer and shipto must exist in `ARCUST` and `SHIPTO`.
   - Copied descriptions are applied via the `BI9078` program.

5. **History Tracking**:
   - All add, update, delete, and reactivate operations are logged to `ARCUPHS` with date, time, and user ID.

6. **Subfile Filters**:
   - In Update Mode, filters by container type (`c1cnty`) and excludes deleted records unless `c$dlyn = 'Y'`.
   - In All Mode, includes all sellable products from `GSPROD`.

---

### **Tables (Files) Used**

The program uses the following files, as defined in the File Specification (`F`) section:

1. **BI907D** (`CF`, Workstation, Update/Input):
   - Display file with subfile `SFL1` and control format `SFLCTL1`, using the `PROFOUNDUI` handler.
   - Includes formats for reactivation (`SFLRST1`), deletion (`SFLDEL1`), and copy (`SFLCCPY`).

2. **GSTABL** (`IF`, Input, Keyed, User Open):
   - General system table for validating container types (`CNTRTY`) and freight codes (`BBFRCD`).

3. **BICONT** (`IF`, Input, Keyed, User Open):
   - Billing contact file for validating company codes and retrieving company names (`bcname`).

4. **GSPROD** (`IF`, Input, Keyed, User Open):
   - Product file for validating product codes and retrieving descriptions (`tpabds`).

5. **SHIPTO** (`IF`, Input, Keyed, User Open):
   - Shipto file for validating shipto codes and retrieving names (`csname`).

6. **ARCUST** (`IF`, Input, Keyed, User Open):
   - Customer master file for validating customer codes and retrieving names (`arname`).

7. **BI907W** (`UF`, Update/Add, Keyed, User Open):
   - Work file for temporary storage of subfile data, built from `ARCUPR` or `GSPROD`.

8. **ARCUPR** (`UF`, Update/Add, Keyed, User Open):
   - Customer product file for storing product-specific data (e.g., `cpdel`, `cpcono`, `cpcust`, `cpship`, `cpprod`, `cpcnty`, `cpcpds`, `cpglcd`, `cpfrcd`, `cpsfrt`, `cpcafr`).

9. **ARCUPHS** (`O`, Output/Add, Keyed, User Open):
   - Customer product history file for logging changes (`ahdel`, `ahcono`, `ahcust`, `ahship`, `ahprod`, `ahcnty`, `ahcpds`, `ahglcd`, `ahfrcd`, `ahsfrt`, `ahcafr`, `ahchd8`, `ahchtm`, `ahuser`).

---

### **External Programs Called**

The program calls the following external programs:

1. **BI9078**:
   - Called in `sf1cpy` to copy alternate product descriptions from a source customer/shipto to the target.
   - Parameters: `c1cono`, `c1cust`, `c1ship` (target), `s3cust`, `s3ship` (source), `p$fgrp`.

2. **BI907C2**:
   - Called in `ClrWrkFile` to clear the work file `BI907W`.
   - No parameters specified.

3. **GB730P**:
   - Called in `histinq` for history inquiries on `ARCUPR` records.
   - Parameters: `x$arcuprhist` (structure with `o$file = 'ARCUPR'`, `o$fgrp`, `c1cono`, `c1cust`, `c1ship`, `s1prod`, `s1cnty`).

4. **LGSPROD**:
   - Called in `prompt` for product code prompting.
   - Parameters: `c1cono`, `s1prod` or `c1prod`, `p$fgrp`.

5. **LGSTABL**:
   - Called in `prompt` for container type prompting.
   - Parameters: `k$ctyp = 'CNTRTY'`, `k$cnty`, `p$fgrp`.

6. **LARCUST**:
   - Called in `prompt` for customer prompting.
   - Parameters: `c1cono`, `o$cust`, `p$fgrp`.

7. **LCSTSHP**:
   - Called in `prompt` for shipto prompting.
   - Parameters: `x$cstshp` (structure with `x$co`, `x$srch`, `x$cust`, `x$ship`, `x$flag`, `x$fgrp`).

8. **QCMDEXC**:
   - Called in `opntbl` to execute file override commands (`ovg` or `ovz`).
   - Parameters: `dbov##` (override command), `dbol##` (length).

9. **QMHSNDPM**:
   - Called in `addmsg` to send error messages to the program message queue.
   - Parameters: `m@id`, `m@msgf`, `m@data`, `m@l`, `m@type`, `m@pgmq`, `m@scnt`, `m@key`, `m@errc`.

10. **QMHRMVPM**:
    - Called in `clrmsg` to clear the message subfile.
    - Parameters: `m@pgmq`, `m@scnt`, `m@rmvk`, `m@rmv`, `m@errc`.

---

### **Summary**

- **Process Steps**: Initializes parameters and files, displays a subfile (`SFL1`) for maintenance/inquiry, processes user inputs (F03, F04, F05, F08, F09, F10, F12, F22, F23, Page Down, Enter), validates subfile inputs, adds/updates/deletes/reactivates records in `ARCUPR`, logs changes to `ARCUPHS`, and supports copying alternate descriptions via `BI9078`.
- **Business Rules**: Enforces mode-based operations (Update, All, Add, Inquiry); validates product codes, container types, freight codes, and related fields; supports special freight scenarios (`JB01`, `JB02`); prevents adding existing/deleted records; logs all changes; and filters subfile data based on user inputs.
- **Tables Used**: `BI907D`, `GSTABL`, `BICONT`, `GSPROD`, `SHIPTO`, `ARCUST`, `BI907W`, `ARCUPR`, `ARCUPHS`.
- **External Programs Called**: `BI9078`, `BI907C2`, `GB730P`, `LGSPROD`, `LGSTABL`, `LARCUST`, `LCSTSHP`, `QCMDEXC`, `QMHSNDPM`, `QMHRMVPM`.

This RPGLE program is a comprehensive tool for managing customer and shipto product data, integrating with other system components to ensure accurate and validated data maintenance. If you need further details on specific subroutines or validations, let me know!