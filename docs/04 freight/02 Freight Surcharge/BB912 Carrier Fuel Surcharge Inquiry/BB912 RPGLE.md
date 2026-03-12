The RPG program `BB912` is designed for Carrier Fuel Surcharge Maintenance/Inquiry within the Brandford Order Entry/Invoices system. It manages fuel surcharge data through a display file with subfiles, allowing users to create, update, copy, delete, or inquire about surcharge records. Below is a detailed explanation of the process steps, followed by a list of external programs called and tables used.

### Process Steps of the BB912 RPG Program

1. **Initialization (*INZSR Subroutine)**:
   - Receives input parameters: company code (`p$co`), carrier code (`p$caid`), run mode (`p$mode`: 'MNT' for maintenance or 'INQ' for inquiry), and file group (`p$fgrp`: 'G' or 'Z').
   - Sets default values for company code if not provided (defaults to 10).
   - Initializes subfile control fields, message handling fields, and date/time stamps.
   - Configures headers and function key labels based on the mode ('MNT' or 'INQ').
   - Defines key lists for database access and initializes work fields.

2. **Open Database Tables (opntbl Subroutine)**:
   - Applies file overrides based on the file group (`p$fgrp`) to point to the correct library ('G' or 'Z') for files like `bicont`, `gstabl`, `bbcfsh`, etc.
   - Opens input files (`bicont`, `gstabl`, `bbcaid`, `bbcfsh1`, `bbcfsdrd`) and update/output files (`bbcfsh`, `bbcfsd`, `bbcfshh`, `bbcfsdh`).

3. **Process Subfile 1 (srsfl1 Subroutine)**:
   - **Clear Message Subfile**: Clears any existing messages in the message subfile.
   - **Position File**: Repositions the `bbcfsh1` file based on user input (company code, carrier code, effective date).
   - **Suppress Initial Errors**: On first display, suppresses error messages to present a clean interface.
   - **Main Loop**:
     - Repositions the subfile if requested (`repsfl = *on`).
     - Displays the command line and message subfile.
     - Checks for existing subfile records to enable/disable display control (`*in41`).
     - Displays the subfile control record (`sflctl1`) using `EXFMT`.
     - Processes user input based on function keys:
       - **F03**: Exits the program.
       - **F04**: Prompts for field input (e.g., region or carrier code).
       - **F05**: Refreshes the subfile, clearing positioning fields.
       - **F07**: Calls `BB910` for National Diesel Fuel Index Maintenance/Inquiry.
       - **F09**: Calls `GB730P` for historical inquiry of header records.
       - **F10**: Positions the cursor to the control record.
       - **Page Down**: Loads additional subfile records.
       - **Enter**: Processes subfile changes (`sf1prc`).
     - Handles user positioning requests by validating and repositioning the subfile.

4. **Process Subfile 1 Records on Enter (sf1prc Subroutine)**:
   - Reads changed subfile records (`readc sfl1`) and processes each changed record (`sf1chg`).
   - Updates the database if no errors are found and the mode is 'MNT'.

5. **Process Subfile 1 Record Change (sf1chg Subroutine)**:
   - Retains selected values (effective date).
   - Validates input (`sf1edt`):
     - Checks region code against `gstabl` to ensure validity.
     - Displays error message (`ERR0010`) if invalid.
   - Updates the database (`sf1upd`) if no errors and in maintenance mode.
   - Processes subfile options (1=Create, 2=Change, 3=Copy, 4=Delete, 5=Display):
     - **Create (sf1s01)**: Opens a window (`sflcrt1`) to input new header record data, validates, and writes to `bbcfsh` and `bbcfshh`.
     - **Change (sf1s02)**: Calls `srsfl2` in maintenance mode to update detail records.
     - **Copy (sf1s03)**: Opens a window (`sflcpy1`) to copy header and detail records to a new effective date.
     - **Delete (sf1s04)**: Opens a window (`sfldel1`) to confirm deletion of header and detail records.
     - **Display (sf1s05)**: Calls `srsfl2` in inquiry mode to view detail records.

6. **Reposition Subfile 1 (sf1rep Subroutine)**:
   - Clears the subfile (`sf1clr`) and resets relative record number (`rrn1`).
   - Validates control record input (`sf1cte`):
     - Checks company code against `bicont`.
     - Validates carrier code against `bbcaid`.
     - Validates effective date using `GSDTEDIT`.
   - Positions the `bbcfsh1` file and loads subfile records (`sf1lod`).

