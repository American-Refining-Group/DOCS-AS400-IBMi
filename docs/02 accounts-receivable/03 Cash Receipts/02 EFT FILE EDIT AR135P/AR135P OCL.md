The provided document is an **OCL (Operation Control Language)** program, specifically `AR135P.ocl36.txt`, which is used on IBM System/36 or AS/400 systems to control job execution and interact with RPG (Report Program Generator) programs. Below, I will explain the process steps of this OCL program, identify any external programs called, and list any tables or files used.

---

### Process Steps of the AR135P OCL Program

OCL is a scripting language used to automate and control job execution on IBM midrange systems. The program `AR135P` defines a sequence of operations to load and execute a program, manage files, and control program flow based on conditions. Here’s a step-by-step breakdown of the process:

1. **Comment Block**:
   ```
   **
   ** AR EFT INDIVIDUAL ENTRY
   **
   ```
   - This is a descriptive comment indicating the purpose of the program, likely related to "Accounts Receivable Electronic Funds Transfer (EFT) Individual Entry."

2. **Call External Program (`GSGENIEC`)**:
   ```
   // CALL PGM(GSGENIEC)
   ```
   - The OCL program invokes an external program named `GSGENIEC`. This program is likely a utility or setup program, possibly for initializing the environment or performing prerequisite checks.
   - The specific functionality of `GSGENIEC` is not defined in the OCL script and would depend on its implementation.

3. **Conditional Check on Location 506**:
   ```
   // IFF ?L'506,3'?/YES   RETURN
   ```
   - This line checks the value at memory location 506 (3 bytes long, denoted by `?L'506,3'?`).
   - If the condition evaluates to `YES` (true), the program executes a `RETURN` statement, which terminates the OCL procedure immediately.
   - This suggests a conditional exit mechanism, possibly checking a system or program status flag.

4. **GSY2K Command**:
   ```
   // GSY2K
   ```
   - This invokes the `GSY2K` command, which is likely a system utility or procedure related to Year 2000 (Y2K) compliance or date handling.
   - The exact function depends on the system configuration, but it may involve setting or validating date-related parameters.

5. **Set Local Variable**:
   ```
   // LOCAL OFFSET-480,DATA-'?9?'
   ```
   - This sets a local variable at memory offset 480 with the value `?9?`.
   - The `?9?` is a substitution variable, meaning its actual value is resolved at runtime, possibly from a parameter passed to the program or a system value.
   - This variable is likely used by the program `AR135P` or its associated files.

6. **Load Program `AR135P`**:
   ```
   // LOAD AR135P
   ```
   - This loads the RPG program `AR135P` into memory for execution.
   - `AR135P` is the main program that performs the core processing for the EFT individual entry.

7. **Specify File `ARCONT`**:
   ```
   // FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR
   ```
   - This defines a file named `ARCONT` to be used by the program.
   - The `LABEL-?9?ARCONT` indicates that the file’s label is dynamically resolved using the substitution variable `?9?` concatenated with `ARCONT`. For example, if `?9?` is `PROD`, the file label might be `PRODARCONT`.
   - `DISP-SHR` specifies that the file is opened in shared mode, allowing other programs or jobs to access it concurrently.

8. **Run the Program**:
   ```
   // RUN
   ```
   - This executes the loaded program `AR135P`, which processes the EFT-related data using the `ARCONT` file.

9. **Check Switch 8**:
   ```
   // IF SWITCH8-1  GOTO END
   ```
   - This checks the status of **Switch 8** (a system or program control switch).
   - If Switch 8 is set to `1` (on), the program branches to the `END` tag, skipping the next step.
   - Switches are often used to control program flow or indicate specific conditions (e.g., error states or configuration settings).

10. **Execute `AR135` with Parameter**:
    ```
    // AR135 ,,,,,,,,?9?
    ```
    - This invokes a procedure or command named `AR135`, passing eight commas (placeholders for parameters) followed by the substitution variable `?9?`.
    - The commas suggest that `AR135` expects multiple parameters, but only the last one is provided (via `?9?`). The placeholders may indicate optional or unused parameters.
    - `AR135` is likely another program or procedure related to the EFT process, possibly a continuation or secondary process after `AR135P`.

11. **Tag for End of Processing**:
    ```
    // TAG END
    ```
    - This defines a label (`END`) that the program can jump to, used in the `IF SWITCH8-1 GOTO END` statement.
    - It marks the point where processing skips to if Switch 8 is set.

12. **Clear Local Variables**:
    ```
    // LOCAL BLANK-*ALL
    ```
    - This clears all local variables defined in the OCL procedure, resetting them to blank or null values.
    - This is a cleanup step to ensure no residual data affects subsequent runs.

13. **Reset Switches**:
    ```
    // SWITCH 00000000
    ```
    - This resets all control switches (1 through 8) to `0` (off).
    - This ensures a clean state for the next execution or job.

---

### External Programs Called

The OCL program explicitly calls the following external programs or procedures:
1. **GSGENIEC**: Called at the start, likely a setup or initialization program.
2. **AR135**: Called with the substitution variable `?9?` as a parameter, possibly a secondary program or procedure related to EFT processing.
3. **AR135P**: The main RPG program loaded and executed, responsible for the core EFT processing.

Additionally, the `GSY2K` command is invoked, which may be a system utility or procedure rather than a standalone program.

---

### Tables/Files Used

The OCL program references the following file:
1. **ARCONT**:
   - File name: `ARCONT`
   - Label: Dynamically resolved as `?9?ARCONT` (e.g., `PRODARCONT` if `?9?` is `PROD`).
   - Disposition: `DISP-SHR` (shared access).
   - Purpose: Likely a database file containing Accounts Receivable or EFT-related data, used by the `AR135P` program.

No other tables or files are explicitly mentioned in the OCL script. However, the RPG program `AR135P` or the procedure `AR135` may use additional files or tables defined within their logic, which are not visible in the OCL.

---

### Summary

The `AR135P` OCL program automates the execution of an Accounts Receivable EFT process. It performs the following key steps:
- Calls an initialization program (`GSGENIEC`).
- Checks a condition at memory location 506 to decide whether to exit early.
- Invokes a Y2K-related utility (`GSY2K`).
- Sets a local variable with a dynamic value (`?9?`).
- Loads and runs the `AR135P` RPG program, using the `ARCONT` file.
- Conditionally executes the `AR135` procedure based on Switch 8.
- Cleans up by clearing variables and resetting switches.

**External Programs Called**:
- `GSGENIEC`
- `AR135`
- `AR135P`

**Files Used**:
- `ARCONT` (label: `?9?ARCONT`, shared access)

If you need further details about the RPG program `AR135P` or the `ARCONT` file structure, additional information (e.g., the RPG source code or file definitions) would be required, as these are not included in the OCL script.