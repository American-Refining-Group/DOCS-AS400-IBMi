The provided `AR930.rpgle.txt` is an RPGLE (RPG IV) program for IBM i (AS/400) that handles Accounts Receivable (A/R) control file maintenance. It is called from the `AR930.ocl36.txt` OCL program, as previously discussed, and is responsible for managing A/R control records, including adding, updating, deleting, or reactivating company records. The program uses a workstation file for user interaction and validates data against customer and general ledger files. Below, I’ll explain the process steps, business rules, tables used, and external programs called, based on the RPGLE source code.

---

### Process Steps of the RPGLE Program (`AR930`)

The `AR930` program is an interactive application that manages A/R control file records through two display formats (`AR930S1` and `AR930S2`) on a workstation screen. It supports adding, updating, deleting, and reactivating A/R company records, with validation against the General Ledger and A/R customer files. The program uses function keys (e.g., F1, F4, F10, F11) to control modes and actions. Here’s a step-by-step breakdown of the process:

1. **Program Initialization**:
   - The program initializes by setting up the environment, clearing message fields (`msg1`, `msg2`), and resetting indicators (e.g., `*IN62` to `*IN90`) used for controlling screen logic and error handling.
   - It checks the `qsctl` variable to determine the initial mode. If `qsctl` is blank, it sets entry mode (`*IN09`, `*IN01`) and initializes `qsctl` to 'R'. Otherwise, it reads the workstation file (`AR930S1` or `AR930S2`) based on indicators `*IN81` or `*IN82`.

2. **Workstation File Read and Screen Handling**:
   - The program reads the workstation file (`AR930D`) for user input, using two display formats:
     - **AR930S1**: Used for company selection (displays company number and messages).
     - **AR930S2**: Used for data entry/update (displays fields like company name, G/L numbers, aging limits, etc.).
   - If an error occurs during reading (e.g., `*IN09` is on), it calls the `rollsr` subroutine to handle roll keys (F11/F12 for scrolling).

3. **Function Key Processing**:
   - The program processes function keys to control its behavior:
     - **KA (F1)**: Bypasses entry/update, clears fields, and displays `AR930S1` in update mode (`*IN81`, `*IN11`).
     - **KD (F4)**: Deletes or reactivates a company record, depending on its status (`*IN21` indicates deleted status).
     - **KG**: Exits the program by setting `*INLR` (last record) and clearing screen indicators.
     - **KJ (F10)**: Switches to entry mode (`*IN82`, `*IN10`), clears fields, and resets indicators.
     - **KK (F11)**: Switches to update mode (`*IN81`, `*IN11`), clears fields, and resets indicators.
   - The program jumps to the `end` tag after processing function keys to avoid further logic execution in the main loop.

4. **Screen Processing**:
   - **Screen 1 (S1, `*IN01`)**:
     - Called via the `s1` subroutine.
     - Validates the company number (`co`) by chaining to the `ARCONT` file.
     - If the company doesn’t exist (`*IN92` on), it displays error messages (“RECORD NOT FOUND”, “PRESS CMD KEY 10 TO ADD”) and sets update mode (`*IN81`, `*IN90`).
     - If found, it moves record data to screen fields using the `move` subroutine and sets update mode (`*IN11`, `*IN82`).
   - **Screen 2 (S2, `*IN02`)**:
     - Called via the `s2` subroutine.
     - Validates input data for adding or updating a company record.
     - Checks include:
       - Company number (`co`) must not be zero in entry mode (`*IN10`).
       - In entry mode, the company must not already exist in `ARCONT`.
       - Company name (`name`) must not be blank.
       - General Ledger numbers (`argl`, `slgl`, `dsgl`, `csgl`, `icgl`, `fcgl`, `efcg`) are validated against `GLMAST` to ensure they exist and are not deleted or inactive.
       - Security code (`secr`) must not be blank.
       - Aging limits (`lmt1`, `lmt2`, `lmt3`, `lmt4`) must be non-zero and in ascending order (e.g., `lmt1` < `lmt2` < `lmt3` < `lmt4`).
     - If validations pass, it writes or updates the `ARCONT` record and clears fields.

