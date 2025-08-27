The provided document is an **Operation Control Language (OCL)** program, typically used in IBM System/3x or AS/400 environments to manage batch processing tasks, such as updating rack pricing files. Below, I’ll explain the process steps of the program, list the external programs called, and identify the tables (files) used.

### Process Steps of the RPG .OCL Program

The OCL program `BB954P.ocl36.txt` is designed to update rack pricing files with new prices for "DA" (likely a specific pricing category or customer type). It performs a series of operations involving file manipulation, conditional logic, and program execution. Here’s a breakdown of the steps:

1. **Conditional Logic Based on Parameter (?1?)**:
   - The program checks the value of parameter `?1?` (likely passed during program invocation).
   - If `?1?` equals "POST", it sets a switch to `10000000` (enabling specific processing logic, possibly for posting updates).
   - Otherwise, it sets the switch to `00000000` (default or non-posting mode).
   - **OCL Syntax**:
     ```ocl
     // IF ?1?/POST   SWITCH 10000000
     // ELSE          SWITCH 00000000
     ```

2. **Delete Temporary File (if exists)**:
   - Checks if a file labeled `?9?BB954S` exists in `DATAF1`.
   - If found, it deletes the file to ensure a clean slate for subsequent steps.
   - **OCL Syntax**:
     ```ocl
     // IF DATAF1-?9?BB954S   DELETE ?9?BB954S,F1
     ```

3. **Create Customer List from PRCTUM (Program BB9541)**:
   - Loads and executes the program `BB9541`.
   - Defines two files:
     - `PRCTUM` (input file, shared access with `DISP-SHR`).
     - `BB954S` (output file, temporary with 999,000 records, retained with `RETAIN-T`).
   - This step likely processes the `PRCTUM` file to generate a customer list stored in `BB954S`.
   - **OCL Syntax**:
     ```ocl
     // LOAD BB9541
     // FILE NAME-PRCTUM,LABEL-?9?PRCTUM,DISP-SHR
     // FILE NAME-BB954S,LABEL-?9?BB954S,RECORDS-999000,RETAIN-T
     // RUN
     ```

4. **Process Additional Files (Program BB954P)**:
   - Loads and executes the program `BB954P` (likely the main program, as it matches the OCL file name).
   - Defines three files:
     - `BICONT` (input file, shared access).
     - `ARCUST` (input file, shared access, likely a customer master file).
     - `BB954S` (input/output file, shared access, previously created).
   - This step likely processes customer and control data to prepare for pricing updates.
   - **OCL Syntax**:
     ```ocl
     // LOAD BB954P
     // FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHR
     // FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR
     // FILE NAME-BB954S,LABEL-?9?BB954S,DISP-SHR
     // RUN
     ```

5. **Check for Cancellation Condition**:
   - Evaluates a condition based on `?L'129,6'?` (possibly a status code or variable at position 129, length 6).
   - If the condition equals "CANCEL", the program jumps to the `END` tag, terminating further processing.
   - **OCL Syntax**:
     ```ocl
     // IF ?L'129,6'?/CANCEL      GOTO END
     ```

6. **Update Pricing Files (Program BB9542)**:
   - Loads and executes the program `BB9542`.
   - Defines six files:
     - `DAPRCUP` (input file, shared access, likely contains new pricing data).
     - `BB954S` (input/output file, shared access, customer list from earlier).
     - `BBPRCE` (input/output file, shared access, likely the main pricing file).
     - `BICONT` (input file, shared access, control data).
     - `PRCTUM` (input file, shared access, customer data).
     - `BBPRCEP` (input/output file, shared access, possibly a backup or parallel pricing file).
   - This step updates the pricing files (`BBPRCE`, `BBPRCEP`) using data from `DAPRCUP` and other files.
   - **OCL Syntax**:
     ```ocl
     // LOAD BB9542
     // FILE NAME-DAPRCUP,LABEL-?9?DAPRCUP,DISP-SHR
     // FILE NAME-BB954S,LABEL-?9?BB954S,DISP-SHR
     // FILE NAME-BBPRCE,LABEL-?9?BBPRCE,DISP-SHR
     // FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHR
     // FILE NAME-PRCTUM,LABEL-?9?PRCTUM,DISP-SHR
     // FILE NAME-BBPRCEP,LABEL-?9?BBPRCE,DISP-SHR
     // RUN
     ```

7. **Cleanup and Termination**:
   - The program reaches the `END` tag.
   - Checks again if `?9?BB954S` exists in `DATAF1` and deletes it if found (cleaning up temporary files).
   - Clears all local variables with `LOCAL BLANK-*ALL`.
   - Resets the switch to `00000000` to ensure a clean state for future runs.
   - **OCL Syntax**:
     ```ocl
     // TAG END
     // IF DATAF1-?9?BB954S   DELETE ?9?BB954S,F1
     // LOCAL BLANK-*ALL
     // SWITCH 00000000
     ```

### External Programs Called
The OCL program invokes the following external programs:
1. **BB9541**: Creates a customer list from the `PRCTUM` file and writes it to `BB954S`.
2. **BB954P**: Processes customer and control data, likely preparing data for pricing updates.
3. **BB9542**: Updates the pricing files (`BBPRCE`, `BBPRCEP`) using new pricing data from `DAPRCUP`.

### Tables (Files) Used
The program references the following files (tables):
1. **PRCTUM** (`?9?PRCTUM`): Customer master file, used as input in `BB9541` and `BB9542`.
2. **BB954S** (`?9?BB954S`): Temporary file for storing customer list, created by `BB9541`, used in `BB954P` and `BB9542`, and deleted at the end.
3. **BICONT** (`?9?BICONT`): Control file, used as input in `BB954P` and `BB9542`.
4. **ARCUST** (`?9?ARCUST`): Customer master file, used as input in `BB954P`.
5. **DAPRCUP** (`?9?DAPRCUP`): Pricing update file, used as input in `BB9542`.
6. **BBPRCE** (`?9?BBPRCE`): Main pricing file, used as input/output in `BB9542`.
7. **BBPRCEP** (`?9?BBPRCE`): Parallel or backup pricing file, used as input/output in `BB9542`.

### Summary
The OCL program `BB954P.ocl36.txt` orchestrates a batch process to update rack pricing files. It:
- Conditionally sets processing switches based on input parameters.
- Deletes temporary files to ensure a clean start.
- Calls `BB9541` to generate a customer list.
- Calls `BB954P` to process customer and control data.
- Checks for a cancellation condition.
- Calls `BB9542` to apply pricing updates.
- Cleans up temporary files and resets variables.

The program relies on three external RPG programs (`BB9541`, `BB954P`, `BB9542`) and seven files (`PRCTUM`, `BB954S`, `BICONT`, `ARCUST`, `DAPRCUP`, `BBPRCE`, `BBPRCEP`) to manage the pricing update process.