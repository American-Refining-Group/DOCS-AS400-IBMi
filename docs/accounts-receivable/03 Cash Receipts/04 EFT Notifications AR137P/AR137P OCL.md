The provided document is an Operation Control Language (OCL) script, specifically `AR137P.ocl36.txt`, used in IBM midrange systems like the AS/400 (now IBM i) to control program execution and file operations. OCL is typically used to define job steps, file assignments, and program flow in conjunction with RPG (Report Program Generator) programs. Below, I’ll explain the process steps of the OCL script, identify the external programs called, and list the tables (files) used.

### Process Steps of the OCL Program

The OCL script outlines a sequence of steps to execute a job, likely related to accounts receivable (AR) processing, specifically for EFT (Electronic Funds Transfer) email selection by customer or all. Here’s a step-by-step breakdown of the script’s execution flow:

1. **Program Invocation (`// CALL PGM(GSGENIEC)`):
   - The script starts by calling an external program named `GSGENIEC`.
   - This program is likely a utility or initialization program, possibly for setting up the environment, validating user access, or performing preliminary checks.
   - The exact functionality of `GSGENIEC` is not specified in the script, but it’s a common practice in OCL to call a setup program before the main logic.

2. **Conditional Check (`// IFF ?L'506,3'?/YES RETURN`):
   - The `IFF` statement checks a condition based on a system variable or parameter at location `L'506,3'`.
   - The syntax `?L'506,3'?` likely refers to a specific memory location or parameter in the Local Data Area (LDA) or a similar control structure, checking a value at position 506 for a length of 3 characters.
   - If the condition evaluates to `YES` (true), the script executes the `RETURN` command, which terminates the job immediately, skipping all subsequent steps.
   - This suggests a gatekeeping mechanism, possibly checking if the job should proceed based on a system flag or user input.

3. **Local Data Initialization (`// LOCAL BLANK-*ALL`):
   - This command clears or blanks out all local variables or the Local Data Area (LDA) used by the job.
   - It ensures that no residual data from previous runs interferes with the current execution, providing a clean slate for variable storage.

4. **Date Conversion Utility (`// GSY2K`):
   - The `GSY2K` command invokes a system utility, likely related to date handling or Year 2000 (Y2K) compliance.
   - This utility might adjust date formats, validate dates, or set system date parameters to ensure compatibility with the program’s requirements.
   - It’s a common step in older AS/400 programs to handle date-related issues, especially for financial applications like accounts receivable.

5. **Program Load (`// LOAD AR137P`):
   - The `LOAD` command loads the main RPG program named `AR137P` into memory for execution.
   - This is the core program that performs the EFT email selection logic, likely generating reports or processing customer data for electronic funds transfer.

6. **File Definitions**:
   - Two files are defined for use by the `AR137P` program:
     - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`:
       - Defines a file named `ARCONT` (likely an accounts receivable control file).
       - The `LABEL-?9?ARCONT` indicates that the file’s label is dynamically resolved using a parameter or variable (the `?9?` placeholder), which could be a library or file prefix.
       - `DISP-SHR` specifies that the file is opened in shared mode, allowing multiple jobs to access it concurrently without locking.
     - `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`:
       - Defines a file named `ARCUST` (likely an accounts receivable customer file).
       - Similar to `ARCONT`, it uses a dynamic label (`?9?ARCUST`) and is opened in shared mode (`DISP-SHR`).
   - These files are likely database files containing control data (e.g., configuration settings) and customer data (e.g., customer details for EFT).

7. **Program Execution (`// RUN`):
   - The `RUN` command executes the loaded `AR137P` program.
   - This step triggers the RPG program to process the data in the `ARCONT` and `ARCUST` files, performing the EFT email selection logic (e.g., selecting customers for EFT notifications or processing all records).

8. **Second Conditional Check (`// IFF ?L'109,1'?/Y GOTO END`):
   - Another `IFF` statement checks a condition at location `L'109,1'`, examining a single character at position 109 in the LDA or similar structure.
   - If the condition evaluates to `Y` (true), the script jumps to the `END` tag, skipping the next step.
   - This check likely determines whether the main processing step (`AR137`) should be executed or bypassed, possibly based on a flag set by the `AR137P` program or user input.

9. **Main Processing Step (`// AR137 ,,,,,,,,?9?`):
   - This line invokes a procedure or program named `AR137`, passing nine parameters (indicated by the commas, with `?9?` as the ninth parameter).
   - The `?9?` placeholder suggests a dynamic value, possibly a library name, file prefix, or processing option (e.g., customer ID or “ALL” for all customers).
   - The `AR137` program likely performs the core EFT email selection logic, such as generating emails, reports, or updating records based on the input parameters.
   - The multiple commas indicate that the first eight parameters are either blank or not used, which is common in OCL when parameters are optional or fixed.

10. **Clear Local Data Again (`// LOCAL BLANK-*ALL`):
    - After the `AR137` step, the script clears the local variables or LDA again, ensuring no residual data remains.
    - This step is likely included to clean up before the job ends or to prepare for another iteration (though no loop is explicitly defined).

11. **End of Script (`// TAG END`):
    - The `END` tag marks the end of the script or a point to jump to (as referenced in the `IFF` statement).
    - The job terminates here, completing the execution.

### External Programs Called

The OCL script explicitly references the following external programs or utilities:

1. **GSGENIEC**:
   - Called at the start of the script via `// CALL PGM(GSGENIEC)`.
   - Likely a utility program for environment setup, validation, or initialization.

2. **GSY2K**:
   - Invoked via `// GSY2K`.
   - A system utility, likely for date handling or Y2K compliance.

3. **AR137P**:
   - Loaded and executed via `// LOAD AR137P` and `// RUN`.
   - The main RPG program that processes EFT email selection logic.

4. **AR137**:
   - Invoked via `// AR137 ,,,,,,,,?9?`.
   - Likely another program or procedure that performs the core EFT email selection or reporting task.

### Tables (Files) Used

The script references the following files (tables in IBM i terminology):

1. **ARCONT**:
   - Defined as `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`.
   - Likely an accounts receivable control file containing configuration or control data for the EFT process.

2. **ARCUST**:
   - Defined as `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`.
   - Likely an accounts receivable customer file containing customer data, such as EFT details or email addresses.

### Summary

The `AR137P.ocl36.txt` script orchestrates an accounts receivable EFT email selection process by:
- Initializing the environment with `GSGENIEC` and `GSY2K`.
- Checking conditions to control job flow (`L'506,3'` and `L'109,1'`).
- Loading and running the `AR137P` RPG program, which uses the `ARCONT` and `ARCUST` files.
- Optionally calling `AR137` for further processing based on a condition.
- Clearing local data to ensure a clean state.

**External Programs Called**: `GSGENIEC`, `GSY2K`, `AR137P`, `AR137`.
**Tables Used**: `ARCONT`, `ARCUST`.

If you need further details about the RPG program logic or specific file structures, let me know, though this would require access to the RPG source code or file definitions, which are not provided here.