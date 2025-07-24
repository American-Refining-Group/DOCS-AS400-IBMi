The RPGLE program `AR915P` is designed to manage customer form type contacts within a Customer Master Information system. It provides an interactive interface to display, create, update, and delete contact records in a subfile (SFL). Below is a detailed explanation of the process steps, external programs called, and database tables used.

---

### Process Steps of the AR915P Program

The program follows a structured flow to handle customer form type contacts using a subfile interface. Here are the key process steps, organized by the main subroutines and their purposes:

1. **Program Initialization (`*inzsr`)**:
   - **Purpose**: Sets up initial variables, defines key lists, and processes entry parameters.
   - **Actions**:
     - Defines the parameter list for receiving input parameters: `a$cono` (company), `a$cust` (customer), `p$mode` (run mode: maintenance or inquiry), and `p$fgrp` (file group: 'Z' or 'G').
     - Initializes subfile control fields (e.g., `rrn1` for relative record number, `pagsz1` for page size set to 28).
     - Sets up message handling fields and key lists (`klsfl1`, `kls1s1`, `kls1r1`, `klcust`) for database operations.
     - Captures the current date and time into `t#time` and converts it to a `CYMD` format for use in the program.

2. **Open Database Tables (`opntbl`)**:
   - **Purpose**: Opens the required database files with appropriate overrides based on the file group (`p$fgrp`).
   - **Actions**:
     - Checks if `p$fgrp` is 'G' or 'Z' to apply the correct file overrides (`ovg` or `ovz`) using the `QCMDEXC` command.
     - Opens files `arcufm`, `arcufmx`, `arcust`, and `bicont` with user-controlled open (`usropn`).

3. **Error Handling (`*pssr`)**:
   - **Purpose**: Handles errors, particularly related to parameter passing.
   - **Actions**:
     - Checks for error code 221 (invalid number of parameters) and sets a return code if detected.
     - Returns control to the calling program if an error occurs.

4. **Process Parameters (`@parms`)**:
   - **Purpose**: Validates and processes input parameters passed to the program.
   - **Actions**:
     - If parameters are provided (`ps#prm >= 1`), moves `a$cono` to `c1cono` (company) and `a$cust` to `c1cust` (customer) if they are non-blank and non-zero.
     - Sets indicator `*in71` to indicate valid parameters were passed.
     - Includes commented-out test values for `c1cono` and `c1cust` (not used in production).

5. **Process Subfile (`srsfl1`)**:
   - **Purpose**: Manages the main logic for displaying and interacting with the subfile.
   - **Actions**:
     - Calls `*pssr` for error handling.
     - Processes parameters via `@parms`.
     - Clears the message subfile (`clrmsg`) and writes it (`wrtmsg`).
     - Initializes subfile mode (`sfmod1`) to folded (`'1'`) and sets `*in45` for folded display.
     - Sets global protection mode (`*in70`) based on `p$mode` ('MNT' for maintenance, else inquiry).
     - Repositions the subfile (`sf1rep`) to the first record.
     - Enters a main loop (`sf1agn`) that:
       - Handles repositioning if requested (`repsfl`).
       - Displays the command line and message subfile.
       - Checks for existing subfile records to enable/disable display (`*in41`).
       - Sets folded/unfolded mode (`*in45`) based on `sfmod1`.
       - Displays the subfile control (`sflctl1`) using `exfmt`.
       - Processes user input based on function keys:
         - **F03**: Exits the program.
         - **F04**: Calls the `prompt` subroutine for field prompting.
         - **F05**: Refreshes the subfile by clearing `r$fmty` and triggering repositioning.
         - **F08**: Toggles include/exclude deleted records (`w$del`) and updates the display label (`s1f08d`).
         - **PAGEDN**: Loads additional subfile records (`sf1lod`).
         - **ENTER**: Processes subfile changes (`sf1prc`).
         - **F06**: Creates a new contact record (`sf1f06`).
         - **F10**: Positions the cursor to the control record.
       - Repositions the subfile if user input changes (`c1cono`, `c1cust`, `c1fmty` differ from retained values).
     - Clears format indicators and updates cursor location (`row1`, `col1`) and subfile record number (`rcdnb1`).

6. **Process Subfile on ENTER (`sf1prc`)**:
   - **Purpose**: Processes user selections in the subfile when the ENTER key is pressed.
   - **Actions**:
     - Reads changed subfile records (`readc sfl1`) and processes each record via `sf1chg` if the subfile is not empty.

