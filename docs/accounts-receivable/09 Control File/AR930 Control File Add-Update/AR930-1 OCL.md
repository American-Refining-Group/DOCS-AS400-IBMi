The provided document is an Operation Control Language (OCL) program snippet, typically used in IBM midrange systems like the IBM System/36 or AS/400 (now IBM i). OCL is a control language used to define job steps, manage files, and execute programs. The program `AR930.ocl36.txt` appears to be related to an Accounts Receivable (A/R) control file maintenance process. Below, I will explain the process steps, identify any external programs called, and list the tables (files) used.

---

### Process Steps of the RPG .OCL Program

The OCL program defines a sequence of steps to set up and execute a program (`AR930`) for maintaining A/R control files. Here’s a breakdown of the steps based on the provided code:

1. **Program Invocation (`// CALL PGM(GSGENIEC)`)**:
   - The OCL program starts by calling an external program named `GSGENIEC`.
   - This program is likely a utility or initialization program, possibly for setting up the environment, checking system conditions, or performing preliminary tasks before the main A/R process.
   - The exact functionality of `GSGENIEC` is not specified in the snippet, but it’s a common practice in System/36 environments to call such programs to validate the runtime environment or user permissions.

2. **Conditional Check (`// IFF ?L'506,3'?/YES RETURN`)**:
   - The `IFF` (If) statement checks a condition using a system variable or parameter `?L'506,3'?`.
   - This likely refers to a location in a control area (e.g., a Local Data Area or system register) at position 506, checking 3 characters.
   - If the condition evaluates to `YES` (true), the program executes a `RETURN` command, which terminates the OCL procedure immediately.
   - This acts as a gatekeeper, ensuring that the program proceeds only if the specified condition is not met (e.g., a specific system state, user authorization, or configuration).

3. **Procedure Call (`// SCPROCP ,,,,,,,,?9?`)**:
   - The `SCPROCP` statement invokes a procedure, passing a parameter `?9?`.
   - The commas (`,,,,,,,,`) indicate placeholders for optional parameters that are not used in this call.
   - The `?9?` is a substitution variable, likely representing a library, file, or runtime value (e.g., a specific A/R control file or environment setting).
   - This step may set up additional processing parameters or invoke a sub-procedure related to A/R processing.

4. **GSY2K Execution (`// GSY2K`)**:
   - The `GSY2K` command calls a program or utility named `GSY2K`.
   - This is likely a Year 2000 (Y2K) compliance utility or a date-handling program, common in older systems to manage date formats or ensure proper date calculations.
   - Its role here might be to initialize date-related settings or validate date data before the main A/R program runs.

5. **Load Main Program (`// LOAD AR930`)**:
   - The `LOAD` command loads the main program `AR930`, which is presumably an RPG (Report Program Generator) program responsible for A/R control file maintenance.
   - This program likely performs the core logic, such as updating, validating, or querying A/R control records.

6. **File Definitions**:
   - Three files are defined for use by the `AR930` program:
     - `ARCONT`: The A/R control file, labeled as `?9?ARCONT` with disposition `DISP-SHR` (shared access). This file likely contains control data, such as A/R configuration settings, totals, or parameters.
     - `ARCUST`: The A/R customer file, labeled as `?9?ARCUST` with `DISP-SHR`. This file likely stores customer account details, such as balances, credit limits, or transaction summaries.
     - `GLMAST`: The General Ledger master file, labeled as `?9?GLMAST` with `DISP-SHR`. This file contains G/L account data, likely used for posting A/R transactions to the general ledger.
   - The `?9?` prefix in the file labels suggests a library or environment variable (e.g., a specific library name or default system library).
   - The `DISP-SHR` indicates that these files can be accessed by multiple jobs or programs concurrently without exclusive locks.

7. **Program Execution (`// RUN`)**:
   - The `RUN` command executes the loaded `AR930` program.
   - The program uses the defined files (`ARCONT`, `ARCUST`, `GLMAST`) to perform its A/R control file maintenance tasks, such as updating control records, validating customer data, or posting to the general ledger.

---

### External Programs Called

The OCL program explicitly calls the following external programs:
1. **GSGENIEC**:
   - Called at the start of the program, likely for initialization or environment setup.
2. **GSY2K**:
   - Called before loading `AR930`, possibly for date-related processing or Y2K compliance.
3. **AR930**:
   - The main RPG program loaded and executed for A/R control file maintenance.

Additionally, the `SCPROCP` statement may invoke a sub-procedure, but it’s unclear if this is a separate program or an internal procedure within the OCL environment.

---

### Tables (Files) Used

The program references the following files (tables):
1. **ARCONT**:
   - Description: Accounts Receivable control file.
   - Label: `?9?ARCONT`.
   - Disposition: `DISP-SHR` (shared access).
   - Purpose: Stores A/R configuration data, such as control totals, aging parameters, or system settings.
2. **ARCUST**:
   - Description: Accounts Receivable customer file.
   - Label: `?9?ARCUST`.
   - Disposition: `DISP-SHR` (shared access).
   - Purpose: Contains customer-specific data, such as account details, balances, or credit information.
3. **GLMAST**:
   - Description: General Ledger master file.
   - Label: `?9?GLMAST`.
   - Disposition: `DISP-SHR` (shared access).
   - Purpose: Stores G/L account data for posting A/R transactions.

---

### Summary

The `AR930.ocl36.txt` OCL program orchestrates an Accounts Receivable control file maintenance process by:
1. Initializing the environment with `GSGENIEC`.
2. Checking a condition to determine whether to proceed.
3. Setting up parameters or a sub-procedure with `SCPROCP`.
4. Handling date-related processing with `GSY2K`.
5. Loading and executing the `AR930` RPG program, which uses three files (`ARCONT`, `ARCUST`, `GLMAST`) for A/R maintenance tasks.

**External Programs Called**: `GSGENIEC`, `GSY2K`, `AR930`.  
**Tables Used**: `ARCONT`, `ARCUST`, `GLMAST`.

If you need further details about the `AR930` RPG program’s logic or the specific roles of `GSGENIEC` or `GSY2K`, additional context (e.g., the RPG source code or system documentation) would be required. Let me know if you have more information or specific questions!