The RPGLE program `AR9154` is a specialized program within the Customer Master Information system, designed to handle the deletion or restoration of customer form type contact records. It is called from the main program `AR915P` to process the delete (option 4) functionality for a specific contact record. The program presents a confirmation window (`delwdw`) to the user, allowing them to either restore a previously deleted record (F22) or mark an active record as deleted (F23). Below is a detailed explanation of the process steps, business rules, database tables used, and external programs called.

---

### Process Steps of the AR9154 Program

The `AR9154` program follows a structured flow to manage the deletion or restoration of a customer form type contact record through a display file window. The process steps are organized by the main subroutines:

1. **Program Initialization (`*inzsr`)**:
   - **Purpose**: Initializes variables, defines key lists, and processes input parameters.
   - **Actions**:
     - Defines the parameter list for receiving input parameters: `p$cono` (company), `p$seq#` (sequence number), `p$fgrp` (file group: 'Z' or 'G'), and `p$flag` (return flag).
     - Moves input parameters to display file fields (`f$cono`, `f$seq#`).
     - Initializes message handling fields (`dspmsg`, `m@pgmq`, `m@key`) and date validation parameters (`pld010`).
     - Defines key lists (`klcufm` for `arcufm`, `klcust` for `arcust`) for database operations.

2. **Open Database Tables (`opntbl`)**:
   - **Purpose**: Opens the required database files with appropriate overrides based on the file group (`p$fgrp`).
   - **Actions**:
     - Checks if `p$fgrp` is 'G' or 'Z' to apply the correct file overrides (`ovg` or `ovz`) using the `QCMDEXC` command.
     - Opens files `arcust`, `bicont`, and `arcufm` with user-controlled open (`usropn`).

3. **Retrieve Data (`rtvdta`)**:
   - **Purpose**: Retrieves data for the provided company and sequence number to populate the display file fields.
   - **Actions**:
     - Chains to `arcufm` using `klcufm` (company, sequence number) to retrieve the contact record.
     - If the record exists (`*in99 = *off`):
       - Checks the deletion status (`fmdel`):
         - If deleted (`fmdel = 'D'`), sets the header to "Customer Form Type Contacts Restore" (`hdr(01)`), function key label to "F22=Restore" (`fky(01)`), and enables F22 (`*in72`).
         - If not deleted, sets the header to "Customer Form Type Contacts Delete" (`hdr(02)`), function key label to "F23=Delete" (`fky(02)`), and enables F23 (`*in73`).
     - Chains to `bicont` to retrieve the company name (`bcname`) into `f$conm`; clears if not found.
     - Chains to `arcust` to retrieve the customer name (`arname`) into `f$csnm`; clears if not found.
     - Moves the contact name (`fmcntc`) to the display field `s1cntc`.

4. **Process Window (`prcwdw`)**:
   - **Purpose**: Manages the main loop for displaying and processing the deletion/restoration confirmation window (`delwdw`).
   - **Actions**:
     - Enters a loop (`winagn`) that:
       - Displays the message subfile if needed (`wrtmsg`) or clears it (`msgclr`).
       - Displays the `delwdw` format using `exfmt`.
       - Clears the message subfile (`clrmsg`) if displayed.
       - Clears error indicators (`*in50`-`*in69`).
       - Processes user input:
         - **F12**: Exits the window by setting `winagn` to off.
         - **F22 or F23**: Validates input (`winedt`), and if no errors (`*in50 = *off`), updates the database (`winupd`) and exits the loop.
         - **Other (e.g., ENTER)**: Validates input (`winedt`) and redisplays the window.
     - Continues until `winagn` is turned off.

5. **Edit Window Input (`winedt`)**:
   - **Purpose**: Validates user input, specifically checking for activity before deletion.
   - **Actions**:
     - If F23 (delete) is pressed, calls `chkact` to check for activity (e.g., open orders).
     - Note: The `chkact` subroutine has commented-out logic for checking open orders, so no validation currently occurs.

6. **Check Activity Prior to Deleting (`chkact`)**:
   - **Purpose**: Intended to verify if the record can be deleted (e.g., no open orders).
   - **Actions**:
     - Currently, the logic is commented out, so no checks are performed, and deletion proceeds without validation.
     - Commented logic would check `arcust` for open orders and display an error (`ERR0000` with `com(01)`: "This Company Has Assigned Customers, Cannot Delete") if found.

7. **Update Database from Window Input (`winupd`)**:
   - **Purpose**: Updates the `arcufm` record to mark it as deleted or restored.
   - **Actions**:
     - For F22 (Restore):
       - Chains to `arcufm` using `klcufm`.
       - If the record exists and is deleted (`fmdel = 'D'`), sets `fmdel` to 'A' (active), updates the record, and sets `p$flag = 'A'`.
     - For F23 (Delete):
       - Chains to `arcufm` using `klcufm`.
       - If the record exists and is not deleted (`fmdel != 'D'`), sets `fmdel` to 'D', updates the record, and sets `p$flag = 'D'`.

