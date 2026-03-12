### Process Steps of the RPG .OCL Program

This OCL (Operation Control Language) program appears to be a batch processing script for handling proforma invoices on an IBM midrange system (e.g., AS/400 or iSeries). It includes conditional logic, file clearing, program calls, and job submissions. The program uses variables like ?9? (likely a library prefix), switches for controlling flow, and offsets for data placement (possibly in display files or memory). Below is a step-by-step breakdown of the process:

1. **Initial Prompt and Validation Loop**:
   - Displays a "PROFORMA INVOICE PROMPT" and pauses with a message 'CHECKING AGAIN'.
   - Checks for active conditions (ACTIVE-BB114P or ACTIVE-BB114):
     - If active, loads and runs the external program BB110E.
     - Evaluates a condition (?L'300,6'?/CANCEL) to decide whether to goto END (terminate) or AGAIN (loop back for re-checking).
   - This acts as a validation or setup loop to ensure prerequisites are met before proceeding.

2. **Switch and Local Variable Setup**:
   - Resets switches to 00000000 and blanks all local variables.
   - Calls GSY2K (possibly a system routine or subroutine for year 2000 compliance or date handling).
   - Checks a condition (?4?/Y): If true, sets a specific switch (XXXXXX1X) for "PROFORMA INVOICES ONLY" mode and sets a local offset (232) with data 'Y'.

3. **Order Batch Selection**:
   - Sets local offsets with data values:
     - Offset 470: '?13?' (possibly a batch or order identifier).
     - Offset 494: '?USER?' (user-related data).
     - Offset 502: '?WS?' (workstation or session data).
     - Offset 504: 'L' (likely a literal or flag).
   - If switch 7 is set (SWITCH7-1), sets offset 60 with '*     PROFORMA INVOICES      *' (display header).
   - Resets switches to 0XXXXXXX for batch deletion mode.
   - Evaluates P20 as '?L'490,2'?' (likely retrieving or calculating a parameter value from a location).

4. **Proforma Invoice Selection and File Preparation**:
   - Clears (CLRPFM) several physical files to prepare for new data:
     - ?9?BBPROB
     - ?9?BBPROD
     - ?9?BBPROH
     - ?9?BBPROI
     - ?9?BBPROM
     - ?9?BBPROO
   - Calls the external program BB114 with parameter '?9?' (likely passing the library prefix).

5. **Job Submission or Execution**:
   - Jumps to a tag (JUMP) based on prior logic.
   - Checks conditions:
     - If ?L'166,1'?/Y and SWITCH7-1 is set, submits BB114O to a job queue (?CLIB?,BB114O) with parameters ,,,,,,,,?9?.
     - Otherwise, if SWITCH7-1 is set, runs BB114O directly with the same parameters.
   - Proceeds to the END tag.

6. **Termination**:
   - Resets switches to 00000000 and blanks all local variables.
   - Ends the program.

The overall flow is conditional and looped for error handling, focusing on preparing data files, calling programs for processing proforma invoices, and optionally submitting jobs for background execution.

### External Programs Called

- BB110E: Loaded and run conditionally during the initial validation loop.
- BB114: Called directly with parameter '?9?' during the invoice selection phase.
- BB114O: Submitted to a job queue or run directly, depending on conditions, with parameters ,,,,,,,,?9?.

### Tables/Files Used

These are physical files (PFs) explicitly cleared via CLRPFM, prefixed with ?9? (likely a variable library name). They are used for storing proforma invoice-related data:

- BBPROB
- BBPROD
- BBPROH
- BBPROI
- BBPROM
- BBPROO