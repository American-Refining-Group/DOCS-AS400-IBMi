The RPG program `BB912.rpgle` is designed for carrier fuel surcharge maintenance and inquiry within the Bradford Order Entry/Invoices system. It is called by `BB945.rpgle` to manage fuel surcharge records for specific carriers, allowing users to view, add, update, or delete surcharge entries. The program uses two subfiles (`SFL1` for header records and `SFL2` for detail records) to display and manage carrier fuel surcharge data. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of `BB912.rpgle`

The program operates through a series of subroutines that handle the display and processing of carrier fuel surcharge records. The main steps are:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Receives Input Parameters**:
     - `a$co`: Company number.
     - `p$caid`: Carrier ID.
     - `p$mode`: Run mode ('MNT' for maintenance, 'INQ' for inquiry).
     - `p$fgrp`: File group ('Z' or 'G' for library selection).
   - Sets global protection mode (`*in70`) based on `p$mode`:
     - `MNT`: Allows editing (`*in70 = *off`).
     - `INQ`: Read-only mode (`*in70 = *on`).
   - Initializes subfile control fields (`rrn1`, `rrn2`, `pagsz1`, `pagsz2`), message handling fields, and key lists for file access.
   - Sets the current date and time (`ps#dat`, `ps#cen`, `ps#ymd`) for validation and history logging.
   - Moves input parameters to control fields (`c1co`, `c1caid`) and sets the header based on mode (`c$hdr1` = "Carrier Fuel Surcharge Maintenance" or "Inquiry").
   - Defines key lists (`kls1s1`, `kls1r1`, `klsfl1`, `kls2s1`, `kls2r1`, `klsfl2`, `klregn`, `klcaid`, `klcpy1`, `klcpy2`) for file access.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - Applies file overrides based on `p$fgrp` ('Z' or 'G') to select the appropriate library for files.
   - Opens input files (`bicont`, `gstabl`, `bbcaid`, `bbcfsh1`, `bbcfsdrd`) and update/output files (`bbcfsh`, `bbcfsd`, `bbcfshh`, `bbcfsdh`).

3. **Process Subfile 1 (`srsfl1` Subroutine)**:
   - Clears the message subfile (`clrmsg`) and writes it (`wrtmsg`).
   - Positions the file cursor (`sf1rep`) to load header records into `SFL1`.
   - Enters a main loop (`sf1agn`) to display and process `SFL1`:
     - Writes the command line (`sflcmd1`).
     - Displays the message subfile if errors exist (`wrtmsg`).
     - Checks for existing subfile records to enable/disable display (`*in41` for `SFLDSP`).
     - Displays the subfile control format (`sflctl1`) using `EXFMT`.
     - Processes user input:
       - **F03**: Exits the program.
       - **F04 (Subfile)**: Prompts for carrier ID (`prompt`) and processes subfile changes (`sf1pro`).
       - **F04 (Control)**: Prompts for control fields and repositions the subfile if the carrier ID changes (`sf1rep`).
       - **F05**: Refreshes the subfile, clearing positioning fields (`r$emdy`, `c1emdy`, `d1opt`, `d1emdy`).
       - **F07**: Calls `BB910` for National Diesel Fuel Index maintenance/inquiry.
       - **F09**: Calls `GB730P` for history inquiry of a selected header record.
       - **Direct Access**: Processes direct input options (`sf1dir`) if `d1opt` or `d1emdy` is non-zero.
       - **Page Down**: Loads additional subfile records (`sf1lod`).
       - **Enter**: Processes subfile changes (`sf1prc`).
     - Handles user repositioning requests by checking control fields (`c1co`, `c1caid`, `c1emdy`) and repositioning if needed (`sf1rep`).
     - Sets cursor position (`row1`, `col1`) for the next display.

