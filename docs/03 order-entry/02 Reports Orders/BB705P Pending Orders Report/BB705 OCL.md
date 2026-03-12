The provided document, `BB705.ocl36.txt`, is an **Open Control Language (OCL)** program for the IBM System/36, called by the main OCL program `BB705P.ocl36.txt` to generate a **Pending Orders Report**. It orchestrates the processing of order data, sorting, and report generation. Below, I’ll explain the process steps, business rules, tables used, and external programs called, referencing the context from the previous OCL (`BB705P.ocl36.txt`) and RPG (`BB705P.rpg36.txt`) programs for continuity.

### Process Steps of the OCL Program

The `BB705.ocl36.txt` program manages the workflow for generating a pending orders report by location, container, date, and carrier. It involves data preparation, sorting, and report generation. Here’s a step-by-step breakdown:

1. **Initialization and Configuration**:
   - `// GSY2K`: Indicates Year 2000 compliance or a specific system configuration (likely for date handling).
   - `// IF ?L'118,1'?/Y LOCAL OFFSET-1,DATA-'I*CIAC'`: If the parameter at position 118 (likely `KYZEYN` from `BB705P.rpg36.txt`, indicating zero date flag) is 'Y', set a local variable at offset 1 to `'I*CIAC'`.
   - `// ELSE LOCAL OFFSET-1,DATA-'IACI*C'`: If `KYZEYN` is not 'Y', set the local variable to `'IACI*C'`. These values (`I*CIAC` or `IACI*C`) likely control sort or selection criteria later in the process.

2. **Delete Temporary Files**:
   - `// GSDELETE BB705O,BB705S,,,,,,,?9?`: Deletes temporary files `BB705O` and `BB705S` from the library specified by `?9?` to ensure a clean slate for new data.

3. **Load and Run BB7051 (Data Preparation)**:
   - `// LOAD BB7051`: Loads the program `BB7051`.
   - `// FILE NAME-BBORDR,LABEL-?9?BBORDR,DISP-SHR`: Opens the `BBORDR` file (order data) in shared read mode.
   - `// FILE NAME-BB705O,LABEL-?9?BB705O,RETAIN-T,RECORDS-999000,EXTEND-999000`: Defines a temporary output file `BB705O` with a capacity of 999,000 records, marked as temporary (`RETAIN-T`) and extensible.
   - `// RUN`: Executes `BB7051`, which likely reads `BBORDR`, applies filters (e.g., based on `KYCO`, `KYLOC`, `KYFRDT`, `KYTODT`, `KYZEYN` from `BB705P`), and writes filtered order data to `BB705O`.

4. **Load and Run #GSORT (Sorting)**:
   - `// LOAD #GSORT`: Loads the system sort utility `#GSORT`.
   - `// FILE NAME-INPUT,LABEL-?9?BB705O,DISP-SHR`: Uses `BB705O` (output from `BB7051`) as the input file in shared read mode.
   - `// FILE NAME-OUTPUT,LABEL-?9?BB705S,RECORDS-999000,EXTEND-999000,RETAIN-T`: Defines a temporary output file `BB705S` for sorted data, with similar capacity and attributes as `BB705O`.
   - `// RUN`: Executes the sort with the following specifications:
     - **Header**: `HSORTR 24A 3X 532 N`: Defines a sort with a 24-byte header, 3 control fields, a 532-byte record, and no sequence checking (`N`).
     - **Include Conditions**:
       - `I C 1 1NECD`: Include records where position 1 is not equal to 'D' (excludes deleted records).
       - `IAC 2 3EQC?L'101,2'?`: Include records where positions 2-3 equal the company code (`KYCO`) from position 101-102.
       - `?L'4,3'? 516 523EQC00000000`: If the location code (`KYLOC`) from positions 4-6 is non-blank, include records where positions 516-523 (request date) equal zero.
       - `?L'1,3'? 516 523GEC?L'169,8'?`: If the location code is non-blank, include records where the request date (516-523) is greater than or equal to the from date (`FRDT8`, positions 169-176).
       - `?L'1,3'? 516 523LEC?L'177,8'?`: If the location code is non-blank, include records where the request date is less than or equal to the to date (`TODT8`, positions 177-184).
     - **Sort Fields** (ascending, `FNC`):
       - Positions 2-3: Company (`COMPANY`).
       - Positions 513-515: Location temp in header (`LOCATION TEMP IN HEADER`).
       - Positions 516-523: Request date (`REQUEST DATE`).
       - Positions 524-525: Carrier code (`CARRIER CODE`).
       - Positions 4-9: Order number (`ORDER`).
       - Positions 10-12: Sequence (`SEQUENCE`).
     - **Data Fields** (`FDC`):
       - Positions 1-256: Record data.
       - Positions 257-512: Additional record data.
       - Positions 513-532: Header or control data.
   - **Outcome**: Sorts `BB705O` into `BB705S` based on company, location, request date, carrier, order, and sequence, applying filters for valid records and date ranges.

5. **Load and Run BB705 (Report Generation)**:
   - `// LOAD BB705`: Loads the `BB705` program, which generates the final report.
   - **Files**:
     - `BBORDR` (sorted data from `BB705S`, labeled `?9?BB705S`, shared read).
     - `ARCUST` (customer data, shared read).
     - `SHIPTO` (ship-to address data, shared read).
     - `GSCONT` (container data, shared read).
     - `BICONT` (company data, shared read, also used in `BB705P`).
     - `GSCTUM` (possibly unit of measure or control data, shared read).
     - `INLOC` (location data, shared read, also used in `BB705P`).
   - `// RUN`: Executes `BB705`, which uses the sorted data and additional files to produce the pending orders report, formatted by location, container, date, and carrier.

