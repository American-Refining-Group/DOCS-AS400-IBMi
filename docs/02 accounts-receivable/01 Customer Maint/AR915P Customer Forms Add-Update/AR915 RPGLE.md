The RPGLE program `AR915` is a customer form type contacts maintenance and inquiry program within the Customer Master Information system. It is called from the main program `AR915P` (as seen in the previous document) to handle the creation, updating, or displaying of individual customer form type contact records. The program supports both maintenance (`MNT`) and inquiry (`INQ`) modes, providing a user interface to manage contact details such as form type, contact name, email, and various flags. Below is a detailed explanation of the process steps, business rules, database tables used, and external programs called.

---

### Process Steps of the AR915 Program

The `AR915` program follows a structured flow to manage a single customer form type contact record through a display file interface. The process steps are organized by the main subroutines:

1. **Program Initialization (`*inzsr`)**:
   - **Purpose**: Initializes variables, defines key lists, and processes input parameters.
   - **Actions**:
     - Defines the parameter list for receiving input parameters: `p$cono` (company), `p$seq#` (sequence number), `p$cust` (customer), `p$mode` (run mode: 'MNT' or 'INQ'), `p$fgrp` (file group: 'Z' or 'G'), and `p$flag` (return flag).
     - Moves input parameters to display file fields (`f$cono`, `f$seq#`) and initializes output parameters (`o$fgrp`, `o$mode`, `o$flag`).
     - Defines key lists (`klcufm`, `klcust`, `klfrmtyp`) for database operations.
     - Initializes work fields, message handling fields, and date validation parameters.
     - Sets up the display file fields and headers based on the mode (`MNT` or `INQ`).

2. **Open Database Tables (`opntbl`)**:
   - **Purpose**: Opens the required database files with appropriate overrides based on the file group (`p$fgrp`).
   - **Actions**:
     - Checks if `p$fgrp` is 'G' or 'Z' to apply the correct file overrides (`ovg` or `ovz`) using the `QCMDEXC` command.
     - Opens files `arcust`, `gstabl`, `arcufm`, and `bicont` with user-controlled open (`usropn`).

3. **Retrieve Data for Passed Parameters (`rtvdta`)**:
   - **Purpose**: Retrieves data for the provided company, sequence number, and customer to populate the display file fields.
   - **Actions**:
     - Chains to `arcufm` using `klcufm` (company, sequence number) to retrieve the contact record; if not found (`*in99`), clears the record and sets `fmcust` to `p$cust`.
     - Sets `w$exists` to indicate whether the record exists.
     - Chains to `bicont` to validate the company code (`f$cono`); clears `bcname` if not found.
     - Chains to `arcust` to validate the customer code (`fmcust`); clears `arname` if not found.
     - Sets the display header (`c$hdr1`) and protection indicator (`*in70`) based on `p$mode` ('MNT' for maintenance, 'INQ' for inquiry).

4. **Process Panel Formats (`srfmt`)**:
   - **Purpose**: Manages the main loop for displaying and processing the panel format (`fmt01`).
   - **Actions**:
     - Clears the screen (`clrscr`).
     - Initializes the panel format (`f01mov`) and sets the format name to `FMT01`.
     - Enters a loop (`fmtagn`) that:
       - Displays the message subfile if needed (`wrtmsg`) or clears the screen.
       - Displays the `fmt01` format using `exfmt` and clears error indicators (`*in50`-`*in69`).
       - Processes user input via `f01sr`.
       - Clears the message subfile (`clrmsg`) if displayed.
     - Continues until `fmtagn` is turned off (e.g., via F12 or inquiry mode completion).

5. **Process Format (`f01sr`)**:
   - **Purpose**: Handles user input for the `fmt01` format based on function keys or ENTER.
   - **Actions**:
     - Processes function keys:
       - **F04**: Calls the `prompt` subroutine for field prompting.
       - **F10**: Resets the cursor position (`row`, `col`) to home.
       - **F12**: Exits the program by setting `fmtagn` to off.
     - In inquiry mode (`p$mode = 'INQ'`), determines the next format (`f01nxt`) and exits.
     - For ENTER, validates input (`f01edt`) and, if no errors (`*in50 = *off`), updates the database (`upddbf`) in maintenance mode and determines the next format (`f01nxt`).

6. **Determine Next Format (`f01nxt`)**:
   - **Purpose**: Decides whether to continue or exit the panel loop.
   - **Actions**:
     - If no input change occurred (`*in19 = *off`), sets `fmtagn` to off to exit the loop.
     - Note: The subroutine is prepared to handle a second format (`FMT02`), but it is commented out, so the program only uses `FMT01`.

