The provided document, `AR490.ocl36.txt`, is an Operation Control Language (OCL) program used on IBM midrange systems (e.g., System/36 or AS/400) to control job execution and invoke an RPG program for generating a credit limit grouping list. This OCL program is likely called by the `AR490P.rpgle.txt` program or the earlier `AR490P.ocl36.txt` based on user input (e.g., `KYJOBQ` or conditional logic). Below, I explain the process steps, list the external programs called, and identify the tables/files used.

---

### Process Steps of the OCL Program

The OCL program orchestrates the sorting of data and the execution of the `AR490` RPG program to produce a credit limit grouping list. The steps are as follows:

1. **Invoke Year 2000 Utility (`GSY2K`)**:
   - `// GSY2K`:
     - Calls the `GSY2K` utility to handle Year 2000 date conversions or validations, ensuring the program processes dates correctly in a legacy system environment.

2. **Set Local Variable Based on Company Selection**:
   - `// IF ?L'111,3'?/CO LOCAL OFFSET-1,DATA-'IAC'`:
     - Checks the value at location 111 (likely `KYALCO` from `AR490P.rpgle.txt`) for 3 characters equal to `'CO'`.
     - If true, sets a local variable at `OFFSET-1` to `'IAC'`, indicating specific company processing.
   - `// ELSE LOCAL OFFSET-1,DATA-'I*C'`:
     - If false (e.g., `KYALCO = 'ALL'`), sets the variable to `'I*C'`, indicating all companies.
     - This variable likely controls the behavior of the sorting or RPG program.

3. **Load and Run Sort Program (`#GSORT`)**:
   - `// LOAD #GSORT`:
     - Loads the `#GSORT` program, a system utility for sorting data.
   - `// FILE NAME-INPUT,LABEL-?9?ARCLGR,DISP-SHR`:
     - Opens the input file `ARCLGR` (label prefixed with parameter `?9?`, likely a library name) in shared mode (`DISP-SHR`).
     - This file likely contains raw data for credit limit grouping.
   - `// FILE NAME-OUTPUT,LABEL-?9?AR490,RECORDS-999000,EXTEND-999000,RETAIN-J`:
     - Defines the output file `AR490` (label prefixed with `?9?`) with a capacity of 999,000 records, extendable by 999,000, and retained as a job file (`RETAIN-J`).
     - This file will store the sorted data.
   - `// RUN`:
     - Executes the `#GSORT` program with the following sort specifications:
       - `HSORTR 8A 3X 240 N`:
         - Defines a header sort record with an 8-byte ascending key, 3-byte exclusion, 240-byte record length, and no sequence checking (`N`).
       - `I C 1NECD ?L'1,3'? 2 3EQC?L'114,2'?`:
         - Includes records where positions 2-3 equal the value at location 114 (likely `KYCO1`, a company number).
         - `1NECD` indicates no exclusion for deleted records (or a specific condition).
       - `I*` (repeated twice):
         - Additional include conditions for company numbers at locations 116 (`KYCO2`) and 118 (`KYCO3`).
       - `FNC 2 3 CO #`:
         - Defines a sort field for company number (positions 2-3, numeric).
       - `FNC 4 9 CUST`:
         - Defines a sort field for customer number (positions 4-9, numeric).
       - `FDC 1 240`:
         - Includes the entire record (positions 1-240) in the output.
     - **Purpose**: Sorts the `ARCLGR` file by company and customer number, filtering by up to three company numbers specified by the user (from `AR490P`).

4. **Load and Run RPG Program (`AR490`)**:
   - `// LOAD AR490`:
     - Loads the RPG program `AR490`, which processes the sorted data to generate the credit limit grouping list.
   - **File Definitions**:
     - `// FILE NAME-ARCLGR,LABEL-?9?AR490`:
       - Opens the sorted file `AR490` (previously created by `#GSORT`) as `ARCLGR`.
     - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`:
       - Opens the `ARCONT` file (company control file) in shared mode.
     - `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`:
       - Opens the `ARCUST` file (customer file) in shared mode.
     - `// FILE NAME-GSTABL,LABEL-?9?GSTABL,DISP-SHR`:
       - Opens the `GSTABL` file (likely a general table or configuration file) in shared mode.
   - `// RUN`:
     - Executes the `AR490` program, which uses the sorted `ARCLGR` file and references `ARCONT`, `ARCUST`, and `GSTABL` to produce the final report.

---

### External Programs Called

The OCL program explicitly calls or references the following external programs:
1. **GSY2K**:
   - Invoked via `// GSY2K`.
   - A Year 2000 utility for date handling.
2. **#GSORT**:
   - Loaded via `// LOAD #GSORT`.
   - A system sort utility that sorts the `ARCLGR` file by company and customer number.
3. **AR490**:
   - Loaded via `// LOAD AR490`.
   - The main RPG program that processes the sorted data to generate the credit limit grouping list.

---

### Tables/Files Used

The OCL program references the following files:
1. **ARCLGR**:
   - Input file for `#GSORT` (label `?9?ARCLGR`, shared mode).
   - Also used as the input file `ARCLGR` (label `?9?AR490`) for the `AR490` program after sorting.
   - Likely contains raw credit limit grouping data (e.g., company and customer numbers).
2. **AR490**:
   - Output file from `#GSORT` (label `?9?AR490`).
   - Becomes the sorted input file for the `AR490` program.
3. **ARCONT**:
   - Used by `AR490` (label `?9?ARCONT`, shared mode).
   - Contains company data (e.g., company number, name).
4. **ARCUST**:
   - Used by `AR490` (label `?9?ARCUST`, shared mode).
   - Contains customer data (e.g., customer number, details).
5. **GSTABL**:
   - Used by `AR490` (label `?9?GSTABL`, shared mode).
   - Likely a general table file containing configuration or reference data.

---

### Summary

- **Process Steps**: The OCL program initializes the environment with `GSY2K`, sets a local variable based on company selection (`'CO'` or `'ALL'`), sorts the `ARCLGR` file by company and customer number using `#GSORT` (filtered by user-specified companies), and runs the `AR490` RPG program to generate the credit limit grouping list using the sorted `AR490` file and supporting files (`ARCONT`, `ARCUST`, `GSTABL`).
- **External Programs**: `GSY2K` (date utility), `#GSORT` (sort utility), `AR490` (main RPG program).
- **Tables/Files**: `ARCLGR` (input/output for sorting), `AR490` (sorted output/input), `ARCONT` (company data), `ARCUST` (customer data), `GSTABL` (configuration data).

If you need further analysis (e.g., details of the `AR490` RPG program or the structure of the files), please provide the RPG source code for `AR490` or additional file definitions.