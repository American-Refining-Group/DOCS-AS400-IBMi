### Process Steps of the AR100 OCL Program

The `AR100.ocl36` script is an OCL (Operation Control Language) procedure designed for the IBM System/36 environment to facilitate individual cash receipts entry, allowing users to enter and process single transactions (e.g., a specific payment or check) outside of batch or payment statement modes. It is conditionally invoked from the main OCL script (`AR050P.ocl36`)—specifically, when the user selection at memory location `100,1` is `2`. This script prepares the environment by invoking procedures, ensuring file availability, loading the `AR100` program, and defining necessary files for execution. Here's a step-by-step breakdown:

1. **Procedure Invocation**:
   - `// SCPROCP ,,,,,,,,?9?`: Calls the `SCPROCP` procedure (a standard System/36 setup routine, likely for session or process initialization, such as allocating resources or setting up the local data area). It passes the parameter `?9?` (a substitution variable, typically representing the workstation or session identifier, e.g., library prefix like `LIB` or user-specific value). This step ensures the environment is ready for transaction processing and references the individual entry transaction file `?9?CRIE?WS?` (where `?WS?` is replaced by `GG` for Atrium compatibility).

2. **Year 2000 Compliance**:
   - `// GSY2K`: Executes the `GSY2K` utility, a standard System/36 routine to handle Year 2000 date compliance. This ensures any date fields in the transaction processing are interpreted correctly (e.g., converting 2-digit years to 4-digit format to avoid millennium bugs).

3. **Switch Initialization**:
   - `// SWITCH 1XXXXXXX`: Sets the switch register (an 8-bit control flag used for conditional logic in System/36 programs). The first bit is set to `1` (enabling a specific mode, such as allowing updates or a particular processing path), while the remaining bits (`XXXXXXX`) are left undefined (retaining previous values). This prepares the environment for the `AR100` program's execution.

4. **File Build Operation**:
   - `// IFF DATAF1-?9?CRIEGG BLDFILE ?9?CRIEGG,A,RECORDS,5000,256,,P,2,5`:
     - Checks if the file `?9?CRIEGG` (individual entry transaction file, with `GG` suffix for Atrium) exists using `DATAF1` (a System/36 file existence check).
     - If it exists, rebuilds the file using `BLDFILE`:
       - `A`: Add mode (allows appending records).
       - `RECORDS,5000`: Allocates space for up to 5,000 records.
       - `256`: Each record is 256 bytes long.
       - `,,P`: Packed decimal format for numerics (the two commas are placeholders for optional parameters like blocking factor and record length type).
       - `2,5`: Primary key starts at position 2 with length 5 (likely a transaction or customer key).
     - This ensures the transaction file is available and properly structured for storing individual cash receipts data.

5. **Program Load and File Definitions**:
   - `// LOAD AR100`: Loads the `AR100` RPG program, which handles the core logic for individual entry (e.g., customer lookup, transaction input, GL validation, and posting).
   - Defines the following files for the program, all in shared mode (`DISP-SHR`) to allow concurrent access:
     - `// FILE NAME-CRTRAN,LABEL-?9?CRIEGG,DISP-SHR,EXTEND-100`: Transaction file alias `CRTRAN`, labeled as `?9?CRIEGG`, extended by 100 records for additional capacity.
     - `// FILE NAME-CRTRANR,LABEL-?9?CRIEGG,DISP-SHR`: Read-only alias `CRTRANR` for the same transaction file.
     - `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Customer master file.
     - `// FILE NAME-ARCUSTX,LABEL-?9?ARCUSX,DISP-SHR`: Alternate index for customer master.
     - `// FILE NAME-GLMAST,LABEL-?9?GLMAST,DISP-SHR`: General Ledger master file.
     - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`: Accounts Receivable control file.
     - `// FILE NAME-ARDETL,LABEL-?9?ARDETL,DISP-SHR`: AR detail file (e.g., for invoice lookups).
   - `// RUN`: Executes the `AR100` program with the defined files, allowing it to process individual cash receipts entries interactively.

