The `FR713P` program is an RPG program designed to prompt users for input parameters to generate a Freight Out Reconciliation Report. Below, I’ll outline the process steps, list the external programs called, and identify the database tables used, based on the provided RPGLE source code.

### Process Steps of the FR713P Program

The program follows a structured flow to collect user input, validate it, and call a print program to generate the report. Here’s a detailed breakdown of the process steps:

1. **Program Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Sets up initial conditions and prepares the environment.
   - **Actions**:
     - Receives an input parameter `p$fgrp` (file group: 'Z' or 'G') to determine which set of database files to use.
     - Initializes default values:
       - Sets `c1sort` to 'C' (default sort option).
       - Sets `c1car$` to 'N' (default for including zero-dollar freight records).
     - Loads the header text for the display (`c$hdr1`) from the `hdr` array.
     - Captures the current date and time using the `time` operation and populates date fields (`t#cymd`) for use in the program.
     - Clears the customer field (`c1cust`).
     - Defines key lists (`key1` for location lookup and `klcust` for customer lookup) for database operations.
     - Sets `fmtagn` to `*on` to indicate the prompt screen loop should continue.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Purpose**: Opens the necessary database files with appropriate overrides based on the file group (`p$fgrp`).
   - **Actions**:
     - Checks if `p$fgrp` is 'G' or 'Z'.
     - Applies file overrides using the `ovg` (for 'G') or `ovz` (for 'Z') arrays, which contain `OVRDBF` commands to redirect file access to the appropriate library (e.g., `gglcont` for 'G' or `zglcont` for 'Z').
     - Executes the override commands using the `QCMDEXC` system API.
     - Opens the files `glcont`, `inloc`, and `arcust` for data access.

3. **Prompt for Input (`srfmt` Subroutine)**:
   - **Purpose**: Displays the prompt screen (`FMT01`) to collect user input and processes it until valid or the user exits.
   - **Actions**:
     - Enters a loop (`fmtagn = *on`) to repeatedly display the prompt screen using `EXFMT FMT01`.
     - Clears error indicators (50-69) to reset error states.
     - Handles function key presses:
       - **F3 (Exit)**: Sets `fmtagn` to `*off`, exits the loop, and proceeds to program termination.
       - **F4 (Prompt)**: Calls the `prompt` subroutine to assist with field input (e.g., customer lookup).
     - Validates input fields by calling the `f01edt` subroutine.
     - If no errors are found (`*in50 = *off`), calls the `callpgm` subroutine to proceed with report generation and exits the loop.

4. **Field Validation (`f01edt` Subroutine)**:
   - **Purpose**: Validates user input fields to ensure they meet the required criteria.
   - **Actions**:
     - **Company (`c1co`)**:
       - Checks if the company code exists in the `glcont` file using a `CHAIN` operation.
       - If invalid or zero, sets error indicators (`*in50`, `*in51`) and displays error message `com(01)` ("Invalid Company Entered").
       - If valid, retrieves the company name (`gcname`) into `f$cnam`.
     - **From Date (`c1fmdy`)**:
       - Calls the `GSDTEDIT` program to validate the date format and convert it to `CYMD` format.
       - If invalid, sets error indicators (`*in50`, `*in52`) and displays error message `com(02)` ("Invalid Date Entered").
       - If valid, stores the converted date in `w$fdat`.
     - **To Date (`c1tmdy`)**:
       - Similarly validates the to-date using `GSDTEDIT`.
       - If invalid, sets error indicators (`*in50`, `*in53`) and displays error message `com(02)`.
       - If valid, stores the converted date in `w$tdat`.
     - **Date Range Check**:
       - Ensures the from date (`w$fdat`) is not greater than the to date (`w$tdat`).
       - If invalid, sets error indicators (`*in50`, `*in52`) and displays error message `com(03)` ("From Date Cannot be GT than to Date").
     - **Location (`c1loc`)**:
       - If non-blank, validates the location code in the `inloc` file using a `CHAIN` operation with `key1` (company and location).
       - If invalid, sets error indicators (`*in50`, `*in54`) and displays error message `com(04)` ("Invalid Loc Entered").
     - **Customer (`c1cust`)**:
       - If non-zero, validates the customer code in the `arcust` file using a `CHAIN` operation with `klcust` (company and customer).
       - If invalid or marked as deleted (`ardel = 'D'`), sets error indicators (`*in50`, `*in56`) and displays error message `com(06)` ("Invalid Customer Entered").
     - **Zero Dollar Freight Records (`c1car$`)**:
       - Validates that the value is either 'Y' or 'N'.
       - If invalid, sets error indicators (`*in50`, `*in57`) and displays error message `com(07)` ("Invalid Code, Valid Codes: Y/N").
     - **Sort Option (`c1sort`)**:
       - Validates that the sort code is 'C', 'L', or 'P'.
       - If invalid, sets error indicators (`*in50`, `*in58`) and displays error message `com(05)` ("Invalid Sort Entered, Valid Codes: C/L").