7. **Load Subfile 1 Records (sf1lod Subroutine)**:
   - Loads records from `bbcfsh1` into the subfile up to the page size (`pagsz1`).
   - Formats each record (`sf1fmt`):
     - Sets effective date, region, and occurrence count.
     - Retrieves region description from `gstabl`.
   - Applies protection schemes (`sf1pro`) based on inquiry mode.

8. **Process Subfile 2 (srsfl2 Subroutine)**:
   - Handles detail records for a selected header record.
   - Sets protection indicators based on mode (`*in71`).
   - Initializes to update mode and sets function key labels.
   - Main loop processes user input similarly to `srsfl1`:
     - **F03**: Exits the program.
     - **F05**: Refreshes the subfile.
     - **F06**: Toggles between add and update modes.
     - **F09**: Calls `GB730P` for historical inquiry of detail records.
     - **F12**: Cancels and returns to subfile 1.
     - **F23**: Deletes a detail record.
     - **Page Down**: Loads additional detail records.
     - **Enter**: Processes subfile changes (`sf2prc`).

9. **Process Subfile 2 Records on Enter (sf2prc Subroutine)**:
   - Reads changed subfile records (`readc sfl2`) and processes each changed record (`sf2chg`).
   - Updates the database if no errors and in maintenance mode.

10. **Process Subfile 2 Record Change (sf2chg Subroutine)**:
    - Validates input (`sf2edt`):
      - Ensures all or no fields (`s2frpc`, `s2topc`, `s2fspc`) are entered.
      - Validates price ranges and fuel surcharge percentages against existing records.
    - Updates the database (`sf2upd`) by adding or updating records in `bbcfsd` and `bbcfsdh`.
    - Applies protection schemes (`sf2pro`).

11. **Reposition Subfile 2 (sf2rep Subroutine)**:
    - Clears the subfile (`sf2clr`) and resets `rrn2`.
    - Validates control record input (`sf2cte`) and loads detail records (`sf2lod`).

12. **Field Prompting (prompt Subroutine)**:
    - Handles F04 key for field prompting:
      - For `S1REGN`: Calls `MGSTABL` to select a region code.
      - For `C1CAID`: Calls `LBBCAID` to select a carrier code.

13. **Message Handling (addmsg, wrtmsg, clrmsg Subroutines)**:
    - Adds error messages to the program message queue (`QMHSNDPM`).
    - Writes messages to the message subfile (`msgctl`).
    - Clears the message subfile (`QMHRMVPM`).

14. **History Logging (writehdrhist, writedtlhist Subroutines)**:
    - Logs changes to header (`bbcfshh`) and detail (`bbcfsdh`) history files with timestamps and user information.

15. **Program Termination**:
    - Closes all files and sets `*inlr = *on` to end the program.

### External Programs Called
1. **BB910**: National Diesel Fuel Index Maintenance/Inquiry, called when F07 is pressed.
2. **GB730P**: Historical inquiry program for header and detail records, called when F09 is pressed.
3. **GSDTEDIT**: Date validation module, used to validate effective dates.
4. **MGSTABL**: Table lookup program for region codes, called during prompting for `S1REGN`.
5. **LBBCAID**: Carrier ID lookup program, called during prompting for `C1CAID` (replaced `BB912A` per revision JK02).
6. **QCMDEXC**: Executes file override commands.
7. **QMHSNDPM**: Sends messages to the program message queue.
8. **QMHRMVPM**: Removes messages from the program message queue.

### Tables Used
1. **bicont**: Input file for company code validation (read-only).
2. **gstabl**: Input file for region code validation (read-only).
3. **bbcaid**: Input file for carrier code validation (read-only, added per JK03).
4. **bbcfsh**: Update file for header records (add/update).
5. **bbcfsd**: Update file for detail records (add/update).
6. **bbcfsh1**: Input file for header records (read-only, renamed format `bbcfhpr`).
7. **bbcfsdrd**: Input file for detail records (read-only, renamed format `bbcfdpr`).
8. **bbcfshh**: Output file for header history records (write-only, added per JK01).
9. **bbcfsdh**: Output file for detail history records (write-only, added per JK01).

This program is structured to handle complex data maintenance tasks with robust error checking, user interaction through subfiles, and integration with external lookup and validation programs.