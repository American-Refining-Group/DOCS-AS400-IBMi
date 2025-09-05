The document `BB751.ocl36.txt` is an **Open Control Language (OCL)** program for the IBM System/36 or AS/400, responsible for generating a **Summary Open Railcar Bill of Lading (BOL) Listing by Order**. It is called from the main OCL program (`BB750P.ocl36.txt`) when a summary report is requested (based on the `KYDETL` flag not set to `'Y'`). This program is similar to `BB750.ocl36.txt` (for the detailed report) but generates a summarized version of the BOL listing. Below, I’ll explain the process steps, business rules, tables/files used, and external programs called.

---

### **Process Steps of the OCL Program**

The `BB751` OCL program performs two main tasks: sorting the BOL data and generating a summary report. It uses the `#GSORT` utility to sort the data and then executes the `BB751` program to produce the report. Here’s a step-by-step breakdown:

1. **Initialization and Environment Setup**:
   - `// GSY2K`: Likely a system or environment setup command, possibly related to Y2K compliance or system configuration.

2. **Conditional Local Variable Setup**:
   - `// IF ?L'103,3'?/SEL LOCAL OFFSET-1,DATA-'O COAC'`
     - Checks if the parameter at position 103 (length 3) equals `'SEL'`.
     - If true, sets a local variable at offset 1 to `'O COAC'` (likely indicating a specific company or account selection).
   - `// ELSE LOCAL OFFSET-1,DATA-'O*CO*C'`
     - If false, sets the local variable to `'O*CO*C'` (likely a wildcard for all companies or a default selection).

3. **Sort Operation**:
   - `// LOAD #GSORT`:
     - Loads the `#GSORT` utility, a system program for sorting data files.
   - **File Definitions for Sort**:
     - `// FILE NAME-INPUT,LABEL-?9?BBBOL,DISP-SHR`:
       - Input file is `BBBOL` (Bill of Lading file), with label `?9?BBBOL` (where `?9?` is a system-specific prefix or library) and opened in shared mode (`DISP-SHR`).
     - `// FILE NAME-OUTPUT,LABEL-?9?BB751S,RECORDS-999000,EXTEND-999000,RETAIN-J`:
       - Output file is `BB751S`, a temporary sorted file with a capacity of 999,000 records, extendable by another 999,000, and retained as a job file (`RETAIN-J`).
   - `// RUN`:
     - Executes the `#GSORT` utility with the following sort specifications:
       - `HSORTR 11A 3X 512 N`:
         - Defines a sort operation with a header (`H`), sorting records (`SORTR`), 11 ascending keys (`11A`), 3 extension records (`3X`), 512-byte record length, and no summary (`N`).
       - Sort Keys (Multiple Lines: `?L'1,3'? 4 9NEC?L'106,6'?`, etc.):
         - Sorts on multiple fields, each starting with a 3-byte field (`?L'1,3'?`) and a 6-byte field from positions 106, 112, 118, 124, 130, 136, 142, 148, 154, and 160.
         - `4 9NEC` indicates sorting on positions 4-9 (likely order number) with no equal comparison (`NEC`).
       - Inclusion Criteria:
         - `I C 1 1NECD`: Includes records where position 1 is not equal to `'D'` (likely excluding deleted records, consistent with `BCDEL` in `BICONT`).
         - `IAC 2 3EQC?L'101,2'?`: Includes records where positions 2-3 (likely company code) equal the 2-byte parameter at position 101.
         - `IAC 10 12EQC000`: Includes records where positions 10-12 (sequence number) equal `'000'` (specific to the summary report, likely selecting header or primary records).
       - Field Specifications for Output:
         - `FNC 2 3 COMPANY`: Maps positions 2-3 to a field named `COMPANY`.
         - `FNC 4 9 ORDER`: Maps positions 4-9 to a field named `ORDER`.
         - `FNC 10 12 SEQUENCE`: Maps positions 10-12 to a field named `SEQUENCE`.
         - `FDC 1 512`: Copies the entire 512-byte record from position 1.
   - `// END`: Completes the sort operation, producing the sorted file `BB751S`.

4. **Report Generation**:
   - `// LOAD BB751`:
     - Loads the `BB751` program (likely an RPG or CL program) to generate the summary BOL listing.
   - **File Definitions for Report**:
     - `// FILE NAME-BBBOL,LABEL-?9?BB751S,DISP-SHR`:
       - Uses the sorted file `BB751S` (output from `#GSORT`) as the input BOL file, renamed as `BBBOL`.
     - `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`:
       - Customer file (`ARCUST`) for customer-related data, opened in shared mode.
     - `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHR`:
       - Inventory control file (`BICONT`) for railcar or company data, opened in shared mode.
     - `// FILE NAME-GSCONT,LABEL-?9?GSCONT,DISP-SHR`:
       - Control file (`GSCONT`) for system or configuration data, opened in shared mode.
   - `// RUN`:
     - Executes the `BB751` program to generate the summary BOL listing using the sorted `BB751S` file and other reference files.

---

### **Business Rules**

