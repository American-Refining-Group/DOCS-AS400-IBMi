The RPG program **BB911** is designed for the maintenance and inquiry of the National Diesel Fuel Index within the Brandford Order Entry/Invoices system. It manages data related to fuel index records, allowing users to create, update, display, and inquire about these records through a display file with subfiles (SFL1 and SFL2). Below is an explanation of the process steps, external programs called, and the tables .

---

### **Process Steps of BB911**

The program follows a structured flow with subroutines that handle initialization, subfile processing, user input validation, and database operations. Here’s a breakdown of the main process steps:

1. **Initialization (*inzsr Subroutine)**:
   - **Purpose**: Sets up the program's environment and initializes variables.
   - **Steps**:
     - Receives entry parameters: company code (`a$co`), run mode (`p$mode`, either 'MNT' for maintenance or 'INQ' for inquiry), and file group (`p$fgrp`, either 'G' or 'Z').
     - Moves input parameters to format fields (e.g., `c1co` for company code).
     - Initializes subfile control fields (e.g., `rrn1`, `rrn2`, `rrnsv1`, `rrnsv2`) to zero.
     - Sets page sizes for subfiles (`pagsz1`, `pagsz2`) to 28 records.
     - Determines the header (`c$hdr1`) based on the mode ('MNT' or 'INQ') and sets field protection indicator `*in70` accordingly.
     - Initializes date and time fields using the system date (`ps#mdy`) and time (`time12`).
     - Defines key lists (`kls1s1`, `kls1r1`, `klnxt1`, `klsfl1`, `kls2r1`, `kls2r2`, `klsfl2`) for database access.

2. **Open Database Tables (opntbl Subroutine)**:
   - **Purpose**: Opens the required database files with appropriate overrides based on the file group.
   - **Steps**:
     - If the file group is 'G' or 'Z', applies overrides from arrays `ovg` or `ovz` using the `QCMDEXC` command to override files (`bicont`, `gstabl`, `bbrcsc`, `bbrcsc1`, `bbrcscrd`, `bbrcsch`) to the correct library.
     - Opens files: `bicont` (input), `gstabl` (input), `bbrcsc1` (input), `bbrcscrd` (input), `bbrcsc` (update/add), and `bbrcsch` (output for history).

3. **Process Subfile 1 (srsfl1 Subroutine)**:   - **Purpose**: Manages the main subfile (SFL1) for displaying and processing fuel index records.
   - **Steps**:
     - Clears the message subfile (`clrmsg`) and writes it (`wrtmsg`).
     - Sets a default company value (`c1co = 10`) if not provided.
     - Positions the file (`sf1rep`) based on user input (company and effective date).
     - Suppresses errors on the first display (`w$frst`).
     - Enters a loop (`sf1agn`) to:
       - Reposition the subfile if needed (`repsfl`).
       - Display the command line (`sflcmd1`) and message subfile.
       - Check for existing subfile records to set the display control indicator (`*in41`).
       - Display the subfile control record (`sflctl1`) using `exfmt`.
       - Clear message subfile if displayed.
       - Clear format indicators (`*in50` to `*in69`).
       - Determine cursor location (`csrloc`) and set the subfile record number (`rcdnb1`).
       - Process user input based on function keys:
         - **F03**: Exit the loop and program.
         - **F05**: Refresh the subfile, clear positioning fields, and reposition.
         - **Direct Access**: If `d1opt`, `d1emdy`, or `d1eftm` are non-zero, process direct access (`sf1dir`).
         - **Page Down**: Load more subfile records (`sf1lod`).
         - **Enter**: Process subfile records (`sf1prc`).
         - **F10**: Position cursor to control record, clearing `row1` and `col1`.
       - Reposition the subfile if the company code or effective date changes.

4. **Process SFL1 Records on Enter (sf1prc Subroutine)**:
   - **Purpose**: Processes changes made to SFL1 records when the user presses Enter.
   - **Steps**:
     - Reads changed subfile records (`readc sfl1`) if the subfile is not empty.
     - For each changed record, calls `sf1chg` to process the changes.
     - Sets a flag (`s1chng`) to indicate changes were processed.

