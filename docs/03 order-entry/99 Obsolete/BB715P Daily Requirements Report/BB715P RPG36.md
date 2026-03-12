The provided `BB715P.rpg36.txt` is an RPG program (likely for IBM System/36 or AS/400) that serves as the core logic for generating a "Daily Requirements Report," as called by the `BB715P.ocl36.txt` OCL program. Below, I’ll explain the process steps, business rules, tables used, and external programs called, ensuring a clear and concise analysis based on the RPG code.

### Process Steps of the RPG Program

The RPG program `BB715P` handles input validation and processing for generating a daily requirements report. It uses a workstation file (`SCREEN`) for user interaction and a data file (`BICONT`) for validation checks. Here’s a step-by-step breakdown of the process:

1. **Program Initialization**:
   - **Header (H-spec)**: Line 0001 defines the program name (`BB715P`) and likely a page length (`P064`).
   - **File Definitions (F-spec)**:
     - `SCREEN`: A workstation file (likely a display file) with 500 bytes for user input/output (Line 0003).
     - `BICONT`: A disk file, 256 bytes, indexed (`IC`), with two keys (`2AI`), used for data validation (Line 0004).
   - **Array Definition (E-spec)**: `COM` is a 1x4 array of 40-character strings holding error messages (Line 0005, populated in lines 0305–0308).
   - **Indicator Initialization**: Lines 0029–0030 reset indicators 30, 35, 79, 81, and 90 to off, ensuring a clean state.
   - **Clear Message**: Line 0031 initializes the `MSG` field (40 characters) to blanks.

2. **One-Time Setup (ONETIM Subroutine)**:
   - Triggered if indicator 09 is on (Line 0033), typically on the first program execution.
   - **Steps** (Lines 0107–0116):
     - Sets `KYCO` to 10 (company number, Line 0109).
     - Sets `KYJOBQ` to 'N' (no job queue by default, Line 0110).
     - Sets `KYCOPY` to 1 (number of report copies, Line 0111).
     - Sets `KYSTDT` to the system date (`UDATE`, Line 0112).
     - Sets indicator 81 on (Line 0114) to trigger screen display.
     - Jumps to `END` tag (Line 0035), terminating the program after setup.

3. **Main Processing (S1 Subroutine)**:
   - Triggered if indicator 01 is on (Line 0044), indicating user input from the screen.
   - **Steps** (Lines 0051–0105):
     - **Company Validation**:
       - Uses `KYCO` to chain (lookup) the `BICONT` file (Line 0053).
       - If no record is found (`N30`, Line 0054) or the record’s `BCDEL` field equals 'D' (deleted, Line 0054), sets indicator 30, moves error message `COM,1` ("INVALID COMPANY NUMBER ENTERED") to `MSG`, sets indicators 81 and 90, and jumps to `ENDS1` (Lines 0055–0057).
     - **Month Validation**:
       - Checks if `KYMM` (month) is not 01–12 (Lines 0060–0061). If invalid, sets indicators 81, 90, and 35, moves error message `COM,3` ("INVALID DATE ENTERED") to `MSG`, and jumps to `ENDS1` (Lines 0062–0064).
     - **Day Validation**:
       - Checks if `KYDD` (day) is not 01–31 (Lines 0066–0067). If invalid, sets indicators 81, 90, and 35, moves error message `COM,3` to `MSG`, and jumps to `ENDS1` (Lines 0068–0070).
     - **Date Validation (@DTEDT Subroutine)**:
       - Called with `KYSTDT` (start date) moved to `MMDDYY` (Line 0073).
       - Validates the date format (MMDDYY) and checks for leap years (Lines 0118–0229). If invalid, sets indicator 79, moves error message `COM,3` to `MSG`, and jumps to `ENDS1` (Lines 0075–0077).
     - **Day-of-Week Check (@GTOJ Subroutine)**:
       - Converts `KYSTDT` to Julian format and determines the day of the week (`G$JW`, Lines 0080–0091).
       - If the date is a Saturday (`G$JW = 3`) or Sunday (`G$JW = 4`), sets indicators 81, 90, and 35, moves error message `COM,4` ("CANNOT START ON SATURDAY OR SUNDAY") to `MSG`, and jumps to `ENDS1` (Lines 0082–0090).
     - **Job Queue Validation**:
       - Checks if `KYJOBQ` is blank, 'N', or 'Y' (Lines 0094–0096). If not, sets indicators 81, 90, and 33, moves error message `COM,2` ("INVALID PARAMETER ENTERED - ' ', N OR Y") to `MSG`, and jumps to `ENDS1` (Lines 0097–0099).
     - **Copy Count Default**:
       - If `KYCOPY` is zero, sets it to 1 (Lines 0101–0103).
     - **End Subroutine**: `ENDS1` returns control (Line 0105).

