The CLP program `FR711C` is a control program called by `FR711PC` to build a work file and generate the Freight Out Reconciliation Report sorted by location. It orchestrates the creation of a temporary work file, populates it with data, and then calls a print program to produce the report. Below is an explanation of the process steps, business rules, tables used, and external programs called, based on the provided source code.

---

### Process Steps of the CLP Program (`FR711C`)

The program receives parameters from `FR711PC`, creates or clears a temporary work file, builds data in the work file, and calls a print program to generate the report. Here are the detailed steps:

1. **Program Declaration and Parameter Receipt**:
   - **Purpose**: Defines the program and its input parameters.
   - **Actions**:
     - Declares the program with seven input parameters:
       - `&P$CO`: Company code (2 characters).
       - `&P$RDAT`: Report date (8 characters).
       - `&P$LOC`: Location code (3 characters).
       - `&P$CAID`: Carrier ID (6 characters).
       - `&P$SORT`: Sort option (1 character, expected to be 'L' for location since this program is called when sorting by location).
       - `&P$SELMO`: Month selection (1 character, added in revision `jb03`).
       - `&P$FGRP`: File group (1 character, 'G' or 'Z').

2. **Create Work File (`JB01` and `JK01` Revisions)**:
   - **Purpose**: Ensures the existence of a temporary work file (`FR711W`) in the `QTEMP` library for multi-user capability.
   - **Actions**:
     - Checks if `FR711W` exists in `QTEMP` using `CHKOBJ`.
     - If the file does not exist (`CPF9801` message), creates a duplicate of `FR711W` using `CRTDUPOBJ`:
       - If `&P$FGRP = 'G'`, copies `FR711W` from the `DATA` library (revised from `ARGDEV` in `JK01`).
       - If `&P$FGRP = 'Z'`, copies `FR711W` from the `DATADEV` library (revised from `ARGDEVTEST` in `JK01`).
       - The duplicate is created in `QTEMP` with constraints (`CST(*NO)`) and triggers (`TRG(*NO)`) disabled.
     - Clears the `FR711W` file in `QTEMP` using `CLRPFM` to ensure it is empty (revised in `JK01` to explicitly reference `QTEMP/FR711W`).
     - Overrides the `FR711W` file to point to `QTEMP/FR711W` using `OVRDBF` for subsequent operations (revised in `JK01`).

3. **Build Work File**:
   - **Purpose**: Populates the temporary work file with data for the report.
   - **Actions**:
     - Calls the program `FR711A`, passing six parameters: `&P$CO`, `&P$RDAT`, `&P$LOC`, `&P$CAID`, `&P$SELMO`, and `&P$FGRP`.
     - `FR711A` is responsible for querying the necessary data (likely from `gglcont`/`ginloc` or `zglcont`/`zinloc` based on `&P$FGRP`) and writing it to `FR711W`.

4. **Call Print Program**:
   - **Purpose**: Generates the Freight Out Reconciliation Report using the data in the work file.
   - **Actions**:
     - Calls the program `FR711B`, passing all seven parameters: `&P$CO`, `&P$RDAT`, `&P$LOC`, `&P$CAID`, `&P$SORT`, `&P$SELMO`, and `&P$FGRP`.
     - `FR711B` uses the data in `FR711W` to produce the report, sorted by location (as `&P$SORT = 'L'`).

5. **Program Termination**:
   - **Purpose**: Ends the program after processing.
   - **Actions**:
     - The program ends with `ENDPGM` after calling `FR711A` and `FR711B`.

---

### Business Rules

- **Work File Management**:
  - The temporary work file `FR711W` is created in `QTEMP` to ensure multi-user capability (revision `JB01`). Using `QTEMP` provides a unique, job-specific file instance, preventing conflicts between concurrent users.
  - The file is copied from either `DATA` (for `&P$FGRP = 'G'`) or `DATADEV` (for `&P$FGRP = 'Z'`), as updated in revision `JK01`, ensuring the correct file structure is used based on the file group.
  - The file is cleared (`CLRPFM`) before use to ensure no residual data affects the report.
  - The `OVRDBF` command ensures that all references to `FR711W` in `FR711A` and `FR711B` point to the `QTEMP` copy.

