### Process Steps of the RPG Program (GS928)

The program GS928 is an interactive RPG III program for maintaining or inquiring about Product/Container entries on an IBM i (AS/400) system. It is designed to be called from another program (e.g., GS928P, as referenced in the query) and operates in either maintenance (MNT) or inquiry (INQ) mode based on input parameters. The core functionality involves displaying a single-panel format (FMT01) for viewing or editing a specific Product/Container record, validating input, and performing create/update operations on the database if in MNT mode. It handles user input via function keys, manages errors/messages, and applies field protection rules. Note that a second format (FMT02) is commented out, suggesting it may be incomplete or unused. The program does not use subfiles; it's a simple panel-based entry/inquiry screen.

The high-level process flow is as follows:

1. **Initialization (*INZSR Subroutine)**:
   - Receives entry parameters: `p$co` (company), `p$prod` (product code), `p$cntr` (container code), `p$mode` (MNT for maintenance or INQ for inquiry), `p$fgrp` (file group: 'G' or 'Z' to determine file overrides), and `p$flag` (output flag to indicate success).
   - Moves input parameters to display fields (`f$co`, `f$prod`, `f$cntr`).
   - Sets up work fields, key lists (e.g., for chaining to GSPRCT and GSPROD), message handling fields, and default values.
   - Defines constants for function keys (e.g., F3=Exit (but not used), F4=Prompt, F12=Previous/Exit).
   - Prepares output parameters and clears flags.

2. **Open Database Tables (OPNTBL Subroutine)**:
   - Applies file overrides using QCMDEXC based on `p$fgrp` (e.g., overrides to GGS* files for 'G' or ZGS* files for 'Z').
   - Opens files: GSPRCT (for update/add), BICONT, GSPROD, and GSCNTR1.
   - Chains to BICONT to validate/fetch company (though no error if not found).
   - Chains to GSPROD to fetch product description (`tpdesc` to `f$prds`).
   - Chains to GSCNTR1 to fetch container description (`tcdesc` to `f$ctds`).

3. **Retrieve Data for Passed Parameters (RTVDTA Subroutine)**:
   - Chains to GSPRCT using the passed keys (`p$co`, `p$prod`, `p$cntr`).
   - If record not found, clears the record buffer and sets `w$exists` to *OFF (indicating a create scenario in MNT mode).
   - If found, sets `w$exists` to *ON.
   - Sets the screen header (`c$hdr1`) based on mode: "Product/Container Entry Maintenance" for MNT or "Product/Container Inquiry" for INQ.
   - Applies global field protection (*IN70=*ON) in INQ mode.

4. **Process Panel Formats (SRFMT Subroutine - Main Processing Loop)**:
   - Clears the screen.
   - Initializes panel fields from database (F01MOV subroutine, which also calls F01EDT to validate but clears errors if any).
   - Sets initial format to 'FMT01'.
   - Enters a loop to handle user interaction:
     - Displays message subfile if errors/messages exist (WRTMSG subroutine); otherwise, clears the screen.
     - Sets *IN19=*OFF (no format change).
     - Displays the format (EXFMT FMT01) after applying protection (F01PRO subroutine).
     - Clears error indicators (*IN50-*IN69).
     - Clears cursor position variables (`row`, `col`).
     - Clears message subfile if displayed (CLRMSG subroutine).
     - Processes the current format (F01SR subroutine):
       - Handles function keys:
         - F4: Calls PROMPT subroutine for field prompting (but the subroutine is empty/no-op in the provided code; no actual prompting occurs).
         - F10: Positions cursor to home (clears `row` and `col`).
         - F12: Exits the loop (sets `fmtagn`=*OFF).
       - In INQ mode, skips to next format determination (F01NXT).
       - On ENTER (default case):
         - Edits input (F01EDT subroutine).
         - If no errors (*IN50=*OFF) and in MNT mode, updates the database (UPDDBF subroutine).
         - Determines next format (F01NXT subroutine; but since FMT02 is commented, it effectively exits the loop if no input change).
   - Loops until exit (F12) or no further actions (e.g., no input change in *IN19=*OFF).

