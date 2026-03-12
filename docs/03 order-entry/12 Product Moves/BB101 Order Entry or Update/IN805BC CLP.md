The `IN805BC.clp.txt` document is a CLP (Control Language Program) written for an IBM System/36 or AS/400 (now IBM i) environment, called from the `BB101.ocl36.txt` OCL program. Its purpose is to build work files (`INWZHW` and `INWZ10W`) in the `QTEMP` library to accumulate totals for tanks in the work file `IN805W`. The program was written by Jimmy Krajacic on 03/14/2018 and revised on 08/30/21 (revision JK01) to update library references and copy files to `QTEMP`. Below is a detailed explanation of the process steps, business rules, tables (files) used, and any external programs called.

---

### Process Steps of the CLP Program

The `IN805BC` CLP program creates and populates temporary work files (`INWZHW` and `INWZ10W`) in the `QTEMP` library by copying data from source files based on input parameters. The steps are as follows:

1. **Program Initialization**:
   - The program accepts two parameters:
     - `&P$FGRP` (1 character): File group identifier (e.g., 'G' or other values).
     - `&P$CO#` (2 characters): Company number.
   - Declares variables:
     - `&WRKFIL1` (10 characters): Name of the first source file, constructed as `&P$FGRP + 'INWZH'`.
     - `&WRKFIL2` (10 characters): Name of the second source file, constructed as `&P$FGRP + 'INWZ' + &P$CO#`.

2. **Construct File Names**:
   - Concatenates `&P$FGRP` with `'INWZH'` to form `&WRKFIL1` (e.g., 'GINWZH' for `&P$FGRP='G'`).
   - Concatenates `&P$FGRP`, `'INWZ'`, and `&P$CO#` to form `&WRKFIL2` (e.g., 'GINWZ01' for `&P$FGRP='G'` and `&P$CO#='01'`).

3. **Clear or Create Work File `INWZHW`**:
   - Attempts to clear the physical file `QTEMP/INWZHW` using `CLRPFM`.
   - If the file does not exist (`CPF3142` message), executes a `DO` block:
     - If `&P$FGRP = 'G'`:
       - Creates a duplicate object of `INWZHW` from the `DATA` library to `QTEMP` using `CRTDUPOBJ`, with constraints (`CST(*NO)`) and triggers (`TRG(*NO)`) disabled.
       - Monitors for errors `CPF5813` (template file locked) and `CPF7302` (constraint error).
     - If `&P$FGRP ≠ 'G'`:
       - Creates a duplicate object of `INWZHW` from the `DATADEV` library to `QTEMP` with the same settings.
       - Monitors for the same errors.

4. **Clear or Create Work File `INWZ10W`**:
   - Attempts to clear the physical file `QTEMP/INWZ10W` using `CLRPFM`.
   - If the file does not exist (`CPF3142` message), executes a `DO` block:
     - If `&P$FGRP = 'G'`:
       - Creates a duplicate object of `INWZ10W` from the `DATA` library to `QTEMP` with constraints and triggers disabled.
       - Monitors for errors `CPF5813` and `CPF7302`.
     - If `&P$FGRP ≠ 'G'`:
       - Creates a duplicate object of `INWZ10W` from the `DATADEV` library to `QTEMP` with the same settings.
       - Monitors for the same errors.

5. **Copy Data to Work Files**:
   - Copies data from the source file `&WRKFIL1` (e.g., `GINWZH`) to `QTEMP/INWZHW` using `CPYF` with:
     - `MBROPT(*REPLACE)`: Replaces existing data in the target file.
     - `FMTOPT(*NOCHK)`: Bypasses format checking for faster copying.
     - Monitors for error `CPF2817` (copy failed, e.g., if source file is empty or does not exist).
   - Copies data from the source file `&WRKFIL2` (e.g., `GINWZ01`) to `QTEMP/INWZ10W` using `CPYF` with the same options and error monitoring.

6. **Program Termination**:
   - The program ends with `ENDPGM` after completing the file creation and data copy operations.

---

### Business Rules

The program enforces the following business rules:

1. **Dynamic File Naming**:
   - Source file names are dynamically constructed using `&P$FGRP` and `&P$CO#` to ensure the correct files are accessed (e.g., `GINWZH`, `GINWZ01`).
   - This allows the program to handle different file groups and company-specific data.

2. **Library Selection Based on File Group**:
   - If `&P$FGRP = 'G'`, source files are copied from the `DATA` library (production environment).
   - If `&P$FGRP ≠ 'G'`, source files are copied from the `DATADEV` library (development or test environment, per revision JK01).

3. **Temporary Work Files**:
   - Work files (`INWZHW`, `INWZ10W`) are created in the `QTEMP` library, which is temporary and unique to the job, ensuring isolation and cleanup upon job completion.
   - If the files do not exist, they are created by duplicating objects from the appropriate library (`DATA` or `DATADEV`).

4. **Error Handling**:
   - The program handles missing files (`CPF3142`) by creating them dynamically.
   - It monitors for file copy errors (`CPF2817`) and object creation errors (`CPF5813`, `CPF7302`) to ensure robust execution without abending.
   - Constraints and triggers are disabled (`CST(*NO)`, `TRG(*NO)`) to simplify file creation and avoid conflicts.

5. **Data Replacement**:
   - The `CPYF` command uses `MBROPT(*REPLACE)` to overwrite any existing data in `INWZHW` and `INWZ10W`, ensuring the work files contain the latest data.
   - `FMTOPT(*NOCHK)` bypasses format validation, assuming the source and target file formats are compatible.

---

### Tables (Files) Used

The program interacts with the following files:

1. **INWZHW** (in `QTEMP`):
   - Temporary work file created or cleared to store data copied from `&WRKFIL1` (e.g., `GINWZH`).
   - Source library: `DATA` (if `&P$FGRP = 'G'`) or `DATADEV` (if `&P$FGRP ≠ 'G'`).
2. **INWZ10W** (in `QTEMP`):
   - Temporary work file created or cleared to store data copied from `&WRKFIL2` (e.g., `GINWZ01`).
   - Source library: `DATA` (if `&P$FGRP = 'G'`) or `DATADEV` (if `&P$FGRP ≠ 'G'`).
3. **&WRKFIL1** (e.g., `GINWZH`):
   - Source file in `DATA` or `DATADEV`, dynamically named using `&P$FGRP + 'INWZH'`.
4. **&WRKFIL2** (e.g., `GINWZ01`):
   - Source file in `DATA` or `DATADEV`, dynamically named using `&P$FGRP + 'INWZ' + &P$CO#`.

---

### External Programs Called

The `IN805BC` CLP program does not call any external programs. It relies entirely on CL commands (`CHGVAR`, `CLRPFM`, `CRTDUPOBJ`, `CPYF`) for its operations.

---

### Summary

The `IN805BC` CLP program, called from the `BB101.ocl36.txt` OCL program, builds temporary work files (`INWZHW` and `INWZ10W`) in the `QTEMP` library to accumulate tank totals for the `IN805W` work file. It constructs source file names using input parameters `&P$FGRP` and `&P$CO#`, clears or creates the target files, and copies data from source files in the `DATA` (production) or `DATADEV` (development) library based on the file group. Business rules ensure dynamic file naming, appropriate library selection, error handling, and data replacement in temporary files. The program interacts with four files (`INWZHW`, `INWZ10W`, and their source files) and does not call external programs, relying on internal CL commands for all processing.