- **Month Selection** (from revision `jb03`):
  - The `&P$SELMO` parameter supports the following values, as defined in `FR711P`:
    - `M`: Selected month only, including all (open and closed) records.
    - `O`: Selected month, open records only.
    - `C`: Selected month, closed records only.
    - `A`: All open records from previous and selected months.
    - Blank: Previous month open records and selected month all records.
  - This parameter is passed to `FR711A` and `FR711B` to filter the data included in the work file and report.

- **File Group**:
  - The `&P$FGRP` parameter ('G' or 'Z') determines the source library for the work file and likely influences the data files used by `FR711A` (e.g., `gglcont`/`ginloc` for 'G', `zglcont`/`zinloc` for 'Z').

- **Sort Option**:
  - The program assumes `&P$SORT = 'L'` (sort by location), as it is called by `FR711PC` only when this condition is met. The `&P$SORT` parameter is passed to `FR711B` to ensure the report is sorted appropriately.

- **Error Handling**:
  - The program handles the case where `FR711W` does not exist in `QTEMP` by creating it. Other errors (e.g., file access issues) are not explicitly handled in this program and are likely managed by `FR711A` or `FR711B`.

- **Revision History**:
  - `JB01` (10/08/2011): Made the work file multi-user capable by creating it in `QTEMP`.
  - `JB02` (06/06/2012): Revised the source libraries for the work file to `ARGDEV` or `ARGDEVTEST`.
  - `jb03` (04/07/15): Added the `&P$SELMO` parameter for month selection filtering.
  - `JK01` (07/22/21): Updated library references to `DATA` and `DATADEV`, and explicitly overrode `FR711W` to `QTEMP/FR711W`.

---

### Tables Used

- **Work File**:
  - `FR711W`: A temporary physical file in the `QTEMP` library, used to store data for the report. It is created by copying from:
    - `DATA/FR711W` if `&P$FGRP = 'G'`.
    - `DATADEV/FR711W` if `&P$FGRP = 'Z'`.
  - The file is cleared before use and overridden to ensure `FR711A` and `FR711B` access the `QTEMP` copy.

- **No Direct Data File Access**:
  - `FR711C` does not directly access data files like `gglcont`, `ginloc`, `zglcont`, or `zinloc`. These are likely accessed by `FR711A` to populate `FR711W`, as set up by the overrides in `FR711P`.

---

### External Programs Called

- **FR711A**: Called to build the work file `FR711W` by querying data based on the provided parameters (`&P$CO`, `&P$RDAT`, `&P$LOC`, `&P$CAID`, `&P$SELMO`, `&P$FGRP`).
- **FR711B**: Called to generate the Freight Out Reconciliation Report from the data in `FR711W`, using all seven parameters.
- **System Commands**:
  - `CHKOBJ`: Checks for the existence of `FR711W` in `QTEMP`.
  - `CRTDUPOBJ`: Creates a duplicate of `FR711W` in `QTEMP` if it does not exist.
  - `CLRPFM`: Clears the `FR711W` file in `QTEMP`.
  - `OVRDBF`: Overrides the `FR711W` file to point to `QTEMP/FR711W`.

---

### Additional Notes

- **Purpose**: `FR711C` acts as a coordinator, managing the creation and population of a temporary work file and invoking the print program to produce the report sorted by location.
- **Multi-User Support**: The use of `QTEMP` (revision `JB01`) ensures that each job has its own instance of `FR711W`, preventing conflicts in a multi-user environment.
- **Parameter Validation**: The program assumes that input parameters are valid, as they are validated by `FR711P` before being passed through `FR711PC`.
- **Execution Context**: The program runs in the same job session as its caller, using direct calls to `FR711A` and `FR711B` rather than submitting them as batch jobs.
- **Temporary File Management**: The work file is created and cleared for each run, ensuring fresh data and avoiding residual data issues.

This program efficiently sets up the environment for report generation by managing the temporary work file and delegating data processing and printing to specialized programs (`FR711A` and `FR711B`).