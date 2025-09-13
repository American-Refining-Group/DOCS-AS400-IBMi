The provided document is an OCL (Operation Control Language) program named `AR200P.ocl36.txt`, likely used on an IBM System/36 or AS/400 system to control the execution of an RPG (Report Program Generator) program for cash receipts posting in an accounts receivable (AR) module. Below, I’ll explain the process steps of the OCL program, identify any external programs called, and note any tables used.

### Process Steps of the OCL Program

OCL is a scripting language used to control job execution and program flow on IBM midrange systems. The steps in `AR200P.ocl36.txt` are executed sequentially unless conditional logic alters the flow. Here’s a breakdown of each line or block:

1. **Comments (`** PROMPT FOR JOURNAL DATE FOR CASH RECEIPTS POSTING`)**:
   - These are descriptive comments indicating the purpose of the program: prompting for a journal date for cash receipts posting in an accounts receivable process.

2. **`// CALL PGM(GSGENIEC)`**:
   - Calls an external program named `GSGENIEC`.
   - This program is likely a general-purpose utility or initialization program, possibly for setting up the environment or performing a preliminary check.
   - The `CALL` statement transfers control to `GSGENIEC`, and execution returns to the next line after the called program completes.

3. **`// IFF ?L'506,3'?/YES RETURN`**:
   - A conditional statement (`IFF` means "if").
   - Checks the value of a location parameter `?L'506,3'?`, which refers to a specific position in a data area or buffer (likely positions 506 to 508, assuming a length of 3 characters).
   - If the condition evaluates to `YES`, the program executes the `RETURN` command, which terminates the OCL program immediately, preventing further execution.
   - This suggests `GSGENIEC` sets a flag or value that determines whether the cash receipts posting should proceed.

4. **`// SCPROCP ,,,,,,,,?9?`**:
   - Executes a procedure named `SCPROCP`.
   - The commas indicate parameters, with `?9?` being the ninth parameter, likely a variable or value passed to the procedure.
   - `SCPROCP` is likely a system or user-defined procedure for additional setup or processing related to the cash receipts module. The exact functionality depends on the procedure’s definition, which isn’t provided here.

5. **`// GSY2K`**:
   - Invokes a procedure or command named `GSY2K`.
   - This could be a Year 2000 compliance utility or a date-related procedure, common in legacy systems to handle date formatting or validation.
   - No parameters are explicitly passed, suggesting it uses default or system-defined values.

6. **`// SWITCH 0XXXXXXX`**:
   - Sets a switch (a set of binary flags, typically 8 bits) to `0XXXXXXX`.
   - The first bit is explicitly set to `0`, and the remaining bits (`XXXXXXX`) are either unchanged or set to a default state (likely `0` or undefined).
   - Switches in OCL are used to control program flow or pass conditions to RPG programs. Here, it’s preparing a switch setting for the subsequent program.

7. **`// LOAD AR200P`**:
   - Loads the RPG program `AR200P` into memory.
   - This is the main program for cash receipts posting, likely responsible for prompting the user for a journal date and processing cash receipt transactions.

8. **`// RUN`**:
   - Executes the loaded program (`AR200P`).
   - The program runs with the environment and parameters set up by the preceding steps.

9. **`// IF SWITCH1-1 CANCEL`**:
   - Checks the first bit of the switch (set earlier by `SWITCH 0XXXXXXX` or modified by `AR200P`).
   - If the first bit is `1` (i.e., `SWITCH1-1` is true), the program executes the `CANCEL` command, terminating the job or program abnormally.
   - This suggests `AR200P` may set the first switch bit to `1` to indicate an error or condition that requires stopping the process.

10. **`// AR200 ,,,,,,,,?9?`**:
    - Calls another program or procedure named `AR200`, passing `?9?` as the ninth parameter.
    - This could be a follow-up program or a related module in the accounts receivable system, possibly for finalizing the cash receipts posting or updating related files.
    - The relationship between `AR200P` and `AR200` is unclear without more context, but they may be part of the same AR module.

11. **`// LOCAL BLANK-*ALL`**:
    - Clears all local variables or data areas used in the OCL program.
    - This ensures that no residual data remains in memory, preventing issues in subsequent runs or programs.
    - `*ALL` indicates that all local storage is reset to blanks or zeros.

### Summary of Process Flow
- The program starts by calling `GSGENIEC` to perform initial setup or validation.
- If a specific condition (`?L'506,3'? = YES`) is met, the program exits early.
- Otherwise, it executes the `SCPROCP` procedure with a parameter, followed by `GSY2K` for date-related processing.
- It sets a switch, loads and runs the main RPG program `AR200P` for cash receipts posting.
- If `AR200P` sets the first switch bit to `1`, the program cancels.
- If not, it proceeds to call `AR200` with a parameter and clears local variables to clean up.

### External Programs Called
The OCL program explicitly calls or references the following external programs or procedures:
1. **GSGENIEC**: Likely a utility or initialization program.
2. **SCPROCP**: A procedure, possibly for setup or configuration.
3. **GSY2K**: A procedure, likely for Year 2000 compliance or date handling.
4. **AR200P**: The main RPG program for cash receipts posting.
5. **AR200**: Another program or procedure, possibly for finalizing or updating AR data.

### Tables Used
- The OCL program itself does not explicitly reference any tables (files or databases).
- However, the RPG program `AR200P` (and possibly `AR200`) likely interacts with database files (tables) for accounts receivable, such as customer master files, cash receipts journals, or general ledger files. These are not specified in the OCL code.
- The `?L'506,3'?` parameter might reference a data area or a field in a file, but no specific table is named.

### Notes
- The exact behavior of `GSGENIEC`, `SCPROCP`, `GSY2K`, `AR200P`, and `AR200` depends on their internal logic, which isn’t provided.
- The `?9?` parameter and `?L'506,3'?` suggest dynamic data passing, possibly user inputs or system variables.
- The program assumes a System/36 or AS/400 environment, where OCL controls job execution and RPG handles business logic.
- If you have access to the RPG source code for `AR200P` or definitions for the procedures, I can provide a more detailed analysis of their interactions with tables or files.

If you need further clarification or have additional context (e.g., the RPG code or procedure definitions), please provide it, and I can refine the explanation!