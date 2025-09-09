The CL program `FR710PC` is a control language program designed to serve as an intermediary between the prompt program `FR710P` and the report generation program `FR710C`. Its primary purpose is to receive parameters from `FR710P`, call the `FR710C` program to generate the Freight Out Accrual Report, and display a status message upon completion. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

### Process Steps of the FR710PC Program

1. **Program Declaration and Parameter Definition**:
   - **Purpose**: Declares the program and its input parameters.
   - **Steps**:
     - The program is defined with three input parameters:
       - `&P$CO`: A 2-character field for the company number.
       - `&P$RDAT`: An 8-character field for the report date (in `CCYYMMDD` format).
       - `&P$FGRP`: A 1-character field for the file group ('G' or 'Z').
     - Declares a variable `&MSG` (512 characters) to store a status message.

2. **Call the Report Generation Program**:
   - **Purpose**: Invokes the `FR710C` program to generate the Freight Out Accrual Report.
   - **Steps**:
     - Calls the `FR710C` program, passing the parameters `&P$CO`, `&P$RDAT`, and `&P$FGRP` directly.
     - Note: A commented-out `SBMJOB` command suggests that the program could submit `FR710C` as a batch job, but the current implementation runs it interactively within the same job.

3. **Display Status Message**:
   - **Purpose**: Informs the user that the report has been printed.
   - **Steps**:
     - Sets the `&MSG` variable to the message "Freight Out Accrual Report By Company Has Been Printed".
     - Sends the message to the external program message queue (`*EXT`) using `SNDPGMMSG` with message ID `CPF9898` from the `QCPFMSG` message file, indicating a status message.
     - Delays the job for 2 seconds using `DLYJOB` to ensure the message is visible to the user before the program ends.

4. **Program Termination**:
   - **Purpose**: Ends the program cleanly.
   - **Steps**:
     - The `ENDPGM` command terminates the program, returning control to the calling program (`FR710P`).

### Business Rules
- **Parameter Validation**: The program assumes that the input parameters (`&P$CO`, `&P$RDAT`, `&P$FGRP`) are validated by the calling program (`FR710P`). No additional validation is performed in `FR710PC`.
- **Report Execution**: The program is responsible for triggering the report generation by calling `FR710C`. The commented-out `SBMJOB` command suggests an alternative batch processing option, which may be used for performance or scheduling purposes but is not currently active.
- **User Feedback**: The program provides immediate feedback to the user via a status message, ensuring they are informed of the report's completion.
- **File Group Handling**: The `&P$FGRP` parameter ('G' or 'Z') determines which file set (`gglcont` or `zglcont`) is used by `FR710C` for report data, as set up by the file overrides in `FR710P`.
- **Assumption of Success**: The program assumes that `FR710C` executes successfully and does not include error handling for cases where `FR710C` fails.

### Tables Used
- **None**: The `FR710PC` program itself does not directly reference or access any database tables. Any table access (e.g., `glcont`, `gglcont`, or `zglcont`) is handled by the `FR710P` program (via file overrides) or the `FR710C` program (for report data retrieval).

### External Programs Called
- **FR710C**: The main report generation program, called with parameters `&P$CO` (company number), `&P$RDAT` (report date), and `&P$FGRP` (file group). This program is responsible for producing the Freight Out Accrual Report.
- **SNDPGMMSG (System Command)**: Used to send a status message to the user. While not a standalone program, it interacts with the system message queue (`QCPFMSG`) to display the completion message.

### Additional Notes
- **Commented SBMJOB**: The commented-out `SBMJOB` command indicates a potential design choice to run `FR710C` as a batch job, which could allow for asynchronous processing or better resource management. The current interactive call suggests that immediate execution is preferred in the current implementation.
- **Minimal Error Handling**: The program does not check for errors returned from `FR710C` or handle exceptions, relying on `FR710P` for input validation and `FR710C` for report generation logic.
- **Purpose as Intermediary**: `FR710PC` acts as a simple pass-through to invoke `FR710C` and provide user feedback, with no significant business logic or data manipulation.

This program is a lightweight coordinator between the user interface (`FR710P`) and the report generator (`FR710C`), ensuring the report is triggered with the correct parameters and the user is notified of its completion.