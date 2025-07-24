The `BB929` RPGLE program is part of the Billing and Invoicing system and is designed for CSR (Customer Service Representative) ID Entry Maintenance and Inquiry. It is called by the main program `BB929P` to handle the creation, modification, or inquiry of individual CSR records. Below, I will explain the process steps, outline the business rules, list the tables used, and identify any external programs called.

---

### Process Steps of the RPGLE Program (`BB929`)

The program is an interactive workstation-based application that uses a display file (`bb929d`) to present a single record format (`FMT01`) for maintaining or inquiring CSR ID records. It operates in either maintenance (`MNT`) or inquiry (`INQ`) mode, allowing users to create or update CSR records or view them without modification. Below is a step-by-step explanation of its process flow, based on the mainline logic and subroutines:

1. **Program Initialization (`*inzsr` Subroutine)**:
   - **Parameter Reception**: The program receives five parameters:
     - `p$co`: Company number (input).
     - `p$crid`: CSR ID (input).
     - `p$mode` (3 characters): Run mode (`MNT` for maintenance, `INQ` for inquiry).
     - `p$fgrp` (1 character): File group (`Z` or `G`) for database overrides.
     - `p$flag` (1 character): Return flag (output, set to `1` on successful update/create).
   - **Field Initialization**:
     - Moves input parameters `p$co` and `p$crid` to display file fields `f$co` and `f$crid`.
     - Initializes output parameters (`o$co`, `o$crid`, `o$mode`, `o$fgrp`, `o$flag`) to blanks or zeros.
     - Sets up message handling fields (`dspmsg`, `m@pgmq`, `m@key`).
     - Defines key list `klcsr` for accessing the `bbcsr` file using `f$co` and `f$crid`.
     - Initializes flags (`fmtagn`, `delagn`, `winagn`) to `*OFF`.
   - **Commented Test Code**: Includes commented-out code for testing with hardcoded values (e.g., `p$mode = 'MNT'`).

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **File Overrides**: Based on `p$fgrp` (`Z` or `G`), applies database overrides using the `QCMDEXC` API to redirect file access to the appropriate library (`gbbcsr` or `zbcsr` for `bbcsr`, `gbicont` or `zbicont` for `bicont`).
   - **File Opening**: Opens two files:
     - `bbcsr` (update/add, keyed access).
     - `bicont` (input-only, keyed access).
   - **Company Validation**: Chains to `bicont` using `f$co` to validate the company number (no specific error handling is implemented here).

3. **Retrieve Data (`rtvdta` Subroutine)**:
   - **Record Retrieval**: Chains to `bbcsr` using the key list `klcsr` (`f$co`, `f$crid`) to check if the record exists.
     - If `*IN99` is `*ON` (record not found), clears the `bbcsrpf` record format and sets `w$exists` to `*OFF`.
     - If `*IN99` is `*OFF` (record found), sets `w$exists` to `*ON`.
   - **Header and Protection Setup**:
     - If `p$mode` is `MNT`, sets `*IN70` to `*OFF` (no global protection) and sets the header `c$hdr1` to "CSR Id Entry Maintenance".
     - If `p$mode` is `INQ`, sets `*IN70` to `*ON` (global protection) and sets `c$hdr1` to "CSR Id Inquiry".

4. **Process Panel Formats (`srfmt` Subroutine)**:
   - **Clear Screen**: Writes the `clrscr` format to clear the display.
   - **Initialize Format**: Calls `f01mov` to initialize fields for the `FMT01` format and sets `w$fmt` to `FMT01`.
   - **Main Loop (`fmtagn`)**:
     - **Display Messages**: If `dspmsg` is `*ON`, calls `wrtmsg` to display the message subfile; otherwise, writes `clrscr`.
     - **Clear Format Change Indicator**: Sets `*IN19` to `*OFF`.
     - **Display Format**: 
       - If `w$fmt` is `FMT01`, calls `f01pro` to set protection indicators and executes `exfmt fmt01` to display the format and accept user input.
       - If `w$fmt` is not `FMT01`, defaults to processing `FMT01` (note: `FMT02` is commented out, indicating it may not be used).
     - **Clear Error Indicators**: Resets indicators `*IN50` to `*IN69` to zero.
     - **Clear Cursor Position**: Clears `row` and `col`.
     - **Clear Messages**: If `dspmsg` is `*ON`, calls `clrmsg` to clear the message subfile.
     - **Process Format Input**: If the current format is `FMT01`, calls `f01sr` to process user input.
     - The loop continues until `fmtagn` is `*OFF`.

