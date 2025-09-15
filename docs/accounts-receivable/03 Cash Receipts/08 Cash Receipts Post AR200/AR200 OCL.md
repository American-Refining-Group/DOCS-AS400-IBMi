### Process Steps of the OCL Program

This OCL (Operation Control Language) program, named `AR200.ocl36.txt`, is a control script for the core cash receipts posting process in an accounts receivable (AR) system on an IBM System/36 or AS/400 environment. It is called from the main OCL program (`AR200P.ocl36.txt`) after the user confirms the journal date via `AR200P`. The script orchestrates data sorting, transaction posting, journal generation, general ledger updates, and cleanup. The `?9?` appears to be a dynamic library or prefix (e.g., a user-specific or date-specific library name), used for temporary file labeling. Here's a sequential breakdown of the steps:

1. **Initialization with Y2K Procedure**:
   - `// GSY2K`: Calls the `GSY2K` procedure (likely for Year 2000 date compliance, setting century values or formatting dates). This prepares the environment for date-sensitive processing.

2. **Delete Existing Temporary Output File (If Exists)**:
   - `// IF DATAF1-?9?CRTRGG DELETE ?9?CRTRGG,F1`: Checks if a temporary file `?9?CRTRGG` (Cash Receipts Transaction Register, Generation Group) exists in the `DATAF1` library. If it does, deletes it to ensure a clean start for the new posting run.

3. **Sort Input Files for Posting**:
   - `// LOAD #GSORT`: Loads the system sort utility `#GSORT`.
   - `// IF DATAF1-?9?CRIEGG FILE NAME-INPUT1,LABEL-?9?CRIEGG`: If `?9?CRIEGG` (Cash Receipts Invoice Entry, Generation Group) exists in `DATAF1`, sets it as the first input file.
   - `// IF DATAF1-?9?CRPSGG FILE NAME-INPUT2,LABEL-?9?CRPSGG`: If `?9?CRPSGG` (Cash Receipts Payment Summary, Generation Group) exists in `DATAF1`, sets it as the second input file.
   - `// FILE NAME-OUTPUT,LABEL-?9?CRTRGG,RECORDS-5000,EXTEND-1000`: Defines the output as `?9?CRTRGG` with initial 5000 records, extendable by 1000.
   - `// RUN`: Executes the sort with the following control statements:
     - `HSORTR 16A 3X 256 SORT FOR A/R POSTING`: Header for the sort, specifying 16-character alphanumeric sort key, 3 times (multiple passes?), 256-byte records.
     - `I C 1 1NECD`: Input control for field 1 (likely a sequence or control field).
     - `IAC 54 54NECE`: Input alphanumeric control for field 54 (error check?).
     - `FNC 7 8 TRCO`: Numeric sort field positions 7-8 (Transaction Code).
     - `FOC 31 31 TRTYPE`: Output control for position 31 (Transaction Type).
     - `FNC 9 21 TRCUST/TRINV#`: Numeric sort field positions 9-21 (Transaction Customer/Invoice Number).
     - `FDC 1 256 RECORD`: Full record dump/output for positions 1-256.
     - `// END`: Ends the sort definition.
   - This step merges and sorts cash receipts data from entry and payment files into a sorted transaction register for posting.

4. **Post Transactions to AR Ledger**:
   - `// LOAD AR200`: Loads the `AR200` RPG program (likely the posting engine).
   - File assignments:
     - `// FILE NAME-ARTRAN,LABEL-?9?CRTRGG`: Input from the sorted transaction register.
     - `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Shared access to customer master file.
     - `// FILE NAME-ARDETL,LABEL-?9?ARDETL,DISP-SHR`: Shared access to AR detail (open items) file.
     - `// FILE NAME-ARHIST,LABEL-?9?ARHIST,DISP-SHR`: Shared access to AR history file.
     - `// FILE NAME-ARDIST,LABEL-?9?CRDIGG,RECORDS-999000,EXTEND-100,RETAIN-J`: Output distribution file for debits/credits, large size (999000 records), extendable by 100, retained until job end (`RETAIN-J`).
     - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`: Shared access to AR control file.
   - Printer setup:
     - `// PRINTER NAME-REPORTP,DEVICE-PJ,FORMSNO-DEL,PRIORITY-0`: Sets printer for reports, device PJ (printer job?), delete forms, high priority.
     - `// IF ?9?/G OVRPRTF FILE(REPORT) OUTQ(QUSRSYS/ARPOST)`: If `?9?` ends with `/G` (production mode?), override print file to output queue `ARPOST`.
     - `// IFF ?9?/G OVRPRTF FILE(REPORT) OUTQ(QUSRSYS/TESTOUTQ)`: Otherwise (test mode), override to `TESTOUTQ`.
   - `// RUN`: Executes `AR200` to post sorted transactions to customer ledgers, updating open items, history, and generating distribution entries. Produces a posting transaction register report.

