The RPG program `BB910.rpgle` is designed for National Diesel Fuel Index maintenance and inquiry within the Bradford Order Entry/Invoices system. It is called by `BB912.rpgle` to manage records related to the National Diesel Fuel Index, which tracks diesel fuel prices by region, effective date, and time. The program uses two subfiles (`SFL1` for header records and `SFL2` for detail records) to display and manage these records, allowing users to view, add, update, or delete entries. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of `BB910.rpgle`

The program operates through a series of subroutines that handle the display and processing of National Diesel Fuel Index records. The main steps are:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Receives Input Parameters**:
     - `a$co`: Company number.
     - `p$mode`: Run mode ('MNT' for maintenance, 'INQ' for inquiry).
     - `p$fgrp`: File group ('Z' or 'G' for library selection).
   - Sets global protection mode (`*in70`) based on `p$mode`:
     - `MNT`: Allows editing (`*in70 = *off`).
     - `INQ`: Read-only mode (`*in70 = *on`).
   - Initializes subfile control fields (`rrn1`, `rrn2`, `pagsz1`, `pagsz2`), message handling fields, and key lists for file access.
   - Sets the current date and time (`ps#dat`, `ps#cen`, `ps#ymd`, `t#hms`) for validation and history logging.
   - Moves input parameters to control fields (`c1co`) and sets the header (`c$hdr1`) to "National Diesel Fuel Index Maintenance" or "Inquiry" based on mode.
   - Defines key lists (`kls1s1`, `kls1r1`, `klnxt1`, `klsfl1`, `kls2a1`, `kls2a2`, `kls2r1`, `kls2r2`, `klsfl2`, `klregn`) for file access.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - Applies file overrides based on `p$fgrp` ('Z' or 'G') to select the appropriate library for files.
   - Opens input files (`bicont`, `gstabl`, `bbndfi1`, `bbndfird`) and update/output files (`bbndfi`, `bbndfih`).

3. **Process Subfile 1 (`srsfl1` Subroutine)**:
   - Clears the message subfile (`clrmsg`) and writes it (`wrtmsg`).
   - Sets a default company value (`c1co = 10`) if not provided.
   - Positions the file cursor (`sf1rep`) to load header records into `SFL1`.
   - Enters a main loop (`sf1agn`) to display and process `SFL1`:
     - Writes the command line (`sflcmd1`).
     - Displays the message subfile if errors exist (`wrtmsg`).
     - Checks for existing subfile records to enable/disable display (`*in41` for `SFLDSP`).
     - Displays the subfile control format (`sflctl1`) using `EXFMT`.
     - Processes user input:
       - **F03**: Exits the program.
       - **F05**: Refreshes the subfile, clearing positioning fields (`r$emdy`, `c1emdy`, `d1opt`, `d1emdy`, `d1eftm`).
       - **F06**: Toggles between "Review Mode" and "All Mode" (`s1f06`), controlling which records are displayed.
       - **F09**: Calls `GB730P` for history inquiry of a selected header record.
       - **Direct Access**: Processes direct input options (`sf1dir`) if `d1opt`, `d1emdy`, or `d1eftm` is non-zero.
       - **Page Down**: Loads additional subfile records (`sf1lod`).
       - **Enter**: Processes subfile changes (`sf1prc`).
     - Handles user repositioning requests by checking control fields (`c1co`, `c1emdy`) and repositioning if needed (`sf1rep`).
     - Sets cursor position (`row1`, `col1`) for the next display.

4. **Reposition Subfile 1 (`sf1rep` Subroutine)**:
   - Clears `SFL1` and resets the relative record number (`rrn1`).
   - Validates control fields (`sf1cte`):
     - Company (`c1co`): Chains to `bicont` for name.
     - Effective Date (`c1emdy`): Validates format using `GSDTEDIT`.
   - Positions the file cursor using key lists (`kls1r1` or `kls1s1`) based on control fields.
   - Loads `SFL1` with header records (`sf1lod`).

