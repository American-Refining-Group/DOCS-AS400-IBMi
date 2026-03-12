The provided document, `AR201.ocl36.txt`, is an OCL (Operation Control Language) script used in an IBM AS/400 or iSeries environment, called from the main OCL script `AR201P.ocl36.txt`. It orchestrates the posting of accounts receivable (A/R) invoices and adjustments, including sorting transactions, updating files, generating journals, and producing reports. Below, I’ll explain the process steps, business rules, tables used, and external programs called.

### Process Steps of the OCL Program (AR201)

The `AR201.ocl36.txt` script automates the A/R transaction posting process, leveraging multiple programs and file operations. Here’s a step-by-step breakdown:

1. **Call Program `GSY2K`**:
   - The line `// GSY2K` invokes the program `GSY2K`, likely a Year 2000 compliance utility to ensure proper date handling for the journal date (passed as `?9?` from the main OCL). This ensures dates are formatted correctly for downstream processing.

2. **Sort A/R Transactions (`#GSORT`)**:
   - Loads and runs the sort utility `#GSORT` to sort the A/R transaction file:
     - **Input File**: `?9?ARTRGG` (likely a temporary transaction file, e.g., `GGARTRGG` if `?9? = 'GG'`).
     - **Output File**: `?9?AR201S` (sorted transaction file, e.g., `GGAR201S`), with 999,000 records, extendable by 999,000, retained as a job file (`RETAIN-J`).
     - **Sort Specifications** (`HSORTR`):
       - Sort by:
         - `CE` (Customer Code, position 1) in ascending order (`A`), not equal to blank (`NECD`).
         - `CE` (likely another code, position 54) in ascending order, not equal to blank (`NECE`).
         - `TRCO` (Company Code, positions 7–8).
         - `TRICCD` (Invoice/Credit Code, positions 191–192).
         - `TRTYPE` (Transaction Type, position 31).
         - `TRCUST/TRINV#` (Customer Number/Invoice Number, positions 9–21).
       - Includes the entire record (positions 1–256) in the output (`FDC 1 256`).
   - **Purpose**: Organizes transactions for efficient processing by customer, company, and transaction details.

3. **Post Transactions and Generate Register (`AR200`)**:
   - Loads and runs the program `AR200` to post A/R transactions and produce a transaction register:
     - **Input Files**:
       - `ARTRAN`: Sorted transaction file (`?9?AR201S`).
       - `ARCUST`: Customer master file (`?9?ARCUST`), shared access (`DISP-SHR`).
       - `ARDETL`: A/R detail file (`?9?ARDETL`), shared access.
       - `ARHIST`: A/R history file (`?9?ARHIST`), shared access.
       - `ARCONT`: A/R control file (`?9?ARCONT`), shared access.
     - **Output File**:
       - `ARDIST`: Distribution file (`?9?ARDIGG`), 999,000 records, extendable by 100, retained as a job file.
     - **Printers**:
       - `REPORTP1`: Device `P1`, priority 0.
       - `REPORTP4`: Device `P3`, priority 0.
     - **Conditional Output Queue**:
       - If `?9? = 'G'`, overrides printer file `REPORT` to output queue `QUSRSYS/ARPOST` or `QUSRSYS/TESTOUTQ`.
   - **Purpose**: Updates customer balances, records transaction history, and generates a transaction register report.

4. **Generate S/J or C/R Journal (`AR210`)**:
   - Loads and runs the program `AR210` to create a sales journal (S/J) or cash receipts journal (C/R):
     - **Input Files**:
       - `ARDIST`: Distribution file (`?9?ARDIGG`) from `AR200`.
       - `ARCONT`: A/R control file, shared access.
     - **Output File**:
       - `ARTEMG`: Temporary general ledger file (`?9?ARTGGG`), 999,000 records, extendable by 100, retained as a job file.
   - **Purpose**: Prepares journal entries for general ledger integration based on A/R distributions.

5. **Sort Journal Entries (`#GSORT`)**:
   - Loads and runs `#GSORT` again to sort the temporary general ledger file:
     - **Input File**: `?9?ARTGGG`.
     - **Output File**: `?9?AR211S`, 1,000 records, extendable by 100, retained as a job file.
     - **Sort Specifications** (`HSORTA`):
       - Sort by:
         - `GDCO` (Company Code, positions 2–3).
         - `JRNL/REF` (Journal/Reference, positions 4–7).
         - `S` (Summarize flag, position 96).
         - `ACCOUNT #` (Account Number, positions 13–20).
         - `CR/DB CODE` (Credit/Debit Code, position 12).
   - **Purpose**: Organizes journal entries for posting to the general ledger.

6. **Create Daily File if Needed**:
   - The line `// IFF DATAF1-?9?ARDALY BLDFILE ?9?ARDALY,S,RECORDS,1000,96,,T,,,DFILE` checks if the daily file `?9?ARDALY` exists on unit `F1`. If not, it creates it with 1,000 records, 96 bytes each, as a temporary file (`T`).
   - **Purpose**: Ensures a daily transaction file is available for general ledger posting.