6. **Switch Reset**:
   - `// SWITCH 0XXXXXXX`: Resets the first bit of the switch register to `0` (disabling the mode set earlier) after the program runs. The remaining bits remain undefined. This cleans up the environment, ensuring no lingering flags affect subsequent processing.

### Business Rules

The `AR100.ocl36` script enforces the following business rules for individual cash receipts entry in the AR system:

1. **Conditional Invocation**:
   - This script is only called from the main OCL (`AR050P.ocl36`) when the user selects option `2` (individual entry mode). Option `1` would invoke the payment statement mode (`AR050`). This allows users to choose between batch/statement processing and single-transaction entry.

2. **Environment Adaptation for Atrium**:
   - Per the JB01 update (11/28/23), the `?WS?` placeholder in file names (e.g., `?9?CRIE?WS?`) is replaced with `GG` to ensure compatibility with the Atrium environment. This prevents file naming conflicts or access issues in the target system.

3. **File Management and Integrity**:
   - The transaction file (`?9?CRIEGG`) must be rebuilt if it exists, ensuring a clean structure with fixed record size (256 bytes) and key (positions 2-6). This supports up to 5,000 transactions with an additional extension of 100 records during runtime, preventing overflow in high-volume sessions.
   - All files are shared (`DISP-SHR`), enforcing concurrent access rules for multi-user environments—multiple users can read/write without locking, but the system handles contention via System/36's inherent queuing.

4. **Compliance and Setup**:
   - `GSY2K` must run before any processing to ensure date fields (e.g., transaction dates) are Y2K-compliant, avoiding errors in date calculations or validations.
   - Switch settings (`1XXXXXXX` before load, `0XXXXXXX` after) control program behavior: Bit 1 enables update/add capabilities during `AR100` execution and disables them afterward to prevent unintended changes in follow-on processes.

5. **Transaction Isolation**:
   - Individual entry uses a dedicated transaction file (`CRIEGG`), isolating single transactions from batch files (e.g., `CRPSGG` or `CRCKGG` in other modes). This rule ensures data separation, reducing errors in posting or reporting.

6. **Resource Allocation**:
   - `SCPROCP` initializes session-specific resources (e.g., local data area, user switches), ensuring the workstation (`?9?`) has exclusive access to its transaction file during entry.

### Tables (Files) Used

The script interacts with or defines the following files (tables), all prefixed with `?9?` (session/library identifier):
1. **?9?CRIEGG** (aliased as `CRTRAN` and `CRTRANR`):
   - Type: Transaction work file for individual cash receipts.
   - Record Length: 256 bytes, up to 5,000 records (extendable by 100).
   - Key: Positions 2-6 (length 5, likely customer or transaction ID).
   - Usage: Stores entered transactions; supports add/update in shared mode.

2. **?9?ARCUST** (aliased as `ARCUST`):
   - Type: Customer master file.
   - Usage: Lookup customer details during entry.

3. **?9?ARCUSX** (aliased as `ARCUSTX`):
   - Type: Alternate index for customer master.
   - Usage: Efficient searching and retrieval of customer records.

4. **?9?GLMAST** (aliased as `GLMAST`):
   - Type: General Ledger master file.
   - Usage: Validates GL accounts for transaction posting.

5. **?9?ARCONT** (aliased as `ARCONT`):
   - Type: AR control file.
   - Usage: Provides defaults (e.g., GL accounts, company codes) and updates control totals.

6. **?9?ARDETL** (aliased as `ARDETL`):
   - Type: AR detail file.
   - Usage: Retrieves invoice or detail records to apply payments.

### External Programs Called

The script explicitly calls or loads the following:
1. **SCPROCP**:
   - A built-in System/36 procedure for session/process setup (e.g., initializing the local data area or allocating workstation resources). Invoked with parameter `?9?`.

2. **GSY2K**:
   - A standard System/36 utility program for Y2K date compliance.

3. **AR100**:
   - The main RPG program for individual cash receipts entry logic (e.g., screen prompts, validations, posting). Loaded and run with the defined files.

No other external programs are called; the script focuses on environment preparation and execution of `AR100`.### Process Steps of the AR100 OCL Program