4. **Reposition Subfile 1 (`sf1rep` Subroutine)**:
   - Clears `SFL1` and resets the relative record number (`rrn1`).
   - Validates control fields (`sf1cte`):
     - Company (`c1co`): Chains to `bicont` for name.
     - Carrier ID (`c1caid`): Chains to `bbcaid` for name; allows "##" for all carriers.
     - Effective Date (`c1emdy`): Validates format using `GSDTEDIT`.
   - Positions the file cursor using key lists (`kls1r1` or `kls1s1`) based on control fields.
   - Loads `SFL1` with header records (`sf1lod`).
   - Retains control field values (`h$co`, `h$caid`) for subsequent repositioning.

5. **Load Subfile 1 (`sf1lod` Subroutine)**:
   - Reads header records from `bbcfsh1` based on control fields (company, carrier ID, effective date).
   - Filters out deleted records (`bjdel = 'D'`) unless explicitly included.
   - Formats each subfile line (`sf1fmt`) and applies color coding (`sf1col`).
   - Writes records to `SFL1` and updates `rrn1` until the page size (`pagsz1 = 28`) is reached or no more records are available.

6. **Format Subfile 1 Line (`sf1fmt` Subroutine)**:
   - Populates `SFL1` fields (`s1opt`, `s1efdt`, `s1regn`, etc.) from the header record.
   - Converts effective date (`bjefdt`) to MMDDYY format (`s1emdy`).
   - Retrieves region description from `gstabl` if `bjregn` is non-blank.

7. **Color Coding for Subfile 1 (`sf1col` Subroutine)**:
   - Applies color indicators:
     - Red (`*in72`): Expired records (`s1efdt < ps#dat`).
     - Turquoise (`*in73`): Deleted records (`s1del = 'D'`).

8. **Process Subfile 1 on Enter (`sf1prc` Subroutine)**:
   - Reads changed `SFL1` records using `READC`.
   - Processes user-selected options (`s1opt`):
     - **2**: Update header record (`sf1s02`).
     - **3**: Copy header record (`sf1s03`).
     - **4**: Delete header record (`sf1s04`).
     - **5**: Display header record (`sf1s05`).
     - **6**: Process detail records (`sf1s06`, opens `SFL2`).
   - Updates the subfile record after processing.

9. **Subfile 1 Option Processing**:
   - **Update (`sf1s02`)**:
     - Validates the header record and effective date (`sf1s2e`).
     - Displays the header window (`srwin`) for editing.
     - Updates `bbcfsh` and writes to history (`writehdrhist`).
   - **Copy (`sf1s03`)**:
     - Copies the header record to a new effective date.
     - Validates the new date and updates `bbcfsh`.
     - Writes to history (`writehdrhist`).
   - **Delete (`sf1s04`)**:
     - Marks the header record as deleted (`bjdel = 'D'`) or reactivates it if previously deleted.
     - Updates `bbcfsh` and writes to history (`writehdrhist`).
   - **Display (`sf1s05`)**:
     - Displays the header record in inquiry mode (`srwin`).
   - **Detail Processing (`sf1s06`)**:
     - Opens `SFL2` to manage detail records for the selected header (`srsfl2`).

10. **Process Subfile 2 (`srsfl2` Subroutine)**:
    - Clears `SFL2` and positions the file cursor (`sf2rep`) for detail records.
    - Enters a loop (`sf2agn`) to display and process `SFL2`:
      - Displays the command line and message subfile.
      - Loads detail records (`sf2lod`) based on the header’s company, carrier ID, and effective date.
      - Processes user input:
        - **F03**: Returns to `SFL1`.
        - **F04**: Prompts for fuel surcharge percentage (`prompt`).
        - **F05**: Refreshes `SFL2`.
        - **F12**: Returns to `SFL1`.
        - **F14**: Adds a new detail record (`sf2s14`).
        - **F20**: Copies detail records (`sf2s20`).
        - **Enter**: Processes subfile changes (`sf2prc`).
        - **Page Down**: Loads additional detail records (`sf2lod`).

