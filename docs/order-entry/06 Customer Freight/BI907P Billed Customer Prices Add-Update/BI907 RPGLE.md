The provided RPGLE program, `BI907`, is a core component of the Bradford Order Entry/Invoices system, designed for customer and ship-to file maintenance and inquiry. It is called by the `BI907C` CLP program, which sets up a temporary work file and passes parameters to `BI907`. This program manages a subfile-based interface to allow users to maintain or inquire about customer/ship-to product descriptions and freight terms. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the BI907 RPGLE Program

The `BI907` program is an interactive workstation program that uses a display file (`bi907d`) with a subfile (`sfl1`) to present and manage customer/ship-to data. It operates in either maintenance (`MNT`) or inquiry (`INQ`) mode, with functionality to add, update, or delete records, and includes features like field prompting, history inquiry, and copy functionality. The process steps are as follows:

1. **Initialization (`*inzsr` Subroutine)**:
   - Receives input parameters: `p$co` (company number), `p$cst` (customer number), `p$shp` (ship-to number), `p$mode` (run mode: `MNT` or `INQ`), and `p$fgrp` (file group: `Z` or `G`).
   - Retrieves the local data area (`lda`) to access environment information (e.g., `tstprd` for test/production indicator).
   - Sets the header (`c$hdr1`) based on the mode: "Customer & Ship To File Maint" for `MNT`, or "Customer & Ship To File Inquiry" for `INQ`.
   - Sets global protection indicator (`*in71`) to protect input fields in inquiry mode (`INQ`).
   - Initializes subfile control fields (`rrn1`, `rrnsv1`), page size (`pagsz1 = 12`), and mode flags (`c1mode`, `s1updt`, `s1f10d`).
   - Checks if a record exists in `arcupr` using the key list `kls1s1` (company, customer, ship-to):
     - If found, sets to "Update Mode" (`*in87 = *on`, `s1f10d = "F10=All Mode"`, `c1mode = "Update Mode"`).
     - If not found, sets to "All Mode" or "Add Mode" based on `p$mode` and protection settings.
   - Sets the current date and time using the `TIME` operation, formatting it into the `t#cymd` data structure.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - Applies file overrides based on the `p$fgrp` parameter (`Z` or `G`) for files `gstabl`, `arcupr`, `bicont`, `gsprod`, `arcust`, `shipto`, and `arcuphs` using the `QCMDEXC` API.
   - Opens these files with the `usropn` option for input or update, and opens `bi907w` without override (as it is in `QTEMP`).
   - Files are opened for read-only (`if`) or read/write (`uf`) access as needed.

3. **Subfile Processing (`srsfl1` Subroutine)**:
   - **Clear Message Subfile**: Calls `clrmsg` to clear messages and `wrtmsg` to write the message control record.
   - **Default Values**: Sets the default company number (`c1cono = 10`).
   - **Position File**: Calls `sf1rep` to position the file cursor based on input parameters.
   - **Suppress Initial Errors**: Clears error indicators and messages on the first display (`w$frst = *on`).
   - **Main Loop (`sf1agn`)**:
     - Repositions the subfile if needed (`repsfl = *on`) by updating control fields (`c1prod`, `c1cnty`) and calling `sf1rep`.
     - Sets cursor position for add mode (`row1 = 10`, `col1 = 02`, `*in75 = *on`) or clears it for other modes.
     - Displays the command line (`sflcmd1`) and message subfile (`msgctl` or `msgclr`).
     - Checks if the subfile has records (`rrn1 > 0`) to enable display (`*in41`).
     - Writes and formats the subfile control record (`sflctl1`) using `exfmt`.
     - Clears message subfile if errors are present.
     - Processes user input via function keys or direct access:
       - **F03**: Exits the program.
       - **F04**: Prompts for field input (calls `prompt` for control or subfile fields).
       - **F05**: Refreshes the subfile by clearing `r$cnty` and repositioning.
       - **F08**: Copies alternate descriptions (`sf1cpy`).
       - **F09**: Calls history inquiry (`histinq`).
       - **F10**: Toggles between "Add Mode," "Update Mode," and "All Mode" (`c1mode`, `s1updt`, `*in87`, `s1f10d`).
       - **ENTER**: Processes subfile changes (`sf1pro`).
     - Updates cursor location (`row1`, `col1`) and subfile record number (`rcdnb1`) for redisplay.

4. **Field Prompting (`prompt` Subroutine)**:
   - Handles prompting for fields in `SFLCTL1` or `SFL1`:
     - For `C1CUST` or `C1SHIP`: Calls `LARCUST` or `LCSTSHP` to select customer or ship-to.
     - For `S1PROD` or `C1PROD`: Calls `LGSPROD` to select a product.
     - For `S1FRCD`: Calls `LTABLE` to select a freight code.
     - For `S1CNTY`: Calls `LTABLE` to select a container type.
   - Updates fields with selected values and sets `*in19` for input change.

