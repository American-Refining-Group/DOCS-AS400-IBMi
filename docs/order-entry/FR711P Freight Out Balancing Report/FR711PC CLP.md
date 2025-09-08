The CLP program `FR711PC` serves as an intermediary program that processes parameters received from the main program (`FR711P`) and submits the appropriate report generation program based on the sort option provided. Below is an explanation of the process steps, business rules, tables used, and external programs called, based on the provided source code.

---

### Process Steps of the CLP Program (`FR711PC`)

The program receives parameters, evaluates the sort option, and calls one of two report generation programs (`FR711C` or `FR712C`) to produce the Freight Out Reconciliation Report. Here are the steps:

1. **Program Declaration and Parameter Receipt**:
   - **Purpose**: Defines the program and its input parameters.
   - **Actions**:
     - Declares the program with seven input parameters:
       - `&P$CO`: Company code (2 characters).
       - `&P$RDAT`: Report date (8 characters).
       - `&P$LOC`: Location code (3 characters).
       - `&P$CAID`: Carrier ID (6 characters).
       - `&P$SORT`: Sort option (1 character, 'L' for Location or 'C' for Carrier/Routing).
       - `&P$SELMO`: Month selection (1 character, added in revision `jb01`).
       - `&P$FGRP`: File group (1 character, 'G' or 'Z').
     - Declares a variable `&MSG` (512 characters) for status messages.

2. **Sort by Location (`&P$SORT = 'L'`)**:
   - **Purpose**: Processes the report when sorted by location.
   - **Actions**:
     - Checks if `&P$SORT` equals 'L'.
     - If true, calls the program `FR711C`, passing all seven parameters (`&P$CO`, `&P$RDAT`, `&P$LOC`, `&P$CAID`, `&P$SORT`, `&P$SELMO`, `&P$FGRP`).
     - Sets a status message in `&MSG`: "Freight Out Reconciliation Report By Location, Has Been Submitted".
     - Sends the status message to the external program queue using `SNDPGMMSG` with message ID `CPF9898` from message file `QCPFMSG`.
     - Delays execution for 2 seconds using `DLYJOB` to ensure the message is displayed.

3. **Sort by Carrier/Routing (`&P$SORT = 'C'`)**:
   - **Purpose**: Processes the report when sorted by carrier/routing.
   - **Actions**:
     - Checks if `&P$SORT` equals 'C'.
     - If true, calls the program `FR712C`, passing all seven parameters (`&P$CO`, `&P$RDAT`, `&P$LOC`, `&P$CAID`, `&P$SORT`, `&P$SELMO`, `&P$FGRP`).
     - Sets a status message in `&MSG`: "Freight Out Reconciliation Report By Carrier, Has Been Submitted".
     - Sends the status message to the external program queue using `SNDPGMMSG` with message ID `CPF9898` from message file `QCPFMSG`.
     - Delays execution for 2 seconds using `DLYJOB` to ensure the message is displayed.

4. **Program Termination**:
   - **Purpose**: Ends the program after processing.
   - **Actions**:
     - The program ends with `ENDPGM` after executing the appropriate branch based on the sort option.

---

### Business Rules

- **Sort Option Handling**:
  - The program uses the `&P$SORT` parameter to determine which report program to call:
    - `'L'`: Calls `FR711C` to generate the report sorted by location.
    - `'C'`: Calls `FR712C` to generate the report sorted by carrier/routing.
  - If `&P$SORT` is neither 'L' nor 'C', the program does not call any report program and simply ends, which could be an oversight since the calling program (`FR711P`) validates `&P$SORT` to be either 'C' or 'L'.

- **Month Selection** (from revision `jb01`):
  - The `&P$SELMO` parameter (added in 2015) supports the following values, as defined in `FR711P`:
    - `M`: Selected month only, including all (open and closed) records.
    - `O`: Selected month, open records only.
    - `C`: Selected month, closed records only.
    - `A`: All open records from previous and selected months.
    - Blank: Previous month open records and selected month all records.
  - This parameter is passed to the called programs (`FR711C` or `FR712C`) for report filtering.

- **File Group**:
  - The `&P$FGRP` parameter ('G' or 'Z') determines the database files used (e.g., `gglcont`/`ginloc` for 'G', `zglcont`/`zinloc` for 'Z'), as handled by the calling program `FR711P`.

- **Status Messaging**:
  - The program provides feedback to the user via status messages, indicating whether the report is sorted by location or carrier.
  - The 2-second delay (`DLYJOB`) ensures the status message is visible before the program ends or control returns to the caller.

- **Commented-Out Code**:
  - The commented-out `SBMJOB` command suggests that the program may have originally submitted `FR711C` as a batch job rather than calling it directly. The current implementation uses direct calls to `FR711C` or `FR712C` based on the sort option.

---

### Tables Used

- **No Direct Table Access**:
  - The `FR711PC` program does not directly access any database tables. It relies on the called programs (`FR711C` or `FR712C`) to handle data retrieval and processing.
  - The `&P$FGRP` parameter implies that the called programs will access overridden files (`gglcont`/`ginloc` or `zglcont`/`zinloc`), as set up by the calling program `FR711P`.

- **Message File**:
  - `QCPFMSG`: A system message file used by the `SNDPGMMSG` command to send status messages with message ID `CPF9898`.

---

### External Programs Called

- **FR711C**: Called when `&P$SORT = 'L'` to generate the Freight Out Reconciliation Report sorted by location.
- **FR712C**: Called when `&P$SORT = 'C'` to generate the Freight Out Reconciliation Report sorted by carrier/routing.
- **SNDPGMMSG (System Command)**: Used to send status messages to the external program queue, indicating report submission.

---

### Additional Notes

- **Purpose**: The program acts as a dispatcher, routing the report generation to the appropriate program (`FR711C` or `FR712C`) based on the sort option, ensuring the correct report format is produced.
- **Parameter Validation**: The program assumes that the input parameters are valid, as they are validated by the calling program `FR711P`.
- **Revision History**:
  - `jb01` (04/07/15): Added the `&P$SELMO` parameter to support month selection filtering, enhancing report flexibility.
- **Error Handling**: The program does not include explicit error handling, relying on the called programs to manage errors and on `FR711P` for input validation.
- **Execution Context**: The direct calls to `FR711C` or `FR712C` (instead of `SBMJOB`) suggest that the report generation occurs in the same job session, with immediate feedback via status messages.

This program streamlines the submission of the Freight Out Reconciliation Report by directing the process to the appropriate report generation program based on the user-specified sort option.