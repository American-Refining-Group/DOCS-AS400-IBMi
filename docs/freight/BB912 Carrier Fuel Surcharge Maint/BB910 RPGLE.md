The RPG program `BB910` is part of the Brandford Order Entry/Invoices system and is designed for the maintenance and inquiry of the National Diesel Fuel Index. It is called from the `BB912` program (via the F07 key) to manage fuel index data, allowing users to add, update, or delete records in maintenance mode ('MNT') or view them in inquiry mode ('INQ'). The program uses a display file with two subfiles (`SFL1` and `SFL2`) to interact with users, displaying header and detail records respectively. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called .

### Process Steps of the BB910 RPG Program

1. **Initialization (*INZSR Subroutine)**:
   - **Receive Parameters**: Accepts input parameters: company code (`a$co`), run mode (`p$mode`: 'MNT' for maintenance or 'INQ' for inquiry), and file group (`p$fgrp`: 'G' or 'Z').
   - **Set Defaults**: If the company code is blank or zero, defaults to 10. Moves the company code to the control field `c1co`.
   - **Initialize Fields**: Sets up subfile control fields (`rrn1`, `rrn2`, `rrnsv1`, `rrnsv2`), page sizes (`pagsz1`, `pagsz2` set to 28), and message handling fields. Initializes date and time stamps using the system date (`ps#mdy`) and time (`time12`).
   - **Configure Headers**: Sets the display header (`c$hdr1`) to "National Diesel Fuel Index Maintenance" or "Inquiry" based on the mode. Sets field protection indicator `*IN70` to `*ON` for inquiry mode to prevent updates.
   - **Define Key Lists**: Sets up key lists (`kls1s1`, `kls1r1`, `klnxt1`, `klsfl1`, `kls2a1`, `kls2a2`, `kls2r1`, `kls2r2`, `klsfl2`, `klregn`) for database access.

2. **Open Database Tables (opntbl Subroutine)**:
   - **File Overrides**: Applies file overrides based on the file group (`p$fgrp`) to point to the correct library ('G' or 'Z') for files like `bicont`, `gstabl`, `bbndfi`, etc., using the `QCMDEXC` program to execute override commands.
   - **Open Files**: Opens input files (`bicont`, `gstabl`, `bbndfi1`, `bbndfird`), update file (`bbndfi`), and history output file (`bbndfih`).

3. **Process Subfile 1 (srsfl1 Subroutine)**:
   - **Clear Message Subfile**: Calls `clrmsg` to clear the message subfile and `wrtmsg` to write any messages.
   - **Default Company**: Sets `c1co` to 10 if not provided.
   - **Position File**: Calls `sf1rep` to position the `bbndfi1` file based on user input (company code and effective date).
   - **Suppress Initial Errors**: On the first display (`w$frst = *ON`), clears errors to present a clean interface and resets `w$frst`.
   - **Main Loop**:
     - Repositions the subfile if requested (`repsfl = *ON`).
     - Writes the command line (`sflcmd1`) and displays the message subfile if needed (`dspmsg = *ON`).
     - Checks for existing subfile records to enable/disable display control (`*IN41`).
     - Displays the subfile control record (`sflctl1`) using `EXFMT`.
     - Clears the message subfile if displayed.
     - Determines cursor location (`csrloc`) and sets the subfile record number (`rcdnb1`) to the current page (`pagrrn`).
     - **Processes User Input (Before Subfile Read)**:
       - **F03**: Exits the program by setting `sf1agn` to `*OFF`.
       - **F05**: Refreshes the subfile, clearing positioning fields (`r$emdy`, `c1emdy`, `d1opt`, `d1emdy`, `d1eftm`).
       - **Direct Access (d1opt, d1emdy, or d1eftm non-zero)**: Calls `sf1dir` for direct processing.
       - **Page Down**: Loads additional subfile records (`sf1lod`).
     - **Processes Enter Key**: Calls `sf1prc` to process subfile changes.
     - **Processes User Input (After Subfile Read)**:
       - **Position Request**: If the company code (`c1co`) or effective date (`c1emdy`) changes, repositions the subfile (`sf1rep`).
       - **F10**: Positions the cursor to the control record by clearing `row1` and `col1`.