5. **Generate S/J or C/R Journal**:
   - `// LOAD AR210`: Loads the `AR210` RPG program (journal generator).
   - File assignments:
     - `// FILE NAME-ARDIST,LABEL-?9?CRDIGG`: Input from distribution file created in step 4.
     - `// FILE NAME-ARTEMG,LABEL-?9?CRTGGG,RECORDS-1000,EXTEND-100,RETAIN-J`: Temporary general ledger journal file, retained until job end.
     - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`: Shared AR control file.
   - `// RUN`: Executes `AR210` to summarize distribution entries into a temporary journal (`CRTGGG`) for general ledger posting (S/J = Sales Journal, C/R = Cash Receipts).

6. **Sort Temporary Journal for GL Posting**:
   - `// LOAD #GSORT`: Loads the sort utility again.
   - `// FILE NAME-INPUT,LABEL-?9?CRTGGG`: Input from temporary journal.
   - `// FILE NAME-OUTPUT,LABEL-?9?AR211S,RECORDS-1000,EXTEND-100,RETAIN-J`: Sorted output `AR211S`, retained.
   - `// RUN`: Executes the sort with control statements:
     - `HSORTA 16A 3 SORT FOR TEMGEN POSTING`: Header for alphanumeric sort, 3 passes.
     - `FNC 2 3 GDCO`: Numeric sort field 2-3 (General Distribution Company?).
     - `FNC 4 7 JOURNAL TYPE & NUMBER`: Numeric sort 4-7 (Journal identifier).
     - `FNC 96 96 S-SUMMARIZE`: Control for summarization (S flag at position 96).
     - `FNC 13 20 ACCOUNT #`: Numeric sort 13-20 (GL Account Number).
     - `FNC 12 12 CR/DB CODE`: Numeric sort 12 (Credit/Debit Code).
     - `// END`: Ends sort.
   - This sorts the journal entries by company, journal type, account, and debit/credit for balanced GL posting.

7. **Build Daily AR File (If Needed) and Post to General Ledger**:
   - `// IFF DATAF1-?9?ARDALY BLDFILE ?9?ARDALY,S,RECORDS,1000,96,,T,,,DFILE`: If `?9?ARDALY` (AR Daily file) does not exist in `DATAF1`, builds it as a sequential file (`S`), 1000 records, 96-byte length, type T (temporary?), using DFILE.
   - `// LOAD AR211`: Loads the `AR211` RPG program (GL posting engine).
   - File assignments:
     - `// FILE NAME-ARTEMG,LABEL-?9?CRTGGG`: Input temporary journal.
     - `// FILE NAME-AR211S,LABEL-?9?AR211S`: Sorted journal input.
     - `// FILE NAME-TEMGEN,LABEL-?9?TEMGEN,DISP-SHR`: Shared temporary general ledger master.
     - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`: Shared AR control.
     - `// FILE NAME-GLMAST,LABEL-?9?GLMAST,DISP-SHR`: Shared GL master file.
     - `// FILE NAME-ARDALY,LABEL-?9?ARDALY,DISP-SHR`: Shared AR daily file.
   - Printer setup: Same as step 4 (reports to `ARPOST` or `TESTOUTQ`).
   - `// RUN`: Executes `AR211` to post journal entries to the GL master, update daily AR summaries, and generate reports (e.g., GL journal).

8. **Delete Work Files**:
   - `// GSDELETE CRWXGG,,,,,,,,?9?`: Calls `GSDELETE` procedure to delete work files matching pattern `CRWXGG` in library `?9?` (e.g., temporary cash receipts work files).

9. **Call Additional AR Program**:
   - `// AR2011 ,,,,,,,,?9?`: Calls `AR2011` with `?9?` as the eighth parameter (likely for post-posting tasks, such as aging or reconciliation; exact purpose unclear without source).

