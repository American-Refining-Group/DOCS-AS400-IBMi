The provided document, `AR933P.rpgle.txt`, is an RPGLE (RPG IV) program that updates the Accounts Receivable (A/R) control file (`ARCONT`) with a new closed month/year value for a specified company. It is called from the OCL program `AR933P.ocl36.txt` and interacts with a workstation display file (`AR933PD`) for user input and output. Below, I’ll explain the process steps, business rules, tables used, and external programs called, based on the RPGLE code.

---

### Process Steps of the RPGLE Program

The `AR933P` RPGLE program manages the update of the A/R closed date in the `ARCONT` file, with user interaction via a display file (`AR933PD`). The program validates user input (company number, month, and year) and updates the corresponding record in `ARCONT`. Here’s a detailed breakdown of the process steps:

1. **Program Initialization**:
   - **Header Specifications**:
     - `DFTACTGRP(*NO)`: Runs in a named activation group, not the default, allowing better control over program resources.
     - `DFTNAME(AR933P)`: Sets the default program name to `AR933P`.
     - `FIXNBR(*ZONED:*INPUTPACKED)`: Ensures zoned and packed numeric fields are handled correctly during input.
   - **File Declarations**:
     - `AR933PD CF E WORKSTN`: A workstation file for user interaction, using a handler (`PROFOUNDUI(HANDLER)`) for modern UI support (e.g., Profound UI).
     - `ARCONT UF F 256 2AIDISK KEYLOC(2)`: The A/R control file, updatable, with a record length of 256 bytes and a key starting at position 2 (company number, `ACCO`).
     - `GSCONT IF F 512 2AIDISK KEYLOC(2)`: A general system control file, input-only, with a key at position 2 (company number, `GXCONO`).
   - **Data Definitions**:
     - Arrays (`COM`, `DCNO`, `DCO`, `DCM`, `DCY`) store error messages and company data (company numbers, names, months, and years) for display.
     - Data structure (`UDS`) defines `KYCANC` (positions 129-134) to store a cancellation flag.
     - Input specifications define fields for `AR933PD` (company number, month, year) and `ARCONT` (delete flag, company number, name, closed date fields).

2. **Workstation File Read and Initial Logic** (`C` Specs, Lines T4A):
   - **Check `QSCTL`**:
     - If `QSCTL` (a control field, likely set by the system or OCL) is blank, the program sets indicators `*IN09` (display initial screen) and `*IN01` (process screen input) to control flow and sets `QSCTL` to `'R'` (read mode).
     - Otherwise, it reads the display file (`AR933PD`), sets `*INLR` (last record), and returns, terminating the program. This likely handles a specific system state or reinvocation scenario.
   - **Clear Messages and Indicators**:
     - Clears message fields (`MSG`, `MSG2`) and resets indicators (`*IN51`, `*IN52`, `*IN53`, `*IN54`, `*IN81`, `*IN90`) to ensure a clean state.
   - **Handle Cancel Key**:
     - If `*INKG` (cancel key) is on, the program sets `*INLR` (last record), clears `*IN01` and `*IN09`, sets `KYCANC` to `'CANCEL'`, and exits. This corresponds to the OCL error check (`L'129,6'`).

3. **Initial Screen Display** (`*IN09` Logic):
   - If `*IN09` is on (initial screen display), the program:
     - Sets `*IN81` to display the screen.
     - Calls the `ONETIM` subroutine to load company data for display.

4. **Screen Input Processing** (`*IN01` Logic):
   - If `*IN01` is on (user input received), the program calls the `SCREEN1` subroutine to validate and process the input.

5. **Write to Display and Termination**:
   - If `*IN81` is off, the program sets `*INLR` and exits (no screen display needed).
   - If `*IN81` is on, it writes to the display file (`AR933PD`) to show results or errors.
   - The program ends at the `END` tag.

6. **ONETIM Subroutine**:
   - **Purpose**: Loads company data from `ARCONT` for display.
   - **Steps**:
     - Chains to `GSCONT` using `'00'` to retrieve a control record. If found and `GXCONO` (company number) is non-zero, it sets `KYCONO` (key company number).
     - Captures the current time (`TIME12`) and moves it to `SYDATE` and `TIMEOF`.
     - Clears the `DCO` array and initializes `X` (index) and `ACLIM` (limit counter).
     - Sets the lower limit (`SETLL`) on `ARCONT` using `ACLIM` (likely `00`).
     - Reads `ARCONT` records in a loop until end-of-file (`*IN10`) or three records are processed:
       - Skips records with `ACDEL = 'D'` (deleted).
       - Stores company number (`ACCO`), name (`ACNAME`), month (`ACMNTH`), and year (`ACYEAR`) in arrays (`DCNO`, `DCO`, `DCM`, `DCY`).
       - Increments `X` until it reaches 3 or end-of-file.
     - Moves array data to display fields (`DCCO1-3`, `DCNM1-3`, `DCMN1-3`, `DCYR1-3`).
     - Sets `*IN81` to display the screen.

