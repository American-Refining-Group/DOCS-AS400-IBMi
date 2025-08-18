The RPG program `GS929` is a workstation program designed for product load maintenance and inquiry within an IBM i (AS/400) environment. It is called from the main program `GS929P` to handle the creation, modification, and display of individual product load records. The program interacts with a display file and database files, providing a user interface for entering or viewing product load details, with validation to ensure data integrity. Below, I outline the process steps, business rules, database tables used, and external programs called .

---

### Process Steps of GS929

The program follows a structured flow to manage product load records through two panel formats (`FMT01` and `FMT02`). The steps are organized around the mainline logic and subroutines:

#### 1. **Initialization (*INZSR Subroutine)**
   - **Purpose**: Sets up initial parameters and variables.
   - **Steps**:
     - Receives input parameters: `p$cono` (company), `p$loc` (load location), `p$prod` (product), `p$cntr` (container), `p$prty` (priority), `p$seq#` (sequence number), `p$racd` (responsibility area), `p$mlcd` (major location), `p$type` (product type), `p$mode` (run mode: 'MNT' for maintenance or 'INQ' for inquiry), `p$fgrp` (file group: 'Z' or 'G'), and `p$flag` (return flag).
     - Moves input parameters to format fields (`f$`) and holding fields (`h$`) for processing and comparison.
     - Defines output parameters (`o$`), key lists (`klprct`, `klprcth`, `klprod`), and work fields.
     - Initializes message handling fields, display file fields, and format control flags (`fmtagn`, `delagn`, `dspmsg`).
     - Sets up a data structure (`wkds01`) to mirror the `prdlod1` file record format.

#### 2. **Open Database Tables (opntbl Subroutine)**
   - **Purpose**: Opens required database files with appropriate overrides.
   - **Steps**:
     - Checks if `p$fgrp` is 'G' or 'Z' and applies file overrides from `ovg` or `ovz` arrays using the `QCMDEXC` system API.
     - Opens files: `prdlod1`, `bicont`, `gsprod`, and `gscntr1`.
     - Retrieves descriptions for the product (`f$prds` from `gsprod`) and container (`f$ctds` from `gscntr1`) if records exist.

#### 3. **Retrieve Data for Passed Parameters (rtvdta Subroutine)**
   - **Purpose**: Retrieves existing record data or initializes a new record.
   - **Steps**:
     - Chains to `prdlod1` using the key list `klprct` to check for an existing record.
     - If no record is found (`*in99 = *on`):
       - Sets `*in46` to indicate a new record.
       - Clears the `prdlodpf` record format.
       - Marks the record as non-existent (`w$exists = *off`).
     - If a record is found, sets `w$exists` to `*on`.
     - Sets the screen header (`c$hdr1`) and protection mode (`*in70`) based on `p$mode`:
       - 'MNT': Enables input (`*in70 = *off`, header = "Product/Load Entry Maintenance").
       - 'INQ': Protects fields (`*in70 = *on`, header = "Product/Load Inquiry").

#### 4. **Process Panel Formats (srfmt Subroutine)**
   - **Purpose**: Manages the display and interaction with the two panel formats (`FMT01` and `FMT02`).
   - **Steps**:
     - Clears the screen (`clrscr`).
     - Initializes the first panel format (`FMT01`) by calling `f01mov`.
     - Enters a loop (`fmtagn`) to process panels:
       - Displays the message subfile if needed (`wrtmsg`) or clears the screen.
       - Resets the format change indicator (`*in19`).
       - Displays the appropriate format based on `w$fmt`:
         - `FMT01`: Calls `f01pro` and displays with `EXFMT`.
         - `FMT02`: Calls `f02pro` and displays with `EXFMT`.
         - Default: Falls back to `FMT01`.
       - Clears error indicators (`*in50-*in69`).
       - Clears cursor position (`row`, `col`).
       - Clears the message subfile if needed (`clrmsg`).
       - Processes the current format based on the record name (`rcdnam`):
         - `FMT01`: Calls `f01sr`.
         - `FMT02`: Calls `f02sr`.
     - Exits the loop when `fmtagn` is turned off (e.g., F12 is pressed).

