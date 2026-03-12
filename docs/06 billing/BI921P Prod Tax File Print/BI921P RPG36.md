The provided document, `BI921P.rpg36.txt`, is an RPG (Report Program Generator) program written for an IBM System/36 or AS/400 environment. This program, `BI921P`, is called from the OCL program `BI921P.ocl36.txt` (previously analyzed) and is responsible for processing a "Product Tax Master List." Below, I’ll explain the process steps, business rules, tables/files used, and any external programs called, based on the RPG source code.

### Process Steps of the RPG Program

The RPG program `BI921P` handles user input for company selection and job queue settings, validates the input, and interacts with a file to prepare data for the product tax master list. Here’s a detailed breakdown of the process steps, derived from the RPG code:

1. **Program Initialization (Header and File Definitions)**:
   - **Line 0002 (H-Spec)**: The header specification (`H    P064`) defines program attributes, such as `P064` for program identification and `B` for batch mode. The program name is `BI921P`.
   - **Line 0004-0005 (F-Specs)**:
     - `SCREEN`: Defines a workstation file (`WORKSTN`) for interactive screen input/output, with a record length of 500 bytes, used for user prompts.
     - `BICONT`: Defines a disk file (`DISK`) with input (`I`) and update (`F`) capabilities, 256 bytes per record, indexed (`L`), with 2 access paths (`2AI`) and 2 keys (`2`).
   - **Line 0006-0007 (E-Specs)**:
     - `COM`: An array of 5 elements, each 40 bytes, used to store error messages (defined in the `COM` table at the end).
     - `DCO`: An array of 10 elements, each 35 bytes, used to store company codes and names retrieved from `BICONT`.
   - **Line 0008-0027 (I-Specs)**:
     - Defines input formats for `SCREEN` (format `BI921PS1`) and `BICONT`.
     - `SCREEN` input fields: `KYALCO` (company selection, e.g., 'ALL' or 'CO'), `KYCO1`, `KYCO2`, `KYCO3` (company numbers), `KYJOBQ` (job queue flag, 'Y' or 'N'), `KYCOPY` (number of copies), and `KYCANC` (cancel flag).
     - `BICONT` input fields: `BCDEL` (delete flag), `BCCO` (company code), `BCNAME` (company name).
     - `UDS` (User Data Structure) maps fields from positions 111-134, mirroring screen input fields for program control.

2. **Main Program Logic (C-Specs)**:
   - **Line 0030-0034**: Initializes variables:
     - Clears `MSG` (error message) to blanks.
     - Turns off indicators 51, 52, 53, 54, 81, and 90, which control screen display and error conditions.
   - **Line 0035-0037**: For key group `KG` (likely a control condition), sets `KYCANC` to 'CANCEL' and turns off indicators 01 and 09, preparing for potential job cancellation.
   - **Line 0038-0039**: If indicator 09 is on (initial call or specific condition), executes the `ONETIM` subroutine to initialize data.
   - **Line 0041**: If indicator 01 is on (screen input received), executes the `S1` subroutine to validate input and process data.
   - **Line 0043**: Defines an `END` tag, marking the end of the main program logic.

3. **ONETIM Subroutine (Lines 0046-0066)**:
   - **Purpose**: Initializes data by reading the `BICONT` file and populating the `DCO` array with company codes and names.
   - **Steps**:
     - Clears `DCO` array to blanks (`MOVEL*BLANKS DCO`).
     - Initializes counter `X` to 1 and `BCLIM` (limit) to 00.
     - Sets the file cursor to the beginning of `BICONT` using `BCLIM` (`SETLLBICONT`).
     - Reads `BICONT` records in a loop (`READ BICONT`):
       - If end-of-file (indicator 10), branches to `ENDONE`.
       - If `BCDEL` (delete flag) is 'D' (indicating a deleted record), skips to the next record (`AGNONE`).
       - Moves `BCCO` (company code) and `BCNAME` (company name) to `DCO` array at index `X`.
       - Increments `X` and checks if `X` reaches 10; if not, continues reading (`GOTO AGNONE`).
     - Sets defaults: `KYALCO` to 'ALL', `KYJOBQ` to 'N', and `KYCOPY` to 01.
     - Ends the subroutine (`ENDSR`).

4. **S1 Subroutine (Lines 0068-0127)**:
   - **Purpose**: Validates user input from the `SCREEN` file and checks company numbers against `BICONT`.
   - **Steps**:
     - **Validate Company Selection (Lines 0072-0088)**:
       - Checks if `KYALCO` is 'ALL' or 'CO':
         - If neither, sets indicators 81 and 90, moves error message `COM,1` ("SECOND ENTRY MUST BE 'CO' OR 'ALL'") to `MSG`, and exits (`GOTO ENDS1`).
       - If `KYALCO` is 'CO', checks if `KYCO1`, `KYCO2`, and `KYCO3` are all 00 (no companies entered):
         - If true, sets indicators 81, 90, and 51, moves error message `COM,2` ("IF CO, THEN ENTER VALID COMPANIES") to `MSG`, and exits.
       - If `KYALCO` is 'ALL', checks if any of `KYCO1`, `KYCO2`, or `KYCO3` are non-zero:
         - If true, sets indicators 81, 90, and 51, moves error message `COM,3` ("IF ALL, THEN DO NOT ENTER COMPANIES") to `MSG`, and exits.
     - **Validate Company Numbers (Lines 0093-0114)**:
       - If `KYALCO` is 'CO' (indicator 10), validates each non-zero company number (`KYCO1`, `KYCO2`, `KYCO3`):
         - Uses `CHAIN` to search `BICONT` for the company number.
         - If not found (indicator 10), sets error indicators (81, 90, 51/52/53), moves error message `COM,4` ("INVALID COMPANY NUMBER") to `MSG`, and exits.
     - **Validate Job Queue Selection (Lines 0117-0123)**:
       - Checks if `KYJOBQ` is 'Y', 'N', or blank:
         - If invalid, sets indicators 81, 90, and 54, moves error message `COM,5` ("JOB QUEUE ENTRY MUST BE 'Y' OR 'N'") to `MSG`, and exits.
     - **Validate Number of Copies (Lines 0125-0126)**:
       - If `KYCOPY` is 00, sets it to 01 (default minimum).
     - Ends the subroutine (`ENDS1`).