6. **Cleanup**:
   - `// GSDELETE BB705O,BB705S,,,,,,,?9?`: Deletes the temporary files `BB705O` and `BB705S` after report generation to free up disk space.

### Business Rules

The OCL program enforces the following business rules for generating the pending orders report:

1. **Data Filtering**:
   - Exclude records marked as deleted (position 1 ≠ 'D').
   - Filter by company code (positions 2-3 must match `KYCO` from position 101-102).
   - If a location is specified (`KYLOC` non-blank), filter by:
     - Zero request date (if `KYZEYN` = 'Y', positions 516-523 = 00000000).
     - Request date within the specified range (`FRDT8` ≤ date ≤ `TODT8`).

2. **Sort Order**:
   - Sort records by:
     1. Company (positions 2-3).
     2. Location (positions 513-515).
     3. Request date (positions 516-523).
     4. Carrier code (positions 524-525).
     5. Order number (positions 4-9).
     6. Sequence (positions 10-12).
   - This ensures the report is organized hierarchically for easy reading.

3. **Temporary File Management**:
   - Temporary files `BB705O` and `BB705S` are created and deleted within the job to manage intermediate data.
   - Files are allocated with a large capacity (999,000 records) to handle large datasets.

4. **Date Range Handling**:
   - If `KYZEYN` = 'Y', only records with a zero request date are included (for special reporting cases).
   - Otherwise, records must fall within the user-specified date range (`FRDT8` to `TODT8`).

5. **File Access**:
   - All files are opened in shared read mode (`DISP-SHR`) to allow concurrent access by other jobs.
   - The report generation (`BB705`) uses multiple files to enrich order data with customer, ship-to, container, and location details.

### Tables Used

No explicit RPG-style tables (e.g., `TABxxx` or arrays like `COM` in `BB705P.rpg36.txt`) are defined in the OCL program. However, the following files are used as data sources, functioning as tables in the context of the System/36:

1. **BBORDR**: Contains order data, used as input for `BB7051` and sorted input (`BB705S`) for `BB705`.
2. **BB705O**: Temporary file for intermediate data output by `BB7051`.
3. **BB705S**: Temporary file for sorted data output by `#GSORT`.
4. **ARCUST**: Customer data, used by `BB705` for report details.
5. **SHIPTO**: Ship-to address data, used by `BB705`.
6. **GSCONT**: Container data, used by `BB705`.
7. **BICONT**: Company data, used by `BB705` (also used in `BB705P` for validation).
8. **GSCTUM**: Likely unit of measure or control data, used by `BB705`.
9. **INLOC**: Location data, used by `BB705` (also used in `BB705P` for validation).

These files serve as relational tables, providing data for filtering, sorting, and reporting.

### External Programs Called

1. **BB7051**:
   - Purpose: Prepares order data by reading `BBORDR`, applying filters (e.g., company, location, date range), and writing to `BB705O`.
   - Called: In the first `LOAD`/`RUN` block.

2. **#GSORT**:
   - Purpose: System sort utility that sorts `BB705O` into `BB705S` based on specified fields and include conditions.
   - Called: In the second `LOAD`/`RUN` block.

3. **BB705**:
   - Purpose: Generates the final pending orders report using sorted data (`BB705S`) and additional files (`ARCUST`, `SHIPTO`, `GSCONT`, `BICONT`, `GSCTUM`, `INLOC`).
   - Called: In the third `LOAD`/`RUN` block.

4. **GSDELETE**:
   - Purpose: System utility to delete temporary files `BB705O` and `BB705S`.
   - Called: Twice, before and after processing.

### Summary

- **Process Steps**:
  1. Initialize with Y2K settings and set sort criteria based on `KYZEYN`.
  2. Delete temporary files `BB705O` and `BB705S`.
  3. Run `BB7051` to filter order data from `BBORDR` to `BB705O`.
  4. Sort `BB705O` into `BB705S` using `#GSORT` by company, location, date, carrier, order, and sequence.
  5. Run `BB705` to generate the report using sorted data and additional files.
  6. Delete temporary files.

- **Business Rules**:
  - Filter out deleted records and apply company, location, and date range criteria.
  - Sort report hierarchically for clarity.
  - Manage temporary files to optimize disk usage.
  - Support zero-date reporting when `KYZEYN` = 'Y'.
  - Use shared read mode for concurrent file access.

- **Tables Used**:
  - Files: `BBORDR`, `BB705O`, `BB705S`, `ARCUST`, `SHIPTO`, `GSCONT`, `BICONT`, `GSCTUM`, `INLOC` (function as relational tables).

- **External Programs Called**:
  - `BB7051` (data preparation).
  - `#GSORT` (sorting utility).
  - `BB705` (report generation).
  - `GSDELETE` (file cleanup).

This OCL program integrates with `BB705P.rpg36.txt` (for input validation) and relies on `BB7051` and `BB705` to process and report order data. If you have the RPG code for `BB7051` or `BB705`, or file layouts for `BBORDR`, `ARCUST`, etc., I can provide further details. Let me know if you need additional analysis or a search for related information!