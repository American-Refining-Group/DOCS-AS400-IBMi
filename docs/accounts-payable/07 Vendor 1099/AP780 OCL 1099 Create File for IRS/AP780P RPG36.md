The provided document is an RPG/36 program, `AP780P.rpg36.txt`, which is called from the main OCL program (`AP780.ocl36.txt`) to perform preprocessing tasks for creating an IRS 1099 file for Accounts Payable (A/P) processing. Below, I’ll explain the process steps, business rules, tables used, and any external programs called.

### Process Steps of the AP780P RPG/36 Program

The `AP780P` program is designed to validate user input for the 1099 file creation process, ensuring that key fields such as company information, year, and form type are valid before proceeding. It interacts with a display screen and a table file to perform these validations. Here are the detailed steps:

1. **File and Data Structure Definitions**:
   - **Files**:
     - `SCREEN`: A workstation file (display screen) with a record length of 200 characters, used for user input/output.
     - `GSTABL`: An indexed file (256 bytes, 12-byte key, alternate index) opened for input, containing table data.
   - **Data Structures**:
     - `SCREEN` (format `AP780PS1`): Defines fields for user input:
       - `HEAD1` (positions 3–32): First heading (company name).
       - `HEAD2` (positions 33–62): Second heading (address).
       - `CITY` (positions 63–91): City.
       - `STATE` (positions 92–93): State.
       - `ZIP` (positions 94–102): ZIP code.
       - `ID#` (positions 103–112): Tax ID number.
       - `ENTAMT` (positions 113–120, zoned decimal): Entered amount.
       - `YEAR` (positions 121–124, zoned decimal): Year of the 1099.
       - `CURLST` (position 125): Current or last year indicator (`C` or `L`).
       - `FORM` (position 126): Form type.
     - `GSTABL`: Defines table fields:
       - `TBDEL` (position 1): Delete flag.
       - `TBTYPE` (positions 2–7): Table type.
       - `TBCODE` (positions 8–13): Table code.
       - `TBDESC` (positions 14–43): Table description.
     - `UDS` (User Data Structure): Mirrors the `SCREEN` fields for internal processing, with `FORM` at position 160.
   - **Error Messages**: An array `MSG` with six 40-character messages for validation errors (e.g., "FIRST HEADING CANNOT BE BLANK").

2. **Initialization**:
   - Clears indicators 81, 90, 91, 92, 93, 94, 95, 96, and 97 using `SETOF`.
   - Clears the error message field `MSGE` to blanks.

3. **Key Press Handling (Indicator KG)**:
   - If the `KG` indicator is on (likely a function key press, e.g., cancel):
     - Clears indicator 81.
     - Sets indicators `U1` and `LR` (last record).
     - Jumps to the `END` tag, terminating the program.

4. **Screen Input Processing (Indicator 09)**:
   - If indicator `09` is on (indicating a screen input event):
     - Sets indicators 81 and 94.
     - Sets `CURLST` to `C` (current).
     - Jumps to the `END` tag, bypassing further validation.

5. **Validation Subroutine (SUBSC1)**:
   - If indicator `01` is on (initial screen display), executes the `SUBSC1` subroutine:
     - **Validation Checks**:
       - **HEAD1**: Compares `HEAD1` to blanks. If blank, sets indicators 81 and 91 lambdas, moves "FIRST HEADING CANNOT BE BLANK" to `MSGE`, and jumps to `ENDSC1`.
       - **HEAD2**: Compares `HEAD2` to blanks. If blank, sets indicators 81 and 92, moves "SECOND HEADING CANNOT BE BLANK" to `MSGE`, and jumps to `ENDSC1`.
       - **ID#**: Compares `ID#` to blanks. If blank, sets indicators 81 and 93, moves "ID# CANNOT BE BLANK - TRY AGAIN!" to `MSGE`, and jumps to `ENDSC1`.
       - **YEAR**: Compares `YEAR` to 0. If zero, sets indicators 81 and 95, moves "ENTER VALID YEAR" to `MSGE`, and jumps to `ENDSC1`.
       - **CURLST**: Checks if `CURLST` is `C` or `L`. If neither, sets indicators 81 and 96, moves "ENTER 'C'-CURR OR 'L'-LAST" to `MSGE`, and jumps to `ENDSC1`.
       - **FORM**: Constructs `TBLKEY` by concatenating "AP1099" and `FORM`, then chains to `GSTABL`. If the record is not found, sets indicators 81 and 97, moves "INVALID FORM TYPE" to `MSGE`, and jumps to `ENDSC1`.
     - If any validation fails, the program jumps to `ENDSC1`, which outputs the screen with the error message.
   - If all validations pass, the program proceeds to the `END` tag.

6. **Screen Output**:
   - If indicator 81 is on (validation error or first-time display):
     - Outputs the `SCREEN` file with format `AP780PS1`, displaying:
       - `HEAD1`, `HEAD2`, `CITY`, `STATE`, `ZIP`, `ID#`, `ENTAMT`, `YEAR`, `CURLST`, `MSGE`, and `FORM`.
   - The program loops back to accept further input if validations fail.

7. **Program Termination**:
   - Reaches the `END` tag.
   - If validations are successful, the program ends with indicator 60 set (likely indicating successful completion).

### Business Rules

The program enforces the following business rules for the 1099 file creation:
1. **Mandatory Fields**:
   - The first heading (`HEAD1`), second heading (`HEAD2`), and tax ID (`ID#`) must not be blank.
   - The year (`YEAR`) must be a non-zero value.
   - The current/last indicator (`CURLST`) must be either `C` (current) or `L` (last).
   - The form type (`FORM`) must exist in the `GSTABL` table under the key `AP1099` + `FORM`.
2. **Error Handling**:
   - If any validation fails, an appropriate error message is displayed, and the user is prompted to correct the input.
   - The program allows the user to cancel the operation (via the `KG` indicator).
3. **Default Behavior**:
   - If indicator `09` is triggered (specific input event), `CURLST` defaults to `C`.
4. **Data Integrity**:
   - The program ensures that only valid data is passed to subsequent processes by checking against the `GSTABL` table and enforcing field requirements.

### Tables Used

The program uses the following table/file:
1. **GSTABL**:
   - An indexed disk file (256 bytes, 12-byte key, alternate index).
   - Fields:
     - `TBDEL` (1 byte): Delete flag.
     - `TBTYPE` (6 bytes): Table type.
     - `TBCODE` (6 bytes): Table code.
     - `TBDESC` (30 bytes): Table description.
   - Used to validate the `FORM` field by checking if the key `AP1099` + `FORM` exists.

### External Programs Called

The `AP780P` program does not explicitly call any external programs. It is a standalone preprocessing program invoked by the main OCL program (`AP780.ocl36.txt`) to validate user input before further processing.

### Summary

The `AP780P` RPG/36 program is a preprocessing step for the IRS 1099 file creation process. It:
- Displays a screen to collect and validate user input for fields like company name, address, tax ID, year, current/last indicator, and form type.
- Enforces strict business rules to ensure all required fields are non-blank, the year is valid, `CURLST` is `C` or `L`, and the form type exists in `GSTABL`.
- Outputs error messages and prompts for correction if validations fail.
- Uses the `GSTABL` file for form type validation and the `SCREEN` file for user interaction.
- Sets indicators (e.g., 60 for success) to control the flow back to the calling OCL program.

This program ensures that only valid data is passed to subsequent steps in the 1099 file creation process, maintaining data integrity for IRS compliance.