7. **Post to General Ledger (`AR211`)**:
   - Loads and runs the program `AR211` to post journal entries to the general ledger:
     - **Input Files**:
       - `ARTEMG`: Temporary general ledger file (`?9?ARTGGG`).
       - `AR211S`: Sorted journal file (`?9?AR211S`).
       - `TEMGEN`: General ledger temporary file (`?9?TEMGEN`), shared access.
       - `ARCONT`: A/R control file, shared access.
       - `GLMAST`: General ledger master file (`?9?GLMAST`), shared access.
       - `ARDALY`: Daily transaction file (`?9?ARDALY`), shared access.
     - **Printers**:
       - `PRINT`, `PRINTP1`: Device `P1`, priority 0.
     - **Conditional Output Queue**:
       - If `?9? = 'G'`, overrides printer file `REPORT` to `QUSRSYS/ARPOST` or `QUSRSYS/TESTOUTQ`.
   - **Purpose**: Updates the general ledger with A/R journal entries and generates related reports.

8. **Clean Up Temporary File (`$DELET`)**:
   - Loads and runs `$DELET` to perform cleanup.
   - Checks if the transaction file `?9?ARTRGG` exists on unit `F1` and, if so, deletes it using `SCRATCH`.
   - **Purpose**: Removes temporary transaction file to free disk space.

### Business Rules

1. **Transaction Sorting**:
   - Transactions are sorted by customer code, company, invoice/credit code, transaction type, and customer/invoice number to ensure consistent processing order.
   - Journal entries are sorted by company, journal/reference, account number, and credit/debit code, with summarization for general ledger posting.

2. **File Sharing**:
   - Master files (`ARCUST`, `ARDETL`, `ARHIST`, `ARCONT`, `TEMGEN`, `GLMAST`, `ARDALY`) are accessed in shared mode (`DISP-SHR`) to allow concurrent access by other processes.

3. **Temporary Files**:
   - Temporary files (`?9?AR201S`, `?9?ARDIGG`, `?9?ARTGGG`, `?9?AR211S`, `?9?ARDALY`) are created with large record capacities (e.g., 999,000) and retained as job files (`RETAIN-J`) to persist during the job’s execution.
   - The transaction file `?9?ARTRGG` is deleted at the end to clean up.

4. **Output Queue Selection**:
   - If the parameter `?9? = 'G'`, reports are directed to specific output queues (`QUSRSYS/ARPOST` or `QUSRSYS/TESTOUTQ`), likely for production or testing environments.

5. **Daily File Creation**:
   - The daily file `?9?ARDALY` is created only if it doesn’t exist, ensuring availability for general ledger updates.

6. **Journal Date Integration**:
   - The parameter `?9?` (likely the journal date from `AR200P`) is embedded in file labels (e.g., `?9?ARTRGG`, `?9?AR201S`), ensuring all files are tied to the same posting period.

### Tables Used

The OCL script references the following files (tables):
1. **Input Files**:
   - `?9?ARTRGG`: Temporary A/R transaction file (input to first sort).
   - `?9?AR201S`: Sorted A/R transaction file (output from first sort, input to `AR200`).
   - `?9?ARCUST`: A/R customer master file (shared).
   - `?9?ARDETL`: A/R detail file (shared).
   - `?9?ARHIST`: A/R history file (shared).
   - `?9?ARCONT`: A/R control file (shared).
   - `?9?ARDIGG`: A/R distribution file (output from `AR200`, input to `AR210`).
   - `?9?ARTGGG`: Temporary general ledger file (output from `AR210`, input to second sort).
   - `?9?AR211S`: Sorted general ledger file (output from second sort, input to `AR211`).
   - `?9?TEMGEN`: General ledger temporary file (shared).
   - `?9?GLMAST`: General ledger master file (shared).
   - `?9?ARDALY`: Daily transaction file (shared, created if needed).

2. **Output Files**:
   - `?9?AR201S`: Sorted transaction file.
   - `?9?ARDIGG`: A/R distribution file.
   - `?9?ARTGGG`: Temporary general ledger file.
   - `?9?AR211S`: Sorted general ledger file.
   - `?9?ARDALY`: Daily transaction file (if created).

### External Programs Called

The OCL script explicitly calls the following programs:
1. **GSY2K**: Year 2000 compliance utility for date handling.
2. **#GSORT**: Sort utility, called twice for sorting transactions and journal entries.
3. **AR200**: Posts A/R transactions and generates a transaction register.
4. **AR210**: Generates sales or cash receipts journal entries.
5. **AR211**: Posts journal entries to the general ledger.
6. **$DELET**: Deletes the temporary transaction file `?9?ARTRGG`.

### Notes
- **Parameter `?9?`**: Likely the journal date or a prefix (e.g., `GG` as noted in the comment about Atrium compatibility), used consistently in file labels to tie files to the posting period.
- **Atrium Compatibility**: The comment indicates a change to use `GG` as the workstation ID (`?WS?`) to support running in the Atrium environment.
- **File Retention**: Temporary files are marked `RETAIN-J` to persist during the job, except for `?9?ARTRGG`, which is deleted at the end.
- **Output Queues**: Conditional overrides suggest support for production (`ARPOST`) and testing (`TESTOUTQ`) environments.

If you have additional details (e.g., source code for `AR200`, `AR210`, or `AR211`, or specific file layouts), I can provide a deeper analysis. Let me know if you need further clarification or additional information!