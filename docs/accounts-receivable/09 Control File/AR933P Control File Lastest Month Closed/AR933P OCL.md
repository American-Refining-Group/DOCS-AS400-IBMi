The provided document is an Operation Control Language (OCL) program, `AR933P.ocl36.txt`, used in IBM midrange systems (like AS/400 or iSeries) to control job execution and interact with RPG programs. OCL is a scripting language for defining job steps, file operations, and program flow in such environments. Below, I’ll explain the process steps of this OCL program, identify external programs called, and list the tables (files) used, based on the provided code.

### Process Steps of the OCL Program

The OCL program `AR933P` appears to manage an accounts receivable update process, likely interacting with an RPG program to update the `ARCONT` table based on a month/year closed value. Here’s a step-by-step breakdown of the process:

1. **Program Invocation (`// CALL PGM(GSGENIEC)`):
   - The program starts by calling an external program named `GSGENIEC`.
   - This is likely a utility or initialization program that performs setup tasks, such as validating the environment, checking user authority, or preparing data for the accounts receivable update.
   - Since no parameters are explicitly passed, `GSGENIEC` likely operates on predefined system variables or files.

2. **Conditional Check (`// IFF ?L'506,3'?/YES RETURN`):
   - The `IFF` statement checks a condition using a system variable or field at location `L'506,3'`. This could be a status code, flag, or value stored in a specific memory location or data area.
   - The notation `?L'506,3'?` suggests checking a 3-byte field starting at position 506.
   - If the condition evaluates to `YES` (true), the program executes the `RETURN` command, which terminates the OCL program immediately, halting further execution.
   - This acts as a gatekeeper, ensuring the program only proceeds if the condition is false (e.g., a specific status is not set).

3. **Procedure Call (`// SCPROCP ,,,,,,,,?9?`):
   - The `SCPROCP` command invokes a system procedure or utility, passing a parameter `?9?` in the ninth position (indicated by the commas, which act as placeholders for eight empty parameters).
   - The `?9?` likely references a variable or value (e.g., a job number, user ID, or control value) defined elsewhere in the system or passed to the OCL program.
   - The exact purpose of `SCPROCP` is unclear without system context, but it might perform tasks like setting up a job queue, logging, or initializing system resources for the accounts receivable process.

4. **Set Local Variable (`// LOCAL BLANK-*ALL`):
   - This command initializes all local variables in the OCL program’s scope to blanks (empty values).
   - This ensures that any previously set values in the local variable space are cleared, preventing unintended carryover of data from prior runs or jobs.

5. **Date Conversion (`// GSY2K`):
   - The `GSY2K` command invokes a system utility or procedure to handle date processing, likely ensuring dates are in a two-digit year format (e.g., for Y2K compliance or legacy date handling).
   - This step might adjust or validate date fields used in the accounts receivable update, such as the month/year closed value mentioned in the program’s header.

6. **Load RPG Program (`// LOAD AR933P`):
   - The `LOAD` command loads the RPG program named `AR933P` into memory for execution.
   - This suggests that `AR933P` is both the name of the OCL program and the associated RPG program, a common practice where the OCL script sets up the environment for an RPG program with the same name.
   - The RPG program likely contains the core logic for updating the `ARCONT` table.

7. **File Definition (`// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`):
   - This defines a file named `ARCONT` to be used by the RPG program.
   - The `LABEL-?9?ARCONT` clause indicates that the file’s label (or alias) includes a variable `?9?`, which might dynamically specify a library, member, or version of the `ARCONT` file (e.g., a specific dataset or period).
   - `DISP-SHR` (disposition shared) allows multiple jobs to access the file simultaneously, typically in read-only or controlled update mode, preventing exclusive locks.
   - The `ARCONT` file is likely the accounts receivable control table, storing data like closed month/year values or summary totals.

8. **Execute Program (`// RUN`):
   - The `RUN` command executes the loaded RPG program (`AR933P`).
   - The RPG program processes the `ARCONT` file, performing the accounts receivable update (e.g., updating the month/year closed value or related financial data).

9. **Error Check (`// IF ?L'129,6'?/CANCEL GOTO END`):
   - After running the RPG program, the `IF` statement checks a condition at location `L'129,6'`, likely a return code or status flag set by the RPG program.
   - If the condition is true (e.g., an error occurred), the `CANCEL` action terminates the job, and control jumps to the `END` tag, ending the program.
   - This acts as an error-handling mechanism to stop processing if the RPG program fails (e.g., due to invalid data or file issues).

10. **End Tag (`// TAG END`):
    - The `END` tag marks a label where the program can jump, in this case, used to terminate execution after an error or normal completion.
    - No additional logic follows, so the program ends here if the error condition is met or after normal execution.

11. **Clear Local Variables Again (`// LOCAL BLANK-*ALL`):
    - This second `LOCAL BLANK-*ALL` command clears all local variables again, ensuring no residual data remains in memory after the program completes.
    - This is a cleanup step to maintain a clean environment for subsequent jobs or to free resources.

### External Programs Called

The OCL program explicitly calls the following external programs or procedures:
1. **GSGENIEC**:
   - Called via `// CALL PGM(GSGENIEC)`.
   - Likely a system utility or initialization program for setting up the job environment.
2. **SCPROCP**:
   - Invoked via `// SCPROCP ,,,,,,,,?9?`.
   - A system procedure, possibly for job control, logging, or resource allocation.
3. **GSY2K**:
   - Invoked via `// GSY2K`.
   - A date-handling utility, likely for Y2K-compliant date processing or validation.
4. **AR933P** (RPG program):
   - Loaded and executed via `// LOAD AR933P` and `// RUN`.
   - The core RPG program that performs the accounts receivable update logic.

### Tables (Files) Used

The program references the following table (file):
1. **ARCONT**:
   - Defined via `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`.
   - This is the accounts receivable control table, likely containing data such as closed month/year values, account totals, or control records for the accounts receivable process.
   - The `?9?` in the label suggests a dynamic or parameterized file name, possibly indicating a specific library or member.

### Summary

The `AR933P` OCL program orchestrates an accounts receivable update process by:
- Initializing the environment with `GSGENIEC` and checking a condition to proceed.
- Calling a system procedure (`SCPROCP`) and handling date formatting (`GSY2K`).
- Loading and running the `AR933P` RPG program, which processes the `ARCONT` file.
- Checking for errors and cleaning up local variables.

**External Programs Called**: `GSGENIEC`, `SCPROCP`, `GSY2K`, `AR933P` (RPG).
**Tables Used**: `ARCONT`.

If you need further clarification or analysis (e.g., diving into the RPG program’s logic or specific system context), please let me know!