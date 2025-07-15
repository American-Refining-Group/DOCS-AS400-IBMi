The provided document is an OCL (Operation Control Language) program, specifically `AP765P.ocl36.txt`, used for processing 1099 forms in an IBM System/36 or AS/400 environment. Below, I’ll explain the process steps of the program, identify the external programs called, and list the tables/files used.

### Process Steps of the RPG .OCL Program

The OCL program orchestrates the generation of 1099 forms by setting up parameters, validating inputs, and invoking associated programs. Here’s a step-by-step breakdown of the process:

1. **Program Initialization and Parameter Setup**:
   - The program starts with the `SCPROCP` command, which likely initiates a procedure with a parameter `?9?` (a placeholder for a library or environment-specific value).
   - The `SWITCH 00000000` command resets all job switches to `0`, ensuring a clean state.
   - The `LOCAL BLANK-*ALL` command clears all local variables to blanks, preventing carryover of stale data.
   - The `GSY2K` command likely adjusts for year 2000 compliance, ensuring proper date handling (common in legacy systems like System/36).

2. **Prompt for Year Input**:
   - The program prompts the user with: `'WHAT YEAR ARE THESE 1099'S FOR? PLEASE ENTER THE "FOUR DIGIT YEAR"...'`.
   - The input is stored in the `?10?` parameter, which represents the four-digit year for the 1099 forms (e.g., `2012`).
   - If `?10?` is blank (`* ''`), the program likely pauses or awaits valid input (though the exact behavior isn’t fully specified here).

3. **Construct 1099 File Name**:
   - The program evaluates the `?13?` parameter (the 1099 file name) based on the value of `?9?`:
     - If `?9?/G` is true (indicating a specific condition, possibly a library or global setting), set `?13?` to `'APVN?10?'` (e.g., `APVN2012` for year 2012).
     - Otherwise, set `?13?` to `'?9?VN?10?'` (e.g., if `?9?` is `LIB`, the file name might be `LIBVN2012`).
   - This constructs the file name for the `GAPVEND` file, which contains vendor data for 1099 processing, created during the period-end process (`AP300`).

4. **File Existence Check**:
   - The program checks if the file specified in `?13?` (e.g., `APVN2012`) exists in the `DATAF1` library or file system.
   - If the file is not found (`IFF DATAF1-?13?`), the program:
     - Displays a pause message: `'?L'142,4'? NOT FOUND'` (indicating the file is missing).
     - Cancels execution (`CANCEL`), halting the process to prevent errors.

5. **Set Company Information**:
   - The program sets local variables with hardcoded data for the 1099 form header:
     - `OFFSET-1`: Company name (`AMERICAN REFINING GROUP INC`).
     - `OFFSET-31`: Address line 1 (`55 ALPHA DRIVE WEST`).
     - `OFFSET-61`: Address line 2 (`PITTSBURGH PA 15238`).
     - `OFFSET-91`: Tax ID (`22-2318612`).
     - `OFFSET-201`: Year (`?10?`, the user-provided year).
   - These values are likely used to populate the 1099 forms with the payer’s information.

6. **Load and Run AP765P**:
   - The `LOAD AP765P` command loads the `AP765P` program (likely an RPG or CL program responsible for 1099 processing).
   - The `RUN` command executes the loaded program.
   - If `SWITCH1-1` is set (indicating an error or specific condition in `AP765P`), the program jumps to the `END` tag, skipping further processing.

7. **Conditional Execution of 1099 Processing Programs**:
   - The program checks two flags stored in local variables:
     - `?L'120,1'?/Y`: Indicates whether the job should be queued (`Y` for yes, run in batch mode).
     - `?L'110,1'?/M` or `/N`: Indicates whether to run the `M` (magnetic media, e.g., for IRS electronic filing) or `N` (non-magnetic media, e.g., printed forms) version of the 1099 processing program.
   - Based on these flags, one of four scenarios is executed:
     - **Batch Mode, Magnetic Media**: If `?L'120,1'? = Y` and `?L'110,1'? = M`, submit `AP765` to the job queue (`JOBQ ?CLIB?,AP765,,,,,,,,,?9?,?10?,,,?13?`).
     - **Interactive Mode, Magnetic Media**: If `?L'120,1'? ≠ Y` and `?L'110,1'? = M`, run `AP765` interactively (`AP765 ,,,,,,,,?9?,?10?,,,?13?`).
     - **Batch Mode, Non-Magnetic Media**: If `?L'120,1'? = Y` and `?L'110,1'? = N`, submit `AP765N` to the job queue (`JOBQ ?CLIB?,AP765N,,,,,,,,?9?,?10?,,,?13?`).
     - **Interactive Mode, Non-Magnetic Media**: If `?L'120,1'? ≠ Y` and `?L'110,1'? = N`, run `AP765N` interactively (`AP765N ,,,,,,,,?9?,?10?,,,?13?`).
   - The programs `AP765` and `AP765N` process the 1099 data, with parameters `?9?` (library/environment), `?10?` (year), and `?13?` (file name) passed to them.

8. **Program Termination**:
   - The `TAG END` marks the end of the program.
   - The program clears local variables (`LOCAL BLANK-*ALL`) and resets switches (`SWITCH 00000000`) to ensure a clean exit.

### External Programs Called

The OCL program invokes the following external programs:
1. **AP765P**: The main program loaded and run to initiate 1099 processing. It likely performs initial setup or validation.
2. **AP765**: The program for processing 1099 forms for magnetic media (e.g., IRS electronic filing).
3. **AP765N**: The program for processing 1099 forms for non-magnetic media (e.g., printed forms).

### Tables/Files Used

The program references the following table/file:
1. **GAPVEND (via ?13?, e.g., APVN2012)**:
   - This is the vendor file containing data for 1099 forms, created during the period-end process (`AP300`).
   - The file name is dynamically constructed as `APVN?10?` (e.g., `APVN2012`) or `?9?VN?10?` based on the `?9?` parameter.
   - The program checks for its existence in the `DATAF1` library or file system.

### Additional Notes
- The programmer’s note indicates that `APVNYYYY` (e.g., `APVN2012`) is created by the `AP300` program during period-end processing, before monthly and yearly totals are cleared.
- The program supports both batch (`JOBQ`) and interactive execution modes, allowing flexibility in how 1099 forms are processed.
- The `?9?`, `?10?`, and `?13?` parameters are critical for dynamic file and program handling, making the program reusable across different years and environments.

This OCL program is a typical example of legacy System/36 automation for financial processes, coordinating user input, file validation, and program execution for 1099 form generation.