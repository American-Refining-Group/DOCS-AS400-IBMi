The provided document is an **Open Control Language (OCL)** program, specifically `BB750P.ocl36.txt`, used in IBM midrange systems like the AS/400 (now IBM i). OCL is a scripting language for controlling job flow and executing programs. Below, I’ll explain the process steps of the program, identify external programs called, and list any tables or files referenced.

---

### **Process Steps of the RPG .OCL Program**

The OCL program `BB750P` appears to be a control procedure for generating a Bill of Lading (BOL) listing for railcars, with options for summary or detail reports. Here’s a step-by-step breakdown of the process:

1. **Header and Initialization**:
   - The program starts with metadata or comments (`// SCPROCP ,,,,,,,,?9?`, `// LOCAL BLANK-*ALL`, `// SWITCH 00000000`, `// GSY2K`).
   - `// LOCAL BLANK-*ALL`: Clears all local variables to ensure a clean slate.
   - `// SWITCH 00000000`: Resets all eight control switches (SWITCH1 to SWITCH8) to `0`. These switches are used for conditional logic.
   - `// GSY2K`: Likely a system or environment setup command, possibly related to Y2K compliance or a specific system configuration.

2. **Load Statement**:
   - `// LOAD BB750P`: Loads the program `BB750P` into memory for execution. This is the main program being controlled by this OCL script.

3. **File Definitions**:
   - `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHRRM`:
     - Defines a file named `BICONT` with a label `?9?BICONT` (the `?9?` is a placeholder for a system-specific value, likely a library or prefix).
     - `DISP-SHRRM` indicates the file is opened in shared read mode, allowing multiple processes to read it concurrently.
   - `** FILE NAME-BBBOL,LABEL-?9?BBBOL,DISP-SHRRM`:
     - Defines another file named `BBBOL` with a similar label and shared read mode.
     - The double asterisk (`**`) indicates a comment, so this line is ignored during execution but suggests `BBBOL` is a relevant file, possibly for BOL data.

4. **Run Command**:
   - `// RUN`: Initiates the execution of the loaded program (`BB750P`).

5. **Conditional Logic for Cancellation**:
   - `// IF SWITCH8-1 GOTO END`:
     - Checks if `SWITCH8` is set to `1`. If true, the program jumps to the `END` tag, effectively canceling the procedure.
     - A comment (`* 'PROCEDURE IS CANCELLED'`) clarifies this behavior.

6. **Conditional Logic for Report Type**:
   - `// IF ?L'169,1'?/Y GOTO DTL`:
     - Checks a local variable or parameter at position 169, length 1 (likely a flag in a parameter string).
     - If the value is `Y`, the program jumps to the `DTL` (detail) tag to generate a detailed report.

7. **Summary Report Processing**:
   - If the condition `?L'169,1'?/Y` is false (i.e., no detail report is requested), the program proceeds to generate a summary report.
   - `// IF ?L'166,1'?/Y JOBQ ?CLIB?,BB751,,,,,,,,,?9?`:
     - Checks another local variable at position 166, length 1.
     - If true (`Y`), submits a job to the job queue (`JOBQ`) in the library `?CLIB?` (a placeholder for a specific library) to run the program `BB751` with parameters `,,,,,,,,,?9?`.
     - If false, executes `BB751` directly without queuing (`BB751 ,,,,,,,,?9?`).
   - After this, the program jumps to the `END` tag (`// GOTO END`).

8. **Detail Report Processing**:
   - `// TAG DTL`: Marks the start of the detail report processing section.
   - `// IF ?L'166,1'?/Y JOBQ ?CLIB?,BB750,,,,,,,,,?9?`:
     - Similar to the summary report logic, checks the local variable at position 166.
     - If true (`Y`), submits `BB750` to the job queue in `?CLIB?`.
     - If false, executes `BB750` directly (`BB750 ,,,,,,,,?9?`).

9. **Program Termination**:
   - `// TAG END`: Marks the end of the program logic.
   - `// LOCAL BLANK-*ALL`: Clears all local variables again.
   - `// SWITCH 00000000`: Resets all switches to `0`.
   - The program ends here.

---

### **External Programs Called**

The OCL program references two external programs:
1. **BB750**:
   - Called in the detail report section (`// TAG DTL`).
   - Likely an RPG or CL program responsible for generating the detailed BOL listing.
   - Executed directly or submitted to a job queue based on the condition `?L'166,1'?/Y`.

2. **BB751**:
   - Called in the summary report section.
   - Likely an RPG or CL program responsible for generating the summary BOL listing.
   - Executed directly or submitted to a job queue based on the condition `?L'166,1'?/Y`.

---

### **Tables/Files Used**

The program references the following files:
1. **BICONT**:
   - Defined with `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHRRM`.
   - Likely a data file containing railcar or inventory control information.
   - Opened in shared read mode (`DISP-SHRRM`).

2. **BBBOL**:
   - Mentioned in a commented line (`** FILE NAME-BBBOL,LABEL-?9?BBBOL,DISP-SHRRM`).
   - Likely a file containing Bill of Lading data, but since it’s commented out, it may not be used in this specific execution unless uncommented or referenced indirectly by `BB750` or `BB751`.

---

### **Summary of Program Flow**

- The program decides whether to produce a **summary report** (via `BB751`) or a **detail report** (via `BB750`) based on the value of a parameter at position 169.
- It uses a conditional check (`?L'166,1'?/Y`) to determine whether to run these programs directly or submit them to a job queue.
- The program can be canceled if `SWITCH8` is set to `1`.
- It interacts with files `BICONT` (and possibly `BBBOL`) for data processing.
- The program ensures a clean state by resetting variables and switches at the start and end.

---

### **Notes**
- The `?9?` and `?CLIB?` placeholders suggest dynamic substitution of library or system-specific values, typical in OCL scripts.
- The exact behavior of `BB750` and `BB751` (e.g., RPG programs, CL programs, or reports) would depend on their internal logic, which isn’t provided.
- No explicit tables (e.g., database tables) are mentioned beyond the files `BICONT` and `BBBOL`. If these are database files, they would be defined in the IBM i database.

If you have additional details (e.g., the code for `BB750` or `BB751`, or the structure of `BICONT`/`BBBOL`), I can provide a more detailed analysis!