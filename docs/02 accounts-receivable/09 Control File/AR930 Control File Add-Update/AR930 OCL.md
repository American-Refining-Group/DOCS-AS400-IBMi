The provided document is an Operation Control Language (OCL) script, specifically for an IBM System/36 environment, used to manage the execution of an RPG (Report Program Generator) program named `AR930`. Below, I’ll explain the process steps of the OCL script, identify the external programs called, and list the tables (files) used.

### Process Steps of the OCL Program

OCL is a scripting language used on IBM System/36 to control job execution, load programs, and manage file operations. The script `AR930.ocl36.txt` outlines the steps to execute the `AR930` RPG program, which appears to be related to Accounts Receivable (A/R) control file maintenance. Here’s a breakdown of each step:

1. **Invoke the GSGENIEC Program**:
   ```ocl
   // CALL PGM(GSGENIEC)
   ```
   - The script starts by calling an external program named `GSGENIEC`. This program likely performs some initialization, validation, or setup tasks required before proceeding with the main program. The exact functionality of `GSGENIEC` is not specified in the script, but it’s a prerequisite for the job.

2. **Conditional Check**:
   ```ocl
   // IFF ?L'506,3'?/YES   RETURN
   ```
   - The `IFF` (If) statement checks a condition using a system variable or parameter `?L'506,3'?`. This likely refers to a specific location in the Local Data Area (LDA) or a similar system construct, checking positions 506 to 508 (3 bytes).
   - If the condition evaluates to `YES` (true), the script executes the `RETURN` command, which terminates the job immediately. This suggests a conditional exit based on some predefined system state or parameter (e.g., a flag indicating whether the job should proceed).

3. **Set Procedure Parameters**:
   ```ocl
   // SCPROCP ,,,,,,,,?9?
   ```
   - The `SCPROCP` command sets up procedure parameters. The commas indicate placeholder values, and `?9?` is a parameter substitution, likely specifying a library, file, or environment setting (e.g., a diskette or library identifier). This step configures the environment for the subsequent program execution.

4. **Invoke GSY2K**:
   ```ocl
   // GSY2K
   ```
   - The `GSY2K` command calls another program or procedure, likely related to Year 2000 (Y2K) compliance or date handling. This could be a utility to ensure date fields are processed correctly, which was critical for A/R systems during the Y2K transition.

5. **Load the AR930 Program**:
   ```ocl
   // LOAD AR930
   ```
   - The `LOAD` command loads the RPG program `AR930` into memory for execution. This is the main program responsible for A/R control file maintenance.

6. **Specify Input Files**:
   ```ocl
   // FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR
   // FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR
   // FILE NAME-GLMAST,LABEL-?9?GLMAST,DISP-SHR
   ```
   - These `FILE` statements define the files used by the `AR930` program:
     - `ARCONT`: Likely the A/R control file, containing configuration or summary data for accounts receivable.
     - `ARCUST`: The A/R customer file, storing customer-specific data (e.g., account details, balances).
     - `GLMAST`: The General Ledger master file, used for integrating A/R transactions with the general ledger.
   - The `LABEL-?9?ARCONT` syntax indicates that the file label is dynamically substituted with the value of `?9?` (e.g., a library or diskette name). `DISP-SHR` specifies that the files are opened in shared mode, allowing concurrent access by other programs.

7. **Execute the Program**:
   ```ocl
   // RUN
   ```
   - The `RUN` command executes the loaded `AR930` program, which processes the specified files (`ARCONT`, `ARCUST`, `GLMAST`) to perform A/R control file maintenance tasks, such as updating control records, validating customer data, or posting transactions to the general ledger.

### External Programs Called

The OCL script explicitly calls the following external programs or procedures:
1. **GSGENIEC**: Likely a setup or validation program executed at the start.
2. **GSY2K**: Possibly a Y2K compliance or date-handling utility.

Additionally, the main program `AR930` (an RPG program) is loaded and executed, but it is not considered an "external program call" in the same sense as `CALL PGM` or procedure invocations, as it is the primary program being run.

### Tables (Files) Used

The script specifies the following files (referred to as "tables" in some contexts, especially in RPG programming):
1. **ARCONT**: Accounts Receivable control file.
2. **ARCUST**: Accounts Receivable customer file.
3. **GLMAST**: General Ledger master file.

These files are accessed in shared mode (`DISP-SHR`) and are critical to the A/R control file maintenance process performed by `AR930`.

### Summary

The OCL script orchestrates the execution of the `AR930` RPG program for A/R control file maintenance. It performs initial setup and validation (`GSGENIEC`, conditional check, `GSY2K`), loads the necessary files (`ARCONT`, `ARCUST`, `GLMAST`), and executes the main program. The process ensures that the environment is properly configured and that the program only runs if specific conditions are met, preventing unnecessary or erroneous execution.