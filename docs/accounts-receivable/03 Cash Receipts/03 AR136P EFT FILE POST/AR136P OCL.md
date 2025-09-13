The provided document is an OCL (Operation Control Language) program, specifically `AR136P.ocl36.txt`, which is used on IBM midrange systems (like AS/400 or IBM i) to control the execution of RPG programs and manage file operations. Below, I’ll explain the process steps of this OCL program, identify any external programs called, and list the tables or files referenced.

### Process Steps of the OCL Program

OCL is a scripting language used to automate tasks and control program execution on IBM midrange systems. The steps in this OCL program are as follows:

1. **Comment Block**:
   ```ocl
   **
   ** AR EFT INDIVIDUAL ENTRY
   **
   ```
   - This is a comment describing the purpose of the program, indicating it is related to "AR EFT Individual Entry" (likely Accounts Receivable Electronic Funds Transfer processing).

2. **Call External Program**:
   ```ocl
   // CALL PGM(GSGENIEC)
   ```
   - The `CALL` statement invokes an external program named `GSGENIEC`. This program is likely a utility or setup program that performs some preliminary processing or validation before the main logic of `AR136P` proceeds.
   - No parameters are explicitly passed to `GSGENIEC` in this call.

3. **Conditional Check and Return**:
   ```ocl
   // IFF ?L'506,3'?/YES   RETURN
   ```
   - The `IFF` (If) statement checks a condition using a substitution expression `?L'506,3'?`. This likely references a location in a data area, control record, or system variable (positions 506 to 508, for 3 characters).
   - If the condition evaluates to `YES`, the program executes the `RETURN` command, which terminates the OCL procedure immediately, halting further execution.
   - This acts as an early exit mechanism, possibly checking a flag or status to determine if the rest of the program should run.

4. **Date Conversion Utility**:
   ```ocl
   // GSY2K
   ```
   - The `GSY2K` command invokes a system utility, likely related to date handling or Year 2000 compliance. This utility might adjust date formats or perform date-related validations to ensure compatibility with Y2K standards.
   - This step ensures that any date-related processing in the subsequent program (`AR136P`) is handled correctly.

5. **Load the Main Program**:
   ```ocl
   // LOAD AR136P
   ```
   - The `LOAD` command loads the RPG program `AR136P` into memory, preparing it for execution. This is the main program that performs the core processing for the AR EFT Individual Entry.

6. **File Specification**:
   ```ocl
   // FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR
   ```
   - This defines a file named `ARCONT` to be used by the `AR136P` program.
   - The `LABEL-?9?ARCONT` indicates that the file’s label is dynamically substituted with a value from position 9 in a substitution expression (possibly a library or prefix).
   - `DISP-SHR` specifies that the file is opened in shared mode, allowing multiple processes to access it simultaneously (read-only or controlled access).
   - This file is likely a key data file (e.g., an Accounts Receivable control file) used by the `AR136P` program.

7. **Run the Program**:
   ```ocl
   // RUN
   ```
   - The `RUN` command executes the loaded `AR136P` program, which performs the main business logic for processing AR EFT individual entries.

8. **Second Conditional Check**:
   ```ocl
   // IFF ?L'109,1'?/Y  GOTO END
   ```
   - Another `IFF` statement checks a condition at position 109 (1 character long) in a data area or control record.
   - If the condition evaluates to `Y` (yes), the program branches to the `END` label, skipping the next step.
   - This condition likely checks a status flag or result from the `AR136P` program to determine whether further processing is needed.

9. **Invoke AR136 Procedure**:
   ```ocl
   // AR136 ,,,,,,,,?9?
   ```
   - This line invokes another OCL procedure or program named `AR136`, passing nine parameters (eight commas indicate placeholders for parameters, with the ninth being a substitution expression `?9?`).
   - The `?9?` substitution likely provides a dynamic value (e.g., a library, file prefix, or control value) to the `AR136` procedure.
   - This step might handle additional processing, such as generating reports, updating files, or performing post-processing tasks related to the EFT entries.

10. **Clear Local Storage**:
    ```ocl
    // LOCAL BLANK-*ALL
    ```
    - The `LOCAL BLANK-*ALL` command clears all local storage areas used by the OCL procedure. This ensures that any temporary variables or data areas are reset, preventing data leakage or interference with subsequent runs.

11. **End Label**:
    ```ocl
    // TAG END
    ```
    - The `TAG END` defines a label named `END`, which serves as a target for the `GOTO END` command in the earlier conditional check.
    - When the program reaches this point (or branches to it), the OCL procedure terminates.

### Summary of Process Flow
1. Call `GSGENIEC` to perform initial setup or validation.
2. Check a condition at position 506 (3 characters). If true, exit the program.
3. Run the `GSY2K` utility for date handling.
4. Load the `AR136P` RPG program and specify the `ARCONT` file in shared mode.
5. Execute `AR136P` to process AR EFT entries.
6. Check a condition at position 109 (1 character). If true, skip to the end.
7. If the condition is false, call the `AR136` procedure with dynamic parameters.
8. Clear all local storage.
9. End the program.

### External Programs Called
1. **GSGENIEC**: A utility or setup program called at the start, possibly for initialization or validation.
2. **GSY2K**: A system utility for date handling or Y2K compliance.
3. **AR136P**: The main RPG program loaded and executed to process AR EFT individual entries.
4. **AR136**: An additional OCL procedure or program called conditionally with dynamic parameters.

### Tables/Files Used
1. **ARCONT**:
   - File name: `ARCONT`
   - Label: Dynamically substituted via `?9?ARCONT`
   - Disposition: Shared (`DISP-SHR`)
   - Purpose: Likely an Accounts Receivable control file containing data needed for EFT processing.

### Notes
- The substitution expressions (`?L'506,3'?`, `?L'109,1'?`, `?9?`) suggest that this OCL program relies on dynamic values from a data area, control record, or job parameters. The exact meaning of these positions (506, 109, 9) would depend on the system’s configuration or data area definitions.
- The `AR136P` RPG program is the core of the processing, but its specific logic (e.g., calculations, file updates) is not detailed in the OCL. It likely processes EFT transactions using the `ARCONT` file.
- The `AR136` procedure might be a follow-up step for additional processing, such as generating output or updating related files.

If you have additional details (e.g., the RPG source code for `AR136P`, the structure of `ARCONT`, or the purpose of `GSGENIEC`), I can provide a more detailed explanation of the program’s functionality.