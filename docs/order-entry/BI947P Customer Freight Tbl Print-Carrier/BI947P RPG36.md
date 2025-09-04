The provided document is an RPG (Report Program Generator) program, `BI947P.rpg36.txt`, written for an IBM System/36 or AS/400 environment. This program, called from the OCL script `BI947P.ocl36.txt`, handles the processing of a "Customer Freight Table Master File List by Carrier ID." Below, I’ll explain the process steps, business rules, tables used, and any external programs called.

---

### Process Steps of the RPG Program

The RPG program `BI947P` is designed to prompt the user for input via a workstation screen and validate selections for generating a report or processing data from the `BICONT` file. Here’s a detailed breakdown of the process steps:

1. **File and Data Structure Definitions**:
   - **Files**:
     - `SCREEN` (Line 0004): A workstation file (display file) with a record length of 512 bytes, used for user input/output.
     - `BICONT` (Line 0005): A disk file with a record length of 256 bytes, indexed (L02AI), used as the customer freight table master file.
   - **Data Structures**:
     - `DCO` (Line 0006): An array of 3 elements, each 35 bytes, likely used to store company numbers and names.
     - `CS` (Line 0007): An array of 10 elements, each 6 bytes, likely for carrier IDs.
     - `COM` (Line 0007, JB01): An array of 7 elements, each 40 bytes, used for error messages (e.g., "INVALID COMPANY NUMBER ENTERED").
   - **Input Specifications**:
     - `SCREEN` input (Lines 0009–0017, JB01): Defines fields for user input, including:
       - `KYALCO` (ALL or CO selection, positions 3–5).
       - `KYCO1`, `KYCO2`, `KYCO3` (company numbers, positions 6–11).
       - `KYALCS` (ALL or SEL for carrier selection, positions 12–14 cognito
14).
       - `CS` (carrier IDs, positions 15–74).
       - `KYJOBQ` (job queue flag, Y/N, position 75性
75).
       - `KYCOPY` (copy count, positions 76–77).
       - `KYCUR` (current records only flag, CUR or blank, positions 78–80, added by JB01).
     - `BICONT` input (Lines 0019–0031): Defines fields for the freight table:
       - `BCDEL` (delete flag, D or not, position 1).
       - `BCCO` (company number, positions 2–3).
       - `BCNAME` (company name, positions 4–33).
       - Same fields as `SCREEN` for User Data Structure (UDS).

2. **Main Processing Logic**:
   - **Initialization (Lines 0034–0043)**:
     - Clears indicators (31–35, 38–39, 81–90) and sets `MSG` to blanks.
     - If the `KG` indicator is on (likely set by the OCL), sets `U1` and `LR`, clears indicators 01, 09, 81, and jumps to `END`.
   - **One-Time Processing (Subroutine ONETIM, Lines 0125–0163)**:
     - Triggered if indicator 09 is on (first screen interaction).
     - Clears `DCO` array and initializes variables (`X=1`, `BILIM=00`).
     - Sets the lower limit for `BICONT` file (`SETLL`) to start reading from the beginning.
     - Reads `BICONT` records in a loop (`AGNCO` to `ENDCO`):
       - Skips records where `BCDEL = 'D'` (deleted records).
       - Moves company number (`BCCO`) and name (`BCNAME`) to `DCO` array, incrementing `X` until `X > 3` or end of file.
     - Initializes screen fields: `KYALCO='ALL'`, `KYCO1–3=0`, `KYALCS='ALL'`, `CS=blanks`, `KYJOBQ='N'`, `KYCOPY=1`.
     - Sets indicator 81 to display the screen.
   - **Screen Processing (Subroutine S1, Lines 0054–0121)**:
     - Triggered if indicator 01 is on (user input validation).
     - Validates user input:
       - **KYALCO**: Checks if `KYALCO` is 'ALL' or 'CO'. If valid, sets indicators 81, 90; otherwise, sets error message (COM,1) and jumps to `ENDS1`.
       - If `KYALCO = 'CO'`, validates company numbers (`KYCO1`, `KYCO2`, `KYCO3`):
         - If all are zero, sets error message (COM,5) and jumps to `ENDS1`.
         - For each non-zero company number, chains to `BICONT` to verify:
           - If not found or deleted (`BCDEL='D'`), sets error message (COM,2).
           - If zero, sets error message (COM,2).
       - **KYALCS**: Checks if `KYALCS` is 'ALL' or 'SEL'. If valid, sets indicators 81, 90; otherwise, sets error message (COM,3).
       - If `KYALCS = 'SEL'`, checks if `CS,1` (first carrier ID) is blank. If blank, sets error message (COM,6).
       - **KYJOBQ**: Validates as blank, 'N', or 'Y'. If invalid, sets error message (COM,4).
       - **KYCUR** (JB01): Validates as blank or 'CUR'. If invalid, sets error message (COM,7).
       - **KYCOPY**: If zero, sets to 1.
     - Jumps to `ENDS1` to end the subroutine.

