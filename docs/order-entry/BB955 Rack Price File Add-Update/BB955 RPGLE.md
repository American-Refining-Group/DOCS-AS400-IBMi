The provided `BB955.rpgle` file is an RPGLE (RPG IV) program for the IBM AS/400 (iSeries) system, designed for "Rack Price File Maintenance" within the Bradford Order Entry/Invoices system. It is called from the `BB955.ocl36` OCL program discussed previously. Below, I will explain the process steps, outline the business rules, list the tables used, and identify external programs called, based on the provided RPGLE source code.

### Process Steps of the RPGLE Program (BB955)

The `BB955` program manages the maintenance of rack price data, allowing users to view, update, add, delete, or restore pricing records via a subfile-based interface. The program follows a structured flow, leveraging subroutines to handle initialization, file operations, user input, and data processing. Here’s a step-by-step breakdown of the process:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Sets up the program environment and initializes variables.
   - **Actions**:
     - Receives an input parameter `p$fgrp` (file group, 'G' or 'Z'), which determines the file overrides to use (e.g., `gbicont` vs. `zbicont`).
     - Initializes work fields, subfile control fields (e.g., `rrn1`, `pagsz1`), and message handling fields.
     - Sets up date and time stamps using the program status data structure (`psds##`) for current date (`ps#mdy`) and time (`ps#hms`).
     - Defines key lists (e.g., `kls1s1`, `klbbprce`) for database access.
     - Clears format and reposition fields (e.g., `c1cono`, `s1date`, `s1time`) to prepare for user input.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Purpose**: Opens all necessary database files with appropriate overrides based on the file group (`p$fgrp`).
   - **Actions**:
     - Checks the `p$fgrp` parameter ('G' or 'Z') to apply file overrides from arrays `ovg` or `ovz` (e.g., `ovrdbf file(bicont) tofile(*libl/gbicont)` for 'G').
     - Executes overrides using the `QCMDEXC` API to redirect file access to the correct library (e.g., `QTEMP` for `bb955w`).
     - Opens input files (`bicont`, `gscntr1`, `gsctum`, `gsprod`, `gstabl`, `inloc`, `bbprcw`), update files (`bbprce`, `bb955w`), and output file (`bbprceh`).
     - Ensures files are opened with `usropn` (user-controlled open) for flexibility.

3. **Retrieve Data for Parameters (`rtvdta` Subroutine)**:
   - **Purpose**: Loads initial data for display based on passed parameters.
   - **Actions**:
     - Sets the header for the display (`c$hdr1 = 'Rack Price File Maintenance'`).
     - Likely retrieves initial data for the subfile or control fields (though the subroutine is minimal in the provided code, suggesting additional logic may be truncated).

4. **Process Subfile (`srsfl1` Subroutine)**:
   - **Purpose**: Manages the main user interface loop, displaying and processing a subfile (SFL1) for rack price records.
   - **Actions**:
     - Initializes subfile mode (`sfmod1 = '1'`, folded) and clears the message subfile (`clrmsg`).
     - Sets default behaviors: excludes deleted records (`s1showdel = *off`) and prevents updates to existing records (`s1alwupd = *off`).
     - Enters a loop (`sf1agn`) that:
       - Repositions the subfile if needed (`sf1rep`).
       - Conditions function key availability (e.g., F7=Accept Changes enabled only if subfile records exist).
       - Displays the subfile control record (`sflctl1`) and command line (`sflcmd1`).
       - Handles user input based on function keys (F3, F4, F5, F6, F8, F9, F10, F23) or Enter key.
       - Processes subfile changes (`sf1prc`) when Enter or F7 is pressed.
     - Key actions include:
       - **F3**: Exits the program.
       - **F4**: Prompts for valid values for fields like company, location, or product (`prompt` subroutine).
       - **F5**: Refreshes the subfile.
       - **F6**: Toggles allow/prevent updates for existing records.
       - **F8**: Toggles include/exclude deleted records.
       - **F9**: Calls `GB730P` to display customer history for the selected subfile record.
       - **F10**: Repositions the cursor to the control record.
       - **F23**: Deletes or restores a subfile record (`sf1del`).
       - **Enter/F7**: Processes subfile changes (`sf1prc`) and accepts updates (`sf1f07`).

5. **Process Subfile Changes (`sf1prc` Subroutine)**:
   - **Purpose**: Validates and processes changes made to subfile records.
   - **Actions**:
     - Reads changed subfile records (`readc sfl1`) and processes each (`sf1chg`).
     - Sets a flag (`s1chng`) if changes are detected.
     - Calls `sf1edt` to validate input and `sf1upd` to update the database if no errors occur.