5. **Field Prompting (`prompt` Subroutine)**:
   - **Purpose**: Assists the user in selecting valid values for fields when F4 is pressed.
   - **Actions**:
     - Checks the cursor location and record format (`FMT01`).
     - If the cursor is on the customer field (`C1CUST`), calls the `LARCUST` program to display a customer selection window.
     - Passes `c1co` (company) and `c1cust` (customer) as input parameters and receives updated values in `o$co#`, `o$cust#`, and `o$fgrp`.

6. **Call Print Program (`callpgm` Subroutine)**:
   - **Purpose**: Invokes the report generation program with validated parameters.
   - **Actions**:
     - Moves validated input fields to output parameters:
       - `c1co` to `o$co` (company).
       - `w$fdat` to `o$fdat` (from date).
       - `w$tdat` to `o$tdat` (to date).
       - `c1loc` to `o$loc` (location).
       - `c1caid` to `o$caid` (carrier ID).
       - `c1sort` to `o$sort` (sort option).
       - `c1cust` to `o$cust` (customer).
       - `c1car$` to `o$car$` (zero-dollar freight flag).
       - `p$fgrp` to `o$fgrp` (file group).
     - Calls the `FR713PC` program, passing the above parameters to generate the report.

7. **Program Termination**:
   - **Purpose**: Cleans up and exits the program.
   - **Actions**:
     - Closes all open files using `CLOSE *ALL`.
     - Sets `*INLR` to `*ON` to indicate the program is ending.
     - Returns control to the caller.

### External Programs Called

The program interacts with the following external programs:
1. **QCMDEXC**:
   - System API used to execute file override commands (`OVRDBF`) in the `opntbl` subroutine.
   - Parameters: `dbov##` (override command string) and `dbol##` (length of the command).
2. **GSDTEDIT**:
   - Called in the `f01edt` subroutine to validate and convert dates from `MMDDYY` to `CYMD` format.
   - Parameters: `p#mdy` (input date), `p#cymd` (output converted date), `p#err` (error flag).
3. **LARCUST**:
   - Called in the `prompt` subroutine when F4 is pressed on the customer field to display a customer selection window.
   - Parameters: `o$co#` (company), `o$cust#` (customer), `o$fgrp` (file group).
4. **FR713PC**:
   - Called in the `callpgm` subroutine to generate the Freight Out Reconciliation Report.
   - Parameters: `o$co` (company), `o$fdat` (from date), `o$tdat` (to date), `o$loc` (location), `o$caid` (carrier ID), `o$sort` (sort option), `o$cust` (customer), `o$car$` (zero-dollar freight flag), `o$fgrp` (file group).

### Database Tables Used

The program accesses the following database files, with overrides applied based on the `p$fgrp` parameter:
1. **glcont** (General Ledger Control):
   - Used to validate the company code (`c1co`) and retrieve the company name (`gcname`).
   - Overrides: `gglcont` (for `p$fgrp = 'G'`) or `zglcont` (for `p$fgrp = 'Z'`).
2. **inloc** (Inventory Location):
   - Used to validate the location code (`c1loc`) for the specified company.
   - Overrides: `ginloc` (for `p$fgrp = 'G'`) or `zinloc` (for `p$fgrp = 'Z'`).
3. **arcust** (Accounts Receivable Customer):
   - Used to validate the customer code (`c1cust`) and check if the customer is deleted (`ardel = 'D'`).
   - Overrides: `garcust` (for `p$fgrp = 'G'`) or `zarcust` (for `p$fgrp = 'Z'`).

### Additional Notes
- **Display File**: The program uses a display file (`fr713pd`) to present the prompt screen (`FMT01`) and handle user input.
- **Indicators**: The program uses indicators (e.g., 50-69 for errors, 70 for protect mode) to control screen behavior and error handling.
- **Data Structures**: The program defines data structures for time (`t#`) and date (`d#`) conversions, as well as a program status data structure (`psds##`) and display file data structure (`dspf_ds`).
- **Compile Notes**: The program can be compiled using PDM options `c1` or `c2`, indicating specific compilation configurations.

This program serves as a front-end prompt for collecting and validating input parameters before passing them to the `FR713PC` program to produce the Freight Out Reconciliation Report.