#### 5. **Process FMT01 (f01sr Subroutine)**
   - **Purpose**: Handles user input for the first panel format.
   - **Steps**:
     - Processes function keys and input:
       - **F04**: Calls `prompt` for field prompting.
       - **F10**: Resets the cursor to the home position.
       - **F12**: Exits the program by setting `fmtagn` to `*off`.
       - **INQ Mode**: Moves to the next format (`f01nxt`).
       - **Enter**: Validates input (`f01edt`), updates the database if in 'MNT' mode (`upddbf`), and moves to the next format if no errors.
     - If validation passes (`*in50 = *off`), updates the database and proceeds to `FMT02`.

#### 6. **Determine Next Format (f01nxt Subroutine)**
   - **Purpose**: Transitions to the next panel format (`FMT02`).
   - **Steps**:
     - Calls `f02mov` to populate `FMT02` fields.
     - Sets `w$fmt` to 'FMT02'.

#### 7. **Edit FMT01 Input (f01edt Subroutine)**
   - **Purpose**: Validates input fields in `FMT01`.
   - **Steps**:
     - Checks if key fields (`f$prty`, `f$seq#`) have changed and verifies that the new key combination does not already exist in `prdlod1` using `klprct`.
     - Validates fields:
       - **Category Description (`pddesc`)**: Cannot be blank (`ERR0012`).
       - **Common Names (`pdcomm`)**: Cannot be blank (`ERR0012`).
       - **Status (`pdstat`)**: Must be 'A' (active) or 'I' (inactive) (`ERR0000`, message: "Status - Must be 'A' or 'I'").
       - **Carrier Type (`pdcaty`)**: Must be 'TRUCK' or 'RAILCAR' (`ERR0000`, message: "Carrier Type - Must be 'TRUCK' or 'RAILCAR'").
       - **Truck Blend (`pdtkbl`)**: Must be 'Y' or 'N' (`ERR0014`).
       - **Responsible Person (`pdresp`)**: Cannot be blank (`ERR0012`).
       - **Loading Priority (`f$prty`)**: Cannot be zero (`ERR0000`, message: "Loading Priority - Must be a Range of 1-9").
     - Sets error indicators (`*in50`, `*in51-*in56`, `*in77`) and sends error messages (`addmsg`) if validation fails.

#### 8. **Process FMT02 (f02sr Subroutine)**
   - **Purpose**: Handles user input for the second panel format, which includes scheduling fields (e.g., `Sun`, `Mon`, etc.).
   - **Steps**:
     - Processes function keys:
       - **F04**: Calls `prompt` for field prompting.
       - **F05**: Resets scheduling fields to 'N' (`f02rst`) and redisplays `FMT02`.
       - **F09**: Toggles all scheduling fields to 'Y' or 'N' (`f02tgl`) and redisplays.
       - **F12**: Returns to `FMT01` or exits if no changes were made.
       - **F14**: Updates the database (`upddbf`) and exits.
       - **Enter**: Validates input (`f02edt`), updates the database if in 'MNT' mode, and exits if no errors.
     - If validation passes, updates the database and exits the program.

#### 9. **Edit FMT02 Input (f02edt Subroutine)**
   - **Purpose**: Validates scheduling fields in `FMT02`.
   - **Steps**:
     - Ensures at least one scheduling field (e.g., `pdsund`, `pdmond`, etc.) is set to 'Y'.
     - Validates that scheduling fields are either 'Y' or 'N'.
     - Sets error indicators and sends messages (`ERR0016`, `ERR0017`) if validation fails.

#### 10. **Update Database (upddbf Subroutine)**
   - **Purpose**: Updates or creates a record in `prdlod1`.
   - **Steps**:
     - Saves the current record state (`wkds01` to `svds`).
     - Chains to `prdlod1` using `klprcth` to check for an existing record.
     - If the record exists (`*in80 = *off`):
       - Updates the record with new values if key fields or data have changed.
       - Sets `p$flag` to '1' to indicate a successful update.
     - If the record does not exist:
       - Clears the `prdlodpf` record format.
       - Populates fields from input parameters (`f$cono`, `f$loc`, etc.).
       - Sets the delete flag (`pddel`) to 'A' (active).
       - Writes a new record to `prdlod1`.
       - Sets `p$flag` to '1'.
     - Updates `pdprty` and `pdseq#` with `f$prty` and `f$seq#`.

#### 11. **Field Prompting (prompt Subroutine)**
   - **Purpose**: Placeholder for field prompting (incomplete in the provided code).
   - **Steps**:
     - Determines the cursor location for returning from a prompt window.
     - Sets `*in19` to indicate a format change.