11. **Reposition Subfile 2 (`sf2rep` Subroutine)**:
    - Clears `SFL2` and resets `rrn2`.
    - Validates control fields (`sf2cte`):
      - Fuel Surcharge Percentage (`c2frpc`): Ensures non-negative.
      - Effective Date (`c2emdy`): Validates format.
    - Positions the file cursor using key lists (`kls2r1` or `kls2s1`).
    - Loads `SFL2` with detail records (`sf2lod`).

12. **Load Subfile 2 (`sf2lod` Subroutine)**:
    - Reads detail records from `bbcfsdrd` based on header fields.
    - Filters out deleted records (`bkdel = 'D'`) unless included.
    - Formats each subfile line (`sf2fmt`) and applies color coding (`sf2col`).
    - Writes records to `SFL2` until the page size (`pagsz2 = 26`) is reached.

13. **Format Subfile 2 Line (`sf2fmt` Subroutine)**:
    - Populates `SFL2` fields (`s2opt`, `s2frpc`, `s2topc`, `s2fspc`) from the detail record.
    - Formats percentages for display.

14. **Color Coding for Subfile 2 (`sf2col` Subroutine)**:
    - Applies color indicators:
      - Red (`*in72`): Expired records (`c2efdt < ps#dat`).
      - Turquoise (`*in73`): Deleted records (`s2del = 'D'`).

15. **Process Subfile 2 on Enter (`sf2prc` Subroutine)**:
    - Reads changed `SFL2` records using `READC`.
    - Processes user-selected options (`s2opt`):
      - **2**: Update detail record (`sf2s02`).
      - **4**: Delete detail record (`sf2s04`).
      - **5**: Display detail record (`sf2s05`).
    - Updates `bbcfsd` and writes to history (`writedtlhist`).

16. **Subfile 2 Option Processing**:
    - **Update (`sf2s02`)**:
      - Validates percentage ranges (`sf2s2e`).
      - Updates `bbcfsd` and writes to history.
    - **Delete (`sf2s04`)**:
      - Marks the detail record as deleted (`bkdel = 'D'`) or reactivates it.
      - Updates `bbcfsd` and writes to history.
    - **Display (`sf2s05`)**:
      - Displays the detail record in inquiry mode.

17. **Direct Access Processing (`sf1dir` Subroutine)**:
    - Handles direct input options (`d1opt`):
      - **6**: Opens `SFL2` for detail processing.
      - **14**: Adds a new header record.
      - **20**: Copies header records to a new effective date.
    - Validates input and updates `bbcfsh` as needed.

18. **Write History (`writehdrhist`, `writedtlhist` Subroutines)**:
    - **writehdrhist**: Writes header history to `bbcfshh` with timestamp, user ID, and record data (`khco`, `khcaid`, `khefdt`, `khregn`).
    - **writedtlhist**: Writes detail history to `bbcfsdh` with timestamp, user ID, and record data (`jhco`, `jhcaid`, `jhefdt`, `jhfrpc`, `jhtopc`, `jhfspc`).
    - Sets deletion flag (`khdel`, `jhdel`) to 'D' if F23 is pressed.

19. **Field Prompting (`prompt` Subroutine)**:
    - Calls `LBBCAID` to prompt for carrier ID (`c1caid`) when the cursor is on the carrier field.
    - Updates control fields with selected values.

20. **Message Handling**:
    - **Add Message (`addmsg`)**: Sends error or confirmation messages to the program message queue.
    - **Write Message (`wrtmsg`)**: Displays messages in the message subfile.
    - **Clear Message (`clrmsg`)**: Clears the message subfile.

21. **Program End**:
    - Closes all files and terminates the program, returning control to `BB945`.

---

### Business Rules

The program enforces the following business rules to ensure data integrity and proper fuel surcharge management:
1. **Validation of Key Fields**:
   - Company (`c1co`) must exist in `bicont`.
   - Carrier ID (`c1caid`) must exist in `bbcaid` or be "##" (all carriers).
   - Effective Date (`c1emdy`, `c2emdy`) must be in valid MMDDYY format, validated by `GSDTEDIT`.

