The provided document is an Operation Control Language (OCL) program, `AR490P.ocl36.txt`, used in IBM midrange systems (e.g., AS/400 or System/36) to control job execution and interact with RPG programs. Below, I explain the process steps of the OCL program, list the external programs called, and identify the tables/files used.

---

### Process Steps of the OCL Program

OCL programs are used to define job control procedures, including loading programs, opening files, setting parameters, and controlling program flow. The steps in `AR490P.ocl36.txt` are as follows:

1. **Invoke a Program (`GSGENIEC`)**:
   - `// CALL PGM(GSGENIEC)`:
     - Calls the program `GSGENIEC`, likely a utility or initialization program.
     - This step may perform environment setup or validation before proceeding.

2. **Conditional Check on Location 506 (`?L'506,3'?/YES RETURN`)**:
   - `// IFF ?L'506,3'?/YES RETURN`:
     - Checks the value at location 506 (likely a system or program variable) for a length of 3 characters.
     - If the condition is true (`YES`), the program terminates early with a `RETURN` statement, halting further execution.
     - This acts as a gatekeeper to prevent unnecessary processing.

3. **Set Procedure Parameter (`SCPROCP`)**:
   - `// SCPROCP ,,,,,,,,?9?`:
     - Sets a procedure parameter, passing the value from parameter `?9?` (a placeholder for a runtime parameter, likely a library or file name).
     - The commas indicate unused parameter positions (up to 8 parameters are skipped).
     - This step prepares the environment for subsequent steps.

4. **Clear Local Variables**:
   - `// LOCAL BLANK-*ALL`:
     - Initializes all local variables to blanks, ensuring a clean state for the program.
     - This prevents residual data from affecting execution.

5. **Invoke Year 2000 Utility (`GSY2K`)**:
   - `// GSY2K`:
     - Calls a Year 2000 (Y2K) compliance utility, likely to handle date-related conversions or validations.
     - This ensures the program handles dates correctly, especially for legacy systems.

6. **Load the Main Program (`AR490P`)**:
   - `// LOAD AR490P`:
     - Loads the RPG program `AR490P` into memory for execution.
     - This is the core program responsible for the business logic (e.g., credit limit grouping).

7. **Open File (`ARCONT`)**:
   - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`:
     - Opens the file `ARCONT` with the label derived from parameter `?9?` concatenated with `ARCONT`.
     - `DISP-SHR` indicates the file is opened in shared mode, allowing concurrent access by other jobs.
     - This file is likely a customer or accounts receivable control file used by `AR490P`.

8. **Execute the Program**:
   - `// RUN`:
     - Initiates execution of the loaded program `AR490P`.
     - The program processes data using the opened `ARCONT` file.

9. **Conditional Check on Location 129 (`?L'129,6'?/CANCEL GOTO END`)**:
   - `// IF ?L'129,6'?/CANCEL GOTO END`:
     - Checks the value at location 129 (likely a status or error code) for a length of 6 characters.
     - If the condition is true (`CANCEL`), the program jumps to the `END` tag, terminating execution.
     - This acts as an error or cancellation check after `AR490P` runs.

10. **Conditional Job Submission Based on Location 120**:
    - `// IF ?L'120,1'?/Y JOBQ 5,?CLIB?,AR490,,,,,,,,,?9?`:
      - Checks the value at location 120 for a length of 1 character.
      - If true (`Y`), submits a job to job queue `5` with:
        - Library name from `?CLIB?` (a runtime parameter).
        - Program `AR490` (likely a related RPG program).
        - Parameter `?9?` passed to the job.
      - The commas indicate unused parameters.
    - `// ELSE AR490 ,,,,,,,,?9?`:
      - If the condition is false, runs `AR490` directly (not in a job queue) with parameter `?9?`.
      - This provides an alternate execution path for `AR490`.

11. **End of Program (`TAG END`)**:
    - `// TAG END`:
      - Marks the end of the program or a jump target for the `GOTO END` statement.
      - Execution stops here if the `CANCEL` condition is met.

12. **Clear Local Variables Again**:
    - `// LOCAL BLANK-*ALL`:
      - Resets all local variables to blanks, cleaning up after execution.
      - This ensures no residual data persists for subsequent runs.

---

### External Programs Called

The OCL program explicitly calls or references the following external programs:
1. **GSGENIEC**:
   - Called via `// CALL PGM(GSGENIEC)`.
   - Likely a utility for initialization or environment setup.
2. **GSY2K**:
   - Invoked via `// GSY2K`.
   - A Year 2000 utility for date handling.
3. **AR490P**:
   - Loaded and executed via `// LOAD AR490P` and `// RUN`.
   - The main RPG program for credit limit grouping.
4. **AR490**:
   - Referenced in the conditional job submission (`JOBQ`) or direct execution (`ELSE`).
   - Likely a related RPG program, possibly a variant or wrapper for `AR490P`.

---

### Tables/Files Used

The OCL program references the following file:
1. **ARCONT**:
   - Defined via `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`.
   - A file (likely a database table or physical file) used by `AR490P`.
   - The label includes the parameter `?9?`, suggesting the file resides in a library specified at runtime.
   - Likely contains customer or accounts receivable data for credit limit grouping.

No other files or tables are explicitly mentioned in the OCL program. However, `AR490P` (the RPG program) may internally reference additional files, which are not visible in the OCL.

---

### Summary

- **Process Steps**: The OCL program initializes the environment, checks conditions, loads and runs the `AR490P` RPG program with the `ARCONT` file, handles errors, and conditionally submits or runs `AR490` based on runtime parameters. It ensures proper setup, execution, and cleanup.
- **External Programs**: `GSGENIEC`, `GSY2K`, `AR490P`, `AR490`.
- **Tables/Files**: `ARCONT`.

If you need further analysis (e.g., details about `AR490P`’s RPG logic or additional context about `ARCONT`), please provide the RPG source code or more details about the system environment.