7. **Process Subfile Record Change (`sf1chg`)**:
   - **Purpose**: Handles user-selected options (2, 4, 5) for subfile records.
   - **Actions**:
     - For option 2 (Change, maintenance mode, not deleted): Calls `sf1s02`.
     - For option 4 (Delete, maintenance mode): Calls `sf1s04`.
     - For option 5 (Display): Calls `sf1s05`.
     - Updates the subfile record after processing by chaining to `arcufm`, formatting (`sf1fmt`), applying color coding (`sf1col`), and updating the subfile (`sfl1`).

8. **Reposition Subfile (`sf1rep`)**:
   - **Purpose**: Clears and repositions the subfile based on user input.
   - **Actions**:
     - Clears the subfile (`sf1clr`).
     - Validates control input (`sf1cte`).
     - If no errors, positions the file (`arcufmx`) using `kls1s1` and loads subfile records (`sf1lod`).
     - Retains control fields (`c1cono`, `c1cust`, `c1fmty`) for future repositioning.

9. **Edit Subfile Control Input (`sf1cte`)**:
   - **Purpose**: Validates company (`c1cono`) and customer (`c1cust`) input.
   - **Actions**:
     - Chains to `bicont` to validate company code; if valid and not deleted, sets `c1conm` (company name); else, adds an error message.
     - Chains to `arcust` to validate customer code; if valid and not deleted, sets `c1csnm` (customer name); else, adds an error message.
     - Sets error indicators (`*in50`, `*in51`, `*in52`) if validation fails.

10. **Load Subfile Records (`sf1lod`)**:
    - **Purpose**: Loads records into the subfile up to the page size (`pagsz1`).
    - **Actions**:
      - Sets the relative record number (`rrn1`) to the last saved value (`rrnsv1`).
      - Reads records from `arcufmx` using `kls1r1` (company/customer key).
      - Skips deleted records if `w$del` is off.
      - Formats each record (`sf1fmt`), applies color coding (`sf1col`), and writes to the subfile (`sfl1`).
      - Updates `rrn1` and saves it to `rrnsv1`.

11. **Format Subfile Detail Line (`sf1fmt`)**:
    - **Purpose**: Populates subfile fields for display.
    - **Actions**:
      - Clears the subfile record.
      - Moves fields from `arcufmx` (`fmseq#`, `fmfmty`, `fmcntc`, etc.) to subfile fields (`s1seq#`, `s1fmty`, etc.).
      - Sets `s1emfx` and `s1note` based on whether email (`fmemla`) or fax (`fmfax#`) is present.
      - Sets `s1del` if the record is marked deleted.

12. **Subfile Color Coding (`sf1col`)**:
    - **Purpose**: Applies color to subfile records based on their status.
    - **Actions**:
      - Sets `*in76` (blue color) if the record is marked deleted (`s1del = 'D'`).

13. **Clear Subfile (`sf1clr`)**:
    - **Purpose**: Clears the subfile and resets control indicators.
    - **Actions**:
      - Resets `rrn1` and `rrnsv1` to zero.
      - Sets `*in42` (SFLCLR) on, writes `sflctl1`, and turns `*in42` off.
      - Disables subfile display (`*in41`) and control (`*in40`).

14. **Create Contact (`sf1f06`)**:
    - **Purpose**: Initiates creation of a new contact record.
    - **Actions**:
      - Calls program `AR915` with parameters for company, sequence number (zero), customer, mode ('MNT'), file group, and return flag.
      - If the return flag is '1', adds a confirmation message and triggers subfile repositioning.

15. **Change Contact (`sf1s02`)**:
    - **Purpose**: Updates an existing contact record.
    - **Actions**:
      - Validates that the record is not deleted; if deleted, adds an error message.
      - If valid, calls `AR915` with parameters for company, sequence number, customer, mode ('MNT'), file group, and return flag.
      - If the return flag is '1', adds a confirmation message.

16. **Delete Contact (`sf1s04`)**:
    - **Purpose**: Marks a contact record as deleted or reactivated.
    - **Actions**:
      - Calls `AR9154` with parameters for company, sequence number, file group, and return flag.
      - Based on the return flag ('D' for deleted, 'A' for reactivated), adds the appropriate confirmation message.

17. **Display Customer Order (`sf1s05`)**:
    - **Purpose**: Displays contact details in inquiry mode.
    - **Actions**:
      - Calls `AR915` with parameters for company, sequence number, customer, mode ('INQ'), file group, and return flag.

18. **Field Prompting (`prompt`)**:
    - **Purpose**: Provides lookup functionality for company and customer fields.
    - **Actions**:
      - If the cursor is on `C1CONO` and input is not protected (`*in71`), calls `LBICONT` to prompt for a company code.
      - If the cursor is on `C1CUST` and input is not protected, calls `LARCUST` to prompt for a customer code.
      - Sets `*in19` to indicate a panel format change.

