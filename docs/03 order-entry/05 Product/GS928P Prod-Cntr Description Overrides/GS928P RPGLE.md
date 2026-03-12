### Process Steps of the RPG Program (GS928P)

The program GS928P is an interactive RPG III program for managing Product/Container entries on an IBM i (AS/400) system. It operates in either maintenance (MNT) or inquiry (INQ) mode, based on input parameters. The core functionality revolves around displaying and processing a subfile (SFL) list of records from the GSPRCT file, allowing users to browse, position, filter (include/exclude inactive entries), and perform actions like create, change, inactivate/reactivate, display, or print. It handles user input via function keys, validates data, manages errors/messages, and calls external programs for specific actions.

The high-level process flow is as follows:

1. **Initialization (*INZSR Subroutine)**:
   - Receives entry parameters: `p$mode` (MNT for maintenance or INQ for inquiry) and `p$fgrp` (file group: 'G' or 'Z' to determine file overrides).
   - Sets up work fields, key lists (e.g., for chaining to files), message handling fields, and default values (e.g., subfile page size = 14 records).
   - Prepares date/time stamps and message queue fields.
   - Defines constants for function keys (e.g., F3=Exit, F5=Refresh).

2. **Open Database Tables (OPNTBL Subroutine)**:
   - Applies file overrides using QCMDEXC based on `p$fgrp` (e.g., overrides to GGS* files for 'G' or ZGS* files for 'Z').
   - Opens input files: GSPRCT, GSPRCTRD (renamed record format), BICONT, GSPROD, and GSCNTR1.

3. **Process Subfile (SRSFL1 Subroutine - Main Processing Loop)**:
   - Clears the message subfile and initializes subfile control fields (e.g., company, product, container filters).
   - Sets initial subfile mode (folded/unfolded) and global protection (protect fields in INQ mode).
   - Positions the file cursor using user-provided filters (SF1REP subroutine).
   - Enters a loop to handle user interaction:
     - Displays command line, message subfile (if needed), and subfile control.
     - Writes the subfile control format (SFLCTL1) and reads user input (EXFMT).
     - Clears message indicators and determines cursor position for redisplay.
     - Processes function keys and user actions **before** reading subfile changes:
       - F3: Exit the program.
       - F4: Prompt for fields (e.g., product or container) using external programs (PROMPT subroutine).
       - F5: Refresh the subfile (clear reposition fields and reload).
       - F8: Toggle include/exclude inactive entries filter (updates display label and reloads subfile).
       - F15: Calls external program for printing a listing and adds a confirmation/error message.
       - Page Down: Loads additional subfile records (SF1LOD subroutine).
       - Direct access: If option/company/product/container fields are filled in the control area, processes direct create/change/delete/display (SF1DIR subroutine).
     - If ENTER is pressed, processes subfile changes (SF1PRC subroutine): Reads changed subfile records (READC), validates, and handles selected options (e.g., 2=Change, 4=InActivate/ReActivate, 5=Display).
     - Processes function keys and user actions **after** reading subfile changes:
       - Repositions subfile if position-to fields (company/product/container) are changed.
       - F10: Positions cursor to control record.
   - Loops until exit (F3) or no further actions.
   - Handles repositioning (SF1REP): Clears subfile, edits control input (SF1CTE), positions file, and reloads records.
   - Loads subfile records (SF1LOD): Reads from GSPRCTRD, skips inactive if filtered, formats fields (SF1FMT), applies color coding (SF1COL for deleted/inactive), and writes to subfile.
   - Clears subfile (SF1CLR) when needed (e.g., for repositioning).

4. **Process Subfile Options (SF1CHG and Related Subroutines)**:
   - For each selected subfile record:
     - Retains key values (company, product, container).
     - Chains to GSPRCT for validation.
     - Calls external programs based on option:
       - Option 1 (Create, direct only): Validates keys, calls external program for creation, adds confirmation message, repositions subfile.
       - Option 2 (Change): Checks if record is active, calls external program for change, adds confirmation message.
       - Option 4 (InActivate/ReActivate): Calls external program, processes return flag ('I' or 'A'), adds confirmation message.
       - Option 5 (Display): Calls external program in INQ mode.
     - Updates subfile record after processing (formats and colors it again).

5. **Direct Access Processing (SF1DIR Subroutine)**:
   - Validates direct input fields (e.g., company exists in BICONT, product in GSPROD, container in GSCNTR1).
   - Checks for existence/duplicates in GSPRCT.
   - If valid, processes the option (create/change/delete/display) directly without subfile selection.

6. **Error and Message Handling**:
   - Validates inputs (e.g., cannot modify inactive records).
   - Adds messages to program message queue (ADDMSG subroutine using QMHSNDPM).
   - Writes/clears message subfile (WRTMSG and CLRMSG subroutines using QMHRMVPM).
   - Displays errors (e.g., invalid keys) and sets indicators for field highlighting.

7. **Prompting (PROMPT Subroutine)**:
   - Based on cursor field (e.g., product or container), calls external programs to prompt and return values.

8. **Program End**:
   - Closes all files.
   - Sets *INLR = *ON and returns.

The program emphasizes error handling, validation (e.g., chaining for existence), and user-friendly features like subfile folding, color coding (blue for deleted/inactive), and function key support. It does not perform direct database updates; instead, it calls external programs for CRUD operations.

### External Programs Called

The program calls the following external RPG or utility programs for specific actions (parameters are passed via PARM statements):

- **GS9285**: Called for printing a Product/Container listing (F15). Parameters: file group (`p$fgrp`).
- **GS928**: Called for create (option 1), change (option 2), or display (option 5). Parameters: company, product, container, mode (MNT/INQ), file group, return flag.
- **GS9284**: Called for inactivate/reactivate (option 4). Parameters: company, product, container, file group, return flag.
- **LGSPROD**: Called for prompting product code (F4 on product field). Parameters: company, product, file group.
- **LGSCNTR**: Called for prompting container code (F4 on container field). Parameters: container key, file group.

System utilities called (via CALL):
- **QCMDEXC**: Executes override commands for file redirection.
- **QMHSNDPM**: Sends messages to the program message queue.
- **QMHRMVPM**: Removes messages from the program message queue.

### Tables (Files) Used

The program uses the following database files (PF for physical, LF for logical) and display file. Files are input-only unless noted, and are overridden based on `p$fgrp` (to GGS* for 'G' or ZGS* for 'Z'):

- **GS928PD**: Display file (CF - control format, E - external). Used for workstation interaction, including subfile (SFL1) and message subfile.
- **GSPRCT**: Primary file (IF - input full procedural, E - externally described, K - keyed). Used for chaining and updating Product/Container records.
- **GSPRCTRD**: Renamed record format of GSPRCT (IF, E, K). Used for reading/positioning records in the subfile.
- **BICONT**: Input file (IF, E, K). Used to validate company numbers.
- **GSPROD**: Input file (IF, E, K). Used to validate product codes (with renamed field TPFIL5 to X@FIL5).
- **GSCNTR1**: Input file (IF, E, K). Used to validate container codes.

All files are opened with USROPN and shared access controlled via overrides. No direct writes/updates occur in this program; modifications are handled by called programs.