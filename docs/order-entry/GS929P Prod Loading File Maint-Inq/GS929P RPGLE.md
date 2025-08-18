The RPG program `GS929P` is designed to manage product load entries within an IBM i (AS/400) environment. It is a workstation program that interacts with a display file and multiple database files to allow users to work with product load data, including creating, updating, copying, inactivating/reactivating, and displaying records. Below, I will outline the process steps of the program, list the external programs called, and identify the database tables used .

---

### Process Steps of GS929P

The program follows a structured flow to manage product load entries through a subfile-based user interface. The steps are organized around the mainline logic and subroutines, as described below:

#### 1. **Initialization (*INZSR Subroutine)**
   - **Purpose**: Sets up initial parameters and defines variables.
   - **Steps**:
     - Receives entry parameters: `p$mode` (run mode: 'MNT' for maintenance or 'INQ' for inquiry) and `p$fgrp` (file group: 'Z' or 'G').
     - Defines fields for repositioning (`r$`), holding control fields (`h$`), and output parameters (`o$`).
     - Initializes subfile control fields, such as `rrn1` (relative record number) and `pagsz1` (page size set to 14).
     - Sets up headers, message handling fields, and key lists for database access.
     - Captures the current date and time for use in the program.

#### 2. **Open Database Tables (opntbl Subroutine)**
   - **Purpose**: Opens the required database files with appropriate overrides based on the file group (`p$fgrp`).
   - **Steps**:
     - Checks if `p$fgrp` is 'G' or 'Z' to apply the corresponding file overrides (`ovg` or `ovz` arrays) using the `QCMDEXC` system API.
     - Opens the following files: `prdlod1`, `prdlodrd`, `bicont`, `gsprod`, `gscntr1`, `gsmlcd`, and `inloc`.

#### 3. **Process Subfile (srsfl1 Subroutine)**
   - **Purpose**: Manages the display and interaction with the subfile (`sfl1`) for product load entries.
   - **Steps**:
     - Clears the message subfile and writes initial messages.
     - Initializes subfile mode (`sfmod1` set to '1' for folded display) and control fields (`c1co`, `c1loc`, etc.).
     - Sets global protection mode (`*in70`) based on `p$mode` ('MNT' allows editing; 'INQ' protects fields).
     - Repositions the file cursor using `sf1rep`.
     - Enters a main loop (`sf1agn`) that:
       - Handles repositioning if requested (`repsfl` flag).
       - Displays the command line and message subfile.
       - Checks for existing subfile records to enable/disable display (`*in41`).
       - Toggles between folded/unfolded subfile mode (`*in45`).
       - Displays the subfile control record (`sflctl1`) using `EXFMT`.
       - Clears message subfile if needed.
       - Processes user input based on function keys and direct access fields:
         - **F03**: Exits the program.
         - **F04**: Calls the `prompt` subroutine for field prompting.
         - **F05**: Refreshes the subfile by clearing reposition fields.
         - **F08**: Toggles include/exclude inactive entries filter (`w$inact`).
         - **F15**: Calls `GS9295` to print a product/load listing.
         - **Direct Access (d1opt)**: Processes direct input for creating or editing records (`sf1dir`).
         - **Page Down**: Loads additional subfile records (`sf1lod`).
         - **Enter**: Processes subfile changes (`sf1prc`).
         - **User Positioning**: Repositions the subfile if control fields change (`sf1rep`).
         - **F10**: Resets the cursor to the control record.

#### 4. **Reposition Subfile (sf1rep Subroutine)**
   - **Purpose**: Clears and repositions the subfile based on user input.
   - **Steps**:
     - Clears the subfile (`sf1clr`) and resets `rrn1`.
     - Validates control field input (`sf1cte`).
     - Positions the file cursor using `SETLL` on `prdlodrd`.
     - Loads the subfile (`sf1lod`).
     - Retains control field values for future repositioning.

