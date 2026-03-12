The provided document, `BI941.ocl36.txt`, is an Operation Control Language (OCL) program for IBM System/36 or AS/400, designed to print a customer sales agreement master file. It orchestrates data processing, sorting, and reporting by invoking various programs and applying selection criteria based on user inputs from the `BI941P` RPG program. Below, I’ll explain the process steps, business rules, external programs called, and tables used.

---

### **Process Steps of the BI941 OCL Program**

The OCL program sets up the environment, processes data, sorts records, and generates a report based on user-specified criteria (e.g., company, customer, contract, salesman, product, and sort order). Here’s a detailed breakdown of the process steps:

1. **Initial Setup and Comments**:
   - The program starts with comments indicating its purpose: "PRINT CUSTOMER SALES AGREEMENT MASTER FILE."
   - `// GSY2K`: Likely a reference to Year 2000 compliance or a system configuration module.
   - No initial switch settings or local variable clearing are specified at the start (unlike `BI941P.ocl36`), but they appear at the end.

2. **Parameter-Based Local Data Assignments**:
   - The program uses conditional `IF` statements to set local variables based on user input parameters from the `BI941P` RPG program (stored in `?L'position,length'?`). These parameters control filtering and sorting:
     - **Company Selection** (`?L'111,3'?`):
       - If `CO`, set `OFFSET-1` to `IAC` (include all companies).
       - Else, set to `I*C` (specific company).
     - **Customer Selection** (`?L'123,3'?`):
       - If `SEL`, set `OFFSET-4` to `O COAC` (selected customers).
       - Else, set to `O*CO*C` (all customers).
     - **Contract Selection** (`?L'186,3'?`):
       - If `CUR`, set `OFFSET-10` to `IAC` (current contracts).
       - Else, set to `I*C` (all contracts).
     - **Sort Order** (`?L'197,1'?`):
       - If `S`, set `OFFSET-13` to `FNC` (sort by salesman).
       - Else, set to `F*C` (sort by name).
     - **Location Selection** (`?L'198,3'?`):
       - If blank, set `OFFSET-16` to `I*C` (all locations).
       - Else, set to `IAC` (specific location).
     - **Product Selection** (`?L'201,4'?`):
       - If blank, set `OFFSET-19` to `I*C` (all products).
       - Else, set to `IAC` (specific products).
     - **Salesman Range Selection** (`?L'209,3'?`):
       - If `SEL`, set `OFFSET-22` to `IAC` (specific salesman).
       - Else, set to `I*C` (all salesmen).

3. **Pre-Processing Zero Ending Dates**:
   - `CLRPFM FILE(?9?BICUAGO)`: Clears the physical file `BICUAGO` (output file for pre-processing).
   - `// LOAD BI9413`: Loads the `BI9413` program to preprocess records with zero ending dates, replacing them with `123179` (likely December 31, 1979, to ensure compatibility).
   - **Files Used**:
     - `BICUAG`: Input customer agreement file.
     - `GSCNTR1`: Input contract file.
     - `BICUAGO`: Output file for processed agreements.
     - `GSPROD`: Product file for reference data.
   - `// RUN`: Executes `BI9413` to process the data.

4. **Initial Sorting with #GSORT**:
   - `// LOAD #GSORT`: Loads the system sort utility (`#GSORT`).
   - **Files**:
     - Input: `?9?BICUAGO` (pre-processed agreements).
     - Output: `?9?BI941S` (sorted output file, up to 999,000 records, retained as a job file).
   - **Sort Specifications**:
     - `HSORTR 52A 3X 258 N`: Sorts records (52-character keys, ascending, no sequence checking).
     - Multiple sort conditions check for non-deleted records (`1NECD`) and match criteria like company (`?L'1,3'?`), ending date (`?L'10,3'?`), location (`?L'16,3'?`), and up to 10 products (`?L'19,3'?`).
     - Sort fields include company number, customer number, container code, unit of measure, bill-to PO, location, contract, and sequence number.
   - `// IF ?L'201,4'?/ GOTO ENDSEL`: Skips additional product sorting if no product is specified.
   - Additional sort conditions handle up to 10 products, ensuring records match product codes (`?L'201,4'?`) at different positions.
   - `// END`: Completes the sort.

5. **Conditional Processing Based on Salesman and Sort Order**:
   - **If Salesman is `ALL` and Sort is `S`** (`?L'209,3'?/ALL` and `?L'197,1'?/S`):
     - Jump to `REPT` (report generation).
   - **Load BI9411**:
     - If the above condition is not met, load `BI9411`.
     - **Files**:
       - `BICUAGXX` (input from `?9?BI941S`).
       - `ARCUST` (customer file, shared read mode).
     - `// RUN`: Executes `BI9411` to process sorted data.

6. **Salesman-Based Sorting**:
   - **If Salesman is `SEL` and Sort is `N`** (`?L'209,3'?/SEL` and `?L'197,1'?/N`):
     - Jump to `CUSSEL` (customer selection).
   - **Load #GSORT** (for salesman sort):
     - Input: `?9?BI941S`.
     - Output: `?9?BI941S` (overwrites with sorted data).
     - **Sort Specifications**:
       - Sorts by salesman (`?L'22,3'?` matches `?L'205,2'?`), company, customer, and other fields.
       - Resets salesman number to zero for customer report sorting.
     - `// END`: Completes the sort.

