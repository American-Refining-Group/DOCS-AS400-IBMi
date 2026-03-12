The provided document is an Operation Control Language (OCL) program, specifically `AP991P.ocl36.txt`, used in IBM System/36 environments to manage processes, typically for accounts payable tasks. Below, I’ll explain the process steps of the OCL program, identify any external programs called, and list any tables used, based on the content provided.

### Process Steps of the RPG .OCL Program

OCL is a scripting language used on IBM System/36 to control job execution, file handling, and program flow. The `AP991P` OCL program appears to be a control script that sets up and executes a specific process, likely related to accounts payable. Here’s a step-by-step breakdown of the program’s process:

1. **Header/Comment Section (`// SCPROCP ,,,,,,,,?9?`)**
   - The `SCPROCP` line is likely a comment or identifier for the procedure, possibly indicating a specific accounts payable process. The `?9?` placeholder suggests a parameter or variable that may be substituted at runtime, potentially defining a specific job or configuration.

2. **Clear Local Variables (`// LOCAL BLANK-*ALL`)**
   - This command resets all local variables in the job environment to blank, ensuring no residual data from prior executions affects the current run. This is a common initialization step to maintain a clean state.

3. **Set System Date Compliance (`// GSY2K`)**
   - The `GSY2K` command likely enables Year 2000 compliance for date handling in the System/36 environment. This ensures that date-related operations in the program interpret years correctly (e.g., distinguishing between 19xx and 20xx).

4. **Load Program (`// LOAD AP991P`)**
   - This command loads the `AP991P` program into memory for execution. `AP991P` is likely an RPG (Report Program Generator) program, given the context of accounts payable and the OCL structure, which will perform the core processing logic.

5. **Specify File Access (`// FILE NAME-GSCONT,LABEL-?9?GSCONT,DISP-SHR`)**
   - This line defines a file named `GSCONT` to be used by the program:
     - `NAME-GSCONT`: Specifies the logical file name `GSCONT`.
     - `LABEL-?9?GSCONT`: Indicates the file’s label, where `?9?` is likely a parameter or variable substituted at runtime (e.g., a specific library or prefix).
     - `DISP-SHR`: Sets the file disposition to shared, allowing multiple jobs to access the file concurrently without exclusive locking.
   - The file `GSCONT` is likely a data file (e.g., a general ledger or control file) used by the `AP991P` program for accounts payable processing.

6. **Execute Program (`// RUN`)**
   - This command initiates the execution of the loaded `AP991P` program. The program will perform its defined logic, likely processing data from the `GSCONT` file or other inputs.

7. **Conditional Check and Termination (`// IF ?L'129,6'?/CANCEL GOTO Himmel GOTA END`)**
   - This line checks a condition based on a parameter or variable `?L'129,6'?`:
     - The syntax suggests a comparison or status check, possibly related to a return code, error condition, or data value at position 129,6 (e.g., a 6-character field starting at position 129 in a record).
     - If the condition evaluates to true (e.g., an error or specific status), the job is canceled, and control jumps to the `END` tag.
     - The exact nature of `?L'129,6'?` is unclear without more context, but it could represent a specific error code or validation check.

8. **End Tag (`// TAG END`)**
   - Marks the `END` label, where the job flow is directed if the condition in the `IF` statement is met. This effectively terminates the job.

9. **Clear Local Variables Again (`// LOCAL BLANK-*ALL`)**
   - After reaching the `END` tag (whether normally or via cancellation), this command clears all local variables again, ensuring a clean exit from the job.

### External Programs Called

- **AP991P**: This is the primary program loaded and executed by the OCL script using the `// LOAD AP991P` and `// RUN` commands. It is likely an RPG program responsible for the core accounts payable processing logic.
- No other external programs are explicitly called in the provided OCL code.

### Tables Used

- **GSCONT**: The only explicitly mentioned file is `GSCONT`, defined in the `// FILE` statement. It is likely a data file or table (e.g., a control file, general ledger, or accounts payable master file) used by the `AP991P` program.
- No other tables or files are explicitly referenced in the provided code.

### Notes and Assumptions

- **Limited Context**: The OCL code is brief and lacks detailed context about the `AP991P` program’s internal logic or the specific role of `GSCONT`. The explanation assumes typical accounts payable processing based on the filename and System/36 conventions.
- **Placeholders (`?9?`, `?L'129,6'?`)**: The placeholders suggest runtime parameters or variables. Without additional documentation, their exact values or purposes are speculative. `?9?` might represent a library or job-specific prefix, while `?L'129,6'?` could be a field or status check.
- **System/36 Environment**: The OCL syntax and commands like `GSY2K` and `DISP-SHR` are specific to the IBM System/36, indicating this is a legacy system used for business applications like accounts payable.
- **No Additional Files or Tables**: Only `GSCONT` is mentioned. If `AP991P` uses additional files or tables, they are not specified in the OCL script and would be defined within the RPG program or other configuration files.

### Summary

- **Process Steps**: The OCL program initializes the environment, ensures Y2K compliance, loads and runs the `AP991P` program with the `GSCONT` file, checks a condition to potentially cancel the job, and cleans up variables upon completion.
- **External Programs**: `AP991P` (RPG program).
- **Tables Used**: `GSCONT` (likely a control or data file).

If you have additional details about the `AP991P` program, the `GSCONT` file structure, or the specific parameters (`?9?`, `?L'129,6'?`), I can provide a more detailed analysis.