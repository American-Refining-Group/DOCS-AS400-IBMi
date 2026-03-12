The provided RPGLE program, `BB928P`, is designed to manage salesman IDs within a billing and invoicing system. It uses a display file with a subfile to interact with users, allowing them to view, create, update, inactivate, reactivate, or print salesman ID records. Below, I’ll explain the process steps, list the external programs called, and identify the database tables used.

### Process Steps of the BB928P Program

The program follows a structured flow, primarily driven by user interaction with a subfile (SFL1) displayed on a workstation. Here’s a breakdown of the key process steps based on the program’s logic:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Parameters**: Receives input parameters `p$mode` (run mode: 'MNT' for maintenance or 'INQ' for inquiry) and `p$fgrp` (file group: 'Z' or 'G').
   - **Genie Check**: Calls `GSGENIE2C` to verify if the program is running in a Genie environment. If `genievar` is not 'YES', the program closes all files and exits.
   - **Field Setup**: Initializes subfile control fields, message handling fields, and key lists (`klsfl1`, `kls1s1`) for database access. Sets up date/time fields using the system timestamp.
   - **Headers**: Sets the display header (`c$hdr1`) from the `hdr` array.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **File Overrides**: Based on `p$fgrp` ('G' or 'Z'), applies database overrides using `ovg` or `ovz` arrays to point to the correct files (`gbbslsm`, `zbbslsm`, `gbicont`, `zbicont`).
   - **File Open**: Opens the database files `bbslsm`, `bbslsmrd` (renamed record format), and `bicont` for input.

3. **Subfile Processing (`srsfl1` Subroutine)**:
   - **Clear Message Subfile**: Calls `clrmsg` to clear any existing messages in the message subfile and `wrtmsg` to write the message subfile control.
   - **Subfile Initialization**: Sets the subfile mode (`sfmod1`) to folded (`#fold = '0'`, `*in45 = *on`) and initializes control fields (`c1co`, `c1smid`) to zeros or blanks.
   - **Inactive Records Filter**: Sets `w$inact` to `*on` (include inactive records) due to the `jb01` revision, which removed the F8 toggle functionality.
   - **Protection Mode**: Sets `*in70` based on `p$mode` (`*off` for 'MNT', `*on` for 'INQ' to protect input fields in inquiry mode).
   - **Initial Positioning**: Calls `sf1rep` to position the file and load the subfile initially.
   - **Main Loop**:
     - **Display**: Writes the command line (`sflcmd1`) and message subfile if needed, checks if subfile records exist to set `*in41` (SFLDSP), and determines folded/unfolded mode (`*in45`).
     - **User Input**: Displays the subfile control (`sflctl1`) using `exfmt` and processes user input based on function keys and direct input:
       - **F3 (Exit)**: Sets `sf1agn` to `*off` to exit the loop.
       - **F4 (Field Prompting)**: Calls `prompt` to set `*in19` for cursor positioning.
       - **F5 (Refresh)**: Clears positioning fields (`r$co`, `r$smid`) and sets `repsfl` to reposition the subfile.
       - **F8 (Toggle Inactive)**: Previously toggled `w$inact` to include/exclude inactive records, but `jb01` revision defaults to include all records.
       - **F15 (Print Listing)**: Calls `BB9285` to print a salesman ID listing, passing `p$fgrp` as a parameter, and displays a confirmation message.
       - **Direct Access**: If `d1opt`, `d1co`, or `d1smid` are non-zero, calls `sf1dir` for direct processing (create, change, inactivate, or display).
       - **Page Down**: Calls `sf1lod` to load more subfile records.
       - **Enter Key**: Calls `sf1prc` to process subfile changes.
       - **Repositioning**: If `c1co` or `c1smid` are non-zero, calls `sf1rep` to reposition the subfile. F10 clears cursor positioning (`row1`, `col1`).
     - **Cursor and Record Number**: Updates cursor location (`row1`, `col1`) and sets the subfile record number (`rcdnb1`) based on `pagrrn`.

4. **Subfile Record Processing (`sf1prc` Subroutine)**:
   - Reads subfile records (`sfl1`) using `readc` until no more changed records (`*in81`).
   - For each changed record, calls `sf1chg` to process user-selected options.

5. **Process Subfile Record Change (`sf1chg` Subroutine)**:
   - Stores selected values (`s1co`, `s1smid`) in `s$co`, `s$smid`.
   - Processes options based on `s1opt`:
     - **Option 2 (Change)**: If in maintenance mode (`p$mode = 'MNT'`) and the record is not deleted/inactive, calls `sf1s02`.
     - **Option 4 (Inactivate/Reactivate)**: If in maintenance mode, calls `sf1s04`.
     - **Option 5 (Display)**: Calls `sf1s05`.
   - Updates the subfile record after processing by chaining to `bbslsm`, clearing `s1opt`, formatting (`sf1fmt`), applying color coding (`sf1col`), and updating `sfl1`.

