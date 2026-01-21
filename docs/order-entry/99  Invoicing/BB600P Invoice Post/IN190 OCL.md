The `IN190.ocl36.txt` is an OCL (Operation Control Language) program for the IBM System/36, designed to split inventory transaction files during the invoice posting process. It creates temporary work files, sets up file labels, and runs the `IN190` program to process inventory transactions. The program handles company-specific data, creates duplicate objects for transaction files, and supports future holding file updates for receipts (per `JB01`). Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the IN190 OCL Program

The `IN190` OCL program performs initialization, file creation, and setup tasks before executing the `IN190` program to process inventory transactions. It uses parameters for batch number and company number to customize file names and operations.

1. **Parameter Evaluation**:
   - The program uses parameters provided by the calling OCL (likely `BB600.ocl36.txt`):
     - `?1?`: Batch identifier (e.g., `BBIN?20?`, where `?20?` is the batch number).
     - `?20?`: Batch number (e.g., `01`).
     - `?9?`: File group prefix (e.g., `G` for the environment or library).
     - `?60?`: Company number (e.g., `01`).
     - `?61?`: Temporary file label for split transactions (e.g., `INTS01`).
     - `?62?`: Temporary file label for zero-amount transactions (e.g., `INTZ01`).
   - If `?1?` is blank, the program jumps to the `END` tag and terminates.
   - The company number (`P60`) is extracted from the Local Data Area (LDA):
     - If `?1?` is `BBIN?20?`, `P60` is set to LDA positions 501–502.
     - Otherwise, `P60` is set to LDA positions 101–102.
   - File labels are constructed:
     - `P61` is set to `INTS + ?L'501,2'?` or `INTS + ?L'101,2'?` (e.g., `INTS01`).
     - `P62` is set to `INTZ + ?L'501,2'?` or `INTZ + ?L'101,2'?` (e.g., `INTZ01`).

2. **Create Temporary Work Files**:
   - The program creates duplicate objects for temporary transaction files (`?9??61?` and `?9??62?`, e.g., `GINTS01` and `GINTZ01`) from the `?9?INTRAN` template in libraries `QS36F` or `QS36FTEST`:
     - If `?9? = G` (indicating production environment):
       - Creates `?9??61?` (e.g., `GINTS01`) in `QS36F` using `CRTDUPOBJ` with constraints (`CST(*NO)`) and no triggers (`TRG(*NO)`).
       - Creates `?9??62?` (e.g., `GINTZ01`) in `QS36F` similarly.
     - For testing environment:
       - Creates `?9??61?` in `QS36FTEST`.
       - Creates `?9??62?` in `QS36FTEST`.
   - Commented-out `BLDFILE` commands suggest older file creation methods (using `BLDFILE` for `?9??61?`, `?9??62?`, `?9?INTZH`, `?9?INFZH` and `BLDINDEX` for `?9?INTZH1`) were replaced by `CRTDUPOBJ` (per `JIMMY K 11/12/2021`).

3. **File Setup and LOAD Statement**:
   - The `LOAD IN190` statement prepares to run the `IN190` program.
   - File assignments are defined with the `FILE` statement:
     - `INTRAN`: Input inventory transaction file, labeled `?9??1?` (e.g., `GBBIN01` for batch `01`).
     - `INCONT`: Invoice control file, labeled `?9?INCONT`, shared access (`DISP-SHR`).
     - `INTSXX`: Temporary split transaction file, labeled `?9??61?` (e.g., `GINTS01`), shared access, extended by 50 records.
     - `INTZXX`: Temporary zero-amount transaction file, labeled `?9??62?` (e.g., `GINTZ01`), shared access, extended by 50 records.
     - `INFRMP`: Invoice ramp file, labeled `?9?INFRMP`, shared access.
     - `INTZH`: Future holding file for receipts, labeled `?9?INTZH`, shared access, extended by 50 records (per `JB01`).
     - `INTANK`: Inventory tank file, labeled `?9?INTANK`, shared access.
     - `ARCUST`: Accounts receivable customer file, labeled `?9?ARCUST`, shared access.
     - `GSTABL`: General table file, labeled `?9?GSTABL`, shared access (commented out, possibly unused).
     - `INTRIT`: Inventory transaction item file, labeled `?9?INTRIT`, shared access, extended by 5000 records.
     - `GSPROD`: Product master file, labeled `?9?GSPROD`, shared access.

