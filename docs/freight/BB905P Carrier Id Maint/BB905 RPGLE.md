The RPGLE program `BB905`, titled "Carrier Code Id Entry Maintenance/Inquiry," is a component of a billing and invoicing system designed to handle the maintenance and inquiry of carrier ID records. It is called by the main program `BB905P` (as described in the previous query) to perform specific operations such as creating, updating, or displaying carrier ID details. Below, I’ll explain the process steps, business rules, database tables used, and external programs called by `BB905`.

### Process Steps of the RPGLE Program (`BB905`)

The program manages a display file interface for maintaining or inquiring about carrier ID records, operating in either maintenance (`MNT`) or inquiry (`INQ`) mode. It processes a single panel format (`FMT01`) and interacts with database files to validate and update carrier data. Here’s a detailed breakdown of the process steps:

1. **Program Initialization (`*inzsr` Subroutine)**:
   - **Receives Input Parameters**: The program accepts five parameters:
     - `p$co`: Company number.
     - `p$caid`: Carrier ID.
     - `p$mode`: Run mode (`MNT` for maintenance, `INQ` for inquiry).
     - `p$fgrp`: File group (`Z` or `G` for database file overrides).
     - `p$flag`: Return flag to indicate the outcome of processing.
   - **Initializes Fields**: Moves input parameters to display file fields (`f$co`, `f$caid`), sets up output parameters (`o$co`, `o$caid`, `o$fgrp`, `o$mode`, `o$flag`), and initializes work fields, message handling fields (`dspmsg`, `m@pgmq`, `m@key`), and key lists (`klcaid` for database access).
   - **Sets Defaults**: Initializes flags like `fmtagn` (to control format processing) and work fields (`w$wk54`, `w$wk92`).

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Applies File Overrides**: Based on the `p$fgrp` parameter (`Z` or `G`), executes `OVRDBF` commands to override the database files (`bbcaid`, `bicont`, `bbcaid1`, `gstabl`) to the appropriate library (e.g., `gbbcaid` or `zbbcaid`).
   - **Opens Files**: Opens the database files `bbcaid` (update/add), `bicont` (input), `bbcaid1` (input with renamed format), and `gstabl` (update/add).
   - **Validates Company Number**: Chains to `bicont` using `f$co` to ensure the company number is valid.

3. **Retrieve Data for Passed Parameters (`rt Lillrtvdta` Subroutine)**:
   - **Chains to `bbcaid`**: Uses the key list `klcaid` (`f$co`, `f$caid`) to retrieve the carrier ID record.
   - **Handles Record Existence**: If the record exists (`*in99 = *off`), sets `w$exists = *on`; otherwise, clears the record and sets `w$exists = *off`.
   - **Sets Display Headers and Protection**: Sets the display header (`c$hdr1`) based on the mode (`MNT` for maintenance, `INQ` for inquiry) and sets `*in70` to protect input fields in inquiry mode.

4. **Process Panel Formats (`srfmt` Subroutine)**:
   - **Clears Screen**: Writes the `clrscr` format to clear the display.
   - **Initializes Format**: Calls `f01mov` to initialize fields for `FMT01` and sets `w$fmt = 'FMT01'`.
   - **Main Processing Loop**:
     - **Displays Message Subfile**: If `dspmsg = *on`, calls `wrtmsg` to display messages; otherwise, clears the screen.
     - **Handles Format Change**: Resets `*in19` (format change indicator).
     - **Displays Format**: Processes `FMT01` (the only active format) by calling `f01pro` and executing `EXFMT fmt01`.
     - **Clears Indicators and Messages**: Resets error indicators (`*in50` to `*in69`) and clears the message subfile if needed.
     - **Processes User Input**: Calls `f01sr` to handle user input for the current format.
     - **Loop Control**: Continues until `fmtagn = *off`.

5. **Process Format Input (`f01sr` Subroutine)**:
   - **Handles Function Keys**:
     - **F04 (Field Prompting)**: Calls `prompt` to handle field prompting.
     - **F10 (Position Cursor Home)**: Clears cursor coordinates (`row`, `col`).
     - **F12 (Exit)**: Sets `fmtagn = *off` to exit the program.
   - **Inquiry Mode**: If `p$mode = 'INQ'`, calls `f01nxt` to determine the next format (though only `FMT01` is active).
   - **Enter Key**:
     - Validates input by calling `f01edt`.
     - If `p$mode = 'MNT'` and no errors (`*in50 = *off`), captures replacement carrier ID information by calling `rtvrplcar`.
     - Updates the database if no errors (`upddbf`).
     - Calls `f01nxt` if no input changes occur.

