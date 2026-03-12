Let’s break down the process steps of the provided RPG .OCL (Operation Control Language) program, `AP760P.ocl36.txt`, which appears to be a control program for generating a Vendor 1099 Register. I’ll explain each step clearly, identify external programs called, and list the tables/files used, based on the provided document.

---

### Process Steps of the RPG .OCL Program

The program is designed to initiate the generation of a Vendor 1099 Register for a specified year, using a file named `APVNYYYY` (where `YYYY` is the four-digit year). Below is a step-by-step explanation of the program’s logic, derived from the OCL code:

1. **Program Initialization and Parameter Setup**:
   - The program begins with the `// SCPROCP` command, indicating it’s a system control procedure, likely running on an IBM System/3x or AS/400 environment.
   - Parameters `?9?` (likely a library or environment identifier) and `?10?` (the four-digit year for the 1099 forms) are expected as inputs.
   - The `// LOCAL BLANK-*ALL` command clears all local variables to ensure a clean state before processing.
   - The `// GSY2K` command may invoke a utility or subroutine to handle Year 2000 compliance, ensuring date-related calculations are correct.

2. **Prompt for Year (if needed)**:
   - The commented-out line `// * 'WHAT YEAR ARE THESE 1099''S FOR? PLEASE ENTER THE "FOUR DIGIT YEAR"...'` suggests that the program may prompt the user for the year (`?10?`) if it’s not provided. However, since it’s commented out, the year is likely passed as a parameter when the program is invoked.

3. **Determine the 1099 File Name**:
   - The program constructs the file name for the 1099 data using the `EVALUATE` command:
     - If `?9?` (the library/environment identifier) equals `'G'`, the file name is set to `APVN?10?` (e.g., `APVN2025` for year 2025).
     - Otherwise, the file name is set to `?9?VN?10?` (e.g., `LIBVN2025` if `?9?` is `LIB`).
   - The resulting file name is stored in parameter `P13`.

4. **Check for File Existence**:
   - The program checks if the file specified in `?13?` exists using `// IFF DATAF1-?13?`.
   - If the file does not exist, the program displays a pause message: `'?10? NOT FOUND ( ?13? )'`, informing the user that the file for the specified year (e.g., `2025 NOT FOUND ( APVN2025 )`) is missing.
   - If the file is not found, the program executes `// CANCEL`, terminating the process.

5. **Load the Main Program**:
   - If the file exists, the program proceeds with `// LOAD AP760P`, which loads the main RPG program (`AP760P`) responsible for generating the 1099 Register.
   - Two files are opened with shared access (`DISP-SHR`):
     - `APCONT`, labeled as `?9?APCONT` (e.g., `LIBAPCONT` if `?9?` is `LIB`), likely containing accounts payable control information.
     - `GSTABL`, labeled as `?9?GSTABL`, likely a general system table containing configuration or reference data.

6. **Execute the Program**:
   - The `// RUN` command executes the loaded `AP760P` program.
   - The program checks a condition using `// IF ?L'129,6'?/CANCEL`:
     - This likely checks a specific field or flag at position 129, character 6 in a control record or parameter.
     - If the condition evaluates to `CANCEL`, the program jumps to the `END` tag, terminating execution.

7. **Job Submission or Direct Execution**:
   - The program checks another condition using `// IF ?L'120,1'?/Y`:
     - If the condition at position 120, character 1 is `'Y'`, the program submits `AP760` to a job queue (`JOBQ`) with parameters `?CLIB?`, `?9?`, `?10?`, and `?13?`. The `?CLIB?` likely specifies the library for the job queue.
     - If the condition is not `'Y'`, the program directly executes `AP760` with the same parameters (`?9?`, `?10?`, `?13?`).
   - `AP760` is likely the program that processes the `APVNYYYY` file to generate the 1099 Register.

8. **Program Termination**:
   - The `// TAG END` marks the end of the program.
   - The final `// LOCAL BLANK-*ALL` clears all local variables again, ensuring no residual data remains in memory.

---

### External Programs Called

Based on the OCL code, the following external programs are referenced or called:
1. **AP760P**: The main RPG program loaded and executed to generate the 1099 Register. It processes the vendor data from the `APVNYYYY` file.
2. **AP760**: Invoked either directly or via the job queue, likely a compiled version of `AP760P` or a related program that performs the core 1099 processing.
3. **GSY2K** (implied): The `// GSY2K` command suggests a utility or subroutine for Year 2000 date handling, though it’s not explicitly a separate program.

Note: The `AP300` program is mentioned in the programmer’s note as the program that creates the `APVNYYYY` file during period-end processing, but it is not called within this OCL program.

---

### Tables/Files Used

The program references the following files/tables:
1. **APVNYYYY** (referenced as `?13?`, e.g., `APVN2025`):
   - This is the primary input file containing vendor data for the specified year (`YYYY` = `?10?`).
   - Created by the `AP300` program during period-end processing, it holds the `GAPVEND` file data before monthly and yearly totals are cleared.
2. **APCONT** (labeled `?9?APCONT`, e.g., `LIBAPCONT`):
   - A shared file (`DISP-SHR`) likely containing accounts payable control information, such as vendor master data or configuration settings.
3. **GSTABL** (labeled `?9?GSTABL`, e.g., `LIBGSTABL`):
   - A shared file (`DISP-SHR`) likely containing general system tables with reference data, such as tax codes, vendor categories, or other configuration details.

---

### Summary

The `AP760P.ocl36.txt` OCL program orchestrates the generation of a Vendor 1099 Register by:
1. Setting up the environment and determining the 1099 file name (`APVNYYYY`).
2. Verifying the existence of the input file.
3. Loading and running the `AP760P` program, which uses the `APCONT` and `GSTABL` files.
4. Either submitting the `AP760` program to a job queue or executing it directly, based on a control condition.
5. Cleaning up local variables at the start and end of the process.

**External Programs Called**:
- `AP760P` (loaded and run)
- `AP760` (executed directly or via job queue)
- `GSY2K` (implied utility, not explicitly called)

**Tables/Files Used**:
- `APVNYYYY` (e.g., `APVN2025`, primary 1099 data file)
- `APCONT` (e.g., `LIBAPCONT`, accounts payable control file)
- `GSTABL` (e.g., `LIBGSTABL`, general system table)

If you need further clarification or additional analysis (e.g., specific RPG code within `AP760P`), let me know!