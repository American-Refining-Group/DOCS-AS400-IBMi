The CL program `FR710C` is a control language program that serves as a coordinator in the Freight Out Accrual Report process. It is called by `FR710PC` and is responsible for creating and managing a work file (`FR710W`), invoking a program (`FR710A`) to build the work file with report data, and then calling another program (`FR710B`) to print the report. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

### Process Steps of the FR710C Program

1. **Program Declaration and Parameter Definition**:
   - **Purpose**: Declares the program and its input parameters.
   - **Steps**:
     - The program is defined with three input parameters:
       - `&P$CO`: A 2-character field for the company number.
       - `&P$RDAT`: An 8-character field for the report date (in `CCYYMMDD` format).
       - `&P$FGRP`: A 1-character field for the file group ('G' or 'Z').

2. **Create and Prepare Work File (`FR710W`)**:
   - **Purpose**: Ensures the work file `FR710W` exists in the `QTEMP` library and is ready for use.
   - **Steps**:
     - **Check for Work File Existence**:
       - Uses `CHKOBJ` to check if the `FR710W` file exists in the `QTEMP` library.
       - If the file does not exist (`CPF9801` message ID), executes a `DO` block to create it.
     - **Create Work File**:
       - If `&P$FGRP` is 'G', creates a duplicate of the `FR710W` file from the `DATA` library into `QTEMP` using `CRTDUPOBJ` with constraints (`CST(*NO)`) and triggers (`TRG(*NO)`) disabled.
       - If `&P$FGRP` is 'Z', creates a duplicate of the `FR710W` file from the `DATADEV` library into `QTEMP` with the same settings.
     - **Clear Work File**:
       - Clears the contents of the `FR710W` file in `QTEMP` using `CLRPFM` to ensure it is empty before new data is added.
     - **Override Work File**:
       - Applies a database file override using `OVRDBF` to ensure that references to `FR710W` point to the `QTEMP/FR710W` file, making it specific to the job and user (multi-user capable, as per revision `JB01`).

3. **Build Work File**:
   - **Purpose**: Populates the `FR710W` work file with data for the report.
   - **Steps**:
     - Calls the `FR710A` program, passing the parameters `&P$CO`, `&P$RDAT`, and `&P$FGRP`.
     - The `FR710A` program is responsible for retrieving and processing data based on the company number, report date, and file group, and writing it to the `FR710W` file.

4. **Call Print Program**:
   - **Purpose**: Generates the Freight Out Accrual Report using the data in the work file.
   - **Steps**:
     - Calls the `FR710B` program, passing the same parameters (`&P$CO`, `&P$RDAT`, `&P$FGRP`).
     - The `FR710B` program uses the data in `FR710W` to produce the final report output.

5. **Program Termination**:
   - **Purpose**: Ends the program cleanly.
   - **Steps**:
     - The `ENDPGM` command terminates the program, returning control to the calling program (`FR710PC`).

### Business Rules
- **Work File Management**:
  - The `FR710W` file is created in `QTEMP`, a job-specific temporary library, to ensure multi-user capability (revision `JB01`). This prevents conflicts when multiple users run the report simultaneously.
  - The source library for `FR710W` depends on the `&P$FGRP` parameter: `DATA` for 'G' and `DATADEV` for 'Z' (revision `jK01`), allowing the program to work with different data environments (e.g., production vs. development).
  - The work file is cleared before use to ensure no residual data affects the report.
- **File Overrides**:
  - The `OVRDBF` command ensures that `FR710W` references the job-specific copy in `QTEMP`, reinforcing multi-user support (revision `jK01`).
- **Parameter Dependency**:
  - The program relies on the calling program (`FR710PC`) to provide valid parameters (`&P$CO`, `&P$RDAT`, `&P$FGRP`), with no additional validation performed.
- **Data Processing**:
  - The `FR710A` program is responsible for data retrieval and processing, while `FR710B` handles report formatting and output. `FR710C` acts as a coordinator, ensuring the work file is properly set up before these steps.
- **Error Handling**:
  - The program handles the case where `FR710W` does not exist in `QTEMP` by creating it. However, it does not include explicit error handling for failures in `FR710A` or `FR710B`.

### Tables Used
- **FR710W**: A work file created in the `QTEMP` library, used to store temporary data for the Freight Out Accrual Report. It is copied from either the `DATA` or `DATADEV` library based on the `&P$FGRP` parameter ('G' or 'Z').
  - The file is cleared (`CLRPFM`) and overridden (`OVRDBF`) to ensure it is job-specific and empty before use.
- **No Direct Access to Other Tables**: The `FR710C` program itself does not directly access other database files (e.g., `glcont`, `gglcont`, or `zglcont`). Any such access is handled by `FR710A` or `FR710B`.

### External Programs Called
- **FR710A**: Called to build the `FR710W` work file by retrieving and processing data based on the company number, report date, and file group.
- **FR710B**: Called to generate and print the Freight Out Accrual Report using the data in the `FR710W` file.
- **CHKOBJ, CRTDUPOBJ, CLRPFM, OVRDBF (System Commands)**: These are system commands used to manage the `FR710W` file:
  - `CHKOBJ`: Checks if `FR710W` exists in `QTEMP`.
  - `CRTDUPOBJ`: Creates a copy of `FR710W` in `QTEMP` from the appropriate source library.
  - `CLRPFM`: Clears the contents of `FR710W`.
  - `OVRDBF`: Overrides references to `FR710W` to point to `QTEMP/FR710W`.

### Additional Notes
- **Revisions**:
  - **JB01 (10/8/2011)**: Added multi-user capability by creating the work file in `QTEMP`, ensuring each job has its own copy of `FR710W`.
  - **JB02 (6/6/2012)**: Updated the work file creation to copy from `ARGDEV` or `ARGDEVTEST` (later revised to `DATA` and `DATADEV`).
  - **jK01 (8/30/2021)**: Replaced `ARGDEV` with `DATA` and `ARGDEVTEST` with `DATADEV`, and explicitly overrode `FR710W` to `QTEMP/FR710W` to reinforce job-specific file usage.
- **Minimal Error Handling**: The program handles the absence of `FR710W` but does not include comprehensive error checking for the calls to `FR710A` or `FR710B`.
- **Role as Coordinator**: `FR710C` is a lightweight program that orchestrates the report generation process by managing the work file and delegating data processing and printing to `FR710A` and `FR710B`, respectively.

This program ensures that the temporary work file is properly set up and that the report generation process is executed in the correct sequence, supporting multi-user environments and different data sources based on the file group.