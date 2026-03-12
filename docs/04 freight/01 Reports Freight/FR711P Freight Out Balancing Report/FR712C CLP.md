The CLP program `FR712C` is a control program called by `FR711PC` to build a work file and generate the Freight Out Reconciliation Report sorted by carrier/routing. It is structurally similar to `FR711C` but focuses on producing a report sorted by carrier rather than location. Below is an explanation of the process steps, business rules, tables used, and external programs called, based on the provided source code.

---

### Process Steps of the CLP Program (`FR712C`)

The program receives parameters from `FR711PC`, creates or clears a temporary work file, populates it with data, and calls a print program to generate the report. Here are the detailed steps:

1. **Program Declaration and Parameter Receipt**:
   - **Purpose**: Defines the program and its input parameters.
   - **Actions**:
     - Declares the program with seven input parameters:
       - `&P$CO`: Company code (2 characters).
       - `&P$RDAT`: Report date (8 characters).
       - `&P$LOC`: Location code (3 characters).
       - `&P$CAID`: Carrier ID (6 characters).
       - `&P$SORT`: Sort option (1 character, expected to be 'C' for carrier/routing since this program is called when sorting by carrier).
       - `&P$SELMO`: Month selection (1 character, added in revision `jb03`).
       - `&P$FGRP`: File group (1 character, 'G' or 'Z').

2. **Create Work File (`JB01` and `JK01` Revisions)**:
   - **Purpose**: Ensures the existence of a temporary work file (`FR712W`) in the `QTEMP` library for multi-user capability.
   - **Actions**:
     - Checks if `FR712W` exists in `QTEMP` using `CHKOBJ`.
     - If the file does not exist (`CPF9801` message), creates a duplicate of `FR712W` using `CRTDUPOBJ`:
       - If `&P$FGRP = 'G'`, copies `FR712W` from the `DATA` library (revised from `ARGDEV` in `JK01`).
       - If `&P$FGRP = 'Z'`, copies `FR712W` from the `DATADEV` library (revised from `ARGDEVTEST` in `JK01`).
       - The duplicate is created in `QTEMP` with constraints (`CST(*NO)`) and triggers (`TRG(*NO)`) disabled.
     - Clears the `FR712W` file in `QTEMP` using `CLRPFM` to ensure it is empty (revised in `JK01` to explicitly reference `QTEMP/FR712W`).
     - Overrides the `FR712W` file to point to `QTEMP/FR712W` using `OVRDBF` for subsequent operations (revised in `JK01`).

3. **Build Work File**:
   - **Purpose**: Populates the temporary work file with data for the report.
   - **Actions**:
     - Calls the program `FR712A`, passing six parameters: `&P$CO`, `&P$RDAT`, `&P$LOC`, `&P$CAID`, `&P$SELMO`, and `&P$FGRP`.
     - `FR712A` is responsible for querying the necessary data (likely from `gglcont`/`ginloc` or `zglcont`/`zinloc` based on `&P$FGRP`) and writing it to `FR712W`, with data organized for sorting by carrier.

4. **Call Print Program**:
   - **Purpose**: Generates the Freight Out Reconciliation Report using the data in the work file.
   - **Actions**:
     - Calls the program `FR712B`, passing all seven parameters: `&P$CO`, `&P$RDAT`, `&P$LOC`, `&P$CAID`, `&P$SORT`, `&P$SELMO`, and `&P$FGRP`.
     - `FR712B` uses the data in `FR712W` to produce the report, sorted by carrier/routing (as `&P$SORT = 'C'`).

5. **Program Termination**:
   - **Purpose**: Ends the program after processing.
   - **Actions**:
     - The program ends with `ENDPGM` after calling `FR712A` and `FR712B`.

---

### Business Rules

- **Work File Management**:
  - The temporary work file `FR712W` is created in `QTEMP` to ensure multi-user capability (revision `JB01`). Using `QTEMP` provides a unique, job-specific file instance, preventing conflicts between concurrent users.
  - The file is copied from either `DATA` (for `&P$FGRP = 'G'`) or `DATADEV` (for `&P$FGRP = 'Z'`), as updated in revision `JK01`, ensuring the correct file structure is used based on the file group.
  - The file is cleared (`CLRPFM`) before use to ensure no residual data affects the report.
  - The `OVRDBF` command ensures that all references to `FR712W` in `FR712A` and `FR712B` point to the `QTEMP` copy.

