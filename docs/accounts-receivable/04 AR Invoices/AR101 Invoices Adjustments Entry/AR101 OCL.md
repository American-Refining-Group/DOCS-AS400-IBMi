The provided document is an OCL (Operation Control Language) program, specifically `AR101.ocl36.txt`, which appears to be a script for an IBM System/36 environment used to manage Accounts Receivable (A/R) processes. Below, I will explain the process steps of the program, identify the external programs called, and list the tables (files) used.

---

### **Process Steps of the AR101 OCL Program**

The OCL program is designed to handle A/R transactions, specifically for editing invoices and adjustments. It includes conditional logic, file operations, sorting, and program execution. Here’s a step-by-step breakdown of the process:

1. **Initial Program Call and Conditional Check**:
   - The program starts by calling `GSGENIEC`, likely a utility or initialization program.
   - A condition checks the value at memory location `L'506,3'`. If the condition is met (`/YES`), the program returns immediately, halting further execution.
   - If the condition is not met, the program proceeds.

2. **SCPROCP and GSY2K Execution**:
   - The `SCPROCP` command is executed with placeholder parameters (`?9?`), which may resolve to a specific library or file name at runtime.
   - `GSY2K` is called, likely a program to handle Year 2000 compliance or date-related processing.

3. **Conditional Branching for Editing**:
   - The program checks if the first parameter (`?1?`) is set to `EDIT`. If true, it branches to the `EDIT` tag to process invoice and adjustment edits. If false, it continues with the next steps.

4. **File Creation for A/R Transactions**:
   - If the file `?9?ARTRGG` exists in `DATAF1`, the program builds a new file named `?9?ARTRGG` with the following attributes:
     - Type: `A` (alternate index or sequential file).
     - Record size: 500 bytes, 256 records.
     - File type: Permanent (`P`), with 2 keys and a key length of 5.

5. **Load and Run AR101**:
   - The program loads `AR101`, the main A/R processing program.
   - It defines several files with `DISP-SHR` (shared disposition) and specific labels:
     - `ARTRAN`: A/R transaction file (`?9?ARTRGG`), with an extension of 100 records.
     - `ARTRANR`: A read-only version of the A/R transaction file (`?9?ARTRGG`).
     - `ARCUST`: Customer master file (`?9?ARCUST`).
     - `ARCUSTX`: Customer index file (`?9?ARCUSX`).
     - `ARDETL`: A/R detail file (`?9?ARDETL`).
     - `GLMAST`: General ledger master file (`?9?GLMAST`).
     - `ARCONT`: A/R control file (`?9?ARCONT`).
     - `GSTABL`: General system table (`?9?GSTABL`).
   - The `RUN` command executes `AR101` to process A/R transactions.

6. **Edit Invoices and Adjustments (EDIT Tag)**:
   - If the `EDIT` condition is met, the program branches to the `EDIT` tag.
   - **Delete Temporary File**:
     - Loads `$DELET`, a utility program to delete files.
     - If `?9?AR111S` exists in `DATAF1`, it scratches (deletes) the file labeled `?9?AR111S`.
   - **Sort Transactions**:
     - Loads `#GSORT`, a sorting utility.
     - Defines input and output files:
       - Input: `?9?ARTRGG` (A/R transaction file).
       - Output: `?9?AR111S` (sorted output file, with 999,000 records and extendable by 999,000).
     - Executes a sort operation (`HSORTA`) with the following parameters:
       - Sort field: 15 bytes, ascending (`A`), starting at position 3, numeric (`N`).
       - Condition: `1NECD` (possibly a flag or condition code).
       - Fields to sort: Positions 7 to 21 (likely company/customer/invoice number).
     - The sorted output is written to `?9?AR111S`.

7. **Check for Sorted File Existence**:
   - The program checks if the file `?9?AR111S` has zero records (`/00000000`). If true, it branches to the `END` tag, skipping further processing.

8. **Load and Run AR111**:
   - If the sorted file exists and has records, the program loads `AR111`, a program for editing invoices and adjustments.
   - Defines the following files (all shared):
     - `AR111S`: Sorted transaction file (`?9?AR111S`).
     - `ARTRAN`: A/R transaction file (`?9?ARTRGG`).
     - `ARCUST`: Customer master file (`?9?ARCUST`).
     - `ARDETL`: A/R detail file (`?9?ARDETL`).
     - `GLMAST`: General ledger master file (`?9?GLMAST`).
     - `ARCONT`: A/R control file (`?9?ARCONT`).
     - `GSTABL`: General system table (`?9?GSTABL`).
   - The `RUN` command executes `AR111` to process the sorted transactions.

9. **Final Cleanup (END Tag)**:
   - The program branches to the `END` tag.
   - Loads `$DELET` again to delete the temporary sorted file.
   - If `?9?AR111S` exists in `DATAF1`, it scratches the file.

---

### **External Programs Called**

The OCL program calls the following external programs:
1. **GSGENIEC**: Likely a general initialization or utility program.
2. **GSY2K**: A program for Year 2000 compliance or date processing.
3. **AR101**: The main A/R transaction processing program.
4. **$DELET**: A system utility for deleting files.
5. **#GSORT**: A system utility for sorting files.
6. **AR111**: A program for editing invoices and adjustments.

---

### **Tables (Files) Used**

The program references the following files (referred to as "tables" in the context of System/36):
1. **ARTRAN** (`?9?ARTRGG`): A/R transaction file, used for reading and writing transaction data.
2. **ARTRANR** (`?9?ARTRGG`): A read-only version of the A/R transaction file.
3. **ARCUST** (`?9?ARCUST`): Customer master file, containing customer information.
4. **ARCUSTX** (`?9?ARCUSX`): Customer index file, likely used for quick lookups.
5. **ARDETL** (`?9?ARDETL`): A/R detail file, containing detailed transaction records.
6. **GLMAST** (`?9?GLMAST`): General ledger master file, used for financial integration.
7. **ARCONT** (`?9?ARCONT`): A/R control file, containing control parameters for A/R processing.
8. **GSTABL** (`?9?GSTABL`): General system table, containing system-wide configuration data.
9. **AR111S** (`?9?AR111S`): Temporary sorted transaction file, created and deleted during execution.

---

### **Summary**

The `AR101.ocl36.txt` program manages A/R transactions by:
- Initializing with `GSGENIEC` and `GSY2K`.
- Conditionally building and processing A/R transaction files.
- Sorting transactions and editing invoices/adjustments using `AR111`.
- Performing cleanup by deleting temporary files.
- Interacting with multiple A/R and general ledger files to ensure data integrity.

The program uses a combination of conditional logic, file operations, and sorting to prepare and process A/R data, with a focus on invoice and adjustment editing. The external programs and files listed above are critical to its operation.