5. **Subfile Processing (`sf1pro` Subroutine)**:
   - Reads changed subfile records (`readc sfl1`) if `rrn1 > 0`.
   - Validates input (`sf1val`) for fields like freight code (`s1frcd`), container type (`s1cnty`), and separate freight (`s1sfrt`).
   - Updates or adds records to `arcupr` or `bi907w` based on the mode (`s1updt`).
   - Writes history records to `arcuphs` for changes or deletions.
   - Updates the subfile with formatted data (`sf1fmt`) and applies color coding (`sf1col`).

6. **Reposition Subfile (`sf1rep` Subroutine)**:
   - Clears the subfile (`sf1clr`) and resets `rrn1`.
   - Loads records into the subfile (`sf1lod`) based on key lists (`kls1s1`, `kls1s2`).
   - Filters records by company, customer, ship-to, product, and container type.

7. **Load Subfile Records (`sf1lod` Subroutine)**:
   - Reads records from `arcupr` or `bi907w` based on key lists.
   - Filters records based on mode and user input (e.g., `c1prod`, `c1cnty`).
   - Formats each record (`sf1fmt`) and applies color coding (`sf1col`).
   - Writes records to the subfile, incrementing `rrn1`.

8. **Validate Subfile Input (`sf1val` Subroutine)**:
   - Validates freight code (`s1frcd`) against `gstabl` (table type `BBFRCD`).
   - Validates container type (`s1cnty`) against `gstabl` (table type `CNTRTY`).
   - Enforces business rules for freight codes (e.g., `C` requires `s1sfrt` and `s1cafr` to be `Y`, `N`, or blank; `A` requires `s1sfrt = 'Y'`).
   - Sets error indicators and messages for invalid inputs.

9. **Copy Alternate Description (`sf1cpy` Subroutine)**:
   - Copies product descriptions from an alternate customer/ship-to pair to the current record.
   - Updates the subfile and writes history records.

10. **History Inquiry (`histinq` Subroutine)**:
    - Calls `GB730P` to display historical data for the selected customer/ship-to/product.

11. **Message Handling**:
    - **Add Message (`addmsg`)**: Sends error messages to the program message queue using `QMHSNDPM`.
    - **Write Message Subfile (`wrtmsg`)**: Displays the message subfile.
    - **Clear Message Subfile (`clrmsg`)**: Clears messages using `QMHRMVPM`.

12. **Program Termination**:
    - Closes all open files.
    - Sets `*inlr = *on` and returns.

---

### Business Rules

The `BI907` program enforces the following business rules:

1. **Mode-Based Processing**:
   - In `MNT` mode, users can add, update, or delete records; input fields are editable (`*in71 = *off`).
   - In `INQ` mode, fields are protected (`*in71 = *on`), allowing only viewing of records.

2. **Freight Code Validation** (JB01, JB02 Revisions):
   - Freight code (`s1frcd`) must exist in `gstabl` (table type `BBFRCD`).
   - For freight code `C` (freight collect):
     - Separate freight (`s1sfrt`) and calculate freight (`s1cafr`) must be `Y`, `N`, or blank.
   - For freight code `A` (non-Bradford location, e.g., Anchor):
     - Separate freight (`s1sfrt`) must be `Y`.
   - For freight code `CYY` (freight collect with service fee, JB02):
     - Represents a situation where shipping is arranged by ARG but billed to the customer by the carrier, with a $100 service fee charged to the customer.

3. **Container Type Validation**:
   - Container type (`s1cnty`) must exist in `gstabl` (table type `CNTRTY`).

4. **Record Existence and Deletion**:
   - Prevents adding a record if it already exists or is marked as deleted.
   - Allows reactivation of deleted records.
   - Records deleted or reactivated are logged in `arcuphs` for history.

5. **Mode Toggling**:
   - Users can toggle between "Add Mode," "Update Mode," and "All Mode" using F10, controlling whether new records can be added or existing records updated.

6. **File Group Flexibility**:
   - Supports different file sets (`Z` or `G`) via overrides, allowing the program to work with different data environments.

7. **Error Handling**:
   - Validates all user inputs and displays specific error messages (e.g., "Invalid Freight Code," "Must Enter At Least One Product Code in ADD Mode").
   - Ensures at least one product code is entered in add mode.

---

### Tables Used

