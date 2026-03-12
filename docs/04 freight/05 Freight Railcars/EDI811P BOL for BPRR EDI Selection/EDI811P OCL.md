The provided document is an Operation Control Language (OCL) program, specifically `EDI811P.ocl36.txt`, used in IBM midrange systems like the AS/400 (now IBM i) to control job execution and file operations. OCL is a scripting language used to define job steps, manage files, and control program flow in RPG-based environments. Below, I’ll explain the process steps of the OCL program, identify any external programs called, and list the tables (files) used.

---

### Process Steps of the EDI811P OCL Program

The OCL program `EDI811P` is a control script for an EDI (Electronic Data Interchange) Bill of Lading (BOL) selection process. Here’s a step-by-step breakdown of its operations based on the provided code:

1. **Header and Initial Setup**:
   - The program begins with a comment indicating it is the "EDI BOL SELECTION SCREEN."
   - `// SCPROCP ,,,,,,,,?9?`: This line likely specifies the procedure name (`SCPROCP`) and may include parameters (represented by `?9?`, a placeholder for a value passed at runtime, possibly a library or system name).

2. **Initialization**:
   - `// LOCAL BLANK-*ALL`: Initializes all local variables to blank, clearing any residual data from previous runs to ensure a clean slate.
   - `// GSY2K`: Likely a directive or command to set up the environment, possibly related to Y2K compliance or a specific system configuration. The exact function depends on the system’s configuration but is typically a setup for date handling or system parameters.
   - `// SWITCH 00000000`: Initializes all eight control switches to `0` (off). Switches are used in OCL to control conditional logic later in the program.

3. **Conditional File Deletion**:
   - `// IF DATAF1-EDI DELETE EDI,F1`: Checks if the `DATAF1` field equals `EDI`. If true, it deletes the file labeled `EDI` in the `F1` format. This step ensures that any existing EDI file is removed before processing to avoid conflicts or stale data.

4. **Load Program**:
   - `// LOAD EDI811P`: Loads the `EDI811P` program into memory for execution. This is the main RPG program responsible for the EDI BOL selection logic.
   - The `LOAD` command prepares the program but does not yet execute it.

5. **File Definitions**:
   - The following lines define the files used by the program:
     - `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHRRM`: Defines a file named `BICONT` with a label that includes the `?9?` placeholder (likely a library or prefix). `DISP-SHRRM` indicates the file is opened in shared read mode, allowing multiple processes to read it simultaneously.
     - `// FILE NAME-EDIBOL,LABEL-EDI,RETAIN-T,RECORDS-999000`: Defines a file named `EDIBOL` with the label `EDI`. `RETAIN-T` specifies that the file is temporary and retained for the job’s duration. `RECORDS-999000` allocates space for up to 999,000 records, indicating a large dataset for BOL data.
     - `// FILE NAME-EDIBOLTX,LABEL-?9?EDIBOLX,DISP-SHR`: Defines a file named `EDIBOLTX` with a label that includes the `?9?` placeholder. `DISP-SHR` indicates shared access, typically read-only or shared update.
   - These files are likely used for input (e.g., `BICONT` for contract or master data, `EDIBOL` for BOL data, and `EDIBOLTX` for transaction or extended BOL data).

6. **Program Execution**:
   - `// RUN`: Executes the loaded `EDI811P` program. This is where the main RPG logic for the EDI BOL selection screen is processed, using the defined files for input and output.

7. **Cancellation Check**:
   - `// IF SWITCH8-1 * 'PROCEDURE IS CANCELLED'`: Checks if switch 8 is set to `1`. If true, it indicates the procedure was canceled (likely by user action or an error condition in the RPG program).
   - `// IF SWITCH8-1 GOTO END`: If switch 8 is `1`, the program branches to the `END` tag, skipping further processing.

8. **External Program Call**:
   - `// EDI811 ,,,,,,,,?9?`: Calls another program or procedure named `EDI811`, passing the `?9?` parameter. This could be a follow-up program or a subroutine that processes the output of `EDI811P`, possibly handling EDI transmission or further BOL processing.

9. **End of Processing**:
   - `// TAG END`: Marks the `END` label, where the program jumps if canceled (via switch 8).
   - `// LOCAL BLANK-*ALL`: Clears all local variables again, ensuring no residual data remains.
   - `// SWITCH 00000000`: Resets all switches to `0`, cleaning up the environment before the job ends.

---

### External Programs Called

The OCL program explicitly references the following external programs or procedures:
1. **EDI811P**: The main RPG program loaded and executed via `// LOAD EDI811P` and `// RUN`. This program handles the core logic for the EDI BOL selection screen.
2. **EDI811**: Called via `// EDI811 ,,,,,,,,?9?`. This is likely another RPG program or procedure that processes the output of `EDI811P`, possibly for EDI data formatting or transmission.

No other external programs are explicitly called in the OCL script.

---

### Tables (Files) Used

The OCL program defines the following files (referred to as "tables" in some RPG contexts):
1. **BICONT**:
   - Label: `?9?BICONT` (with `?9?` as a placeholder for a library or prefix).
   - Disposition: `SHRRM` (shared read mode).
   - Purpose: Likely a master file containing contract or inventory data used as input for BOL selection.
2. **EDIBOL**:
   - Label: `EDI`.
   - Attributes: Temporary (`RETAIN-T`), with a capacity of 999,000 records.
   - Purpose: Stores Bill of Lading data generated or processed by the program.
3. **EDIBOLTX**:
   - Label: `?9?EDIBOLX`.
   - Disposition: `SHR` (shared access).
   - Purpose: Likely a transaction or extended data file related to BOLs, possibly for additional details or EDI output.

Additionally, the program references a file labeled `EDI` in the `DELETE` statement (`// IF DATAF1-EDI DELETE EDI,F1`), which may be the same as `EDIBOL` or a related file in the `F1` format.

---

### Summary

- **Process Overview**: The `EDI811P` OCL program initializes the environment, conditionally deletes an existing EDI file, loads and runs the `EDI811P` RPG program, defines three files (`BICONT`, `EDIBOL`, `EDIBOLTX`) for processing, checks for cancellation, and calls a secondary program (`EDI811`) before cleaning up and ending.
- **External Programs**: `EDI811P` (main program), `EDI811` (secondary program/procedure).
- **Tables/Files**: `BICONT`, `EDIBOL`, `EDIBOLTX`, and potentially an `EDI` file (if distinct from `EDIBOL`).

If you need further details, such as analyzing the RPG program logic (if available) or specific file formats, please provide additional context or files.