7. **Customer Selection Sorting (CUSSEL)**:
   - **Load #GSORT**:
     - Input: `?9?BI941S`.
     - Output: `?9?BI941S`.
     - **Sort Specifications**:
       - Similar to salesman sort but prioritizes customer sorting.
     - `// RUN`: Executes the sort.
   - **Load BI9412**:
     - Input: `?9?BI941S` (sorted file).
     - `// RUN`: Executes `BI9412` to further process customer-selected data.

8. **Report Generation (REPT)**:
   - **Load BI941**:
     - **Files**:
       - `BICUAGXX` (input from `?9?BI941S`).
       - `ARCUST` (customer file, shared read).
       - `BICONT` (company control file, shared read).
       - `GSTABL` (general system table, shared read).
       - `GSPROD` (product file, shared read).
       - `GSCNTR1` (contract file, shared read).
     - `// RUN`: Executes `BI941` to generate the final customer sales agreement report.

9. **Optional Duplicate Agreement Check**:
   - A commented section warns against running `BI941A` unless its key references are updated.
   - `CLRPFM FILE(BI941O)`: Clears the output file `BI941O`.
   - **Load BI941A**:
     - Input: `?9?BI941S`.
     - Output: `BI941O` (shared mode).
     - `// RUN`: Executes `BI941A` to check for duplicate current agreements (skipped unless explicitly enabled).

10. **Cleanup**:
    - `// TAG JUMP`: Skips the duplicate check if not needed.
    - `// SWITCH 00000000`: Resets all switches.
    - `// LOCAL BLANK-*ALL`: Clears all local variables.

---

### **Business Rules**

1. **Parameter-Based Filtering**:
   - Filters records based on user inputs for company (`ALL`/`CO`), customer (`ALL`/`SEL`), contract (`ALL`/`CUR`), sort order (`N`/`S`), location, product, and salesman (`ALL`/`SEL`).
   - Uses `IAC` (include all) or `I*C` (specific) to control record inclusion.

2. **Zero Ending Date Handling**:
   - Replaces zero ending dates with `123179` to ensure compatibility (handled by `BI9413`).

3. **Sorting Logic**:
   - Initial sort (`#GSORT`) filters records by company, date, location, and up to 10 products.
   - Additional sorting depends on:
     - Salesman sort (`S`): Sorts by salesman, company, customer, etc., and resets salesman number for customer reports.
     - Customer sort (`N` with `SEL`): Sorts by customer and other fields.

4. **Report Generation**:
   - The final report (`BI941`) uses sorted data and reference files (`ARCUST`, `BICONT`, `GSTABL`, `GSPROD`, `GSCNTR1`) to produce a customer sales agreement list.
   - Fields in the report include company number, customer number, container code, unit of measure, bill-to PO, location, contract, and sequence number.

5. **Duplicate Check**:
   - An optional step (`BI941A`) checks for duplicate agreements but is disabled by default due to key reference issues.

---

### **External Programs Called**

The OCL program invokes the following external programs:
1. **BI9413**: Pre-processes customer agreement data to handle zero ending dates.
2. **#GSORT**: System sort utility, used multiple times for sorting by salesman or customer.
3. **BI9411**: Processes sorted data when salesman is `ALL` and sort is not `S`.
4. **BI9412**: Processes customer-selected data when salesman is `SEL` and sort is `N`.
5. **BI941**: Generates the final customer sales agreement report.
6. **BI941A**: Optional program to check for duplicate agreements (disabled by default).

---

### **Tables Used**

The following files/tables are used:
1. **BICUAG**: Input customer agreement file (source data).
2. **BICUAGO**: Output file for pre-processed agreements (after `BI9413`).
3. **BI941S**: Sorted output file used across multiple steps.
4. **BI941O**: Output file for duplicate agreement check (used by `BI941A`).
5. **ARCUST**: Customer file (shared read mode, used by `BI9411` and `BI941`).
6. **BICONT**: Company control file (shared read mode, used by `BI941`).
7. **GSTABL**: General system table (shared read mode, used by `BI941`).
8. **GSPROD**: Product file (shared read mode, used by `BI9413` and `BI941`).
9. **GSCNTR1**: Contract file (shared read mode, used by `BI9413` and `BI941`).

---

### **Summary**

- **Process Steps**:
  1. Set local variables based on user input parameters for filtering.
  2. Pre-process agreement data (`BI9413`) to handle zero ending dates.
  3. Sort data (`#GSORT`) by company, customer, and other criteria, with product filtering.
  4. Conditionally process data based on salesman and sort order:
     - Run `BI9411` for non-salesman sorts.
     - Sort again (`#GSORT`) for salesman or customer selection.
     - Run `BI9412` for customer-selected data.
  5. Generate the final report (`BI941`).
  6. Optionally check for duplicates (`BI941A`, disabled).
  7. Clean up switches and variables.

- **External Programs Called**:
  - `BI9413`, `#GSORT`, `BI9411`, `BI9412`, `BI941`, `BI941A` (optional).

- **Tables Used**:
  - `BICUAG`, `BICUAGO`, `BI941S`, `BI941O`, `ARCUST`, `BICONT`, `GSTABL`, `GSPROD`, `GSCNTR1`.

This OCL program orchestrates a complex workflow to filter, sort, and report customer sales agreements based on user inputs validated by `BI941P`, ensuring flexibility in handling various selection criteria.