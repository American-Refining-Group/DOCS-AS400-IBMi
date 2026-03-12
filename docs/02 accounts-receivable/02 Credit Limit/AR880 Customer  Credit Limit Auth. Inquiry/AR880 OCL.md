The provided document is an OCL (Operation Control Language) program, specifically `AR880.ocl36.txt`, used on IBM AS/400 or iSeries systems to manage operations for a Customer Credit Limit Authorization Inquiry. OCL is a scripting language used to control program execution, file handling, and system operations on these systems. Below, I’ll explain the process steps of the program, list the external programs called, and identify the tables (files) used.

### Process Steps of the RPG .OCL Program

The OCL program orchestrates the execution of a credit limit authorization inquiry by setting up the environment, overriding database files, loading files, and calling a program. Here’s a step-by-step breakdown of the process:

1. **Delete Overrides for All Files**:
   - `// DLTOVR FILE(*ALL)`: Deletes all existing file overrides to ensure a clean slate for file redirections. This prevents conflicts from prior overrides.

2. **Call Initial Program**:
   - `// CALL PGM(GSGENIEC)`: Calls the program `GSGENIEC`. This is likely a general-purpose initialization or environment setup program. It may perform tasks like setting up user-specific parameters or validating the environment.

3. **Conditional Return Based on Screen Input**:
   - `// IFF ?L'506,3'?/YES RETURN`: Checks a specific screen location (likely a field at line 506, position 3) for a value of `YES`. If true, the program terminates immediately with a `RETURN` statement, halting further execution.

4. **Procedure Call with Parameter**:
   - `// SCPROCP ,,,,,,,,?9?`: Invokes a stored procedure or command with a parameter `?9?`. The commas indicate placeholder parameters, and `?9?` is likely a dynamic value (e.g., a company code or environment identifier) passed to the procedure. The exact purpose depends on the system configuration.

5. **Year 2000 Compliance**:
   - `// GSY2K`: Executes a command or program related to Year 2000 compliance, ensuring date-related operations are handled correctly. This was common in legacy systems to address Y2K issues.

6. **Set Local Variables**:
   - `// LOCAL OFFSET-200,DATA-'        '`: Initializes a local variable at offset 200 with 8 blank spaces. This could be used to clear a specific data area or memory location.
   - `// LOCAL OFFSET-480,DATA-'?9?'`: Sets a local variable at offset 480 with the value of `?9?`, likely a dynamic parameter like a company code or environment identifier.

7. **Set User Information**:
   - `// LOCAL OFFSET-103,DATA-'?USER?'`: Stores the current user ID at offset 103, likely for audit or display purposes in the inquiry process.

8. **Override Database Files**:
   - `OVRDBF FILE(BBCSR) TOFILE(QS36F/GBBCSR)`: Overrides the logical file `BBCSR` to point to the physical file `QS36F/GBBCSR`. This ensures the program uses the correct physical file in the `QS36F` library.
   - `OVRDBF FILE(BBSLSM) TOFILE(QS36F/GBBSLSM)`: Similarly, overrides the logical file `BBSLSM` to `QS36F/GBBSLSM`.

9. **Load Database Files**:
   - The program loads several files with the `DISP-SHRMM` (shared, multiple-member) disposition, allowing shared access to the files:
     - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHRMM`: Loads the `ARCONT` file (likely accounts receivable control file) with a label prefixed by `?9?`.
     - `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHRMM`: Loads the `ARCUST` file (customer master file).
     - `// FILE NAME-ARCUSP,LABEL-?9?ARCUSP,DISP-SHRMM`: Loads the `ARCUSP` file (possibly customer-specific data or preferences).
     - `// FILE NAME-BBORCL,LABEL-?9?BBORCL,DISP-SHRMM`: Loads the `BBORCL` file (credit limit or order control file).
     - `// FILE NAME-ARCLGR,LABEL-?9?ARCLGR,DISP-SHRMM`: Loads the `ARCLGR` file (accounts receivable ledger or general ledger interface).
     - `// FILE NAME-BBORCLAU,LABEL-?9?BBORCL,DISP-SHRMM`: Loads the `BBORCLAU` file, pointing to the same physical file as `BBORCL`.

10. **Printer Configuration (Commented Out)**:
    - `** PRINTER NAME-JBLIST,DEVICE-PJ,FORMSNO-JBCL,PRIORITY-0`: This commented line suggests a printer setup for a report named `JBLIST`, but it’s not active in this version.