6. **Edit Format Input (`f01edt` Subroutine)**:
   - **Validates Carrier Description**: Ensures `cicanm` is not blank; if blank, sets error `ERR0012` and indicators `*in50` and `*in51`.
   - **Validates Fuel Facs ID (Commented Out)**: Temporarily disabled (per revision `JB02`) to allow multiple carrier IDs with the same Fuel Facs ID (`ciffid`). If enabled, it checks if `ciffid` exists in `bbcaid1` and ensures it’s unique unless it matches the current `p$caid`.
   - **Inquiry Mode**: Clears error indicators and messages in inquiry mode.

7. **Initialize Format Field Values (`f01mov` Subroutine)**:
   - Calls `f01edt` to validate fields and clears error indicators/messages if validation fails.

8. **Format Protection Schemes (`f01pro` Subroutine)**:
   - **Sets Protection Indicators**: Resets indicators `*in70` to `*in74` to `*off`.
   - **Inquiry Mode**: Sets `*in70` to `*in73` to `*on` to protect fields in inquiry mode.
   - **Existing Record**: If `w$exists = *on`, protects key fields (`*in71 = *on`).

9. **Update Database Files (`upddbf` Subroutine)**:
   - **Saves Current Data**: Stores the current record in `svds`.
   - **Chains to `bbcaid`**: Uses `klcaid` to check if the record exists.
   - **Updates Existing Record**:
     - If the record exists and data has changed (`svds ≠ wkds01`), updates `bbcaidpf`.
     - If company number is 10 (`p$co = 10`), calls `updtables` to update `gstabl`.
     - Sets `p$flag = '1'` to indicate success.
   - **Creates New Record**:
     - Clears `bbcaidpf`, populates fields (`cico`, `cicaid`), and sets creation date (`cidate`) to the current date (per revision `JK01`).
     - Writes to `bbcaidpf` and updates `gstabl` if `p$co = 10`.
     - Sets `p$flag = '1'`.
   - **Forces End of Data**: Uses `FEOD` to ensure proper file handling for updates.

10. **Capture Replacement Carrier ID (`rtvrplcar` Subroutine)**:
    - **Checks for New Entries**: Ensures the record does not exist (`*in99 = *off`).
    - **Calls `BB9059`**: Passes `f$co`, `f$caid`, `o$flag`, and `p$fgrp` to handle replacement carrier ID logic.
    - **Handles Cancellation**: If `o$flag = '2'`, displays a "New Carrier Not Created" message and sets error indicator `*in50`.

11. **Update `gstabl` Table (`updtables` Subroutine)**:
    - **For Company 10**: If `p$co = 10`, updates or adds a record to `gstabl`:
      - Chains to `gstabl` using `type` (`BBCAID`) and `cicaid`.
      - If found, updates `tbdel`, `tbdesc`, and `tbein`.
      - If not found, creates a new record with `tbdel`, `tbtype`, `tbcode`, `tbdesc`, and `tbein`.

12. **Field Prompting (`prompt` Subroutine)**:
    - Determines cursor location (`row`, `col`) and sets `*in19` to indicate a format change.

13. **Message Handling**:
    - **Add Message (`addmsg`)**: Sends messages to the program message queue using `QMHSNDPM`, sets `dspmsg = *on`.
    - **Write Message Subfile (`wrtmsg`)**: Displays the message subfile (`msgctl`) with `*in49`.
    - **Clear Message Subfile (`clrmsg`)**: Clears messages using `QMHRMVPM`, preserving the current record format and page number.

14. **Program Termination**:
    - Closes all files and sets `*inlr = *on` to end the program.

### Business Rules

The program enforces the following business rules:

1. **Input Validation**:
   - **Carrier Description**: Must not be blank (`ERR0012`).
   - **Fuel Facs ID**: Temporarily allows multiple carrier IDs to share the same Fuel Facs ID (revision `JB02`). If validation is enabled, the Fuel Facs ID must be unique unless it matches the current carrier ID (`ERR0018`).
   - **Company Number**: Must exist in the `bicont` file (validated in `opntbl`).

