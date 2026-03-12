The provided document is an **OCL (Operation Control Language) program** named `BI945.ocl36.txt`, used on IBM midrange systems (e.g., AS/400 or IBM i) to control the execution of a job that summarizes sales gallons for pricing, as part of the freight table processing workflow. This program is called before `BI946` in the OCL program `BI946.ocl36.txt` to prepare data for freight calculations. Below, I’ll explain the process steps, business rules (where applicable), external programs called, and tables/files used.

---

### Process Steps of the OCL Program

The OCL program `BI945.ocl36.txt` is designed to summarize sales gallons for pricing purposes by sorting sales data and generating a work file for subsequent processing. Here’s a step-by-step breakdown of the process:

1. **Header and Context**:
   - The header `** PRICING - SUMMARIZE SALES GALLONS` indicates the purpose: to aggregate sales data (likely gallons sold) for freight pricing.
   - Parameters:
     - `?L'201,8'?`: Represents today’s date in CCYYMMDD format.
     - `?L'209,8'?`: Represents the date one year ago in CCYYMMDD format.
   - These dates are used to filter sales data within a one-year period.

2. **GSY2K Directive**:
   - `// GSY2K`: A system-specific directive or comment, likely ensuring Year 2000 compliance for date handling.

3. **Date Handling (Commented Out)**:
   - The commented section (`** GET TODAY'S DATE (CYMD)` and `** GET 1 YR AGO TODAY DATE (CYMD)`) outlines how the program would retrieve and set today’s date and the date one year ago:
     - `LOCAL OFFSET-501,DATA-'?DATE?'`: Would retrieve the system date.
     - `LOCAL OFFSET-201,DATA-'?L'509,2'?'`: Would set the century for today’s date.
     - `LOCAL OFFSET-203,DATA-'?L'505,2'??L'501,2'??L'503,2'?'`: Would format the date as CCYYMMDD.
     - `EVALUATE P20=?L'201,8'?-00010000`: Would calculate one year ago by subtracting 1 year from today’s date.
     - `LOCAL OFFSET-209,DATA-'?20?'`: Would set the one-year-ago date.
   - These lines are commented out, suggesting the dates (`?L'201,8'?` and `?L'209,8'?`) are passed as parameters from the calling program (e.g., `BI946P`).

4. **Pause (Commented Out)**:
   - `// PAUSE '?L'201,20'?'`: Would pause execution to display a message (likely today’s date with additional text). Commented out, so not executed.

5. **Clear Pricing Work File**:
   - `// CLRPFM ?9?PRICWK`: Clears the physical file `PRICWK` (prefixed by `?9?`, a dynamic library or identifier) to ensure no residual data affects the process.

6. **Sort Data Using #GSORT**:
   - `// LOAD #GSORT`: Loads the system sort utility (`#GSORT`) to sort sales data.
   - **Input Files**:
     - `// FILE NAME-INPUT1,LABEL-?9?SA5FILD,DISP-SHR`: Specifies `SA5FILD` (sales file, shared read mode) as the primary input.
     - `// FILE NAME-INPUT2,LABEL-?9?SA5MOVD,DISP-SHR`: Specifies `SA5MOVD` (movement or transaction file, shared read mode) as a secondary input.
   - **Output File**:
     - `// FILE NAME-OUTPUT,LABEL-?9?BI945S,RECORDS-999000,EXTEND-999000,RETAIN-J`: Defines `BI945S` as the output file with a capacity of 999,000 records, extensible by another 999,000, and retained as a job file (`RETAIN-J`).
   - **Run Sort**:
     - `// RUN` executes the sort with the following specifications:
       - `HSORTR 34A 3X1024 N`: Defines a sort with a 34-byte ascending key, repeated 3 times (multiple key fields), handling up to 1024 bytes of data, with no sequence checking (`N`).
       - `I C 1NECD`: Excludes records where byte 1 is not equal to `'D'` (deleted records).
       - `IAC 227 234GEC?L'209,8'?`: Includes records where the date (bytes 227–234, CCYYMMDD) is greater than or equal to one year ago.
       - `IAC 227 234LEC?L'201,8'?`: Includes records where the date is less than or equal to today.
       - Output fields:
         - `FNC 2 3 CO`: Company number (bytes 2–3).
         - `FNC 4 9 CUST`: Customer number (bytes 4–9).
         - `FNC 17 19 SHIPTO`: Ship-to number (bytes 17–19).
         - `FNC 45 47 LOC`: Location code (bytes 45–47).
         - `FNC 48 51 PROD`: Product code (bytes 48–51).
         - `FNC 193 195 CNTR`: Container code (bytes 193–195).
         - `FNC 76 78 UNMS`: Unit of measure (bytes 76–78).
         - `FDC 1 256`, `FDC 257 512`, `FDC 513 768`, `FDC 769 1024`: Copies full data records (up to 1024 bytes) to the output file.
   - **End Sort**:
     - `// END`: Completes the sort operation, producing the sorted file `BI945S`.