10. **Final Cleanup of Temporary Files**:
    - `// IF DATAF1-?9?CRTRGG DELETE ?9?CRTRGG,F1`: Deletes sorted transaction register if exists.
    - `// IF DATAF1-?9?CRPSGG DELETE ?9?CRPSGG,F1`: Deletes payment summary.
    - `// IF DATAF1-?9?CRIEGG DELETE ?9?CRIEGG,F1`: Deletes invoice entry.
    - `// IF DATAF1-?9?CRCKGG DELETE ?9?CRCKGG,F1`: Deletes check file (if used).
    - `// IF DATAF1-?9?CRGLGG DELETE ?9?CRGLGG,F1`: Deletes GL temp file.
    - `// IF DATAF1-?9?CRWKGG DELETE ?9?CRWKGG,F1`: Deletes work file.
    - This ensures no residual temporary files remain after the job.

### Business Rules

The program enforces standard accounting rules for cash receipts posting in an AR/GL integrated system:
1. **Input Preparation and Sorting**:
   - Cash receipts data from invoice entries (`CRIEGG`) and payment summaries (`CRPSGG`) must be sorted by transaction code, type, customer/invoice before posting to ensure sequential processing.
   - Temporary files are deleted at start and end to prevent data contamination from prior runs.

2. **Transaction Posting**:
   - Transactions update customer balances (`ARCUST`), close/open AR details (`ARDETL`), archive to history (`ARHIST`), and generate debit/credit distributions (`CRDIGG`).
   - Posting requires shared access to control files (`ARCONT`) for integrity (e.g., batch totals, sequence checks).
   - Reports are generated for audit trails; output queues differ by mode (`/G` for production vs. test).

3. **Journal Generation and GL Integration**:
   - Distributions are summarized into journals (`CRTGGG`) for GL posting, ensuring debits equal credits.
   - Journals are sorted by company, type, account, and CR/DB code for balanced entry.
   - GL updates (`GLMAST`) and daily summaries (`ARDALY`) are mandatory for reconciliation.
   - Only summarized entries (flagged 'S') are posted to GL to avoid detail-level bloat.

4. **Error Handling and Cleanup**:
   - Files are checked for existence before use or deletion to avoid errors.
   - Retained files (`RETAIN-J`) ensure data availability during the job but are cleaned up post-run.
   - Y2K compliance is enforced at start for date accuracy.
   - Test/production mode (`?9?/G`) routes reports separately to prevent live data impact.

5. **Customization Note**:
   - Comment `JB01 112823 JAN BECCARI - CHANGE ?WS? IN AR TO 'GG' SO PROGRAMS CAN BE RUN IN ATRIUM`: Indicates a modification to use 'GG' suffix for files, allowing execution in a specific environment (Atrium, possibly a subsystem or test bed).

6. **Overall Flow**:
   - The process assumes pre-loaded input files from upstream (e.g., data entry). No user interaction; it's batch-oriented.
   - Completion relies on successful RPG runs; failures (e.g., sort errors) would halt via OCL error handling (not explicit here).

### Tables Used

In this context, "tables" refer to database files (physical or logical) accessed or created. The OCL references labels (file names with `?9?` prefix/library), which are assigned to files. Key files include:

1. **Input/Temporary Files** (Created/Used Temporarily):
   - `?9?CRIEGG`: Cash Receipts Invoice Entry (input for sorting).
   - `?9?CRPSGG`: Cash Receipts Payment Summary (input for sorting).
   - `?9?CRTRGG`: Cash Receipts Transaction Register (sorted output from step 3; input to posting).
   - `?9?CRDIGG`: Cash Receipts Distribution (output from posting; input to journal).
   - `?9?CRTGGG`: Cash Receipts Temporary GL Journal (output from AR210; input to GL posting).
   - `?9?AR211S`: Sorted Journal for GL (output from step 6).
   - `?9?ARDALY`: AR Daily Summary (built if needed; shared for GL).
   - `?9?CRCKGG`, `?9?CRGLGG`, `?9?CRWKGG`: Additional temp files (checks, GL work, general work; cleaned up).

