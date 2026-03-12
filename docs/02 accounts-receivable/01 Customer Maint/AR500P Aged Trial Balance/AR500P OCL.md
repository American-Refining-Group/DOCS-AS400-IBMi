The provided document is an OCL (Operation Control Language) program, `AR500P.ocl36.txt`, used on IBM midrange systems like the AS/400 (now IBM i) to control job execution and interact with programs and files. This OCL script appears to be part of an Accounts Receivable (A/R) Aged Trial Balance process. Below, I’ll explain the process steps, identify external programs called, and list the tables (files) used.

### Process Steps of the OCL Program

OCL is a scripting language used to automate job control and program execution on IBM midrange systems. The `AR500P` OCL script orchestrates the execution of an Accounts Receivable process. Here’s a step-by-step breakdown of the process:

1. **Program Call to GSGENIEC**:
   - `// CALL PGM(GSGENIEC)`:
     - The script starts by calling an external program named `GSGENIEC`. This is likely a utility or initialization program that performs setup tasks, such as setting up the environment or validating parameters.
     - The purpose of `GSGENIEC` is not specified in the OCL, but it could handle tasks like date validation, user authorization, or library list setup.

2. **Conditional Check on Location 506, Position 3**:
   - `// IFF ?L'506,3'?/YES RETURN`:
     - This checks the value at memory location 506, position 3 (likely a system or program variable).
     - If the condition evaluates to `YES` (true), the script executes a `RETURN`, which terminates the OCL procedure immediately, halting further execution.
     - This acts as an early exit condition, possibly to skip processing if a specific flag or parameter is set.

3. **Procedure Call with Parameter**:
   - `// SCPROCP ,,,,,,,,?9?`:
     - This invokes a procedure named `SCPROCP`, passing a parameter referenced as `?9?`.
     - The commas indicate placeholder parameters (up to 8) that are not used here.
     - `?9?` is a substitution variable, likely representing a dynamic value like a company code, report date, or job parameter.
     - The purpose of `SCPROCP` is not detailed, but it could be a system procedure for setting up job control or environment variables.

4. **Clear Local Variables**:
   - `// LOCAL BLANK-*ALL`:
     - This clears all local variables in the job’s memory to a blank state, ensuring no residual data affects subsequent steps.

5. **Call to GSY2K**:
   - `// GSY2K`:
     - This invokes a program or procedure named `GSY2K`, likely related to Year 2000 (Y2K) date handling or conversion.
     - On IBM midrange systems, `GSY2K` utilities were common for ensuring date fields were processed correctly (e.g., converting two-digit years to four-digit years).
     - This step ensures date-related data in the A/R process is Y2K-compliant.

6. **Load Program AR500P**:
   - `// LOAD AR500P`:
     - This loads the main program `AR500P` into memory for execution.
     - `AR500P` is likely an RPG (Report Program Generator) program responsible for generating the A/R Aged Trial Balance report.

7. **File Declaration for ARCONT**:
   - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHRMM`:
     - Declares a file named `ARCONT` with a label that includes the substitution variable `?9?` (e.g., a company or library identifier).
     - `DISP-SHRMM` indicates the file is opened in shared mode, allowing multiple jobs to access it simultaneously (read-only or read/write depending on the program).
     - `ARCONT` is likely the Accounts Receivable control file containing customer balances, aging buckets, or transaction data.

8. **Run the Loaded Program**:
   - `// RUN`:
     - Executes the loaded `AR500P` program, which processes the A/R data and generates the Aged Trial Balance report.
     - The program likely reads from the `ARCONT` file and performs calculations to categorize receivables into aging buckets (e.g., current, 30 days, 60 days, etc.).

9. **Conditional Check on Location 124, Position 6**:
   - `// IF ?L'124,6'?/CANCEL GOTO END`:
     - Checks the value at memory location 124, position 6.
     - If the condition evaluates to `CANCEL` (true), the script jumps to the `END` tag, terminating the job.
     - This could be a check for an error condition or user-initiated cancellation.

10. **Conditional Job Queue Submission**:
    - `// IF ?L'120,1'?/Y JOBQ ?CLIB?,AR500,,,,,,,,,?9?,,?11?`:
      - Checks the value at memory location 120, position 1.
      - If the condition is `Y` (true), the `AR500` program is submitted to a job queue specified by `?CLIB?` (a library or job queue name), with parameters `?9?` and `?11?`.
      - This implies the A/R process can run in batch mode, possibly for large datasets or scheduled processing.
    - `// ELSE AR500 ,,,,,,,,?9?,,?11?`:
      - If the condition is false, the `AR500` program is executed directly (not in a job queue) with the same parameters `?9?` and `?11?`.
      - The commas again represent unused parameter positions.
      - `?11?` is another substitution variable, possibly for additional report options or filters.

11. **End Tag**:
    - `// TAG END`:
      - Marks the `END` label, used as a target for the `GOTO END` statement in case of cancellation.

12. **Clear Local Variables Again**:
    - `// LOCAL BLANK-*ALL`:
      - Clears all local variables again at the end of the script, ensuring a clean exit.

### External Programs Called

The OCL script explicitly calls or references the following external programs or procedures:
1. **GSGENIEC**:
   - Called at the start, likely a utility for environment setup or validation.
2. **SCPROCP**:
   - A procedure invoked with the `?9?` parameter, possibly for job control or configuration.
3. **GSY2K**:
   - A program or procedure for Y2K date handling, ensuring proper date processing.
4. **AR500P**:
   - The main RPG program loaded and executed to generate the A/R Aged Trial Balance.
5. **AR500**:
   - Referenced in the job queue or direct execution step, likely the same program as `AR500P` or a related module.

### Tables (Files) Used

The OCL script explicitly references the following file:
1. **ARCONT**:
   - File name: `ARCONT`
   - Label: `?9?ARCONT` (dynamic label based on the `?9?` parameter)
   - Disposition: `SHRMM` (shared mode)
   - Purpose: Likely the Accounts Receivable control file containing customer data, balances, or transaction details used for the Aged Trial Balance report.

No other files are explicitly mentioned, but the `AR500P` program may access additional files (e.g., customer master, transaction history) not declared in the OCL.

### Additional Notes
- **Substitution Variables**:
  - `?9?` and `?11?` are placeholders for dynamic values passed to the script, such as company codes, report dates, or job parameters.
  - `?CLIB?` likely represents a library name for the job queue.
- **Conditional Logic**:
  - The script uses memory location checks (`?L'506,3'?`, `?L'124,6'?`, `?L'120,1'?`) to control flow, typical in OCL for decision-making based on system or program states.
- **Job Queue vs. Direct Execution**:
  - The script supports both batch (via `JOBQ`) and interactive execution of `AR500`, providing flexibility for different processing needs.
- **A/R Aged Trial Balance**:
  - The process generates a report categorizing receivables by aging periods, commonly used for financial reporting and credit management.

If you have additional details (e.g., the contents of `AR500P` or the structure of `ARCONT`), I can provide a more detailed analysis of the RPG program’s logic or file interactions. Let me know if you’d like me to search for related information or analyze specific aspects further!