4. **Error Handling (KG Condition)**:
   - If indicator `KG` is on (Line 0038), likely indicating a user cancel or error state:
     - Resets indicators 01, 09, and 81 (Line 0039).
     - Sets indicators U8 (user switch) and LR (last record) on (Line 0040).
     - Jumps to `END` tag (Line 0041), terminating the program.

5. **Screen Output**:
   - If indicator 81 is on (Line 0299), outputs to `SCREEN`:
     - Fields: `KYCO`, `KYSTDT`, `KYJOBQ`, `KYCOPY`, and `MSG` (Lines 0300–0305).
     - Displays the form `BB715PFM` (Line 0300), showing input fields and any error messages.

6. **Program Termination**:
   - The `END` tag (Line 0047) is reached after successful processing, errors, or one-time setup, ending the program cycle.
   - If errors occur (indicators 81, 90 set), the program displays the error on the screen and waits for user correction.

### Business Rules

The program enforces the following business rules for the Daily Requirements Report:
1. **Company Number Validation**:
   - The company number (`KYCO`) must exist in the `BICONT` file and not be marked as deleted (`BCDEL ≠ 'D'`).
   - Invalid company numbers trigger the error: "INVALID COMPANY NUMBER ENTERED."
2. **Date Validation**:
   - The start date (`KYSTDT`) must be a valid date (MMDDYY format).
   - The month (`KYMM`) must be 01–12.
   - The day (`KYDD`) must be valid for the month (e.g., 01–31 for most months, 01–28/29 for February, considering leap years).
   - Invalid dates trigger the error: "INVALID DATE ENTERED."
3. **Day-of-Week Restriction**:
   - The start date cannot be a Saturday or Sunday, as the report is likely for business days only.
   - Violation triggers the error: "CANNOT START ON SATURDAY OR SUNDAY."
4. **Job Queue Parameter**:
   - The job queue parameter (`KYJOBQ`) must be blank, 'N', or 'Y'.
   - Invalid values trigger the error: "INVALID PARAMETER ENTERED - ' ', N OR Y."
5. **Copy Count Default**:
   - If the number of copies (`KYCOPY`) is zero, it defaults to 1.
6. **Initial Setup**:
   - On first run (indicator 09), the program sets default values: company number = 10, job queue = 'N', copies = 1, and start date = system date.

### Tables Used

- **BICONT**: A disk file used for validating the company number (`KYCO`). It is indexed with two keys (likely `KYCO` and another field) and contains a `BCDEL` field to mark deleted records (Lines 0004, 0013, 0053–0054).
- **COM**: An array (1x4, 40 characters) storing error messages (Lines 0005, 0305–0308). It is not a database table but a predefined data structure:
  - `COM,1`: "INVALID COMPANY NUMBER ENTERED"
  - `COM,2`: "INVALID PARAMETER ENTERED - ' ', N OR Y"
  - `COM,3`: "INVALID DATE ENTERED"
  - `COM,4`: "CANNOT START ON SATURDAY OR SUNDAY"

### External Programs Called

- **None**: The RPG program does not explicitly call any external programs. It interacts with the `SCREEN` (display file `BB715PFM`) and `BICONT` file but does not invoke other RPG or system programs directly.

### Additional Notes

- **Subroutines**:
  - `ONETIM`: Handles one-time initialization of input parameters.
  - `S1`: Performs validation of user inputs (`KYCO`, `KYSTDT`, `KYJOBQ`, `KYCOPY`).
  - `@DTEDT`: Validates the date format and checks for leap years.
  - `@GTOJ`: Converts Gregorian dates to Julian format and determines the day of the week.
- **Indicators**:
  - `01`: Triggers main processing (`S1`).
  - `09`: Triggers one-time setup (`ONETIM`).
  - `30`: Indicates invalid company number or deleted record.
  - `33`: Indicates invalid `KYJOBQ` value.
  - `35`: Indicates invalid date (month, day, or weekend).
  - `79`: Indicates date format or leap year errors.
  - `81`: Triggers screen output.
  - `90`: Indicates an error condition (with `MSG` displayed).
  - `U8`, `LR`: Used for program termination.
  - `KG`: Indicates a cancel or error state.
- **Integration with OCL**:
  - The OCL program (`BB715P.ocl36.txt`) loads and runs `BB715P`, passing parameters like `?9?` (likely `KYCO` or `KYSTDT`) and checking `KYJOBQ` (position 109) to decide whether to queue or run `BB715`.
  - The RPG program validates inputs and displays errors, looping until valid inputs are provided or the user cancels.

### Summary

The `BB715P` RPG program validates user inputs for a Daily Requirements Report, ensuring a valid company number, date, job queue parameter, and number of copies. It enforces business rules to prevent reports from starting on weekends and ensures proper date formats. It uses the `BICONT` file for company validation and the `COM` array for error messages, with no external program calls. The program interacts with the user via the `SCREEN` file (`BB715PFM`) and terminates upon valid input or user cancellation.

If you need further details or analysis (e.g., specific field formats or additional context about `BICONT`), please let me know!