5. **Process Format (`f01sr` Subroutine)**:
   - **Handle User Input**:
     - **F04 (Field Prompting)**: Calls `prompt` to handle field prompting.
     - **F10 (Position Cursor Home)**: Clears `row` and `col` to reposition the cursor.
     - **F12 (Exit)**: Sets `fmtagn` to `*OFF` to exit the loop.
     - **Inquiry Mode**: If `p$mode` is `INQ`, calls `f01nxt` to determine the next format (which terminates the loop since `*IN19` is `*OFF`).
     - **Enter Key**:
       - Calls `f01edt` to validate input fields.
       - If no errors (`*IN50` is `*OFF`) and in `MNT` mode, calls `upddbf` to update the database.
       - Calls `f01nxt` to determine the next format (terminates the loop if `*IN19` is `*OFF`).

6. **Determine Next Format (`f01nxt` Subroutine)**:
   - If `*IN19` is `*OFF`, sets `fmtagn` to `*OFF` to exit the main loop.
   - (Note: Code for `FMT02` is commented out, suggesting only `FMT01` is used.)

7. **Edit Format Input (`f01edt` Subroutine)**:
   - **Validate CSR Name**: If `crcrnm` (CSR name) is blank, sets error `ERR0012`, sets `*IN50` and `*IN51` to `*ON`, and calls `addmsg`.
   - **Validate Email Address**: If `cremal` (email address) is blank, sets error `ERR0012`, sets `*IN50` and `*IN52` to `*ON`, and calls `addmsg`.
   - **Inquiry Mode**: If `p$mode` is `INQ`, clears error indicators (`*IN50` to `*IN69`) and clears messages using `clrmsg`.

8. **Initialize Format Field Values (`f01mov` Subroutine)**:
   - Calls `f01edt` to validate fields.
   - If errors exist (`*IN50` is `*ON`), clears error indicators and messages.

9. **Format Protection Schemes (`f01pro` Subroutine)**:
   - Clears protection indicators `*IN70` to `*IN74` to `0`.
   - If `p$mode` is not `MNT` (i.e., `INQ`), sets `*IN70` to `*IN73` to `1` for global input protection.
   - If in `MNT` mode and the record exists (`w$exists` is `*ON`), sets `*IN71` to `*ON` to protect key fields (`f$co`, `f$crid`).

10. **Update Database (`upddbf` Subroutine)**:
    - Saves the current `wkds01` (external data structure for `bbcsr`) to `svds`.
    - Chains to `bbcsr` using `klcsr`:
      - If the record exists (`*IN80` is `*OFF`):
        - If `svds` differs from `wkds01` (indicating changes), restores `svds` to `wkds01`, updates the `bbcsrpf` record, and sets `p$flag` to `1`.
        - If no changes, forces end-of-data (`feod`) on `bbcsr`.
      - If the record does not exist (`*IN80` is `*ON`):
        - Clears `bbcsrpf`, restores `svds` to `wkds01`, sets `crco` (company), `crcrid` (CSR ID), and `crdel` to `A` (active), writes the new record, and sets `p$flag` to `1`.

11. **Field Prompting (`prompt` Subroutine)**:
    - Calculates cursor position (`row`, `col`) using `csrloc`.
    - Checks the current format (`rcdnam` = `FMT01`) but has no specific logic implemented (likely a placeholder for field-specific prompting).
    - Sets `*IN19` to `*ON` to indicate a panel format change.

12. **Message Handling (`addmsg`, `wrtmsg`, `clrmsg` Subroutines)**:
    - **Add Message (`addmsg`)**:
      - Sets `dspmsg` to `*ON`.
      - Calculates the length of `m@data` and sends a message to the program message queue using `QMHSNDPM`.
      - Clears message data fields.
    - **Write Message Subfile (`wrtmsg`)**:
      - Sets `*IN49` to `*ON` and writes the `msgctl` format to display messages.
    - **Clear Message Subfile (`clrmsg`)**:
      - Sets `dspmsg` to `*OFF`.
      - Saves and restores `rcdnam` and `pagrrn`.
      - Clears messages using `QMHRMVPM`.

13. **Program Termination**:
    - Closes all files.
    - Sets `*INLR` to `*ON` and returns.

---

### Business Rules

The `BB929` program enforces the following business rules for managing CSR ID records:

1. **Operational Modes**:
   - **Maintenance Mode (`MNT`)**: Allows creation and updating of CSR records. Key fields (`f$co`, `f$crid`) are protected if the record already exists (`w$exists` is `*ON`). Input validation is performed, and the database is updated if no errors occur.
   - **Inquiry Mode (`INQ`)**: Displays CSR records in read-only mode, with all input fields protected (`*IN70` to `*IN73` set to `1`). No database updates are performed.

2. **Input Validation**:
   - **CSR Name (`crcrnm`)**: Must not be blank; otherwise, error `ERR0012` is displayed, and `*IN51` is set.
   - **Email Address (`cremal`)**: Must not be blank; otherwise, error `ERR0012` is displayed, and `*IN52` is set.
   - In inquiry mode, validation errors are cleared, and no updates are attempted.

3. **Database Operations**:
   - **Create Record**: If the record does not exist (`*IN80` is `*ON`), a new record is written to `bbcsr` with `crdel` set to `A` (active).
   - **Update Record**: If the record exists and changes are detected (`svds` differs from `wkds01`), the record is updated.
   - **No Changes**: If no changes are detected, the file is set to end-of-data (`feod`).
   - **Success Flag**: Sets `p$flag` to `1` on successful create or update.

4. **Field Protection**:
   - In maintenance mode, key fields (`f$co`, `f$crid`) are protected for existing records to prevent modification.
   - In inquiry mode, all input fields are protected to prevent any changes.

5. **File Overrides**:
   - The program uses the `p$fgrp` parameter to determine the file library (`Z` or `G`) and applies overrides to access `gbbcsr` or `zbcsr` for `bbcsr`, and `gbicont` or `zbicont` for `bicont`.

6. **User Interface**:
   - Displays a single format (`FMT01`) for entering or viewing CSR data.
   - Supports function keys:
     - **F04**: Initiates field prompting (logic not fully implemented).
     - **F10**: Repositions the cursor to the home position.
     - **F12**: Exits the program.
   - Provides message feedback for validation errors and successful operations.

7. **Message Handling**:
   - Uses the `GSMSGF` message file in `*LIBL` for error messages (e.g., `ERR0012` for blank fields).
   - Displays messages in a message subfile (`msgctl`) and clears them as needed.

---

### Tables (Files) Used

The program uses the following files:
1. **bb929d**:
   - Type: Workstation file (display file).
   - Usage: Contains the `FMT01` format (and possibly `FMT02`, commented out) for user interaction, along with a message subfile (`msgctl`) and clear screen format (`clrscr`).
   - Handler: `PROFOUNDUI(HANDLER)` for Profound UI integration.
2. **bbcsr**:
   - Type: Physical file (update/add, keyed access).
   - Usage: Stores CSR records, used for creating, updating, and retrieving records. Accessed using the key list `klcsr` (`f$co`, `f$crid`).
   - Override: Redirected to `gbbcsr` or `zbcsr` based on `p$fgrp`.
3. **bicont**:
   - Type: Physical file (input-only, keyed access).
   - Usage: Used to validate the company number (`f$co`).
   - Override: Redirected to `gbicont` or `zbicont` based on `p$fgrp`.

---

### External Programs Called

The program calls the following external programs (all IBM i APIs):
1. **QCMDEXC**:
   - Purpose: Executes file override commands to redirect `bbcsr` and `bicont` to the appropriate library.
   - Parameters:
     - `dbov##` (80 characters, override command).
     - `dbol##` (15.5, command length).
2. **QMHSNDPM**:
   - Purpose: Sends messages (e.g., error `ERR0012`) to the program message queue for display in the message subfile.
   - Parameters: Message ID, message file, message data, data length, message type, program queue, stack counter, message key, error code.
3. **QMHRMVPM**:
   - Purpose: Clears messages from the program message queue.
   - Parameters: Program queue, stack counter, message key, remove option, error code.

---

### Summary

The `BB929` RPGLE program, called by `BB929P`, is responsible for the maintenance and inquiry of individual CSR ID records in the Billing and Invoicing system. It operates in `MNT` or `INQ` mode, allowing users to create or update CSR records (with validation for non-blank name and email fields) or view them in read-only mode. The program uses a single display format (`FMT01`) with Profound UI integration, enforces field protection based on mode and record existence, and updates the `bbcsr` file accordingly. It validates the company number against `bicont` and uses file overrides to access the correct library (`Z` or `G`). The program relies on IBM i APIs (`QCMDEXC`, `QMHSNDPM`, `QMHRMVPM`) for file overrides and message handling, and interacts with three files: `bb929d` (display), `bbcsr` (CSR data), and `bicont` (company validation).