6. **Edit Subfile Input (`sf1edt` Subroutine)**:
   - **Purpose**: Validates user input in the subfile.
   - **Actions**:
     - Clears error indicators (`*in50` to `*in69`, `*in30` to `*in39`).
     - Constructs an item identifier (`w$item = s1prod + '/' + s1cntr + '/' + s1unms`).
     - Validates fields on the left (existing records) and right (new records) sides of the subfile (`sf1edtlt` and assumed `sf1edtrt`, though truncated).
     - Sets error indicators if validation fails.

7. **Update Database (`sf1upd` Subroutine)**:
   - **Purpose**: Updates the database (`bbprce`) with validated subfile changes.
   - **Actions**: Writes or updates records in `bbprce` and logs changes to the history file (`bbprceh`) via `writehist`.

8. **Write to History File (`writehist` Subroutine)**:
   - **Purpose**: Logs changes to the `bbprceh` history file.
   - **Actions**:
     - Captures current date and time (`t#time12`, `ps#dat`).
     - Populates history record fields (e.g., `ihcono`, `ihloc`, `ihprod`, `ihprce`) from the current rack price record.
     - Sets `ihdel` to 'A' (active) or retains the deletion flag.
     - Writes the history record with user and timestamp (`ihuser`, `ihchtm`).

9. **Field Prompting (`prompt` Subroutine)**:
   - **Purpose**: Provides lookup functionality for input fields using F4.
   - **Actions**:
     - Calls external programs (e.g., `LBICONT`, `LINLOC`, `LGSPROD`) to display valid values for fields like company (`c1cono`), location (`c1loc`), or product (`c1prod`).
     - Updates control fields with selected values.

10. **Program Termination**:
    - Closes all open files (`close *all`).
    - Sets `*inlr = *on` to indicate the program is ending and returns control to the caller (the OCL program).

### Business Rules

The program enforces several business rules to ensure data integrity and usability:

1. **Display Compatibility**:
   - The OCL program checks for `DSPLY-27X132` and halts if the terminal uses this format, indicating potential display issues with the subfile interface.

2. **File Group Selection**:
   - The `p$fgrp` parameter ('G' or 'Z') determines which set of files to use (e.g., `gbicont` vs. `zbicont`), allowing the program to operate in different environments or data sets.

3. **Input Validation**:
   - Fields like company (`c1cono`), location (`c1loc`), product (`c1prod`), container (`c1cntr`), unit of measure (`c1unms`), and product class (`c1prcl`) must exist in reference files (`bicont`, `inloc`, `gsprod`, `gscntr1`, `gstabl`).
   - Error messages (e.g., "Invalid Company...F4 To View Valid Values") are displayed if validation fails (see `com` array).
   - Quantities and prices must follow specific rules:
     - Quantities cannot be zero if a price is entered (`Cannot Be Zero If Price Is Entered`).
     - The last quantity must be 9999999 (`Is The Last Qty And Must Be 9999999`).
     - Quantities must be in ascending order (`Is Not In Ascending Order`).
     - Prices cannot be zero if subsequent prices exist (`Cannot Be Zero With Prices Following`).

4. **Subfile Behavior**:
   - The subfile supports folded/unfolded modes, toggled automatically if multiple quantities exist (`w$mltqty`).
   - Deleted records can be included or excluded via F8 (`s1showdel`).
   - Updates to existing records can be allowed or prevented via F6 (`s1alwupd`).
   - New records require an effective date if additional locations are entered (`Must Enter Effective Date For New Entries If Additional Locations Are Entered`).

5. **Data Integrity**:
   - Duplicate pricing entries for the same product, container, unit of measure, and effective date are prevented (`New Pricing Already Entered For Same Product, Container, And UM`).
   - Additional locations cannot match the selected location (`Additional Location Cannot Be Same As Selected Location`).
   - Inactive records (`GSCTUM`, `GSCTWT`, `GSUMCV`) are treated as deleted (`JB05` revision).

6. **History Tracking**:
   - All changes (add, update, delete) are logged to the `bbprceh` history file with user, date, and time stamps (`writehist`).

7. **User Interface**:
   - Function keys (F3, F4, F5, F6, F8, F9, F10, F23) provide specific actions (e.g., exit, prompt, refresh, toggle modes, delete/restore).
   - The subfile supports cursor positioning (F10) and error highlighting (`SFLNXTCHG`).

### Tables Used

The program uses the following files, as defined in the file specifications (`F` specs) and overrides:

1. **Display File**:
   - `bb955d`: Display file for the user interface, containing subfile `sfl1` and control record `sflctl1`.

