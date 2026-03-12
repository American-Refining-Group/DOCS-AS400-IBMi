The provided document is an **Open Control Language (OCL)** program, specifically named `BB705P.ocl36.txt`, which is used to manage operations in an IBM System/36 environment. OCL is a scripting language used to control job execution, file handling, and program flow on IBM midrange systems like the System/36. Below, I’ll explain the process steps of the OCL program, identify any external programs called, and list any tables used.

### Process Steps of the OCL Program

The OCL program outlines a sequence of steps to manage the execution of a job, likely related to generating an **Open Order Listing by Order**. Here’s a step-by-step breakdown of the program’s logic:

1. **Header and Initialization**:
   - The program starts with a header indicating it is an **"OPEN ORDER LISTING BY ORDER"**.
   - Comments (`//`) provide metadata and configuration details:
     - `SCPROCP ,,,,,,,,?9?`: Likely specifies the procedure name (`SCPROCP`) and a parameter placeholder (`?9?`) for dynamic substitution (e.g., library or file name).
     - `LOCAL BLANK-*ALL`: Clears all local variables to ensure a clean environment.
     - `GSY2K`: Possibly a system identifier or configuration setting (e.g., for year 2000 compliance).
     - `SWITCH 00000000`: Initializes a switch register (8-bit) to all zeros. Switches are used for conditional logic in OCL.

2. **Load and File Definitions**:
   - `// LOAD BB705P`: Loads the program or procedure named `BB705P`. This is likely the main program responsible for generating the open order listing.
   - File definitions:
     - `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHRRM`: Defines a file named `BICONT` with a label that includes the `?9?` parameter (dynamic substitution). `DISP-SHRRM` indicates the file is opened in shared read mode, allowing multiple jobs to read it simultaneously.
     - `// FILE NAME-INLOC,LABEL-?9?INLOC,DISP-SHRRM`: Defines another file named `INLOC` with a similar label and shared read mode.
   - `// RUN`: Initiates the execution of the loaded program (`BB705P`) with the defined files.

3. **Conditional Logic for Cancellation**:
   - `// IF SWITCH8-1 GOTO END`: Checks if the 8th bit of the switch register is set to 1. If true, the program jumps to the `END` tag, effectively canceling the procedure.
   - The comment `// IF SWITCH8-1 * 'PROCEDURE IS CANCELLED'`: Indicates that setting `SWITCH8` to 1 is a mechanism to cancel the job.

4. **Conditional Job Queue Submission**:
   - `// IF ?L'166,1'?/Y JOBQ ?CLIB?,BB705,,,,,,,,,?9?`: If the condition `?L'166,1'?/Y` is true (i.e., a specific condition at position 166, length 1, equals 'Y'), the program submits a job named `BB705` to the job queue in the library specified by `?CLIB?`. The `?9?` parameter is passed along.
   - `// ELSE BB705 ,,,,,,,,?9?`: If the condition is false, the program executes `BB705` directly (not through the job queue) with the `?9?` parameter.
   - The `?L'166,1'?` likely refers to a system variable or parameter (e.g., a flag in a control file or system setting) that determines whether the job should be queued or run immediately.

5. **End of Program**:
   - `// TAG END`: Marks the `END` label, where the program jumps if `SWITCH8` is 1.
   - `// LOCAL BLANK-*ALL`: Clears all local variables again to clean up.
   - `// SWITCH 00000000`: Resets the switch register to all zeros.

### External Programs Called

- **BB705**: This is the primary program or procedure called by the OCL script. It is either executed directly (`BB705 ,,,,,,,,?9?`) or submitted to the job queue (`JOBQ ?CLIB?,BB705,,,,,,,,,?9?`). This program is likely responsible for generating the open order listing based on the data in the specified files.

### Tables Used

The OCL program references two files, which may be considered tables or datasets in the context of the System/36:

1. **BICONT**: A file used by the `BB705P` program, opened in shared read mode (`DISP-SHRRM`). The label `?9?BICONT` suggests it is stored in a library specified by the `?9?` parameter.
2. **INLOC**: Another file used by the program, also opened in shared read mode with the label `?9?INLOC`.

These files likely contain data related to open orders (e.g., inventory, customer orders, or location-specific data). However, the OCL script does not explicitly define them as "tables" in the RPG sense (e.g., RPG table arrays like `TABxxx`). If `BB705` is an RPG program, it might internally define tables, but no such tables are explicitly mentioned in the OCL code.

### Summary

- **Process Steps**:
  1. Initialize environment (clear variables, set switches).
  2. Load the `BB705P` program and define input files (`BICONT`, `INLOC`).
  3. Check if the job is canceled (`SWITCH8=1`); if so, skip to the end.
  4. Evaluate a condition (`?L'166,1'?/Y`) to decide whether to submit `BB705` to a job queue or run it directly.
  5. Clean up variables and switches at the end.

- **External Programs Called**:
  - `BB705`: The main program or procedure executed or queued.

- **Tables Used**:
  - No explicit RPG tables (e.g., `TABxxx`) are defined in the OCL code.
  - Files `BICONT` and `INLOC` are used as data sources, which may function as tables in the context of the `BB705` program.

If you have additional details (e.g., the RPG code for `BB705` or more context about the files), I can provide a more detailed analysis of the tables or program logic. Let me know if you’d like me to search for related information or analyze further!