#### 12. **Move Data to Panel Formats (f01mov, f02mov Subroutines)**
   - **f01mov**: Populates `FMT01` fields from `prdlod1` or input parameters, retrieves company name (`bcname`), and sets scheduling flags.
   - **f02mov**: Populates `FMT02` scheduling fields (e.g., `pdsund`, `pdmond`) from `prdlod1` using data structures (`Sun`, `Mon`, etc.).

#### 13. **Reset and Toggle Scheduling Fields (f02rst, f02tgl Subroutines)**
   - **f02rst**: Resets all scheduling fields to 'N'.
   - **f02tgl**: Toggles scheduling fields between 'Y' and 'N' based on the current state of `pdsund`.

#### 14. **Message Handling (addmsg, wrtmsg, clrmsg Subroutines)**
   - **addmsg**: Sends error or informational messages to the program message queue using `QMHSNDPM`.
   - **wrtmsg**: Writes the message subfile (`msgctl`) for display.
   - **clrmsg**: Clears the message subfile using `QMHRMVPM`.

#### 15. **Program Termination**
   - Closes all files and sets `*inlr` to `*on` to end the program.

---

### Business Rules

The program enforces the following business rules during validation and processing:

1. **Record Existence**:
   - When changing key fields (`f$prty`, `f$seq#`), the new combination must not already exist in `prdlod1` (`ERR0000`, message: "Cannot Change, Record Already Exists").
2. **Mandatory Fields**:
   - Category Description (`pddesc`) and Common Names (`pdcomm`) cannot be blank (`ERR0012`).
   - Responsible Person (`pdresp`) cannot be blank (`ERR0012`).
3. **Status Validation**:
   - Status (`pdstat`) must be 'A' (active) or 'I' (inactive) (`ERR0000`, message: "Status - Must be 'A' or 'I'").
4. **Carrier Type**:
   - Carrier Type (`pdcaty`) must be 'TRUCK' or 'RAILCAR' (`ERR0000`, message: "Carrier Type - Must be 'TRUCK' or 'RAILCAR'").
5. **Truck Blend**:
   - Truck Blend (`pdtkbl`) must be 'Y' or 'N' (`ERR0014`).
6. **Loading Priority**:
   - Loading Priority (`f$prty`) must be between 1 and 9, not zero (`ERR0000`, message: "Loading Priority - Must be a Range of 1-9").
7. **Product Type**:
   - Product Type (`p$type`) must be 'BULK', 'PACKAGED', or 'RAILCAR' (enforced by `GS929P`, but referenced here).
8. **Scheduling Fields**:
   - At least one scheduling field (e.g., `pdsund`, `pdmond`) must be 'Y' (`ERR0016`).
   - Scheduling fields must be 'Y' or 'N' (`ERR0017`).
9. **Mode-Based Access**:
   - In 'MNT' mode, users can edit and update records.
   - In 'INQ' mode, fields are protected (`*in70 = *on`), allowing only viewing.

---

### Database Tables Used

The program interacts with the following database files:
1. **prdlod1**: Product load file (update/add mode, `UF A`).
2. **bicont**: Company file (input-only, used to retrieve company name).
3. **gsprod**: Product file (input-only, used to retrieve product description).
4. **gscntr1**: Container file (input-only, used to retrieve container description).

All files are user-opened (`USROPN`) and use file overrides based on the `p$fgrp` parameter ('G' or 'Z').

---

### External Programs Called

The program calls the following external programs:
1. **QCMDEXC**: Executes file override commands for database files (`opntbl`).
2. **QMHSNDPM**: Sends messages to the program message queue (`addmsg`).
3. **QMHRMVPM**: Removes messages from the program message queue (`clrmsg`).

Note: The `prompt` subroutine is incomplete in the provided code, so no additional lookup programs (e.g., `LINLOC`, `LGSPROD`) are called, unlike in `GS929P`.

---

### Summary

`GS929` is a focused RPG program called by `GS929P` to handle the maintenance and inquiry of individual product load records. It uses two panel formats (`FMT01` for key fields and metadata, `FMT02` for scheduling) and enforces strict business rules to ensure data integrity. The program supports creating new records, updating existing ones, and displaying records in inquiry mode, with robust validation for fields like status, carrier type, and scheduling. It interacts with four database files and uses system APIs for message handling and file overrides, ensuring seamless integration with the broader application.