2. **Master/Shared Files** (Accessed in SHR Mode for Updates/Reads):
   - `?9?ARCUST`: AR Customer Master (balances, details).
   - `?9?ARDETL`: AR Detail/Open Items (invoices, payments).
   - `?9?ARHIST`: AR History (archived transactions).
   - `?9?ARCONT`: AR Control (batch controls, totals).
   - `?9?TEMGEN`: Temporary General Ledger (intermediate GL updates).
   - `?9?GLMAST`: General Ledger Master (accounts, postings).
   - `DATAF1-*`: Library for checking/deleting temps (e.g., `DATAF1-?9?CRTRGG`).

3. **Reports/Output**:
   - Printer files: `REPORTP` (posting register, GL journal) to queues `ARPOST` (production) or `TESTOUTQ` (test).

No direct database queries; files are passed to RPG programs for processing.

### External Programs Called

The OCL calls the following external programs/procedures (loaded and run):
1. **GSY2K**: Procedure for Y2K/date initialization (step 1).
2. **#GSORT**: System sort utility (steps 3 and 6; twice).
3. **AR200**: RPG program for transaction posting to AR (step 4).
4. **AR210**: RPG program for journal generation (step 5).
5. **AR211**: RPG program for GL posting (step 7).
6. **GSDELETE**: Procedure for deleting work files (step 8; pattern `CRWXGG`).
7. **AR2011**: RPG program for additional AR processing (step 9; parameters include `?9?`).

These are invoked sequentially, with file assignments passed as parameters. The main OCL (`AR200P`) calls this as `AR200 ,,,,,,,,?9?`, passing `?9?` for library context.### Process Steps of the OCL Program

The provided document is an OCL (Operation Control Language) procedure named `AR200.ocl36.txt`, which handles the core processing for cash receipts posting in an accounts receivable (AR) system on an IBM System/36 or AS/400 System/36 Environment. It is called from the main OCL program (`AR200P.ocl36.txt`) with the parameter `?9?`, which appears to be a library or prefix identifier (e.g., 'GG' for a specific environment like "Atrium," as noted in the JB01 comment). The procedure orchestrates sorting, posting, journal generation, and cleanup for cash receipt transactions.

OCL procedures are interpreted (not compiled) and use statements starting with `//` to control job flow, load programs, and manage files. The steps are executed sequentially unless conditional (`IFF`/`IF`) logic branches. Here's a detailed breakdown:

1. **Initialization and Year 2000 Setup** (`// GSY2K`):
   - Invokes the `GSY2K` procedure to handle Year 2000 date compliance, likely updating century values or formatting dates in the environment (e.g., setting `y2kcen` and `y2kcmp` used in related RPG programs).

2. **Cleanup of Existing Output File** (`// IF DATAF1-?9?CRTRGG DELETE ?9?CRTRGG,F1`):
   - Checks if the transaction register file (`?9?CRTRGG`) exists in the F1 file device (typically a disk unit).
   - If it exists, deletes it to ensure a clean start for the new run. This prevents appending to old data.

3. **Sort Input Files for Posting** (`// LOAD #GSORT` ... `// END`):
   - Loads the system sort utility `#GSORT` (a special OCL sort program).
   - Conditionally assigns input files:
     - If `?9?CRIEGG` exists, sets it as `INPUT1` (likely entry-level cash receipts).
     - If `?9?CRPSGG` exists, sets it as `INPUT2` (likely payment or summary cash receipts).
   - Creates an output file `OUTPUT` labeled `?9?CRTRGG` (transaction register) with initial allocation of 5000 records, extendable by 1000.
   - Runs the sort with inline specifications (`HSORTR 16A 3X 256 SORT FOR A/R POSTING`):
     - Sorts the entire record (1-256) on multiple character (A) and numeric (N) fields: customer code (positions 1), entity code (54), transaction code (7-8), customer/invoice number (9-21).
     - The `I C 1 1NECD` and similar lines define sort keys for ascending (A) order on customer/entity/transaction fields.
   - Ends the sort block (`// END`), producing a sorted transaction file for posting.