The `AR100.ocl36` script is an OCL (Operation Control Language) procedure designed for the IBM System/36 environment, handling individual cash receipts entry as part of an accounts receivable (AR) system. It is conditionally invoked from the main OCL script (`AR050P.ocl36`) when the user selection at memory location `100,1` is `2` (as per the main script's logic: `// IF ?L'100,1'?/2 AR100 ,,,,,,,,?9?`). This script prepares the environment for individual transaction entry, creates necessary work files, loads the `AR100` program, and executes it. The process is tailored for the Atrium environment by replacing workstation identifiers (`?WS?`) with `GG` (per the JB01 comment dated 11/28/23). Here's a step-by-step breakdown:

1. **Procedure Call**:
   - `// SCPROCP ,,,,,,,,?9?`: Invokes a procedure named `SCPROCP`, passing the parameter `?9?` (likely a workstation or session identifier, such as a library prefix). This step performs preliminary setup, such as initializing the session or loading common AR files (e.g., `?9?CRIE?WS?` as the individual entry transaction file). `SCPROCP` is a custom or system procedure for processing setup in the AR module.

2. **Year 2000 Compliance**:
   - `// GSY2K`: Executes a Year 2000 (Y2K) compliance utility. This ensures date fields in the system are handled correctly (e.g., using 4-digit years to avoid millennium bugs), which is standard in legacy System/36 environments for date-sensitive AR transactions.

3. **Switch Initialization**:
   - `// SWITCH 1XXXXXXX`: Sets the first bit of an 8-bit switch register to 1 (enabling a specific processing mode, such as individual entry), while leaving the other bits undefined (`X`). This controls conditional logic within the loaded program or subsequent procedures.

4. **File Build Operation**:
   - `// IFF DATAF1-?9?CRIEGG BLDFILE ?9?CRIEGG,A,RECORDS,5000,256,,P,2,5`: Conditionally checks if the file `?9?CRIEGG` (individual entry transaction work file, suffixed with `GG` for Atrium compatibility) exists using `DATAF1` (a system data flag). If it exists, the `BLDFILE` command rebuilds or recreates it with the following parameters:
     - `A`: Add mode (allows appending records).
     - `RECORDS,5000`: Allocates space for up to 5000 records.
     - `256`: Each record is 256 bytes long.
     - `,,P`: Packed decimal format for numerics (third parameter is likely a placeholder or default).
     - `2,5`: Primary key starts at position 2 with a length of 5 bytes.
   This ensures the work file is ready for temporary storage of individual cash receipt transactions.

5. **Program Load and File Definitions**:
   - `// LOAD AR100`: Loads the `AR100` program, which handles the core logic for individual cash receipts entry (e.g., entering single payments without batch processing).
   - Defines multiple files for the program, all in shared mode (`DISP-SHR`) to allow concurrent access:
     - `// FILE NAME-CRTRAN,LABEL-?9?CRIEGG,DISP-SHR,EXTEND-100`: Maps `CRTRAN` to the work file `?9?CRIEGG`, with an extension of 100 additional records for overflow handling.
     - `// FILE NAME-CRTRANR,LABEL-?9?CRIEGG,DISP-SHR`: Maps `CRTRANR` (likely a read-only alias) to the same work file.
     - `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Customer master file.
     - `// FILE NAME-ARCUSTX,LABEL-?9?ARCUSX,DISP-SHR`: Alternate index for customer master.
     - `// FILE NAME-GLMAST,LABEL-?9?GLMAST,DISP-SHR`: General Ledger master file.
     - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`: AR control file.
     - `// FILE NAME-ARDETL,LABEL-?9?ARDETL,DISP-SHR`: AR detail file (e.g., for invoices).
   - `// RUN`: Executes the `AR100` program with the defined files, allowing it to process individual entries interactively.

6. **Switch Reset**:
   - `// SWITCH 0XXXXXXX`: Resets the first bit of the switch register to 0 after program execution, likely to disable the individual entry mode and restore the system to a default state for subsequent processing.

### Business Rules

The `AR100.ocl36` script enforces the following business rules for individual cash receipts entry:

1. **Conditional Invocation**:
   - This script is only called from the main OCL (`AR050P.ocl36`) if the user selects option `2` (individual entry mode). This distinguishes it from batch or payment statement modes (e.g., option `1` calls `AR050`).

2. **Workstation and Environment Adaptation**:
   - Uses the `?9?` parameter for library/session-specific file naming to support multi-user environments.
   - Replaces `?WS?` with `GG` (per JB01, 11/28/23) to ensure compatibility with the Atrium system, preventing file conflicts in a shared or virtualized setup.

3. **File Management and Integrity**:
   - The work file (`?9?CRIEGG`) is rebuilt only if it already exists (`IFF DATAF1-`), ensuring it is clean and properly structured for the session. This prevents data corruption from prior runs.
   - Allocates fixed resources: 5000 records of 256 bytes each, with a 5-byte primary key starting at position 2, suitable for transaction data (e.g., customer keys, amounts).
   - `EXTEND-100` on `CRTRAN` provides buffer space for unexpected transaction volume, but the fixed allocation promotes efficient resource use in a legacy system.

4. **Shared Access and Concurrency**:
   - All files use `DISP-SHR` (disposition shared), allowing multiple users/sessions to access master files (e.g., `ARCUST`, `GLMAST`) simultaneously without locking, which is essential for a multi-user AR system.

5. **Y2K Compliance**:
   - Mandatory execution of `GSY2K` ensures all date-related fields in cash receipts (e.g., payment dates) are processed correctly, avoiding errors in post-2000 operations.

6. **Switch Control**:
   - The switch is set to `1XXXXXXX` before loading to enable individual entry logic in `AR100` and reset to `0XXXXXXX` afterward, ensuring mode-specific behavior without affecting other processes.

7. **Resource Efficiency**:
   - No unnecessary file creations; focuses on temporary work files for individual entries, minimizing disk usage compared to batch processing.

### Tables (Files) Used

The script interacts with or defines the following files (tables), prefixed with `?9?` for session-specific libraries:
1. **?9?CRIEGG** (aliased as `CRTRAN` and `CRTRANR`):
   - Type: Temporary work file for individual entry transactions.
   - Record Length: 256 bytes, up to 5000 records (extendable by 100).
   - Key: 5-byte primary key starting at position 2.
   - Usage: Stores individual cash receipt data during entry; shared for read/write.

2. **?9?ARCUST** (aliased as `ARCUST`):
   - Type: Customer master file.
   - Usage: Provides customer details for validation during entry; shared access.

3. **?9?ARCUSX** (aliased as `ARCUSTX`):
   - Type: Alternate index file for customer master.
   - Usage: Enables efficient customer lookups; shared access.

4. **?9?GLMAST** (aliased as `GLMAST`):
   - Type: General Ledger master file.
   - Usage: Validates GL accounts for receipts; shared access.

5. **?9?ARCONT** (aliased as `ARCONT`):
   - Type: AR control file.
   - Usage: Stores AR system controls (e.g., defaults); shared access.

6. **?9?ARDETL** (aliased as `ARDETL`):
   - Type: AR detail file.
   - Usage: Accesses invoice/transaction details for applying payments; shared access.

### External Programs Called

The script explicitly calls or loads the following:
1. **SCPROCP**:
   - Invoked as a procedure with parameter `?9?`.
   - Purpose: Preliminary processing setup, likely initializing AR session data or files (e.g., the individual entry transaction file `?9?CRIE?WS?`).

2. **GSY2K**:
   - A utility program/procedure for Y2K date compliance.
   - Purpose: Ensures date handling in the AR system is millennium-safe.

3. **AR100**:
   - Loaded and executed via `// LOAD AR100` and `// RUN`.
   - Purpose: The main RPG program for interactive individual cash receipts entry, using the defined files to process single transactions (e.g., customer selection, amount entry, GL posting).

### Summary

The `AR100.ocl36` script supports individual cash receipts entry by setting up a Y2K-compliant environment, building a dedicated work file, and loading the `AR100` program with shared access to AR and GL master files. It is conditionally triggered for non-batch processing, emphasizing efficiency and concurrency in a multi-user System/36 setup adapted for Atrium. The script's rules focus on data integrity, resource allocation, and mode-specific control via switches, integrating seamlessly with the broader cash receipts workflow from the main OCL.