7. **SCREEN1 Subroutine**:
   - **Purpose**: Validates user input and updates `ARCONT`.
   - **Steps**:
     - **Validate Company Number**:
       - Chains to `ARCONT` using `KYCONO` (input company number).
       - If not found (`*IN99`), sets error indicators (`*IN81`, `*IN90`, `*IN51`), displays error message `COM(1)` ("INVALID COMPANY NUMBER"), and jumps to `ENDS1`.
     - **Validate Month**:
       - Checks if `KYMNTH` (input month) is between 01 and 12. If not, sets error indicators and displays `COM(2)` ("INVALID MONTH ENTERED").
     - **Validate Year**:
       - Checks if `KYYEAR` (input year) is between 2000 and 2079. If not, sets error indicators and displays `COM(3)` ("INVALID YEAR ENTERED").
     - **Validate Date**:
       - Combines `KYYEAR` and `KYMNTH` into `DATE` (6-digit format, e.g., YYYYMM).
       - Compares `ACDTCL` (current closed date in `ARCONT`) with `DATE`. If `DATE` is earlier, sets error indicators and displays `COM(4)` ("NEW DATE IS EARLIER THAN PREVIOUS DATE").
     - **Update `ARCONT`**:
       - If all validations pass, updates `ARCONT` with `KYMNTH` and `KYYEAR` using the `UPDATE` operation.
     - **Reload and Display**:
       - Calls `ONETIM` to refresh company data.
       - Sets success message `COM(5)` ("COMPANY A/R CLOSED DATE WAS SUCCESSFULLY UPDATED").
       - Sets `*IN81` to display the updated screen.
     - Ends at `ENDS1`.

8. **Output Specifications**:
   - Updates `ARCONT` with `KYMNTH` (position 131) and `KYYEAR` (positions 126-129).
   - Writes to `AR933PD` (commented out in original, modified to `AR933PFM`) with fields like company number, month, year, and messages.

---

### Business Rules

The program enforces the following business rules for updating the A/R closed date:
1. **Company Number Validation**:
   - The entered company number (`KYCONO`) must exist in the `ARCONT` file. If not, display "INVALID COMPANY NUMBER".
2. **Month Validation**:
   - The entered month (`KYMNTH`) must be between 01 and 12. If not, display "INVALID MONTH ENTERED".
3. **Year Validation**:
   - The entered year (`KYYEAR`) must be between 2000 and 2079. If not, display "INVALID YEAR ENTERED".
4. **Date Progression**:
   - The new closed date (`KYYEAR` + `KYMNTH`) must not be earlier than the existing closed date (`ACDTCL`) in `ARCONT`. If it is, display "NEW DATE IS EARLIER THAN PREVIOUS DATE".
5. **Successful Update**:
   - If all validations pass, update the `ARCONT` record with the new month and year, and display "COMPANY A/R CLOSED DATE WAS SUCCESSFULLY UPDATED".
6. **Cancellation**:
   - If the user presses the cancel key (`*INKG`), set `KYCANC` to `'CANCEL'` and exit, allowing the OCL program to detect this via `L'129,6'`.
7. **Display Logic**:
   - Display up to three non-deleted (`ACDEL ≠ 'D'`) company records from `ARCONT`, including company number, name, and current closed month/year.
   - Show error or success messages based on validation results.

---

### Tables (Files) Used

1. **AR933PD**:
   - Type: Workstation (display) file.
   - Purpose: Handles user input (company number, month, year) and output (company data, messages).
   - Fields: `CONO` (company number), `MONTH`, `YEAR`, `DCNO` (company numbers), `DCO` (company names), `DCM` (months), `DCY` (years), `MSG`, `MSG2`.
2. **ARCONT**:
   - Type: Updatable disk file, 256 bytes, keyed on company number (`ACCO`, position 2).
   - Purpose: Stores A/R control data, including company number (`ACCO`), name (`ACNAME`), closed month (`ACMNTH`), year (`ACYEAR`), and delete flag (`ACDEL`).
   - Fields Updated: `ACMNTH` (position 130-131), `ACYEAR` (position 126-129).
3. **GSCONT**:
   - Type: Input-only disk file, 512 bytes, keyed on company number (`GXCONO`, position 2).
   - Purpose: General system control file, used to retrieve a control record (key `'00'`) to set the company number (`KYCONO`).

---

### External Programs Called

- **None**:
  - The RPGLE program does not explicitly call external programs (e.g., via `CALL` operation).
  - The `PROFOUNDUI(HANDLER)` in the `AR933PD` file specification indicates integration with Profound UI for display handling, but this is a runtime handler, not a separate program.

---

### Summary

The `AR933P` RPGLE program, called by the OCL program `AR933P.ocl36`, updates the A/R closed date in the `ARCONT` file based on user input from the `AR933PD` display file. It:
- Loads company data from `ARCONT` and `GSCONT` for display.
- Validates user input (company number, month, year) against strict business rules.
- Updates the `ARCONT` record if valid, or displays error messages.
- Handles cancellation and displays success messages.
- Uses subroutines `ONETIM` (load data) and `SCREEN1` (validate/update).

**Tables Used**: `AR933PD` (workstation), `ARCONT` (A/R control), `GSCONT` (system control).
**External Programs Called**: None.

If you need further details (e.g., interaction with the OCL program, specific field mappings, or UI behavior), please let me know!