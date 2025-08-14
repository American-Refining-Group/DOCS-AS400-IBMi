The `BB110P.rpg36.txt` is an **RPG II** program running on an IBM System/36, designed to prompt users for order entry picking ticket selection and validate inputs, including freight processor details. It is called by the `BB110P.ocl36` program as part of the order entry picking ticket print process. Below, I’ll explain the process steps, business rules, tables (files) used, and any external programs called.

---

### Process Steps of the BB110P RPG Program

The program facilitates user input for selecting orders and specifying freight processors (or freight arrangement options) for generating picking tickets. It validates order numbers, company codes, and freight processor selections, displaying errors when necessary. Here’s a step-by-step breakdown of the process:

1. **Initialization (Main Line)**:
   - **Line 0047–0048**: If the `KG` key (cancel key) is pressed, sets indicator `U8` and the Last Record (`LR`) indicator to terminate the program.
   - **Line 0050**: If indicators 10 and 11 are off (no prior validation errors), calls the `EDIT` subroutine to validate user input.
   - **Line 0051**: If indicators 10 and 11 are off (first execution), calls the `ONETIM` subroutine to initialize default values.

2. **ONETIM Subroutine (Lines 0101–0109)**:
   - Initializes fields in the Local Data Area (LDA):
     - `KYOE = 'OE'` (Order Entry mode, offset 255–256).
     - `KYCO = 10` (company number, offset 101–102).
     - `KYALSL = 'SEL'` (selection mode, offset 103–105).
     - `KYJOBQ = 'N'` (no job queue, offset 166).
     - `KYCOPY = 01` (number of copies, offset 167–168).
     - `KYCOPB = 01` (backup copies, offset 169–170).
   - Sets indicator 10 to indicate initialization is complete.

3. **EDIT Subroutine (Lines 0053–0099)**:
   - **Line 0054–0055**: Clears error message fields (`MSG35`, `MSG2`) and resets indicators 20, 21, 22, and 25.
   - **Validate Selection Mode (Lines 0058–0060)**:
     - If `KYALSL ≠ 'SEL'` (and indicator 20 is set), displays error "INVALID SELECTION ALL / SEL" (MSG,1) and jumps to `ENDEDT`.
   - **Validate Company Number (Lines 0062–0064)**:
     - Checks `KYCO` against the `BICONT` file. If not found, displays error "INVALID COMPANY #" (MSG,2) and jumps to `ENDEDT`.
   - **Validate Order Numbers (Lines 0069–0086)**:
     - Loops through up to 10 order numbers (`KYO,1` to `KYO,0`, stored in `KYORD1` to `KYORD0`).
     - For each non-zero order (`KYO,X`):
       - Constructs an 11-character key (`KY11`) by combining `KYCO`, `KYO,X`, and `'000'`.
       - Chains to `BBORTR` to verify the order exists. If not found, displays error "INVALID ORDER #" (MSG,3) with the order number and jumps to `ENDEDT`.
     - **Get BOL Marks Record (Lines JB02)**:
       - Constructs a key (`KY11 = '962'`) to retrieve the Bill of Lading (BOL) marks record from `BBORTR`. If not found, sets `BXFRNM` to blanks.
     - **Validate Freight Processor (Lines JB01–JB03, JB12, JB04)**:
       - For each freight processor field (`KYF,X`, from `KYFPC1` to `KYFPC0`):
         - If `KYF,X ≠ 'NO    '`, `KYF,X ≠ 'ARG   '`, and `KYF,X ≠ 'NTT   '`:
           - Constructs a key (`KFPKEY`) from `KYCO` and `KYF,X` and chains to `BBFRPR`. If not found, displays error "INVALID FREIGHT PROCESSOR FOR" (MSG,4) with the order number and jumps to `ENDEDT`.
           - Validates that `KYF,X` matches `BOFPCD` (freight processor code in order header). If not, displays error "INVALID FREIGHT PROCESSOR FOR" (MSG,4) and jumps to `ENDEDT`.
         - If `BXFRNM ≠ *BLANK` (freight bill name exists in BOL marks record):
           - Sets indicator 20, displays errors "XXXXXX ORDER HAS FREIGHT BILL TO" (MSG,5) and "NAME. CANNOT SEND TO FRT PROCESSOR" (MSG,6), and jumps to `ENDEDT`.
         - If `KYF,X = 'NO    '`:
           - If `BOFPCD ≠ 'NO    '` and `BOFPCD ≠ *BLANK`, displays errors "XXXXXX ORDER WAS SENT TO XXXXXX" (MSG,7) and "FRT PROCESSOR" (MSG,8) with the order number and `BOFPCD`, and jumps to `ENDEDT`.
         - If `KYF,X = 'ARG   '` (added JB12):
           - If `BOFPCD ≠ 'ARG   '` and `BOFPCD ≠ *BLANK`, displays errors "XXXXXX ORDER WAS SENT TO XXXXXX" (MSG,7) and "FRT PROCESSOR" (MSG,8) with the order number and `BOFPCD`, and jumps to `ENDEDT`.
         - If `KYF,X = 'NTT   '` (not this time, added JB03):
           - If `BOFPCD = 'NO    '` or `BOFPCD = 'ARG   '`, displays error "XXXXXX ORDER WAS NOT SENT TO XXXXXX" (MSG,9) with the order number and `BOFPCD`, and jumps to `ENDEDT`.
   - **Validate Job Queue (Lines 0089–0092)**:
     - If `KYJOBQ ≠ 'Y'`, `' '`, or `'N'`, sets indicator 20 and jumps to `ENDEDT`.
   - **Set Default Copies (Lines 0094–0097)**:
     - If `KYCOPY = 00`, sets it to 01.
     - If `KYCOPB = 00`, sets it to 01.
   - **Set Completion Indicator (Line 0098)**:
     - Sets indicator 11 to indicate validation is complete.

