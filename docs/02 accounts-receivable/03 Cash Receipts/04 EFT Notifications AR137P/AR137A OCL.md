The `AR137A.ocl36.txt` is an Operation Control Language (OCL) script used in IBM i (AS/400) systems, invoked by the `AR137.ocl36.txt` OCL script as part of the EFT (Electronic Funds Transfer) draft notice filtering process. This script is responsible for clearing an EFT summary file and running a program to process EFT transactions and generate reports. Below, I’ll explain the process steps, business rules, tables (files) used, and external programs called, providing a clear and concise analysis.

### Process Steps of the AR137A OCL Program

The `AR137A` OCL script is a concise component of the EFT process, focusing on preparing and processing data for EFT report generation. Here’s a step-by-step breakdown of its execution:

1. **Clear EFT Summary File**:
   - `CLRPFM FILE(?9?AREFTS)`:
     - Clears the physical file member `?9?AREFTS` using the `CLRPFM` (Clear Physical File Member) command.
     - The `?9?` placeholder represents a dynamically resolved library prefix (e.g., a specific library for the environment).
     - This step ensures the `AREFTS` file is empty before new summary data is generated, preventing residual data from affecting the current run.

2. **Load and Run AR137A Program**:
   - `// LOAD AR137A`:
     - Loads the `AR137A` program into memory for execution.
   - File Definitions:
     - `FILE NAME-AREFTD,LABEL-?9?E?L'110,6'?,DISP-SHR`:
       - Defines the `AREFTD` file, labeled as `?9?E?L'110,6'?`, where `?L'110,6'?` is the update date from the Local Data Area (LDA, positions 110-115) prefixed by ‘E’ and `?9?` for the library.
       - Opened in shared mode (`DISP-SHR`) for reading transaction data.
     - `FILE NAME-AREFTD,LABEL-?9?AREFTX,DISP-SHR`:
       - Defines another instance of `AREFTD`, labeled as `?9?AREFTX`, also in shared mode.
       - This may be an alternate or filtered transaction file used for processing.
     - `FILE NAME-AREFTS,LABEL-?9?AREFTS,DISP-SHR`:
       - Defines the `AREFTS` file, labeled as `?9?AREFTS`, in shared mode.
       - Used as an output file for EFT summary data.
     - `FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`:
       - Defines the `ARCONT` file, labeled as `?9?ARCONT`, in shared mode.
       - Contains accounts receivable control data.
     - `FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`:
       - Defines the `ARCUST` file, labeled as `?9?ARCUST`, in shared mode.
       - Contains customer data for EFT processing.
   - `// RUN`:
     - Executes the `AR137A` program, which processes the transaction data from `AREFTD` (or `AREFTX`), uses `ARCONT` and `ARCUST` for reference data, and generates summary data in `AREFTS`.
     - The program likely produces initial EFT reports or data structures for further processing.

### Business Rules

The `AR137A` OCL script enforces the following business rules:

1. **File Cleanup**:
   - The `AREFTS` file must be cleared before processing to ensure no residual data from previous runs affects the current EFT summary output.
   - This ensures data integrity for the EFT report generation process.

2. **File Access**:
   - All files (`AREFTD`, `AREFTX`, `AREFTS`, `ARCONT`, `ARCUST`) are opened in shared mode (`DISP-SHR`), allowing concurrent access by multiple jobs.
   - This supports the batch processing nature of the EFT system, where multiple programs may access these files simultaneously.

3. **Dynamic File Naming**:
   - File names use the `?9?` prefix for the library and `?L'110,6'?` for the update date, ensuring the correct transaction file is processed based on the input from the `AR137P` program.
   - This allows flexibility across different environments or libraries.

4. **EFT Data Processing**:
   - The `AR137A` program processes transaction data from `AREFTD` (or `AREFTX`) and uses `ARCONT` and `ARCUST` to validate or enrich data, producing summary output in `AREFTS`.
   - The output is likely used for EFT report generation or as input for subsequent programs (`AR137E`, `AR137B`).

### Integration with AR137 OCL Script

The `AR137A` OCL script is called by the `AR137.ocl36.txt` script as part of the EFT draft notice filtering process, immediately after the transaction file is filtered (for specific customers) or copied (for all customers) into `?9?AR137S` or `?9?AREFTD`. The `AR137A` program uses the filtered transaction data (`AREFTD` or `AREFTX`), control data (`ARCONT`), and customer data (`ARCUST`) to generate summary data in `AREFTS`, which is later used by `AR137E` and `AR137B` for detailed processing and final report printing. The update date (`?L'110,6'?`) links back to the `KYUPDT` field validated by the `AR137P` RPGLE program, ensuring consistency across the process.

### Tables (Files) Used

The script uses the following files:

1. **AREFTD** (labeled `?9?E?L'110,6'?` and `?9?AREFTX`):
   - Type: Input (shared mode).
   - Purpose: Contains EFT transaction data, either the full set (`?9?E?L'110,6'?`) or a filtered set (`?9?AREFTX`), used as the source for processing.

2. **AREFTS** (labeled `?9?AREFTS`):
   - Type: Input/Output (shared mode).
   - Purpose: Stores EFT summary data, cleared before processing to ensure a fresh dataset.

3. **ARCONT** (labeled `?9?ARCONT`):
   - Type: Input (shared mode).
   - Purpose: Contains accounts receivable control data for validating company information.

4. **ARCUST** (labeled `?9?ARCUST`):
   - Type: Input (shared mode).
   - Purpose: Contains customer data for EFT customer details, such as names and addresses.

### External Programs Called

The script invokes the following external program:

1. **AR137A**:
   - Loaded and executed via `// LOAD AR137A` and `// RUN`.
   - Processes EFT transactions from `AREFTD` (or `AREFTX`), uses `ARCONT` and `ARCUST` for reference, and generates summary data in `AREFTS`.

### Summary

The `AR137A` OCL script is a critical step in the EFT draft notice process, clearing the `AREFTS` summary file and running the `AR137A` program to process transaction data and generate initial EFT summary data or reports. It ensures a clean output file and uses validated transaction, control, and customer data to produce accurate EFT summaries for subsequent steps.

**Tables Used**: `AREFTD` (two instances: `?9?E?L'110,6'?` and `?9?AREFTX`), `AREFTS`, `ARCONT`, `ARCUST`.
**External Programs Called**: `AR137A`.