5. **Output to Screen (Lines 0130-0139)**:
   - Defines the output format for `SCREEN` when indicator 81 is on (error or display condition).
   - Writes to format `BI921PFM` with fields: `KYALCO`, `KYCO1`, `KYCO2`, `KYCO3`, `DCO` (company data), `KYJOBQ`, `KYCOPY`, and `MSG` (error message).
   - Displays the screen with user inputs and any error messages for correction.

### Business Rules

The RPG program enforces the following business rules, as defined in the validation logic and the `COM` table at the end of the code:

1. **Company Selection Rule**:
   - The `KYALCO` field must be either 'ALL' (process all companies) or 'CO' (process specific companies).
   - **Error Message (COM,1)**: "SECOND ENTRY MUST BE 'CO' OR 'ALL'" is displayed if `KYALCO` is neither.

2. **Company Number Input Rules**:
   - If `KYALCO` is 'CO', at least one valid company number (`KYCO1`, `KYCO2`, or `KYCO3`) must be entered (non-zero).
     - **Error Message (COM,2)**: "IF CO, THEN ENTER VALID COMPANIES" is displayed if all company numbers are 00.
   - If `KYALCO` is 'ALL', no company numbers should be entered (all must be 00).
     - **Error Message (COM,3)**: "IF ALL, THEN DO NOT ENTER COMPANIES" is displayed if any company number is non-zero.

3. **Company Number Validation**:
   - Each non-zero company number (`KYCO1`, `KYCO2`, `KYCO3`) must exist in the `BICONT` file.
     - **Error Message (COM,4)**: "INVALID COMPANY NUMBER" is displayed if a company number is not found in `BICONT`.

4. **Job Queue Selection**:
   - The `KYJOBQ` field must be 'Y' (submit to job queue) or 'N' (run interactively).
     - **Error Message (COM,5)**: "JOB QUEUE ENTRY MUST BE 'Y' OR 'N'" is displayed if `KYJOBQ` is invalid (e.g., blank or another value).

5. **Number of Copies**:
   - The `KYCOPY` field (number of copies for the report) must be at least 01. If 00 is entered, it is automatically set to 01.

### Tables/Files Used

The RPG program uses the following files:
1. **SCREEN**:
   - Type: Workstation file (`WORKSTN`).
   - Purpose: Handles interactive input/output for user prompts and error messages.
   - Format: `BI921PS1` (input) and `BI921PFM` (output).
   - Fields: `KYALCO`, `KYCO1`, `KYCO2`, `KYCO3`, `KYJOBQ`, `KYCOPY`, `KYCANC`, `DCO` (output), `MSG` (output).

2. **BICONT**:
   - Type: Disk file (`DISK`), input and update capable, indexed with 2 keys.
   - Purpose: Stores company data (codes and names) used for validation and populating the `DCO` array.
   - Fields: `BCDEL` (delete flag), `BCCO` (company code), `BCNAME` (company name).

3. **COM** (Array/Table):
   - Type: Program-defined array (5 elements, 40 bytes each).
   - Purpose: Stores error messages for display on the screen.
   - Content (defined at the end):
     - `COM,1`: "SECOND ENTRY MUST BE 'CO' OR 'ALL'"
     - `COM,2`: "IF CO, THEN ENTER VALID COMPANIES"
     - `COM,3`: "IF ALL, THEN DO NOT ENTER COMPANIES"
     - `COM,4`: "INVALID COMPANY NUMBER"
     - `COM,5`: "JOB QUEUE ENTRY MUST BE 'Y' OR 'N'"

### External Programs Called

The RPG program does not explicitly call any external programs (no `CALL` operations are present in the C-Specs). However:
- The OCL program (`BI921P.ocl36.txt`) references a program named `BI921`, which is likely the compiled version of this RPG program (`BI921P`). The OCL may queue or execute `BI921` based on the `KYJOBQ` value set by this program.
- No other external programs are called within the RPG code itself.

### Summary

- **Process Steps**: The program initializes by clearing variables and reading the `BICONT` file to populate company data (`ONETIM` subroutine). It then validates user input from the `SCREEN` file (`S1` subroutine), checking company selection (`KYALCO`), company numbers (`KYCO1`, `KYCO2`, `KYCO3`), job queue flag (`KYJOBQ`), and number of copies (`KYCOPY`). Invalid inputs trigger error messages displayed on the screen. The program prepares data for the product tax master list but does not generate the report itself (likely handled by `BI921` in a subsequent step).
- **Business Rules**:
  - `KYALCO` must be 'ALL' or 'CO'.
  - If 'CO', valid company numbers must be entered; if 'ALL', no company numbers should be entered.
  - Company numbers must exist in `BICONT`.
  - `KYJOBQ` must be 'Y' or 'N'.
  - `KYCOPY` defaults to 01 if 00.
- **Tables/Files Used**: `SCREEN` (workstation file), `BICONT` (disk file), `COM` (error message array).
- **External Programs Called**: None directly in the RPG code; `BI921` is referenced in the OCL, likely the compiled RPG program.

If you have additional details (e.g., the logic of `BI921` or related programs, or specific data in `BICONT`), I can provide further analysis. Let me know if you’d like me to search for related information or clarify specific aspects!