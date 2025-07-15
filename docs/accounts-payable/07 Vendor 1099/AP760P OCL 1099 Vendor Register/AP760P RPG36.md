The provided `AP760P.rpg36.txt` is an RPG/36 program (likely running on an IBM System/36 or AS/400 environment) that serves as the main program for generating a Vendor 1099 Register, called from the `AP760P.ocl36.txt` OCL program previously discussed. This program handles user input validation and setup for the 1099 report generation process. Below, I’ll explain the process steps, business rules, tables/files used, and any external programs called, based on the RPG code provided.

---

### Process Steps of the RPG/36 Program (AP760P)

The `AP760P` program is responsible for prompting the user for parameters (via a workstation screen) and validating inputs before passing them to another program (likely `AP760`) for generating the Vendor 1099 Register. Here’s a step-by-step breakdown of the process:

1. **Program Initialization**:
   - The program defines a workstation file (`SCREEN`) for user interaction, and two disk files: `APCONT` (accounts payable control file) and `GSTABL` (general system table).
   - Arrays are defined: `MSG` (10 error messages, 40 characters each) and `DCO` (5 company numbers/names, 35 characters each).
   - The program clears indicators (50–58, 81–90) and initializes the `MSG40` field to blanks to reset any previous error messages (lines 0047–0051).
   - If the `KG` (cancel) condition is set, the program sets `KYCANC` to `'CANCEL'` and turns off indicators 01 and 09, likely preparing to exit (lines 0053–0054).

2. **One-Time Setup (ONETIM Subroutine)**:
   - Executed if indicator 09 is on (line 0056), typically on the first run.
   - **Purpose**: Populates the `DCO` array with company numbers and names from the `APCONT` file for display on the screen.
   - **Steps**:
     - Initializes counter `X` to 1 and limit `ACLIM` to 0 (lines 0066–0067).
     - Positions the file pointer to the start of `APCONT` using `SETLL` (line 0068).
     - Reads records from `APCONT` in a loop (lines 0069–0079):
       - Skips records marked as deleted (`ACRCCD = 'D'`) (lines 0073–0074).
       - Moves the company number (`ACCONO`) and name (`ACNAME`) into the `DCO` array at index `X` (lines 0075–0076).
       - Increments `X` and continues until 5 companies are loaded or the end of the file is reached (indicator 79) (lines 0077–0079).
     - Sets default values for screen fields (lines 0084–0089):
       - `KYALCO = 'ALL'` (select all companies by default).
       - `KYALTY = 'TYP'` (select specific 1099 types by default).
       - `KYCRLS = 'C'` (current year data).
       - `KYJOBQ = 'N'` (do not submit to job queue).
       - `KYCOPY = 01` (one copy of the report).
       - Sets indicator 81 to display the screen.

3. **Screen Processing and Validation (S1 Subroutine)**:
   - Executed if indicator 01 is on (line 0058), handling user input validation from the screen (`SCREEN`, format `AP760PS1`).
   - **Purpose**: Validates user inputs for company selection, 1099 types, current/last year, job queue option, and number of copies.
   - **Steps**:
     - **Validate Company Selection** (lines 0096–0137):
       - Checks if `KYALCO` is `'ALL'` or `'CO'` (lines 0096–0097).
       - If neither, sets error indicators (81, 90, 50), displays error message 1 (“ENTRY MUST BE 'CO' OR 'ALL'”), and exits to `ENDS1` (lines 0098–0100).
       - If `KYALCO = 'CO'` and company numbers (`KYCO1`, `KYCO2`, `KYCO3`) are all zero, or if `KYALCO = 'ALL'` and any company number is non-zero, sets error indicators (81, 90, 51) and displays error message 2 or 3 (lines 0102–0112).
       - If `KYALCO = 'CO'`, validates each company number (`KYCO1`, `KYCO2`, `KYCO3`) by chaining to `APCONT` (lines 0116–0135):
         - If a company number is invalid (not found in `APCONT`), sets error indicators (81, 90, 51–53) and displays error message 4 (“INVALID COMPANY NUMBER”).
     - **Validate 1099 Type Selection** (lines 0142–0188):
       - Checks if `KYALTY` is `'ALL'` or `'TYP'` (lines 0142–0143).
       - If neither, sets error indicators (81, 90, 54), displays error message 6 (“ENTRY MUST BE 'TYP' OR 'ALL'”), and exits (lines 0144–0146).
       - If `KYALTY = 'TYP'` and types (`KYTY1`, `KYTY2`, `KYTY3`) are blank, or if `KYALTY = 'ALL'` and any type is non-blank, sets error indicators (81, 90, 55) and displays error message 7 or 8 (lines 0148–0158).
       - If `KYALTY = 'TYP'`, validates each type (`KYTY1`, `KYTY2`, `KYTY3`) by chaining to `GSTABL` with key `AP1099` concatenated with the type (lines 0162–0186):
         - If a type is invalid (not found in `GSTABL`), sets error indicators (81, 90, 55–57) and displays error message 9 (“INVALID 1099 TYPE”).
     - **Validate Current/Last Year Selection** (lines 0192–0197):
       - Checks if `KYCRLS` is `'C'` (current year) or `'L'` (last year) (lines 0192–0193).
       - If neither, sets error indicators (81, 90, 58), displays error message 10 (“CURR/LAST ENTRY MUST BE 'C' OR 'L'”), and exits (lines 0194–0196).
       - Sets `KYCYYR` (year for 1099s):
         - If `KYCRLS = 'C'`, sets `KYCYYR` to the current year (`UYEAR`) (lines 0197).
         - If `KYCRLS = 'L'`, sets `KYCYYR` to the previous year (`UYEAR - 1`) (lines 0197).
     - **Validate Job Queue Selection** (lines 0199–0204):
       - Checks if `KYJOBQ` is `'Y'`, `'N'`, or blank (lines 0199–0201).
       - If invalid, sets error indicators (81, 90, 59), displays error message 5 (“JOB QUEUE ENTRY MUST BE 'Y' OR 'N'”), and exits (lines 0202–0204).
     - **Validate Number of Copies** (lines 0206–0207):
       - If `KYCOPY` is zero, sets it to 1 (lines 0206–0207).
     - **Output to Screen** (lines 0211–0225):
       - If indicator 81 is on, displays the `AP760PS1` screen format with fields (`KYALCO`, `KYCO1–3`, `DCO`, `KYALTY`, `KYTY1–3`, `KYCRLS`, `KYJOBQ`, `KYCOPY`, `MSG40`).