4. **Output to SCREEN (Lines 0111–0128, JB01, JB02)**:
   - Displays format `BB110PFM` (when indicators 01 and 11 are off) with:
     - Company number (`KYCO`).
     - Selection mode (`KYALSL`).
     - Order numbers (`KYORD1` to `KYORD0`).
     - Job queue flag (`KYJOBQ`).
     - Number of copies (`KYCOPY`).
     - Backup copies (`KYCOPB`).
     - Freight processor codes (`KYFPC1` to `KYFPC0`).
     - Error messages (`MSG35`, `MSG2`).

---

### Business Rules

1. **Selection Mode Validation**:
   - The selection mode (`KYALSL`) must be `'SEL'`. Other values trigger an "INVALID SELECTION ALL / SEL" error.

2. **Company Number Validation**:
   - The company number (`KYCO`) must exist in the `BICONT` file, or an "INVALID COMPANY #" error is displayed.

3. **Order Number Validation**:
   - Up to 10 order numbers (`KYORD1` to `KYORD0`) are validated against the `BBORTR` file using a composite key (company number + order number + `'000'`).
   - Invalid orders trigger an "INVALID ORDER #" error with the specific order number.

4. **Freight Processor Validation (JB01, JB02, JB03, JB04, JB12)**:
   - **Freight Processor Code**:
     - Users can enter a freight processor code (`KYF,X`), `'NO    '` (no freight processor, renamed to `'ARG   '` in JB12), or `'NTT   '` (not this time, added in JB03).
     - The code must exist in the `BBFRPR` file, or an "INVALID FREIGHT PROCESSOR FOR" error is displayed.
     - The entered code (`KYF,X`) must match the order header’s freight processor code (`BOFPCD`), or an error is displayed (JB04).
   - **Freight Bill Name Check (JB02)**:
     - If a freight bill name exists in the BOL marks record (`BXFRNM ≠ *BLANK`), users cannot select a freight processor, triggering errors "XXXXXX ORDER HAS FREIGHT BILL TO" and "NAME. CANNOT SEND TO FRT PROCESSOR".
   - **No Freight Processor ('NO' or 'ARG')**:
     - If `KYF,X = 'NO    '` or `'ARG   '`, the order must not have been previously sent to a freight processor (`BOFPCD = 'NO    '` or `'ARG   '`), or errors "XXXXXX ORDER WAS SENT TO XXXXXX" and "FRT PROCESSOR" are displayed.
   - **Not This Time ('NTT')**:
     - If `KYF,X = 'NTT   '`, the order must have been previously sent to a freight processor (`BOFPCD ≠ 'NO    '` and `BOFPCD ≠ 'ARG   '`), or an error "XXXXXX ORDER WAS NOT SENT TO XXXXXX" is displayed.
   - **Update Handling**:
     - Freight processor updates occur in the `BB110` program, not in `BB110P`.

5. **Job Queue Validation**:
   - The job queue flag (`KYJOBQ`) must be `'Y'`, `'N'`, or blank, or an error is triggered.

6. **Default Copies**:
   - If the number of copies (`KYCOPY`) or backup copies (`KYCOPB`) is zero, it defaults to 1.

7. **Error Handling**:
   - Errors are displayed with specific messages, including the problematic order number or freight processor code, ensuring users correct invalid inputs before proceeding.

---

### Tables (Files) Used

1. **SCREEN**:
   - **Type**: Workstation file (WORKSTN).
   - **Purpose**: Displays the input prompt for order numbers, company code, job queue flag, copies, and freight processor codes, along with error messages.
   - **Format**: `BB110PFM`.

2. **BICONT**:
   - **Type**: Indexed disk file (256 bytes, read-only, shared access).
   - **Purpose**: Validates the company number (`KYCO`).

3. **BBORTR**:
   - **Type**: Indexed disk file (512 bytes, read-only, shared access).
   - **Purpose**: Validates order numbers and retrieves freight processor code (`BOFPCD`) and BOL marks record (`BXFRNM`).
   - **Records**:
     - Header record: Contains `BOFPCD` (freight processor code), `BOFPST` (freight processor status), `BOVERN` (order version number).
     - Marks record: Contains `BXFRNM` (freight bill name).

4. **BBFRPR**:
   - **Type**: Indexed disk file (256 bytes, read-only, shared access).
   - **Purpose**: Validates freight processor codes entered by the user (`KYF,X`).

---

### External Programs Called

- **None**: The `BB110P` program does not explicitly call any external RPG programs. It performs input validation and relies on the calling OCL program (`BB110P.ocl36`) to proceed with further processing (e.g., invoking `BB110` for freight processor updates).

---

### Summary

The `BB110P` RPG program is a user interface for selecting orders and freight processors for picking ticket generation. It:
- Initializes default values for company, selection mode, job queue, and copies.
- Validates company numbers, order numbers, and freight processor codes against `BICONT`, `BBORTR`, and `BBFRPR` files.
- Enforces strict business rules for freight processor selection, ensuring consistency with order header data and preventing conflicts with existing freight bill names.
- Displays errors for invalid inputs and allows users to correct them.

**Tables Used**: SCREEN, BICONT, BBORTR, BBFRPR.
**External Programs Called**: None.