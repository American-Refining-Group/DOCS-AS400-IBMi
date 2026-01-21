The RPG program `BB945.rpgle` is a customer freight table maintenance and inquiry program, designed to manage individual freight entries within the customer freight system. It is called by the main program `BB945P.rpgle` to handle operations such as adding, updating, copying, or inquiring about specific freight records. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of `BB945.rpgle`

The program operates through a series of subroutines that manage the display and processing of freight entry details on a single panel format (`FMT01`). The steps are:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Receives Input Parameters**:
     - `p$cono`: Company number.
     - `p$sqky`: Sequence key for the freight record to process.
     - `p$frsq`: Sequence key for copying a record (if applicable).
     - `p$mode`: Run mode ('MNT' for maintenance, 'INQ' for inquiry).
     - `p$fgrp`: File group ('Z' or 'G' for library selection).
     - `p$flag`: Return flag to indicate processing outcome.
   - Sets up global protection mode (`*in70`) based on `p$mode`:
     - `MNT`: Allows editing (`*in70 = *off`).
     - `INQ`: Read-only mode (`*in70 = *on`).
   - Initializes message handling fields, work fields, and key lists for file access.
   - Sets the current date and time (`ps#dat`, `ps#cen`, `ps#ymd`) for validation and processing.
   - Defines key lists for accessing files (e.g., `klcust`, `klship`, `klloc`, `klcnty`, `klcaid`, `klcacd`, `klprod`, `klrtcd`, `klcfsh`).

2. **Open Database Tables (`opntbl` Subroutine)**:
   - Applies file overrides based on `p$fgrp` ('Z' or 'G') to select the appropriate library for files.
   - Overrides `bb945w` to `QTEMP/bb945w` for temporary storage of missing freight GL records.
   - Opens input files (`arcust`, `bicont`, `gscntr1`, `gsprod`, `gstabl`, `inloc`, `shipto`, `trrtcd`, `glmast`, `bbcaid`) and update/output files (`bicuf2`, `bicufrh`, `bb945w`).

3. **Retrieve Data (`rtvdta` Subroutine)**:
   - If `p$sqky` is provided (non-blank), retrieves the freight record from `bicuf2` using the sequence key.
   - If `p$frsq` is provided (copy mode), retrieves the source record to populate fields for copying.
   - If no sequence key is provided (add mode), initializes fields with default values.
   - Sets up display fields (`bfcono`, `bfcust`, `bfship`, etc.) and validates key fields (company, customer, ship-to, etc.).
   - Checks for missing or invalid freight GL accounts and writes to `bb945w` if necessary (`chkfrt` subroutine).
   - Sets protection indicators (`*in71`, `*in72`, `*in73`, `*in74`) based on mode and record status.

4. **Process Panel Formats (`srfmt` Subroutine)**:
   - Clears the screen and initializes the panel format (`FMT01`).
   - Enters a loop (`fmtagn`) to display and process the panel until exit:
     - Displays the message subfile if errors exist (`wrtmsg`).
     - Clears error indicators (`*in50`–`*in69`).
     - Displays the `FMT01` format using `EXFMT`.
     - Processes user input based on the current format (`f01sr`).
     - Clears the message subfile after processing (`clrmsg`).

5. **Process FMT01 Input (`f01sr` Subroutine)**:
   - Handles function key inputs and Enter key:
     - **F04**: Calls the `prompt` subroutine for field prompting.
     - **F06**: Toggles display mode (`w$showmode`) between 'A' (all values), 'B' (billed values), or 'C' (carrier values), updating the function key label (`f01f06`).
     - **F09**: Calls `BB912` (for non-railcar carriers) or `BB911` (for railcar carriers, `bfcacd = 'RC'`) for fuel surcharge maintenance/inquiry.
     - **F10**: Positions the cursor to the home position.
     - **F12**: Exits the program, setting `p$flag = '0'`.
     - **Enter**: Validates input fields (`f01ed1`, `f01ed2`, `f01ed3`), processes changes (`f01prc`), and calculates freight (`calcfrt`).
   - Updates the display format if input changes are detected (`*in19`).