5. **Process SFL1 Record Changes (sf1chg Subroutine)**:
   - **Purpose**: Handles specific options selected in SFL1 records.
   - **Steps**:
     - Copies selected values (`s1efdt`, `s1eftm`, `s1emdy`) to SFL2 control fields (`c2efdt`, `c2eftm`, `c2emdy`).
     - Processes options:
       - **Option 2 (Change)**: Calls `sf1s02` if in maintenance mode.
       - **Option 5 (Display)**: Calls `sf1s05`.
     - Updates the subfile record with the occurrence count (`s1occr`) after processing and clears the option field (`s1opt`).

6. **Reposition Subfile 1 (sf1rep Subroutine)**:
   - **Purpose**: Repositions the subfile based on user input.
   - **Steps**:
     - Clears the subfile (`sf1clr`) and resets the relative record number (`rrn1`).
     - Edits control input (`sf1cte`).
     - If no errors, positions the file (`bbrcsc1`) using the key list (`kls1s1`) and retains control fields for repositioning.
     - Loads the subfile (`sf1lod`).

7. **Edit SFL1 Control Input (sf1cte Subroutine)**:
   - **Purpose**: Validates user input in the SFL1 control record.
   - **Steps**:
     - Clears error indicators (`*in50` to `*in69`).
     - Validates the company code (`c1co`) against `bicont`. If invalid, sets error `ERR0021`.
     - Validates the effective date (`c1emdy`) using the `GSDTEDIT` program. If invalid, sets error `ERR0020`.
     - Sets error indicators (`*in50`, `*in51`, `*in52`) if validation fails.

8. **Load SFL1 Records (sf1lod Subroutine)**:
   - **Purpose**: Loads records into SFL1.
   - **Steps**:
     - Sets the relative record number (`rrn1`) to the last saved value (`rrnsv1`).
     - Loads up to `pagsz1` (28) records by reading `bbrcsc1` using key list `kls1r1`.
     - Formats each record (`sf1fmt`) and writes it to the subfile.
     - Updates the saved relative record number (`rrnsv1`).

9. **Format SFL1 Detail Line (sf1fmt Subroutine)**:
   - **Purpose**: Formats a single SFL1 record.
   - **Steps**:
     - Clears the subfile record.
     - Populates fields (`s1efdt`, `s1eftm`, `s1rprc`, `s1emdy`) from the file record (`bbrcsc1`).
     - Retrieves the occurrence count (`rtvent#`) and sets `s1occr`.

