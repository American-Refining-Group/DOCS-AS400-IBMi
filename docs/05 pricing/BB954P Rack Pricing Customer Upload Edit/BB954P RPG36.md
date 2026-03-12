The provided document, `BB954P.rpg36.txt`, is an RPG (Report Program Generator) program for IBM System/3x or AS/400 systems, called by the OCL program `BB954P.ocl36.txt`. It appears to handle the validation and display of customer and pricing data as part of a rack pricing update process. Below, I’ll explain the process steps, business rules, tables (files) used, and any external programs called.

### Process Steps of the RPG Program

The RPG program `BB954P` is responsible for validating user input (e.g., company number, customer number, date, job queue, and password) and preparing customer data for display on a workstation screen. It also sets up defaults and handles error conditions. Here’s a detailed breakdown of the process steps:

1. **File and Data Structure Definitions**:
   - **Files**:
     - `SCREEN`: Workstation file (1024 bytes) for user interaction.
     - `BICONT`: Input file (256 bytes, indexed) for company control data.
     - `ARCUST`: Input file (384 bytes, indexed) for customer master data.
     - `BB954S`: Input file (9 bytes, indexed) for temporary customer list.
   - **Arrays**:
     - `MSG`: Array of 6 error messages (40 characters each).
     - `DCO`: Array of 10 customer names (35 characters each).
     - `DCS`: Array of 10 customer numbers (6 digits each).
   - **Data Structures**:
     - `GCFFYM`, `GCLFYM`, `GCFFYR`, `GCFFMO`, `GCLSYR`, `GCLFMO`: Date-related fields for fiscal year/month processing.
     - `UDS`: User data structure for key fields like `KYDATE`, `KYMO`, `KYYR`, `KYCO`, `KYCUST`, `KYJOBQ`, `KYCOPY`, `KYCANC`, `KYPASS`, `Y2KCEN`, `Y2KCMP`.
   - **OCL Syntax**:
     ```rpg
     FSCREEN  CP  F    1024            WORKSTN
     FBICONT  IF  F 256 256  2AI     2 DISK
     FARCUST  IF  F 384 384  8AI     2 DISK
     FBB954S  IF  F   9   9L 8AI     2 DISK
     ```

2. **Initialization**:
   - Clears the `MSG40` field (used for error messages).
   - Resets various indicators (50–56, 81, 90) to ensure a clean state.
   - **OCL Syntax**:
     ```rpg
     MOVEL*BLANKS   MSG40  40
     SETOF                     505152
     SETOF                     535455
     SETOF                     56
     SETOF                     8190
     ```

3. **Handle Cancel Key (KG Indicator)**:
   - If the cancel key (`KG`) is pressed, the program:
     - Sets indicators 01–09 off.
     - Sets `KYCANC` to 'CANCEL'.
     - Sets the Last Record (`LR`) indicator on.
     - Jumps to the `END` tag to terminate processing.
   - **OCL Syntax**:
     ```rpg
     C   KG                SETOF                     0109
     C   KG                MOVEL'CANCEL'  KYCANC
     C   KG                SETON                     LR
     C   KG                GOTO END
     ```

4. **One-Time Setup (ONETIM Subroutine)**:
   - Executed only for indicator `09` (first cycle).
   - **Purpose**: Populates the `DCO` and `DCS` arrays with customer names and numbers from `BB954S` and `ARCUST` for screen display.
   - **Steps**:
     - Clears the `DCO` array.
     - Initializes index `X` to 1 and sets `SCUST` to 10000000 (likely a starting customer number).
     - Positions the file pointer in `BB954S` using `SETLL`.
     - Reads records from `BB954S` in a loop until end-of-file (indicator 10):
       - Skips records marked for deletion (`PSDEL = 'D'`).
       - For valid records, retrieves the customer name from `ARCUST` using `PSKEY` (customer number) via `CHAIN`.
       - Stores the customer number (`PSCUST`) in `DCS,X` and name (`ARNAME`) in `DCO,X`.
       - Increments `X` until 10 customers are processed or no more records are found.
     - Sets defaults: `KYJOBQ = 'N'`, `KYCOPY = 01`, and indicator 81 on (likely for screen display).
   - **OCL Syntax**:
     ```rpg
     CSR         ONETIM    BEGSR
     MOVEL*BLANKS   DCO
     Z-ADD1         X       20
     Z-ADD10000000  SCUST   80
     SCUST     SETLLBB954S
     READ BB954S                   10
     PSDEL     COMP 'D'                      10
     PSKEY     CHAINARCUST               98
     N98       MOVELARNAME    DCO,X
     ADD  1         X
     X         COMP 10                   10
     SETON                     81
     MOVEL'N'       KYJOBQ
     Z-ADD01        KYCOPY
     ```

