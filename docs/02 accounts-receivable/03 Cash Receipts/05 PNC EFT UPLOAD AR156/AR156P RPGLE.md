The provided document is an RPGLE (RPG IV) program named `AR156P.rpgle`, which is called by the `AR156.ocl36` OCL script. This program is designed to prompt for and validate the selection of a date for EFT (Electronic Funds Transfer) posting table creation, ensuring the input data (company number and batch date) is valid before proceeding with EFT file processing. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the AR156P.rpgle Program

1. **Program Initialization**:
   - The program uses the `DFTACTGRP(*NO)` directive, indicating it runs in a non-default activation group, allowing for better control over resources.
   - The `FIXNBR(*ZONED:*INPUTPACKED)` option ensures proper handling of zoned and packed numeric data during input.
   - The program defines a workstation file `AR156PD` (likely a display file for user interaction) using the Profound UI handler for modernized user interfaces.
   - Data structures and variables are defined:
     - `filenm`: An 8-character field to construct a file name.
     - `kyco`: A 2-digit company number (positions 101-102).
     - `kysldt`: A 6-digit selected date (positions 103-108).
     - `status`: A 1-character field (position 109) to indicate validation status ('Y' for valid, 'N' for invalid).
     - `kyupdt`: A 6-digit update date (positions 110-115).
     - `p$fgrp`: A 1-character file group identifier (position 501).
     - `y2kcen` and `y2kcmp`: Year 2000-related fields for century and company (positions 509-512).
     - `msg`: An array of three 40-character error messages loaded from compile-time data.

2. **Workstation File Read and Initial Logic**:
   - The program checks the `qsctl` field (likely a control flag):
     - If `qsctl` is blank, it sets indicator `*IN09` to '1' (first-time display), `*IN01` to '0', and sets `qsctl` to 'R' (read mode).
     - Otherwise, it sets `*IN09` to '0', `*IN01` to '1', reads the `AR156PD` display file, and returns if the last record is read (`lr` indicator).
   - Indicators 50-55, 20, and 81 are reset to off, ensuring a clean state for error handling and screen output.
   - The `msg35` field (35 characters) is cleared to blanks.

3. **Handle Function Key (F3/F12 Exit)**:
   - If the function key indicator `*INKG` is on (likely F3 or F12 for exit), the program:
     - Sets `*IN01` to off (disables further input processing).
     - Sets `*INU8` and `*INLR` to on (signals program termination).
     - Jumps to the `end` tag to exit.

4. **First-Time Processing (One-Time Setup)**:
   - If `*IN09` is on (first-time display), the `onetim` subroutine is executed:
     - Chains to the `GSCONT` file with key '00' to retrieve a default company number.
     - If the record is found (`*IN99` off) and `GXCONO` (company number) is non-zero, it moves `GXCONO` to `kyco`.
     - Sets `*IN81` to on (indicating screen output is needed) and clears `msg40` (a 40-character message field).
     - Jumps to the `end` tag to display the screen.

5. **Input Validation (Edit Subroutine)**:
   - If `*IN01` is on (input received from the screen), the `edit` subroutine is executed:
     - Resets indicators 20, 30-41, and 90 to off, clearing previous error states.
     - Validates the company number (`kyco`):
       - Chains to the `ARCONT` file using `kyco` as the key.
       - If no record is found (`*IN20` on), sets error indicators `*IN81` and `*IN90`, moves the first error message ("INVALID COMPANY #") to `msg40`, and jumps to `endedt`.
     - Validates the batch create date (`kyupdt`):
       - Compares `kyupdt` to zero.
       - If zero (`*IN20` on), sets error indicators `*IN81` and `*IN90`, moves the second error message ("MUST ENTER BATCH CREATE DATE") to `msg40`, and jumps to `endedt`.
     - Checks if the selected date table exists:
       - Constructs a file name in `filenm` by combining `p$fgrp` (file group) and 'E' with `kyupdt` (e.g., `GE` + date).
       - Calls the external program `AR135TC`, passing `filenm` and `status` as parameters to verify the file's existence.
       - If `status` is 'N' (file does not exist), sets error indicators `*IN81`, `*IN20`, and `*IN90`, moves the third error message ("FILE WITH DATE SELECTED DOES NOT EXIST") to `msg40`, and jumps to `endedt`.
     - If all validations pass, sets `status` to 'Y' (valid input).