10. **Retrieve Number of Entries (rtvent# Subroutine)**:
    - **Purpose**: Counts the number of records for a given effective date and time.
    - **Steps**:
      - Clears the occurrence counter (`w$occr`).
      - Reads records from `bbrcscrd` using key list `klsfl1` and increments `w$occr` for each record.

11. **Direct Access Processing (sf1dir Subroutine)**:
    - **Purpose**: Processes direct input from the control record for creating or updating records.
    - **Steps**:
      - Validates the effective date (`d1emdy`) using `GSDTEDIT` and effective time (`d1eftm`) for valid hours and minutes. Sets errors (`ERR0020`, `ERR0019`) if invalid.
      - Ensures required fields are specified for create operations (`d1opt = 1`). Sets error `ERR0103` if missing.
      - Checks if the record exists (`klsfl1` on `bbrcsc`). Sets errors `ERR0102` (does not exist for update) or `ERR0101` (already exists for create).
      - If no errors, processes options:
        - **Option 1 (Create)**: Writes a new record to `bbrcsc`, calls `writehist` to log the change, and increments the occurrence count.
        - **Option 2 (Change)**: Calls `sf1s02`.
        - **Option 5 (Display)**: Calls `sf1s05`.
      - Clears input fields and cursor location if no errors.

12. **Clear SFL1 (sf1clr Subroutine)**:
    - **Purpose**: Clears the SFL1 subfile and resets its state.
    - **Steps**:
      - Resets `rrn1` and `rrnsv1` to zero.
      - Sets indicators to clear (`*in42`) and disable display (`*in40`, `*in41`).
      - Writes the subfile control record (`sflctl1`).

13. **SFL1 Option 01 - Create (sf1s01 Subroutine)**:
    - **Purpose**: Initiates the creation of a new record by switching to SFL2 in maintenance mode.
    - **Steps**:
      - Saves the current cursor location and indicators.
      - Sets `s2mode` to 'MNT' and calls `srsfl2` to process SFL2.
      - Restores indicators and cursor location.
      - Sets the reposition flag (`repsfl`).

14. **SFL1 Option 02 - Change (sf1s02 Subroutine)**:
    - **Purpose**: Allows modification of an existing record via SFL2 in maintenance mode.
    - **Steps**:
      - Similar to `sf1s01`, but maintains the current record context.

15. **SFL1 Option 05 - Display (sf1s05 Subroutine)**:
    - **Purpose**: Displays a record in inquiry mode via SFL2.
    - **Steps**:
      - Similar to `sf1s01`, but sets `s2mode` to 'INQ'.

16. **Process Subfile 2 (srsfl2 Subroutine)**:
    - **Purpose**: Manages the second subfile (SFL2) for detailed record maintenance or inquiry.
    - **Steps**:
      - Clears the message subfile and initializes mode-specific settings (`s2all`, `s2f06d`).
      - Retrieves the occurrence count (`rtvent#`) and sets `c2occr`.
      - Repositions the subfile (`sf2rep`).
      - Enters a loop (`sf2agn`) to:
        - Reposition the subfile if needed.
        - Display the command line (`sflcmd2`) and message subfile.
        - Check for existing subfile records to set display indicators.
        - Display the subfile control record (`sflctl2`).
        - Process user input:
          - **F03**: Exit both SFL1 and SFL2 loops.
          - **F05**: Refresh the subfile.
          - **F06**: Toggle between All and Review modes (`s2all`).
          - **F09**: Call `GB730P` for history inquiry, passing parameters (`x$hist`).
          - **F12**: Cancel and reposition SFL1.
          - **Page Down**: Load more SFL2 records (`sf2lod`).
          - **Enter**: Process SFL2 records (`sf2prc`).
          - **F10**: Position cursor to control record.

17. **Process SFL2 Records on Enter (sf2prc Subroutine)**:
    - **Purpose**: Processes changes made to SFL2 records.
    - **Steps**:
      - Reads changed subfile records (`readc sfl2`) and calls `sf2chg` for each changed record.
      - Sets a flag (`s2chng`) to indicate changes.

18. **Process SFL2 Record Changes (sf2chg Subroutine)**:
    - **Purpose**: Validates and updates SFL2 record changes.
    - **Steps**:
      - Edits input (`sf2edt`).
      - Clears errors in inquiry mode.
      - Updates the database (`sf2upd`) if in maintenance mode and no errors.
      - Sets the next change indicator (`*in44`) if errors occur.
      - Updates the subfile record (`sf2pro`).

19. **Edit SFL2 Input (sf2edt Subroutine)**:
    - **Purpose**: Validates SFL2 input.
    - **Steps**:
      - Clears error indicators.
      - Checks if `s2rprc` (retail price) is zero. If so, sets error `ERR0012`.

20. **Update Database from SFL2 Input (sf2upd Subroutine)**:
    - **Purpose**: Updates or deletes records in `bbrcsc` based on SFL2 input.
    - **Steps**:
      - Chains to `bbrcsc` using key list `klsfl2`.
      - If the record exists and `s2rprc` is non-zero, updates the record with new values and logs the change (`writehist`).
      - If the record does not exist and `s2rprc` is non-zero, creates a new record and logs the change.
      - If `s2rprc` is zero, deletes the record and logs the change.
      - Adjusts the occurrence count (`c2occr`).

21. **Reposition Subfile 2 (sf2rep Subroutine)**:
    - **Purpose**: Repositions SFL2 based on user input.
    - **Steps**:
      - Clears the subfile (`sf2clr`) and validates control input (`sf2cte`).
      - Positions the file (`bbrcscrd`) using key list `kls2r1`.
      - Loads the subfile (`sf2lod`).

22. **Edit SFL2 Control Input (sf2cte Subroutine)**:
    - **Purpose**: Placeholder for validating SFL2 control input (currently empty).

23. **Load SFL2 Records (sf2lod Subroutine)**:
    - **Purpose**: Loads records into SFL2.
    - **Steps**:
      - Sets the relative record number (`rrn2`) to the last saved value (`rrnsv2`).
      - Loads up to `pagsz2` (28) records by reading `bbrcscrd` using key list `kls2r2`.
      - Formats each record (`sf2fmt`) and writes it to the subfile.
      - Updates the saved relative record number (`rrnsv2`).

24. **Format SFL2 Detail Line (sf2fmt Subroutine)**:
    - **Purpose**: Formats a single SFL2 record.
    - **Steps**:
      - Clears the subfile record.
      - In All mode (`s2all`), populates fields from `gstabl` and checks `bbrcsc` for existing prices.
      - In Review mode, populates fields directly from `bbrcscrd`.
      - Applies protection schemes (`sf2pro`).

25. **SFL2 Protection Schemes (sf2pro Subroutine)**:
    - **Purpose**: Sets field protection for SFL2.
    - **Steps**:
      - Clears protection indicators (`*in71` to `*in74`).
      - Sets protection indicators in inquiry mode (`s2mode = 'INQ'`).

26. **Message Handling (addmsg, wrtmsg, clrmsg Subroutines)**:
    - **addmsg**: Sends error messages to the program message queue using `QMHSNDPM`.
    - **wrtmsg**: Writes the message subfile (`msgctl`) to display messages.
    - **clrmsg**: Clears the message subfile using `QMHRMVPM` and saves/restores the current record format and page number.

27. **Write History (writehist Subroutine)**:
    - **Purpose**: Logs changes to the history file (`bbrcsch`).
    - **Steps**:
      - Clears the history record (`bbrcschpf`).
      - Sets the delete flag (`lhdel = 'D'`) if `s2rprc` is zero.
      - Populates fields (`lhco`, `lhefdt`, `lheftm`, `lhrprc`, `lhchd8`, `lhchtm`, `lhuser`) with current values.
      - Writes the history record.

28. **Program Termination**:
    - Closes all files (`close *all`).
    - Sets `*inlr = *on` and returns.

---

### **External Programs Called**

The program calls the following external programs:
1. **QCMDEXC**: Executes override commands to set up file mappings for the correct library based on the file group ('G' or 'Z').
2. **GSDTEDIT**: Validates the effective date (`p#mdy`) and returns a converted date (`p#cymd`) or an error flag (`p#err`).
3. **QMHSNDPM**: Sends error messages to the program message queue for display.
4. **QMHRMVPM**: Removes messages from the program message queue when clearing the message subfile.
5. **GB730P**: Called for history inquiry (F09 in SFL2), passing parameters via the `x$hist` data structure to display historical data for a specific record.

---

### **Tables Used**

The program interacts with the following database files:
1. **bicont**: Input-only file for company code validation.
2. **gstabl**: Input-only file, likely containing reference data (e.g., region codes and descriptions).
3. **bbrcsc1**: Input-only file with renamed record format (`bbrcspf` to `bbrcsc1r`), used for reading fuel index records for SFL1.
4. **bbrcscrd**: Input-only file with renamed record format (`bbrcspf` to `bbrcsc2r`), used for reading fuel index records for SFL2.
5. **bbrcsc**: Update/add file for maintaining fuel index records.
6. **bbrcsch**: Output file for writing history records of changes.

---

### **Summary**

The **BB911** program is a comprehensive RPG application for managing National Diesel Fuel Index data. It uses two subfiles (SFL1 and SFL2) to display and manipulate records, with support for create, update, and inquiry operations. The program validates user input, handles database operations, and logs changes to a history file. It integrates with external programs for date validation, message handling, and history inquiries, and uses multiple database files to store and retrieve data. The structured use of subroutines ensures modular processing, with clear separation of concerns for initialization, subfile management, input validation, and database updates.