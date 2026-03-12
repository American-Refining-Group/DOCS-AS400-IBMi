### Explanation of the Process Steps in the RPG .OCL Program (AP765P.ocl36.txt)

This .OCL (Operation Control Language) program, likely written for an IBM System/36 or AS/400 environment, is designed to handle the processing of 1099 vendor forms for a specified year. Below is a step-by-step explanation of the process steps based on the provided code:

1. **Program Initialization**:
   - The program starts with `SCPROCP`, which likely sets up the procedure environment with parameters (up to 9 parameters, as indicated by `,,,,,,,?9?`).
   - `SWITCH 00000000`: Initializes all switches to off (binary 0s).
   - `LOCAL BLANK-*ALL`: Clears all local variables to blanks.
   - `GSY2K`: Likely a call to a system routine to handle Year 2000 compliance (date formatting).

2. **Prompt for Year Input**:
   - The program prompts the user with the message: *"WHAT YEAR ARE THESE 1099'S FOR? PLEASE ENTER THE 'FOUR DIGIT YEAR'..."*.
   - The input is stored in parameter `?10?` (a four-digit year, e.g., 2012).

3. **Construct 1099 File Name**:
   - If parameter `?9?` is 'G', the program sets `P13` (the 1099 file name) to `'APVN?10?'`, where `?10?` is the year (e.g., `APVN2012` for year 2012).
   - Otherwise, if `?9?` is not 'G', it sets `P13` to `'?9?VN?10?'`, incorporating `?9?` as a prefix (e.g., if `?9?` is 'XX', the file name becomes `XXVN2012`).
   - **Programmer Note**: The file `APVNYYYY` (e.g., `APVN2012`) is created during the period-end process (`AP300`) and contains vendor data (`GAPVEND`) before monthly and yearly totals are cleared.

4. **Validate File Existence**:
   - The program checks if the file specified in `?13?` exists using `DATAF1-?13?`.
   - If the file is not found, it displays an error message (`'?L'142,4'? NOT FOUND'`) and pauses for user interaction.
   - If the file still cannot be found, the program cancels execution (`CANCEL`).

5. **Set Company Information**:
   - The program sets local variables with hardcoded company data for the 1099 form:
     - `OFFSET-1`: Company name (`AMERICAN REFINING GROUP INC`).
     - `OFFSET-31`: Address line 1 (`55 ALPHA DRIVE WEST`).
     - `OFFSET-61`: Address line 2 (`PITTSBURGH PA 15238`).
     - `OFFSET-91`: Tax ID (`22-2318612`).
     - `OFFSET-201`: Year (`?10?`, the input year).
   - These values are likely used in the 1099 form output.

6. **Load and Run Main Program**:
   - `LOAD AP765P`: Loads the main program `AP765P` (this could be the current program or a related module).
   - `RUN`: Executes the loaded program.
   - If `SWITCH1-1` is set (indicating an error or specific condition), the program jumps to the `END` tag, terminating execution.

7. **Conditional Program Execution**:
   - The program checks two flags:
     - `?L'120,1'?`: Likely indicates whether to submit the job to a job queue (`Y` for yes, else run directly).
     - `?L'110,1'?`: Determines which program to run (`M` for `AP765`, `N` for `AP765N`).
   - Based on these flags, the program executes one of four scenarios:
     - If `?L'120,1'?` is `Y` and `?L'110,1'?` is `M`: Submits `AP765` to the job queue (`JOBQ ?CLIB?,AP765,,,,,,,,,?9?,?10?,,,?13?`).
     - If `?L'120,1'?` is not `Y` and `?L'110,1'?` is `M`: Runs `AP765` directly (`AP765 ,,,,,,,,?9?,?10?,,,?13?`).
     - If `?L'120,1'?` is `Y` and `?L'110,1'?` is `N`: Submits `AP765N` to the job queue (`JOBQ ?CLIB?,AP765N,,,,,,,,?9?,?10?,,,?13?`).
     - If `?L'120,1'?` is not `Y` and `?L'110,1'?` is `N`: Runs `AP765N` directly (`AP765N ,,,,,,,,?9?,?10?,,,?13?`).
   - Parameters `?9?`, `?10?`, and `?13?` are passed to the called programs.

8. **Program Termination**:
   - `TAG END`: Marks the end of the program.
   - `LOCAL BLANK-*ALL`: Clears all local variables again.
   - `SWITCH 00000000`: Resets all switches to off.

### External Programs Called
The program references the following external programs:
1. **AP765**: Likely the main program for processing 1099 forms when `?L'110,1'?` is `M`.
2. **AP765N**: An alternate program for processing 1099 forms when `?L'110,1'?` is `N` (possibly for a different format or condition).
3. **AP300**: Mentioned in the programmer note as the period-end process that creates the `APVNYYYY` file (not called directly in this program).

### Tables Used
The program references the following table/file:
1. **GAPVEND**: The vendor file, stored as `APVNYYYY` (e.g., `APVN2012`), which contains vendor data before monthly and yearly totals are cleared. This file is accessed via the file name constructed in `?13?`.

### Summary
The program facilitates the generation of 1099 forms by:
- Prompting for the year and constructing the appropriate file name (`APVNYYYY`).
- Validating the existence of the vendor file.
- Setting company information for the forms.
- Conditionally running or queuing one of two programs (`AP765` or `AP765N`) based on control flags.
- The process relies on the `GAPVEND` file and interacts with external programs `AP765` and `AP765N`.