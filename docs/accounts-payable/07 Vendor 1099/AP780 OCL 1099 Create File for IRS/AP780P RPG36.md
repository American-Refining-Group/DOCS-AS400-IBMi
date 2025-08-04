The provided document, `AP780P.rpg36.txt`, is an RPG (Report Program Generator) program for an IBM System/3x or AS/400 system, called by the main OCL program (`AP780.ocl36.txt`) to perform initial validation and processing for creating an IRS 1099 file. Below is a detailed explanation of the process steps, business rules, tables used, external programs called, and outputs of the `AP780P` RPG program.

### Process Steps of the AP780P RPG Program

The `AP780P` program validates user input for creating an IRS 1099 file, specifically focusing on company and vendor data used in the 1099-MISC file generation process. It uses a display file (`SCREEN`) for user interaction and a table file (`GSTABL`) for validation. Here’s a step-by-step breakdown of the process:

1. **Program Initialization**:
   - **Set Indicators Off**: The program begins by resetting indicators 81, 90, 91, 92, 93, 94, 95, 96, and 97 (`SETOF 819091`, `SETOF 929394`, `SETOF 959697`) to ensure a clean state.
   - **Clear Error Message**: The message field `MSGE` (40 characters) is cleared to blanks (`MOVEL*BLANKS MSGE`).

2. **Check for Test Environment (KG Indicator)**:
   - If the `KG` indicator is on (likely set by the OCL program when `?9?` indicates a test environment, e.g., `?9?=TEST`):
     - Indicator 81 is turned off (`SETOF 81`).
     - Indicator `U1` (user indicator) and `LR` (last record) are turned on (`SETON U1LR`).
     - The program jumps to the `END` tag, bypassing further processing and effectively terminating early.
   - This ensures no validation or screen interaction occurs in a test environment, possibly to avoid unintended changes.

3. **Set Indicator 60**:
   - Indicator 60 is turned on (`SETON 60`), likely used to control display file behavior or program flow, though its specific purpose is not detailed in the code.

4. **Check for Screen Input (Indicator 09)**:
   - If indicator 09 is on (indicating input from the `SCREEN` file, format `AP780PS1`):
     - Sets indicator 81 and 94 (`SETON 8194`).
     - Sets the `CURLST` field to `'C'` (indicating "current" list).
     - Jumps to the `END` tag, skipping further processing.
   - This suggests that if the screen has already been processed, the program assumes default values or skips validation.

5. **Execute Validation Subroutine (SUBSC1)**:
   - If indicator 09 is not on, the program calls the `SUBSC1` subroutine (`EXSR SUBSC1`) to validate input fields from the `SCREEN` file or user data structure (`UDS`).

6. **SUBSC1 Subroutine (Validation Logic)**:
   The `SUBSC1` subroutine validates key fields required for the 1099 file. Each validation step checks for specific conditions, sets error indicators, and assigns error messages if validation fails. If any validation fails, the program jumps to `ENDSC1` to exit the subroutine.

   - **Validate HEAD1 (Company Name)**:
     - Checks if `HEAD1` (positions 3-32, company name) is blank (`COMP *BLANKS`).
     - If blank, sets indicators 81 and 91 (`SETON 8191`), moves message "FIRST HEADING CANNOT BE BLANK" to `MSGE`, and jumps to `ENDSC1`.

   - **Validate HEAD2 (Address Line)**:
     - Checks if `HEAD2` (positions 33-62, address line) is blank.
     - If blank, sets indicators 81 and 92 (`SETON 8192`), moves message "SECOND HEADING CANNOT BE BLANK" to `MSGE`, and jumps to `ENDSC1`.

   - **Validate ID# (Tax ID)**:
     - Checks if `ID#` (positions 103-112, tax ID) is blank.
     - If blank, sets indicators 81 and 93 (`SETON 8193`), moves message "ID# CANNOT BE BLANK - TRY AGAIN!" to `MSGE`, and jumps to `ENDSC1`.

   - **Validate YEAR**:
     - Checks if `YEAR` (positions 121-124, four-digit year) is zero (`COMP 0`).
     - If zero, sets indicators 81 and 95 (`SETON 8195`), moves message "ENTER VALID YEAR" to `MSGE`, and jumps to `ENDSC1`.

   - **Validate CURLST (Current/Last List Indicator)**:
     - Checks if `CURLST` (position 125, current/last list flag) is neither `'C'` nor `'L'` (`COMP 'C'` and `COMP 'L'`).
     - If invalid, sets indicators 81 and 96 (`SETON 8196`), moves message "ENTER 'C'-CURR OR 'L'-LAST" to `MSGE`, and jumps to `ENDSC1`.

   - **Validate FORM (1099 Form Type)**:
     - Constructs a key `TBLKEY` (12 characters) by combining the literal `'AP1099'` with the `FORM` field (position 160, e.g., `'C'` for 1099-MISC).
     - Performs a `CHAIN` operation on the `GSTABL` file using `TBLKEY` to check if the form type exists in the table.
     - If the record is not found (indicator 90 on), sets indicators 81 and 97 (`SETON 8197`), moves message "INVALID FORM TYPE" to `MSGE`, and jumps to `ENDSC1`.

