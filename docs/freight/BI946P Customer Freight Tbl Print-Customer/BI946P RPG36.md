The provided document is an **RPG III program** named `BI946P.rpg36.txt`, which is called from the OCL program `BI946P.ocl36.txt`. This RPG program appears to handle the prompting and validation for generating a customer freight tables print report. Below, I’ll explain the process steps, business rules, tables/files used, and any external programs called.

---

### Process Steps of the RPG Program

The RPG program `BI946P` is designed to prompt the user for input parameters (via a display file) and validate those inputs to prepare for generating a freight table report. It processes data from the `BICONT` file and outputs validated parameters to a display file (`SCREEN`). Here’s a step-by-step breakdown of the process:

1. **File and Data Structure Definitions** (Lines 0005–0032):
   - **Files**:
     - `SCREEN`: A workstation file (display file) with a record length of 512 bytes, used for user interaction (input/output prompts).
     - `BICONT`: A disk file (256 bytes, keyed, indexed) opened for input with two access paths (logical files or indexes, `L02AI`).
   - **Data Structures**:
     - `DCO`: An array of 3 elements, each 35 bytes, used to store company data (company number and name).
     - `CS`: An array of 10 elements, each 6 bytes, likely for customer numbers.
     - `COM`: An array of 7 elements, each 40 bytes, used for error messages (defined at the end of the program).
   - **Input Specifications**:
     - `SCREEN` (record format `NS`, indicators `01`, `09`):
       - Fields like `KYALCO` (ALL or CO), `KYCO1`, `KYCO2`, `KYCO3` (company numbers), `KYALCS` (ALL or SEL), `CS` (customer numbers), `KYJOBQ` (job queue flag), `KYCOPY` (number of copies), and `KYCUR` (current records flag).
     - `BICONT`:
       - Fields include `BCDEL` (delete flag), `BCCO` (company number), `BCNAME` (company name).
     - `UDS` (User Data Structure): Maps fields from `SCREEN` input to program variables (e.g., `KYALCO`, `KYCO1`, etc.).
     - `Y2KCEN` and `Y2KCMP`: Century and company fields for Y2K compliance (likely for date handling).

2. **Initialization** (Lines 0035–0043):
   - Clears indicators (31–35, 38–39, 81–90) to reset program state.
   - Clears the `MSG` field (40 bytes) for error messages.
   - If the `KG` indicator (likely a kill/go flag) is on:
     - Sets `U1` (user indicator) and `LR` (last record) to terminate the program.
     - Clears indicators `01`, `09`, `81` and jumps to the `END` tag, effectively halting execution.

3. **One-Time Setup Subroutine (`ONETIM`)** (Lines 0127–0165, triggered by indicator `09`):
   - Clears the `DCO` array.
   - Initializes variables: `X` (index) to 1, `BILIM` (limit) to 00.
   - Sets the lower limit (`SETLL`) on `BICONT` using `BILIM` to position the file pointer.
   - Reads `BICONT` records in a loop (`AGNCO` to `ENDCO`):
     - Skips records where `BCDEL = 'D'` (deleted records).
     - Moves `BCCO` (company number) and `BCNAME` (company name) to the `DCO` array at index `X`.
     - Increments `X` and continues until `X > 3` or end-of-file (`20` indicator).
   - Initializes default values:
     - `KYALCO = 'ALL'`, `KYCO1/2/3 = 0`, `KYALCS = 'ALL'`, `CS = 0`, `KYJOBQ = 'N'`, `KYCOPY = 1`.
   - Sets indicator `81` to display the screen.
   - Ends the subroutine and jumps to `END` (program termination).

4. **Main Validation Subroutine (`S1`)** (Lines 0055–0123, triggered by indicator `01`):
   - Validates user input from the `SCREEN` file.
   - **KYALCO Validation** (Lines 0057–0062):
     - Checks if `KYALCO` is `'ALL'` or `'CO'`. If true, sets indicator `31`.
     - If `31` is on, sets indicators `81` and `90`, moves error message `COM,1` (“ENTER ALL OR CO”) to `MSG`, and jumps to `ENDS1`.
   - **Company Number Validation** (Lines 0063–0096):
     - If `KYALCO = 'CO'`:
       - Checks if `KYCO1`, `KYCO2`, `KYCO3` are all zero. If so, sets indicators `32`, `81`, `90`, moves error message `COM,5` (“ENTER COMPANY NUMBER”) to `MSG`, and jumps to `ENDS1`.
       - For each non-zero `KYCO1`, `KYCO2`, `KYCO3`:
         - Chains (looks up) the company number in `BICONT`.
         - If not found (`32`, `33`, `34`) or `BCDEL = 'D'`, sets indicators `81`, `90`, moves error message `COM,2` (“INVALID COMPANY NUMBER ENTERED”) to `MSG`, and jumps to `ENDS1`.
   - **KYALCS Validation** (Lines 0098–0110):
     - Checks if `KYALCS` is `'ALL'` or `'SEL'`. If true, sets indicator `35`.
     - If `35` is on, sets indicators `81`, `90`, moves error message `COM,3` (“ENTER ALL OR SEL”) to `MSG`, and jumps to `ENDS1`.
     - If `KYALCS = 'SEL'`:
       - Sums the `CS` array (customer numbers) into `CSXFT`.
       - If `CSXFT = 0`, sets indicators `39`, `81`, `90`, moves error message `COM,6` (“ENTER CUSTOMER NUMBER”) to `MSG`, and jumps to `ENDS1`.
   - **KYCUR Validation** (Lines JB01, after 0110):
     - Checks if `KYCUR` is blank or `'CUR'`. If true, sets indicator `37`.
     - If `37` is on, sets indicators `81`, `90`, moves error message `COM,7` (“ENTER 'CUR' OR LEAVE BLANK”) to `MSG`, and jumps to `ENDS1`.
   - **KYJOBQ Validation** (Lines 0112–0117):
     - Checks if `KYJOBQ` is blank, `'N'`, or `'Y'`. If true, sets indicator `38`.
     - If `38` is on, sets indicators `81`, `90`, moves error message `COM,4` (“ENTER BLANK, N OR Y”) to `MSG`, and jumps to `ENDS1`.
   - **KYCOPY Validation** (Lines 0119–0121):
     - If `KYCOPY = 0`, sets it to `1` (ensures at least one copy is printed).
   - Ends the subroutine (`ENDS1`).

