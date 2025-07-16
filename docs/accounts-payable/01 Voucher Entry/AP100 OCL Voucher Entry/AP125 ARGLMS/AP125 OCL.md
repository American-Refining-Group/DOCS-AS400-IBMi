The provided OCL (Operation Control Language) script, `AP125.ocl36.txt`, is part of an IBM AS/400 (iSeries) system and appears to be related to Accounts Payable (A/P) voucher transaction processing. It is called from another OCL script that sets up voucher entry tables. Below, I will explain the process steps of this OCL program, identify the external programs called, and list the tables (files) used.

---

### **Process Steps of the OCL Program**

The OCL script performs a series of steps to prepare and process Accounts Payable voucher transactions. Here's a breakdown of the steps based on the script:

1. **Invoice Type Selection**:
   - The script starts with a comment indicating `AP ARGLMS OR PAPER` and a parameter `?8?` for selecting the invoice type (`N` or `P`). This suggests the script supports different processing modes for invoices (possibly non-PO or PO-based).

2. **File Creation/Verification**:
   - The script checks for the existence of specific files using `IFF` (If) conditions:
     - `DATAF1-?9??20?`: Verifies if the file `?9??20?` exists. If it does, it builds a file (`BLDFILE`) with specific attributes (500 records, 404 bytes per record, etc.).
     - `DATAF1-?9?APTX?WS?`: If this file exists, it builds an index (`BLDINDEX`) for it with specific key fields and attributes.
     - `DATAF1-?9?LMS?WS?`: If this file exists, it builds another file with similar attributes.
   - These steps ensure that necessary temporary or work files (`?9??20?`, `?9?APTX?WS?`, `?9?LMS?WS?`) are created or validated before proceeding.

3. **Override Database Files (OVRDBF)**:
   - The script uses multiple `OVRDBF` commands to override the default file definitions, specifying the library (`*LIBL/?9?`) and member (`*FIRST`) for various files. This ensures the program uses the correct files from the specified library. The files overridden include:
     - `APCONT`, `GSTABL`, `FRCINH`, `FRCFBH`, `FRCINH4`, `GLMAST`, `APVEND`, `APVENY`, `SA5FIUD`, `SA5FIUM`, `SA5MOUD`, `SA5MOUM`, `GSCTUM`, `APDATE`, `BICONT`, `APTRAN`.
   - These overrides are critical to ensure the correct data files are accessed during processing.

4. **Call External Program (AP125P)**:
   - The script calls an external program named `AP125P` with parameters:
     - `'10'`: Likely a mode or control parameter.
     - `'?8?'`: The invoice type (`N` or `P`).
     - `'MNT'`: Indicates maintenance mode (possibly for voucher entry or editing).
     - `'?9?'`: A library or file prefix (dynamic substitution variable).
   - This program (`AP125P`) likely performs the core logic for voucher transaction processing, such as editing or validating transactions.

5. **Conditional File Deletion and Creation**:
   - The commented-out `GSDELETE APCT?WS?` suggests a potential step to delete an existing work file (not executed in this script).
   - A commented-out `BLDFILE ?9?APCT?WS?` indicates a step to create a temporary file with 500 records and 80 bytes per record (not executed).
   - A `LOCAL OFFSET-221,DATA-'?9??20?'` command suggests setting a local variable or data area with the file name `?9??20?`.

6. **Conditional Exit**:
   - The `IFF DATAF1-?9??20? GOTO END` checks if the file `?9??20?` exists. If it does not, the script jumps to the `END` tag, bypassing the subsequent steps.

7. **Voucher Transaction Edit (AP110)**:
   - If the file `?9??20?` exists, the script proceeds to the `LOAD AP110` section, which is responsible for A/P voucher transaction editing.
   - **File Definitions**:
     - The script defines multiple files with their labels and attributes (e.g., `DISP-SHR` for shared access, `EXTEND-100` for extending file size). The files include:
       - `APTRAN` (labeled `?9??20?`): Likely the main transaction file.
       - `APCONT`, `APCHKR`, `APCHKT` (labeled `?9?APCT?WS?`), `APTRNX` (labeled `?9?APTX?WS?`), `GLMAST`, `APOPNHC`, `APINVH`, `GSTABL`, `APSTAT`, `APVEND`, `INFIL1`, `INTZH1`.
   - **Printer Overrides**:
     - If the condition `?9?/G` is true, the script overrides the printer file `APLIST` to output to `QUSRSYS/APEDIT`.
     - If `?9?/G` is false, it overrides `APLIST` to output to `QUSRSYS/TESTOUTQ`.
   - **Execution**:
     - The `RUN` command executes the `AP110` program, which performs the voucher transaction edit, likely validating or processing transactions and generating a report.

