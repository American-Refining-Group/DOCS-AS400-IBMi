The RPG program `BB946.rpgle` is part of the Bradford Order Entry/Invoices system and is designed to handle the deletion or reactivation of customer freight records (`BICUFR`) by marking them as deleted (`bfdel = 'D'`) or reactivating them (`bfdel = *blank`). The program uses a display file (`bb946d`) with a subfile (`SFL1`) to allow users to view, delete, or reactivate customer freight records based on company, customer, ship-to, and location data. It is called by another program (likely `BB945.rpgle`) and logs changes to a history file (`bicufrh`). Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of `BB946.rpgle`

The program operates through a series of subroutines to initialize, display, and process customer freight records. The main steps are:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Receives Input Parameters** (via `*entry` plist):
     - `p$cono`: Company number.
     - `p$sqky`: Sequence key (for specific record selection).
     - `p$fgrp`: File group ('Z' or 'G' for library selection).
     - `p$flag`: Return flag (output, indicates processing status).
   - Initializes the display file (`bb946d`) and subfile (`SFL1`) fields, setting `rrn1` (relative record number) to zero.
   - Sets the program message queue (`m@pgmq = '*'`) and initializes message handling fields (`dspmsg`, `m@key`).
   - Sets the current date (`t#cymd`) and time (`t#hms`) using the system timestamp (`t#time`) for validation and history logging.
   - Defines key lists (`klship`, `klcnty`) for accessing files (`shipto`, `gscntr1`).
   - Sets the report header (`c$hdr1`) to "Customer Freight Entry Delete" or "Customer Freight Entry Re-Activate" based on processing context.
   - Initializes work fields (e.g., `l$cncd`, `w$cnty`) and output parameters (`o$fgrp`, `o$mode`, `o$flag`).

2. **Open Database Tables (`opntbl` Subroutine)**:
   - Applies file overrides based on `p$fgrp` ('Z' for `z*` libraries, `G` for `g*` libraries) using the `ovg` or `ovz` arrays and `QCMDEXC`.
   - Opens input files (`gscntr1`, `gstabl`, `shipto`) and update/output files (`bicuf2`, `bicufrh`) in user-open mode (`USROPN`).

3. **Main Processing Loop (`srsfl1` Subroutine)**:
   - Clears the subfile (`SFL1`) and message subfile (`clrmsg`).
   - Positions the file cursor (`sf1rep`) to load customer freight records into `SFL1`.
   - Enters a loop (`sf1agn`) to display and process the subfile:
     - Writes the command line (`sflcmd1`) and message subfile (`wrtmsg`) if errors exist.
     - Sets subfile display indicators:
       - `*in41`: Enables subfile control display (`SFLDSPCTL`).
       - `*in42`: Enables subfile display (`SFLDSP`) if records exist (`rrn1 > 0`).
       - `*in43`: Sets `SFLEND` if no more records are available.
     - Displays the subfile control format (`sflctl1`) using `EXFMT`.
     - Processes user input:
       - **F03/F12**: Exits the program or returns to the calling program.
       - **F05**: Refreshes the subfile, clearing positioning fields (e.g., `c$cono`, `c$cust`, `c$ship`).
       - **F22**: Reactivates selected records (sets `bfdel = *blank`).
       - **F23**: Deletes selected records (sets `bfdel = 'D'`).
       - **Page Down**: Loads additional subfile records (`sf1lod`).
       - **Enter**: Processes subfile changes (`sf1prc`).
     - Handles repositioning requests based on control fields (e.g., `c$cono`, `c$cust`, `c$ship`) and repositions the subfile (`sf1rep`).
     - Sets the cursor position (`row1`, `col1`) for the next display.

4. **Reposition Subfile (`sf1rep` Subroutine)**:
   - Clears `SFL1` (`*in40`) and resets `rrn1`.
   - Validates control fields:
     - Company (`c$cono`): Chains to `shipto` or `gscntr1` for validation.
     - Customer (`c$cust`): Ensures valid customer number.
     - Ship-to (`c$ship`): Validates ship-to location.
   - Positions the cursor in `bicuf2` using the key list `klship` (company: `bfcono`, customer: `bfcust`, ship-to: `bfship`).
   - Loads `SFL1` with customer freight records (`sf1lod`).

5. **Load Subfile (`sf1lod` Subroutine)**:
   - Reads records from `bicuf2` based on company, customer, ship-to, and location.
   - Filters records based on control fields and sequence key (`p$sqky`, if provided).
   - Formats each subfile line (`sf1fmt`):
     - Populates subfile fields (`s1cono`, `s1cust`, `s1ship`, `s1loc`, `s1efdt`, `s1exdt`, `s1caid`, `s1cncd`) from `bicuf2`.
     - Converts effective (`bfefdt`) and expiration (`bfexdt`) dates to MMDDYY format.
     - Retrieves container code descriptions from `gscntr1`.
   - Applies color coding (`sf1col`):
     - `*in71`: Expired records (`bfexdt < t#cymd`).
     - `*in72`: Deleted records (`bfdel = 'D'`).
   - Writes records to `SFL1` until the page size (typically 14 records) is reached or no more records are available.