11. **Override Printer Files (Conditional)**:
    - `// IF ?9?/G OVRPRTF FILE(CREMAL) OUTQ(CSROUTQ)`: If `?9?` equals `G`, overrides the `CREMAL` printer file to output to the `CSROUTQ` queue.
    - `// IF ?9?/G OVRPRTF FILE(SMEMAL) OUTQ(SLMNOUTQ)`: Similarly, overrides the `SMEMAL` printer file to `SLMNOUTQ`.
    - `// IFF ?9?/G OVRPRTF FILE(CREMAL) OUTQ(TESTOUTQ)`: If `?9?` equals `G`, overrides `CREMAL` to `TESTOUTQ` (for testing purposes).
    - `// IFF ?9?/G OVRPRTF FILE(SMEMAL) OUTQ(TESTOUTQ)`: Similarly, overrides `SMEMAL` to `TESTOUTQ`.

12. **Run the Program**:
    - `// RUN`: Executes the main program (likely `AR880`, implied by the file name). This program performs the core credit limit authorization inquiry logic using the loaded files.

13. **Commented Goto Statement**:
    - `***GOTO AGN`: A commented-out line that would loop back to a label `AGN` if active, suggesting a potential loop for repeated processing (not used here).

14. **End Tag**:
    - `// TAG END`: Marks the end of the main processing block.

15. **Clear Local Variables**:
    - `// LOCAL BLANK-*ALL`: Clears all local variables, resetting the data areas used during execution.

16. **Reset Switches**:
    - `// SWITCH 00000000`: Resets all system switches to `0`, ensuring a clean state for subsequent operations.

17. **Conditional Call to Test Program**:
    - `// IF ?9?/G CALL AR880TC`: If `?9?` equals `G`, calls the program `AR880TC`, likely a test version of the credit limit authorization program.

18. **Exit Tag**:
    - `// TAG OUT`: Marks the exit point of the program, indicating completion.

### External Programs Called

The OCL program explicitly calls the following external programs:
1. **GSGENIEC**: Likely an initialization or environment setup program, called at the start.
2. **AR880TC**: A test version of the credit limit authorization program, called conditionally if `?9?` equals `G`.

Additionally, the `// RUN` statement implies the execution of a program named `AR880` (based on the file name `AR880.ocl36.txt`), though it’s not explicitly listed in a `CALL` statement.

### Tables (Files) Used

The program references the following database files (tables), all loaded with a `DISP-SHRMM` disposition, indicating shared access to multi-member files:
1. **ARCONT**: Accounts receivable control file, prefixed with `?9?`.
2. **ARCUST**: Customer master file, prefixed with `?9?`.
3. **ARCUSP**: Customer-specific data or preferences file, prefixed with `?9?`.
4. **BBORCL**: Credit limit or order control file, prefixed with `?9?`.
5. **ARCLGR**: Accounts receivable ledger or general ledger interface file, prefixed with `?9?`.
6. **BBORCLAU**: Likely an alias for `BBORCL`, pointing to the same physical file, prefixed with `?9?`.
7. **BBCSR**: Customer service representative file, overridden to `QS36F/GBBCSR`.
8. **BBSLSM**: Salesman file, overridden to `QS36F/GBBSLSM`.

Additionally, two printer files are referenced for output:
1. **CREMAL**: Output queue overridden to `CSROUTQ` or `TESTOUTQ` (conditional on `?9?`).
2. **SMEMAL**: Output queue overridden to `SLMNOUTQ` or `TESTOUTQ` (conditional on `?9?`).

### Summary

The `AR880.ocl36.txt` OCL program manages a Customer Credit Limit Authorization Inquiry by:
- Initializing the environment (`GSGENIEC`).
- Setting up dynamic parameters and user data.
- Overriding and loading necessary database files for customer and credit data.
- Conditionally configuring output queues for reports.
- Executing the main inquiry logic (`AR880`) and, if in a specific environment (`?9?=G`), a test program (`AR880TC`).
- Cleaning up variables and switches after execution.

**External Programs Called**: `GSGENIEC`, `AR880TC` (conditional), and implicitly `AR880`.
**Tables Used**: `ARCONT`, `ARCUST`, `ARCUSP`, `BBORCL`, `ARCLGR`, `BBORCLAU`, `BBCSR`, `BBSLSM`, with printer files `CREMAL` and `SMEMAL`.