2. **Percentage Range Validation**:
   - In `SFL2`, the "To Price" (`s2topc`) must not be less than the "From Price" (`s2frpc`).
   - Percentage ranges must not conflict with existing entries for the same header (company, carrier ID, effective date).
   - Fuel surcharge percentage (`s2fspc`) must be non-negative and consistent with existing entries.

3. **Deletion Handling**:
   - Header and detail records can be marked as deleted (`bjdel = 'D'`, `bkdel = 'D'`) or reactivated.
   - Deletion actions (via F23) are logged in history files (`bbcfshh`, `bbcfsdh`).

4. **Mode-Based Protection**:
   - In inquiry mode (`p$mode = 'INQ'`), all fields are protected (`*in70`, `*in71`).
   - In maintenance mode (`p$mode = 'MNT'`), fields are editable, with validation enforced.

5. **History Tracking**:
   - All add, update, copy, and delete operations on header (`bbcfsh`) and detail (`bbcfsd`) records are logged to `bbcfshh` and `bbcfsdh`, respectively, with user ID and timestamp.

6. **Subfile Navigation**:
   - `SFL1` displays header records (company, carrier ID, effective date, region).
   - `SFL2` displays detail records (fuel surcharge percentages and price ranges) for a selected header.
   - Users can navigate between subfiles using options (e.g., option 6 to access details).

7. **Error Handling**:
   - Errors (e.g., invalid carrier, date format, or percentage conflicts) are displayed in the message subfile with specific error messages (e.g., "To Price Cannot Be Less Than From Price").
   - The first display suppresses errors to avoid overwhelming the user (`w$frst`).

---

### Tables (Files) Used

The program accesses the following files, with overrides applied based on `p$fgrp` ('Z' or 'G'):
1. **bb912d**: Display file (workstation file with subfiles `SFL1` and `SFL2`, and message subfile).
2. **bicont**: Company master file (input only).
3. **gstabl**: Table file for region codes (input only).
4. **bbcaid**: Carrier ID file (input only).
5. **bbcfsh**: Carrier fuel surcharge header file (update and add).
6. **bbcfsh1**: Carrier fuel surcharge header file (input only, renamed record format `bbcfhpr`).
7. **bbcfsd**: Carrier fuel surcharge detail file (update and add).
8. **bbcfsdrd**: Carrier fuel surcharge detail file (input only, renamed record format `bbcfdpr`).
9. **bbcfshh**: Carrier fuel surcharge header history file (output only).
10. **bbcfsdh**: Carrier fuel surcharge detail history file (output only).

---

### External Programs Called

The program interacts with the following external programs:
1. **LBBCAID**: Prompts for carrier ID selection.
2. **BB910**: Manages National Diesel Fuel Index maintenance/inquiry.
3. **GB730P**: Displays history inquiry for header records.
4. **QMHSNDPM**: Sends messages to the program message queue.
5. **QMHRMVPM**: Removes messages from the program message queue.
6. **QCMDEXC**: Executes file override commands.
7. **GSDTEDIT**: Validates dates (called via `pldted` parameter list).

---

### Summary

The `BB912.rpgle` program, called by `BB945.rpgle`, provides a robust interface for managing carrier fuel surcharge records. It uses two subfiles: `SFL1` for header records (company, carrier ID, effective date, region) and `SFL2` for detail records (fuel surcharge percentages and price ranges). The program supports adding, updating, copying, deleting, and inquiring about surcharge records, with strict validation of fields and percentage ranges. It logs all changes to history files (`bbcfshh`, `bbcfsdh`) and integrates with external programs for prompting, history inquiry, and date validation. The revisions indicate enhancements like history tracking and replacing older carrier lookup logic with `LBBCAID`.