5. **Delete/Reactivate Processing**:
   - **Delete (subroutine `delete`)**:
     - Checks if customer records exist in `ARCUST` for the company (`co`).
     - If customer records exist and are not deleted, it prevents deletion and displays an error (“COMPANY MASTER RECORDS EXIST”, “DELETION OF COMPANY NOT ALLOWED”).
     - If no active customer records exist, it marks the `ARCONT` record as deleted (`acdel = 'D'`).
   - **Reactivate (subroutine `reacti`)**:
     - Marks a deleted `ARCONT` record as active (`acdel = 'A'`) and displays a confirmation message (“PREVIOUS RECORD WAS REACTIVATED”).

6. **Scrolling (Roll Forward/Backward)**:
   - **Roll Forward (subroutine `rollfw`)**:
     - Reads the next record from `ARCONT` using the company number (`co`).
     - If a record is found, it updates the company number and sets `*IN01` for `AR930S1`.
     - If end-of-file is reached, it displays “END OF FILE HAS BEEN REACHED”.
   - **Roll Backward (subroutine `rollbw`)**:
     - Reads the previous record from `ARCONT`.
     - If a record is found, it updates the company number and sets `*IN01`.
     - If beginning-of-file is reached, it displays “BEGINNING OF FILE REACHED”.

7. **Write to Screen**:
   - After processing, the program writes to the workstation file (`AR930S1` or `AR930S2`) to display the current state, including any error messages or field values.

8. **Write/Update `ARCONT`**:
   - If validations pass, the program writes a new record (`*IN70`, `*IN92`) or updates an existing record (`*IN70`, not `*IN92`, not `*IN25`) to `ARCONT`.
   - For reactivation or deletion, it updates the `acdel` field (`'A'` or `'D'`) in `ARCONT`.

9. **Loop and Termination**:
   - The program loops back to read the workstation file unless `*INLR` is set (via KG or program end).
   - It clears fields and resets indicators as needed to prepare for the next user interaction.

---

### Business Rules

The program enforces the following business rules for A/R control file maintenance:

1. **Company Number Validation**:
   - The company number (`co`) must not be zero when adding a new record.
   - The company must not already exist in `ARCONT` when adding (`*IN10`).
   - The company must exist in `ARCONT` when updating or deleting (`*IN11`).

2. **Field Validations**:
   - **Company Name**: Cannot be blank (`*IN62` triggers if blank).
   - **General Ledger Accounts**:
     - A/R G/L (`argl`), Sales G/L (`slgl`), Discount G/L (`dsgl`), Cash G/L (`csgl`), Intercompany G/L (`icgl`), Finance Charge G/L (`fcgl`), and EFT Cash G/L (`efcg`) must exist in `GLMAST` and not be marked as deleted (`gldel = 'D'`) or inactive (`gldel = 'I'`).
     - Non-zero G/L numbers are validated; zero is allowed (bypasses validation).
   - **Security Code**: Must not be blank (`*IN68` triggers if blank).
   - **Aging Limits**:
     - `lmt1`, `lmt2`, `lmt3`, `lmt4` must be non-zero.
     - Must be in ascending order: `lmt1` < `lmt2` < `lmt3` < `lmt4`.
     - These limits define aging buckets (e.g., 0-30, 31-60, 61-90, 91-120 days from invoice date, per revision log).

3. **Deletion Rules**:
   - A company cannot be deleted if active customer records exist in `ARCUST` (i.e., `ardel ≠ 'D'`).
   - Deletion sets `acdel = 'D'` in `ARCONT`.

4. **Reactivation**:
   - A deleted company (`acdel = 'D'`) can be reactivated by setting `acdel = 'A'`.

5. **Aging Logic (Revision JB01)**:
   - Aging is based on invoice date (not due date), with buckets: 0-30, 31-60, 61-90, 91-120, and over 120 days.
   - This was a specific change for PNC Bank, affecting how `lmt1` to `lmt4` are used.

6. **Error Handling**:
   - The program displays specific error messages (from the `msg` array) for invalid inputs, such as non-existent G/L accounts, blank fields, or invalid aging limits.
   - Errors set `*IN90` and display the appropriate screen (`*IN81` or `*IN82`) with the error message.

7. **Mode Control**:
   - **Entry Mode (`*IN10`)**: For adding new companies.
   - **Update Mode (`*IN11`)**: For modifying existing companies or deleting/reactivating.
   - Function keys (F10, F11) toggle between modes.

---

### Tables (Files) Used

The program uses the following files, as defined in the File Specification (F-specs):

