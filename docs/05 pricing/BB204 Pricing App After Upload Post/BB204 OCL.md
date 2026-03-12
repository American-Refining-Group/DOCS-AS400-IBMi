The provided document is an **OCL (Operation Control Language)** script used on IBM AS/400 (now IBM i) systems to control the execution of programs and manage files. This script, named `BB204.ocl36`, outlines a process for running a program called `BB204` with specific file operations and conditional checks. Below, I’ll explain the process steps, list the external programs called, and identify the tables (files) used.

---

### Process Steps of the RPG .OCL Program

The OCL script orchestrates a sequence of operations to prepare, execute, and clean up after running the `BB204` program. Here’s a step-by-step explanation of the process:

1. **Header and Initial Setup**:
   - The script begins with a comment block describing its purpose: to run the `BB204` process, create a temporary table (`CTABLE`) from a workstation file into `QTEMP`, delete an existing `BB204W` file in `QTEMP`, and add an override for the `BB203W` file to `QTEMP`.
   - The `?9?` placeholder is a parameter passed to the script, likely representing a specific library, file, or job identifier.

2. **SCPROCP Command**:
   - `// SCPROCP ,,,,,,,,?9?`: This appears to be a custom command or job step that initializes the process, passing the `?9?` parameter. The exact functionality depends on how `SCPROCP` is defined in the system.

3. **GSY2K Command**:
   - `// GSY2K`: This is likely a command to set up or verify the system environment (possibly related to date handling or Y2K compliance). Its exact purpose depends on the system's configuration.

4. **Conditional Check on `BB204T`**:
   - `// IF ?F'A,?9?BB204T'?/0000000  PAUSE 'OPTION 17 DID NOT CREATE RECORDS TO BE POSTED'`: This checks if the file `?9?BB204T` (with the library prefix replaced by `?9?`) exists and contains records. If no records are found (`/0000000`), the script pauses and displays the message "OPTION 17 DID NOT CREATE RECORDS TO BE POSTED."
   - `// IF ?F'A,?9?BB204T'?/0000000  CANCEL`: If the same condition is true (no records), the script cancels the job entirely. This ensures the process only continues if `BB204T` has records to process.

5. **File Creation in `QTEMP`**:
   - `CRTDUPOBJ OBJ(?9?BB204?WS?) FROMLIB(*LIBL) OBJTYPE(*FILE) TOLIB(QTEMP) NEWOBJ(BB204W) DATA(*YES)`: Creates a duplicate of the file `?9?BB204?WS?` (or `?9?BB204WS` in the next line, suggesting a possible typo or variant) from the library list (`*LIBL`) into the `QTEMP` library, naming it `BB204W`. The `DATA(*YES)` parameter copies the file’s data along with its structure.
   - This step prepares a temporary working file for the `BB204` program.

6. **File Override**:
   - `OVRDBF FILE(BB204W) TOFILE(QTEMP/BB204W)`: Overrides the file `BB204W` to point to the newly created `QTEMP/BB204W`. This ensures the `BB204` program uses the temporary file in `QTEMP` instead of the original file.

7. **Call to `BB204` Program**:
   - `CALL BB204 PARM(?9?)`: Executes the `BB204` program, passing the `?9?` parameter. This is the main program that processes the data in the files prepared earlier.

8. **Cleanup Operations**:
   - `DLTF QTEMP/BB204W`: Deletes the temporary `BB204W` file from `QTEMP` after processing.
   - `DLTF *LIBL/?9?BB204?WS?` and `DLTF *LIBL/?9?BB204WS`: Deletes the original workstation file (again, possibly handling a naming variation). These steps ensure no residual files remain in the library list.
   - `DLTOVR FILE(BB204W)`: Removes the file override for `BB204W`, cleaning up the environment.
   - `CLRPFM FILE(?9?BB204T)`: Clears all records from the `?9?BB204T` file, preparing it for the next run.
   - `CLRPFM FILE(?9?PRSABLO)`: Clears all records from the `?9?PRSABLO` file, likely another output or temporary file used in the process.

9. **Commented-Out Program Calls**:
   - The script ends with commented-out references to programs `BIFX43`, `BIFX44`, `BI944B`, `BI942E`, and `PRICES`, each with the `?9?` parameter. These are likely "fix" programs for data correction or additional processing, but they are not currently executed (indicated by `//`).
   - The comment `** DO WE STILL NEED THESE TWO FIX PROGRAMS ?` suggests uncertainty about whether `BIFX43` and `BIFX44` are still required, indicating potential obsolescence or review.

---

### External Programs Called

The script explicitly calls or references the following external programs:
1. **BB204**: The main program executed via `CALL BB204 PARM(?9?)`.
2. **BIFX43** (commented out): A potential fix program, not currently executed.
3. **BIFX44** (commented out): Another potential fix program, not currently executed.
4. **BI944B** (commented out): Likely a batch or correction program, not executed.
5. **BI942E** (commented out): Likely another correction or processing program, not executed.
6. **PRICES** (commented out): Likely a program for pricing updates or calculations, not executed.

Note: The commented-out program `BB204AC` (`CALL PGM(BB204AC) PARM('?9?')`) suggests it was once part of the process but is no longer active.

---

### Tables (Files) Used

The script references the following files:
1. **?9?BB204T**: A primary table checked for records and cleared at the end. Likely an output or transaction file.
2. **?9?BB204?WS?** or **?9?BB204WS**: A workstation file duplicated into `QTEMP` as `BB204W`. This is the input or working file for the `BB204` program.
3. **QTEMP/BB204W**: A temporary file created in `QTEMP` to hold the duplicated data from `?9?BB204?WS?` or `?9?BB204WS`.
4. **?9?PRSABLO**: A file cleared at the end, likely an output or log file related to the process.
5. **BB203W** (mentioned in comments): Referenced in the initial comment about adding an override to `QTEMP`, but not explicitly used in the script.

---

### Summary

The `BB204.ocl36` script is a control procedure that:
- Verifies the presence of records in `?9?BB204T` and halts if none are found.
- Creates a temporary copy of the workstation file (`?9?BB204?WS?` or `?9?BB204WS`) in `QTEMP` as `BB204W`.
- Overrides the file reference to use the temporary file.
- Calls the `BB204` program with the `?9?` parameter to process the data.
- Cleans up by deleting temporary files, removing overrides, and clearing `?9?BB204T` and `?9?PRSABLO`.
- Includes commented-out references to additional programs (`BIFX43`, `BIFX44`, etc.) that may no longer be needed.

**External Programs Called**: `BB204`, and potentially `BIFX43`, `BIFX44`, `BI944B`, `BI942E`, `PRICES` (commented out).
**Tables Used**: `?9?BB204T`, `?9?BB204?WS?`/`?9?BB204WS`, `QTEMP/BB204W`, `?9?PRSABLO`, and `BB203W` (mentioned but not used).