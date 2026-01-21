The provided document, `BI921.ocl36.txt`, is an Operation Control Language (OCL) program for an IBM System/36 or AS/400 environment, designed to manage the processing of a "Product Tax Master List." It is related to the previously analyzed `BI921P.ocl36.txt` and `BI921P.rpg36.txt`, as it continues the workflow by sorting data and executing a report generation program. Below, I’ll explain the process steps, identify external programs called, and list the tables/files used.

### Process Steps of the OCL Program

The OCL program `BI921.ocl36.txt` orchestrates the sorting of tax-related data and the execution of a report generation program. Here’s a step-by-step breakdown of the process :

1. **Header and Metadata**:
   - The file starts with a comment: `** PRODUCT TAX MASTER LIST **`, indicating its purpose, which is to generate or process a product tax master list.
   - The comment `// GSY2K` suggests a system check or configuration for Year 2000 (Y2K) compliance, ensuring proper handling of date-related data.

2. **Set Local Data Based on Condition**:
   - `// IF ?L'111,3'?/CO LOCAL OFFSET-1,DATA-'IAC'`
     - Checks if the value at position 111, length 3 (likely from a user data structure or parameter) equals 'CO'.
     - If true, sets a local variable at offset 1 to 'IAC' (possibly a company-specific code or flag).
   - `// ELSE LOCAL OFFSET-1,DATA-'I*C'`
     - If the condition is false, sets the local variable at offset 1 to 'I*C' (likely a wildcard or default code).
   - This step customizes the job based on whether specific companies ('CO') or all companies are selected.

3. **Sort Data Using #GSORT**:
   - `// LOAD #GSORT`: Loads the system sort utility `#GSORT`, a standard IBM System/36 program for sorting data files.
   - `// FILE NAME-INPUT,LABEL-?9?BIPRTX,DISP-SHR`:
     - Specifies the input file `BIPRTX` with a label prefixed by `?9?` (a dynamic parameter, e.g., library or job number).
     - `DISP-SHR` indicates shared access, allowing concurrent reads.
   - `// FILE NAME-OUTPUT,LABEL-?9?BI921,RECORDS-999000,EXTEND-999000,RETAIN-J`:
     - Specifies the output file `BI921` with the same `?9?` prefix.
     - `RECORDS-999000,EXTEND-999000` allocates space for up to 999,000 records, with the ability to extend by the same amount.
     - `RETAIN-J` retains the output file after job completion for further processing.
   - `// RUN`:
     - Executes the sort operation with the following specifications (`HSORTR` and subsequent lines):
       - `HSORTR 9A 3X 64 N`: Defines the sort header:
         - `9A`: 9-character alphanumeric key.
         - `3X`: 3 fields to sort.
         - `64`: Record length of 64 bytes.
         - `N`: No sequence checking (ascending order by default).
       - **Conditional Input Specifications**:
         - Three conditional checks (`I*` blocks) filter records based on company numbers:
           - `I C 1NECD ?L'1,3'? 2 3EQC?L'114,2'?`: Includes records where the company number (positions 2-3) matches the value at position 114, length 2 (likely `KYCO1` from `BI921P`).
           - `IOC 1NECD ?L'1,3'? 2 3EQC?L'116,2'?`: Includes records where the company number matches the value at position 116, length 2 (`KYCO2`).
           - `IOC 1NECD ?L'1,3'? 2 3EQC?L'118,2'?`: Includes records where the company number matches the value at position 118, length 2 (`KYCO3`).
         - These conditions filter `BIPRTX` records based on user-selected company numbers (if `KYALCO` = 'CO') or include all records if no specific companies are selected.
       - **Sort Fields**:
         - `FNC 2 3 CO #`: Sorts by company number (positions 2-3).
         - `FNC 4 5 STATE`: Sorts by state code (positions 4-5).
         - `FNC 6 9 PRODUCE CODE`: Sorts by produce code (positions 6-9).
         - `FNC 10 10 DELVRD CODE`: Sorts by delivered code (position 10).
       - `FDC 1 64`: Includes the entire record (positions 1-64) in the output.
   - **// END**: Completes the sort operation, producing a sorted file `?9?BI921`.

