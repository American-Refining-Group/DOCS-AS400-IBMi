The `AR137.ocl36.txt` is an Operation Control Language (OCL) script used in IBM i (AS/400) systems, invoked by the main `AR137P.ocl36.txt` OCL script for EFT (Electronic Funds Transfer) draft notice filtering, specifically for re-sending notices to selected customers or all customers. The script orchestrates file operations, data filtering, and report generation. Below, I’ll explain the process steps, business rules, tables (files) used, and external programs called, ensuring a clear and concise analysis.

### Process Steps of the AR137 OCL Program

The `AR137` OCL script performs EFT draft notice filtering based on the selection criteria (`ALL` or specific customers) set by the `AR137P` program. It processes transaction data, generates output files, and triggers report generation. Here’s a step-by-step breakdown of the script’s execution:

1. **Date Conversion Utility (`// GSY2K`)**:
   - Invokes the `GSY2K` utility to handle date-related processing, likely ensuring Y2K compliance for date fields used in the EFT process.
   - This step prepares the system environment for date-sensitive operations.

2. **Delete Existing Output File (`// IF DATAF1-?9?AR137S DLTF ?9?AR137S`)**:
   - Checks if the file `?9?AR137S` exists in the `DATAF1` library.
   - If it exists, deletes it using the `DLTF` (Delete File) command to ensure a fresh output file for the current run.

3. **Clear Output Queues**:
   - If the environment parameter `?9?` equals ‘G’ (likely indicating a specific library or environment):
     - Clears the `EFTEMALOTQ` output queue (`// IF ?9?/G CLROUTQ OUTQ(EFTEMALOTQ)`).
     - Clears the `EFTSPLITQ` output queue (`// IF ?9?/G CLROUTQ OUTQ(EFTSPLITQ)`).
   - This ensures no residual print jobs interfere with the current EFT report generation.

4. **Remove Deleted Records and Reorganize File**:
   - Loads and runs the `AR137X` program (`// LOAD AR137X`, `// RUN`):
     - Uses the `CRTRAN` file, labeled as `?9?E?L'110,6'?` (where `?L'110,6'?` is the update date from the LDA, prefixed by ‘E’ and `?9?` for the library).
     - The file is opened in shared mode (`DISP-SHR`).
   - Executes the `RGZPFM` (Reorganize Physical File Member) command on the `?9?E?L'110,6'?` file to remove deleted records and optimize the file structure.
   - This step ensures the transaction file is clean and efficient for processing.

5. **Copy Transaction File for ‘ALL’ Selection**:
   - If the selection type (`?L'121,3'?`) is ‘ALL’:
     - Copies the entire transaction file `?9?E?L'110,6'?` to `?9?AR137S` using `CPYF` (Copy File), replacing any existing content (`MBROPT(*REPLACE)`) and creating the file if it doesn’t exist (`CRTFILE(*YES)`).
     - If `?9?` is ‘G’, copies the file to `?9?AREFTD` in `QS36F` library (`MBROPT(*REPLACE)`, `CRTFILE(*NO)`, `FMTOPT(*NOCHK)`).
     - Also copies to `?9?AREFTD` in `QS36FTEST` library under the same conditions.
     - Jumps to the `AROUND` tag to skip customer-specific filtering.
   - This step prepares the full transaction dataset for EFT processing when all customers are selected.

6. **Set Customer Selection Flags**:
   - For each customer number (`?L'124,6'?` to `?L'178,6'?`, corresponding to `KYCS01` to `KYCS10` in the LDA):
     - If the customer number is `000000`, sets the corresponding LDA offset (13, 16, 19, 22, 25, 28, 31, 34, 37, 40) to ‘IAC’ (include all customers).
     - If non-zero, sets the offset to ‘I*C’ (include specific customer).
   - These flags are used in the sorting step to filter transactions by customer.

7. **Sort and Filter Transactions for Specific Customers**:
   - Loads and runs the `#GSORT` program (`// LOAD #GSORT`, `// RUN`):
     - Input file: `?9?E?L'110,6'?` (transaction file, shared mode).
     - Output file: `?9?AR137S` (temporary output, up to 999,000 records, retained temporarily).
     - Sorting specification (`HSORTR 5A 3X 256 N`):
       - Sorts records in ascending order (`5A`) with a 256-byte record length, no sequence checking (`N`).
     - Input conditions (`I*`):
       - For each customer number (`KYCS01` to `KYCS10`):
         - If the customer number is not `000000`, includes records where:
           - `NECD` (position 1) is not deleted.
           - Company number (`positions 7-8`) matches `?L'101,2'?` (KYCO).
           - Customer number (`positions 9-14`) matches the corresponding customer number (`?L'124,6'?` to `?L'178,6'?`).
         - If the customer number is `000000`, skips the condition (handled by ‘IAC’).
     - Output fields (`F*`):
       - `FNC 2 6`: Outputs the sequence number (positions 2-6).
       - `FDC 1 256`: Outputs the entire 256-byte record.
     - Jumps to `SKIP` tag after each customer’s conditions to avoid redundant checks.
   - This step filters transactions for specific customers when `KYALSL` is ‘SEL’, creating a subset in `?9?AR137S`.

