This is an old **AS/400 (IBM i) OCL (Operator Control Language)** procedure that serves as a **wrapper/script** to run the **A/R Aged Trial Balance report** program **AR500P**.  
It was typically used in System/36-compatible environments or early AS/400 SSP mode.

Here is a step-by-step breakdown of what this OCL does:

| Line(s) | OCL Statement | Meaning / Purpose |
|---------|---------------|-------------------|
| `** A/R AGED TRIAL BALANCE **` | Comment lines | Just titles/documentation |
| `// CALL PGM(GSGENIEC)` | Calls program **GSGENIEC** | Almost certainly a **global security or environment initialization** routine (common in many packages). It sets up library list, overrides, user authority checks, etc. |
| `// IFF ?L'506,3'?/YES   RETURN` | Conditional return | Checks positions 506-508 of the **?parameter string?**. If they contain “YES”, the procedure returns immediately (probably a “bypass” switch). |
| `// SCPROCP ,,,,,,,,?9?` | Set procedure parameters | Copies the 9th incoming parameter into local variable ?9? (usually the **company code** or **A/R control file suffix**). |
| `// LOCAL BLANK-*ALL` | Clears all local variables | Good housekeeping |
| `// GSY2K` | Calls procedure **GSY2K** | Very common “Year 2000 company/library setup” routine. It sets the library list, data library, date format, etc., based on company (?9?). |
| `// LOAD AR500P` | Loads (brings into memory) program **AR500P** | The actual A/R Aged Trial Balance report program |
| `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHRMM` | File override | Overrides the file **ARCONT** used by AR500P to open **?9?ARCONT** (e.g., if ?9?=01 → opens 01ARCONT). This is the **A/R Control file** for the specific company. |
| `// RUN` | Executes the loaded program **AR500P** | Actually runs the Aged Trial Balance report |
| `// IF ?L'124,6'?/CANCEL      GOTO END` | Checks positions 124-129 of the parameter string | If those 6 characters = “CANCEL”, skip job queue submission and go straight to end (used for interactive runs that should not be spooled). |
| `// IF ?L'120,1'?/Y           JOBQ ?CLIB?,AR500,,,,,,,,,?9?,,?11?` | Conditional JOBQ submission | If position 120 of the parameter string is “Y”, submit the job **AR500** (a batch version) to the job queue in library ?CLIB? (usually the control library), passing parameters including company ?9? and another parameter ?11?. |
| `// ELSE                      AR500 ,,,,,,,,?9?,,?11?` | Else call AR500 directly | If not “Y”, just call the batch program **AR500** synchronously (rarely used). |
| `// TAG END` | Label for GOTO | Target of the earlier GOTO |
| `// LOCAL BLANK-*ALL` | Clean up | Clears local variables before exit |

### Summary of the process flow

1. Run global initialization/security program (GSGENIEC).
2. Allow early exit if a certain switch is set.
3. Pick up the company code (usually in parameter 9).
4. Run the standard company/library setup procedure (GSY2K).
5. Load and run the interactive Aged Trial Balance program **AR500P**, pointing it to the correct company’s ARCONT file (?9?ARCONT).
6. After the interactive report finishes, decide whether to submit the batch version:
   - If a flag says “Y” → submit **AR500** to the job queue (normal nightly/month-end run).
   - Otherwise just call **AR500** directly or do nothing.

### External programs/procedures called

| Program/Procedure | Purpose |
|-------------------|---------------------------------------------|
| **GSGENIEC**      | Global security / environment initialization |
| **GSY2K**         | Company/library list setup (Y2K-era standard) |
| **AR500P**        | Interactive A/R Aged Trial Balance report (loaded & run) |
| **AR500**         | Batch version of the Aged Trial Balance (submitted or called) |

### Physical files (tables) referenced

| File name | How it is used | Real file opened |
|-----------|----------------|------------------|
| **ARCONT** | Declared in AR500P | Dynamically overridden to **?9?ARCONT** (e.g., 01ARCONT, 02ARCONT, etc.) – this is the A/R control/parameters file for the company |

Other files (ARHIST, ARCUST, AROPEN, etc.) are almost certainly opened directly inside **AR500P/AR500** using the library list set by GSY2K, so they are not explicitly overridden here.

This OCL is a classic pattern from third-party ERP packages (MAPICS, JBA, JDE World early versions, PRMS, etc.) running on System/36 or AS/400 in the 1990s–early 2000s.