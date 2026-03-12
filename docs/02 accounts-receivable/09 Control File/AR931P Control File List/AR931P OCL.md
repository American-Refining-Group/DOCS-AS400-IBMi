Let’s break down the provided OCL (Operation Control Language) program, `AR931P.ocl36.txt`, which is used to control job execution in an IBM System/36 environment, typically for RPG (Report Program Generator) programs. OCL is a scripting language for managing job flows, file operations, and program execution on the System/36. Below, I’ll explain the process steps, identify external programs called, and list any tables or files referenced.

### Process Steps of the OCL Program

The OCL script outlines a sequence of steps for executing a job related to an Accounts Receivable Control File List. Here’s a step-by-step explanation of the process:

1. **Program Invocation (`// CALL PGM(GSGENIEC)`)**:
   - The script starts by calling a program named `GSGENIEC`. This is likely a utility or control program that performs initial setup or validation tasks before the main job proceeds.
   - This step suggests that `GSGENIEC` may handle system-level checks or configurations, possibly related to the job environment or user permissions.

2. **Conditional Check (`// IFF ?L'506,3'?/YES RETURN`)**:
   - The `IFF` statement checks a condition at location `506,3` in the system’s library or memory (likely a status code or flag).
   - If the condition evaluates to `YES`, the script executes a `RETURN`, which terminates the OCL procedure at this point, preventing further execution.
   - This acts as a gatekeeper, ensuring the job only proceeds if the condition is not met (i.e., the condition is `NO`).

3. **Procedure Call (`// SCPROCP ,,,,,,,,?9?`)**:
   - The `SCPROCP` command invokes a system procedure, passing a parameter `?9?` (a placeholder for a value, likely a library or file identifier).
   - The commas indicate unused parameter positions, typical in System/36 OCL for aligning with a procedure’s expected parameter list.
   - This step likely prepares the environment or sets up a specific processing context, but the exact function depends on what `SCPROCP` does (not defined in the script).

4. **Clear Local Variables (`// LOCAL BLANK-*ALL`)**:
   - This command resets all local variables to blank, clearing any residual data from prior runs to ensure a clean state for the job.
   - This is a housekeeping step to prevent data contamination.

5. **Year 2000 Compliance (`// GSY2K`)**:
   - The `GSY2K` command likely invokes a Year 2000 compliance routine, ensuring date-related operations in the program are Y2K-compliant (e.g., handling four-digit years).
   - This was common in legacy systems like the System/36 to address date issues around the millennium.

6. **Load Program (`// LOAD AR931P`)**:
   - The `LOAD` command loads the RPG program `AR931P` into memory for execution.
   - This is the main program responsible for generating the Accounts Receivable Control File List.

7. **File Specification (`// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`)**:
   - Defines a file named `ARCONT` to be used by the program.
   - The `LABEL-?9?ARCONT` indicates the file is located in a library or directory specified by the `?9?` parameter, with the file name `ARCONT`.
   - `DISP-SHR` (disposition shared) allows multiple jobs to access the file simultaneously, typically for read-only operations.
   - This file is likely the Accounts Receivable control file containing data for the report.

8. **Run Program (`// RUN`)**:
   - Executes the loaded `AR931P` program, which processes the `ARCONT` file to generate the Accounts Receivable Control File List.

9. **Error Check (`// IF ?L'129,6'?/CANCEL GOTO END`)**:
   - Checks a condition at location `129,6` (likely an error or status code set by `AR931P`).
   - If the condition is met (`CANCEL`), the script jumps to the `END` tag, terminating the job.
   - This ensures the job halts if an error occurs during `AR931P` execution.

10. **Conditional Job Submission (`// IF ?L'120,1'?/Y JOBQ ?CLIB?,AR931,,,,,,,,,?9?`)**:
    - Checks a condition at location `120,1`.
    - If the condition is `Y` (yes), the script submits a job named `AR931` to a job queue in the library specified by `?CLIB?`, passing the `?9?` parameter.
    - This suggests `AR931` is a related job or program (possibly another RPG program or a continuation job) that runs asynchronously in the job queue.

11. **Alternative Execution (`// ELSE AR931 ,,,,,,,,?9?`)**:
    - If the condition at `120,1` is not `Y`, the script directly executes `AR931`, passing the `?9?` parameter.
    - This provides an alternative execution path, running `AR931` immediately rather than queuing it.

12. **End Tag (`// TAG END`)**:
    - Marks the `END` label, where the script jumps if the `CANCEL` condition is met in step 9.
    - This defines the termination point for error scenarios.

13. **Clear Local Variables Again (`// LOCAL BLANK-*ALL`)**:
    - Resets all local variables to blank again, ensuring no residual data remains after execution.
    - This is another housekeeping step to clean up the environment.

### External Programs Called

The OCL script explicitly references the following external programs:

1. **GSGENIEC**:
   - Called at the start of the script via `// CALL PGM(GSGENIEC)`.
   - Likely a utility program for system setup, validation, or initialization.

2. **AR931P**:
   - Loaded and executed via `// LOAD AR931P` and `// RUN`.
   - The main RPG program that generates the Accounts Receivable Control File List.

3. **AR931**:
   - Referenced in the conditional job submission (`JOBQ ?CLIB?,AR931`) or direct execution (`AR931`).
   - Likely another RPG program or job related to the Accounts Receivable process, possibly for further processing or reporting.

4. **SCPROCP**:
   - Invoked via `// SCPROCP ,,,,,,,,?9?`.
   - A system procedure, possibly for environment setup or configuration, but its exact function is not specified in the script.

### Tables/Files Used

The script explicitly references one file:

1. **ARCONT**:
   - Defined via `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`.
   - This is the Accounts Receivable control file, likely containing data such as customer accounts, balances, or transaction details used by `AR931P` to generate the report.
   - The `?9?` in the `LABEL` indicates the file resides in a library or directory specified by the `?9?` parameter.

No other tables or files are explicitly mentioned, though `AR931P` or `AR931` may internally reference additional files or tables not specified in the OCL script.

### Summary

- **Process Overview**: The OCL script controls the execution of an Accounts Receivable Control File List job. It starts by calling a setup program (`GSGENIEC`), checks a condition to proceed, sets up the environment, loads and runs the main RPG program (`AR931P`) using the `ARCONT` file, handles errors, and conditionally submits or runs another job/program (`AR931`). Housekeeping steps ensure a clean environment.
- **External Programs**: `GSGENIEC`, `AR931P`, `AR931`, `SCPROCP`.
- **Files/Tables**: `ARCONT`.

If you need further clarification or analysis (e.g., specific behavior of `AR931P` or `AR931`, or details about `GSY2K`), please provide additional context or files, as these components are system-specific and not fully detailed in the provided OCL script.