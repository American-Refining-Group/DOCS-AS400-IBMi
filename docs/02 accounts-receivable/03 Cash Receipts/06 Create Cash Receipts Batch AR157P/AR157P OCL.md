The provided document is an Operations Control Language (OCL) program, specifically `AR157P.ocl36.txt`, used in IBM midrange systems (e.g., AS/400, iSeries) to manage AR (Accounts Receivable) EFT (Electronic Funds Transfer) cash receipts batch file creation. Below is an explanation of the process steps, the external programs called, and the tables/files used, based on the OCL code provided.

### Process Steps of the RPG .OCL Program

The OCL program `AR157P` orchestrates the creation of a cash receipts batch file by performing a series of checks, file operations, and program executions. Here’s a step-by-step breakdown of the process:

1. **Call the GSGENIEC Program**:
   - The program starts by calling an external program named `GSGENIEC` using the `// CALL PGM(GSGENIEC)` statement.
   - This program likely performs initial setup or validation tasks, such as checking system parameters or environment settings.

2. **Check for Specific Condition and Return**:
   - The statement `// IFF ?L'506,3'?/YES RETURN` checks a system or program variable at location `506` with a length of 3 characters.
   - If the condition evaluates to `YES` (true), the program terminates immediately with a `RETURN` statement, halting further execution.
   - This likely serves as a gatekeeper to ensure the program only proceeds under specific conditions (e.g., correct environment or user permissions).

3. **Set SCPROCP Parameter**:
   - The statement `// SCPROCP ,,,,,,,,?9?` sets a parameter or variable for the program, where `?9?` is likely a placeholder for a dynamic value (e.g., a company or batch identifier).
   - This could be used to pass context or configuration to subsequent steps or programs.

4. **Execute GSY2K Program**:
   - The `// GSY2K` statement invokes the `GSY2K` program or routine, which might handle date-related processing (e.g., Year 2000 compliance) or other system-level tasks.
   - This step ensures the system is prepared for date-sensitive operations in the cash receipts process.

5. **Check for Correct Table and Transaction File**:
   - The program checks if the correct table is selected (though the specific table is not named in the provided code).
   - It then checks for the existence of a cash receipts transaction file named `?9?CRIEGG` (where `?9?` is a placeholder and `GG` replaces `?WS?` as per the change noted by Jan Beccari).
   - If the file `?9?CRIEGG` exists in `DATAF1`, the program displays a message: `CANNOT POST. A CASH RECEIPTS TRANSACTION FILE EXISTS` and proceeds to the `END` tag, terminating the program.
   - This prevents overwriting or duplicating an existing transaction file.

6. **Set Local Variable if Transaction File Exists**:
   - The statement `// IF DATAF1-?9?CRIEGG LOCAL OFFSET-117,DATA-'Y'` checks again for the existence of the `?9?CRIEGG` file.
   - If found, it sets a local variable at offset 117 to `'Y'`, likely flagging that the transaction file exists for use in subsequent logic.

7. **Load and Run AR157P Program**:
   - The statements `// LOAD AR157P` and `// RUN` load and execute the main `AR157P` program.
   - Before execution, the program opens a file:
     - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR` specifies a file named `ARCONT` with a label `?9?ARCONT` (dynamic naming) in shared mode (`DISP-SHR`), allowing multiple processes to access it concurrently.
   - This step likely processes the cash receipts data using the `ARCONT` file.

8. **Conditional File Deletion and Termination**:
   - The nested condition `// IF ?L'116,1'?/Y IF ?L'117,1'?/Y IF DATAF1-?9?CRIEGG DELETE ?9?CRIEGG,F1` checks:
     - If the variable at location `116` (length 1) is `'Y'`.
     - If the variable at location `117` (length 1, set earlier to `'Y'` if `?9?CRIEGG` exists) is `'Y'`.
     - If the file `?9?CRIEGG` exists in `DATAF1`.
   - If all conditions are true, the program deletes the `?9?CRIEGG` file from `DATAF1`.
   - It then proceeds to the `END` tag (`// IF ?L'116,1'?/Y IF ?L'117,1'?/Y GOTO END`), terminating the program.
   - This step ensures cleanup of the transaction file under specific conditions.

9. **Additional Conditional Termination**:
   - The statement `// IFF ?L'109,1'?/Y GOTO END` checks a variable at location `109` (length 1).
   - If the condition is true (value is `'Y'`), the program jumps to the `END` tag, terminating execution.
   - This acts as another conditional exit point, possibly based on a status or error flag.

10. **Invoke AR157 Program**:
    - The statement `// AR157 ,,,,,,,,?9?` calls the `AR157` program, passing the `?9?` parameter (likely the same company or batch identifier used earlier).
    - This program likely performs the core logic of creating or processing the cash receipts batch file.

11. **End of Program**:
    - The `// TAG END` marks the end of the program, where execution terminates if reached.
    - The final statement `// LOCAL BLANK-*ALL` clears all local variables, resetting the environment to prevent data leakage between runs.

### External Programs Called

The OCL program explicitly calls the following external programs:
1. **GSGENIEC**: Called at the start, likely for initialization or environment validation.
2. **GSY2K**: Invoked for system-level processing, possibly related to date handling.
3. **AR157**: Called to perform the main cash receipts batch file creation or processing logic.

Additionally, the OCL program loads and runs itself (`AR157P`), but this is the main program, not an external one.

### Tables/Files Used

The program references the following files/tables:
1. **ARCONT**:
   - File name: `ARCONT`
   - Label: `?9?ARCONT` (dynamic, with `?9?` as a placeholder)
   - Disposition: `DISP-SHR` (shared access)
   - Purpose: Likely the main Accounts Receivable control file containing configuration or master data for cash receipts processing.
2. **?9?CRIEGG**:
   - File name: `?9?CRIEGG` (dynamic, with `?9?` as a placeholder and `GG` replacing `?WS?` as per the change log)
   - Library: `DATAF1`
   - Purpose: Cash receipts transaction file, used to store batch transaction data. The program checks for its existence and may delete it under certain conditions.

### Notes
- The `?9?` placeholder suggests dynamic naming, likely substituted with a company code, batch number, or similar identifier at runtime.
- The `?WS?` to `GG` change (noted by Jan Beccari) indicates the program was modified to run in a specific environment (Atrium), affecting the transaction file name (`?9?CRIEGG`).
- The program includes multiple exit points (via `GOTO END`) to handle errors or conditions that prevent processing, such as an existing transaction file or specific variable states.
- The OCL syntax and structure suggest it runs on an IBM midrange system, with variables like `?L'506,3'?` referencing system or program storage locations.

If you need further clarification or have additional context (e.g., the contents of `AR157` or `GSGENIEC`), let me know!