4. **Post Transactions to AR Ledger** (`// LOAD AR200` ... `// RUN`):
   - Loads the RPG program `AR200` (likely the posting engine).
   - Assigns files:
     - Input: `ARTRAN` labeled `?9?CRTRGG` (sorted transactions from step 3).
     - Shared read-only (DISP-SHR): `ARCUST` (customer master), `ARDETL` (AR detail), `ARHIST` (AR history).
     - Output: `ARDIST` labeled `?9?CRDIGG` (distribution file for debits/credits), allocated 999000 records, extendable by 100, retained on job end (RETAIN-J).
     - Shared: `ARCONT` (AR control file).
   - Printer setup: `REPORTP` on device PJ (printer), delete forms after printing (FORMSNO-DEL), priority 0.
   - Conditional override print file (`OVRPRTF`):
     - If parameter `?9?` contains '/G', output to queue `QUSRSYS/ARPOST`.
     - Otherwise, to `QUSRSYS/TESTOUTQ` (test queue).
   - Runs `AR200`, which posts transactions: updates customer balances, generates history, and produces a posting register report.

5. **Generate Summary/Journal Entries** (`// LOAD AR210` ... `// RUN`):
   - Loads `AR210` (likely a journal generator).
   - Files:
     - Input: `ARDIST` labeled `?9?CRDIGG` (from step 4).
     - Output: `ARTEMG` labeled `?9?CRTGGG` (temporary general ledger entries), 1000 records, extendable by 100, retained (RETAIN-J).
     - Shared: `ARCONT`.
   - Runs `AR210` to create summarized journal entries for GL (general ledger) posting.

6. **Sort Temporary GL Entries** (`// LOAD #GSORT` ... `// END`):
   - Loads `#GSORT` again.
   - Input: `INPUT` labeled `?9?CRTGGG` (from step 5).
   - Output: `OUTPUT` labeled `?9?AR211S` (sorted GL entries), 1000 records, extendable by 100, retained (RETAIN-J).
   - Runs sort with specs (`HSORTA 16A 3 SORT FOR TEMGEN POSTING`):
     - Sorts on general distribution company (2-3), journal type/number (4-7), summarize flag (96), account number (13-20), credit/debit code (12).
   - Ends sort, preparing data for GL posting.

7. **Conditional Daily AR File Creation and GL Posting** (`// IFF DATAF1-?9?ARDALY BLDFILE ?9?ARDALY,S,RECORDS,1000,96,,T,,,DFILE` ... `// RUN`):
   - If `?9?ARDALY` (daily AR file) does not exist, creates it as a sequential (S) file with 1000 records, record length 96, temporary (T), using default file (DFILE).
   - Loads `AR211` (GL posting program).
   - Files:
     - Input: `ARTEMG` labeled `?9?CRTGGG` (from step 5).
     - Sorted input: `AR211S` labeled `?9?AR211S` (from step 6).
     - Shared: `TEMGEN` (template general ledger), `ARCONT`, `GLMAST` (GL master), `ARDALY` (daily AR).
   - Printer setup: Same as step 4 (report to ARPOST or TESTOUTQ).
   - Runs `AR211` to post journal entries to the GL, updating accounts and generating reports.

8. **Delete Work Files** (`// GSDELETE CRWXGG,,,,,,,,?9?`):
   - Calls `GSDELETE` procedure to remove temporary work files matching pattern `CRWXGG` in library `?9?` (e.g., deletes files like CRIEGG, CRPSGG, etc., where X varies).

9. **Run AR2011** (`// AR2011 ,,,,,,,,?9?`):
   - Calls program `AR2011` with `?9?` as the 9th parameter (purpose unclear without source, but likely a post-processing or update routine, e.g., for aging or reconciliation).

10. **Final Cleanup** (`// IF DATAF1-?9?CRTRGG DELETE ...` etc.):
    - Conditionally deletes output and work files if they exist: `?9?CRTRGG` (transaction register), `?9?CRPSGG` (payments), `?9?CRIEGG` (entries), `?9?CRCKGG` (checks), `?9?CRGLGG` (GL), `?9?CRWKGG` (work).
    - Ensures no residual files remain after processing.

### Business Rules

This OCL enforces business rules for batch cash receipts posting in an AR system, ensuring data integrity, audit trails, and separation of concerns (e.g., AR updates before GL). Key rules include:

1. **Environment-Specific Processing**:
   - Uses `?9?` (e.g., 'GG') as a prefix for file labels, allowing multi-tenant or test/production separation (e.g., Atrium environment per JB01 comment).
   - Report output queues depend on `?9?` containing '/G' (production to ARPOST; otherwise, test to TESTOUTQ).