1. **ARCONT**:
   - Type: Update file (`UF A`), disk, keyed (key starts at position 2).
   - Description: A/R control file, stores company-level A/R settings.
   - Fields: Company number (`acco`), name (`acname`), G/L numbers (`acargl`, `acslgl`, `acdsgl`, `accsgl`, `actrgl`, `acfngl`, `acefcg`), journal numbers (`acarj#`, `acslj#`), finance charge (`acfinc`), security code (`acsecr`), fiscal month (`acffmo`), aging limits (`aclmt1` to `aclmt4`), last close date (`acdtcl`), cash receipts date (`acdtcr`), delete flag (`acdel`).
   - Record Length: 256 bytes.

2. **ARCUST**:
   - Type: Input file (`IF`), disk, keyed (key starts at position 2).
   - Description: A/R customer file, used to check for active customer records during deletion.
   - Fields: Company number (`arco`), delete flag (`ardel`).
   - Record Length: 384 bytes.

3. **GLMAST**:
   - Type: Input file (`IF`), disk, keyed (key starts at position 2).
   - Description: General Ledger master file, used to validate G/L account numbers.
   - Fields: Delete/inactive flag (`gldel`).
   - Record Length: 256 bytes.

4. **AR930D**:
   - Type: Workstation file (`CF E`), handled by `PROFOUNDUI(HANDLER)`.
   - Description: Display file for user interaction, with two formats:
     - `AR930S1`: Company selection screen (company number, messages).
     - `AR930S2`: Data entry/update screen (company details, G/L numbers, aging limits, etc.).
   - Uses an information data structure (`infds`) for status and error handling.

---

### External Programs Called

The `AR930` program does not explicitly call any external programs (no `CALL` operations or external procedure calls are present). It relies entirely on internal subroutines and file operations. The `PROFOUNDUI(HANDLER)` specified in the F-spec for `AR930D` indicates that the workstation file is managed by a Profound UI handler, which is a user interface framework, but this is not an external program call in the traditional sense.

---

### Subroutines Used

The program uses the following subroutines to organize its logic:
1. **s1**: Handles `AR930S1` processing (company selection, validation, and data retrieval).
2. **s2**: Handles `AR930S2` processing (data entry/update, validations).
3. **clear**: Clears screen fields (e.g., `name`, `argl`, `lmt1`, etc.) to prepare for new input.
4. **move**: Moves data from `ARCONT` record to screen fields for display.
5. **rollsr**: Handles roll key errors (e.g., F11/F12 status codes 01122, 01123).
6. **rollfw**: Scrolls forward through `ARCONT` records.
7. **rollbw**: Scrolls backward through `ARCONT` records.
8. **delete**: Deletes a company record after validating no active customers exist.
9. **reacti**: Reactivates a deleted company record.

---

### Additional Notes

- **Conversion Details**: The program was converted using TARGET/400 on 04/08/23, with 246 added lines, 9 modified lines, and 549 total lines processed. The `T4A`, `T4M`, `T4O` prefixes indicate added, modified, and original lines, respectively.
- **Revision History**:
  - **JB01 (04/13/05)**: Changed aging to use invoice date instead of due date, with buckets 0-30, 31-60, 61-90, 91-120, over 120 days.
  - **JB02 (08/24/16)**: Treats inactive G/L accounts (`gldel = 'I'`) the same as deleted accounts (`gldel = 'D'`).
  - **JB03 (09/27/22)**: Notes that positions 113-114 in `ARCONT` are numeric (2.0 format), though not used in output specs.
- **Profound UI**: The use of `PROFOUNDUI(HANDLER)` suggests a modernized user interface, replacing traditional 5250 green-screen displays with a web-based or graphical interface.
- **Message Array**: The program uses a 22-element message array (`msg`) to display specific error or status messages, enhancing user feedback.

---

### Summary

The `AR930` RPGLE program is an interactive A/R control file maintenance application that:
- Manages company records in `ARCONT` (add, update, delete, reactivate).
- Validates inputs against `ARCUST` (for deletion) and `GLMAST` (for G/L accounts).
- Uses two display formats (`AR930S1`, `AR930S2`) for user interaction via Profound UI.
- Enforces strict business rules, such as non-blank fields, valid G/L accounts, and ordered aging limits.
- Supports scrolling and mode switching (entry/update) via function keys.

**Tables Used**: `ARCONT`, `ARCUST`, `GLMAST`, `AR930D` (workstation file).  
**External Programs Called**: None (only internal subroutines and Profound UI handler).  

If you need further clarification on specific subroutines, business rules, or integration with the OCL program, please let me know!