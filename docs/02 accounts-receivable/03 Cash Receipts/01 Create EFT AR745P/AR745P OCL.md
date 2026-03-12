The provided document is an Operation Control Language (OCL) script, specifically `AR745P.ocl36.txt`, used to control the execution of an RPG program in an IBM midrange system (likely AS/400 or iSeries). Below, I’ll explain the process steps of the OCL program, identify the external programs called, and list the tables (files) used.

### Process Steps of the OCL Program

OCL is a scripting language used to control job execution, manage files, and handle conditional logic in IBM midrange systems. The steps in this OCL script are as follows:

1. **Invoke External Program (`GSGENIEC`)**:
   - `// CALL PGM(GSGENIEC)`:
     - The script starts by calling an external program named `GSGENIEC`. This is likely a utility or initialization program that performs setup tasks or validations before the main program runs.
     - The exact functionality of `GSGENIEC` is not specified in the script, but it’s a prerequisite for further execution.

2. **Conditional Check on Location 506 (`?L'506,3'?/YES RETURN`)**:
   - `// IFF ?L'506,3'?/YES RETURN`:
     - The script checks a condition at location 506 (likely a field in a control area or system register) with a length of 3 characters.
     - If the condition evaluates to `YES` (true), the script terminates early with a `RETURN` statement, halting further execution.
     - This acts as an early exit condition, possibly checking a system flag or parameter.

3. **Set Procedure Parameter (`SCPROCP`)**:
   - `// SCPROCP ,,,,,,,,?9?`:
     - This sets up a procedure parameter, likely passing a value (`?9?`) to the program or environment.
     - The commas indicate placeholder parameters (up to 8), with `?9?` being a substitution variable (e.g., a library, file, or job-specific value).
     - This step configures the environment for the subsequent program execution.

4. **Initialize Local Variables (`LOCAL BLANK-*ALL`)**:
   - `// LOCAL BLANK-*ALL`:
     - Clears all local variables in the job’s storage area, resetting them to blanks.
     - This ensures a clean slate for variable usage in the program.

5. **Invoke GSY2K Processing**:
   - `// GSY2K`:
     - This likely invokes Year 2000 (Y2K) compliance processing, a common routine in legacy systems to handle date-related calculations or conversions.
     - It may adjust date fields to ensure proper handling of two-digit year formats.

6. **Load the Main Program (`AR745P`)**:
   - `// LOAD AR745P`:
     - Loads the RPG program `AR745P` into memory for execution.
     - This is the main program that performs the Electronic Funds Transfer (EFT) processing.

7. **Define Input Files**:
   - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`:
     - Declares a file named `ARCONT` (likely a control file for accounts receivable).
     - The `LABEL` parameter uses a substitution variable (`?9?ARCONT`), which specifies the file’s label or library.
     - `DISP-SHR` indicates the file is opened in shared mode, allowing multiple jobs to access it concurrently.
   - `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`:
     - Declares a second file named `ARCUST` (likely a customer master file).
     - Similarly, it uses a substitution variable (`?9?ARCUST`) and is opened in shared mode.

8. **Execute the Program (`RUN`)**:
   - `// RUN`:
     - Executes the loaded `AR745P` program, which processes data from the `ARCONT` and `ARCUST` files to generate the EFT report.

9. **Conditional Check on Location 124 (`?L'124,6'?/CANCEL GOTO END`)**:
   - `// IF ?L'124,6'?/CANCEL GOTO END`:
     - Checks a condition at location 124 (6 characters long).
     - If the condition evaluates to `CANCEL` (true), the script jumps to the `END` tag, terminating the job.
     - This is likely a validation to check for an error or cancellation condition.

10. **Conditional Job Queue Submission**:
    - `// IF ?L'120,1'?/Y JOBQ ?CLIB?,AR745,,,,,,,,,?9?,,?11?`:
      - Checks a condition at location 120 (1 character).
      - If the condition is `Y` (true), the job is submitted to a job queue (`JOBQ`) in the library specified by `?CLIB?`, running the `AR745` program with parameters (substitution variables `?9?` and `?11?`).
      - This allows the job to run asynchronously in a batch queue.
    - `// ELSE AR745 ,,,,,,,,?9?,,?11?`:
      - If the condition is false, the `AR745` program is executed directly (not in a job queue) with the same parameters.
      - This provides flexibility to run the job interactively or in batch mode based on the condition.

11. **End of Processing (`TAG END`)**:
    - `// TAG END`:
      - Marks the end of the script’s logic, used as a target for the `GOTO END` statement in the cancellation check.

12. **Clear Local Variables Again (`LOCAL BLANK-*ALL`)**:
    - `// LOCAL BLANK-*ALL`:
      - Clears all local variables again, ensuring no residual data remains after execution.
      - This is a cleanup step to free up resources.

### External Programs Called

The OCL script explicitly calls or references the following external programs:
1. **GSGENIEC**:
   - Called at the start of the script (`// CALL PGM(GSGENIEC)`).
   - Likely a utility or initialization program for setup or validation.
2. **AR745P**:
   - The main RPG program loaded and executed (`// LOAD AR745P`).
   - Responsible for processing the EFT report using the `ARCONT` and `ARCUST` files.
3. **AR745**:
   - Referenced in the conditional job queue submission (`JOBQ ?CLIB?,AR745` or `AR745` in the `ELSE` clause).
   - This may be an alias or alternative entry point for `AR745P`, or it could be a different program or job invoked with similar parameters.

Additionally, `GSY2K` is invoked, but it’s unclear if this is a program or a system-level command for Y2K date processing. For the purpose of this analysis, it’s treated as a processing step rather than a distinct external program unless further context confirms otherwise.

### Tables (Files) Used

The script references the following files (tables):
1. **ARCONT**:
   - File name: `ARCONT`
   - Label: `?9?ARCONT` (substitution variable for library or file label)
   - Disposition: `DISP-SHR` (shared access)
   - Likely an accounts receivable control file containing EFT transaction details or configuration data.
2. **ARCUST**:
   - File name: `ARCUST`
   - Label: `?9?ARCUST` (substitution variable for library or file label)
   - Disposition: `DISP-SHR` (shared access)
   - Likely a customer master file containing customer account information.

### Summary

The `AR745P.ocl36.txt` OCL script orchestrates the execution of an EFT report generation process by:
- Initializing the environment with `GSGENIEC` and Y2K processing.
- Loading and running the `AR745P` RPG program, which uses `ARCONT` and `ARCUST` files.
- Handling conditional logic to exit early, cancel, or submit the job to a queue based on system conditions.
- Cleaning up variables to ensure proper resource management.

**External Programs Called**:
- `GSGENIEC`
- `AR745P`
- `AR745` (potentially an alias or separate job)

**Tables Used**:
- `ARCONT` (Accounts Receivable Control File)
- `ARCUST` (Customer Master File)

If you need further clarification or additional analysis (e.g., specific behavior of `GSGENIEC` or `AR745P`), let me know, and I can provide guidance based on typical AS/400 practices or search for relevant context if required!