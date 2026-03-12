The RPGLE program `BB928` is a component of a billing and invoicing system designed to handle the maintenance and inquiry of salesman ID entries. It is called by the main program `BB928P` (as referenced in the previous document) to perform specific operations such as creating, updating, or displaying salesman ID records. Below, I’ll explain the process steps, outline the business rules, list the database tables used, and identify any external programs called.

### Process Steps of the BB928 Program

The program is structured to interact with a display file (`bb928d`) and manage salesman ID records through a single panel format (`fmt01`). It operates in either maintenance (`MNT`) or inquiry (`INQ`) mode, based on input parameters. Here’s a detailed breakdown of the process steps:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Parameters**: Receives input parameters:
     - `p$co`: Company number.
     - `p$smid`: Salesman ID.
     - `p$mode`: Run mode (`MNT` for maintenance, `INQ` for inquiry).
     - `p$fgrp`: File group (`Z` or `G`).
     - `p$flag`: Return flag to indicate success or failure.
   - **Field Setup**: Moves input parameters `p$co` and `p$smid` to display file fields (`f$co`, `f$smid`). Initializes output parameters (`o$co`, `o$smid`, `o$mode`, `o$fgrp`, `o$flag`) and miscellaneous fields (`fmtagn`, `delagn`, `dspmsg`, `m@pgmq`, `m@key`).
   - **Keylist**: Defines `klslsm` keylist using `f$co` and `f$smid` for database access.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **File Overrides**: Based on `p$fgrp` (`G` or `Z`), applies database overrides from `ovg` or `ovz` arrays to point to the correct files (`gbbslsm` or `zbbslsm`, `gbicont` or `zbicont`) using the `QCMDEXC` program.
   - **File Open**: Opens `bbslsm` (update/add mode) and `bicont` (input mode) with `usropn`. Chains to `bicont` using `f$co` to validate the company number.

3. **Retrieve Data (`rtvdta` Subroutine)**:
   - **Record Retrieval**: Chains to `bbslsm` using `klslsm` (company and salesman ID) to check if the record exists.
     - If the record is not found (`*in99 = *on`), clears the record buffer (`bbslsmpf`) and sets `w$exists` to `*off`.
     - If found, sets `w$exists` to `*on`.
   - **Header and Protection**: Sets the display header (`c$hdr1`) to "Salesman Id Entry Maintenance" or "Salesman Id Inquiry" based on `p$mode`. Sets `*in70` to `*on` for inquiry mode (protecting input fields) or `*off` for maintenance mode.

4. **Process Panel Formats (`srfmt` Subroutine)**:
   - **Clear Screen**: Writes the `clrscr` format to clear the display.
   - **Initial Setup**: Calls `f01mov` to initialize format fields and sets `w$fmt` to `FMT01`.
   - **Main Loop**:
     - Displays the message subfile if `dspmsg` is `*on` (via `wrtmsg`) or clears the screen.
     - Sets `*in19` to `*off` to indicate no format change.
     - Displays `fmt01` using `exfmt` (other formats like `fmt02` are commented out, indicating only `fmt01` is currently used).
     - Clears error indicators (`*in50`–`*in69`) and cursor position (`row`, `col`).
     - Clears the message subfile if `dspmsg` is `*on` (via `clrmsg`).
     - Processes the current format (`fmt01`) by calling `f01sr`.

5. **Process Format (`f01sr` Subroutine)**:
   - Handles user input for `fmt01`:
     - **F4 (Field Prompting)**: Calls `prompt` to set `*in19` for cursor positioning.
     - **F10 (Position Cursor Home)**: Clears cursor position (`row`, `col`).
     - **F12 (Exit)**: Sets `fmtagn` to `*off` to exit the loop.
     - **Inquiry Mode**: If `p$mode = 'INQ'`, calls `f01nxt` to determine the next format (though it currently exits the loop).
     - **Enter Key**:
       - Validates input fields via `f01edt`.
       - If no errors (`*in50 = *off`) and in maintenance mode (`p$mode = 'MNT'`), updates the database via `upddbf`.
       - Calls `f01nxt` to determine the next format or exit.

6. **Determine Next Format (`f01nxt` Subroutine)**:
   - If `*in19` is `*off`, sets `fmtagn` to `*off` to exit the main loop (no additional formats like `fmt02` are used).

7. **Edit Format Input (`f01edt` Subroutine)**:
   - Validates input fields in `fmt01`:
     - Checks if `smsmnm` (salesman name) is blank, setting `m@id` to `ERR0012`, `*in50`, and `*in51` if true.
     - Checks if `smemal` (email address) is blank, setting `m@id` to `ERR0012`, `*in50`, and `*in52` if true.
   - In inquiry mode (`p$mode = 'INQ'`), clears error indicators and messages to prevent validation errors.

