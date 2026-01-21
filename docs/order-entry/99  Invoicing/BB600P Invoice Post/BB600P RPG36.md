The RPG program `BB600P.rpg36.txt` is called by the `BB600P.ocl36.txt` program to prompt the user for the Accounts Receivable (A/R) journal posting date for invoice batches. It ensures that the entered date is valid and complies with business rules, such as falling within acceptable date ranges and aligning with the invoice ship dates in the batch. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the BB600P RPG Program

The `BB600P` program uses a workstation display file (`SCREEN`) to prompt the user for a journal posting date and validates it against system constraints and invoice data in the `BBTRAN` file. It operates through a main loop and several subroutines (`$SBLK`, `$S1`, `$S1ENT`, `$S1CK`, `$S2`, `$S2ENT`, `$S2CK`, `$DTEDT`, `CLRIND`, `CLRFLD`, `CODEUP`, `$XCPT`) to manage screen displays, date validation, and user input.

#### Process Steps:
1. **Main Loop**:
   - The program runs in a loop until the screen format (`@SFNEX`) is set to `'EJ'` (exit job) or the Last Record (`LR`) indicator is set (`DOWNE 'EJ'`).
   - It processes the current screen format (`@SFID`) using a `CASE` structure:
     - If `@SFID = *BLANK`, executes `$SBLK` (initial blank screen).
     - If `@SFID = 'S1'`, executes `$S1` (journal date input screen).
     - If `@SFID = 'S2'`, executes `$S2` (confirmation/override screen).
   - Calls `$XCPT` to display the next screen and reads user input from `SCREEN` (`READ SCREEN`), setting `LR` if the read fails or the program exits.

2. **Initial Blank Screen (`$SBLK` Subroutine)**:
   - Executes only the first time (`@SFID = *BLANK`).
   - Retrieves the system date and time (`TIME`) and formats the current date (`SYDATE`) into `SYDYMD` (MMDDYY format).
   - Calculates the allowable date range for posting:
     - Determines the current month (`MONTH`) and year (`YEAR`) from `SYDATE`.
     - Sets `FRDATE` (from date) to the first day of the previous month and `TODATE` (to date) to the last day of the current month:
       - If `MONTH = 12`, `FRMNTH = 11`, `FRYEAR = YEAR`, `TOMNTH = MONTH`, `TOYEAR = YEAR + 1`.
       - If `MONTH = 01`, `FRMNTH = 12`, `FRYEAR = YEAR - 1`, `TOMNTH = MONTH`, `TOYEAR = YEAR`.
       - Otherwise, `FRMNTH = MONTH - 1`, `FRYEAR = YEAR`, `TOMNTH = MONTH`, `TOYEAR = YEAR`.
     - Formats `FRDATE` and `TODATE` as MMDDYY.
   - Initializes `JRDATE` (journal date), `YN` (confirmation flag), and message fields (`MSGOUT`, `MSGC1`, `MSGC2`) to blank/zero.
   - Sets indicator `53` to enable the first screen and sets `@SFNEX = 'S1'` to display the journal date prompt.
   - Reads the `BBTRAN` file (batch transaction file) for the specified batch (`BATCH#`) to determine the earliest (`LODATE`) and latest (`HIDATE`) ship dates (`BOSHDT`):
     - Skips records with `BODEL = 'D'` or `'E'` (deleted or excluded, per `JB02`).
     - Converts ship dates to MMDDYY (`SHPYMD`) and updates `LODATE` and `HIDATE` if the ship date is outside the current range.
     - Continues reading until the end of the file (`DONE1` tag).
   - Validates the journal date (`JRDATE`) against system and batch constraints:
     - If `JRDATE > SYDYMD` (future date), displays error `MSG,6` ("Cannot enter a future date") and sets indicators `90`, `69`.
     - If `JRDATE ≠ HIDATE` (not the latest ship date), displays error `MSG,8` ("Date entered should be most recent shipto date") and sets indicators `90`, `69`, `51`.
     - If `JRDCYM` (journal date in CYM format) is less than or equal to `ACDTCL` (A/R closed date from `ARCONT`), displays error `MSG,10` ("A/R is closed for the month selected") and sets indicators `90`, `69`.

3. **Journal Date Input Screen (`$S1` Subroutine)**:
   - Handles the `BB600PS1` screen format where the user enters the journal date (`JRDATE`, split into `MM`, `DD`, `YY`).
   - Checks the function key pressed (`@VKEY`):
     - `0` (Enter): Calls `$S1ENT` to validate the date.
     - `2` (Command Key): Calls `$S1CK` to handle cancellation.