5. **Edit Format Input (F01EDT Subroutine)**:
   - Validates that description line 1 (`ptdes1`) is not blank; if blank, adds error message 'ERR0012' and sets *IN50/*IN51=*ON.
   - In INQ mode, clears all errors and messages.

6. **Format Protection Schemes (F01PRO Subroutine)**:
   - Clears protection indicators (*IN70-*IN74=*OFF).
   - In INQ mode (not MNT), protects all fields (*IN70-*IN73=*ON).
   - In MNT mode, if record exists (`w$exists`=*ON), protects key fields (*IN71=*ON).

7. **Update Database (UPDDBF Subroutine)**:
   - Saves current record buffer to `svds`.
   - Chains to GSPRCT.
   - If record found:
     - If fields have changed (`svds` != current buffer), restores from `svds` and updates the record.
     - Forces end-of-data (FEOD) if no change.
   - If not found (create scenario):
     - Clears record, restores from `svds`, sets keys, sets delete flag `ptdel`='A' (active), and writes new record.
   - Sets output `p$flag`='1' on success (create or update).

8. **Initialize Format Fields (F01MOV Subroutine)**:
   - Calls F01EDT for initial validation but clears errors if any.

9. **Error and Message Handling**:
   - Adds messages to program message queue (ADDMSG subroutine using QMHSNDPM).
   - Writes/clears message subfile (WRTMSG and CLRMSG subroutines using QMHRMVPM).
   - Displays errors (e.g., blank description) and sets indicators for field highlighting.

10. **Program End**:
    - Closes all files.
    - Sets *INLR = *ON and returns (with `p$flag` indicating if changes were made).

The program is straightforward for single-record maintenance/inquiry, with no subfile or multi-record handling. It assumes the calling program (e.g., GS928P) manages listing/browsing.

### Business Rules

- **Mode-Specific Behavior**:
  - In MNT (maintenance) mode: Allows editing and saving (create/update). Key fields are protected if the record already exists (`w$exists`=*ON). Updates only if fields have changed.
  - In INQ (inquiry) mode: All fields are protected (*IN70-*IN73=*ON). No database updates; clears any errors for display-only.

- **Validation Rules**:
  - Description line 1 (`ptdes1`) is required and cannot be blank (error 'ERR0012').
  - Keys (company, product, container) are validated indirectly via chaining (descriptions fetched if exist), but no explicit errors if invalid (assumes caller validated).
  - New records are created with `ptdel`='A' (active status).

- **Database Operations**:
  - Create: Only if record not found; writes new with keys and 'A' status.
  - Update: Only if record found and fields changed; no delete/reactivate logic here (handled elsewhere, e.g., GS9284).
  - No direct deletes; uses soft-delete flag `ptdel`.

- **User Interface Rules**:
  - Fields protected based on mode and existence.
  - Headers change by mode.
  - Function keys: F4 (prompting, but unimplemented), F10 (home cursor), F12 (exit/return to caller).
  - Errors displayed via message subfile; input changes tracked via *IN19.

- **General**:
  - File overrides ensure data separation ('G' vs. 'Z' groups, possibly for different environments/companies).
  - Output flag `p$flag`='1' signals successful create/update to caller.
  - No auditing/timestamps beyond what's in the file (e.g., no user/time stamps added here).

### Tables (Files) Used

The program uses the following database files (PF for physical, LF for logical) and display file. Files are input-only unless noted, and are overridden based on `p$fgrp` (to GGS* for 'G' or ZGS* for 'Z'):

- **GS928D**: Display file (CF - control format, E - external). Used for workstation interaction (FMT01 panel and message subfile).
- **GSPRCT**: Primary file (UF A - update full procedural with add, E - externally described, K - keyed). Used for chaining, updating, and writing Product/Container records.
- **BICONT**: Input file (IF, E, K). Used to validate/fetch company details (chained but no data used beyond existence).
- **GSPROD**: Input file (IF, E, K). Used to fetch product description (with renamed field TPFIL5 to X@FIL5).
- **GSCNTR1**: Input file (IF, E, K). Used to fetch container description.

All files are opened with USROPN and shared access controlled via overrides.

### External Programs Called

This program does not call any external RPG or custom programs. It only uses system utilities (via CALL):

- **QCMDEXC**: Executes override commands for file redirection.
- **QMHSNDPM**: Sends messages to the program message queue.
- **QMHRMVPM**: Removes messages from the program message queue.

Note: The PROMPT subroutine is present but empty (no actual calls to prompting programs like LGSPROD or LGSCNTR, unlike in GS928P).