4. **Program Execution**:
   - The `RUN` statement executes the `IN190` program, which processes the inventory transactions using the defined files.
   - The program splits transactions from `INTRAN` into:
     - `INTSXX` (e.g., `GINTS01`): Non-zero amount transactions.
     - `INTZXX` (e.g., `GINTZ01`): Zero-amount transactions.
     - `INTZH`: Future holding file for receipts (per `JB01`).
   - Other files (`INCONT`, `INFRMP`, `INTANK`, `ARCUST`, `INTRIT`, `GSPROD`) provide supporting data for transaction processing.

5. **Program Termination**:
   - The program jumps to the `END` tag if `?1?` is blank or after executing `IN190`.
   - No explicit cleanup of temporary files (`?9??61?`, `?9??62?`) is performed in the OCL, suggesting cleanup occurs elsewhere (e.g., in `BB6001.clp` or another program).

---

### Business Rules

1. **Dynamic File Naming**:
   - File labels are constructed using the file group prefix (`?9?`), batch number (`?20?`), and company number (`?L'501,2'?` or `?L'101,2'?`).
   - Example: For `?9? = G`, `?20? = 01`, `?1? = BBIN01`, `?61? = INTS01`, `?62? = INTZ01`, files are labeled `GINTS01` and `GINTZ01`.

2. **Company Number Extraction**:
   - The company number is retrieved from LDA positions 501–502 if `?1?` is `BBIN?20?`, otherwise from 101–102, ensuring flexibility for different calling contexts.

3. **Temporary File Creation**:
   - Temporary files (`?9??61?`, `?9??62?`) are created as duplicates of `?9?INTRAN` in `QS36F` or `QS36FTEST` libraries, with no constraints or triggers (per `JIMMY K 11/12/2021`).
   - This ensures isolated processing for each batch and environment.

4. **Future Holding File (per JB01)**:
   - Receipts are posted to the `INTZH` file for future processing, supporting deferred inventory updates.

5. **File Sharing and Extensibility**:
   - Most files are opened with shared access (`DISP-SHR`) to allow concurrent access by other programs.
   - `INTSXX`, `INTZXX`, `INTZH`, and `INTRIT` are extended (50 or 5000 records) to accommodate additional records during processing.

6. **Error Handling**:
   - The program assumes input parameters and files are valid.
   - No explicit error handling for file creation or execution failures is included in the OCL.

7. **Integration with Invoice Posting**:
   - The program is part of the invoice posting workflow, splitting inventory transactions to support inventory updates and reporting.

---

### Tables (Files) Used

1. **INTRAN**:
   - **Description**: Inventory transaction file.
   - **Label**: `?9??1?` (e.g., `GBBIN01`).
   - **Attributes**: Input file, shared access (`DISP-SHR`).
   - **Purpose**: Contains inventory transactions to be split.
   - **Usage**: Primary input for splitting into `INTSXX` and `INTZXX`.

2. **INCONT**:
   - **Description**: Invoice control file.
   - **Label**: `?9?INCONT`.
   - **Attributes**: Input file, shared access (`DISP-SHR`).
   - **Purpose**: Provides control data for invoice processing.
   - **Usage**: Referenced by `IN190` for transaction validation.

3. **INTSXX**:
   - **Description**: Temporary split transaction file (non-zero amounts).
   - **Label**: `?9??61?` (e.g., `GINTS01`).
   - **Attributes**: Output file, shared access, extended by 50 records.
   - **Purpose**: Stores non-zero amount transactions.
   - **Usage**: Created by `CRTDUPOBJ`, written to by `IN190`.

4. **INTZXX**:
   - **Description**: Temporary zero-amount transaction file.
   - **Label**: `?9??62?` (e.g., `GINTZ01`).
   - **Attributes**: Output file, shared access, extended by 50 records.
   - **Purpose**: Stores zero-amount transactions.
   - **Usage**: Created by `CRTDUPOBJ`, written to by `IN190`.