4. **Execute Report Generation Program**:
   - `// LOAD BI921`: Loads the `BI921` program, likely an RPG program responsible for generating the final product tax master list report.
   - **File Specifications**:
     - `// FILE NAME-BIPRTX,LABEL-?9?BI921`: Uses the sorted output file `?9?BI921` (from the previous step) as input, referred to as `BIPRTX`.
     - `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHRRM`: Uses the `BICONT` file (shared read mode) for company data (as seen in `BI921P`).
     - `** FILE NAME-GSTABL,LABEL-?9?GSTABL,DISP-SHRRM`: Specifies the `GSTABL` file (shared read mode), likely containing tax table data. The double asterisk (`**`) suggests this line might be commented out or conditionally included.
     - `** FILE NAME-GSPROD,LABEL-?9?GSPROD,DISP-SHRRM`: Specifies the `GSPROD` file (shared read mode), likely containing product data. This line is also commented out or conditional.
   - `// RUN`: Executes the `BI921` program, which processes the sorted `BIPRTX` file and other files to generate the report.

### External Programs Called

The OCL program explicitly calls the following external programs:
1. **#GSORT**: A system sort utility used to sort the `BIPRTX` file based on company number, state, produce code, and delivered code.
2. **BI921**: An RPG (or similar) program loaded and executed to generate the product tax master list report using the sorted `BI921` file and other data files.

### Tables/Files Used

The OCL program references the following files:
1. **BIPRTX** (Input to #GSORT):
   - Label: `?9?BIPRTX`.
   - Purpose: Input file containing product tax data to be sorted.
   - Access: Shared (`DISP-SHR`).
2. **BI921** (Output from #GSORT, Input to BI921):
   - Label: `?9?BI921`.
   - Purpose: Sorted output file from `#GSORT`, used as input for the `BI921` program.
   - Attributes: Up to 999,000 records, extensible, retained after job (`RETAIN-J`).
3. **BICONT**:
   - Label: `?9?BICONT`.
   - Purpose: Contains company data (e.g., company codes and names), used by the `BI921` program.
   - Access: Shared read mode (`DISP-SHRRM`).
4. **GSTABL** (Commented/Conditional):
   - Label: `?9?GSTABL`.
   - Purpose: Likely contains tax table data (e.g., tax rates or rules).
   - Access: Shared read mode (`DISP-SHRRM`).
   - Note: Commented out (`**`), so it may not be used unless uncommented or included conditionally.
5. **GSPROD** (Commented/Conditional):
   - Label: `?9?GSPROD`.
   - Purpose: Likely contains product data (e.g., product codes or descriptions).
   - Access: Shared read mode (`DISP-SHRRM`).
   - Note: Commented out (`**`), so it may not be used unless uncommented or included conditionally.

### Summary

- **Process Steps**: The OCL program sets a local variable based on whether specific companies ('CO') or all companies are selected (`KYALCO`). It then sorts the `BIPRTX` file using `#GSORT` based on company number, state, produce code, and delivered code, producing a sorted file `BI921`. Finally, it executes the `BI921` program, which uses the sorted `BI921` file, `BICONT`, and possibly `GSTABL` and `GSPROD` to generate the product tax master list report.
- **External Programs Called**:
  - `#GSORT`: System sort utility for sorting `BIPRTX`.
  - `BI921`: RPG program for report generation.
- **Tables/Files Used**:
  - `BIPRTX`: Input tax data file.
  - `BI921`: Sorted output file used as input for report generation.
  - `BICONT`: Company data file.
  - `GSTABL` (conditional): Tax table data.
  - `GSPROD` (conditional): Product data.

### Notes

- The `?9?` placeholders indicate dynamic parameters (e.g., library or job number) passed from the calling environment, likely set by the earlier `BI921P` program.
- The commented lines for `GSTABL` and `GSPROD` suggest they may be optional or included based on specific conditions not visible in this OCL.
- The `BI921` program is likely the final step in generating the report, using the sorted data and additional files. If you have the RPG source code for `BI921`, I can provide further details on its logic.
- The process ties back to `BI921P`, which validated user input (company selection, job queue, etc.), and this OCL uses those parameters (e.g., `KYCO1`, `KYCO2`, `KYCO3` at positions 114, 116, 118) to filter and sort data.

If you have additional files (e.g., `BI921` RPG source) or need further analysis of specific components, let me know! I can also search for related information if required.