6. **Screen Output and Termination**:
   - If no errors are detected (`*IN81` off), sets `*INLR` to on to terminate the program.
   - If errors are detected (`*IN81` on), writes to the `AR156PD` display file to show the error message (`msg40`) and input fields (`kyco`, `kyupdt`).
   - The program ends at the `end` tag.

---

### Business Rules

1. **Company Number Validation**:
   - The user must enter a valid company number (`kyco`) that exists in the `ARCONT` file.
   - If invalid, the program displays "INVALID COMPANY #" and prompts the user to correct the input.

2. **Batch Create Date Validation**:
   - The user must enter a non-zero batch create date (`kyupdt`).
   - If zero or blank, the program displays "MUST ENTER BATCH CREATE DATE" and prompts for correction.

3. **File Existence Check**:
   - The program constructs a file name based on the file group (`p$fgrp`) and the batch create date (`kyupdt`) and checks if the corresponding EFT data file exists.
   - If the file does not exist, the program displays "FILE WITH DATE SELECTED DOES NOT EXIST" and prompts for correction.

4. **First-Time Display**:
   - On the first screen display, the program attempts to retrieve a default company number from the `GSCONT` file (key '00') to prefill the `kyco` field.

5. **Error Handling and User Interaction**:
   - The program uses a display file (`AR156PD`) to interact with the user, showing error messages and allowing input corrections.
   - If the user presses a function key to exit (e.g., F3/F12), the program terminates without further processing.

6. **Status Output**:
   - If all validations pass, the program sets the `status` field to 'Y' to indicate successful validation.
   - If any validation fails, `status` is set to 'N', and an error message is displayed.

---

### Tables (Files) Used

1. **AR156PD** (`CF` - Workstation file):
   - A display file used for user interaction, allowing input of company number (`kyco`) and batch create date (`kyupdt`) and displaying error messages (`msg40`).
   - Uses the Profound UI handler for modernized UI.

2. **ARCONT** (`IF` - Input file):
   - A 256-byte file with a 2-byte key (starting at position 2) used to validate the company number (`kyco`).

3. **GSCONT** (`IF` - Input file):
   - A 512-byte file with a 2-byte key (starting at position 2) used to retrieve a default company number (`GXCONO`) for the first-time display.

---

### External Programs Called

1. **AR135TC**:
   - Called to check the existence of the EFT data file specified by `filenm`.
   - Parameters: `filenm` (file name to check) and `status` (output indicating 'Y' for exists or 'N' for does not exist).

---

### Summary

The `AR156P` program is a user interface and validation module that prompts for and validates the company number and batch create date for EFT posting table creation. It ensures the company number exists in `ARCONT`, the batch date is non-zero, and the corresponding EFT data file exists. The program interacts with the user via a display file (`AR156PD`), retrieves default data from `GSCONT`, and calls `AR135TC` to verify file existence. Upon successful validation, it sets the `status` to 'Y'; otherwise, it displays error messages and allows corrections. The program is a critical step in the EFT file creation process orchestrated by the `AR156.ocl36` script.



### Summary of AR135TC.clp

The `AR135TC.clp` is a CLP (Control Language Program) used in the EFT (Electronic Funds Transfer) draft process to check the existence of a specified file in the `QS36F` library. Below is a brief summary of its functionality:

- **Purpose**: Validates whether a file exists in the `QS36F` library and sets a status indicator accordingly.
- **Parameters**:
  - `&FILE`: An 8-character input parameter specifying the name of the file to check.
  - `&STATUS`: A 1-character output parameter indicating the result ('Y' if the file exists, 'N' if it does not).
- **Process**:
  1. Declares two variables: `&FILE` (8 characters) and `&STATUS` (1 character).
  2. Uses the `CHKOBJ` command to check if the file specified by `&FILE` exists in the `QS36F` library with object type `*FILE`.
  3. If the file does not exist (message ID `CPF9801` is caught via `MONMSG`), sets `&STATUS` to 'N' and jumps to the `END` label.
  4. If the file exists, sets `&STATUS` to 'Y'.
  5. Ends the program with `ENDPGM`.
- **Business Rule**: The program ensures that the EFT process only proceeds if the required data file exists, preventing errors in downstream processing.

This program is called by `AR156P.rpgle` to verify the existence of an EFT data file before proceeding with further validation and processing in the EFT workflow.