5. **Load Subfile 1 (`sf1lod` Subroutine)**:
   - Reads header records from `bbndfi1` based on company and effective date.
   - Filters out deleted records (`bndel = 'D'`) unless explicitly included.
   - Formats each subfile line (`sf1fmt`) and applies color coding (`sf1col`).
   - Writes records to `SFL1` and updates `rrn1` until the page size (`pagsz1 = 28`) is reached or no more records are available.

6. **Format Subfile 1 Line (`sf1fmt` Subroutine)**:
   - Populates `SFL1` fields (`s1opt`, `s1efdt`, `s1eftm`, `s1regn`) from the header record.
   - Converts effective date (`bnefdt`) to MMDDYY format (`s1emdy`) and time (`bneftm`) to HHMM format.

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
     - Validates the header record and effective date/time (`sf1s2e`).
     - Displays the header window (`srwin`) for editing.
     - Updates `bbndfi` and writes to history (`writehist`).
   - **Copy (`sf1s03`)**:
     - Copies the header record to a new effective date/time.
     - Validates the new date/time and updates `bbndfi`.
     - Writes to history (`writehist`).
   - **Delete (`sf1s04`)**:
     - Marks the header record as deleted (`bndel = 'D'`) or reactivates it if previously deleted.
     - Updates `bbndfi` and writes to history (`writehist`).
   - **Display (`sf1s05`)**:
     - Displays the header record in inquiry mode (`srwin`).
   - **Detail Processing (`sf1s06`)**:
     - Opens `SFL2` to manage detail records for the selected header (`srsfl2`).

10. **Process Subfile 2 (`srsfl2` Subroutine)**:
    - Clears `SFL2` and positions the file cursor (`sf2rep`) for detail records.
    - Enters a loop (`sf2agn`) to display and process `SFL2`:
      - Displays the command line and message subfile.
      - Loads detail records (`sf2lod`) based on the header’s company, effective date, and time.
      - Processes user input:
        - **F03**: Returns to `SFL1`.
        - **F05**: Refreshes `SFL2`.
        - **F12**: Returns to `SFL1`.
        - **F14**: Adds a new detail record (`sf2s14`).
        - **Enter**: Processes subfile changes (`sf2prc`).
        - **Page Down**: Loads additional detail records (`sf2lod`).

11. **Reposition Subfile 2 (`sf2rep` Subroutine)**:
    - Clears `SFL2` and resets `rrn2`.
    - Validates control fields (`sf2cte`):
      - Region (`c2regn`): Chains to `gstabl` to ensure validity.
      - Effective Date (`c2efdt`) and Time (`c2eftm`): Ensures consistency with header.
    - Positions the file cursor using key lists (`kls2r1`, `kls2r2`, or `kls2a1`).
    - Loads `SFL2` with detail records (`sf2lod`).

12. **Load Subfile 2 (`sf2lod` Subroutine)**:
    - Reads detail records from `bbndfird` based on header fields.
    - Filters out deleted records (`bndel = 'D'`) unless included.
    - Formats each subfile line (`sf2fmt`) and applies color coding (`sf2col`).
    - Writes records to `SFL2` until the page size (`pagsz2 = 28`) is reached.

13. **Format Subfile 2 Line (`sf2fmt` Subroutine)**:
    - Populates `SFL2` fields (`s2opt`, `s2regn`, `s2rprc`) from the detail record.
    - Retrieves region description from `gstabl`.

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
    - Updates `bbndfi` and writes to history (`writehist`).

16. **Subfile 2 Option Processing**:
    - **Update (`sf2s02`)**:
      - Validates region and price (`sf2s2e`).
      - Updates `bbndfi` and writes to history.
    - **Delete (`sf2s04`)**:
      - Marks the detail record as deleted (`bndel = 'D'`) or reactivates it.
      - Updates `bbndfi` and writes to history.
    - **Display (`sf2s05`)**:
      - Displays the detail record in inquiry mode.

17. **Direct Access Processing (`sf1dir` Subroutine)**:
    - Handles direct input options (`d1opt`):
      - **6**: Opens `SFL2` for detail processing.
      - **14**: Adds a new header record.
      - **20**: Copies header records to a new effective date/time.
    - Validates input and updates `bbndfi` as needed.