2. **Mode-Based Access**:
   - **Maintenance Mode (`MNT`)**: Allows creating and updating carrier ID records. Key fields are protected for existing records (`*in71`).
   - **Inquiry Mode (`INQ`)**: Protects all input fields (`*in70` to `*in73`), allowing only viewing of records.

3. **Database Updates**:
   - Updates or creates records in `bbcaid` based on whether the record exists.
   - For company number 10, synchronizes data with the `gstabl` table, ensuring consistency in deletion status, description, and EIN fields.
   - Sets a creation date (`cidate`) for new records (revision `JK01`).

4. **Replacement Carrier ID**:
   - For new carrier IDs, checks for replacement logic via `BB9059` to populate `xBBFX62W` with From/To Carrier IDs (revision `DC01`).
   - Cancels creation if the user opts not to proceed (`o$flag = '2'`).

5. **Error Handling**:
   - Displays error messages for invalid inputs or existing/non-existent records.
   - Uses a message subfile to communicate errors and confirmations to the user.

6. **Inactivation/Reactivation**:
   - Referred to as "Inactivate/Reactivate" instead of "Delete/Reactivate" (revision `DC01`).

### Database Tables Used

The program interacts with the following database files:
1. **bbcaid**:
   - Update/add file for carrier ID records.
   - Key fields: `cico` (company), `cicaid` (carrier ID).
   - Fields include `cicanm` (description), `ciffid` (Fuel Facs ID), `ciein` (EIN), `cidel` (deletion status), `cidate` (creation date).
2. **bicont**:
   - Input-only file to validate company numbers (`f$co`).
3. **bbcaid1**:
   - Input-only file with renamed record format (`bbcaidpf` to `bbcaidlf`, prefixed with `l_`) for Fuel Facs ID validation.
4. **gstabl**:
   - Update/add file for company 10, storing carrier ID data.
   - Key fields: `tbtype` (`BBCAID`), `tbcode` (carrier ID).
   - Fields: `tbdel` (deletion status), `tbdesc` (description), `tbein` (EIN).

### External Programs Called

The program calls the following external programs:
1. **BB9059**:
   - Handles replacement carrier ID logic in the `rtvrplcar` subroutine.
   - Parameters: `o$co` (company), `o$caid` (carrier ID), `o$flag` (return flag), `o$fgrp` (file group).
2. **QCMDEXC**:
   - Executes file override commands (`OVRDBF`) in the `opntbl` subroutine.
   - Parameters: `dbov##` (override command string), `dbol##` (length of command).
3. **QMHSNDPM**:
   - Sends messages to the program message queue in the `addmsg` subroutine.
   - Parameters: `m@id`, `m@msgf`, `m@data`, `m@l`, `m@type`, `m@pgmq`, `m@scnt`, `m@key`, `m@errc`.
4. **QMHRMVPM**:
   - Clears messages from the message subfile in the `clrmsg` subroutine.
   - Parameters: `m@pgmq`, `m@scnt`, `m@rmvk`, `m@rmv`, `m@errc`.

### Additional Notes
- **Single Format Processing**: Only `FMT01` is actively processed, with `FMT02` commented out, indicating a simplified interface compared to `BB905P`.
- **Function Keys**:
  - **F04**: Field prompting.
  - **F10**: Positions cursor to the home position.
  - **F12**: Exits the program.
- **Indicators**: Similar to `BB905P`, indicators control screen errors (`*in50` to `*in69`, `*in21` to `*in39`), input protection (`*in70` to `*in79`), and subfile operations.
- **Revisions**:
  - **DC01 (08/2013)**: Updated terminology to "Inactivate/Reactivate" and added logic for populating `xBBFX62W` for carrier replacement.
  - **JB02 (08/05/13)**: Temporarily disabled unique Fuel Facs ID validation.
  - **JK01 (05/04/16)**: Added creation date (`cidate`) for new `bbcaid` records.

This program complements `BB905P` by providing detailed maintenance and inquiry functions for individual carrier ID records, with a focus on validation, database updates, and user interaction via a display file. Let me know if you need further details or clarification!