4. **Program Termination**:
   - The program loops back to display the screen if validation fails (indicator 81 on), allowing the user to correct inputs.
   - If validation succeeds, the program sets the necessary parameters in the User Data Structure (UDS) fields (`KYALCO`, `KYCO1–3`, `KYJOBQ`, `KYCOPY`, `KYCANC`, `KYALTY`, `KYTY1–3`, `KYCRLS`, `KYCYYR`) for use by the calling OCL or subsequent program (`AP760`).

---

### Business Rules

The program enforces the following business rules for generating the 1099 Register:

1. **Company Selection**:
   - The user must select either `'ALL'` (all companies) or `'CO'` (specific companies) in `KYALCO`.
   - If `'CO'` is selected, at least one valid company number (`KYCO1`, `KYCO2`, or `KYCO3`) must be provided, and each must exist in the `APCONT` file.
   - If `'ALL'` is selected, no company numbers should be specified.
   - Invalid company selections trigger error messages (1–4).

2. **1099 Type Selection**:
   - The user must select either `'ALL'` (all 1099 types) or `'TYP'` (specific types) in `KYALTY`.
   - If `'TYP'` is selected, at least one valid 1099 type (`KYTY1`, `KYTY2`, or `KYTY3`) must be provided, and each must exist in the `GSTABL` file under the `AP1099` key.
   - If `'ALL'` is selected, no specific types should be specified.
   - Invalid type selections trigger error messages (6–9).

3. **Current/Last Year Selection**:
   - The user must select `'C'` (current year) or `'L'` (last year) in `KYCRLS`.
   - The year (`KYCYYR`) is set to the current year (`UYEAR`) for `'C'` or the previous year (`UYEAR - 1`) for `'L'`.
   - Invalid selections trigger error message 10.

4. **Job Queue Option**:
   - The user must specify `'Y'` (submit to job queue) or `'N'` (run interactively) in `KYJOBQ`.
   - Invalid selections trigger error message 5.

5. **Number of Copies**:
   - The number of report copies (`KYCOPY`) must be non-zero; if zero, it defaults to 1.

6. **Error Handling**:
   - If any validation fails, the program displays an error message on the screen and waits for corrected input.
   - The user can cancel the process (setting `KYCANC = 'CANCEL'`), which likely terminates the program or signals the OCL to exit.

---

### Tables/Files Used

The program uses the following files:
1. **SCREEN**:
   - Type: Workstation file (`WORKSTN`), 512 bytes.
   - Used for user interaction via the `AP760PS1` screen format to collect and display input parameters and error messages.
2. **APCONT**:
   - Type: Indexed file (`IF`), 256 bytes, with a 2-byte key, shared access (`DISP-SHR`).
   - Contains accounts payable control data, including:
     - `ACRCCD` (1 byte): Delete code (‘D’ for deleted records).
     - `ACCONO` (2 bytes): Company number.
     - `ACNAME` (30 bytes): Company name.
   - Used to validate company numbers and populate the `DCO` array for display.
3. **GSTABL**:
   - Type: Indexed file (`IC`), 256 bytes, with a 12-byte key, shared access (`DISP-SHR`).
   - Contains general system tables, specifically 1099 type codes under the `AP1099` key.
   - Used to validate 1099 types (`KYTY1`, `KYTY2`, `KYTY3`).

---

### External Programs Called

The RPG program itself does not explicitly call external programs. However:
- It is called from the `AP760P.ocl36.txt` OCL program, as indicated in the OCL’s `// LOAD AP760P` and `// RUN` commands.
- The validated parameters are passed to the `AP760` program (via the OCL’s `JOBQ` or direct execution), which likely performs the actual 1099 Register generation.
- The `GSY2K` utility (mentioned in the OCL) may be indirectly involved for date handling, but it is not called within the RPG code.

---

### Summary

The `AP760P` RPG/36 program is a front-end validation program that:
1. Displays a screen (`AP760PS1`) to collect user inputs for company selection (`KYALCO`, `KYCO1–3`), 1099 types (`KYALTY`, `KYTY1–3`), year (`KYCRLS`), job queue option (`KYJOBQ`), and number of copies (`KYCOPY`).
2. Validates inputs against the `APCONT` (company data) and `GSTABL` (1099 types) files, displaying error messages if invalid.
3. Sets up parameters in the User Data Structure (UDS) for the year (`KYCYYR`) and other options, which are passed to the `AP760` program via the OCL for report generation.
4. Enforces business rules to ensure valid company numbers, 1099 types, year selection, job queue option, and copy count.

**Tables/Files Used**:
- `SCREEN` (workstation file for user interaction)
- `APCONT` (accounts payable control file for company data)
- `GSTABL` (general system table for 1099 types)

**External Programs Called**:
- None directly called within the RPG code, but `AP760` is invoked via the OCL, and `AP760P` is called by the OCL program `AP760P.ocl36.txt`.

If you need further details, such as a deeper analysis of the screen format or the `AP760` program’s expected behavior, let me know!