6. **Edit FMT01 Input (`f01ed1`, `f01ed2`, `f01ed3` Subroutines)**:
   - **f01ed1**: Validates key fields:
     - Company (`bfcono`): Chains to `bicont` for name.
     - Customer (`bfcust`): Chains to `arcust` for name.
     - Ship-to (`bfship`): Chains to `shipto`, requires valid customer.
     - Location (`bfloc`): Chains to `inloc`.
     - Container Code/Type (`bfcncd`): Validates as container type (1 char) or code (>1 char) using `gstabl` or `gscntr1`.
     - Carrier ID (`bfcaid`): Chains to `bbcaid`, checks for active status.
     - Carrier Code (`bfcacd`): Chains to `gstabl` for description.
     - Carrier Codes 2/3 (`bfcac2`, `bfcac3`): Chains to `gstabl`.
     - Sets error indicators (`*in50`–`*in58`) and adds messages for invalid inputs.
   - **f01ed2**: Validates product codes (`bfpr01`–`bfpr10`):
     - Checks for duplicates and ensures at least one valid product code.
     - Validates each code against `gsprod`.
     - Adds error messages for invalid or duplicate codes.
   - **f01ed3**: Validates additional fields:
     - Route Code (`csrout`): Required if `bfcacd = 'RC'`, otherwise must be blank.
     - Rate Fields (`bfrpm`, `bfcwt`, `bfrpg`, `bfflat`, `bfrpum`): Ensures only one rate type is entered.
     - Minimum (`bfmin`): Required for `bfcwt`, `bfrpg`, or `bfrpum`, but not allowed with `bfrpm`.
     - Surcharge Calculation (`bfscca`): Must be 'B', 'C', 'N', or 'M' (not 'M' in some cases).
     - Start/End Dates (`bfstdt`, `bfexdt`): Ensures expiration date is not before start date.
     - Validates against `trrtcd` for route codes and `bbcfsh` for carrier fuel surcharge.

7. **Process FMT01 Changes (`f01prc` Subroutine)**:
   - If no errors (`*in50`–`*in69` off), processes the record:
     - **Add Mode**: Writes a new record to `bicuf2` with a generated sequence key (`bfsqky`).
     - **Update Mode**: Updates the existing record in `bicuf2`.
     - **Copy Mode**: Writes a new record with data from the source record.
   - Writes a history record to `bicufrh` (`writehist`).
   - Sets confirmation messages and the return flag (`p$flag = 'C'` for changed/added, 'D' for deleted).
   - Exits the format loop (`fmtagn = *off`).

8. **Calculate Freight (`calcfrt` Subroutine)**:
   - Populates the `FRGHT` and `FRTTBL` data structures with record values.
   - Sets default values for freight calculation (e.g., quantity, pickup date).
   - Calls `MBBFR1` to compute freight totals (`f$famt` for billed, `f$cfamt` for carrier).
   - Updates display fields (`f01$btot`, `f01$ctot`) with calculated amounts.

9. **Check Freight GL Accounts (`chkfrt` Subroutine)**:
   - Validates product codes against `gsprod` and `gscntr1` to retrieve freight GL accounts.
   - Skips records with `tbcode = '     X'` (no freight charged, GL = 9999).
   - Checks each GL account in `glmast`, flagging deleted or inactive accounts (`GLDEL = 'D'` or `'I'`).
   - Writes missing or invalid GL accounts to `bb945w` (`WrtMissGl`).

10. **Write History (`writehist` Subroutine)**:
    - Writes a history record to `bicufrh` with the current timestamp, user ID, and record data.

11. **Field Prompting (`prompt` Subroutine)**:
    - Calls external programs based on cursor position:
      - `LCSTSHP`: For customer (`bfcust`) and ship-to (`bfship`).
      - `LINLOC`: For location (`bfloc`).
      - `LGSCNCD`: For container code/type (`bfcncd`).
      - `LBBCAID`: For carrier ID (`bfcaid`).
      - `LGSTABL`: For carrier codes (`bfcacd`, `bfcac2`, `bfcac3`).
      - `LGSPROD`: For product codes (`bfpr01`–`bfpr10`).
    - Updates fields with selected values and sets the input change indicator (`*in19`).

12. **Message Handling**:
    - **Add Message (`addmsg`)**: Sends error or confirmation messages to the program message queue.
    - **Write Message (`wrtmsg`)**: Displays messages in the message subfile.
    - **Clear Message (`clrmsg`)**: Clears the message subfile.

13. **Program End**:
    - Closes all files and terminates the program, returning control to `BB945P`.

---

### Business Rules

The program enforces several business rules to ensure data integrity and proper freight entry management:
1. **Validation of Key Fields**:
   - Company, customer, ship-to, location, container code/type, carrier ID, and carrier codes must exist in their respective master files (`bicont`, `arcust`, `shipto`, `inloc`, `gscntr1`, `gstabl`, `bbcaid`).
   - Ship-to requires a valid customer.
   - Carrier ID must be active (not flagged as inactive in `bbcaid`).
   - Route code is mandatory when carrier code is 'RC' (railcar); otherwise, it must be blank.

2. **Product Code Validation**:
   - At least one valid product code (`bfpr01`–`bfpr10`) must be entered.
   - No duplicate product codes are allowed.
   - Product codes must exist in `gsprod`.

