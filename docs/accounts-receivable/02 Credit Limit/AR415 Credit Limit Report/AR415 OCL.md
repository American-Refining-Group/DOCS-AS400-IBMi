The provided document, `AR415.ocl36.txt`, is an Operation Control Language (OCL) program used on IBM midrange systems like the AS/400 (now IBM i). OCL is a scripting language used to control job execution, manage files, and invoke programs. The program appears to generate a **Customer Credit Report** by performing a series of steps involving file overrides, sorting, and report generation. Below, I’ll explain the process steps, list the external programs called, and identify the tables (files) used.

---

### Process Steps of the RPG .OCL Program

The OCL program orchestrates a sequence of operations to produce the Customer Credit Report. Here’s a step-by-step explanation of the process:

1. **Delete All Overrides (DLTOVR *ALL)**:
   - The `DLTOVR *ALL` command clears all existing file overrides in the job to ensure no unintended file mappings interfere with the program’s execution.

2. **Commented-Out Program Call (GSGENIEC)**:
   - The line `// CALL PGM(GSGENIEC)` is commented out (preceded by `//`), so it is not executed. If uncommented, it would call the program `GSGENIEC`, likely a utility or initialization program.
   - The conditional logic `// IFF ?L'506,3'?/YES RETURN` is also commented out. If active, it would check a condition (likely a data area or variable at location `L'506,3'`) and terminate the job (`RETURN`) if the condition is met.

3. **SCPROCP Command**:
   - The line `// SCPROCP ,,,,,,,,?9?` invokes a procedure or command named `SCPROCP` with placeholder `?9?` for a parameter (likely a library or file prefix). The eight commas indicate empty parameters, suggesting `SCPROCP` may use default values or only require the ninth parameter.

4. **Set Local Variables (LOCAL BLANK-*ALL)**:
   - The `LOCAL BLANK-*ALL` command clears all local variables in the job, ensuring a clean state for subsequent operations.

5. **GSY2K Command**:
   - The `GSY2K` command is executed, likely a utility to handle Year 2000 (Y2K) date conversions or system settings, ensuring date fields are processed correctly.

6. **Override Database File (OVRDBF)**:
   - The command `OVRDBF FILE(BICONT) TOFILE(QS36F/?9?BICONT)` overrides the file `BICONT` to point to `QS36F/?9?BICONT` (a file in the `QS36F` library with a prefix `?9?`). This ensures the program uses the correct version of the `BICONT` file.

7. **Load and Run AR415P**:
   - The `// LOAD AR415P` command loads the program `AR415P`, and the `// RUN` command executes it.
   - The file specification `FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHR` indicates that `AR415P` uses the `BICONT` file (with prefix `?9?`) in shared mode (`DISP-SHR`), allowing multiple jobs to access it concurrently.
   - This program likely performs initial processing or validation on the `BICONT` file, preparing data for the report.

8. **Load and Run #GSORT for Sorting**:
   - The `// LOAD #GSORT` command loads the `#GSORT` utility, a general-purpose sort program.
   - **Input and Output Files**:
     - Input file: `ARCUST` (labeled `?9?ARCUST`, shared mode).
     - Output file: `AR415S` (labeled `?9?AR415S`, with a capacity of 999,000 records, extendable by 999,000, and retained as a job file `RETAIN-J`).
   - **Sort Specifications**:
     - `HSORTR 8A 3X 384 N`: Defines the sort header with an 8-character key, 3 fields, 384 bytes per record, and no sequence checking (`N`).
     - `I C 1 1NECD`: Includes records where position 1 (1 byte) is not equal to a specific condition (`NECD`, possibly "not equal to customer delete").
     - `IAC 2 3EQC?L'101,2'?`: Includes records where positions 2–3 equal a value at location `L'101,2'` (likely a data area or variable).
     - `FNC 2 9 KEY`: Specifies positions 2–9 as the sort key (ascending order by default).
     - `FDC 1 256 RECORD`: Includes positions 1–256 in the output record.
     - `FDC 257 384 RECORD`: Includes positions 257–384 in the output record.
   - **Purpose**: This step sorts the `ARCUST` file based on a key (positions 2–9) and filters records based on conditions, writing the sorted output to `AR415S`.

9. **Load and Run AR415**:
   - The `// LOAD AR415` command loads the main report program `AR415`, and `// RUN` executes it.
   - **Files Used**:
     - `ARCUST` (labeled `?9?AR415S`, the sorted output from `#GSORT`, shared mode).
     - `ARCUSA` (labeled `?9?ARCUST`, shared mode).
     - `ARCLGR` (labeled `?9?ARCLGR`, shared mode).
     - `BBORCL` (labeled `?9?BBORCL`, shared mode).
     - `BICONT` (labeled `?9?BICONT`, shared mode).
   - **Purpose**: The `AR415` program processes the sorted `AR415S` file and other input files (`ARCUSA`, `ARCLGR`, `BBORCL`, `BICONT`) to generate the final Customer Credit Report, likely formatting and printing the data.

10. **End of Program**:
    - The `// END` statement marks the end of the `#GSORT` section, though it appears after the `AR415` section, possibly indicating a structured block for `#GSORT` or a minor formatting issue in the OCL.

---

### External Programs Called

The OCL program invokes the following external programs:
1. **GSGENIEC**: Commented out, so not executed. If active, it would be called as a utility or initialization program.
2. **AR415P**: A program that processes the `BICONT` file, likely for data preparation or validation.
3. **#GSORT**: A sort utility that sorts the `ARCUST` file and produces the `AR415S` file.
4. **AR415**: The main report program that generates the Customer Credit Report using multiple input files.

Additionally, the `SCPROCP` command is executed, which may be a system command or procedure rather than a program, depending on its definition.

---

### Tables (Files) Used

The program references the following files (tables) in the IBM i environment:
1. **BICONT** (`QS36F/?9?BICONT`): Used by `AR415P` and `AR415`, likely a control or configuration file.
2. **ARCUST** (`?9?ARCUST`): The input customer file used by `#GSORT` and `AR415` (as `ARCUSA`).
3. **AR415S** (`?9?AR415S`): The sorted output file created by `#GSORT` and used as input by `AR415`.
4. **ARCUSA** (`?9?ARCUST`): An alias or alternate reference to the `ARCUST` file, used by `AR415`.
5. **ARCLGR** (`?9?ARCLGR`): A ledger or transaction file used by `AR415`, likely containing customer credit data.
6. **BBORCL** (`?9?BBORCL`): Another file used by `AR415`, possibly containing billing or order data related to credit.

The `?9?` placeholder in file labels likely represents a library or prefix dynamically substituted at runtime.

---

### Summary

The OCL program `AR415.ocl36.txt` orchestrates the generation of a Customer Credit Report through the following steps:
- Clears overrides and local variables.
- Executes a Y2K utility and overrides the `BICONT` file.
- Runs `AR415P` to process `BICONT`.
- Uses `#GSORT` to sort the `ARCUST` file into `AR415S` based on specific keys and conditions.
- Runs `AR415` to generate the final report using sorted and additional files.

**External Programs**: `GSGENIEC` (commented out), `AR415P`, `#GSORT`, `AR415`, and possibly `SCPROCP` (if a program).
**Files Used**: `BICONT`, `ARCUST`, `AR415S`, `ARCUSA`, `ARCLGR`, `BBORCL`.