6. **Process Subfile on Enter (`sf1prc` Subroutine)**:
   - Reads changed subfile records using `READC` (`*in88`).
   - Processes user-selected options (`s1opt`):
     - **4**: Deletes the record by setting `bfdel = 'D'` (`sf1s04`).
     - **6**: Reactivates the record by clearing `bfdel` (`sf1s06`).
   - Validates the record by chaining to `bicuf2` using key fields.
   - Updates `bicuf2` and writes a history record to `bicufrh` (`writehist`).
   - Clears the option field (`s1opt`) and updates the subfile.

7. **Delete Record (`sf1s04` Subroutine)**:
   - Sets `bfdel = 'D'` to mark the record as deleted.
   - Updates `bicuf2` and logs the change to `bicufrh` with user ID, timestamp, and record data.

8. **Reactivate Record (`sf1s06` Subroutine)**:
   - Clears `bfdel` to reactivate the record.
   - Updates `bicuf2` and logs the change to `bicufrh`.

9. **Write History (`writehist` Subroutine)**:
   - Writes a history record to `bicufrh` with fields like `lhcono`, `lhcust`, `lhship`, `lhefdt`, `lhexdt`, `lhdel`, and user/timestamp data.
   - Sets deletion flag (`lhdel = 'D'`) if the record is deleted.

10. **Message Handling**:
    - **Add Message (`addmsg`)**: Sends error messages (e.g., invalid company or customer) to the program message queue using `QMHSNDPM`.
    - **Write Message (`wrtmsg`)**: Displays messages in the message subfile.
    - **Clear Message (`clrmsg`)**: Clears the message subfile.

11. **Program End**:
    - Closes all files (`gscntr1`, `gstabl`, `shipto`, `bicuf2`, `bicufrh`, `bb946d`).
    - Sets `*inlr = *on` and returns control to the calling program with `p$flag` updated.

---

### Business Rules

The program enforces the following business rules to ensure accurate deletion and reactivation of customer freight records:
1. **Deletion and Reactivation**:
   - Records in `bicuf2` are marked as deleted by setting `bfdel = 'D'` (F23) or reactivated by clearing `bfdel` (F22).
   - Deleted records are displayed with turquoise coloring (`*in72`), and expired records (expiration date before current date) are flagged with `*in71`.

2. **Validation**:
   - Company (`c$cono`) must exist in `shipto` or `gscntr1`.
   - Customer (`c$cust`) and ship-to (`c$ship`) must be valid.
   - Container codes are validated against `gscntr1` (replaced `gscntr` per JK01, 02/24/16).

3. **History Tracking**:
   - All deletion and reactivation actions are logged to `bicufrh` with user ID, timestamp, and record details.

4. **Subfile Navigation**:
   - Users can filter records by company, customer, ship-to, and location using control fields.
   - Supports pagination (`Page Down`), refresh (`F05`), and exit (`F03/F12`).

5. **Error Handling**:
   - Invalid inputs (e.g., non-existent company or customer) trigger error messages displayed in the message subfile.
   - The program ensures only valid records are processed for deletion or reactivation.

---

### Tables (Files) Used

The program accesses the following files, with overrides applied based on `p$fgrp` ('Z' or 'G'):
1. **bb946d**: Display file (workstation file with subfile `SFL1` and message subfile).
2. **gscntr1**: Container code file (input only, replaced `gscntr` per JK01).
3. **gstabl**: Table file for validation (input only).
4. **shipto**: Ship-to master file (input only).
5. **bicuf2**: Customer freight file (update, add).
6. **bicufrh**: Customer freight history file (output only, per JK01).

---

### External Programs Called

The program calls the following external programs:
1. **QCMDEXC**: Executes file override commands for `gscntr1`, `gstabl`, `shipto`, `bicuf2`, and `bicufrh`.
2. **QMHSNDPM**: Sends messages to the program message queue for error handling.
3. **QMHRMVPM**: Removes messages from the program message queue.
4. **GSDTEDIT** (assumed, not explicitly shown): Validates date fields for effective and expiration dates.

---

### Summary

The `BB946.rpgle` program, called by another program (likely `BB945.rpgle`), facilitates the deletion and reactivation of customer freight records in the `BICUFR` file. It uses a subfile (`SFL1`) in the display file `bb946d` to present records, allowing users to mark records as deleted (`bfdel = 'D'`) or reactivate them (`bfdel = *blank`) using F23 and F22, respectively. Changes are logged to the history file `bicufrh`, and the program supports validation, filtering, and error handling. The revision (JK01, 02/24/16) updated the container code file to `gscntr1`. The program integrates with external programs for file overrides and message handling, ensuring robust management of freight data.