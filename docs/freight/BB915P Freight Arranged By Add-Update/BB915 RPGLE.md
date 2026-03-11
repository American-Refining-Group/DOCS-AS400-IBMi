### Process Steps of the RPG Program (BB915)

This RPG IV (RPGLE) program, named BB915, is a detail-level maintenance/inquiry program for managing individual "Freight Arranged By" entries in a customer freight system (part of the Bradford Order Entry/Invoices system). It is designed to be called from a higher-level program (e.g., the "work with" list program BB915P from the previous query) to handle creation, update, or inquiry for a specific record identified by company code and freight arranged by code. The program operates in either Maintenance (MNT) mode for editing/adding or Inquiry (INQ) mode for viewing only, based on input parameters. It uses a single-screen format (FMT01) to display/edit fields and validates inputs before updating the database. Unlike the list program (BB915P), this focuses on a single record and directly performs database adds/updates (no subfile).

The high-level execution flow is as follows:

1. **Initialization (*INZSR Subroutine)**:
   - Receives entry parameters: `p$co` (company code), `p$fpcd` (freight arranged by code), `p$mode` (MNT for maintenance or INQ for inquiry), `p$fgrp` (Z or G for file group selection), and `p$flag` (output flag to indicate if changes were made).
   - Moves parameters to display fields (e.g., `f$co`, `f$fpcd`).
   - Defines data structures for time/date conversions, message handling, display file feedback, and program status.
   - Sets up constants (e.g., function keys like F12 for exit, ENTER).
   - Initializes work fields, message queues, and key lists (e.g., KLFRPR for chaining by company + freight code).
   - Prepares output parameters (initially blank) and sets flags for loop control.

2. **Open Database Tables (opntbl Subroutine)**:
   - Based on `p$fgrp` (G or Z), applies file overrides using `QCMDEXC` to point to the correct library (e.g., gbbfrpr for G, zbbfrpr for Z).
   - Opens input files: `bicont` (company data) and `bbfrpr` (freight entries, with update/add capability).

3. **Retrieve Data for Passed Parameters (rtvdta Subroutine)**:
   - Chains to `bbfrpr` by key (company + freight code) to fetch the record. If not found (indicator 99 ON), clears the record buffer.
   - Chains to `bicont` by company code to retrieve the company name (clears if not found).
   - Sets screen header based on mode (maintenance or inquiry) from compile-time array.
   - Sets global field protection (*IN70 ON for INQ mode to prevent edits).

4. **Process Panel Formats (srfmt Subroutine - Main Program Loop)**:
   - Writes a clear screen record.
   - Initializes format fields (f01mov - calls editing but clears errors if any).
   - Sets the current format to 'FMT01'.
   - Enters a main DO loop (while `fmtagn` is ON):
     - Displays messages if queued (wrtmsg) or clears the screen.
     - Resets format change indicator (*IN19 OFF).
     - Performs EXFMT on FMT01 (the main detail screen) after applying protection schemes (f01pro - protects fields in INQ mode).
     - Clears error indicators (50-69) and cursor position.
     - Clears messages if displayed.
     - Processes the format input (f01sr):
       - F4: Handles field prompting (prompt - sets *IN19 ON for input change and saves cursor position).
       - F10: Positions cursor to home (row/col=0).
       - F12: Exits the loop (sets `fmtagn` OFF).
       - INQ mode: Skips editing and goes to next format determination (f01nxt).
       - ENTER: Edits input (f01edt). If no errors (*IN50 OFF) and MNT mode, updates the database (upddbf). Then determines next format (f01nxt).
   - Loops until F12 or no more formats (FMT02 is commented out, so it effectively exits after one screen unless input changed).

5. **Determine Next Format (f01nxt Subroutine)**:
   - (Commented out: Would move to FMT02, but currently does nothing.)
   - If no format input change (*IN19 OFF), exits the main loop (sets `fmtagn` OFF).

6. **Edit Format Input (f01edt Subroutine)**:
   - Validates required fields: Name (`fpname`), city (`fpcity`), state (`fpstat`), zip (`fpzip`), country (`fpctry`) - queues error "ERR0012" (likely "Required Field") and sets indicators (50 ON, field-specific like 51-56 ON) if blank.
   - Validates zip+4 (`fpzip9`): Requires zip if zip+4 is entered.
   - Validates arranged by type (`fpfpty`): Must be 'E' or 'I' (queues custom error from compile-time array: "Invalid Response...E or I" via "ERR0000").
   - In INQ mode, clears all errors and messages (no validation needed).

7. **Update Database (upddbf Subroutine)**:
   - Saves current record buffer to a save DS (`svds`).
   - Chains to `bbfrpr` by key.
   - If record exists (indicator 80 OFF):
     - If fields changed (save DS != current), moves save DS back and updates the record. Sets `w$exists` ON and output flag `p$flag` to '1'.
     - If no change, forces end-of-data (FEOD).
   - If record does not exist: Clears buffer, moves save DS back, sets key fields, writes new record, sets `w$exists` ON and `p$flag` to '1'.

8. **Message Handling**:
   - **addmsg**: Sends messages to program message queue using QMHSNDPM (e.g., errors like "ERR0012").
   - **wrtmsg**: Writes to message subfile control.
   - **clrmsg**: Clears message subfile using QMHRMVPM.

9. **Program End**:
   - Closes all files.
   - Sets *INLR ON and returns (with `p$flag` indicating if DB was modified).

### Business Rules
- **Required Fields**: Name, city, state, zip code, and country must be provided; blanks trigger errors.
- **Zip Code Validation**: Zip+4 cannot be entered without a base zip code.
- **Arranged By Type**: Must be exactly 'E' (possibly External) or 'I' (possibly Internal); other values are invalid.
- **Mode-Specific Behavior**:
  - MNT: Allows editing, adding new records (if key not found), or updating existing ones. Only updates if fields changed.
  - INQ: Protects all input fields (*IN70 ON), skips validation on ENTER, and displays read-only.
- **Database Operations**:
  - Adds new record if key (company + freight code) not found.
  - Updates only if record exists and fields differ from original.
  - Sets output flag '1' only if add/update occurred (for caller confirmation).
- **Error Handling**: All errors are queued as messages and displayed; no DB update if errors present. Clears errors in INQ mode.
- **Single-Screen Flow**: Processes one detail screen (FMT01); FMT02 is commented out, suggesting potential future expansion.
- **No Deletion**: This program handles add/update/inquiry only; deletion is likely in a separate program (e.g., BB916 from the previous query).

### Tables (Files) Used
- **bb915d**: Display file (workstation file) for the user interface, including format FMT01 and message subfile.
- **bicont**: Input-only keyed file (company master, used for chaining to retrieve company name).
- **bbfrpr**: Update/add keyed file (freight arranged by entries, primary data file for chain, update, write).

Files are opened with USROPN and overridden based on the file group parameter (to g* or z* prefixed files in *LIBL).

### External Programs Called
- **QCMDEXC**: System program for executing override commands (file redirection).
- **QMHSNDPM**: System program for sending program messages.
- **QMHRMVPM**: System program for removing program messages.

No other business programs are called; this program is self-contained for its operations but is invoked by callers like BB915P for detail-level actions.