4. **Journal Date Validation (`$S1ENT` Subroutine)**:
   - Clears error indicators (`51`, `90`, `60`, `61`, `69`) and message fields.
   - Validates `JRDATE`:
     - If `JRDATE = 0`, displays error `MSG,1` ("Invalid date entered") and exits.
     - Checks `MM` (1–12) and `DD` (1–31) for validity, displaying `MSG,1` if invalid.
   - Calls `$DTEDT` to validate the date format:
     - Breaks down `MMDDYY` into month, day, and year.
     - Validates the month (1–12).
     - Validates the day:
       - For February, checks for leap years (divisible by 4 or 400 for century years) and ensures the day is ≤ 29 (leap year) or ≤ 28 (non-leap year).
       - For other months, checks if the day is ≤ 30 (for April, June, September, November) or ≤ 31 (other months).
     - Sets indicator `79` if the date is invalid.
   - If `79` is set, displays `MSG,1` and exits.
   - Converts `JRDATE` to CYM format (`JRDCYM`) and compares it to the date range (`FRDATE`, `TODATE`):
     - If `JRDATE < FRDATE` or `> TODATE`, sets indicator `51` (override needed).
     - If `JRDATE < FRDYMD` (first day of prior month), displays `MSG,5` ("Date entered earlier than first day of prior month").
     - If `LOMNTH ≠ HIMNTH` (ship dates span different months), displays `MSG,9` ("Cannot post to different months in the same batch").
     - If `JRDATE > SYDYMD`, displays `MSG,6` ("Cannot enter a future date").
     - If `JRDATE ≠ HIDATE`, displays `MSG,8` ("Date entered should be most recent shipto date").
     - If `JRDCYM ≤ ACDTCL`, displays `MSG,10` ("A/R is closed for the month selected").
   - If any validation fails, sets `@SFNEX = 'S2'` to display the confirmation/override screen.

5. **Confirmation/Override Screen (`$S2` Subroutine)**:
   - Handles the `BB600PS2` screen format where the user confirms the date (`YN`) or enters an override code (`CODE`).
   - Checks the function key pressed (`@VKEY`):
     - `0` (Enter): Calls `$S2ENT` to process confirmation.
     - `2` (Command Key): Calls `$S2CK` to handle cancellation or return to `S1`.

6. **Confirmation Processing (`$S2ENT` Subroutine)**:
   - If `YN ≠ 'Y'`, clears fields (`CLRFLD`) and returns to `S1` (`@SFNEX = 'S1'`).
   - If indicator `51` is set (override needed), validates the override code:
     - Retrieves the A/R captcha code (`ARCODE`) from `GSCONT` (key `'00'`).
     - If `CODE ≠ ARCODE`, displays errors `MSG,2` ("Override codes do not match"), `MSG,3` ("Date entered is outside the allowable parameters"), and `MSG,4` ("Please get override code from your supervisor to continue"), and sets indicators `90`, `52`.
     - If valid, clears indicators `51`, `90`.
   - If `YN = 'Y'` and no errors, sets `@SFNEX = 'EJ'` to exit the program.
   - Updates the captcha code in `GSCONT` (`EXCPT UPDCOD`) using a calculated `NEWCOD` based on system date and time.

7. **Command Key Handling (`$S1CK` and `$S2CK` Subroutines)**:
   - `$S1CK`: If `KG` (cancel) is pressed, sets `U1` and `@SFNEX = 'EJ'` to exit.
   - `$S2CK`:
     - If `KA` is pressed, returns to `S1` (`@SFNEX = 'S1'`).
     - If `KG` is pressed, sets `U1` and `@SFNEX = 'EJ'` to exit.

8. **Screen Display (`$XCPT` Subroutine)**:
   - Increments a counter (`@CCNT`) and displays the next screen (`@SFNEX`):
     - `@S1`: Outputs `JRDATE`, `MSGOUT`, `LODMDYY`, `HIDMDYY`, `BATCH#`, `ARMN`, `ARYR` to `BB600PS1`.
     - `@S2`: Outputs `JRDATEY`, `YN`, `MSGC1`, `MSGC2`, `CODE`, `MSGOUT`, `LODMDYY`, `HIDMDYY`, `BATCH#`, `ARMN`, `ARYR` to `BB600PS2`.
   - Calls `CLRIND` to reset indicators (`90`, `81`, `82`, `83`, `84`, `85`, `86`, `59`).

9. **Field and Indicator Management**:
   - `CLRFLD`: Clears `JRDATE`, `YN`, and sets `@SFNEX = 'S1'`.
   - `CLRIND`: Clears error indicators and `MSGOUT`.
   - `CODEUP`: Generates a new captcha code (`NEWCOD`) based on system date and time and updates `GSCONT`.

