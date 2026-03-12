##AP290

### Process Steps of the AP290 OCL Program

The OCL program appears to be part of an Accounts Payable check reconciliation post process. Here’s a step-by-step breakdown of the operations based on the provided code:

1. **Program Header and Identification**:
   - The lines `** A/P CHECK RECONCILIATION POST` and `// SCPROCP ,,,,,,,,?9?` indicate the program’s purpose and possibly a procedure identifier (`SCPROCP`). The `?9?` is a placeholder for a specific value (e.g., library or system identifier) that is resolved at runtime.
   - `// GSY2K` suggests a reference to a system or configuration, possibly related to Y2K compliance or a specific system module.

2. **User Interaction (Pause for Confirmation)**:
   - The line `// * 'A/P CHECK RECONCILIATION POST'` displays a message to the user, indicating the program’s purpose.
   - `// PAUSE 'TO CANCEL--PRESS ATTN,2,ENTER  TO CONTINUE--PRESS 0,ENTER'` prompts the user to either cancel the process (by pressing ATTN, 2, ENTER) or continue (by pressing 0, ENTER). This is a user interaction step to confirm whether the reconciliation post should proceed.

3. **Attribute Inquiry**:
   - `// ATTR INQUIRY-YES,CANCEL-NO` likely sets up the program to handle inquiry mode (YES) or cancellation (NO) based on user input. This could control whether the program runs in an interactive inquiry mode or proceeds with processing.

4. **Loading the Program**:
   - `// LOAD AP290` loads the program `AP290`, which is presumably the main RPG program responsible for performing the A/P check reconciliation post. This is the core executable for the process.

5. **File Specifications**:
   - The program specifies three files to be used:
     - `// FILE NAME-APCRTR,LABEL-?9?APCR?WS?`: Defines a file named `APCRTR` (likely a transaction or work file) with a label that includes placeholders (`?9?APCR?WS?`). The placeholders suggest dynamic naming based on system or job parameters.
     - `// FILE NAME-APCONT,LABEL-?9?APCONT,DISP-SHR`: Defines a file named `APCONT` (possibly a control file for A/P) with a shared disposition (`DISP-SHR`), allowing multiple processes to access it concurrently.
     - `// FILE NAME-APCHKR,LABEL-?9?APCHKR,DISP-SHR`: Defines a file named `APCHKR` (likely the check reconciliation file) with shared disposition.
   - These files are opened for use by the `AP290` program.

6. **Running the Program**:
   - `// RUN` executes the loaded `AP290` program, which processes the reconciliation using the specified files.

7. **File Cleanup**:
   - `// GSDELETE APCR?WS?,,,,,,,,?9?` deletes the work file `APCR?WS?` (likely `APCRTR` with resolved placeholders) to clean up temporary data after processing.
   - `CLRPFM FILE(?9?APCHKUP)` clears the physical file `APCHKUP` (a file not mentioned earlier in the file specifications but likely used as an output or temporary file). The `CLRPFM` (Clear Physical File Member) command removes all records from the file, preparing it for new data or ensuring it is empty post-processing.

### External Programs Called

- **AP290**: This is the main RPG program loaded and executed by the OCL script (`// LOAD AP290` and `// RUN`). It performs the core logic for the A/P check reconciliation post.
- No other external programs are explicitly called in the provided OCL snippet.

### Tables (Files) Used

The OCL program references the following files (tables in IBM i terminology):

1. **APCRTR** (`// FILE NAME-APCRTR,LABEL-?9?APCR?WS?`):
   - Likely a transaction or work file used for temporary storage of reconciliation data.
   - The label includes placeholders (`?9?APCR?WS?`), suggesting dynamic naming.

2. **APCONT** (`// FILE NAME-APCONT,LABEL-?9?APCONT,DISP-SHR`):
   - Probably a control file containing configuration or summary data for the A/P process.
   - Opened with shared disposition (`DISP-SHR`), allowing concurrent access.

3. **APCHKR** (`// FILE NAME-APCHKR,LABEL-?9?APCHKR,DISP-SHR`):
   - Likely the main check reconciliation file, storing check data or reconciliation results.
   - Also opened with shared disposition.

4. **APCHKUP** (`CLRPFM FILE(?9?APCHKUP)`):
   - A file cleared at the end of the process, possibly used as an output or update file for storing reconciled check data.
   - Not listed in the initial file specifications, suggesting it may be implicitly used by `AP290` or defined elsewhere.

### Summary

- **Process Overview**: The OCL program initiates an A/P check reconciliation process by prompting the user for confirmation, loading the `AP290` RPG program, opening necessary files (`APCRTR`, `APCONT`, `APCHKR`), running the program, and performing cleanup by deleting a work file and clearing an output file (`APCHKUP`).
- **External Programs**: `AP290` (main RPG program).
- **Files Used**: `APCRTR`, `APCONT`, `APCHKR`, `APCHKUP`.

If you have additional context (e.g., the RPG code for `AP290` or more OCL snippets), I can provide a more detailed analysis of the program’s logic or interactions. Let me know if you’d like me to search for related information or clarify any specific part!