7. **End Subroutine (ENDSC1)**:
   - The `ENDSC1` tag marks the end of the `SUBSC1` subroutine. If any validation fails, the program exits the subroutine and proceeds to output the screen with the error message.

8. **Output to SCREEN**:
   - If indicator 81 is on (indicating a validation error or first-time display), the program writes to the `SCREEN` file using format `AP780PS1`.
   - Outputs the following fields:
     - `HEAD1` (company name, positions 1-30).
     - `HEAD2` (address line, positions 31-60).
     - `CITY` (city, positions 61-89).
     - `STATE` (state, positions 90-91).
     - `ZIP` (zip code, positions 92-100).
     - `ID#` (tax ID, positions 101-110).
     - `ENTAMT` (entered amount, positions 113-118, zoned decimal).
     - `YEAR` (year, positions 119-122).
     - `CURLST` (current/last flag, position 123).
     - `MSGE` (error message, positions 124-163).
     - `FORM` (form type, position 164).
   - The screen displays these fields for user input or correction, along with any error message from validation failures.

9. **Program Termination (END Tag)**:
   - The program reaches the `END` tag, either after validation, screen output, or early termination (e.g., test environment or indicator 09).
   - If `U1` and `LR` are on (set in the test environment case), the program terminates, potentially signaling an error to the calling OCL program via `SWITCH1-1`.

### Business Rules

The `AP780P` program enforces the following business rules for 1099 file creation:
1. **Mandatory Fields**:
   - The company name (`HEAD1`), address line (`HEAD2`), and tax ID (`ID#`) must not be blank, as they are critical for IRS 1099 reporting.
   - The year (`YEAR`) must be a valid non-zero value (e.g., 2025).
   - The current/last list flag (`CURLST`) must be either `'C'` (current) or `'L'` (last), indicating the type of vendor list to process.
   - The form type (`FORM`) must exist in the `GSTABL` table, ensuring only valid 1099 form types (e.g., `'C'` for 1099-MISC) are processed.

2. **Error Handling**:
   - If any validation fails, an appropriate error message is displayed on the screen, and indicators 81 and a specific error indicator (91-97) are set to highlight the issue.
   - The program prevents further processing until all validations pass, ensuring data integrity for the 1099 file.

3. **Test Environment Handling**:
   - In a test environment (indicated by `KG`), the program skips validation and screen interaction, setting `U1` and `LR` to signal early termination, likely to avoid modifying production data.

4. **Screen Interaction**:
   - The program uses the `SCREEN` file (format `AP780PS1`) to interact with the user, displaying input fields and error messages.
   - If the screen has already been processed (indicator 09), it defaults `CURLST` to `'C'` and skips further validation, assuming prior input is valid.