18. **Write History (`writehist` Subroutine)**:
    - Writes history records to `bbndfih` with timestamp, user ID, and record data (`lhco`, `lhefdt`, `lheftm`, `lhregn`, `lhrprc`).
    - Sets deletion flag (`lhdel = 'D'`) if the record is deleted (`s2rprc = 0`).

19. **Message Handling**:
    - **Add Message (`addmsg`)**: Sends error or confirmation messages (e.g., "Invalid Region") to the program message queue.
    - **Write Message (`wrtmsg`)**: Displays messages in the message subfile.
    - **Clear Message (`clrmsg`)**: Clears the message subfile.

20. **Program End**:
    - Closes all files and terminates the program, returning control to `BB912`.

---

### Business Rules

The program enforces the following business rules to ensure data integrity and proper management of National Diesel Fuel Index records:
1. **Validation of Key Fields**:
   - Company (`c1co`) must exist in `bicont`.
   - Region (`c2regn`) must exist in `gstabl` with type `BBNDFI`.
   - Effective Date (`c1emdy`, `c2efdt`) must be in valid MMDDYY format, validated by `GSDTEDIT`.
   - Effective Time (`c2eftm`) must be in valid HHMM format.

2. **Deletion Handling**:
   - Header and detail records can be marked as deleted (`bndel = 'D'`) or reactivated.
   - Deletion actions are logged in the history file (`bbndfih`).

3. **Mode-Based Protection**:
   - In inquiry mode (`p$mode = 'INQ'`), all fields are protected (`*in70`, `*in71`).
   - In maintenance mode (`p$mode = 'MNT'`), fields are editable, with validation enforced.

4. **History Tracking**:
   - All add, update, copy, and delete operations on header and detail records (`bbndfi`) are logged to `bbndfih` with user ID, timestamp, and record data.

5. **Subfile Navigation**:
   - `SFL1` displays header records (company, effective date, effective time).
   - `SFL2` displays detail records (region, retail price) for a selected header.
   - Users can navigate between subfiles using options (e.g., option 6 to access details).

6. **Error Handling**:
   - Errors (e.g., invalid region or date format) are displayed in the message subfile with specific messages (e.g., "Invalid Region").
   - The first display suppresses errors to avoid overwhelming the user (`w$frst`).

7. **Display Modes**:
   - F06 toggles between "Review Mode" (filtered records) and "All Mode" (all records), affecting which records are loaded into `SFL1`.

---

### Tables (Files) Used

The program accesses the following files, with overrides applied based on `p$fgrp` ('Z' or 'G'):
1. **bb910d**: Display file (workstation file with subfiles `SFL1` and `SFL2`, and message subfile).
2. **bicont**: Company master file (input only).
3. **gstabl**: Table file for region codes (input only).
4. **bbndfi**: National Diesel Fuel Index file (update and add).
5. **bbndfi1**: National Diesel Fuel Index file (input only, renamed record format `bbndfi1r`).
6. **bbndfird**: National Diesel Fuel Index file (input only, renamed record format `bbndfi2r`).
7. **bbndfih**: National Diesel Fuel Index history file (output only).

---

### External Programs Called

The program interacts with the following external programs:
1. **GB730P**: Displays history inquiry for header records.
2. **QMHSNDPM**: Sends messages to the program message queue.
3. **QMHRMVPM**: Removes messages from the program message queue.
4. **QCMDEXC**: Executes file override commands.
5. **GSDTEDIT**: Validates dates (called via `pldted` parameter list).

---

### Summary

The `BB910.rpgle` program, called by `BB912.rpgle`, provides a robust interface for managing National Diesel Fuel Index records. It uses two subfiles: `SFL1` for header records (company, effective date, effective time) and `SFL2` for detail records (region, retail price). The program supports adding, updating, copying, deleting, and inquiring about records, with strict validation of company, region, date, and time fields. All changes are logged to the history file (`bbndfih`), and the program integrates with external programs for history inquiry and date validation. The lack of revisions since the initial creation suggests stability, with the core functionality focused on maintaining accurate diesel fuel price data by region.