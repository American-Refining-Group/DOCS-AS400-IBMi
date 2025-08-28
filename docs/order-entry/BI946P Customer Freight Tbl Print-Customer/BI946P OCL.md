The provided document is an **OCL (Operation Control Language) program** named `BI946P.ocl36.txt`, likely used on IBM midrange systems like the AS/400 (now IBM i) to manage batch processing tasks. OCL is a control language used to define job steps, manage files, and control program execution in such environments. Below, I’ll explain the process steps of the OCL program, identify any external programs called, and list any tables or files used.

---

### Process Steps of the OCL Program

The OCL program `BI946P` appears to be a control script for managing a freight table agreement master file listing. Here’s a step-by-step breakdown of the process:

1. **Header and Initial Setup**:
   - The file starts with a header: `FREIGHT TABLE AGREEMENT MASTER FILE LISTING`, indicating the purpose of the program, likely related to generating or processing a freight table report or data.
   - Comments or directives like `// SCPROCP ,,,,,,,,?9?` may specify a procedure or job control parameters. The `?9?` placeholders are likely variables or parameters passed at runtime (common in OCL for dynamic substitution).

2. **Switch Initialization**:
   - `// SWITCH 00000000`: Sets all job switches (typically 8 binary switches) to `0`. Switches are used for conditional logic to control the flow of the job.
   - `// LOCAL BLANK-*ALL`: Clears all local variables, ensuring no residual data from previous runs affects the current execution.

3. **GSY2K Directive**:
   - `// GSY2K`: Likely a system-specific directive or comment, possibly related to Year 2000 compliance or a specific system configuration. It doesn’t directly affect program flow but may be required for environment setup.

4. **Load the Program**:
   - `// LOAD BI946P`: Loads the program `BI946P` into memory for execution. This is likely an RPG or CL program that performs the core logic of the freight table processing.

5. **File Definition**:
   - `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHRRM`:
     - Defines a file named `BICONT` to be used by the program.
     - `LABEL-?9?BICONT` suggests the file’s label includes a dynamic parameter (`?9?`), which could be a library, prefix, or version identifier passed at runtime.
     - `DISP-SHRRM` indicates the file is opened in **shared read mode**, allowing multiple processes to read the file simultaneously without locking it.

6. **Run the Program**:
   - `// RUN`: Executes the loaded program (`BI946P`). This is where the actual processing (e.g., generating the freight table listing) occurs.

7. **Conditional Logic Based on Switch**:
   - `// IF SWITCH1-1 GOTO END`:
     - Checks if the first switch (SWITCH1) is set to `1`.
     - If true, the program jumps to the `END` tag, effectively skipping further processing and terminating the job early.
     - If false, execution continues to the next step.

8. **Conditional Job Queue Submission**:
   - `// IF ?L'120,1'?/Y JOBQ ?CLIB?,BI946,,,,,,,,,?9?`:
     - Evaluates a condition based on a parameter at position 120, character 1 (likely a flag or value in a parameter string).
     - If the condition is `Y` (true), submits a job named `BI946` to the job queue specified by `?CLIB?` (a library name passed as a parameter), with additional parameters (including `?9?`).
     - The commas (`,,,,,,,,`) are placeholders for optional parameters not used in this call.
   - `// ELSE BI946 ,,,,,,,,?9?`:
     - If the condition is not `Y`, the program `BI946` is called directly (not submitted to a job queue) with the parameter `?9?`.

9. **End Tag**:
   - `// TAG END`: Marks the `END` label, where the program jumps if `SWITCH1` is `1`. This effectively ends the job.

10. **Cleanup**:
    - `// SWITCH 00000000`: Resets all switches to `0`, ensuring a clean state if the job is reused or chained.
    - `// LOCAL BLANK-*ALL`: Clears all local variables again, reinforcing cleanup.

---

### External Programs Called

The OCL program references the following external programs:
1. **BI946**:
   - This is likely an RPG or CL program called either directly (`BI946 ,,,,,,,,?9?`) or submitted to a job queue (`JOBQ ?CLIB?,BI946,,,,,,,,,?9?`).
   - It is the main program responsible for processing the freight table data.
   - The exact functionality of `BI946` isn’t specified in the OCL but is likely related to generating or updating the freight table agreement master file listing.

2. **BI946P**:
   - This is the program loaded by `// LOAD BI946P` and executed by `// RUN`. It may be the same as or a prerequisite to `BI946`, possibly handling initial setup or validation before calling `BI946`.

---

### Tables/Files Used

The OCL program explicitly references the following file:
1. **BICONT**:
   - Defined by `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHRRM`.
   - This is likely a database file or table containing freight table agreement data.
   - The `?9?` in the label suggests a dynamic naming convention (e.g., a library or prefix).
   - Opened in shared read mode (`SHRRM`), indicating it’s used for reading data without modifying it.

No other tables or files are explicitly mentioned in the OCL program. However, the program `BI946` or `BI946P` may access additional files internally, which would require examining their source code (not provided).

---

### Summary

- **Process Steps**: The OCL program initializes the environment, loads and runs `BI946P`, defines the `BICONT` file, checks a switch to potentially end early, and conditionally submits or calls `BI946` based on a parameter. It ensures cleanup by resetting switches and variables.
- **External Programs**: `BI946P` (loaded and run), `BI946` (called or submitted to a job queue).
- **Tables/Files**: `BICONT` (database file, shared read mode).

If you need further analysis of `BI946` or `BI946P` (e.g., their RPG/CL code) or more details about the `BICONT` file structure, please provide additional context or files.