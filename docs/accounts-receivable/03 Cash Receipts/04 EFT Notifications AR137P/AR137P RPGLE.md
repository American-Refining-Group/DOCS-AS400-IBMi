The provided `AR137P.rpgle.txt` is an RPGLE (RPG IV) program used in IBM i (AS/400) systems, invoked by the `AR137P.ocl36.txt` OCL script for EFT (Electronic Funds Transfer) email selection by customer or all. This program handles user input validation for EFT posting table date selection, ensuring that company numbers, customer numbers, and selection criteria are valid before proceeding. Below, I’ll explain the process steps, business rules, tables used, and external programs called, providing a clear and concise analysis.

### Process Steps of the RPGLE Program

The `AR137P` program is designed to validate user input from a workstation (display file) and set up parameters for EFT processing. It uses a display file for user interaction and validates data against database files. Here’s a step-by-step breakdown of the program’s execution:

1. **Program Initialization**:
   - **Header Specifications**:
     - `H DFTACTGRP(*NO)`: Runs in a named activation group, allowing better control over program resources.
     - `H FIXNBR(*ZONED:*INPUTPACKED)`: Ensures zoned and packed decimal fields are handled correctly during input.
     - `H DFTNAME(AR137P)`: Sets the default program name to `AR137P`.
   - **File Definitions**:
     - `AR137PD`: A workstation file (display file) for user interaction, using the Profound UI handler for modern UI support.
     - `ARCUST`: Input file (384 bytes, key starting at position 2) for customer data.
     - `ARCONT`: Input file (256 bytes, key starting at position 2) for accounts receivable control data.
     - `GSCONT`: Input file (512 bytes, key starting at position 2) for system control data.
   - **Data Structures and Variables**:
     - `MSG`: A 40-character array of 6 error messages loaded via `CTDATA`.
     - `FILENM`: A data structure with an 8-character `FILE` field for constructing a file name.
     - `UDS`: A data area structure for Local Data Area (LDA) fields, including:
       - `KYCO` (company number, positions 101-102).
       - `KYSLDT` (selection date, positions 103-108).
       - `STATUS` (status flag, position 109).
       - `KYUPDT` (update date, positions 110-115).
       - `KYALSL` (selection type, ‘ALL’ or ‘SEL’, positions 121-123).
       - `KYCS01` to `KYCS10` (customer numbers, positions 124-183).
       - `Y2KCEN` and `Y2KCMP` (positions 509-512) for Y2K date handling.
   - **Indicators**: Uses indicators (e.g., `*IN01`, `*IN09`, `*IN20`, etc.) to control program flow and screen output.

2. **Initial Workstation File Read**:
   - Checks if `QSCTL` (control field) is blank:
     - If blank, sets `*IN09` on (initial run), `*IN01` off, and `QSCTL` to ‘R’ (read mode).
     - If not blank, sets `*IN09` off, `*IN01` on, reads the `AR137PD` display file, and returns if the last record is read (`LR` indicator).
   - Initializes indicators (`*IN50` to `*IN55`, `*IN20`, `*IN81`) to off and clears `MSG35` (message field).

3. **Cancel Check**:
   - If `*INKG` (cancel key, typically F3) is on:
     - Sets `*IN01` off, `*INU8` on (user-controlled indicator), `*INLR` on (last record, program end).
     - Jumps to the `END` tag to terminate the program.

4. **One-Time Setup (`ONETIM` Subroutine)**:
   - If `*IN09` is on (initial run):
     - Executes the `ONETIM` subroutine:
       - Chains to `GSCONT` with key ‘00’ to retrieve the company number (`GXCONO`).
       - If found (`*IN99` off) and `GXCONO` is non-zero, sets `KYCO` to `GXCONO`.
       - Sets `KYALSL` to ‘ALL’ (default selection type).
       - Sets `*IN81` on (display update flag) and clears `MSG40` (message field).
     - Jumps to the `END` tag, skipping further processing for the initial run.