8. **Initialize Format Fields (`f01mov` Subroutine)**:
   - Calls `f01edt` to validate fields and clears error indicators and messages if validation fails.

9. **Format Protection Schemes (`f01pro` Subroutine)**:
   - Sets protection indicators:
     - Clears `*in70`–`*in74` by default.
     - In inquiry mode (`p$mode ≠ 'MNT'`), sets `*in70`–`*in73` to `*on` to protect input fields.
     - In maintenance mode, if the record exists (`w$exists = *on`), sets `*in71` to protect key fields (`f$co`, `f$smid`).

10. **Update Database (`upddbf` Subroutine)**:
    - Saves current field values to `svds`.
    - Chains to `bbslsm` using `klslsm`:
      - If the record exists (`*in80 = *off`):
        - If fields have changed (`svds ≠ wkds01`), restores `svds` to `wkds01` and updates the record (`bbslsmpf`).
        - Sets `p$flag` to `1` to indicate success.
        - If no changes, forces end-of-data (`feod`) to reset the file pointer.
      - If the record does not exist:
        - Clears the record buffer, sets `smco` (company), `smsmid` (salesman ID), and `smdel` to `A` (active), and writes a new record to `bbslsmpf`.
        - Sets `p$flag` to `1`.
    
11. **Field Prompting (`prompt` Subroutine)**:
    - Determines cursor location (`row`, `col`) from `csrloc` and sets `*in19` to indicate a format change.

12. **Message Handling**:
    - **addmsg**: Sends error messages to the program message queue using `QMHSNDPM`, setting `dspmsg` to `*on`.
    - **wrtmsg**: Writes the message subfile control (`msgctl`) with `*in49` enabled.
    - **clrmsg**: Clears the message subfile using `QMHRMVPM`, preserving the current record format and `pagrrn`.

13. **Program Exit**:
    - Closes all files, sets `*inlr` to `*on`, and returns to the calling program (`BB928P`).

### Business Rules

The program enforces the following business rules:
1. **Input Validation**:
   - Salesman name (`smsmnm`) and email address (`smemal`) are mandatory fields. If either is blank, an `ERR0012` message is displayed, and error indicators (`*in50`, `*in51`, `*in52`) are set.
   - In inquiry mode, validation errors are cleared to allow read-only access.
2. **Mode-Based Access**:
   - **Maintenance Mode (`MNT`)**:
     - Allows creation or update of salesman ID records.
     - Protects key fields (`f$co`, `f$smid`) if the record exists (`w$exists = *on`) to prevent changes to the company or salesman ID.
     - Updates or creates records in `bbslsm` and sets `p$flag` to `1` on success.
   - **Inquiry Mode (`INQ`)**:
     - Protects all input fields (`*in70`–`*in73`) to prevent modifications.
     - Displays existing records without allowing changes.
3. **Record Existence**:
   - Checks if the salesman ID record exists in `bbslsm` before displaying or updating.
   - Creates a new record with `smdel = 'A'` (active) if it does not exist.
4. **Company Validation**:
   - Validates the company number (`f$co`) against `bicont` during file open.
5. **Message Handling**:
   - Displays error messages for invalid input and clears them after display or in inquiry mode.
6. **Database Integrity**:
   - Uses file overrides to select the correct database files based on `p$fgrp` (`G` or `Z`).
   - Ensures only changed records are updated, and new records are written with appropriate default values.

### Database Tables Used

The program interacts with the following database files:
1. **bbslsm**: Primary file for salesman ID records, opened in update/add mode (`uf a`) with `usropn`. Used for reading, updating, and creating records.
2. **bicont**: Input file for company records, opened with `usropn`. Used to validate the company number (`f$co`).

### External Programs Called

The program calls the following external programs:
1. **QCMDEXC**: Executes file override commands for `bbslsm` and `bicont`.
2. **QMHSNDPM**: Sends messages to the program message queue for error display.
3. **QMHRMVPM**: Removes messages from the program message queue.

### Additional Notes
- **Display File**: `bb928d` is a workstation file using the `PROFOUNDUI(HANDLER)` for the user interface, with a single active format (`fmt01`). References to `fmt02` are commented out, suggesting it was planned but not implemented.
- **Indicators**: Uses indicators (19, 21–79, 90–99) for screen control, error handling, and field protection, consistent with `BB928P`.
- **Field Prefixes**: Follows the same naming conventions as `BB928P` (e.g., `f$` for display fields, `p$` for input parameters, `o$` for output parameters).
- **Integration with BB928P**: `BB928` is called by `BB928P` for options 1 (create), 2 (change), and 5 (display) to handle individual record operations, returning a flag (`p$flag`) to indicate success.

This program is a focused module for maintaining or viewing salesman ID records, complementing the subfile-based interface of `BB928P` by providing detailed record manipulation functionality.