10. **Termination**:
    - The program exits when `@SFNEX = 'EJ'` or `LR` is set, returning the validated `JRDATE` to the calling OCL program.

---

### Business Rules

1. **Journal Date Validation**:
   - The journal date (`JRDATE`) must be in MMDDYY format and valid (month 1–12, day 1–31, accounting for leap years).
   - The date must fall within the allowable range (`FRDATE` to `TODATE`), typically from the first day of the previous month to the last day of the current month.
   - The date must not be in the future (`JRDATE ≤ SYDYMD`).
   - The date should match the latest ship date (`HIDATE`) in the batch.
   - The date must not be in a closed A/R period (`JRDCYM > ACDTCL`).
   - All invoices in the batch must have ship dates in the same month (`LOMNTH = HIMNTH`).

2. **Override for Out-of-Range Dates**:
   - If the date is outside the allowable range or fails other checks, the user must confirm with `YN = 'Y'` and provide a valid A/R captcha code (`CODE = ARCODE` from `GSCONT`).
   - Invalid codes prevent proceeding and display error messages.

3. **Batch Processing**:
   - Only non-deleted (`BODEL ≠ 'D'`) and non-excluded (`BODEL ≠ 'E'`) invoices in `BBTRAN` are considered for ship date validation (per `JB02`).
   - The batch number (`BATCH#`) is used to filter `BBTRAN` records.

4. **Captcha Code Management**:
   - A new captcha code is generated and stored in `GSCONT` when a valid override is processed.
   - The code is based on a combination of system date and time.

5. **Error Handling**:
   - Displays specific error messages (`MSG,1` to `MSG,10`) for invalid dates, future dates, closed A/R periods, or mismatched ship dates.
   - Requires user confirmation or override for non-compliant dates.

---

### Tables (Files) Used

1. **SCREEN**:
   - A workstation display file (1000 bytes) for user interaction.
   - Supports two formats:
     - `BB600PS1`: Prompts for `JRDATE` and displays batch details and error messages.
     - `BB600PS2`: Prompts for confirmation (`YN`) and override code (`CODE`).
   - Keyed by `@INFDS` for status information.

2. **GSCONT**:
   - General system control file (512 bytes, 2-byte key, update-capable).
   - Fields include:
     - `GXDEL` (1 byte): Delete flag.
     - `GX00` (2 bytes): Key (`'00'`).
     - `GXAR` (1 byte): Accounts Receivable flag.
     - `ARCODE` (6 bytes): A/R captcha code.
   - Used to retrieve and update the A/R captcha code.
   - Disposition: Update-capable (`UF`), shared access (`DISP-SHR` in OCL).

3. **ARCONT**:
   - Accounts Receivable control file (256 bytes, 2-byte key).
   - Fields include:
     - `ACDEL` (1 byte): Delete flag.
     - `ACCO` (2 bytes): Company number.
     - `ACDATE` (6 bytes): Last age date.
     - `ACDTCL` (6 bytes): A/R closed date (CYM format).
     - `ACYEAR` (4 bytes): A/R year.
     - `ACMNTH` (2 bytes): A/R month.
   - Used to validate the journal date against the A/R closed period.
   - Disposition: Input (`IF`), shared access (`DISP-SHR` in OCL).

4. **BBTRAN**:
   - Invoice transaction file (512 bytes, 21-byte key, 492-byte alternate index).
   - Fields include:
     - `BODEL` (1 byte): Delete flag (`'D'` or `'E'` for excluded records).
     - `BOCO` (2 bytes): Company number.
     - `BOSHDT` (6 bytes): Ship date.
     - `BOINV#` (7 bytes): Invoice number.
     - `BOINDT` (6 bytes): Invoice date.
   - Used to retrieve ship dates for the batch to validate `JRDATE`.
   - Disposition: Input (`IF`), shared access (`DISP-SHR` in OCL).

---

### External Programs Called

The `BB600P` RPG program does not explicitly call any external programs. It is a self-contained program that interacts with the user via the `SCREEN` display file and manages data in `GSCONT`, `ARCONT`, and `BBTRAN`.

---

### Summary

The `BB600P` RPG program prompts the user for an A/R journal posting date and validates it against system constraints and invoice ship dates in the specified batch. It:
- Calculates an allowable date range based on the current month.
- Checks invoice ship dates in `BBTRAN` to ensure `JRDATE` aligns with the latest ship date and falls within the same month.
- Requires confirmation or an override code for out-of-range dates.
- Updates the A/R captcha code in `GSCONT` when necessary.
- Exits with a validated `JRDATE` or cancels if the user aborts.

**Tables Used**: `SCREEN` (display file), `GSCONT` (system control), `ARCONT` (A/R control), `BBTRAN` (invoice transactions).
**External Programs Called**: None.