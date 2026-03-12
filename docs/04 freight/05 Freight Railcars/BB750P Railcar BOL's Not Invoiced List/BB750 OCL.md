The document `BB750.ocl36.txt` is an **Open Control Language (OCL)** program for the IBM System/36 or AS/400, responsible for generating a **Detailed Open Railcar Bill of Lading (BOL) Listing by Order**. It is called from the main OCL program (`BB750P.ocl36.txt`) when a detailed report is requested (based on the `KYDETL` flag set to `'Y'`). Below, I’ll explain the process steps, business rules, tables/files used, and external programs called.

---

### **Process Steps of the OCL Program**

The `BB750` OCL program performs two main tasks: sorting the BOL data and generating the detailed report. It involves a sort operation using the `#GSORT` utility and then executes the `BB750` program to produce the report. Here’s a step-by-step breakdown:

1. **Initialization and Environment Setup**:
   - `// GSY2K`: Likely a system or environment setup command, possibly related to Y2K compliance or system configuration.

2. **Conditional Local Variable Setup**:
   - `// IF ?L'103,3'?/SEL LOCAL OFFSET-1,DATA-'O COAC'`
     - Checks if the parameter at position 103 (length 3) equals `'SEL'`.
     - If true, sets a local variable at offset 1 to `'O COAC'` (likely indicating a specific company or selection criteria).
   - `// ELSE LOCAL OFFSET-1,DATA-'O*CO*C'`
     - If false, sets the local variable to `'O*CO*C'` (likely a wildcard or default company selection).

3. **Sort Operation**:
   - `// LOAD #GSORT`:
     - Loads the `#GSORT` utility, a system program for sorting data files.
   - **File Definitions for Sort**:
     - `// FILE NAME-INPUT,LABEL-?9?BBBOL,DISP-SHR`:
       - Input file is `BBBOL` (Bill of Lading file), with label `?9?BBBOL` (where `?9?` is a system-specific prefix or library) and opened in shared mode (`DISP-SHR`).
     - `// FILE NAME-OUTPUT,LABEL-?9?BB750S,RECORDS-999000,EXTEND-999000,RETAIN-J`:
       - Output file is `BB750S`, a temporary sorted file with a capacity of 999,000 records, extendable by another 999,000, and retained as a job file (`RETAIN-J`).
   - `// RUN`:
     - Executes the `#GSORT` utility with the following sort specifications:
       - `HSORTR 11A 3X 512 N`:
         - Defines a sort operation with a header (`H`), sorting records (`SORTR`), 11 ascending keys (`11A`), 3 extension records (`3X`), 512-byte record length, and no summary (`N`).
       - Sort Keys (Multiple Lines: `?L'1,3'? 4 9NEC?L'106,6'?`, etc.):
         - Sorts on multiple fields, each starting with a 3-byte field (`?L'1,3'?`) and a 6-byte field from positions 106, 112, 118, 124, 130, 136, 142, 148, 154, and 160.
         - `4 9NEC` indicates sorting on positions 4-9 (likely order number) with no equal comparison (`NEC`).
       - Inclusion Criteria:
         - `I C 1 1NECD`: Includes records where position 1 is not equal to `'D'` (likely excluding deleted records, as `BCDEL = 'D'` indicates deletion in `BICONT`).
         - `IAC 2 3EQC?L'101,2'?`: Includes records where positions 2-3 (likely company code) equal the 2-byte parameter at position 101.
       - Field Specifications for Output:
         - `FNC 2 3 COMPANY`: Maps positions 2-3 to a field named `COMPANY`.
         - `FNC 4 9 ORDER`: Maps positions 4-9 to a field named `ORDER`.
         - `FNC 10 12 SEQUENCE`: Maps positions 10-12 to a field named `SEQUENCE`.
         - `FDC 1 512`: Copies the entire 512-byte record from position 1.
   - `// END`: Completes the sort operation, producing the sorted file `BB750S`.

4. **Report Generation**:
   - `// LOAD BB750`:
     - Loads the `BB750` program (likely an RPG or CL program) to generate the detailed BOL listing.
   - **File Definitions for Report**:
     - `// FILE NAME-BBBOL,LABEL-?9?BB750S,DISP-SHR`:
       - Uses the sorted file `BB750S` (output from `#GSORT`) as the input BOL file, renamed as `BBBOL`.
     - `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`:
       - Customer file (`ARCUST`) for customer-related data, opened in shared mode.
     - `// FILE NAME-SHIPTO,LABEL-?9?SHIPTO,DISP-SHR`:
       - Ship-to file (`SHIPTO`) for shipping address data, opened in shared mode.
     - `// FILE NAME-GSCONT,LABEL-?9?GSCONT,DISP-SHR`:
       - Control file (`GSCONT`) for system or configuration data, opened in shared mode.
     - `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHR`:
       - Inventory control file (`BICONT`) for railcar or company data, opened in shared mode.
     - `// FILE NAME-GSCTUM,LABEL-?9?GSCTUM,DISP-SHR`:
       - Unit of measure or control file (`GSCTUM`), opened in shared mode.
   - `// RUN`:
     - Executes the `BB750` program to generate the detailed BOL listing using the sorted `BB750S` file and other reference files.