19. **Message Handling (`addmsg`, `wrtmsg`, `clrmsg`)**:
    - **Purpose**: Manages error and confirmation messages displayed in the message subfile.
    - **Actions**:
      - `addmsg`: Sends messages to the program message queue using `QMHSNDPM` with message ID, file, data, and type.
      - `wrtmsg`: Writes the message subfile control (`msgctl`) with `*in49` on.
      - `clrmsg`: Clears the message subfile using `QMHRMVPM` and restores the current record format and page number.

20. **Program Termination**:
    - **Purpose**: Closes files and exits.
    - **Actions**:
      - Closes all open files (`close *all`).
      - Sets `*inlr` to `*on` and returns control to the calling program.

---

### External Programs Called

The program interacts with the following external programs:
1. **AR915**:
   - Called in subroutines `sf1f06` (create), `sf1s02` (change), and `sf1s05` (display).
   - Parameters: `o$cono` (company), `o$seq#` (sequence number), `o$cust` (customer), `o$mode` ('MNT' or 'INQ'), `o$fgrp` (file group), `o$flag` (return flag).
   - Purpose: Manages creation, updating, or displaying of contact records.

2. **AR9154**:
   - Called in subroutine `sf1s04` (delete).
   - Parameters: `o$cono` (company), `o$seq#` (sequence number), `o$fgrp` (file group), `o$flag` (return flag).
   - Purpose: Handles deletion or reactivation of contact records.

3. **LBICONT**:
   - Called in subroutine `prompt` for company code lookup.
   - Parameters: `o$cono` (company), `o$fgrp` (file group).
   - Purpose: Provides a lookup window for valid company codes.

4. **LARCUST**:
   - Called in subroutine `prompt` for customer code lookup.
   - Parameters: `o$cono` (company), `o$cust` (customer), `o$fgrp` (file group).
   - Purpose: Provides a lookup window for valid customer codes.

5. **QCMDEXC**:
   - Called in subroutine `opntbl`.
   - Parameters: `dbov##` (override command), `dbol##` (command length).
   - Purpose: Executes file override commands for `arcufm`, `arcufmx`, `arcust`, and `bicont`.

6. **QMHSNDPM**:
   - Called in subroutine `addmsg`.
   - Parameters: `m@id` (message ID), `m@msgf` (message file), `m@data` (message data), `m@l` (message length), `m@type` (message type), `m@pgmq` (program message queue), `m@scnt` (stack counter), `m@key` (message key), `m@errc` (error code).
   - Purpose: Sends messages to the program message queue.

7. **QMHRMVPM**:
   - Called in subroutine `clrmsg`.
   - Parameters: `m@pgmq` (program message queue), `m@scnt` (stack counter), `m@rmvk` (message key), `m@rmv` (remove option), `m@errc` (error code).
   - Purpose: Removes messages from the program message queue.

---

### Database Tables Used

The program uses the following database files, all opened with `usropn` and keyed access (`k disk`):
1. **arcufm**:
   - Purpose: Primary file for customer form type contact records.
   - Usage: Read and chained to for subfile data and updates.
   - Override: `garcufm` (for 'G' file group) or `zarcufm` (for 'Z' file group).

2. **arcufmx**:
   - Purpose: Same as `arcufm` but with a renamed record format (`arcufmhx`).
   - Usage: Used for reading records to populate the subfile.
   - Override: `garcufmx` (for 'G') or `zarcufmx` (for 'Z').

3. **arcust**:
   - Purpose: Customer master file for validating customer codes.
   - Usage: Chained to for validating `c1cust` and retrieving `arname` (customer name).
   - Override: `garcust` (for 'G') or `zarcust` (for 'Z').

4. **bicont**:
   - Purpose: Company master file for validating company codes.
   - Usage: Chained to for validating `c1cono` and retrieving `bcname` (company name).
   - Override: `gbicont` (for 'G') or `zbicont` (for 'Z').

5. **ar915pd**:
   - Purpose: Display file for the user interface.
   - Usage: Contains subfile `sfl1` and control format `sflctl1` for interactive display and input.
   - Handler: Uses `PROFOUNDUI(HANDLER)` for modern UI rendering.

---

### Summary

The `AR915P` program is a robust RPGLE application for managing customer form type contacts. It uses a subfile to display records, supports create, update, delete, and display operations, and includes field prompting and error handling. The program interacts with external programs for specific actions and uses multiple database files with dynamic overrides based on the file group. The process is highly interactive, driven by function keys and user input, with comprehensive message handling for user feedback.