7. **Load and Run BI945 Program**:
   - `// LOAD BI945`: Loads the program `BI945` (likely an RPG or CL program).
   - **Files Defined**:
     - `SA5FILWK` (`?9?BI945S`, shared mode): The sorted file from the previous step, used as input.
     - `SA5SHY` (`?9?SA5SHY`, shared mode): Likely a history or summary file for sales data.
     - `PRICWK` (`?9?PRICWK`): Output file for pricing work data, used in subsequent programs (e.g., `BI946`).
   - `// RUN`: Executes `BI945`, which likely summarizes sales gallons (e.g., aggregating quantities by company, customer, ship-to, location, product, and container) and writes the results to `PRICWK`.

---

### Business Rules

The OCL program enforces the following business rules:
1. **Date Range Filtering**:
   - Processes sales data within a one-year period, from one year ago (`?L'209,8'?`) to today (`?L'201,8'?`), based on the date field in bytes 227–234 of the input files.
2. **Record Exclusion**:
   - Excludes deleted records (`1NECD`, byte 1 ≠ `'D'`).
3. **Data Sorting**:
   - Sorts sales data by company, customer, ship-to, location, product, container, and unit of measure to organize data for summarization.
4. **File Clearing**:
   - Clears `PRICWK` to ensure a fresh dataset for pricing calculations.
5. **Output Structure**:
   - The sorted file `BI945S` includes key fields (company, customer, ship-to, location, product, container, unit of measure) and full data records for further processing.

---

### External Programs Called

1. **#GSORT**:
   - System sort utility loaded with `// LOAD #GSORT`.
   - Used to sort `SA5FILD` and `SA5MOVD` into `BI945S` based on specified keys and date filters.

2. **BI945**:
   - Loaded with `// LOAD BI945` and executed with `// RUN`.
   - Likely an RPG or CL program that summarizes sales gallons and writes results to `PRICWK`.

---

### Tables/Files Used

1. **Input Files**:
   - `SA5FILD` (`?9?SA5FILD`, shared read mode): Sales file, likely containing detailed sales transactions.
   - `SA5MOVD` (`?9?SA5MOVD`, shared read mode): Movement or transaction file, likely containing additional sales data.
   - `SA5SHY` (`?9?SA5SHY`, shared mode): Sales history or summary file, used by `BI945`.

2. **Output/Temporary Files**:
   - `BI945S` (`?9?BI945S`): Sorted output file from `#GSORT`, with a capacity of 999,000 records.
   - `PRICWK` (`?9?PRICWK`): Pricing work file, cleared and used as output by `BI945`.

---

### Summary

- **Process Steps**: The OCL program uses passed date parameters (today and one year ago), clears the `PRICWK` file, sorts sales data from `SA5FILD` and `SA5MOVD` into `BI945S` using `#GSORT` (filtering by date and excluding deleted records), runs `BI945` to summarize sales gallons, and writes results to `PRICWK`.
- **Business Rules**: Filters sales data for a one-year period, excludes deleted records, and sorts by company, customer, ship-to, location, product, container, and unit of measure.
- **External Programs**: `#GSORT` (sorts data), `BI945` (summarizes sales gallons).
- **Tables/Files**: Input: `SA5FILD`, `SA5MOVD`, `SA5SHY`. Output/Temporary: `BI945S`, `PRICWK`.

If you have the source code for `BI945` or additional details about the file structures, I can provide a more detailed analysis of the summarization logic. Let me know if you need further clarification!