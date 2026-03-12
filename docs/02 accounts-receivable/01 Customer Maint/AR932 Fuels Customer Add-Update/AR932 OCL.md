Let’s break down the provided **A/R Fuels Customer Maintenance** program, which is written in **Operation Control Language (OCL)**, commonly used on IBM midrange systems like the IBM System/36. OCL is a job control language used to define and execute programs, manage files, and control job flow. The program snippet provided appears to be part of a batch process for maintaining customer data in an Accounts Receivable (A/R) system for a fuels-related application.

Below, I’ll explain the **process steps** of the OCL program, identify the **external programs called**, and list the **tables (files)** used, based on the provided code.

---

### Process Steps of the RPG .OCL Program

The OCL code provided is a control script that sets up the environment and executes a program, likely written in RPG (Report Program Generator), a language commonly used with OCL on IBM systems. Here’s a step-by-step explanation of the process:

1. **Program Invocation (`// CALL PGM(GSGENIEC)`)**:
   - The `CALL` statement initiates the execution of a program named `GSGENIEC`.
   - This is likely a utility or initialization program that sets up the environment or performs a prerequisite task for the A/R Fuels Customer Maintenance process.
   - The exact functionality of `GSGENIEC` is not specified in the code, but it could handle tasks like validating the environment, checking user permissions, or preparing data for subsequent steps.

2. **Conditional Check (`// IFF ?L'506,3'?/YES RETURN`)**:
   - The `IFF` (If) statement checks a condition using a system variable or parameter `?L'506,3'?`.
   - The `L` likely refers to a **local data area** or a system variable at position 506, with a length of 3 characters.
   - If the condition evaluates to `YES` (true), the program executes the `RETURN` command, which terminates the OCL procedure immediately.
   - This acts as a gatekeeper, ensuring that the rest of the program only runs if the condition is false (e.g., a specific status or flag is not set).

3. **Procedure Call (`// SCPROCP ,,,,,,,,?9?`)**:
   - The `SCPROCP` command appears to invoke a **system procedure** or a subroutine, passing parameters.
   - The commas (`,,,,,,,,`) indicate placeholder parameters, and `?9?` is a substitution variable (likely a parameter passed to the procedure, such as a library or file identifier).
   - The exact purpose of `SCPROCP` is unclear without more context, but it might set up additional environment settings, call a subroutine, or perform a system-level operation.

4. **Environment Setup (`// GSY2K`)**:
   - The `GSY2K` command is likely a custom or system-specific command related to Year 2000 (Y2K) compliance or date handling.
   - It may initialize date-related settings or ensure that the system handles dates correctly (e.g., using a four-digit year format).
   - This step ensures the program operates in a Y2K-compliant environment, which was critical for older systems processing dates.

5. **Program Load (`// LOAD AR932`)**:
   - The `LOAD` statement loads the main program, `AR932`, into memory for execution.
   - `AR932` is likely an RPG program responsible for the core A/R Fuels Customer Maintenance logic, such as updating customer records, processing fuel-related transactions, or generating reports.
   - This step prepares the system to run the actual business logic.

6. **File Definitions**:
   - The `FILE` statements define the files (tables) that the `AR932` program will use. Each file is associated with a specific dataset and access mode:
     - `FILE NAME-ARFUEL,LABEL-?9?ARFUEL,DISP-SHR`:
       - Defines a file named `ARFUEL` (likely the A/R Fuels file).
       - The `LABEL-?9?ARFUEL` specifies the file’s label, where `?9?` is a substitution variable (e.g., a library or prefix like `QSYS/` or a specific library name).
       - `DISP-SHR` indicates the file is opened in **shared mode**, allowing multiple programs or users to access it concurrently.
     - `FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`:
       - Defines a file named `ARCUST` (likely the A/R Customer file).
       - Similar to `ARFUEL`, it uses a substitution variable for the label and is opened in shared mode.
     - `FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`:
       - Defines a file named `ARCONT` (likely the A/R Control file, which may store configuration or summary data).
       - Also uses a substitution variable and shared mode.
   - These files are critical for the `AR932` program to read or update customer and fuel-related data.

7. **Program Execution (`// RUN`)**:
   - The `RUN` statement executes the loaded `AR932` program.
   - This is where the actual customer maintenance logic is performed, using the files defined above (`ARFUEL`, `ARCUST`, `ARCONT`).
   - The program likely performs tasks like adding, updating, or deleting customer records, processing fuel transactions, or generating maintenance reports.

---

### External Programs Called

The OCL program explicitly calls the following external program:
1. **GSGENIEC**:
   - Called via `// CALL PGM(GSGENIEC)`.
   - Likely a utility or initialization program that performs setup tasks before the main logic.

Additionally, the main program executed is:
2. **AR932**:
   - Loaded and run via `// LOAD AR932` and `// RUN`.
   - This is the core RPG program responsible for A/R Fuels Customer Maintenance.

The `SCPROCP` command may also invoke a system procedure or subroutine, but it’s unclear whether this is a distinct program or a system function. Without further context, I’ll treat it as a system-level operation rather than a named external program.

---

### Tables (Files) Used

The program uses the following files (tables), as defined in the `FILE` statements:
1. **ARFUEL**:
   - Likely contains fuel-specific accounts receivable data, such as fuel purchase records, transaction details, or billing information for fuel customers.
   - Label: `?9?ARFUEL` (with `?9?` as a substitution variable for the library or prefix).
   - Access mode: Shared (`DISP-SHR`).

2. **ARCUST**:
   - Likely the customer master file, storing customer details such as account numbers, names, addresses, and credit information.
   - Label: `?9?ARCUST`.
   - Access mode: Shared (`DISP-SHR`).

3. **ARCONT**:
   - Likely a control file containing configuration data, summary totals, or control parameters for the A/R system (e.g., batch numbers, processing flags, or system settings).
   - Label: `?9?ARCONT`.
   - Access mode: Shared (`DISP-SHR`).

---

### Summary

- **Process Steps**:
  1. Calls `GSGENIEC` to initialize or validate the environment.
  2. Checks a condition (`?L'506,3'?`) and returns if true, halting further execution.
  3. Invokes `SCPROCP` with a parameter (`?9?`), possibly for system setup.
  4. Sets up Y2K compliance with `GSY2K`.
  5. Loads the main program `AR932`.
  6. Defines three files (`ARFUEL`, `ARCUST`, `ARCONT`) for use by `AR932`.
  7. Runs `AR932` to perform customer maintenance tasks.

- **External Programs Called**:
  1. `GSGENIEC` (initialization utility).
  2. `AR932` (main RPG program for A/R Fuels Customer Maintenance).

- **Tables (Files) Used**:
  1. `ARFUEL` (fuel transaction data).
  2. `ARCUST` (customer master data).
  3. `ARCONT` (control or configuration data).

If you have additional details about the system (e.g., IBM System/36 specifics, the purpose of `GSGENIEC`, or the `SCPROCP` command), I can refine the explanation further. Let me know if you need more analysis or clarification!