5. **Output to Screen** (Lines 0168–0179):
   - If indicator `81` is on, writes to the `SCREEN` file (format `D`).
   - Outputs fields: `KYALCO`, `KYCO1`, `KYCO2`, `KYCO3`, `DCO` (company data), `KYALCS`, `CS`, `KYJOBQ`, `KYCOPY`, `MSG`, and `KYCUR`.
   - Includes a constant `'BI946PFM'` (likely a program or form name).

6. **Program Termination** (Lines 0051, 0046, 0049):
   - The program jumps to the `END` tag after executing `ONETIM` (indicator `09`) or `S1` (indicator `01`), or if `KG` is on.
   - Sets `LR` (last record) to terminate the program.

---

### Business Rules

The RPG program enforces the following business rules for validating user input to generate a freight table report:

1. **Company Selection (`KYALCO`)**:
   - Must be `'ALL'` or `'CO'`. Otherwise, displays error: “ENTER ALL OR CO”.
   - If `'CO'`, at least one company number (`KYCO1`, `KYCO2`, or `KYCO3`) must be non-zero and valid (exists in `BICONT` and not deleted).

2. **Company Numbers (`KYCO1`, `KYCO2`, `KYCO3`)**:
   - If `KYALCO = 'CO'`, company numbers must exist in `BICONT` and not have `BCDEL = 'D'`.
   - If any company number is invalid or deleted, displays error: “INVALID COMPANY NUMBER ENTERED”.
   - If all company numbers are zero when `KYALCO = 'CO'`, displays error: “ENTER COMPANY NUMBER”.

3. **Customer Selection (`KYALCS`)**:
   - Must be `'ALL'` or `'SEL'`. Otherwise, displays error: “ENTER ALL OR SEL”.
   - If `'SEL'`, at least one customer number in the `CS` array must be non-zero. Otherwise, displays error: “ENTER CUSTOMER NUMBER”.

4. **Current Records Flag (`KYCUR`)**:
   - Must be blank or `'CUR'`. Otherwise, displays error: “ENTER 'CUR' OR LEAVE BLANK”.
   - `'CUR'` likely filters for current (non-expired) records, as added in the 2017 update (JB01).

5. **Job Queue Flag (`KYJOBQ`)**:
   - Must be blank, `'N'`, or `'Y'`. Otherwise, displays error: “ENTER BLANK, N OR Y”.
   - Determines whether the report job is submitted to a job queue (`'Y'`) or run interactively (`'N'` or blank).

6. **Number of Copies (`KYCOPY`)**:
   - If zero, automatically set to `1` to ensure at least one copy is printed.

7. **Deleted Records**:
   - Records in `BICONT` with `BCDEL = 'D'` are skipped during processing.

8. **Data Collection**:
   - The program collects up to three company numbers and names from `BICONT` into the `DCO` array for display or validation.

---

### Tables/Files Used

1. **BICONT**:
   - A disk file (256 bytes, keyed, input mode) containing freight table data.
   - Fields:
     - `BCDEL` (1 byte): Delete flag (‘D’ indicates deleted).
     - `BCCO` (2–3 bytes): Company number.
     - `BCNAME` (30 bytes): Company name.
   - Used for validating company numbers and retrieving company names.

2. **SCREEN**:
   - A workstation file (display file, 512 bytes) used for user input and output.
   - Fields include `KYALCO`, `KYCO1`, `KYCO2`, `KYCO3`, `KYALCS`, `CS`, `KYJOBQ`, `KYCOPY`, `KYCUR`, `MSG`, and `DCO`.
   - Outputs error messages and prompts to the user.

---

### External Programs Called

The RPG program does not explicitly call any external programs via `CALL` operations. However:
- The output specification includes a constant `'BI946PFM'` (line 0169), which may indicate a related program or form name used in the broader process (possibly called by the OCL program).
- The OCL program (`BI946P.ocl36.txt`) references `BI946`, which is likely called after `BI946P` validates the input parameters, suggesting `BI946` is the program that generates the actual freight table report.

---

### Summary

- **Process Steps**: The program initializes, prompts the user via `SCREEN`, validates input parameters (`KYALCO`, `KYCO1/2/3`, `KYALCS`, `CS`, `KYJOBQ`, `KYCOPY`, `KYCUR`), retrieves company data from `BICONT`, and displays errors or validated data. It terminates after validation or if the `KG` indicator is set.
- **Business Rules**: Enforces valid values for company selection, company numbers, customer selection, current records flag, job queue flag, and number of copies. Skips deleted records and ensures at least one copy is printed.
- **Tables/Files**: `BICONT` (disk file for company data), `SCREEN` (display file for user interaction).
- **External Programs**: None called directly, but `'BI946PFM'` and the OCL’s reference to `BI946` suggest a related program for report generation.

If you have the source code for `BI946` or additional details about `BICONT` or `SCREEN`, I can provide further analysis of the end-to-end process.