---

### **Business Rules**

1. **Selection Criteria**:
   - If the parameter at position 103 equals `'SEL'`, the program uses `'O COAC'` for company selection (specific company or account).
   - Otherwise, it uses `'O*CO*C'` (likely a wildcard for all companies or a default selection).

2. **Sort Logic**:
   - Excludes records where position 1 equals `'D'` (likely a delete flag, consistent with `BCDEL` in `BICONT`).
   - Includes records where the company code (positions 2-3) matches the parameter at position 101.
   - Sorts primarily by order number (positions 4-9) and additional fields (positions 106, 112, etc., likely order line or sequence numbers) in ascending order.
   - Outputs fields for company, order, sequence, and the full 512-byte record.

3. **File Usage**:
   - The input BOL data (`BBBOL`) is sorted into `BB750S` for ordered processing.
   - The report program (`BB750`) uses multiple files (`ARCUST`, `SHIPTO`, `GSCONT`, `BICONT`, `GSCTUM`) to enrich the BOL listing with customer, shipping, and control data.

4. **Data Retention**:
   - The sorted file `BB750S` is retained as a job file (`RETAIN-J`), ensuring it’s available for the duration of the job but not permanently.

5. **Report Generation**:
   - The detailed report is generated only if the `KYDETL` flag (from `BB750P.rpg36.txt`) is `'Y'`, as determined by the calling OCL (`BB750P.ocl36.txt`).
   - The report likely includes detailed order information, customer details, and shipping information, based on the files accessed.

---

### **Tables/Files Used**

1. **BBBOL**:
   - Input file for the sort operation, containing raw BOL data.
   - Label: `?9?BBBOL`, opened in shared mode (`DISP-SHR`).
   - Used by `#GSORT` as the input file and by `BB750` as the sorted file (`BB750S`).

2. **BB750S**:
   - Temporary output file from the sort operation, containing sorted BOL data.
   - Label: `?9?BB750S`, with a capacity of 999,000 records, extendable, and retained as a job file (`RETAIN-J`).
   - Used as the input (`BBBOL`) for the `BB750` program.

3. **ARCUST**:
   - Customer file, likely containing customer master data (e.g., names, addresses).
   - Label: `?9?ARCUST`, opened in shared mode.
   - Used by `BB750` for customer-related information in the report.

4. **SHIPTO**:
   - Ship-to file, containing shipping address or destination data.
   - Label: `?9?SHIPTO`, opened in shared mode.
   - Used by `BB750` for shipping details in the report.

5. **GSCONT**:
   - Control file, likely containing system or configuration data.
   - Label: `?9?GSCONT`, opened in shared mode.
   - Used by `BB750` for control or reference data.

6. **BICONT**:
   - Inventory control file, containing railcar or company data.
   - Label: `?9?BICONT`, opened in shared mode.
   - Used by `BB750` for validation or additional railcar details (consistent with its use in `BB750P.rpg36.txt`).

7. **GSCTUM**:
   - Unit of measure or control file, possibly for measurement or formatting data.
   - Label: `?9?GSCTUM`, opened in shared mode.
   - Used by `BB750` for formatting or unit-related data in the report.

---

### **External Programs Called**

1. **#GSORT**:
   - A system utility program for sorting data files.
   - Used to sort the `BBBOL` file into `BB750S` based on company, order, and sequence fields.
   - Called in the first part of the OCL program.

2. **BB750**:
   - The main program for generating the detailed BOL listing.
   - Likely an RPG or CL program that processes the sorted `BB750S` file and other reference files to produce the report.
   - Called in the second part of the OCL program.

---

### **Summary**

- **Purpose**: The `BB750` OCL program generates a detailed Open Railcar BOL Listing by Order. It first sorts the BOL data (`BBBOL`) using `#GSORT` into a temporary file (`BB750S`), then runs the `BB750` program to produce the report using sorted data and additional reference files.
- **Process Flow**:
  1. Sets up company selection criteria (`'O COAC'` or `'O*CO*C'`) based on a parameter.
  2. Sorts the `BBBOL` file by company, order, and sequence, excluding deleted records, into `BB750S`.
  3. Executes `BB750` to generate the detailed report using `BB750S` and reference files (`ARCUST`, `SHIPTO`, `GSCONT`, `BICONT`, `GSCTUM`).
- **Business Rules**:
  - Filters out deleted records and selects records by company code.
  - Sorts data by order number and additional sequence fields.
  - Uses multiple files to enrich the report with customer, shipping, and control data.
- **Files Used**: `BBBOL` (input), `BB750S` (sorted output), `ARCUST` (customer), `SHIPTO` (shipping), `GSCONT` (control), `BICONT` (inventory), `GSCTUM` (unit of measure).
- **External Programs**: `#GSORT` (sort utility) and `BB750` (report generator).

If you have the RPG code for `BB750` or file layouts for `BBBOL`, `ARCUST`, etc., I can provide further details on the report’s content or processing logic!