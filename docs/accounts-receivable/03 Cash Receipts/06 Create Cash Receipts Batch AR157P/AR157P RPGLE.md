The provided document, `AR157P.rpgle.txt`, is an RPGLE (RPG IV) program used in IBM midrange systems (e.g., AS/400, iSeries) to handle the prompting and validation for EFT (Electronic Funds Transfer) cash receipts batch file creation. It is called from the main OCL program (`AR157P.ocl36.txt`) described previously. Below is a detailed explanation of the process steps, business rules, tables/files used, and external programs called, based on the RPGLE source code.

---

### Process Steps of the RPGLE Program (`AR157P`)

The `AR157P` RPGLE program is responsible for validating user input for creating a cash receipts batch file, interacting with a workstation display file (`AR157PFM`), and performing checks on company numbers, upload dates, and file existence. The program uses a combination of file operations, user interface handling, and subroutine logic to ensure valid data before proceeding. Here’s a step-by-step breakdown of the process:

1. **Program Initialization**:
   - The program declares the default activation group (`DFTACTGRP(*NO)`) and sets the default program name to `AR157P` (`DFTNAME(AR157P)`).
   - It defines the workstation file `AR157PFM` as a combined file (`CF`) for input/output with a handler for Profound UI (`HANDLER('PROFOUNDUI(HANDLER)')`), indicating a modernized user interface.
   - Two input files, `ARCONT` and `GSCONT`, are defined for validation purposes.
   - Data structures (`FILENM`), variables (`KYCO`, `KYSLDT`, `STATUS`, `KYUPDT`, `KYDELT`, `EXISTS`), and arrays (`MSG`, `COM`) are initialized for storing user input, status flags, and error messages.
   - The program sets initial indicator states (`*IN50` to `*IN55`, `*IN20`, `*IN81`, `*IN21`, `*IN90`) to `*OFF` to reset error and control flags.
   - Screen message fields (`MSG35`, `MSG1`, `MSG2`, `MSG40`) are cleared to blanks.

2. **Handle Workstation Input (Read `AR157PFM`)**:
   - The program checks the value of `QSCTL` (likely a control field from the workstation file):
     - If `QSCTL` is blank, it sets `*IN09` to `*ON` (indicating initial screen display), `*IN01` to `*OFF`, and sets `QSCTL` to `'R'` (likely to indicate a read operation).
     - If `QSCTL` is not blank, it sets `*IN09` to `*OFF`, `*IN01` to `*ON`, reads the `AR157PFM` workstation file, and checks for the last record (`LR`). If the last record is read, the program returns immediately.
   - This step manages the initial display of the prompt screen or processes user input from the screen.

3. **Handle Function Key `F3` (Exit)**:
   - If the `F3` key (`*INKG`) is pressed:
     - The program sets `*IN01` to `*OFF`, `*INU8` to `*ON`, and `*INLR` to `*ON` (indicating program termination).
     - It jumps to the `END` tag, exiting the program.
   - This allows the user to exit the program without further processing.

4. **Handle Function Key `F12` (Delete)**:
   - If the `F12` key (`*INKL`) is pressed:
     - The program sets `*IN01` and `*IN81` to `*OFF`, `*INLR` to `*ON`, and sets `KYDELT` to `'Y'` (indicating a request to delete a batch file).
     - It jumps to the `END` tag, exiting the program.
   - This allows the user to signal deletion of an existing cash receipts batch file (handled in the OCL program).

5. **Initial Screen Processing (`ONETIM` Subroutine)**:
   - If `*IN09` is `*ON` (initial screen display):
     - The program executes the `ONETIM` subroutine.
     - It chains (reads) the `GSCONT` file using key `'00'` to retrieve a default company number (`GXCONO`).
     - If a record is found (`*IN99` is `*OFF`) and `GXCONO` is non-zero, it sets `KYCO` to `GXCONO`.
     - It sets `*IN81` to `*ON` to trigger screen output.
     - If `EXISTS` is `'Y'` (indicating a cash receipts batch file already exists), it sets error message `MSG1` to `COM(1)` ("A CASH RECEIPTS BATCH ALREADY EXISTS. PRESS F12 TO DELETE.") and sets `*IN81`, `*IN21`, and `*IN90` to `*ON` to display the error.
     - The program jumps to the `END` tag after completing the subroutine.

