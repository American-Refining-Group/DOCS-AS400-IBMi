The provided document is an OCL (Operation Control Language) script named `AR050P.ocl36.txt`, used to manage cash receipts processing in an IBM System/36 environment. Below, I’ll explain the process steps of the OCL program, identify the external programs called, and list the tables (files) used.

### Process Steps of the AR050P.ocl36 Program

OCL is a control language used on IBM System/36 to manage job execution, file operations, and program flow. The script orchestrates a cash receipts process, likely part of an accounts receivable (AR) system, with conditional logic, file operations, and program calls. Here’s a step-by-step breakdown of the process:

1. **Initial Program Call and Validation Check**:
   - `// CALL PGM(GSGENIEC)`: Invokes the `GSGENIEC` program, likely a utility or initialization program.
   - `// IFF ?L'506,3'?/YES RETURN`: Checks a specific condition at location `506,3` in memory (possibly a status flag). If the condition is true (`YES`), the script terminates (`RETURN`), preventing further execution.

2. **Procedure Call**:
   - `// SCPROCP ,,,,,,,,?9?`: Calls a procedure named `SCPROCP`, passing parameter `?9?` (likely a workstation or session identifier). This might set up the environment or perform preliminary processing.

3. **Local Variable Initialization**:
   - `// LOCAL BLANK-*ALL`: Clears all local variables to blanks, ensuring a clean slate for data manipulation.
   - `// GSY2K`: Executes a Year 2000 compliance routine or utility, possibly to handle date-related processing.
   - `// LOCAL OFFSET-70,DATA-'0000000000000000000000000000000'`: Initializes a 30-character data field at offset 70 with zeros, likely used as a buffer or control field.

4. **Batch Processing Check**:
   - `// IFF DATAF1-?9?CRCKGG IFF DATAF1-?9?CRPSGG IFF DATAF1-?9?CRIEGG GOTO NOBATCH`: Checks if specific files (`CRCKGG`, `CRPSGG`, `CRIEGG`, suffixed with the `?9?` parameter) exist or indicate an active batch process. If none of these files indicate a batch in progress, the script jumps to the `NOBATCH` tag.
   - If a batch is in process:
     - `// PAUSE '** Warning: Batch still in process. Enter 0 to add/update.'`: Displays a warning message to the user, pausing execution to prompt for input (likely requiring the user to enter `0` to proceed with adding or updating records).

5. **NOBATCH Tag - Proceed with Processing**:
   - `// TAG NOBATCH`: Marks the entry point if no batch is in progress.
   - `// TAG PROMPTS`: Defines a `PROMPTS` label, indicating the start of the main processing loop.
   - `// SWITCH 00000000`: Initializes an 8-bit switch register to all zeros, used for conditional control later.

6. **File Build Operation**:
   - `// IFF DATAF1-?9?CRGLGG BLDFILE ?9?CRGLGG,I,RECORDS,10,48,,,2,5`: If the file `CRGLGG` (suffixed with `?9?`) exists, the `BLDFILE` command creates or modifies it with specific parameters:
     - `I`: Input mode.
     - `RECORDS,10,48`: Defines a file with 10 records, each 48 bytes long.
     - `,,,2,5`: Specifies additional file attributes (e.g., 2 for file type, 5 for record format).

7. **File Definitions and Program Load**:
   - `// LOAD AR050P`: Loads the `AR050P` program (the main RPG program associated with this OCL script).
   - `// FILE NAME-GLMAST,LABEL-?9?GLMAST,DISP-SHR`: Defines the `GLMAST` file (likely General Ledger Master) with label `?9?GLMAST` in shared mode (`DISP-SHR`).
   - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`: Defines the `ARCONT` file (likely AR Control file) with label `?9?ARCONT` in shared mode.
   - `// FILE NAME-CRGLDF,LABEL-?9?CRGLGG,DISP-SHR`: Defines the `CRGLDF` file (likely Cash Receipts GL Distribution) with label `?9?CRGLGG` in shared mode.
   - `// RUN`: Executes the loaded `AR050P` program with the defined files.

8. **Conditional Program Calls**:
   - `// IF SWITCH1-1 GOTO EDITS`: If the first bit of the switch register is set to 1, the script jumps to the `EDITS` tag.
   - `// IF ?L'100,1'?/1 AR050 ,,,,,,,,?9?`: If the value at memory location `100,1` is `1`, calls the `AR050` program, passing parameter `?9?`.
   - `// IF ?L'100,1'?/2 AR100 ,,,,,,,,?9?`: If the value at memory location `100,1` is `2`, calls the `AR100` program, passing parameter `?9?`.
   - `// GOTO PROMPTS`: Loops back to the `PROMPTS` tag to repeat the process (likely for interactive prompting).

9. **EDITS Tag - Editing Operations**:
   - `// TAG EDITS`: Marks the start of editing operations.
   - **Individual Edit**:
     - `// LOCAL OFFSET-200,DATA-' '`: Initializes a data field at offset 200 with a single blank character.
     - `// AR110 ,,,,,,,,?9?`: Calls the `AR110` program, passing parameter `?9?`, to perform individual record edits.
   - **Payment Statement Edit**:
     - `// LOCAL OFFSET-200,DATA-'Y'`: Sets the data field at offset 200 to `Y`, likely a flag to enable payment statement processing.
     - `// AR105 ,,,,,,,,?9?`: Calls the `AR105` program, passing parameter `?9?`, to perform payment statement edits.

### External Programs Called

The OCL script invokes the following external programs:
1. **GSGENIEC**: Likely a utility program for initialization or environment setup.
2. **SCPROCP**: A procedure for preliminary processing or setup.
3. **AR050P**: The main RPG program for cash receipts processing.
4. **AR050**: A program called conditionally based on memory location `100,1` value of `1`.
5. **AR100**: A program called conditionally based on memory location `100,1` value of `2`.
6. **AR110**: A program for individual record editing.
7. **AR105**: A program for payment statement editing.

### Tables (Files) Used

The script references the following files (tables):
1. **CRCKGG** (suffixed with `?9?`): Likely a cash receipts check file used to verify batch status.
2. **CRPSGG** (suffixed with `?9?`): Likely a cash receipts posting file used to verify batch status.
3. **CRIEGG** (suffixed with `?9?`): Likely a cash receipts input/edit file used to verify batch status.
4. **CRGLGG** (suffixed with `?9?`): Cash Receipts General Ledger distribution file, used for building and processing.
5. **GLMAST** (suffixed with `?9?`): General Ledger Master file, used in shared mode.
6. **ARCONT** (suffixed with `?9?`): Accounts Receivable Control file, used in shared mode.
7. **CRGLDF** (suffixed with `?9?`): Another reference to the Cash Receipts GL distribution file, aliased to `CRGLGG`.

### Summary

The `AR050P.ocl36` script manages a cash receipts process in an IBM System/36 environment. It:
- Checks for active batch processes and issues warnings if necessary.
- Initializes variables and files, builds a GL distribution file, and loads the main `AR050P` program.
- Conditionally calls programs (`AR050`, `AR100`, `AR110`, `AR105`) based on user input or system state.
- Uses a loop (`PROMPTS`) for interactive processing and supports individual and payment statement edits.
- Interacts with General Ledger and Accounts Receivable files to process cash receipts.

The script is designed to change a workstation identifier (`WS`) to `GG` for running programs in the Atrium environment, as noted in the header comment.