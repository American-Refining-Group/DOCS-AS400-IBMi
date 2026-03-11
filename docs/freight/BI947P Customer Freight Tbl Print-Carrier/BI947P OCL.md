The provided document is an Operation Control Language (OCL) program, specifically `BI947P.ocl36.txt`, which appears to be a control script for an IBM System/36 or AS/400 environment. OCL is used to control job execution and file operations in these systems. Below, I’ll explain the process steps of the OCL program, identify any external programs called, and list the tables or files used.

### Process Steps of the OCL Program

The OCL program defines a sequence of operations to manage a job related to a "Customer Freight Table Master File List by Carrier ID." Here’s a step-by-step breakdown of the program’s logic:

1. **Initial Setup and Comments**:
   - The program begins with comments (lines starting with `//`) that provide metadata:
     - `CUSTOMER FREIGHT TABLE MASTER FILE LIST BY CARRIER ID`: Indicates the purpose of the program, likely generating a report or processing a freight table sorted by carrier ID.
     - `SCPROCP ,,,,,,,,?9?`: This may specify a procedure or library, with `?9?` likely being a placeholder for a parameter (e.g., a library or file name) passed at runtime.
     - `SWITCH 00000000`: Initializes the job’s switch settings to all zeros. Switches are used for conditional logic in OCL.
     - `LOCAL BLANK-*ALL`: Clears all local variables, ensuring no residual data affects the job.
     - `GSY2K`: Possibly a reference to a system library, utility, or compliance setting (e.g., Year 2000 compliance), though its exact purpose is unclear without more context.

2. **Load and File Definition**:
   - `LOAD BI947P`: Loads the program or procedure named `BI947P`. This could be an RPG program or another executable that performs the core processing.
   - `FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHRRM`:
     - Defines a file named `BICONT` with a label that includes the parameter `?9?` (e.g., `LIBNAME.BICONT`).
     - `DISP-SHRRM` indicates the file is opened in shared read mode, allowing multiple jobs to read the file simultaneously.
   - `RUN`: Initiates execution of the loaded program (`BI947P`).

3. **Conditional Logic**:
   - `IF SWITCH1-1 GOTO END`:
     - Checks if the first switch (SWITCH1) is set to 1.
     - If true, the program jumps to the `END` tag, effectively skipping further processing.
   - `IF ?L'120,1'?/Y JOBQ ?CLIB?,BI947,,,,,,,,,?9?`:
     - Tests a condition at position 120, character 1 of a parameter or data area (likely a flag or indicator).
     - If the condition is `Y` (yes), the program submits a job to the job queue (`JOBQ`) in the library specified by `?CLIB?`, running the program `BI947` with the parameter `?9?`.
   - `ELSE BI947 ,,,,,,,,?9?`:
     - If the condition is not `Y`, the program directly runs `BI947` with the parameter `?9?`, bypassing the job queue.

4. **End of Program**:
   - `TAG END`: Marks the end of the program where control jumps if `SWITCH1` is 1.
   - `SWITCH 00000000`: Resets all switches to zero.
   - `LOCAL BLANK-*ALL`: Clears all local variables again, ensuring a clean exit.

### External Programs Called

- **BI947**: This is the primary program called by the OCL script. It is likely an RPG (Report Program Generator) program responsible for processing the customer freight table data (e.g., generating a report or updating the file). It is invoked either directly or via the job queue, depending on the condition at `?L'120,1'?`.

### Tables/Files Used

- **BICONT**: The file `BICONT` is explicitly referenced with the label `?9?BICONT` and opened in shared read mode (`DISP-SHRRM`). This file likely contains the customer freight table data, such as carrier IDs and related information, used by the `BI947` program.

### Additional Notes

- **Parameters (`?9?`, `?CLIB?`)**: The placeholders `?9?` and `?CLIB?` are substitution variables, typically replaced at runtime with specific values (e.g., library names, file labels, or job parameters). These are common in OCL to make scripts reusable.
- **Conditional Logic**: The use of `SWITCH1` and the `?L'120,1'?` condition suggests the program supports conditional execution, possibly to handle different environments or modes (e.g., interactive vs. batch processing).
- **Job Queue**: The `JOBQ` statement indicates the program can submit jobs for batch processing, which is typical for resource-intensive tasks like report generation.

### Summary

- **Process Steps**:
  1. Initialize switches and clear local variables.
  2. Load the `BI947P` program and define the `BICONT` file.
  3. Run the program, with conditional logic to either skip to the end (if `SWITCH1` is 1) or execute `BI947` directly or via the job queue based on a parameter check.
  4. Reset switches and variables at the end.
- **External Programs**: `BI947` (likely an RPG program).
- **Tables/Files**: `BICONT` (customer freight table master file).

If you have additional details (e.g., the contents of `BI947` or specific parameter values), I can provide a more detailed analysis of the program’s behavior!