### Tables Used

- **`GSTABL`**:
  - **Type**: Input file (`IF`), fixed format, 256 bytes per record, 256 records, with an alternate index (`AI`) on 12 bytes.
  - **Fields**:
    - `TBDEL` (position 1): Delete flag ('D' for deleted records).
    - `TBTYPE` (positions 2-7): Table type.
    - `TBCODE` (positions 8-13): Table code.
    - `TBDESC` (positions 14-43): Table description.
  - **Usage**: The program uses `GSTABL` to validate the `FORM` field by constructing a key (`TBLKEY`) combining `'AP1099'` and `FORM` (e.g., `'AP1099C'` for 1099-MISC). The `CHAIN` operation checks if the form type exists in the table.
  - **Access**: Accessed via a `CHAIN` operation with a 12-byte key.

### External Programs Called

- **None**: The `AP780P` program does not explicitly call any external RPG or OCL programs. It is invoked by the main OCL program (`AP780.ocl36.txt`) and performs its tasks independently, relying on file I/O and user interaction via the `SCREEN` file.

### Outputs

1. **SCREEN File (AP780PS1 Format)**:
   - **Purpose**: Displays input fields and error messages to the user for validation or correction.
   - **Fields Output**:
     - `HEAD1`: Company name (30 characters).
     - `HEAD2`: Address line (30 characters).
     - `CITY`: City (29 characters).
     - `STATE`: State (2 characters).
     - `ZIP`: Zip code (9 characters).
     - `ID#`: Tax ID (10 characters).
     - `ENTAMT`: Entered amount (8 bytes, zoned decimal, 2 decimal places).
     - `YEAR`: Four-digit year (4 bytes, zoned decimal).
     - `CURLST`: Current/last flag (1 character, 'C' or 'L').
     - `MSGE`: Error message (40 characters).
     - `FORM`: 1099 form type (1 character, e.g., 'C' for 1099-MISC).
   - **Conditions**: Written when indicator 81 is on (first-time display or validation error).

2. **Indicators**:
   - **Indicator 81**: Set on for screen output or validation errors, signaling the calling OCL program (`AP780.ocl36.txt`) to display the screen.
   - **Indicators 91-97**: Set to indicate specific validation errors (e.g., 91 for blank `HEAD1`, 93 for blank `ID#`, etc.).
   - **U1 and LR**: Set in test environment (`KG` on) to indicate early termination, potentially setting `SWITCH1-1` in the OCL program to signal an error or bypass condition.

3. **No Direct File Output**:
   - The program does not directly create or update any data files (e.g., `AP1099` or `APVNYYYY`). Its primary role is to validate input data, which is then used by subsequent programs (`AP781`, `AP780`) in the OCL process to generate the 1099 file.
   - The validated data is likely stored in the `SCREEN` file or user data structure (`UDS`) and passed back to the OCL program for further processing.

### Additional Notes

- **Purpose in the OCL Workflow**: `AP780P` serves as the initial validation step in the 1099 file creation process, ensuring that critical fields (company name, address, tax ID, year, list type, and form type) are valid before subsequent programs (`#GSORT`, `AP781`, `AP780`) process the vendor data.
- **Form Type Limitation**: The program is designed for 1099-MISC (likely indicated by `FORM='C'`), as noted in the OCL program. Other 1099 types would require modifications to the validation logic and `GSTABL` table.
- **Table Dependency**: The `GSTABL` table is critical for validating the `FORM` field, and its absence or incorrect data could cause validation failures.
- **Error Handling**: The program provides clear, user-friendly error messages via the `SCREEN` file, ensuring users can correct invalid inputs.
- **Test Environment**: The `KG` indicator allows the program to bypass processing in a test environment, preventing unintended data changes, which is a common practice in legacy systems to separate test and production workflows.

This RPG program is a critical front-end component of the 1099 creation process, ensuring data integrity through rigorous input validation before the main file processing occurs in the OCL-driven workflow.