1. **Selection Criteria**:
   - If the parameter at position 103 equals `'SEL'`, the program uses `'O COAC'` for company selection (specific company or account).
   - Otherwise, it uses `'O*CO*C'` (likely a wildcard for all companies or a default selection).

2. **Sort Logic**:
   - Excludes records where position 1 equals `'D'` (likely a delete flag, consistent with `BCDEL` in `BICONT`).
   - Includes records where the company code (positions 2-3) matches the parameter at position 101.
   - Includes only records where the sequence number (positions 10-12) equals `'000'`, which likely selects header or primary records for the summary report (a key difference from `BB750`, which does not have this restriction).
   - Sorts primarily by order number (positions 4-9) and additional fields (positions 106, 112, etc., likely order line or sequence numbers) in ascending order.
   - Outputs fields for company, order, sequence, and the full 512-byte record.

3. **File Usage**:
   - The input BOL data (`BBBOL`) is sorted into `BB751S` for ordered processing.
   - The report program (`BB751`) uses `BB751S`, `ARCUST`, `BICONT`, and `GSCONT` to generate the summary report, likely omitting detailed line items (unlike `BB750`).

4. **Data Retention**:
   - The sorted file `BB751S` is retained as a job file (`RETAIN-J`), ensuring it’s available for the duration of the job but not permanently.

5. **Report Generation**:
   - The summary report is generated when the `KYDETL` flag (from `BB750P.rpg36.txt`) is not `'Y'`, as determined by the calling OCL (`BB750P.ocl36.txt`).
   - The report likely includes summarized order information (e.g., order headers) and customer details, based on the files accessed and the sequence filter (`'000'`).

6. **Differences from Detailed Report (BB750)**:
   - Unlike `BB750`, which includes all sequence numbers, `BB751` restricts records to those with sequence number `'000'`, indicating a focus on summary-level data (e.g., order headers rather than line items).
   - `BB751` uses fewer files (excludes `SHIPTO` and `GSCTUM`), suggesting less detailed output.

---

### **Tables/Files Used**

1. **BBBOL**:
   - Input file for the sort operation, containing raw BOL data.
   - Label: `?9?BBBOL`, opened in shared mode (`DISP-SHR`).
   - Used by `#GSORT` as the input file and by `BB751` as the sorted file (`BB751S`).

2. **BB751S**:
   - Temporary output file from the sort operation, containing sorted BOL data.
   - Label: `?9?BB751S`, with a capacity of 999,000 records, extendable, and retained as a job file (`RETAIN-J`).
   - Used as the input (`BBBOL`) for the `BB751` program.

3. **ARCUST**:
   - Customer file, likely containing customer master data (e.g., names, addresses).
   - Label: `?9?ARCUST`, opened in shared mode.
   - Used by `BB751` for customer-related information in the report.

4. **BICONT**:
   - Inventory control file, containing railcar or company data.
   - Label: `?9?BICONT`, opened in shared mode.
   - Used by `BB751` for validation or additional railcar/company details (consistent with its use in `BB750P.rpg36.txt` and `BB750.ocl36.txt`).

5. **GSCONT**:
   - Control file, likely containing system or configuration data.
   - Label: `?9?GSCONT`, opened in shared mode.
   - Used by `BB751` for control or reference data.

---

### **External Programs Called**

1. **#GSORT**:
   - A system utility program for sorting data files.
   - Used to sort the `BBBOL` file into `BB751S` based on company, order, and sequence fields (with sequence `'000'` restriction).
   - Called in the first part of the OCL program.

2. **BB751**:
   - The main program for generating the summary BOL listing.
   - Likely an RPG or CL program that processes the sorted `BB751S` file and other reference files to produce the report.
   - Called in the second part of the OCL program.

---

### **Summary**

- **Purpose**: The `BB751` OCL program generates a summary Open Railcar BOL Listing by Order. It sorts the BOL data (`BBBOL`) using `#GSORT` into a temporary file (`BB751S`), filtering for sequence `'000'`, and then runs the `BB751` program to produce the summary report using the sorted data and reference files.
- **Process Flow**:
  1. Sets up company selection criteria (`'O COAC'` or `'O*CO*C'`) based on a parameter.
  2. Sorts the `BBBOL` file by company, order, and sequence (restricting to sequence `'000'`), excluding deleted records, into `BB751S`.
  3. Executes `BB751` to generate the summary report using `BB751S`, `ARCUST`, `BICONT`, and `GSCONT`.
- **Business Rules**:
  - Filters out deleted records and selects records by company code and sequence `'000'` (for summary-level data).
  - Sorts data by order number and additional sequence fields.
  - Uses fewer files than `BB750` to produce a summarized report (e.g., order headers without line items).
- **Files Used**: `BBBOL` (input), `BB751S` (sorted output), `ARCUST` (customer), `BICONT` (inventory), `GSCONT` (control).
- **External Programs**: `#GSORT` (sort utility) and `BB751` (report generator).

If you have the RPG code for `BB751` or file layouts for `BBBOL`, `ARCUST`, etc., I can provide further details on the report’s content or processing logic!