8. **End of Script**:
   - The `TAG END` marks the end of the script, where processing terminates if the earlier `GOTO END` condition is met.

---

### **External Programs Called**

The OCL script explicitly calls the following external programs:
1. **AP125P**:
   - Called with parameters: `'10'`, `'?8?'`, `'MNT'`, `'?9?'`.
   - Likely responsible for the main processing of voucher transactions (e.g., data entry, validation, or updates).
2. **AP110**:
   - Loaded in the voucher transaction edit section.
   - Handles the editing or validation of A/P voucher transactions and possibly generates a report.

---

### **Tables (Files) Used**

The script references the following files (tables) for processing:

1. **From OVRDBF Commands**:
   - `APCONT`: Accounts Payable control file.
   - `GSTABL`: General system table.
   - `FRCINH`, `FRCFBH`, `FRCINH4`: Likely related to financial or reconciliation data.
   - `GLMAST`: General Ledger master file.
   - `APVEND`: Vendor master file.
   - `APVENY`: Additional vendor file (possibly for secondary vendor data).
   - `SA5FIUD`, `SA5FIUM`, `SA5MOUD`, `SA5MOUM`: Likely related to specific application modules (e.g., financial or manufacturing).
   - `GSCTUM`: General system customer or control file.
   - `APDATE`: Accounts Payable date file.
   - `BICONT`: Possibly a billing or contract control file.
   - `APTRAN`: Accounts Payable transaction file (labeled `?9??20?`).

2. **From AP110 Section**:
   - `APTRAN` (labeled `?9??20?`): Main transaction file.
   - `APCONT` ( labeled `?9?APCONT`): Control file.
   - `APCHKR`: Check register file.
   - `APCHKT` (labeled `?9?APCT?WS?`): Temporary check file.
   - `APTRNX` (labeled `?9?APTX?WS?`): Transaction index file.
   - `GLMAST` (labeled `?9?GLMAST`): General Ledger master.
   - `APOPNHC`: Open invoice header file.
   - `APINVH`: Invoice header file.
   - `GSTABL` (labeled `?9?GSTABL`): General system table.
   - `APSTAT`: Accounts Payable status file.
   - `APVEND` (labeled `?9?APVEND`): Vendor master.
   - `INFIL1`, `INTZH1`: Additional files, possibly for interface or temporary data.
   - `APLIST`: Printer file for output (report).

3. **Temporary/Work Files**:
   - `?9??20?`: Main transaction work file.
   - `?9?APTX?WS?`: Transaction index work file.
   - `?9?LMS?WS?`: Additional work file.
   - `?9?APCT?WS?`: Temporary check file.

---

### **Summary**

The `AP125.ocl36.txt` OCL script is designed to set up and process Accounts Payable voucher transactions. It performs the following high-level steps:
1. Validates or creates necessary work files.
2. Overrides database files to ensure correct file access.
3. Calls the `AP125P` program for core voucher processing.
4. If the main transaction file exists, it loads the `AP110` program to edit or validate transactions and generate reports.
5. Uses printer overrides to direct output to specific queues.

**External Programs**:
- `AP125P`
- `AP110`

**Tables/Files**:
- Core files: `APCONT`, `GSTABL`, `GLMAST`, `APVEND`, `APVENY`, `APDATE`, `APTRAN`, etc.
- Temporary/work files: `?9??20?`, `?9?APTX?WS?`, `?9?LMS?WS?`, `?9?APCT?WS?`.
- Printer file: `APLIST`.

This script is tightly integrated with the AS/400 environment, relying on dynamic file naming (via `?9?` and `?8?`) and external RPG programs to handle the business logic. If you need further clarification or details about specific files or programs, please let me know!