3. **Rate Validation**:
   - Only one rate type (`bfrpm`, `bfcwt`, `bfrpg`, `bfflat`, `bfrpum`) can be entered per record.
   - Minimum (`bfmin`) is required for `bfcwt`, `bfrpg`, or `bfrpum`, but not allowed with `bfrpm`.
   - Carrier rates (`cfrpm`, `cfcwt`, etc.) follow the same rules.

4. **Surcharge Rules**:
   - Surcharge calculation field (`bfscca`) must be 'B' (billed), 'C' (carrier), 'N' (none), or 'M' (manual), with 'M' restricted in some cases.
   - Displays "Today's Surcharge %" or "NO Carrier Surcharge" based on `bfscsp` and `bfcacd`.

5. **Date Validation**:
   - Expiration date (`bfexdt`) must not be before start date (`bfstdt`).
   - Dates are validated using the `GSDTEDIT` module.

6. **GL Account Validation**:
   - Freight GL accounts are checked against `glmast` for validity.
   - Deleted or inactive GL accounts (`GLDEL = 'D'` or `'I'`) or accounts not matching specific criteria (e.g., `tbsgln ≠ 13810000` for non-bulk containers) are written to `bb945w`.
   - Container codes with `tbcode = '     X'` are skipped (no freight charged).

7. **Mode-Based Protection**:
   - In inquiry mode (`p$mode = 'INQ'`), all fields are protected (`*in70`).
   - In update mode, key fields are protected (`*in71`) to prevent changes to existing records.
   - Billed and carrier values are protected based on `w$showmode` (`*in72`, `*in73`).
   - Actual freight cost fields are protected if `bfacdf ≠ 'Y'` (`*in74`).

8. **History Tracking**:
   - All add, update, and copy operations are logged to `bicufrh` with user ID and timestamp.

9. **Fuel Surcharge Handling**:
   - For railcar carriers (`bfcacd = 'RC'`), calls `BB911` for fuel surcharge maintenance.
   - For other carriers, calls `BB912`.
   - Railcar fuel surcharge can be calculated based on miles (revision `mg08`).

---

### Tables (Files) Used

The program accesses the following files, with overrides applied based on `p$fgrp` ('Z' or 'G'):
1. **bb945d**: Display file (workstation file for `FMT01` and message subfile).
2. **arcust**: Customer master file (input only).
3. **bicont**: Company master file (input only).
4. **gscntr1**: Container code file (input only, replaces `gscntr`).
5. **gsprod**: Product code file (input only).
6. **gstabl**: Table file for container types, carrier types, etc. (input only).
7. **inloc**: Location master file (input only).
8. **shipto**: Ship-to master file (input only).
9. **trrtcd**: Route code file (input only).
10. **glmast**: General ledger master file (input only).
11. **bbcaid**: Carrier ID file (input only).
12. **bicuf2**: Freight file (update and add).
13. **bicufrh**: Freight history file (output only).
14. **bb945w**: Work file for missing/invalid freight GL accounts (output only, overridden to `QTEMP`).

---

### External Programs Called

The program interacts with the following external programs:
1. **LCSTSHP**: Prompts for customer and ship-to selection.
2. **LINLOC**: Prompts for location selection.
3. **LGSCNCD**: Prompts for container code/type selection.
4. **LBBCAID**: Prompts for carrier ID selection.
5. **LGSTABL**: Prompts for carrier code selection (`bfcacd`, `bfcac2`, `bfcac3`).
6. **LGSPROD**: Prompts for product code selection.
7. **BB911**: Fuel surcharge maintenance/inquiry for railcar carriers (`bfcacd = 'RC'`).
8. **BB912**: Fuel surcharge maintenance/inquiry for non-railcar carriers.
9. **MBBFR1**: Calculates freight totals (billed and carrier amounts).
10. **QMHSNDPM**: Sends messages to the program message queue.
11. **QMHRMVPM**: Removes messages from the program message queue.
12. **QCMDEXC**: Executes file override commands.
13. **GSDTEDIT**: Validates dates (called via `pldted` parameter list).

---

### Summary

The `BB945.rpgle` program, called by `BB945P.rpgle`, provides a detailed interface for maintaining or inquiring about individual customer freight entries. It supports adding, updating, copying, and viewing freight records, with robust validation of key fields, product codes, rates, and GL accounts. The program enforces business rules to ensure data integrity, logs changes to a history file, and calculates freight totals for display. It integrates with multiple external programs for prompting and calculations and uses a variety of files for data access and storage. Revisions indicate enhancements like improved GL validation, railcar fuel surcharge calculations, and history tracking.