8. **Copy Filtered Data to EFT Files**:
   - If `?9?` is ‘G’:
     - Copies `?9?AR137S` to `?9?AREFTD` in `QS36F` library (`MBROPT(*REPLACE)`, `CRTFILE(*NO)`, `FMTOPT(*NOCHK)`).
     - Copies `?9?AR137S` to `?9?AREFTD` in `QS36FTEST` library under the same conditions.
   - This step ensures the filtered or full transaction data is available in the EFT data files.

9. **Clear EFT Summary File**:
   - Clears the `?9?AREFTS` file using `CLRPFM` (Clear Physical File Member) to prepare it for new summary data.

10. **Generate EFT Reports (First Pass)**:
    - Loads and runs the `AR137A` program (`// LOAD AR137A`, `// RUN`):
      - Files used:
        - `AREFTD` (labeled `?9?AREFTX`, shared mode, likely the filtered transaction data).
        - `AREFTS` (labeled `?9?AREFTS`, shared mode, for summary output).
        - `ARCONT` (labeled `?9?ARCONT`, shared mode, for control data).
        - `ARCUST` (labeled `?9?ARCUST`, shared mode, for customer data).
      - This program likely processes the filtered transactions to generate initial EFT data or reports.

11. **Generate EFT Detail File**:
    - If the file `?9?ARDTGGC` exists, builds it using `BLDFILE` with 500 records, 11 fields, and a key of 8 bytes.
    - Loads and runs the `AR137E` program (`// LOAD AR137E`, `// RUN`):
      - Files used:
        - `AREFTS` (labeled `?9?AREFTS`, shared mode, EFT summary).
        - `ARCUFMX` (labeled `?9?ARCUFMX`, shared mode, customer format data).
        - `ARDTWSC` (labeled `?9?ARDTGGC`, temporary retention, 50 records, output file).
      - This program likely generates detailed EFT data for upload or reporting.

12. **Generate Final EFT Reports**:
    - Loads and runs the `AR137B` program (`// LOAD AR137B`, `// RUN`):
      - Files used:
        - `AREFTD` (labeled `?9?AREFTX`, shared mode).
        - `ARCONT` (labeled `?9?ARCONT`, shared mode).
        - `ARCUST` (labeled `?9?ARCUST`, shared mode).
        - `ARCUFMX` (labeled `?9?ARCUFMX`, shared mode).
        - `ARDTWSC` (labeled `?9?ARDTGGC`, temporary retention, 50 records).
      - If `?9?` is ‘G’, overrides printer files `LIST1` to `LIST4` to output queues `EFTEMALOTQ` or `TESTOUTQ` with 12 CPI (characters per inch).
      - This program prints the final EFT reports for each EFT customer.

13. **Call Test Program (Conditional)**:
    - If `?9?` is ‘G’, calls the `AR137TC` program, likely for testing or validation of the EFT output.

14. **Clean Up Output File**:
    - If `?9?AR137S` exists, deletes it using `DLTF` to clean up temporary data.

15. **Clear Local Data and End**:
    - Clears all local variables or the Local Data Area (LDA) with `// LOCAL BLANK-*ALL`.
    - Reaches the `END` tag, terminating the script.

### Business Rules

The script enforces the following business rules for EFT draft notice filtering:

1. **Environment Setup**:
   - Uses `GSY2K` to ensure date fields are Y2K-compliant.
   - The `?9?` parameter (likely a library prefix) determines specific output queues (`EFTEMALOTQ` or `TESTOUTQ`) and file locations (`QS36F` or `QS36FTEST`).

2. **File Management**:
   - Deletes the temporary file `?9?AR137S` if it exists to ensure a fresh dataset.
   - Reorganizes the transaction file (`?9?E?L'110,6'?`) to remove deleted records before processing.
   - Clears output queues (`EFTEMALOTQ`, `EFTSPLITQ`) to prevent residual print jobs.