7. **Edit Format Input (`f01edt`)**:
   - **Purpose**: Validates user input fields in maintenance mode.
   - **Actions**:
     - Validates form type code (`fmfmty`) by chaining to `gstabl` using `klfrmtyp`; if valid and not deleted, sets `f$fmty` to the description; else, adds error message `ERR0010` and sets `*in50`, `*in51`.
     - Checks if contact name (`fmcntc`) is non-blank; if blank, adds error message `ERR0012` and sets `*in50`, `*in52`.
     - Validates send original flag (`fmfmyn`), reprint flag (`fmrpyn`), mail flag (`fmmlyn`), and back terms flag (`fmbkyn`); each must be 'Y' or 'N', else adds error message `ERR0014` and sets `*in50`, `*in55`-`*in58` as appropriate.
     - Validates email address (`fmemla`):
       - As of the 08/30/22 revision (JK01), fax number validation is removed.
       - If `fmemla` is non-blank, calls `VALMAILID` to validate the email; if invalid (`p$valid = 'N'`), adds error message `ERR0000` with `com(02)` ("Invalid Email Address Entered") and sets `*in50`, `*in53`.
       - Allows blank email if `fmfmyn = 'N'` (no original sent).
     - In inquiry mode (`p$mode = 'INQ'`), clears error indicators and messages.

8. **Initialize Format Field Values (`f01mov`)**:
   - **Purpose**: Initializes the `fmt01` format fields and clears any prior errors.
   - **Actions**:
     - Calls `f01edt` to validate fields.
     - If errors exist (`*in50`), clears error indicators and messages (`clrmsg`).

9. **Format Protection Schemes (`f01pro`)**:
   - **Purpose**: Sets field protection indicators based on the mode.
   - **Actions**:
     - Clears protection indicators (`*in70`-`*in74`).
     - In inquiry mode (`p$mode != 'MNT'`), sets `*in70`-`*in73` to protect fields.
     - Note: Protection for existing records (`w$exists`) is commented out, so key fields are not protected based on record existence.

10. **Update Database Files (`upddbf`)**:
    - **Purpose**: Updates or creates a record in `arcufm` in maintenance mode.
    - **Actions**:
      - If `f$seq#` is zero, retrieves the next sequence number (`rtvnxtseq`).
      - Saves the current `arcufm` record fields to `svds`.
      - Chains to `arcufm` using `klcufm`:
        - If found (`*in80 = *off`), updates the record if changes exist (`svds != wkds01`) and sets `p$flag = '1'`.
        - If not found, creates a new record with `f$cono` and `f$seq#`, writes to `arcufm`, and sets `p$flag = '1'`.
      - Sets `w$exists` to indicate the record now exists.

11. **Retrieve Next Sequence Number (`rtvnxtseq`)**:
    - **Purpose**: Generates the next sequence number for a new `arcufm` record.
    - **Actions**:
      - Chains to `bicont` to get the current sequence number (`bcseqn`).
      - Increments `f$seq#` until a unique value is found by checking `arcufm` with `klcufm`.
      - Increments `bcseqn` in `bicont` and updates the record.

12. **Field Prompting (`prompt`)**:
    - **Purpose**: Provides lookup functionality for the form type field.
    - **Actions**:
      - If the cursor is on `FMFMTY` and input is not protected (`*in70`), calls `LGSTABL` to prompt for a form type code.
      - Updates `fmfmty` with the selected value if non-blank.
      - Sets `*in19` to indicate a panel format change.

13. **Message Handling (`addmsg`, `wrtmsg`, `clrmsg`)**:
    - **Purpose**: Manages error and confirmation messages displayed in the message subfile.
    - **Actions**:
      - `addmsg`: Sends messages to the program message queue using `QMHSNDPM` with message ID, file, data, and type.
      - `wrtmsg`: Writes the message subfile control (`msgctl`) with `*in49` on.
      - `clrmsg`: Clears the message subfile using `QMHRMVPM` and restores the current record format and page number.

14. **Program Termination**:
    - **Purpose**: Closes files and exits.
    - **Actions**:
      - Closes all open files (`close *all`).
      - Sets `*inlr` to `*on` and returns control to the calling program (`AR915P`).

---

### Business Rules

The program enforces the following business rules during input validation and processing:
1. **Form Type Code (`fmfmty`)**:
   - Must exist in the `gstabl` file with type `FRMTYP` and not be marked deleted (`tbdel != 'D'`).
   - If invalid, displays error `ERR0010` and sets error indicators.

2. **Contact Name (`fmcntc`)**:
   - Must be non-blank.
   - If blank, displays error `ERR0012` and sets error indicators.

3. **Email Address (`fmemla`)**:
   - As of the 08/30/22 revision, fax number validation is removed, and email is the primary contact method.
   - If non-blank, must be valid as determined by the `VALMAILID` program (`p$valid = 'Y'`).
   - If invalid, displays error `ERR0000` with message "Invalid Email Address Entered".
   - Can be blank if `fmfmyn = 'N'` (no original sent); otherwise, a non-blank email is required if `fmfmyn` is 'Y' or blank.

