The provided document is an **OCL (Operation Control Language)** program, specifically `BB953P.ocl36.txt`, which appears to be a control file for managing a rack price file, likely within an IBM System/36 or AS/400 environment. OCL is used to control job execution, manage files, and invoke programs or procedures on these systems. Below, I’ll explain the process steps of the OCL program, identify any external programs called, and list the tables (files) used.

---

### **Process Steps of the OCL Program**

The OCL program defines a sequence of steps to manage the execution of a job related to rack price updates, including file assignments, conditional logic, and program invocation. Here’s a breakdown of the steps based on the provided code:

1. **Header and Metadata**:
   - The file begins with a comment block describing it as a "RACK PRICE FILE - LAST TWO PRICE CHANGES."
   - `// SCPROCP ,,,,,,,,?9?`: This likely specifies the procedure name (`SCPROCP`) and a parameter (`?9?`), which could be a placeholder for a runtime variable or library name. This line sets up the context for the procedure.

2. **Switch Initialization**:
   - `// SWITCH 00000000`: Initializes all eight switches to `0` (off). Switches in OCL are used for conditional control of program flow.
   - `// LOCAL BLANK-*ALL`: Clears all local variables, ensuring no residual data affects the job.

3. **GSY2K Comment**:
   - `// GSY2K`: This is likely a comment or marker indicating compliance with Year 2000 (Y2K) standards or referencing a specific system/process. It has no direct operational impact.

4. **Load Command**:
   - `// LOAD BB953P`: Loads the program or procedure named `BB953P`. This is the main program or job being executed.

5. **File Assignments**:
   - The following lines define file assignments with their respective labels and dispositions:
     - `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHRRM`: Assigns the file `BICONT` with a label prefixed by `?9?` (likely a library or system identifier) and specifies shared read mode (`SHRRM`).
     - `// FILE NAME-INLOC,LABEL-?9?INLOC,DISP-SHRRM`: Assigns the file `INLOC` with shared read mode.
     - `// FILE NAME-GSTABL,LABEL-?9?GSTABL,DISP-SHRRM`: Assigns the file `GSTABL` with shared read mode.
     - `// FILE NAME-GSCNTR1,LABEL-?9?GSCNTR1,DISP-SHR`: Assigns the file `GSCNTR1` with shared access (read-only, no modification).
     - `// FILE NAME-GSPROD,LABEL-?9?GSPROD,DISP-SHRRM`: Assigns the file `GSPROD` with shared read mode.
   - These files are likely input data files or tables used by the `BB953P` program for processing rack price data.

6. **Run Command**:
   - `// RUN`: Initiates the execution of the loaded program (`BB953P`) with the assigned files.

7. **Conditional Logic**:
   - `// IF SWITCH1-1 GOTO END`: Checks if switch 1 is set to `1`. If true, the program jumps to the `END` tag, effectively skipping further execution.
   - This implies that `SWITCH1` controls whether the main processing logic is executed or bypassed.

8. **Conditional Job Queue or Direct Execution**:
   - `// IF ?L'231,1'?/Y JOBQ ?CLIB?,BB953,,,,,,,,,?9?`: If the condition `?L'231,1'?` evaluates to `Y` (yes), the program submits a job named `BB953` to the job queue in the library specified by `?CLIB?`, with `?9?` as a parameter.
   - `// ELSE BB953 ,,,,,,,,?9?`: If the condition is not met, the program `BB953` is executed directly with the `?9?` parameter.
   - The `?L'231,1'?` likely refers to a system variable or parameter at position 231, length 1, which determines whether the job is queued or run immediately.

9. **End Tag**:
   - `// TAG END`: Marks the end of the program flow, where execution terminates if the `SWITCH1-1` condition is met.

10. **Final Cleanup**:
    - `// SWITCH 00000000`: Resets all switches to `0` again, ensuring a clean state.
    - `// LOCAL BLANK-*ALL`: Clears all local variables again, preventing data carryover to subsequent jobs.

---

### **External Programs Called**

The OCL program references the following external program:
- **BB953**: This is the main program or procedure executed or queued for execution. It is likely an RPG (Report Program Generator) program responsible for processing the rack price file, given the context and file extensions.

No other external programs are explicitly called in the OCL code.

---

### **Tables (Files) Used**

The OCL program references the following files (tables) used by the `BB953P` program:
1. **BICONT**: Likely a control or configuration file for rack price data, accessed in shared read mode (`SHRRM`).
2. **INLOC**: Possibly a file containing location-specific data, accessed in shared read mode (`SHRRM`).
3. **GSTABL**: Likely a general table file (e.g., for pricing or product data), accessed in shared read mode (`SHRRM`).
4. **GSCNTR1**: Possibly a control or counter file, accessed in shared read-only mode (`SHR`).
5. **GSPROD**: Likely a product master file containing product details, accessed in shared read mode (`SHRRM`).

These files are prefixed with a `?9?` label, indicating they reside in a specific library or are dynamically resolved at runtime.

---

### **Summary**

The `BB953P.ocl36.txt` OCL program is designed to manage the execution of a rack price update process. It:
- Initializes switches and local variables.
- Loads the `BB953P` program and assigns five input files (`BICONT`, `INLOC`, `GSTABL`, `GSCNTR1`, `GSPROD`).
- Checks if `SWITCH1` is set to bypass execution.
- Evaluates a condition (`?L'231,1'?`) to decide whether to queue the `BB953` program in a job queue or run it directly.
- Cleans up switches and variables after execution.

**External Programs Called**:
- `BB953`

**Tables (Files) Used**:
- `BICONT`
- `INLOC`
- `GSTABL`
- `GSCNTR1`
- `GSPROD`

This OCL program serves as a control script to orchestrate the execution of the `BB953` RPG program, ensuring the correct files are available and the job is processed appropriately based on system conditions.