4. **Process Subfile 1 Records on Enter (sf1prc Subroutine)**:
   - Reads changed subfile records (`readc sfl1`) and processes each changed record (`sf1chg`).
   - Sets a change flag (`s1chng`) to track modifications.

5. **Process Subfile 1 Record Change (sf1chg Subroutine)**:
   - Retains selected values (effective date `s1efdt`, time `s1eftm`, and month/day/year `s1emdy`) for subfile 2 processing.
   - Processes subfile options:
     - **Option 2 (Change)**: In maintenance mode, calls `sf1s02` to update records.
     - **Option 5 (Display)**: Calls `sf1s05` to display detail records.
   - Updates the subfile record by clearing the option field (`s1opt`), retrieving the occurrence count (`rtvent#`), and updating `s1occr`.

6. **Reposition Subfile 1 (sf1rep Subroutine)**:
   - Clears the subfile (`sf1clr`) and resets `rrn1`.
   - Validates control record input (`sf1cte`):
     - Checks company code against `bicont` (error `ERR0021` if invalid).
     - Validates effective date using `GSDTEDIT` (error `ERR0020` if invalid).
   - Positions the `bbndfi1` file using `kls1s1` and loads records (`sf1lod`).
   - Retains control fields for repositioning (`r$co`, `h$co`, `r$emdy`).

7. **Load Subfile 1 Records (sf1lod Subroutine)**:
   - Loads records from `bbndfi1` into the subfile up to the page size (`pagsz1`).
   - Formats each record (`sf1fmt`):
     - Sets effective date (`s1efdt`), time (`s1eftm`), and month/day/year (`s1emdy`).
     - Retrieves occurrence count (`rtvent#`) for the effective date and time.
   - Writes records to the subfile and updates `rrn1` and `rrnsv1`.

8. **Direct Access Processing (sf1dir Subroutine)**:
   - Validates effective date (`d1emdy`) using `GSDTEDIT` and effective time (`d1eftm`) for valid hours (00-23) and minutes (00-59) (error `ERR0019` if invalid).
   - Calls `sf1s01` (create), `sf1s02` (change), or `sf1s05` (display) based on the option (`d1opt`).

9. **Process Subfile 2 (sf1s02 and sf1s05 Subroutines)**:
   - **sf1s02 (Change)**: Sets maintenance mode (`s2mode = 'UPD'`), allows updates, and calls `srsfl2`.
   - **sf1s05 (Display)**: Sets inquiry mode (`s2mode = 'INQ'`), protects fields, and calls `srsfl2`.

10. **Process Subfile 2 (srsfl2 Subroutine)**:
    - Sets protection indicators (`*IN71`) based on mode.
    - Initializes to update mode and sets function key labels (`s2f06d` to "Review Mode" or "All Mode").
    - Main loop processes user input:
      - **F03**: Exits the program.
      - **F05**: Refreshes the subfile.
      - **F06**: Toggles between all regions (`s2all = *ON`) and review mode (`s2all = *OFF`).
      - **F09**: Calls `GB730P` for historical inquiry of detail records.
      - **F12**: Returns to subfile 1.
      - **F23**: Deletes a detail record.
      - **Page Down**: Loads additional detail records (`sf2lod`).
      - **Enter**: Processes subfile changes (`sf2prc`).

11. **Process Subfile 2 Records on Enter (sf2prc Subroutine)**:
    - Reads changed subfile records (`readc sfl2`) and processes each changed record (`sf2chg`).

12. **Process Subfile 2 Record Change (sf2chg Subroutine)**:
    - Validates input (`sf2edt`): Currently empty, indicating minimal validation for detail records.
    - Updates the database (`sf2upd`) if no errors and in maintenance mode.
    - Applies protection schemes (`sf2pro`) and updates the subfile record.

13. **Update Database (sf2upd Subroutine)**:
    - **Add Record**: If the record does not exist (`*IN99 = *ON`) and a price (`s2rprc`) is provided, adds a new record to `bbndfi` with company code, effective date, time, region, and price, and logs to `bbndfih`.
    - **Update Record**: If the record exists and a price is provided, updates the price in `bbndfi` and logs to `bbndfih`.
    - **Delete Record**: If the record exists and no price is provided, deletes the record from `bbndfi`, logs to `bbndfih` with a delete flag ('D'), and decrements the occurrence count (`c2occr`).

