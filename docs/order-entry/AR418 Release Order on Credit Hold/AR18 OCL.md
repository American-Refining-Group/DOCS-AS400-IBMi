The provided document is an **Operation Control Language (OCL)** program, specifically `AR418.ocl36`, which appears to be part of an IBM System/36 environment. OCL is used to control job execution, manage files, and invoke programs. Below, I will explain the process steps of the program, identify the external programs called, and list the tables (files) used.

---

### **Process Steps of the AR418.ocl36 Program**

The OCL program performs a sequence of operations to set up and execute a process related to updating credit limits, likely in an accounts receivable or customer management system. Here’s a step-by-step breakdown of the process:

1. **Delete All File Overrides**:
   - `DLTOVR FILE(*ALL)`: This command removes all existing file overrides to ensure a clean slate for file assignments.

2. **Call Initial Program (Commented Out)**:
   - `// CALL PGM(GSGENIEC)`: This line is commented out (indicated by `//`), so it does not execute. If uncommented, it would call the program `GSGENIEC`, possibly for initialization or environment setup.

3. **Conditional Check (Commented Out)**:
   - `// IFF ?L'506,3'?/YES RETURN`: This conditional statement is commented out. If active, it would check a specific condition at location 506, position 3 in the local data area. If the condition is met (`YES`), the program would terminate (`RETURN`).

4. **Procedure and Local Variable Setup**:
   - `// SCPROCP ,,,,,,,,?9?`: This likely invokes a procedure or sets a parameter, with `?9?` being a placeholder for a runtime value (e.g., a company code or environment identifier).
   - `// LOCAL BLANK-*ALL`: Clears all local variables to ensure no residual data affects the process.

5. **Check for Active Processes**:
   - The program checks if processes `AR880` or `AR418` are already running:
     - `IF ACTIVE-AR880`: Checks if the `AR880` process is active.
       - If true, it displays messages: *"EITHER OPTION 18 OR 20 IS ALREADY IN PROGRESS"* and *"PLEASE TRY AGAIN IN A FEW MINUTES"*.
       - It then prompts the user with `PAUSE 'PRESS 0,ENTER AND JOB WILL CANCEL'`, allowing the user to cancel the job by entering `0`.
       - If active, the program jumps to the `OUT` tag, terminating execution.
     - The same logic applies for `AR418`, ensuring that only one instance of these processes runs at a time to avoid conflicts.

6. **Environment Setup**:
   - `// GSY2K`: Likely sets up a Year 2000-compliant environment or calls a procedure to handle date-related settings.
   - `// SWITCH 00000000`: Resets all job switches to `0`, ensuring a known state for conditional logic.

7. **Set User Information**:
   - `// LOCAL OFFSET-103,DATA-'?USER?'`: Stores the user ID at offset 103 in the local data area, using the runtime value of `?USER?`.

8. **File Overrides**:
   - The program overrides database files to point to specific libraries or files:
     - `OVRDBF FILE(BICONT) TOFILE(QS36F/GBICONT)`: Overrides the `BICONT` file to use `QS36F/GBICONT`.
     - `OVRDBF FILE(BBCSR) TOFILE(QS36F/GBBCSR)`: Overrides the `BBCSR` file to use `QS36F/GBBCSR`.
     - `OVRDBF FILE(BBSLSM) TOFILE(QS36F/GBBSLSM)`: Overrides the `BBSLSM` file to use `QS36F/GBBSLSM`.
   - These overrides ensure the program accesses the correct files in the `QS36F` library.

9. **Load Program and File Definitions**:
   - `// LOAD AR418`: Loads the main program `AR418`, which likely contains the core logic for updating credit limits.
   - File definitions (some commented out, others active):
     - `FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHRRM`: Opens the `BICONT` file with a label prefixed by the value of `?9?` (e.g., a company code) in shared read mode.
     - `FILE NAME-BBORCL,LABEL-?9?BBORCL,DISP-SHR`: Opens the `BBORCL` file in shared mode.
     - `FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHRMM`: Opens the `ARCUST` file in shared read/write mode.
     - Commented-out file definitions for `BBCSR` and `BBSLSM` suggest they were previously used or are optional.

10. **Printer Setup**:
    - `// PRINTER NAME-JBLIST,DEVICE-PJ,FORMSNO-JBCL,PRIORITY-0`: Configures a printer output for a report named `JBLIST`, directed to device `PJ` with form number `JBCL` and priority `0`.