5. **Screen Validation (S1 Subroutine)**:
   - **Purpose**: Validates user input from the workstation screen (`SCREEN`) for company number, customer number, date, job queue, and password.
   - **Steps**:
     - **Validate Company Number** (`KYCO`):
       - Uses `CHAIN` to check if `KYCO` exists in `BICONT`.
       - If not found (indicator 97), sets indicators 81 and 90, sets error message `MSG,4` ("INVALID COMPANY NUMBER ENTERED"), and jumps to `ENDS1`.
     - **Validate Customer Number** (`KYCUST`):
       - Uses `LOKUP` to check if `KYCUST` exists in the `DCS` array.
       - If not found (indicator 40 off), sets indicators 81 and 90, sets Ascertain the business rules enforced by the provided RPG program and the associated OCL program.

### Business Rules

The RPG program enforces several business rules to ensure valid data input and processing:

1. **Company Number Validation**:
   - The company number (`KYCO`) must exist in the `BICONT` file. If not, the program displays the error message "INVALID COMPANY NUMBER ENTERED" and halts processing.

2. **Customer Number Validation**:
   - The customer number (`KYCUST`) must match one of the customer numbers in the `DCS` array (populated from `BB954S` and `ARCUST`). If invalid, the error message "INVALID CUSTOMER NUMBER ENTERED" is displayed, and processing halts.

3. **Date Validation**:
   - The month (`KYMO`) must be between 01 and 12. If invalid, the error message "INVALID DATE ENTERED" is displayed, and processing halts.
   - The date (`KYDATE`) is converted to an 8-digit format (`BEGDT8`) with century adjustment for Y2K compliance:
     - If the year (`BYY`) is greater than or equal to a comparison year (`Y2KCMP`), the century (`Y2KCEN`) is used as is.
     - Otherwise, the century is incremented by 1.

4. **Job Queue Validation**:
   - The job queue code (`KYJOBQ`) must be 'Y', 'N', or blank. If invalid, the error message "INVALID JOB QUEUE CODE ENTERED" is displayed, and processing halts.

5. **Copy Count Validation**:
   - The copy count (`KYCOPY`) must be greater than 0. If 0, it is set to 1.

6. **Password Validation**:
   - The password (`KYPASS`) must be either 'CUSTPROD' or blank. If neither, the error message "INVALID PASSWORD ENTERED" is displayed, and processing halts.

7. **Customer List Display**:
   - Up to 10 customer numbers and names are retrieved from `BB954S` and `ARCUST` (excluding deleted records) and displayed on the screen for user selection.

8. **Cancel Handling**:
   - If the cancel key is pressed, the program sets `KYCANC` to 'CANCEL', terminates processing, and jumps to the `END` tag.

### Tables (Files) Used

The RPG program uses the following files:
1. **SCREEN**: Workstation file (1024 bytes) for user input/output, using format `BB954PFM`.
2. **BICONT**: Company control file (256 bytes, indexed), used to validate company number (`KYCO`).
3. **ARCUST**: Customer master file (384 bytes, indexed), used to retrieve customer names (`ARNAME`) for display.
4. **BB954S**: Temporary customer list file (9 bytes, indexed), used to populate customer numbers and names for screen display.

### External Programs Called

The RPG program `BB954P` does not explicitly call any external programs. It is invoked by the OCL program `BB954P.ocl36.txt` and operates as part of the broader pricing update process, interacting with the workstation and files but not directly invoking other RPG programs.

### Summary

The RPG program `BB954P` is a critical component of the rack pricing update process, focusing on user interaction and input validation. It:
- Populates a screen with up to 10 customer names and numbers from `BB954S` and `ARCUST`.
- Validates user inputs (company number, customer number, date, job queue, password).
- Handles Y2K date adjustments.
- Displays error messages and halts processing for invalid inputs.
- Supports cancellation via the `KG` indicator.
- Uses four files: `SCREEN`, `BICONT`, `ARCUST`, and `BB954S`.
- Enforces strict business rules to ensure data integrity before passing control back to the OCL program for further processing (e.g., by `BB9542`).

No external programs are called within `BB954P`, but it integrates with the OCL-driven workflow involving `BB9541` and `BB9542`.