14. **Reposition Subfile 2 (sf2rep Subroutine)**:
    - Clears the subfile (`sf2clr`) and resets `rrn2`.
    - Validates control record input (`sf2cte`: currently empty).
    - Positions the file (`gstabl` for all mode or `bbndfird` for review mode) and loads records (`sf2lod`).

15. **Load Subfile 2 Records (sf2lod Subroutine)**:
    - Loads records from `gstabl` (all mode) or `bbndfird` (review mode) up to the page size (`pagsz2`).
    - Formats each record (`sf2fmt`):
      - **All Mode**: Loads all region codes from `gstabl`, checks for existing records in `bbndfi`, and sets the price if found.
      - **Review Mode**: Loads existing records from `bbndfird`, retrieves region descriptions from `gstabl`, or uses "## Invalid Region ##" if not found.
    - Applies protection schemes (`sf2pro`).

16. **Message Handling (addmsg, wrtmsg, clrmsg Subroutines)**:
    - Adds error messages to the program message queue (`QMHSNDPM`).
    - Writes messages to the message subfile (`msgctl`).
    - Clears the message subfile (`QMHRMVPM`).

17. **History Logging (writehist Subroutine)**:
    - Logs changes to the history file (`bbndfih`) with company code, effective date, time, region, price, delete flag, change date, time, and user.

18. **Program Termination**:
    - Closes all files and sets `*INLR = *ON` to end the program.

### Business Rules
1. **Mode-Based Access**:
   - In 'MNT' mode, users can add, update, or delete records. Fields are editable (`*IN70`, `*IN71` off).
   - In 'INQ' mode, fields are protected (`*IN70`, `*IN71` on), allowing only viewing.

2. **Validation**:
   - Company code (`c1co`) must exist in `bicont` (error `ERR0021`).
   - Effective date (`c1emdy`, `d1emdy`) must be valid per `GSDTEDIT` (error `ERR0020`).
   - Effective time (`d1eftm`) must have valid hours (00-23) and minutes (00-59) (error `ERR0019`).
   - Region codes are validated against `gstabl` in subfile 2.

3. **Subfile Options**:
   - Subfile 1 supports options 2 (Change) and 5 (Display).
   - Subfile 2 allows adding, updating, or deleting region-specific price records in maintenance mode.
   - All mode (`s2all = *ON`) displays all regions from `gstabl`, while review mode shows only existing `bbndfi` records.

4. **History Logging**:
   - All changes (add, update, delete) are logged to `bbndfih` with a delete flag ('D' for deletions) and timestamp/user information.

5. **Navigation**:
   - F03 exits the program.
   - F05 refreshes the subfile.
   - F06 toggles between all and review modes in subfile 2.
   - F09 calls historical inquiry (`GB730P`).
   - F10 positions the cursor to the control record.
   - F12 returns to subfile 1.
   - F23 deletes a detail record in subfile 2.

### Tables Used
1. **bicont**: Input file for company code validation (read-only).
2. **gstabl**: Input file for region code validation and descriptions (read-only).
3. **bbndfi**: Update file for National Diesel Fuel Index records (add/update/delete).
4. **bbndfi1**: Input file for header records (read-only, renamed format `bbndfi1r`).
5. **bbndfird**: Input file for detail records (read-only, renamed format `bbndfi2r`).
6. **bbndfih**: Output file for history records (write-only, added per JK01).

### External Programs Called
1. **GSDTEDIT**: Validates effective dates.
2. **GB730P**: Historical inquiry program for detail records, called when F09 is pressed.
3. **QCMDEXC**: Executes file override commands.
4. **QMHSNDPM**: Sends messages to the program message queue.
5. **QMHRMVPM**: Removes messages from the program message queue.

The `BB910` program is tightly integrated with `BB912`, providing a focused interface for managing National Diesel Fuel Index data with robust validation and history logging, ensuring data integrity and user-friendly interaction.