6. **Reposition Subfile (`sf1rep` Subroutine)**:
   - Clears the subfile (`sf1clr`) and resets `rrn1`.
   - Validates control input (`sf1cte`).
   - Positions the file using `kls1s1` (based on `c1co`, `c1smid`) and loads the subfile (`sf1lod`).
   - Retains control fields (`r$co`, `r$smid`) for future repositioning.

7. **Edit Subfile Control Input (`sf1cte` Subroutine)**:
   - Currently empty, likely intended for future input validation.

8. **Load Subfile Records (`sf1lod` Subroutine)**:
   - Sets `rrn1` to the last saved record number (`rrnsv1`) and `rcdnb1` to the next record number for display.
   - Loads up to `pagsz1` (14) records:
     - Reads the next record from `bbslsmrd`.
     - Skips deleted/inactive records if `w$inact = *off`.
     - Formats the subfile line (`sf1fmt`), applies color coding (`sf1col`), and writes to `sfl1`.
   - Updates `rrnsv1` with the last `rrn1`.

9. **Format Subfile Line (`sf1fmt` Subroutine)**:
   - Clears the subfile record and populates fields (`s1co`, `s1smid`, `s1smnm`, `s1emal`, `s1del`) from the database record.

10. **Subfile Color Coding (`sf1col` Subroutine)**:
    - Sets `*in72` (blue color) for deleted (`s1del = 'D'`) or inactive (`s1del = 'I'`) records.

11. **Direct Access Processing (`sf1dir` Subroutine)**:
    - Validates direct input (`d1opt`, `d1co`, `d1smid`):
      - For option 1 (create), ensures `d1co` and `d1smid` are non-zero and checks if the company number exists in `bicont`.
      - Checks if the record exists in `bbslsm` to prevent duplicate creation or invalid updates.
    - Processes valid options (1, 2, 4, 5) by calling `sf1s01`, `sf1s02`, `sf1s04`, or `sf1s05`.
    - Clears input fields upon successful processing.

12. **Subfile Options**:
    - **sf1s01 (Create)**: Calls `BB928` in maintenance mode, passing `s$co`, `s$smid`, `p$fgrp`, and mode 'MNT'. Displays a confirmation message if successful (`o$flag = '1'`).
    - **sf1s02 (Change)**: Checks if the record is not deleted/inactive, calls `BB928` in maintenance mode, and displays a confirmation message if successful.
    - **sf1s04 (Inactivate/Reactivate)**: Calls `BB9284`, processes the return flag ('I' for inactivated, 'A' for reactivated, 'E' for vendor-linked error), and displays a message.
    - **sf1s05 (Display)**: Calls `BB928` in inquiry mode to display the record.
    
13. **Field Prompting (`prompt` Subroutine)**:
    - Sets `*in19` to indicate a panel format change for cursor positioning.

14. **Message Handling**:
    - **addmsg**: Sends messages to the program message queue using `QMHSNDPM`, setting `dspmsg` to `*on`.
    - **wrtmsg**: Writes the message subfile control (`msgctl`) with `*in49` enabled.
    - **clrmsg**: Clears the message subfile using `QMHRMVPM`, preserving the current record format and `pagrrn`.

15. **Program Exit**:
    - Closes all files, sets `*inlr = *on`, and returns.

### External Programs Called

The program calls the following external programs:
1. **GSGENIE2C**: Checks if the program is running in a Genie environment, returning `genievar` ('YES' or not).
2. **QCMDEXC**: Executes file override commands for `bbslsm`, `bbslsmrd`, and `bicont`.
3. **QMHSNDPM**: Sends messages to the program message queue.
4. **QMHRMVPM**: Removes messages from the program message queue.
5. **BB928**: Handles create, change, and display operations for salesman ID records, called with parameters for company, salesman ID, mode, file group, and return flag.
6. **BB9284**: Manages inactivate/reactivate operations for salesman ID records.
7. **BB9285**: Prints a salesman ID listing, called with the file group parameter.

### Database Tables Used

The program interacts with the following database files:
1. **bbslsm**: Primary input file for salesman ID records, opened with `usropn`.
2. **bbslsmrd**: Input file with a renamed record format (`bbslsmpr`) for salesman ID records, opened with `usropn`.
3. **bicont**: Input file for company records, used for validation, opened with `usropn`.

### Additional Notes
- **Display File**: `bb928pd` is a workstation file with a subfile (`sfl1`) and uses the `PROFOUNDUI(HANDLER)` for the user interface.
- **Indicators**: The program uses indicators (19, 21–79, 90–99) to control display behavior, error handling, and subfile operations.
- **Field Prefixes**: Uses prefixes like `f$`, `c1`, `s1`, `p$`, etc., to organize fields for display, control, and parameters.
- **Revisions**: The `jb01` revision (10/10/2023) modified the program to default to displaying all records (active and inactive) due to issues with Profound UI’s F8 functionality.

This program is a typical IBM i RPG application for interactive data management, leveraging subfiles for user interaction and database operations for data retrieval and manipulation.