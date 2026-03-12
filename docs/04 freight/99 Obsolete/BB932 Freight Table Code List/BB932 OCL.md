The provided document is a snippet of an IBM System/36 Operator Control Language (OCL) program, specifically related to a "Print Freight Table" process. OCL is a control language used on IBM System/36 to manage job execution, file handling, and program invocation. Below, I’ll explain the process steps of the OCL program, identify any external programs called, and list the tables (files) used, based on the provided document.

### Process Steps of the OCL Program

The OCL program snippet outlines the steps to set up and execute a job, likely for printing a freight table. Here’s a breakdown of the process steps inferred from the document:

1. **Job Initiation (SCPROCP)**:
   - The `// SCPROCP` statement indicates the start of a procedure (job) named `SCPROCP`. This is the main control procedure for the job.
   - The `,,,,,,,?9?` part suggests parameters or placeholders, possibly for library or system-specific values (e.g., a library name or version). The `?9?` could be a variable or system-specific substitution resolved at runtime.

2. **System Context (GSY2K)**:
   - The `// GSY2K` statement likely specifies a system configuration or environment setting, possibly related to the year 2000 compliance or a specific system module. This could set up the environment for the job, ensuring compatibility with the required system settings.

3. **Load Program (LOAD BB932)**:
   - The `// LOAD BB932` statement loads the program named `BB932`. This is likely an RPG (Report Program Generator) or another executable program that performs the core logic for printing the freight table.
   - The `LOAD` command prepares the program for execution by bringing it into memory.

4. **File Definitions**:
   - The OCL defines three files used by the program:
     - **BBFRTB**: File name `BBFRTB`, labeled `?9?BBFRTB`, with disposition `DISP-SHR` (shared access). This is likely the primary freight table file containing data to be printed.
     - **BICONT**: File name `BICONT`, labeled `?9?BICONT`, with disposition `DISP-SHR`. This could be a control file or a configuration file used by the program.
     - **GSTABL**: File name `GSTABL`, labeled `?9?GSTABL`, with disposition `DISP-SHR`. This might be a general system table or another data file required for the freight table processing.
   - The `DISP-SHR` indicates that these files can be accessed concurrently by multiple jobs, suggesting a shared resource environment.
   - The `?9?` prefix in the labels likely refers to a library or system-specific naming convention, resolved at runtime.

5. **Program Execution (RUN)**:
   - The `// RUN` statement initiates the execution of the loaded program (`BB932`). This is where the actual processing (e.g., reading the freight table, formatting data, and printing) occurs.
   - The program likely uses the defined files (`BBFRTB`, `BICONT`, `GSTABL`) to retrieve data, perform calculations, and generate the freight table output.

6. **End of Job**:
   - The document ends with `**`, which typically marks the end of the OCL procedure or job stream. No additional steps (e.g., output handling or cleanup) are shown, but these might be implied or defined elsewhere.

### External Programs Called

Based on the provided OCL snippet:
- **BB932**: This is the primary program loaded and executed by the `// LOAD BB932` and `// RUN` statements. It is likely an RPG program responsible for processing and printing the freight table.
- No other external programs are explicitly called in the provided snippet. If `BB932` calls additional subprograms or utilities (e.g., via RPG’s `CALL` operation), these are not visible in the OCL.

### Tables (Files) Used

The OCL specifies three files (tables) used by the program:
1. **BBFRTB**:
   - File name: `BBFRTB`
   - Label: `?9?BBFRTB`
   - Disposition: `DISP-SHR` (shared access)
   - Purpose: Likely the main freight table containing data such as shipping rates, zones, or weights to be printed.
2. **BICONT**:
   - File name: `BICONT`
   - Label: `?9?BICONT`
   - Disposition: `DISP-SHR`
   - Purpose: Possibly a control file with configuration data, such as print formats, parameters, or constants used by the program.
3. **GSTABL**:
   - File name: `GSTABL`
   - Label: `?9?GSTABL`
   - Disposition: `DISP-SHR`
   - Purpose: Likely a general system table, possibly containing reference data like tax rates, customer codes, or other lookup information.

### Additional Notes
- **Missing Details**: The snippet does not provide the internal logic of `BB932` (e.g., RPG code), so the exact processing steps (e.g., data retrieval, calculations, output formatting) are inferred. If you have the RPG source code for `BB932`, I can analyze it for a more detailed explanation.
- **System Context**: The `?9?` placeholders and `GSY2K` suggest a specific System/36 environment. If you can clarify the system setup or library names, I can refine the explanation.
- **Further Analysis**: If you want me to search for additional context (e.g., on X or the web) about System/36 OCL or typical freight table programs, I can do so. Alternatively, if you have more of the OCL or RPG code, please share it for a deeper analysis.

### Summary
- **Process Steps**: Initiate job (`SCPROCP`), set system context (`GSY2K`), load program (`BB932`), define files (`BBFRTB`, `BICONT`, `GSTABL`), execute program (`RUN`), and end job.
- **External Programs**: `BB932` (primary RPG program).
- **Tables Used**: `BBFRTB` (freight table), `BICONT` (control file), `GSTABL` (system table).

If you have additional code or specific questions about the program’s logic, let me know!