5. **Input Validation (`EDIT` Subroutine)**:
   - If `*IN01` is on (user submitted input):
     - Executes the `EDIT` subroutine to validate input fields:
       - **Initialize Indicators**: Clears indicators `*IN20`, `*IN30` to `*IN41`, and `*IN90` to ensure a clean validation state.
       - **Validate Company**:
         - Chains to `ARCONT` using `KYCO` (company number).
         - If not found (`*IN20` on), sets `*IN81` and `*IN90` on, moves error message 1 (“INVALID COMPANY #”) to `MSG40`, and jumps to `ENDEDT`.
       - **Validate Selection Type**:
         - If `KYALSL` is ‘ALL’ and `KYCS01` (first customer number) is non-zero, sets `*IN81`, `*IN90`, `*IN41` on, moves error message 5 (“CANNOT SELECT CUSTOMERS WITH ‘ALL’”) to `MSG40`, and jumps to `ENDEDT`.
         - If `KYALSL` is ‘SEL’ and `KYCS01` is zero, sets `*IN81`, `*IN90`, `*IN31` on, moves error message 6 (“MUST ENTER CUSTOMER WITH ‘SEL’”) to `MSG40`, and jumps to `ENDEDT`.
       - **Validate Customers**:
         - If `KYALSL` is ‘SEL’, validates up to 10 customer numbers (`KYCS01` to `KYCS10`):
           - For each non-zero customer number, constructs a key (`KYCOCU`) by combining `KYCO` and the customer number.
           - Chains to `ARCUST` using `KYCOCU`.
           - If not found, sets `*IN81`, `*IN90`, and the corresponding indicator (`*IN31` to `*IN40`), moves error message 4 (“INVALID CUSTOMER # ENTERED”) to `MSG40`, and jumps to `ENDEDT`.
       - **Validate Update Date**:
         - If `KYUPDT` (update date) is zero, sets `*IN81`, `*IN90` on, moves error message 2 (“MUST ENTER BATCH CREATE DATE”) to `MSG40`, and jumps to `ENDEDT`.
       - **Check File Existence**:
         - Constructs a file name in `FILENM` by combining ‘GE’ and `KYUPDT`.
         - Calls the `AR135TC` program, passing `FILENM` and `STATUS`.
         - If `STATUS` is ‘N’ (file does not exist), sets `*IN81`, `*IN20`, `*IN90` on, moves error message 3 (“FILE WITH DATE SELECTED DOES NOT EXIST”) to `MSG40`, and jumps to `ENDEDT`.
       - **Successful Validation**:
         - If all validations pass, sets `STATUS` to ‘Y’ (valid input).
     - Jumps to `ENDEDT` to end the subroutine.

6. **Program Termination**:
   - At the `END` tag:
     - If `*IN81` is off (no errors), sets `*INLR` on to end the program.
     - If `*IN81` is on (errors), writes to the `AR137PD` display file to show error messages (e.g., `MSG40`) to the user.

### Business Rules

The program enforces the following business rules for EFT email selection:

1. **Company Validation**:
   - The company number (`KYCO`) must exist in the `ARCONT` file. If invalid, error message “INVALID COMPANY #” is displayed.

2. **Selection Type Consistency**:
   - If `KYALSL` is ‘ALL’ (process all customers), no customer numbers (`KYCS01` to `KYCS10`) can be specified. Violation triggers “CANNOT SELECT CUSTOMERS WITH ‘ALL’”.
   - If `KYALSL` is ‘SEL’ (selective customers), at least one customer number (`KYCS01`) must be provided. If not, “MUST ENTER CUSTOMER WITH ‘SEL’” is displayed.

3. **Customer Validation**:
   - For `KYALSL` = ‘SEL’, each provided customer number (`KYCS01` to `KYCS10`) must exist in the `ARCUST` file for the specified company (`KYCO`). Invalid customers trigger “INVALID CUSTOMER # ENTERED”.

4. **Update Date Requirement**:
   - The update date (`KYUPDT`) must be non-zero. If zero, “MUST ENTER BATCH CREATE DATE” is displayed.

5. **File Existence Check**:
   - The EFT posting table file (named with prefix ‘GE’ and `KYUPDT`) must exist. If not, “FILE WITH DATE SELECTED DOES NOT EXIST” is displayed.

6. **Default Setup**:
   - On initial run, the company number is retrieved from `GSCONT` (key ‘00’), and `KYALSL` defaults to ‘ALL’.

7. **Error Handling**:
   - Any validation failure sets error indicators (`*IN81`, `*IN90`) and displays an error message via the `AR137PD` display file, prompting the user to correct input.

### Tables (Files) Used

The program uses the following files:

1. **AR137PD**:
   - Type: Workstation (display) file.
   - Purpose: Handles user input/output for the EFT selection screen, using Profound UI for modern interface rendering.

2. **ARCUST**:
   - Type: Input file (384 bytes, key at position 2).
   - Fields: `ARDEL` (delete code), `ARCO` (company number), `QQARCU` (customer number), `ARNAME` (customer name), `ARADR1` to `ARADR3` (address lines).
   - Purpose: Stores customer data for validation of customer numbers.

3. **ARCONT**:
   - Type: Input file (256 bytes, key at position 2).
   - Purpose: Stores accounts receivable control data for validating company numbers.

4. **GSCONT**:
   - Type: Input file (512 bytes, key at position 2).
   - Fields: `GXDEL` (delete code), `GXCONO` (company number).
   - Purpose: Stores system control data, used to retrieve the default company number.

### External Programs Called

The program calls the following external program:

1. **AR135TC**:
   - Called in the `EDIT` subroutine to check if the EFT posting table file (constructed from ‘GE’ and `KYUPDT`) exists.
   - Parameters: `FILENM` (input, file name), `STATUS` (output, ‘Y’ or ‘N’).

### Integration with OCL Script

The `AR137P` RPGLE program is invoked by the `AR137P.ocl36.txt` OCL script via `// LOAD AR137P` and `// RUN`. The OCL script:
- Defines the `ARCONT` and `ARCUST` files, matching the RPGLE file definitions.
- Calls `AR137` (likely another program or procedure) after `AR137P` completes, passing parameters including the validated `KYALSL` and customer numbers.
- Uses the `STATUS` field (position 109) to determine whether to proceed with `AR137` (`// IFF ?L'109,1'?/Y GOTO END`).

The RPGLE program sets `STATUS` to ‘Y’ if validations pass, allowing the OCL script to proceed with `AR137`. If validations fail, `STATUS` remains unset or is ‘N’, causing the OCL script to skip `AR137`.

### Summary

The `AR137P` RPGLE program validates user input for EFT email selection, ensuring valid company numbers, customer numbers, selection type, and update date. It interacts with the user via the `AR137PD` display file, checks data against `ARCUST`, `ARCONT`, and `GSCONT` files, and verifies file existence via `AR135TC`. The program enforces strict business rules to ensure data integrity before passing control back to the OCL script for further processing.

**Tables Used**: `AR137PD` (display), `ARCUST` (customer), `ARCONT` (control), `GSCONT` (system control).
**External Programs Called**: `AR135TC`.