- **Month Selection** (from revision `jb03`):
  - The `&P$SELMO` parameter supports the following values, as defined in `FR711P`:
    - `M`: Selected month only, including all (open and closed) records.
    - `O`: Selected month, open records only.
    - `C`: Selected month, closed records only.
    - `A`: All open records from previous and selected months.
    - Blank: Previous month open records and selected month all records.
  - This parameter is passed to `FR712A` and `FR712B` to filter the data included in the work file and report.

- **File Group**:
  - The `&P$FGRP` parameter ('G' or 'Z') determines the source library for the work file and likely influences the data files used by `FR712A` (e.g., `gglcont`/`ginloc` for 'G', `zglcont`/`zinloc` for 'Z').

- **Sort Option**:
  - The program assumes `&P$SORT = 'C'` (sort by carrier/routing), as it is called by `FR711PC` only when this condition is met. The `&P$SORT` parameter is passed to `FR712B` to ensure the report is sorted appropriately.

- **Error Handling**:
  - The program handles the case where `FR712W` does not exist in `QTEMP` by creating it. Other errors (e.g., file access issues) are not explicitly handled in this program and are likely managed by `FR712A` or `FR712B`.

- **Revision History**:
  - `JB01` (06/06/2012): Revised the source libraries for the work file to `ARGDEV` or `ARGDEVTEST`.
  - `jb03` (04/07/15): Added the `&P$SELMO` parameter for month selection filtering.
  - `JK01` (08/30/21): Updated library references to `DATA` and `DATADEV`, and explicitly overrode `FR712W` to `QTEMP/FR712W`.

---

### Tables Used

- **Work File**:
  - `FR712W`: A temporary physical file in the `QTEMP` library, used to store data for the report. It is created by copying from:
    - `DATA/FR712W` if `&P$FGRP = 'G'`.
    - `DATADEV/FR712W` if `&P$FGRP = 'Z'`.
  - The file is cleared before use and overridden to ensure `FR712A` and `FR712B` access the `QTEMP` copy.

- **No Direct Data File Access**:
  - `FR712C` does not directly access data files like `gglcont`, `ginloc`, `zglcont`, or `zinloc`. These are likely accessed by `FR712A` to populate `FR712W`, as set up by the overrides in `FR711P`.

---

### External Programs Called

- **FR712A**: Called to build the work file `FR712W` by querying data based on the provided parameters (`&P$CO`, `&P$RDAT`, `&P$LOC`, `&P$CAID`, `&P$SELMO`, `&P$FGRP`).
- **FR712B**: Called to generate the Freight Out Reconciliation Report from the data in `FR712W`, using all seven parameters.
- **System Commands**:
  - `CHKOBJ`: Checks for the existence of `FR712W` in `QTEMP`.
  - `CRTDUPOBJ`: Creates a duplicate of `FR712W` in `QTEMP` if it does not exist.
  - `CLRPFM`: Clears the `FR712W` file in `QTEMP`.
  - `OVRDBF`: Overrides the `FR712W` file to point to `QTEMP/FR712W`.

---

### Additional Notes

- **Purpose**: `FR712C` acts as a coordinator, managing the creation and population of a temporary work file and invoking the print program to produce the report sorted by carrier/routing.
- **Multi-User Support**: The use of `QTEMP` (revision `JB01`) ensures that each job has its own instance of `FR712W`, preventing conflicts in a multi-user environment.
- **Parameter Validation**: The program assumes that input parameters are valid, as they are validated by `FR711P` before being passed through `FR711PC`.
- **Execution Context**: The program runs in the same job session as its caller, using direct calls to `FR712A` and `FR712B` rather than submitting them as batch jobs.
- **Comparison with `FR711C`**: `FR712C` is nearly identical to `FR711C` in structure, differing only in the work file (`FR712W` vs. `FR711W`), the called programs (`FR712A`/`FR712B` vs. `FR711A`/`FR711B`), and the sort order (carrier vs. location).

This program efficiently sets up the environment for generating the Freight Out Reconciliation Report sorted by carrier, leveraging a temporary work file and specialized programs for data processing and printing.