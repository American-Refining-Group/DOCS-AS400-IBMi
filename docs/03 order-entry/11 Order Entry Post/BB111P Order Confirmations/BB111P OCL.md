### Process Steps of the RPG .OCL Program

This .OCL (Operation Control Language) script appears to be a control program for handling order confirmations in an IBM i (AS/400) environment. It uses conditional logic, switches, tags for jumping, file operations, and calls to external programs to manage batch processing, user prompts, and job submissions. The script is structured around prompts, checks, selections, and executions, with placeholders like `?9?` (likely a library prefix), `?L'position,length'?` (screen or data location checks), and `?3?` (input checks). Below is a step-by-step breakdown of the process flow:

1. **Initial Prompt and Checking Loop**:
   - Displays an "ORDER CONFIRMATION PROMPT" and executes `SCPROCP` (possibly a system command or procedure call with parameters).
   - Tags "AGAIN" and pauses with a message 'CHECKING AGAIN'.
   - Checks if "ACTIVE-BB111P" is active: If so, loads and runs program `BB110E`, then checks a screen/data location `?L'300,6'?` for "/CANCEL". If cancel is detected, jumps to "END"; otherwise, loops back to "AGAIN".
   - Similarly checks if "ACTIVE-BB111" is active: Loads and runs `BB110E`, checks for "/CANCEL", and jumps or loops accordingly.
   - Resets switches to `00000000`, blanks all local variables, and calls `GSY2K` (possibly a year-2000 compliance routine or global setup).

2. **Switch and Local Variable Setup Based on Input**:
   - If input at position `?3?` is 'Y', sets switch to `XXXXX1XX` (for order confirmations only) and sets a local offset (231) to 'Y'.
   - Proceeds to "ORDER BATCH SELECTION":
     - Sets local data offsets: 470 to '?13?', 494 to '?USER?', 502 to '?WS?', 504 to 'L'.
     - If switch bit 6 is 1, sets offset 60 to '*    ORDER CONFIRMATIONS     *' (likely a title display).

3. **Batch Deletion Switch and Evaluation**:
   - Sets switch to `0XXXXXXX` (resets bit 1).
   - If switch bit 6 is 1, jumps to "JUMP".
   - Tags "JUMP".
   - Evaluates `P20` as data from location `?L'490,2'?` (possibly user input or selection code).
   - If switch bit 6 is 1, jumps to "ORDCONF" (order confirmation selection).

4. **Order Confirmation Selection**:
   - Tags "ORDCONF".
   - Clears physical file members (CLRPFM) for several files in library `?9?`: `BBOCFB`, `BBOCFD`, `BBOCFH`, `BBOCFI`, `BBOCFM`, `BBOCFO` (prepares files for new data by emptying them).
   - Calls program `BB112` with parameter '?9?' (likely populates or processes data into the cleared files).

5. **Job Submission or Execution**:
   - Jumps to "JUMP" (another tag, possibly for looping or continuation).
   - Checks location `?L'166,1'?` for 'Y':
     - If 'Y' and switch bit 6 is 1, submits job `BB111` with `*ALL` to job queue `?CLIB?,BB111,*ALL`.
     - Otherwise, if switch bit 6 is 1, runs `BB111 *ALL` directly.
   - Jumps to "END".

6. **Termination**:
   - Tags "END".
   - Resets switches to `00000000` and blanks all local variables (cleanup).

The overall flow is a prompted, interactive batch process for order confirmations: It checks for active processes, handles cancellations, sets up selections, clears data files, calls a selection program, and finally executes or queues a batch job. Error handling is implicit via cancel checks and jumps.

### External Programs Called

The script calls or loads the following external programs:
- `BB110E`: Loaded and run conditionally during initial checks (for both "ACTIVE-BB111P" and "ACTIVE-BB111" scenarios).
- `BB112`: Called directly in the order confirmation selection step, with parameter '?9?'.
- `BB111`: Run directly or submitted to a job queue in the final execution step, with parameter `*ALL`.

### Tables Used

The script interacts with the following physical files (tables) in the library prefixed by `?9?` (e.g., a dynamic library name). These are cleared (CLRPFM) to prepare for new data:
- `BBOCFB`
- `BBOCFD`
- `BBOCFH`
- `BBOCFI`
- `BBOCFM`
- `BBOCFO`

No other files or tables are explicitly mentioned for read/write operations.