3. **Selection Logic**:
   - If `KYALSL` (`?L'121,3'?`) is ‘ALL’, copies the entire transaction file to `?9?AR137S` and `?9?AREFTD` without filtering.
   - If `KYALSL` is ‘SEL’, filters transactions for up to 10 specific customers (`KYCS01` to `KYCS10`) based on company number (`KYCO`) and customer number, excluding deleted records (`NECD`).

4. **Customer Filtering**:
   - For each non-zero customer number, includes records where the company number and customer number match the LDA values.
   - Uses ‘IAC’ (include all customers) or ‘I*C’ (include specific customer) flags to control sorting logic.

5. **Output Generation**:
   - Creates temporary and permanent EFT files (`?9?AR137S`, `?9?AREFTD`, `?9?AREFTS`, `?9?ARDTGGC`).
   - Generates EFT reports via `AR137A` and `AR137B`, using customer and control data.
   - Outputs reports to specified queues (`EFTEMALOTQ` or `TESTOUTQ`) with 12 CPI formatting.

6. **Cleanup**:
   - Clears the `?9?AREFTS` file before generating new summary data.
   - Deletes the temporary `?9?AR137S` file after processing.
   - Clears the LDA to ensure no residual data affects subsequent runs.

### Tables (Files) Used

The script uses the following files:

1. **CRTRAN** (labeled `?9?E?L'110,6'?`):
   - Type: Input (shared mode).
   - Purpose: Source transaction file, reorganized to remove deleted records.

2. **AR137S** (labeled `?9?AR137S`):
   - Type: Output (temporary, up to 999,000 records).
   - Purpose: Stores filtered or full transaction data for EFT processing.

3. **AREFTD** (labeled `?9?AREFTX` or `?9?AREFTD`):
   - Type: Input/Output (shared mode, in `QS36F` or `QS36FTEST`).
   - Purpose: Stores EFT transaction data for processing and reporting.

4. **AREFTS** (labeled `?9?AREFTS`):
   - Type: Input/Output (shared mode).
   - Purpose: Stores EFT summary data, cleared before each run.

5. **ARCONT** (labeled `?9?ARCONT`):
   - Type: Input (shared mode).
   - Purpose: Accounts receivable control data for company validation.

6. **ARCUST** (labeled `?9?ARCUST`):
   - Type: Input (shared mode).
   - Purpose: Customer data for EFT customer details.

7. **ARCUFMX** (labeled `?9?ARCUFMX`):
   - Type: Input (shared mode).
   - Purpose: Customer format data, likely for EFT formatting.

8. **ARDTWSC** (labeled `?9?ARDTGGC`):
   - Type: Output (temporary, 50 records).
   - Purpose: Stores detailed EFT data for upload or reporting.

### External Programs Called

The script invokes the following external programs:

1. **GSY2K**:
   - Utility for Y2K date compliance.

2. **AR137X**:
   - Reorganizes the `CRTRAN` file to remove deleted records.

3. **#GSORT**:
   - Sorts and filters transactions from `?9?E?L'110,6'?` to `?9?AR137S` based on customer selection criteria.

4. **AR137A**:
   - Processes EFT transactions to generate initial EFT data or reports.

5. **AR137E**:
   - Generates detailed EFT data for the `?9?ARDTGGC` file.

6. **AR137B**:
   - Prints final EFT reports for each EFT customer.

7. **AR137TC**:
   - Called conditionally (if `?9?` is ‘G’) for testing or validation of EFT output.

### Integration with Main OCL Script

The `AR137` OCL script is called by the main `AR137P.ocl36.txt` script via the `// AR137 ,,,,,,,,?9?` statement, executed only if the `STATUS` field (`?L'109,1'?`) is ‘Y’ (set by the `AR137P` RPGLE program after successful validation). The `AR137` script uses the LDA fields set by `AR137P` (e.g., `KYCO`, `KYALSL`, `KYCS01` to `KYCS10`, `KYUPDT`) to filter transactions and generate reports, completing the EFT draft notice process.

### Summary

The `AR137` OCL script filters EFT transactions based on the `ALL` or `SEL` criteria, reorganizes the source transaction file, sorts and filters data for specific customers, and generates EFT reports. It ensures clean file management, accurate customer filtering, and proper report output, adhering to strict business rules for data integrity and environment-specific processing.

**Tables Used**: `CRTRAN`, `AR137S`, `AREFTD`, `AREFTS`, `ARCONT`, `ARCUST`, `ARCUFMX`, `ARDTWSC`.
**External Programs Called**: `GSY2K`, `AR137X`, `#GSORT`, `AR137A`, `AR137E`, `AR137B`, `AR137TC`.