11. **Conditional Printer Overrides**:
    - `// IF ?9?/G OVRPRTF FILE(CREMAL) OUTQ(CSROUTQ)`: If the parameter `?9?` equals `G`, redirects the `CREMAL` report to the `CSROUTQ` output queue.
    - `// IF ?9?/G OVRPRTF FILE(SMEMAL) OUTQ(SLMNOUTQ)`: Similarly, redirects the `SMEMAL` report to the `SLMNOUTQ` output queue.
    - `// IFF ?9?/G OVRPRTF FILE(CREMAL) OUTQ(TESTOUTQ)`: If `?9?` is not `G`, redirects `CREMAL` to the `TESTOUTQ` output queue (for testing).
    - `// IFF ?9?/G OVRPRTF FILE(SMEMAL) OUTQ(TESTOUTQ)`: Similarly, redirects `SMEMAL` to `TESTOUTQ`.

12. **Run the Program**:
    - `// RUN`: Executes the loaded program (`AR418`).

13. **Reset Environment**:
    - `// LOCAL BLANK-*ALL`: Clears all local variables again.
    - `// SWITCH 00000000`: Resets job switches to `0`.

14. **Conditional Program Call**:
    - `// IF ?9?/G CALL AR418TC`: If `?9?` equals `G`, calls the program `AR418TC`.
    - A commented-out line `*******CALL AR418TC` suggests this call might have been unconditional in earlier versions.

15. **Program Termination**:
    - `// TAG OUT`: Marks the `OUT` label, where the program jumps if `AR880` or `AR418` is active, or after completing execution.

---

### **External Programs Called**

The OCL program references the following external programs:

1. **GSGENIEC** (commented out):
   - Purpose: Likely an initialization or environment setup program, but it is not executed in this version due to being commented out.
2. **AR418**:
   - Purpose: The main program loaded and executed, responsible for the core logic of updating credit limits.
3. **AR418TC**:
   - Purpose: Called conditionally if `?9?` equals `G`. Likely a test or cleanup program related to the credit limit update process.

---

### **Tables (Files) Used**

The program interacts with the following files (tables), as defined in the `OVRDBF` and `FILE` statements:

1. **BICONT** (`QS36F/GBICONT`):
   - Label: `?9?BICONT` (dynamic label based on `?9?`).
   - Access: Shared read mode (`DISP-SHRRM`).
   - Purpose: Likely contains control or configuration data for the credit limit update process.

2. **BBORCL**:
   - Label: `?9?BBORCL`.
   - Access: Shared mode (`DISP-SHR`).
   - Purpose: Likely stores order or credit limit data.

3. **BBCSR** (`QS36F/GBBCSR`):
   - Label: `?9?BBCSR` (commented out, but implied by override).
   - Access: Shared read/write mode (`DISP-SHRMM` in commented code).
   - Purpose: Likely contains customer service representative data or related records.

4. **ARCUST**:
   - Label: `?9?ARCUST`.
   - Access: Shared read/write mode (`DISP-SHRMM`).
   - Purpose: Likely the customer master file, containing customer details such as credit limits.

5. **BBSLSM** (`QS36F/GBBSLSM`):
   - Label: `?9?BBSLSM` (commented out, but implied by override).
   - Access: Shared read/write mode (`DISP-SHRMM` in commented code).
   - Purpose: Likely contains salesman or sales-related data.

6. **Output Files**:
   - **CREMAL**: Report file, directed to either `CSROUTQ` or `TESTOUTQ` based on the value of `?9?`.
   - **SMEMAL**: Report file, directed to either `SLMNOUTQ` or `TESTOUTQ` based on the value of `?9?`.
   - **JBLIST**: Report file sent to the printer device `PJ` with form number `JBCL`.

---

### **Summary**

The `AR418.ocl36` program manages the execution of a credit limit update process in a System/36 environment. It:
- Ensures no conflicting processes (`AR880` or `AR418`) are running.
- Sets up the environment by overriding files and configuring printer outputs.
- Loads and runs the main program `AR418`, with an optional call to `AR418TC` for specific conditions.
- Uses files like `BICONT`, `BBORCL`, `BBCSR`, `ARCUST`, and `BBSLSM` to manage data, and produces reports (`CREMAL`, `SMEMAL`, `JBLIST`).

The program is designed to handle concurrency, support testing environments (via `?9?/G`), and ensure proper file and printer configurations.