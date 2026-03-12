The provided document is an **OCL (Operation Control Language) program** named `BI941P`, typically used in IBM System/36 or AS/400 environments to control job execution and file operations. Below, I’ll explain the process steps of the OCL program, identify any external programs called, and list the tables used.

---

### **Process Steps of the BI941P OCL Program**

OCL programs are used to define job control procedures, such as loading files, setting switches, and invoking programs or job queues. Here’s a step-by-step breakdown of the `BI941P.ocl36.txt` program:

1. **Initial Setup and Comments**:
   - The file begins with comments (lines starting with `//`) that provide metadata or context, such as:
     - `SCPROCP ,,,,,,,,?9?`: Likely a procedure identifier or parameter placeholder (e.g., `?9?` is a substitution variable).
     - `SWITCH 00000000`: Initializes a switch (a control flag) to all zeros. Switches are used to control program flow.
     - `LOCAL BLANK-*ALL`: Clears all local variables to blanks, ensuring no residual data affects the process.
     - `GSY2K`: Possibly a reference to a system or configuration (e.g., Year 2000 compliance or a specific system module).

2. **Loading the Program**:
   - `// LOAD BI941P`: Loads the `BI941P` program into memory for execution. This is the main program or procedure being initiated.

3. **File Definitions**:
   - `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHRRM`:
     - Defines a file named `BICONT` with a label that includes the substitution variable `?9?` (e.g., a library or prefix).
     - `DISP-SHRRM` indicates the file is opened in shared read mode, allowing multiple processes to read it simultaneously.
   - `// FILE NAME-GSTABL,LABEL-?9?GSTABL,DISP-SHRRM`:
     - Defines another file named `GSTABL` with a similar label structure and shared read mode.

4. **Run Command**:
   - `// RUN`: Initiates the execution of the loaded program (`BI941P`) with the defined files (`BICONT` and `GSTABL`).

5. **Conditional Logic**:
   - `// IF SWITCH1-1 GOTO END`:
     - Checks if the first bit of the switch (SWITCH1) is set to `1`.
     - If true, the program jumps to the `END` tag, effectively skipping further processing.
   - `// IF ?L'120,1'?/Y JOBQ ?CLIB?,BI941,,,,,,,,,?9?`:
     - Checks if a specific condition (`?L'120,1'?` equals `Y`). This likely evaluates a system variable or parameter at position 120, length 1.
     - If true, submits the `BI941` program to a job queue (`JOBQ`) in the library specified by `?CLIB?`, with `?9?` as a parameter.
   - `// ELSE BI941 ,,,,,,,,?9?`:
     - If the condition is false, directly executes the `BI941` program with the `?9?` parameter, bypassing the job queue.

6. **End Tag**:
   - `// TAG END`: Marks the `END` label, where the program jumps if `SWITCH1` is `1`. This effectively terminates the procedure.

7. **Cleanup**:
   - `// SWITCH 00000000`: Resets the switch to all zeros, ensuring a clean state for future runs.
   - `// LOCAL BLANK-*ALL`: Clears all local variables again, preventing data carryover.

---

### **External Programs Called**

The OCL program references the following external program:
- **BI941**: This is the main program invoked either directly (`BI941 ,,,,,,,,?9?`) or through a job queue (`JOBQ ?CLIB?,BI941,,,,,,,,,?9?`). It is likely an RPG or another type of program that performs the core business logic, such as processing customer sales data.

No other external programs are explicitly called in the OCL script.

---

### **Tables Used**

The OCL program references the following files (which may be tables or datasets):
- **BICONT**: A file with a label incorporating the `?9?` substitution variable, opened in shared read mode (`DISP-SHRRM`). This could be a control file or a data file used by the `BI941` program.
- **GSTABL**: Another file with a similar label structure, also opened in shared read mode. This is likely a table containing reference data, such as general system tables or configuration data.

---

### **Summary**

- **Process Steps**:
  1. Initialize switches and clear local variables.
  2. Load the `BI941P` program.
  3. Define and open two files (`BICONT` and `GSTABL`) in shared read mode.
  4. Execute the program.
  5. Check if `SWITCH1` is set to `1`; if so, skip to the end.
  6. Evaluate a condition (`?L'120,1'?` equals `Y`):
     - If true, submit `BI941` to a job queue.
     - If false, run `BI941` directly.
  7. Reset switches and clear variables at the end.

- **External Programs Called**:
  - `BI941`

- **Tables Used**:
  - `BICONT`
  - `GSTABL`

This OCL program acts as a job control script, setting up the environment and conditionally executing the `BI941` program, likely for processing customer sales data as indicated by the document title.