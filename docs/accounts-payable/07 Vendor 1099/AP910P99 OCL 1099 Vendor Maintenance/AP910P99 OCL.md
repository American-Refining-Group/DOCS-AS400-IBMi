The provided document is an Operation Control Language (OCL) program, specifically `AP910P99.ocl36.txt`, used for vendor file maintenance related to 1099 forms in an IBM AS/400 or IBM i environment. Below, I’ll break down the process steps of the OCL program, identify the external programs called, and list the tables (files) used.

---

### Process Steps of the RPG OCL Program

OCL is a control language used on IBM midrange systems to manage job execution, call programs, and handle file operations. The program `AP910P99` appears to be part of an accounts payable (AP) system for maintaining vendor data related to 1099 forms. Here’s a step-by-step explanation of the process:

1. **Initialization and Setup**:
   - `// SCPROCP ,,,,,,,,?9?`: This likely specifies the procedure name and a parameter `?9?`, which is a placeholder for a runtime value (possibly a control flag or environment indicator).
   - `// SWITCH 00000000`: Initializes job switches to `00000000`, resetting any control flags used for conditional logic.
   - `// LOCAL BLANK-*ALL`: Clears all local variables to blank, ensuring a clean state for the job.
   - `// GSY2K`: Likely invokes a system-level setting or module related to Year 2000 (Y2K) compliance, possibly for date handling.

2. **Prompt for Four-Digit Year**:
   - The commented message `// * 'WHAT YEAR ARE THESE 1099''S FOR?  PLEASE ENTER THE "FOUR DIGIT YEAR"...'` suggests that the program expects a four-digit year input (e.g., `2025`) to process 1099 forms for a specific year.
   - `// IF ?10R?/ * ''`: Checks if the input parameter `?10?` (likely the year) is blank. If blank, the program may skip certain steps or terminate (the exact behavior depends on the condition handling).

3. **Conditional Program Calls Based on Parameter `?9?`**:
   - The program checks the value of `?9?` (a control flag or environment indicator, possibly `G` for a specific mode like "Generate").
   - If `?9?` equals `G`:
     - `CALL PGM(AP910P) PARM('MNT' 'APVN?10?' '?9?')`: Calls the RPG program `AP910P` with parameters:
       - `'MNT'`: Indicates maintenance mode.
       - `'APVN?10?'`: The file name `APVN` concatenated with the four-digit year (e.g., `APVN2025` for year 2025).
       - `'?9?'`: Passes the control flag.
     - Alternatively, `CALL PGM(AP910P) PARM('MNT' '?9?VN?10?' '?9?')`: Calls the same program but with a different file name format, `?9?VN?10?` (e.g., `GVN2025` if `?9?` is `G`).
   - These calls likely perform maintenance tasks on the vendor file for the specified year, such as updating payment totals or preparing data for 1099 reporting.

4. **Set File Name for Update**:
   - If `?9?` equals `G`:
     - `EVALUATE P13='APVN?10?'`: Sets the variable `P13` to the file name `APVN` concatenated with the year (e.g., `APVN2025`).
     - Alternatively, `EVALUATE P13='?9?VN?10?'`: Sets `P13` to a file name like `GVN2025`.
   - `P13` is used to specify the file for the subsequent update operation.

5. **User Interaction for Payment Updates**:
   - `// * '---------------------------------------------------------------'`: Displays a separator line (commented, possibly for debugging or documentation).
   - `// PAUSE 'THE NEXT SCREEN ALLOWS FOR PAYMENT UPDATES IF NECESSARY'`: Pauses the job and displays a message to the user, indicating that the next screen allows manual updates to payment data.
   - `// UPDDTA ?13?`: Invokes the Update Data (`UPDDTA`) command to allow the user to interactively update records in the file specified by `P13` (e.g., `APVN2025`). This is likely a screen-based interface where users can modify vendor payment totals for the 1099 process.

6. **Conditional Termination**:
   - `// IF SWITCH1-1 GOTO END`: Checks if switch 1 is set to `1`. If true, the program jumps to the `END` tag, terminating execution.
   - This switch might be set by the `AP910P` program or the `UPDDTA` operation to indicate an error or completion condition.

7. **Cleanup and Exit**:
   - `// TAG END`: Marks the end of the program.
   - `// SWITCH 00000000`: Resets job switches to `00000000` for consistency.
   - `// LOCAL BLANK-*ALL`: Clears all local variables again, ensuring no residual data remains.

---

### External Programs Called

The OCL program calls the following external program:
- **`AP910P`**: An RPG program called with parameters to perform maintenance tasks on the vendor file. It is invoked conditionally based on the value of `?9?` (e.g., when `?9?` is `G`). The parameters passed are:
  - `'MNT'`: Maintenance mode.
  - File name (either `APVN?10?` or `?9?VN?10?`, e.g., `APVN2025` or `GVN2025`).
  - Control flag `?9?`.

---

### Tables (Files) Used

The program references the following files (referred to as "tables" in the context of AS/400):
- **`APVNYYYY`**: A vendor file containing data before monthly and yearly totals are cleared, as noted in the programmer’s comment. The `YYYY` is replaced by the four-digit year input (`?10?`), e.g., `APVN2025`.
- **`?9?VN?10?`**: An alternative file name format (e.g., `GVN2025` if `?9?` is `G`). This may refer to a different version or copy of the vendor file, depending on the environment or mode.
- The file name is dynamically set to the variable `P13` and used in the `UPDDTA` command for interactive updates.

Additionally, the programmer’s note mentions:
- **`GAPVEND`**: The original vendor file from which `APVNYYYY` is created during the period-end process (`AP300`). This file is not directly used in `AP910P99` but provides context for the origin of `APVNYYYY`.

---

### Summary

The `AP910P99` OCL program facilitates vendor file maintenance for 1099 form processing. It:
1. Initializes the environment and prompts for a four-digit year.
2. Conditionally calls the RPG program `AP910P` to perform maintenance on the vendor file (`APVNYYYY` or `?9?VN?10?`) based on a control flag (`?9?`).
3. Allows users to manually update payment totals in the specified file using the `UPDDTA` command.
4. Terminates if a specific switch is set or after completing the update process.
5. Ensures cleanup by resetting switches and variables.

**External Programs**: `AP910P`  
**Tables/Files**: `APVNYYYY` (e.g., `APVN2025`), `?9?VN?10?` (e.g., `GVN2025`), with `GAPVEND` as the source file in the period-end process.

If you need further analysis of the `AP910P` program or the file structures, let me know, and I can provide additional insights or search for related information if available!