5. **INFRMP**:
   - **Description**: Invoice ramp file.
   - **Label**: `?9?INFRMP`.
   - **Attributes**: Input file, shared access (`DISP-SHR`).
   - **Purpose**: Contains ramp-related data for inventory processing.
   - **Usage**: Referenced by `IN190`.

6. **INTZH**:
   - **Description**: Future holding file for receipts (per `JB01`).
   - **Label**: `?9?INTZH`.
   - **Attributes**: Output file, shared access, extended by 50 records.
   - **Purpose**: Stores receipt transactions for future processing.
   - **Usage**: Written to by `IN190`.

7. **INTANK**:
   - **Description**: Inventory tank file.
   - **Label**: `?9?INTANK`.
   - **Attributes**: Input file, shared access (`DISP-SHR`).
   - **Purpose**: Contains tank-related inventory data.
   - **Usage**: Referenced by `IN190`.

8. **ARCUST**:
   - **Description**: Accounts receivable customer file.
   - **Label**: `?9?ARCUST`.
   - **Attributes**: Input file, shared access (`DISP-SHR`).
   - **Purpose**: Provides customer data for transaction processing.
   - **Usage**: Referenced by `IN190`.

9. **GSTABL** (Commented Out):
   - **Description**: General table file.
   - **Label**: `?9?GSTABL`.
   - **Attributes**: Input file, shared access (`DISP-SHR`).
   - **Purpose**: Likely contains reference data (e.g., codes, configurations).
   - **Usage**: Not used in current version (commented out).

10. **INTRIT**:
    - **Description**: Inventory transaction item file.
    - **Label**: `?9?INTRIT`.
    - **Attributes**: Output file, shared access, extended by 5000 records.
    - **Purpose**: Stores detailed transaction items.
    - **Usage**: Written to by `IN190`.

11. **GSPROD**:
    - **Description**: Product master file.
    - **Label**: `?9?GSPROD`.
    - **Attributes**: Input file, shared access (`DISP-SHR`).
    - **Purpose**: Contains product master data.
    - **Usage**: Referenced by `IN190` for product details.

---

### External Programs Called

1. **IN190**:
   - **Description**: RPG program that processes inventory transactions.
   - **Purpose**: Splits transactions from `INTRAN` into `INTSXX` (non-zero amounts), `INTZXX` (zero-amount transactions), and `INTZH` (future holding receipts).
   - **Usage**: Loaded and executed via the `LOAD IN190` and `RUN` statements.

---

### Summary

The `IN190.ocl36.txt` OCL program, part of the invoice posting workflow, prepares and executes the `IN190` program to split inventory transactions by:
- Evaluating parameters (`?1?`, `?20?`, `?9?`, `?60?`, `?61?`, `?62?`) to set company number and file labels.
- Creating temporary work files (`?9??61?`, `?9??62?`) using `CRTDUPOBJ` in `QS36F` or `QS36FTEST`.
- Defining file assignments for `INTRAN`, `INCONT`, `INTSXX`, `INTZXX`, `INFRMP`, `INTZH`, `INTANK`, `ARCUST`, `INTRIT`, `GSPROD`.
- Executing `IN190` to process transactions, splitting them into non-zero (`INTSXX`), zero-amount (`INTZXX`), and future holding (`INTZH`) files.
- Terminating if `?1?` is blank or after execution.

**Tables Used**: `INTRAN` (input transactions), `INCONT` (invoice control), `INTSXX` (split transactions), `INTZXX` (zero-amount transactions), `INFRMP` (ramp data), `INTZH` (future holding receipts), `INTANK` (tank data), `ARCUST` (customer data), `GSTABL` (general tables, commented out), `INTRIT` (transaction items), `GSPROD` (product master).
**External Programs Called**: `IN190` (inventory transaction processing).

This program supports the invoice posting workflow by preparing the environment for transaction splitting, ensuring accurate inventory updates and receipt processing.