The program uses the following database files, opened with the `usropn` option and overridden based on the file group (`Z` or `G`):
1. **gstabl**:
   - Purpose: Stores table data for freight codes (`BBFRCD`) and container types (`CNTRTY`).
   - Used in: `sf1val` to validate `s1frcd` and `s1cnty`.
   - Override: `ggstabl` (G group) or `zgstabl` (Z group).
   - Access: Input (`if`).

2. **arcupr**:
   - Purpose: Stores customer/ship-to cross-reference data.
   - Used in: `sf1rep`, `sf1lod`, `sf1pro` for reading, adding, or updating records.
   - Override: `garcupr` (G group) or `zarcupr` (Z group).
   - Access: Update (`uf a`).

3. **bicont**:
   - Purpose: Stores company information.
   - Used in: Validates company number (`c1cono`).
   - Override: `gbicont` (G group) or `zbicont` (Z group).
   - Access: Input (`if`).

4. **gsprod**:
   - Purpose: Stores product information.
   - Used in: `sf1fmt` to retrieve product descriptions (`tpdesc`).
   - Override: `ggsprod` (G group) or `zgsprod` (Z group).
   - Access: Input (`if`).

5. **arcust**:
   - Purpose: Stores customer master data.
   - Used in: `sf1val` to validate customer numbers.
   - Override: `garcust` (G group) or `zarcust` (Z group).
   - Access: Input (`if`).

6. **shipto**:
   - Purpose: Stores ship-to data.
   - Used in: `sf1val` to validate ship-to codes.
   - Override: `gshipto` (G group) or `zshipto` (Z group).
   - Access: Input (`if`).

7. **arcuphs**:
   - Purpose: Stores history records for customer/ship-to changes.
   - Used in: `sf1pro` to log changes or deletions.
   - Override: `garcuphs` (G group) or `zarcuphs` (Z group).
   - Access: Output (`o`).

8. **bi907w**:
   - Purpose: Temporary work file in `QTEMP` for processing customer/ship-to data.
   - Used in: `sf1pro`, `sf1lod` for temporary storage and processing.
   - Access: Update (`uf a`).

---

### External Programs Called

The `BI907` program calls the following external programs:
1. **LARCUST**:
   - Purpose: Prompts for customer selection.
   - Called in: `prompt` subroutine for `C1CUST`.
   - Parameters: `o$co` (company), `o$cust` (customer), `o$fgrp` (file group).

2. **LCSTSHP**:
   - Purpose: Prompts for ship-to selection.
   - Called in: `prompt` subroutine for `C1SHIP`.
   - Parameters: `x$cstshp` (data structure with company, search, customer, ship-to, flag, file group).

3. **LGSPROD**:
   - Purpose: Prompts for product selection.
   - Called in: `prompt` subroutine for `S1PROD` and `C1PROD`.
   - Parameters: `o$co` (company), `o$prod` (product), `o$fgrp` (file group).

4. **LTABLE**:
   - Purpose: Prompts for table codes (freight code or container type).
   - Called in: `prompt` subroutine for `S1FRCD` and `S1CNTY`.
   - Parameters: `o$tbcd` (table code), `o$tbty` (table type), `o$flag` (flag).

5. **GB730P**:
   - Purpose: Displays historical data for customer/ship-to/product.
   - Called in: `histinq` subroutine.
   - Parameters: `x$arcuprhist` (data structure with file, file group, company, customer, ship-to, product, container type).

6. **QCMDEXC**:
   - Purpose: Executes file override commands.
   - Called in: `opntbl` subroutine.
   - Parameters: `dbov##` (override command), `dbol##` (command length).

7. **QMHSNDPM**:
   - Purpose: Sends messages to the program message queue.
   - Called in: `addmsg` subroutine.
   - Parameters: `m@id` (message ID), `m@msgf` (message file), `m@data` (message data), `m@l` (data length), `m@type` (message type), `m@pgmq` (program queue), `m@scnt` (stack counter), `m@key` (message key), `m@errc` (error code).

8. **QMHRMVPM**:
   - Purpose: Removes messages from the program message queue.
   - Called in: `clrmsg` subroutine.
   - Parameters: `m@pgmq` (program queue), `m@scnt` (stack counter), `m@rmvk` (message key), `m@rmv` (remove option), `m@errc` (error code).

---

### Summary

The `BI907` RPGLE program is a comprehensive tool for maintaining and inquiring about customer/ship-to data within the Bradford Order Entry/Invoices system. It provides a subfile-based interface for adding, updating, or viewing records, with robust validation for freight codes, container types, and other fields. The program supports special freight scenarios (e.g., non-Bradford locations, freight collect with service fees) and logs changes to a history file. It integrates with multiple database files and external programs for prompting and history inquiry, ensuring flexibility and user interaction. The temporary work file in `QTEMP` (set up by `BI907C`) supports processing, and file overrides allow operation across different data environments (`Z` or `G`).