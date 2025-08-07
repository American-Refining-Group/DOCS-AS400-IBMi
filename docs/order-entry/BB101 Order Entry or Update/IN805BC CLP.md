The `IN805BC.clp.txt` is a CLP (Control Language Program) called by the `BB101.ocl36.txt` OCL program as part of the order entry system on an IBM AS/400 (iSeries) platform. Its purpose is to build and populate work files (`INWZHW` and `INWZ10W`) in the `QTEMP` library to accumulate totals for tanks, which are used by the `IN805W` work file for inventory processing. Below is an explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the IN805BC CLP Program

The `IN805BC` program creates and populates temporary work files based on input parameters. The process steps are as follows:

1. **Parameter Declaration and Input Processing**:
   - Declares input parameters:
     - `&P$FGRP` (1 character): File group identifier (e.g., 'G' for production or other values for development).
     - `&P$CO#` (2 characters): Company number.
   - Constructs work file names:
     - `&WRKFIL1` = `&P$FGRP` + 'INWZH' (e.g., 'GINWZH' for `&P$FGRP='G'`).
     - `&WRKFIL2` = `&P$FGRP` + 'INWZ' + `&P$CO#` (e.g., 'GINWZ10' for `&P$FGRP='G'` and `&P$CO#='10'`).

2. **Clear or Create Work File INWZHW**:
   - Attempts to clear the `INWZHW` file in `QTEMP` using `CLRPFM`.
   - If the file does not exist (`CPF3142` message):
     - If `&P$FGRP='G'`, creates a duplicate of `INWZHW` from the `DATA` library using `CRTDUPOBJ`, with no constraints or triggers.
     - If `&P$FGRP≠'G'`, creates a duplicate from the `DATADEV` library.
     - Ignores errors (`CPF5813`, `CPF7302`) during object creation.

3. **Clear or Create Work File INWZ10W**:
   - Attempts to clear the `INWZ10W` file in `QTEMP` using `CLRPFM`.
   - If the file does not exist (`CPF3142` message):
     - If `&P$FGRP='G'`, creates a duplicate of `INWZ10W` from the `DATA` library using `CRTDUPOBJ`.
     - If `&P$FGRP≠'G'`, creates a duplicate from the `DATADEV` library.
     - Ignores errors (`CPF5813`, `CPF7302`) during object creation.

4. **Copy Data to Work Files**:
   - Copies data from the source file `&WRKFIL1` (e.g., `GINWZH`) to `QTEMP/INWZHW` using `CPYF`, replacing existing records (`MBROPT(*REPLACE)`) and bypassing field checks (`FMTOPT(*NOCHK)`).
     - Ignores copy errors (`CPF2817`).
   - Copies data from the source file `&WRKFIL2` (e.g., `GINWZ10`) to `QTEMP/INWZ10W` using `CPYF`, with the same options.
     - Ignores copy errors (`CPF2817`).

5. **Completion**:
   - Ends the program after creating and populating the work files.

---

### Business Rules

1. **File Naming**:
   - Work file names are dynamically constructed using `&P$FGRP` and `&P$CO#` to ensure uniqueness per company and environment (e.g., `GINWZH`, `GINWZ10`).

2. **Environment-Based File Creation**:
   - For production (`&P$FGRP='G'`), source files are copied from the `DATA` library.
   - For development or testing (`&P$FGRP≠'G'`), source files are copied from the `DATADEV` library.

3. **File Management**:
   - Work files are created in `QTEMP` to ensure temporary, job-specific storage.
   - Existing work files are cleared; non-existent files are created as duplicates of source files without constraints or triggers.

4. **Data Copying**:
   - Data is copied with replacement (`MBROPT(*REPLACE)`) and no field validation (`FMTOPT(*NOCHK)`) to ensure compatibility with S/36 files.
   - Copy errors are ignored to allow processing to continue.

5. **Error Handling**:
   - Monitors for file not found (`CPF3142`), object creation errors (`CPF5813`, `CPF7302`), and copy errors (`CPF2817`), proceeding without interruption.

---

### Tables (Files) Used

The program interacts with the following files:

1. **INWZHW**: Temporary work file in `QTEMP` for accumulating tank totals (created or cleared).
2. **INWZ10W**: Temporary work file in `QTEMP` for company-specific tank totals (created or cleared).
3. **Source Files**:
   - `&WRKFIL1` (e.g., `GINWZH`): S/36 work file in `DATA` or `DATADEV`, source for `INWZHW`.
   - `&WRKFIL2` (e.g., `GINWZ10`): S/36 work file in `DATA` or `DATADEV`, source for `INWZ10W`.

---

### External Programs Called

The `IN805BC` program does not call any external programs. It relies on CL commands (`CLRPFM`, `CRTDUPOBJ`, `CPYF`) to perform its tasks.

---

### Summary

- **Process Overview**: The `IN805BC` CLP program creates and populates temporary work files (`INWZHW`, `INWZ10W`) in `QTEMP` by copying data from S/36 files (`&WRKFIL1`, `&WRKFIL2`) based on file group and company number. It clears existing files or creates new ones from `DATA` or `DATADEV` libraries.
- **Business Rules**: Constructs file names dynamically, uses environment-specific source libraries (`DATA` for production, `DATADEV` for development), creates files in `QTEMP`, and copies data without validation. Ignores errors to ensure continuation.
- **Files/Tables**: Uses `INWZHW` and `INWZ10W` in `QTEMP`, with source files like `GINWZH` and `GINWZ10` from `DATA` or `DATADEV`.
- **External Programs**: None called.

This program supports inventory processing by preparing work files for tank total accumulation, ensuring compatibility with the order entry system’s inventory checks.