4. **Flags (`fmfmyn`, `fmrpyn`, `fmmlyn`, `fmbkyn`)**:
   - Each flag (send original, reprint, mail, back terms) must be 'Y' or 'N'.
   - If invalid, displays error `ERR0014` and sets corresponding error indicators (`*in55`-`*in58`).

5. **Mode-Based Behavior**:
   - In maintenance mode (`p$mode = 'MNT'`), fields are editable, and updates are written to `arcufm`.
   - In inquiry mode (`p$mode = 'INQ'`), fields are protected (`*in70`-`*in73`), errors are cleared, and no database updates occur.
   - The display header changes based on the mode: "Customer Form Type Contacts Maintenance" for `MNT`, "Customer Form Type Contacts Inquiry" for `INQ`.

6. **Sequence Number Generation**:
   - For new records (`f$seq# = 0`), the next sequence number is retrieved from `bicont` (`bcseqn`) and incremented until a unique value is found in `arcufm`.
   - The `bicont` record is updated with the new sequence number.

7. **Database Updates**:
   - In maintenance mode, updates or creates records in `arcufm` only if input validation passes.
   - Sets `p$flag = '1'` to indicate a successful update or creation.

8. **Field Prompting**:
   - The form type code (`fmfmty`) can be prompted via F04, calling `LGSTABL` to select a valid value.

---

### Database Tables Used

The program uses the following database files, all opened with `usropn`:
1. **arcust**:
   - Purpose: Customer master file for validating customer codes.
   - Usage: Chained to using `klcust` (company, customer) to retrieve `arname` (customer name).
   - Access: Input only, keyed (`k disk`).
   - Override: `garcust` (for 'G' file group) or `zarcust` (for 'Z' file group).

2. **gstabl**:
   - Purpose: General table file for validating form type codes.
   - Usage: Chained to using `klfrmtyp` (type `FRMTYP`, code `fmfmty`) to retrieve `tbdesc` (description).
   - Access: Input only, keyed (`k disk`).
   - Override: `ggstabl` (for 'G') or `zgstabl` (for 'Z').

3. **arcufm**:
   - Purpose: Primary file for customer form type contact records.
   - Usage: Chained to for retrieving records (`klcufm`), updated, or written to in maintenance mode.
   - Access: Update and add, keyed (`uf a e k disk`).
   - Override: `garcufm` (for 'G') or `zarcufm` (for 'Z').

4. **bicont**:
   - Purpose: Company master file for validating company codes and managing sequence numbers.
   - Usage: Chained to for validating `f$cono` and retrieving `bcname`; updated to increment `bcseqn` for new records.
   - Access: Update, keyed (`uf e k disk`).
   - Override: `gbicont` (for 'G') or `zbicont` (for 'Z').

5. **ar915d**:
   - Purpose: Display file for the user interface.
   - Usage: Contains the `fmt01` format and message subfile control (`msgctl`) for interactive display and input.
   - Access: Work station file (`cf e workstn`).

---

### External Programs Called

The program interacts with the following external programs:
1. **LGSTABL**:
   - Called in subroutine `prompt`.
   - Parameters: `k$ft` (table type, set to `FRMTYP`), `k$fmty` (form type code), `o$fgrp` (file group).
   - Purpose: Provides a lookup window for selecting a valid form type code.

2. **VALMAILID**:
   - Called in subroutine `f01edt` (added in 08/30/22 revision).
   - Parameters: `p$email` (email address, 100 characters), `p$valid` (validity flag, 'Y' or 'N').
   - Purpose: Validates the email address entered in `fmemla`.

3. **QCMDEXC**:
   - Called in subroutine `opntbl`.
   - Parameters: `dbov##` (override command), `dbol##` (command length).
   - Purpose: Executes file override commands for `arcust`, `gstabl`, `arcufm`, and `bicont`.

4. **QMHSNDPM**:
   - Called in subroutine `addmsg`.
   - Parameters: `m@id` (message ID), `m@msgf` (message file), `m@data` (message data), `m@l` (message length), `m@type` (message type), `m@pgmq` (program message queue), `m@scnt` (stack counter), `m@key` (message key), `m@errc` (error code).
   - Purpose: Sends messages to the program message queue.

5. **QMHRMVPM**:
   - Called in subroutine `clrmsg`.
   - Parameters: `m@pgmq` (program message queue), `m@scnt` (stack counter), `m@rmvk` (message key), `m@rmv` (remove option), `m@errc` (error code).
   - Purpose: Removes messages from the program message queue.

---

### Summary

The `AR915` program, called by `AR915P`, provides a detailed interface for maintaining or inquiring about customer form type contact records. It supports creating new records, updating existing ones, and displaying details in a single-panel format (`fmt01`). The program enforces strict validation rules for form type, contact name, email, and flags, with email validation introduced in the 08/30/22 revision. It uses four database files (`arcust`, `gstabl`, `arcufm`, `bicont`) with dynamic overrides and interacts with external programs for field prompting and email validation. The program ensures data integrity through comprehensive input validation and provides user feedback via a message subfile.