#### 5. **Load Subfile Records (sf1lod Subroutine)**
   - **Purpose**: Populates the subfile with records from `prdlodrd`.
   - **Steps**:
     - Sets the starting relative record number (`rrn1`) to the last saved value (`rrnsv1`).
     - Loads up to `pagsz1` (14) records per page.
     - Reads records from `prdlodrd` and applies filters:
       - Excludes deleted/inactive records if `w$inact` is off.
       - Filters based on control fields (`c1racd`, `c1mlcd`, `c1type`, `c1hazm`, `c1prty`, `c1resp`).
     - Formats each subfile line (`sf1fmt`) and applies color coding (`sf1col`).
     - Writes records to the subfile and updates `rrn1`.

#### 6. **Format Subfile Detail Line (sf1fmt Subroutine)**
   - **Purpose**: Populates subfile fields with data from the database record.
   - **Steps**:
     - Clears the subfile record.
     - Moves data from `prdlodrd` fields (e.g., `pdcono`, `pdloc`, `pdprod`) to subfile fields (`s1co`, `s1loc`, `s1prod`, etc.).
     - Sets the delete flag (`s1del`) if the record is marked as deleted or inactive.

#### 7. **Subfile Color Coding (sf1col Subroutine)**
   - **Purpose**: Applies color coding to subfile records.
   - **Steps**:
     - Sets indicator `*in72` to color the record blue if it is marked as deleted or inactive (`s1del` is 'D' or 'I').

#### 8. **Direct Access Processing (sf1dir Subroutine)**
   - **Purpose**: Handles direct input for creating or editing records.
   - **Steps**:
     - Validates input fields for option 1 (create):
       - Checks for valid company number (`bicont`), loading location (`inloc`), product code (`gsprod`), container code (`gscntr1`), responsibility area/major location (`gsmlcd`), and product type (`BULK`, `PACKAGED`, `RAILCAR`).
       - Ensures load priority is not zero and sequence number is valid.
       - Verifies the record does not already exist (`klsfl1`).
     - Processes the selected option (1: create, 2: change, 3: copy, 4: inactivate/reactivate, 5: display) by calling appropriate subroutines (`sf1s01`, `sf1s02`, `sf1s03`, `sf1s04`, `sf1s05`).
     - Clears direct input fields upon successful processing.

#### 9. **Process Subfile on Enter (sf1prc Subroutine)**
   - **Purpose**: Processes changes made to subfile records when the user presses Enter.
   - **Steps**:
     - Reads changed subfile records (`READC sfl1`).
     - Calls `sf1chg` to process each changed record.

#### 10. **Process Subfile Record Change (sf1chg Subroutine)**
   - **Purpose**: Handles changes to a subfile record based on the selected option.
   - **Steps**:
     - Retains selected values in `s$` fields.
     - Processes options:
       - **Option 2**: Change (calls `sf1s02` if not deleted/inactive).
       - **Option 3**: Copy (calls `sf1s03`).
       - **Option 4**: Inactivate/Reactivate (calls `sf1s04`).
       - **Option 5**: Display (calls `sf1s05`).
     - Updates the subfile record after processing.

#### 11. **Subfile Option Subroutines (sf1s01, sf1s02, sf1s03, sf1s04, sf1s05)**
   - **sf1s01 (Create)**:
     - Calls `GS929` with parameters to create a new record.
     - Sends a confirmation message if successful (`o$flag = '1'`).
     - Triggers subfile repositioning.
   - **sf1s02 (Change)**:
     - Validates that the record is not deleted/inactive.
     - Calls `GS929` to update the record.
     - Sends a confirmation message if successful.
   - **sf1s03 (Copy)**:
     - Calls `sf1cpy` to handle copying via a window interface.
   - **sf1s04 (Inactivate/Reactivate)**:
     - Calls `GS9294` to inactivate or reactivate the record.
     - Sends a confirmation message based on the return flag ('I' for inactivated, 'A' for reactivated).
   - **sf1s05 (Display)**:
     - Calls `GS929` in inquiry mode to display the record.