6. **Validate User Input (`EDIT` Subroutine)**:
   - If `*IN01` is `*ON` (user has submitted input):
     - The program executes the `EDIT` subroutine to validate the input.
     - **Validate Company Number**:
       - Chains the `ARCONT` file using `KYCO` (company number).
       - If no record is found (`*IN20` is `*ON`), it sets `*IN81` and `*IN90` to `*ON`, sets `MSG40` to `MSG(1)` ("INVALID COMPANY #"), and jumps to `ENDEDT`.
     - **Validate Bank Upload Date**:
       - Compares `KYUPDT` (bank upload date) to zero.
       - If `KYUPDT` is zero (`*IN20` is `*ON`), it sets `*IN81` and `*IN90` to `*ON`, sets `MSG40` to `MSG(2)` ("MUST ENTER BANK UPLOAD DATE"), and jumps to `ENDEDT`.
     - **Check File Existence**:
       - Constructs a filename by moving `'GE'` and `KYUPDT` into `FILENM`.
       - Calls the external program `AR135TC`, passing `FILENM` and `STATUS` as parameters to check if a file with the selected date exists.
       - If `STATUS` is `'N'` (file does not exist), it sets `MSG40` to `MSG(3)` ("FILE WITH DATE SELECTED DOES NOT EXIST"), sets `*IN81`, `*IN20`, and `*IN90` to `*ON`, and jumps to `ENDEDT`.
     - **Successful Validation**:
       - If all checks pass, sets `STATUS` to `'Y'` to indicate valid input.
     - The `ENDEDT` tag ends the subroutine.

7. **Write to Workstation and Terminate**:
   - If `*IN81` is `*OFF`, the program sets `*INLR` to `*ON` to terminate.
   - If `*IN81` is `*ON`, it writes to the `AR157PFM` workstation file to display errors or updated screen data.
   - The program then terminates at the `END` tag.

---

### Business Rules

The `AR157P` program enforces the following business rules for EFT cash receipts batch file creation:

1. **Company Number Validation**:
   - The user must enter a valid company number (`KYCO`).
   - The program checks the `ARCONT` file to ensure the company number exists. If invalid, it displays "INVALID COMPANY #".

2. **Bank Upload Date Requirement**:
   - The user must enter a non-zero bank upload date (`KYUPDT`).
   - If the date is missing (zero), the program displays "MUST ENTER BANK UPLOAD DATE".

3. **File Existence Check**:
   - The program checks if a file corresponding to the selected upload date exists by calling `AR135TC`.
   - If the file does not exist, it displays "FILE WITH DATE SELECTED DOES NOT EXIST".

4. **Existing Batch File Handling**:
   - If a cash receipts batch file already exists (`EXISTS = 'Y'`), the program displays "A CASH RECEIPTS BATCH ALREADY EXISTS. PRESS F12 TO DELETE." and prompts the user to press `F12` to delete it.

5. **User Exit Options**:
   - Pressing `F3` exits the program without further processing.
   - Pressing `F12` signals a request to delete an existing batch file (sets `KYDELT` to `'Y'`).

6. **Default Company Number**:
   - On initial screen display, the program retrieves a default company number from the `GSCONT` file (key `'00'`) and populates `KYCO` if a valid record is found.

7. **Error Handling and Screen Updates**:
   - Errors are displayed on the workstation screen (`AR157PFM`) with appropriate messages.
   - The program uses indicators (`*IN81`, `*IN90`, etc.) to control screen output and error states.

---

### Tables/Files Used

The program interacts with the following files:

1. **AR157PFM**:
   - Type: Combined workstation file (`CF`)
   - Purpose: Handles user input/output for the prompt screen, managed by Profound UI.
   - Usage: Read for user input, written to display errors or updated data.

2. **ARCONT**:
   - Type: Input file (`IF`), disk-based, 256 bytes, keyed access (`KEYLOC(2)`).
   - Purpose: Stores Accounts Receivable control data, used to validate the company number (`KYCO`).
   - Key: Starts at position 2.

3. **GSCONT**:
   - Type: Input file (`IF`), disk-based, 512 bytes, keyed access (`KEYLOC(2)`).
   - Purpose: Stores general system control data, used to retrieve a default company number (`GXCONO`) with key `'00'`.
   - Key: Starts at position 2.

---

### External Programs Called

The program calls the following external program:
1. **AR135TC**:
   - Purpose: Checks the existence of a file based on the constructed filename (`FILENM`) derived from `'GE'` and `KYUPDT`.
   - Parameters:
     - `FILENM`: Input/output parameter containing the file name to check.
     - `STATUS`: Output parameter indicating whether the file exists (`'Y'` or `'N'`).

---

### Notes
- **Conversion Details**: The program was converted on 08/31/23 using TARGET/400, with 56 added lines, 7 modified lines, and 112 total RPG source lines processed. The `T4A`, `T4M`, and `T4O` prefixes indicate added, modified, and original lines, respectively.
- **Dynamic File Naming**: The `FILENM` data structure constructs a file name using `'GE'` and `KYUPDT`, suggesting a naming convention like `GEYYYYMM` for the cash receipts file.
- **Profound UI Integration**: The use of `HANDLER('PROFOUNDUI(HANDLER)')` indicates a modernized user interface, likely replacing a traditional 5250 green-screen interface.
- **Link to OCL Program**: The `AR157P` RPGLE program is loaded and run by the `AR157P.ocl36` OCL program. Variables like `KYDELT` and `EXISTS` are shared with the OCL program to handle file deletion and existence checks.
- **Error Messages**: The program uses predefined error messages (`MSG` array) and a common message (`COM` array) to communicate validation failures or batch file existence to the user.

If you need further clarification, additional details about related programs (e.g., `AR135TC`), or integration with the OCL program, let me know!