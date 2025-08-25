The provided document is an **Operation Control Language (OCL)** program, specifically `BB951P.ocl36.txt`, used in IBM midrange systems (e.g., IBM System/36 or AS/400) to control job execution and manage file operations. Below, I’ll explain the process steps of this OCL program, identify any external programs called, and list the tables or files used.

### Process Steps of the OCL Program

OCL is a scripting language used to automate tasks and control program execution in IBM midrange environments. The `BB951P` program appears to be a rack price file update process that tracks the last two price changes. Here’s a step-by-step breakdown of the program’s logic based on the provided code:

1. **Header and Initial Setup**:
   - The initial comments (`// SCPROCP ,,,,,,,,?9?`) likely indicate metadata or configuration for the procedure, possibly specifying a library or environment variable (`?9?` is a placeholder for a value like a library name or version).
   - `// SWITCH 00000000`: Initializes all job switches to `0` (off). Switches are used for conditional logic in OCL.
   - `// LOCAL BLANK-*ALL`: Clears all local variables, ensuring no residual data from previous runs affects the program.

2. **File Loading**:
   - The `// LOAD BB951P` statement loads the program `BB951P` into memory for execution.
   - The following `// FILE NAME` statements define the input files required for the program, specifying their names, labels, and access modes:
     - `BICONT` (labeled `?9?BICONT`, DISP-SHRRM): Likely a control file, opened in shared read/modify mode.
     - `INLOC` (labeled `?9?INLOC`, DISP-SHRRM): Likely a location or inventory file, opened in shared read/modify mode.
     - `GSTABL` (labeled `?9?GSTABL`, DISP-SHRRM): A table file, opened in shared read/modify mode.
     - `GSCNTR1` (labeled `?9?GSCNTR1`, DISP-SHR): A control file, opened in shared read-only mode.
     - `GSPROD` (labeled `?9?GSPROD`, DISP-SHRRM): A product file, opened in shared read/modify mode.
   - The `?9?` in labels suggests a dynamic substitution (e.g., a library name or version), resolved at runtime.

3. **Program Execution**:
   - `// RUN`: Initiates the execution of the loaded program (`BB951P`). This step assumes `BB951P` is an RPG or CL program that processes the loaded files to perform the price file update logic (e.g., updating the rack price file with the last two price changes).

4. **Conditional Logic**:
   - `// IF SWITCH1-1 GOTO END`: Checks if switch 1 is set to `1`. If true, the program jumps to the `END` tag, effectively terminating the procedure. This suggests a mechanism to bypass execution under certain conditions (e.g., if a prior step sets the switch due to an error or specific condition).
   - If switch 1 is not set (`0`), the program proceeds to the next step.

5. **Conditional Job Queue or Direct Execution**:
   - `// IF ?L'122,1'?/Y JOBQ ?CLIB?,BB951,,,,,,,,,?9?`:
     - Checks if the first character of the 122nd local variable (`?L'122,1'?`) equals `Y`.
     - If true, the program submits the `BB951` job to a job queue in the library specified by `?CLIB?`, passing parameters (including the `?9?` placeholder). The commas indicate placeholder parameters, likely for compatibility with the job queue system.
     - This suggests the program can be run asynchronously in a batch queue if the condition is met.
   - `// ELSE BB951 ,,,,,,,,?9?`:
     - If the condition (`?L'122,1'?/Y`) is false, the program directly calls `BB951` (likely an RPG or CL program) with parameters, including the `?9?` placeholder.
     - This indicates synchronous execution of `BB951` in the current job stream.

6. **End of Program**:
   - `// TAG END`: Marks the end of the program, used as a target for the `GOTO END` statement.
   - `// SWITCH 00000000`: Resets all switches to `0`, ensuring a clean state for any subsequent processes.
   - `// LOCAL BLANK-*ALL`: Clears all local variables again, ensuring no residual data persists.

### External Programs Called

The OCL program explicitly references the following external program:
- **BB951**: This is likely an RPG or CL program that performs the core logic of updating the rack price file with the last two price changes. It is either executed directly (`ELSE` branch) or submitted to a job queue (`IF` branch).

No other external programs are explicitly called in the OCL code.

### Tables/Files Used

The program references the following files (which may include tables or data files):
1. **BICONT** (`?9?BICONT`, DISP-SHRRM): Likely a control file containing configuration or control data for the price update process.
2. **INLOC** (`?9?INLOC`, DISP-SHRRM): Likely an inventory or location file, possibly containing location-specific pricing or stock data.
3. **GSTABL** (`?9?GSTABL`, DISP-SHRRM): A table file, likely containing static or reference data used in the price update process (e.g., tax rates, price tiers).
4. **GSCNTR1** (`?9?GSCNTR1`, DISP-SHR): A control file, possibly used for counters or transaction tracking, opened in read-only mode.
5. **GSPROD** (`?9?GSPROD`, DISP-SHRRM): A product file, likely containing product details such as SKUs, descriptions, or current prices.

### Summary

- **Process Overview**: The OCL program `BB951P` sets up the environment, loads necessary files, and executes the `BB951` program either directly or via a job queue, depending on a condition based on a local variable. It manages the rack price file update, tracking the last two price changes, and uses switches and variable cleanup to ensure proper execution flow.
- **External Programs**: `BB951` (RPG or CL program).
- **Files/Tables**: `BICONT`, `INLOC`, `GSTABL`, `GSCNTR1`, `GSPROD`.

If you have additional details (e.g., the contents of `BB951` or specific file formats), I can provide a more detailed analysis of the process or logic. Let me know if you’d like me to search for related information or clarify any specific aspect!