#### 12. **Copy Product Load Records (sf1cpy Subroutine)**
   - **Purpose**: Manages the copy operation via a window (`sflcpy1`).
   - **Steps**:
     - Initializes window fields and retrieves company name (`bcname`).
     - Displays the copy window and processes input:
       - **F04**: Calls `prompt` for field prompting.
       - **F12**: Cancels the copy operation.
       - **F22**: Processes the copy input.
     - Validates input (`sf1cpyedt`) and calls `GS9293` to create the copied record.
     - Sends a confirmation message and triggers subfile repositioning.

#### 13. **Edit Copy Window Input (sf1cpyedt Subroutine)**
   - **Purpose**: Validates input in the copy window.
   - **Steps**:
     - Checks for valid loading location, product code, container code, responsibility area/major location, load priority, sequence number, and product type.
     - Ensures the target record does not already exist.
     - Sets error indicators and messages if validation fails.

#### 14. **Field Prompting (prompt Subroutine)**
   - **Purpose**: Provides lookup functionality for input fields.
   - **Steps**:
     - Based on the current record format (`SFLCTL1` or `SFLCPY1`) and field, calls external programs:
       - `LINLOC` for location (`D1LOC`, `C1LOC`, `S$TLOC`).
       - `LGSPROD` for product code (`D1PROD`, `C1PROD`, `S$TPROD`).
       - `LGSCNTR1` for container code (`D1CNTR`, `C1CNTR`, `S$TCNTR`).
       - `LGSMLCD` for responsibility area/major location (`D1RACD`, `C1RACD`, `S$TRACD`).
     - Updates fields with returned values if valid.

#### 15. **Message Handling (addmsg, wrtmsg, clrmsg Subroutines)**
   - **addmsg**: Sends messages to the program message queue using `QMHSNDPM`.
   - **wrtmsg**: Writes the message subfile (`msgctl`) for display.
   - **clrmsg**: Clears the message subfile using `QMHRMVPM` and resets the message flag.

#### 16. **Program Termination**
   - Closes all files and sets `*inlr` to `*on` to end the program.

---

### External Programs Called

The program calls the following external programs:
1. **QCMDEXC**: Executes file override commands for database files.
2. **GS929**: Handles create, change, and display operations for product load records (called in `sf1s01`, `sf1s02`, `sf1s05`).
3. **GS9293**: Processes the copy operation for product load records (called in `sf1cpy`).
4. **GS9294**: Manages inactivate/reactivate operations (called in `sf1s04`).
5. **GS9295**: Generates a product/load listing (called for F15).
6. **LINLOC**: Provides lookup for loading location (called in `prompt`).
7. **LGSPROD**: Provides lookup for product code (called in `prompt`).
8. **LGSCNTR1**: Provides lookup for container code (called in `prompt`).
9. **LGSMLCD**: Provides lookup for responsibility area/major location (called in `prompt`).
10. **QMHSNDPM**: Sends messages to the program message queue (called in `addmsg`).
11. **QMHRMVPM**: Removes messages from the program message queue (called in `clrmsg`).

---

### Database Tables Used

The program interacts with the following database files (all input-only and user-opened):
1. **prdlod1**: Primary product load file.
2. **prdlodrd**: Product load file with renamed record format (`prdlodpf` to `prdlodpr`).
3. **bicont**: Company file for validating company numbers.
4. **gsprod**: Product file for validating product codes.
5. **gscntr1**: Container file for validating container codes.
6. **gsmlcd**: Responsibility area/major location file for validation.
7. **inloc**: Location file for validating loading locations.

---

### Summary
`GS929P` is a comprehensive RPG program for managing product load entries through a subfile-based interface. It supports creating, changing, copying, inactivating/reactivating, and displaying records, with robust validation and error handling. The program integrates with multiple database files and external programs to provide lookup and processing capabilities, ensuring data integrity and user interaction efficiency.