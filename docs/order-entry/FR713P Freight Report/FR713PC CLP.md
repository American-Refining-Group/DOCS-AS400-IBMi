The `FR713PC` program is a CL (Control Language) program that serves as an intermediary to submit the Freight Out Reconciliation Report generation based on user-specified parameters. It is called by the `FR713P` RPG program and determines which specific report generation program to invoke based on the sort option provided. Below, I’ll explain the process steps, business rules, database tables used, and external programs called, based on the provided CLP source code.

### Process Steps of the FR713PC Program

The `FR713PC` program processes input parameters and routes the request to one of three report generation programs (`FR713C`, `FR714C`, or `FR715C`) based on the sort option. Here’s a detailed breakdown of the process steps:

1. **Program Declaration and Parameter Definition**:
   - **Purpose**: Defines the program and its input parameters.
   - **Actions**:
     - Declares the program with nine input parameters, all of type `*CHAR`:
       - `&P$CO` (2): Company code.
       - `&P$FDAT` (8): From date (in `CYMD` format).
       - `&P$TDAT` (8): To date (in `CYMD` format).
       - `&P$LOC` (3): Location code.
       - `&P$CAID` (6): Carrier ID.
       - `&P$SORT` (1): Sort option ('L' for Location, 'C' for Carrier/Routing, 'P' for Product Code).
       - `&P$CUST` (6): Customer code.
       - `&P$CAR$` (1): Flag to include zero-dollar freight records ('Y' or 'N').
       - `&P$FGRP` (1): File group ('G' or 'Z', determining which set of database files to use).
     - Declares a variable `&MSG` (512 characters) to store status messages.

2. **Conditional Logic Based on Sort Option**:
   - **Purpose**: Determines which report generation program to call based on the `&P$SORT` parameter.
   - **Actions**:
     - The program uses `IF` conditions to evaluate the value of `&P$SORT` and execute one of three branches:
       - **Sort by Location (`&P$SORT = 'L'`)**:
         - Calls the `FR713C` program, passing all input parameters (`&P$CO`, `&P$FDAT`, `&P$TDAT`, `&P$LOC`, `&P$CAID`, `&P$SORT`, `&P$CUST`, `&P$CAR$`, `&P$FGRP`).
         - Sets `&MSG` to "Freight Out Reconciliation Report By Location, Has Been Submitted".
         - Sends a status message to the user using `SNDPGMMSG` with message ID `CPF9898` from `QCPFMSG`, displaying the message externally (`TOPGMQ(*EXT)`).
         - Delays execution for 2 seconds using `DLYJOB DLY(2)` to ensure the status message is visible.
       - **Sort by Carrier/Routing (`&P$SORT = 'C'`)**:
         - Calls the `FR714C` program with the same parameters.
         - Sets `&MSG` to "Freight Out Reconciliation Report By Carrier, Has Been Submitted".
         - Sends the status message and delays for 2 seconds, as above.
       - **Sort by Product Code (`&P$SORT = 'P'`)**:
         - Calls the `FR715C` program with the same parameters.
         - Sets `&MSG` to "Freight Out Reconciliation Report By Product Code, Has Been Submitted".
         - Sends the status message and delays for 2 seconds, as above.
   - **Note**: The program does not handle invalid sort options explicitly, as validation is performed in the calling program (`FR713P`).

3. **Program Termination**:
   - **Purpose**: Ends the program after processing the appropriate branch.
   - **Actions**:
     - Executes the `ENDPGM` command to terminate the program.

### Business Rules

The `FR713PC` program enforces the following business rules:
1. **Sort Option Routing**:
   - The program routes the report generation to one of three programs based on the `&P$SORT` value:
     - 'L': Report sorted by location (`FR713C`).
     - 'C': Report sorted by carrier/routing (`FR714C`).
     - 'P': Report sorted by product code (`FR715C`).
   - This ensures the report is generated in the format requested by the user.
2. **Parameter Consistency**:
   - All input parameters are passed unchanged to the called report generation program, ensuring consistency with the validated inputs from `FR713P`.
3. **User Feedback**:
   - A status message is displayed to inform the user that the report has been submitted, specifying the sort option used.
   - The 2-second delay (`DLYJOB`) ensures the status message is visible before the program ends or control returns to the calling program.
4. **No Data Validation**:
   - The program assumes all input parameters are valid, as validation is handled by the `FR713P` program (e.g., company code, dates, location, customer, etc.).
5. **File Group Handling**:
   - The `&P$FGRP` parameter ('G' or 'Z') determines which set of database files the called program will use, as set up by `FR713P` via file overrides.

### Database Tables Used

The `FR713PC` program itself does not directly access any database tables. Instead, it passes parameters to the called programs (`FR713C`, `FR714C`, or `FR715C`), which are responsible for accessing the necessary data. However, based on the context provided by the `FR713P` program, the database tables likely accessed by the called programs include:
1. **glcont** (General Ledger Control):
   - Used for company code validation (overridden to `gglcont` or `zglcont` based on `&P$FGRP`).
2. **inloc** (Inventory Location):
   - Used for location code validation (overridden to `ginloc` or `zinloc`).
3. **arcust** (Accounts Receivable Customer):
   - Used for customer code validation (overridden to `garcust` or `zarcust`).
4. **Additional Tables**:
   - The report generation programs (`FR713C`, `FR714C`, `FR715C`) likely access additional tables related to freight transactions, but these are not specified in the provided code.

Since `FR713PC` does not directly interact with database files, the actual table access depends on the implementation of the called programs.

### External Programs Called

The `FR713PC` program calls one of the following external programs based on the sort option:
1. **FR713C**:
   - Called when `&P$SORT = 'L'` to generate the Freight Out Reconciliation Report sorted by location.
   - Parameters: `&P$CO`, `&P$FDAT`, `&P$TDAT`, `&P$LOC`, `&P$CAID`, `&P$SORT`, `&P$CUST`, `&P$CAR$`, `&P$FGRP`.
2. **FR714C**:
   - Called when `&P$SORT = 'C'` to generate the report sorted by carrier/routing.
   - Parameters: Same as above.
3. **FR715C**:
   - Called when `&P$SORT = 'P'` to generate the report sorted by product code.
   - Parameters: Same as above.

### Additional Notes
- **Commented Code**:
  - The source code includes a commented-out `SNDMSG` command for debugging (sending `&P$TDAT` to user `KRAJTEST`) and a commented-out `SBMJOB` command that suggests an earlier version submitted `FR713C` to a job queue. The current implementation calls the programs directly instead of submitting them as batch jobs.
- **Status Messages**:
  - The use of `SNDPGMMSG` with `CPF9898` (a generic message ID) allows flexible status messages to be displayed to the user, confirming report submission.
- **No Error Handling**:
  - The program does not include explicit error handling, relying on the calling program (`FR713P`) to ensure valid inputs.
- **Integration with FR713P**:
  - The `FR713PC` program is tightly coupled with `FR713P`, which validates inputs and sets up file overrides before calling this program.

### Summary

The `FR713PC` program acts as a dispatcher, routing the Freight Out Reconciliation Report generation to one of three programs (`FR713C`, `FR714C`, or `FR715C`) based on the sort option ('L', 'C', or 'P'). It provides user feedback via status messages and does not directly access database tables, relying on the called programs for data processing. The business rules ensure the correct report format is generated based on user input, with parameters passed consistently to maintain data integrity.