8. **Message Handling (`addmsg`, `wrtmsg`, `clrmsg`)**:
   - **Purpose**: Manages error and confirmation messages displayed in the message subfile.
   - **Actions**:
     - `addmsg`: Sends messages to the program message queue using `QMHSNDPM` with message ID, file, data, and type.
     - `wrtmsg`: Writes the message subfile control (`msgctl`) with `*in49` on.
     - `clrmsg`: Clears the message subfile using `QMHRMVPM`.

9. **Program Termination**:
   - **Purpose**: Closes files and exits.
   - **Actions**:
     - Closes all open files (`close *all`).
     - Sets `*inlr` to `*on` and returns control to the calling program (`AR915P`).

---

### Business Rules

The program enforces the following business rules:
1. **Record Deletion**:
   - A record can be marked as deleted (`fmdel = 'D'`) using F23 if it is currently active (`fmdel != 'D'`).
   - The `p$flag` is set to 'D' to indicate successful deletion.
   - Note: The check for open orders or other activity that might prevent deletion is commented out, so deletion is unrestricted.

2. **Record Restoration**:
   - A deleted record (`fmdel = 'D'`) can be restored to active status (`fmdel = 'A'`) using F22.
   - The `p$flag` is set to 'A' to indicate successful restoration.

3. **Display Logic**:
   - If the record is deleted, the window displays "Customer Form Type Contacts Restore" with F22 enabled.
   - If the record is active, the window displays "Customer Form Type Contacts Delete" with F23 enabled.
   - Company and customer names are retrieved for display, but the operation proceeds even if they are not found.

4. **Input Validation**:
   - No additional input validation is performed beyond checking the record's deletion status, as `chkact` logic is disabled.
   - The program assumes the provided company and sequence number are valid, relying on `AR915P` for prior validation.

5. **Return Flag**:
   - The `p$flag` is set to 'D' for deletion or 'A' for restoration, which `AR915P` uses to display confirmation messages (`com(05)` for deletion, `com(06)` for reactivation).

---

### Database Tables Used

The program uses the following database files, all opened with `usropn`:
1. **arcust**:
   - Purpose: Customer master file for retrieving customer names.
   - Usage: Chained to using `klcust` (company, customer) to retrieve `arname` (customer name) for display.
   - Access: Input only, keyed (`if e k disk`).
   - Override: `garcust` (for 'G' file group) or `zarcust` (for 'Z' file group).

2. **bicont**:
   - Purpose: Company master file for retrieving company names.
   - Usage: Chained to using `f$cono` to retrieve `bcname` (company name) for display.
   - Access: Input only, keyed (`if e k disk`).
   - Override: `gbicont` (for 'G') or `zbicont` (for 'Z').

3. **arcufm**:
   - Purpose: Primary file for customer form type contact records.
   - Usage: Chained to using `klcufm` (company, sequence number) to retrieve and update the record’s deletion status (`fmdel`).
   - Access: Update, keyed (`uf e k disk`).
   - Override: `garcufm` (for 'G') or `zarcufm` (for 'Z').

4. **ar9154d**:
   - Purpose: Display file for the user interface.
   - Usage: Contains the `delwdw` format and message subfile control (`msgctl`) for displaying the confirmation window.
   - Access: Work station file (`cf e workstn`).

---

### External Programs Called

The program interacts with the following external programs:
1. **QCMDEXC**:
   - Called in subroutine `opntbl`.
   - Parameters: `dbov##` (override command), `dbol##` (command length).
   - Purpose: Executes file override commands for `arcust`, `bicont`, and `arcufm`.

2. **QMHSNDPM**:
   - Called in subroutine `addmsg`.
   - Parameters: `m@id` (message ID), `m@msgf` (message file), `m@data` (message data), `m@l` (message length), `m@type` (message type), `m@pgmq` (program message queue), `m@scnt` (stack counter), `m@key` (message key), `m@errc` (error code).
   - Purpose: Sends messages to the program message queue.

3. **QMHRMVPM**:
   - Called in subroutine `clrmsg`.
   - Parameters: `m@pgmq` (program message queue), `m@scnt` (stack counter), `m@rmvk` (message key), `m@rmv` (remove option), `m@errc` (error code).
   - Purpose: Removes messages from the program message queue.

---

### Summary

The `AR9154` program, called by `AR915P`, provides a simple confirmation window for deleting or restoring customer form type contact records. It updates the `arcufm` file by setting the `fmdel` field to 'D' (deleted) or 'A' (active) based on user input (F23 or F22). The program retrieves company and customer names for display but does not currently enforce activity checks (e.g., open orders) due to commented-out logic. It uses three database files (`arcust`, `bicont`, `arcufm`) with dynamic overrides and relies on system programs for message handling and file overrides. The program ensures a straightforward user interaction with clear feedback via the `p$flag` return value.