2. **Input Files**:
   - `bicont`: Company master file (likely contains valid company codes).
   - `gscntr1`: Container master file (replaced `GSCNTR` per `JK03`, uses an alpha key).
   - `gsctum`: Customer master file (possibly for customer-specific data).
   - `gsprod`: Product master file (contains product codes and details).
   - `gstabl`: Table file (likely for codes like unit of measure or product class).
   - `inloc`: Location master file (contains valid location codes).
   - `bbprcw`: Rack price work file (renamed to `bbprcepw` to avoid naming conflicts).

3. **Update Files**:
   - `bbprce`: Primary rack price file, used for adding or updating pricing records.
   - `bb955w`: Work file in `QTEMP`, used for temporary data storage (`jk04` override).

4. **Output File**:
   - `bbprceh`: History file for logging changes to rack price records (`JK01`).

### External Programs Called

The program calls the following external programs, primarily for field prompting (F4) and history inquiry:

1. **LBICONT**: Displays valid company codes for `c1cono`.
2. **LINLOC**: Displays valid locations for `c1loc` or additional locations (`loc(x)`).
3. **LGSPROD**: Displays valid products for `c1prod` or positioning product (`c1ppos`).
4. **LGSCNTR**: Displays valid containers for `c1cntr`.
5. **LGSTABL**: Displays valid units of measure for `c1unms`.
6. **LPRODCL**: Displays valid product classes for `c1prcl`.
7. **GB730P**: Displays customer history for a selected subfile record (called with `x$custhist` parameters).
8. **QCMDEXC**: System API to execute file override commands.
9. **QMHSNDPM**: System API to send program messages to the message queue.
10. **QMHRMVPM**: System API to remove messages from the message queue.

### Additional Notes

- **Truncated Code**: The provided code is truncated (94,989 characters omitted), particularly in subroutines like `sf1edt`, which may contain additional validation logic. This limits a full analysis of all subfile editing rules.
- **File Overrides**: The `jk04` revision ensures `bb955w` is overridden to `QTEMP`, isolating temporary data. The `ovg` and `ovz` arrays allow dynamic file selection based on the environment.
- **Error Handling**: The program uses a message subfile (`msgctl`) and indicators (`*in50` to `*in69`, `*in21` to `*in39`) to display errors, ensuring user feedback for invalid inputs.
- **Revisions**:
   - `JK01` (11/09/12): Added history file logging (`bbprceh`).
   - `JB02` (04/25/13): Changed `c1cono` to `o$cono` in calls to fix a definition issue.
   - `JK02` (06/25/13): Updated timestamp before writing to the history file.
   - `JK03` (02/24/16): Replaced `GSCNTR` with `GSCNTR1` for an alpha key.
   - `JB05` (11/01/16): Treated inactive records in `GSCTUM`, `GSCTWT`, and `GSUMCV` as deleted.
   - `jk04` (07/30/21): Added override for `bb955w` to `QTEMP`.

### Final Answer

- **Process Steps**:
  1. Initialize program variables, key lists, and work fields (`*inzsr`).
  2. Open database files with overrides based on `p$fgrp` (`opntbl`).
  3. Retrieve initial data for display (`rtvdta`).
  4. Process the subfile (`srsfl1`):
     - Display and manage subfile records.
     - Handle function keys (F3, F4, F5, F6, F8, F9, F10, F23) and Enter.
     - Validate and update subfile changes (`sf1prc`, `sf1chg`, `sf1edt`, `sf1upd`).
  5. Log changes to the history file (`writehist`).
  6. Provide field prompting via external programs (`prompt`).
  7. Close files and exit.

- **Business Rules**:
  - Enforces display compatibility (via OCL).
  - Validates input fields against master files.
  - Ensures quantities and prices follow ascending order and non-zero rules.
  - Prevents duplicate pricing entries and invalid locations.
  - Logs all changes to the history file.
  - Supports toggling deleted records and update permissions.
  - Provides user-friendly prompting and error messages.

- **Tables Used**:
  - Display: `bb955d` (subfile `sfl1`, control `sflctl1`).
  - Input: `bicont`, `gscntr1`, `gsctum`, `gsprod`, `gstabl`, `inloc`, `bbprcw`.
  - Update: `bbprce`, `bb955w`.
  - Output: `bbprceh`.

- **External Programs Called**:
  - `LBICONT`, `LINLOC`, `LGSPROD`, `LGSCNTR`, `LGSTABL`, `LPRODCL`, `GB730P`, `QCMDEXC`, `QMHSNDPM`, `QMHRMVPM`.

If you have the truncated portions of the code or additional details, I can provide a more comprehensive analysis of specific subroutines or validations. Let me know if you need further clarification or a search for related information!