3. **Output to Screen (Lines 0166–0177)**:
   - If indicator 81 is on, displays the `SCREEN` with:
     - Form `BI947PFM`.
     - Fields `KYALCO`, `KYCO1–3`, `DCO`, `KYALCS`, `CS`, `KYJOBQ`, `KYCOPY`, `MSG`, `KYCUR`.

4. **End of Program (Line 0050)**:
   - The `END` tag marks the program’s termination point.

---

### Business Rules

The program enforces the following business rules for user input validation:
1. **Company Selection (`KYALCO`)**:
   - Must be 'ALL' or 'CO'. If invalid, displays error message: "ENTER ALL OR CO".
   - If 'CO', at least one company number (`KYCO1`, `KYCO2`, or `KYCO3`) must be non-zero, valid (exists in `BICONT` and not deleted), or an error message is shown: "INVALID COMPANY NUMBER ENTERED" or "ENTER COMPANY NUMBER".
2. **Carrier Selection (`KYALCS`)**:
   - Must be 'ALL' or 'SEL'. If invalid, displays error message: "ENTER ALL OR SEL".
   - If 'SEL', at least one carrier ID (`CS,1`) must be non-blank, or an error message is shown: "ENTER CARRIER ID".
3. **Job Queue Flag (`KYJOBQ`)**:
   - Must be blank, 'N', or 'Y'. If invalid, displays error message: "ENTER BLANK, N OR Y".
4. **Current Records Flag (`KYCUR`, JB01)**:
   - Must be blank or 'CUR'. If invalid, displays error message: "ENTER 'CUR' OR LEAVE BLANK".
5. **Copy Count (`KYCOPY`)**:
   - If zero, automatically set to 1 to ensure at least one copy is requested.
6. **Deleted Records**:
   - Records in `BICONT` with `BCDEL='D'` are skipped during processing (in `ONETIM` and `S1` subroutines).
7. **Screen Interaction**:
   - The program displays the input screen (`SCREEN`) with form `BI947PFM` when indicator 81 is set, showing user inputs and any error messages.
   - The `ONETIM` subroutine populates the `DCO` array with up to three company numbers and names from `BICONT`, excluding deleted records.

---

### Tables/Files Used

- **BICONT**: The customer freight table master file, containing:
  - `BCDEL` (position 1): Delete flag ('D' for deleted).
  - `BCCO` (positions 2–3): Company number.
  - `BCNAME` (positions 4–33): Company name.
- **SCREEN**: A workstation file for user interaction, containing fields for:
  - `KYALCO`, `KYCO1–3`, `KYALCS`, `CS`, `KYJOBQ`, `KYCOPY`, `KYCUR`, `DCO`, `MSG`.

---

### External Programs Called

- **BI947**: Referenced in the OCL and RPG (e.g., Line 0167, form `BI947PFM`). This is likely the program that processes the validated input to generate the freight table list or report. It is called either directly or via the job queue in the OCL script, depending on `KYJOBQ`.

---

### Summary

- **Process Steps**:
  1. Define files (`SCREEN`, `BICONT`) and data structures (`DCO`, `CS`, `COM`).
  2. Initialize variables and indicators; skip to end if `KG` is set.
  3. Run `ONETIM` (if indicator 09) to read up to three non-deleted `BICONT` records into `DCO` and set default screen values.
  4. Run `S1` (if indicator 01) to validate user input for company selection (`KYALCO`, `KYCO1–3`), carrier selection (`KYALCS`, `CS`), job queue (`KYJOBQ`), current records (`KYCUR`), and copy count (`KYCOPY`).
  5. Display the screen with form `BI947PFM` and error messages if validation fails.
  6. End the program at the `END` tag.
- **Business Rules**:
  - Enforce valid values for `KYALCO` ('ALL' or 'CO'), `KYALCS` ('ALL' or 'SEL'), `KYJOBQ` (blank, 'N', 'Y'), and `KYCUR` (blank or 'CUR').
  - Validate company numbers against `BICONT`, skipping deleted records.
  - Ensure at least one copy (`KYCOPY`).
  - Display appropriate error messages for invalid inputs.
- **Tables/Files**: `BICONT` (freight table data), `SCREEN` (user interface).
- **External Programs**: `BI947` (likely the report-generating program).

This RPG program serves as an input validation front-end for the `BI947` program, ensuring user selections meet business requirements before proceeding. If you have additional details (e.g., the `BI947` program or `BI947PFM` form), I can provide further analysis!