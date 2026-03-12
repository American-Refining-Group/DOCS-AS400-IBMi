The provided document is an Operations Control Language (OCL) program, specifically `AR901P.ocl36.txt`, used to control job execution in IBM System/36 environments. OCL is a scripting language for managing job steps, file operations, and program execution on System/36. Below, I’ll explain the process steps of the OCL program, identify any external programs called, and list the tables (or files) used.

---

### Process Steps of the OCL Program

OCL programs define job control steps, including program execution, file handling, and conditional logic. Here’s a breakdown of the steps in `AR901P.ocl36.txt`:

1. **Invoke GSGENIEC Program**:
   - `// CALL PGM(GSGENIEC)`:
     - This command calls an external program named `GSGENIEC`.
     - Likely a utility or initialization program, possibly for setting up the environment or validating conditions before proceeding.
     - No parameters are explicitly passed in this call.

2. **Conditional Check on Location 506, Position 3**:
   - `// IFF ?L'506,3'?/YES RETURN`:
     - This checks the value at memory location 506, position 3 (likely a system switch or flag).
     - If the condition evaluates to `YES` (true), the job terminates (`RETURN`).
     - This acts as an early exit condition, possibly to skip execution based on a specific system state.

3. **Set Local Variables to Blank**:
   - `// LOCAL BLANK-*ALL`:
     - Initializes all local variables to blank (empty).
     - Ensures a clean state for subsequent processing, preventing residual data from affecting the job.

4. **Set Procedure Context**:
   - `// SCPROCP ,,,,,,,,?9?`:
     - This command likely sets up a procedure context or scope for the job.
     - The `,,,,,,,?9?` indicates placeholders for parameters, with `?9?` possibly referring to a specific library, file, or parameter (context-specific, often a system-defined value).
     - Exact meaning depends on the system configuration, but it’s typically for procedure execution control.

5. **Invoke GSY2K**:
   - `// GSY2K`:
     - Calls a program or procedure named `GSY2K`.
     - Likely a utility for Year 2000 compliance or date-related processing, common in legacy systems like System/36.
     - No additional parameters are specified.

6. **Set Switch to 0XXXXXXX**:
   - `// SWITCH 0XXXXXXX`:
     - Sets a system switch to the pattern `0XXXXXXX` (a binary or bit pattern).
     - In System/36, switches control program flow or behavior. Here, the first bit is set to `0`, and the remaining bits (`XXXXXXX`) are unspecified or left unchanged.
     - This configures the environment for the subsequent program load.

7. **Load the AR901P Program**:
   - `// LOAD AR901P`:
     - Loads the main program `AR901P` into memory for execution.
     - This is likely the core RPG program responsible for generating the "Customer Master Listing."

8. **Define File ARCONT**:
   - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`:
     - Declares a file named `ARCONT` with a label `?9?ARCONT` (the `?9?` prefix likely indicates a library or system-specific naming convention).
     - `DISP-SHR` specifies that the file is opened in shared mode, allowing multiple jobs to access it concurrently.
     - This file is likely the customer master file containing data for the listing.

9. **Execute the Program**:
   - `// RUN`:
     - Initiates execution of the loaded program (`AR901P`).
     - The program processes the `ARCONT` file to generate the customer master listing.

10. **Check Switch 1 and Cancel if Set**:
    - `// IF SWITCH1-1 CANCEL`:
      - Checks if the first switch (bit) is set to `1`.
      - If true, the job is canceled, terminating execution.
      - This provides a conditional exit based on runtime conditions (e.g., an error or specific state).

11. **Conditional Job Queue or Direct Execution**:
    - `// IF ?L'120,1'?/Y JOBQ ?CLIB?,AR901,,,,,,,,,?9?`:
      - Checks the value at memory location 120, position 1.
      - If true (`/Y`), the job `AR901` is submitted to a job queue in the library `?CLIB?` (a placeholder for a specific library), with `?9?` indicating additional parameters or a system-specific value.
      - This queues the job for asynchronous execution.
    - `// ELSE AR901 ,,,,,,,,?9?`:
      - If the condition is false, the `AR901` job is executed directly (synchronously) with placeholder parameters (`?9?`).
      - This provides flexibility to either queue or run the job based on system state.

12. **Reset Local Variables**:
    - `// LOCAL BLANK-*ALL`:
      - Again, sets all local variables to blank at the end of the job.
      - Ensures cleanup and prevents data leakage for subsequent jobs.

---

### External Programs Called

The OCL program explicitly calls or references the following external programs:
1. **GSGENIEC**:
   - Called via `// CALL PGM(GSGENIEC)`.
   - Likely a utility program for initialization or environment setup.
2. **GSY2K**:
   - Invoked via `// GSY2K`.
   - Possibly a Year 2000 compliance utility or date-processing program.
3. **AR901P**:
   - Loaded and executed via `// LOAD AR901P` and `// RUN`.
   - The main RPG program that processes the customer master file to produce the listing.
4. **AR901**:
   - Referenced in the conditional job queue or execution step (`JOBQ ?CLIB?,AR901` or `AR901`).
   - Likely the same as `AR901P` or a related program/procedure for generating the customer master listing.

---

### Tables (Files) Used

The OCL program references the following file (table):
1. **ARCONT**:
   - Defined via `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`.
   - This is likely the customer master file, containing records such as customer IDs, names, addresses, or other relevant data used by `AR901P` to generate the listing.
   - The `?9?` prefix in the label suggests a library or system-specific naming convention (e.g., a library like `QS36F` or a user-defined library).
   - Opened in shared mode (`DISP-SHR`), indicating it can be accessed by multiple jobs.

No other files or tables are explicitly mentioned in the OCL code.

---

### Additional Notes
- **Placeholder `?9?`**: The `?9?` appears multiple times (e.g., in `SCPROCP`, file labels, and job queue parameters). This is likely a system-specific variable or library reference, possibly resolved at runtime based on the System/36 configuration.
- **System/36 Context**: The OCL syntax and commands like `SWITCH`, `LOCAL`, and `JOBQ` are specific to IBM System/36, a midrange computer system from the 1980s. The program assumes a legacy environment with RPG II or RPG III for business applications.
- **Conditional Logic**: The use of `IFF` and `IF` statements with memory locations (`?L'506,3'?` and `?L'120,1'?`) suggests dynamic control based on system or job states, typical in batch processing environments.
- **No Chart Generation**: Since the query does not explicitly request a chart and no numerical data is provided, no chart is generated.

If you need further clarification or analysis (e.g., details about `AR901P`’s RPG logic, assuming its code is available), please provide additional context or files.