2. **File Management and Cleanup**:
   - Always delete prior output files (e.g., `CRTRGG`) to avoid data contamination.
   - Create/allocate new files with specific sizes (e.g., 5000 records for transactions, extendable) and retain them during the job (RETAIN-J) but delete post-run.
   - Conditional creation of daily AR file (`ARDALY`) only if missing, ensuring it's available for GL posting.

3. **Sorting and Data Preparation**:
   - Input files (e.g., `CRIEGG`, `CRPSGG`) must be sorted by customer, entity, transaction code, and invoice before posting to maintain sequential processing.
   - GL entries are summarized and sorted by company, journal type, account, and debit/credit for accurate aggregation.

4. **Posting Sequence**:
   - Transactions post to AR first (updating customers, details, history), generating distributions (`CRDIGG`).
   - Distributions then generate GL journals (`CRTGGG`), which are posted to update accounts receivable control and GL master.
   - Ensures AR integrity before GL impact; failures in one step (e.g., via error handling in called programs) would halt the job.

5. **Reporting and Auditing**:
   - Generates a transaction register report during posting (`AR200`).
   - Produces GL journal reports (`AR211`).
   - Uses priority 0 printing and deletes forms after to streamline batch jobs.

6. **Error Handling and Termination**:
   - Relies on called programs (e.g., `AR200`) for validation; OCL assumes success unless files are missing.
   - Work files are aggressively cleaned up to free disk space.

7. **Y2K Compliance**:
   - Early `GSY2K` call ensures date handling is compliant, tying into date validation from the calling `AR200P` RPG.

### Tables Used

In System/36 OCL, "tables" refer to disk files (physical or sequential). The procedure references or creates the following files (with labels prefixed by `?9?`, e.g., GGCRTRGG). DISP-SHR indicates shared read access; others are input/output:

| File Name | Label/Prefix | Type/Usage | Description |
|-----------|--------------|------------|-------------|
| ARTRAN | ?9?CRTRGG | Input (sorted output) | AR transactions (posting register). |
| ARCUST | ?9?ARCUST | Shared (DISP-SHR) | Customer master file. |
| ARDETL | ?9?ARDETL | Shared (DISP-SHR) | AR detail transactions. |
| ARHIST | ?9?ARHIST | Shared (DISP-SHR) | AR history archive. |
| ARDIST | ?9?CRDIGG | Output | AR distributions (debits/credits). |
| ARCONT | ?9?ARCONT | Shared (DISP-SHR) | AR control/totals file. |
| ARTEMG | ?9?CRTGGG | Output (temporary) | Temporary GL journal entries. |
| AR211S | ?9?AR211S | Input (sorted output) | Sorted GL entries for posting. |
| TEMGEN | ?9?TEMGEN | Shared (DISP-SHR) | GL template/master. |
| GLMAST | ?9?GLMAST | Shared (DISP-SHR) | General ledger master accounts. |
| ARDALY | ?9?ARDALY | Shared (DISP-SHR) | Daily AR summary/activity file (created if missing). |
| REPORTP | N/A | Printer file | Output report spool. |
| Input1/Input | ?9?CRIEGG | Conditional input | Cash receipts entry file. |
| Input2 | ?9?CRPSGG | Conditional input | Cash receipts payment/summary file. |
| Various work files | CRCKGG, CRGLGG, CRWKGG | Deleted post-run | Temporary check, GL, and work files. |

No direct database queries; files are assigned via labels for program access.

### External Programs Called

The OCL calls the following external procedures, utilities, and RPG programs (loaded via `// LOAD` and run via `// RUN` or direct call):

1. **GSY2K**: Procedure for Y2K date setup.
2. **#GSORT**: System sort utility (called twice: once for AR sorting, once for GL sorting).
3. **AR200**: RPG program for transaction posting and register generation.
4. **AR210**: RPG program for journal entry generation from distributions.
5. **AR211**: RPG program for GL posting.
6. **GSDELETE**: Procedure for bulk file deletion (with pattern `CRWXGG` and library `?9?`).
7. **AR2011**: RPG program for post-processing (called with `?9?` parameter).

These integrate with the main OCL flow: the journal date from `AR200P` (RPG) is used here for posting, and success/failure may set switches for the caller.