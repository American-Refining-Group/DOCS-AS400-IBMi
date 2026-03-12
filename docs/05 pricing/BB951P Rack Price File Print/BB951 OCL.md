The provided document, `BB951.ocl36.txt`, is an **Operation Control Language (OCL)** program for IBM System/36, designed to print the latest price of each product on the rack price file. It works in conjunction with the previously analyzed `BB951P.ocl36.txt` and `BB951P.rpg36.txt` programs, orchestrating a series of file operations, sorts, and program executions to generate a rack price report. Below, I’ll explain the **process steps**, **external programs called**, and **tables/files used**.

---

### Process Steps of the OCL Program

The OCL program automates the process of generating a rack price report by validating inputs, sorting and filtering data, building temporary files, and invoking programs to process and print the report. Here’s a step-by-step breakdown:

1. **Initial Checks for Concurrent Execution**:
   - `// IF ACTIVE-BB951 ...`: Checks if the `BB951` program is already running on another workstation.
     - If active, displays messages: "RACK PRICE LIST IS BEING RUN BY ANOTHER WORKSTATION", "PLEASE TRY AGAIN LATER...", and a blank line.
     - Prompts the user with `PAUSE 'REPLY 0 (JOB WILL CANCEL)'`, and if confirmed, executes `RETURN` to cancel the job, preventing concurrent execution.

2. **Set Local Variables for Selection Criteria**:
   - A series of `// IFF` statements set local variables based on input parameters (likely passed from `BB951P.rpg36.txt`):
     - `?L'102,3'?` (location from, `KYLOFR`): Sets offset 10 to `'IAC'` (include all) if blank, else `'I*C'` (include specific).
     - `?L'108,4'?` (product from, `KYPRFR`): Sets offset 13 to `'IAC'` if blank, else `'I*C'`.
     - `?L'116,3'?` (container from, `KYCNFR`): Sets offset 16 to `'IAC'` if blank, else `'I*C'`.
     - `?L'125,6'?` (from date, `KYFRD8`): Sets offset 19 to `'IAC'` if blank, else `'I*C'`.
     - `?L'500,2'?` (unknown, possibly a flag): Sets offset 22 to `'IAC'` if blank, else `'I*C'`.
   - These settings determine whether to include all records or specific ranges for location, product, container, and date in the subsequent sort.

3. **Clear Temporary Files**:
   - `GSDELETE BB951X,BB951P,BB9511,BB951S,BB9518,BB951U,BB951T,,?9?`: Deletes temporary files from previous runs to ensure a clean slate.
   - `CLRPFM ?9?BBPRCEC`: Clears the `BBPRCEC` file (rack price exception file) to prepare for new data.

4. **Sort BBPRCE File**:
   - `// LOAD #GSORT`: Loads the system sort utility (`#GSORT`).
   - **Input**: `?9?BBPRCE` (rack price file, shared read/modify).
   - **Output**: `?9?BB951T` (temporary file, 999,000 records, extendable, retained temporarily).
   - **Sort Specifications**:
     - `HSORTR 27A 3X 128 N`: Sorts 27 bytes ascending, excludes 3 bytes, includes 128 bytes, no sequence check.
     - `I C 1 1NECD`: Excludes records where the first byte (`NECD`) is `'D'` (deleted).
     - Conditional includes based on local variables:
       - Location (`KYLOFR`, `KYLOTO`): Includes records where bytes 4–6 match or are within range (`GEC` for greater/equal, `LEC` for less/equal).
       - Product (`KYPRFR`, `KYPRTO`): Includes records where bytes 7–10 match or are within range.
       - Container (`KYCNFR`, `KYCNTO`): Includes records where bytes 11–13 match or are within range.
       - Date (`KYFRD8`, `KYTOD8`): Includes records where bytes 17–24 (date) match or are within range.
     - `FNC 2 28`: Includes fields 2–28 (company, location, product group, product, container, unit of measure).
     - `FDC 1 128`: Includes fields 1–128 (entire record).
   - **Purpose**: Filters and sorts the `BBPRCE` file to create a temporary file (`BB951T`) with relevant records based on user-specified criteria.

5. **Build Index for Temporary File**:
   - `// BLDINDEX ?9?BB951U,2,27,?9?BB951T`: Builds an index for the temporary file `BB951T`, creating `BB951U` with a key length of 27 bytes starting at position 2, enabling efficient access for subsequent processing.

6. **Build Temporary File for Rack Price Processing**:
   - `// LOAD BB9511`: Loads the `BB9511` program (likely an RPG or CL program).
   - **Files**:
     - `BBPRCE` (input, labeled `?9?BB951U`, shared read/modify).
     - `GSTABL` (table, shared read/modify).
     - `BB9511` (output, temporary file, 999,000 records, extendable by 1,000).
     - `GSPROD` (product file, shared read/modify).
   - **Purpose**: Processes the sorted/indexed `BB951U` file, possibly enriching it with data from `GSTABL` and `GSPROD`, to create another temporary file (`BB9511`) for further processing.

7. **Sort Temporary File for Rack Price Processing**:
   - `// LOAD #GSORT`: Loads the sort utility again.
   - **Input**: `?9?BB9511` (temporary file from previous step, shared read/modify).
   - **Output**: `?9?BB951S` (temporary file, 999,000 records, extendable, retained temporarily).
   - **Sort Specifications**:
     - `HSORTR 17A 3X 166 N`: Sorts 17 bytes ascending, excludes 3 bytes, includes 166 bytes, no sequence check.
     - `I C 1 1NECD`: Excludes deleted records (`NECD = 'D'`).
     - `?L'22,3'? 165 166EQC?L'500,2'?`: Includes records where bytes 165–166 equal the value at local offset 500 (unknown flag).
     - `FNC 1 17`: Includes fields 1–17 (company, location, product group, product, container, unit of measure).
     - `FDC 1 166`: Includes fields 1–166 (entire record).
   - **Purpose**: Further sorts the `BB9511` file to create `BB951S`, organizing data for the final report.

8. **Build Temporary Tax Print File**:
   - `// LOAD BB9518`: Loads the `BB9518` program (likely an RPG or CL program).
   - **Files**:
     - `GSPRD4` (product file, shared read/modify).
     - `GSTABL` (table, shared read/modify).
     - `BIPRT1` (print file, shared read/modify).
     - `BISLTX` (sales tax file, shared read/modify).
     - `ICSUMHY` (summary history file, shared read-only).
     - `BB9518` (output, temporary file, 999,000 records, extendable).
   - **Purpose**: Processes data to create a temporary tax print file (`BB9518`), likely incorporating tax-related information for the report.

9. **Print Rack Price Report**:
   - `// LOAD BB951`: Loads the `BB951` program (likely an RPG or CL program, possibly the same as `BB951P.rpg36.txt`).
   - **Files**:
     - `BB9511` (input, labeled `?9?BB951S`).
     - `BICONT` (company file, shared read/modify).
     - `INLOC` (location file, shared read/modify).
     - `GSTABL` (table, shared read/modify).
     - `GLCONT` (general ledger control file, shared read/modify).
     - `ICSUMHY` (summary history file, shared read-only).
     - `GSCNTR1` (container file, shared read/modify).
     - `BBPRCEC` (rack price exception file, shared read-only).
     - `BB9518` (tax print file, 10-block input).
   - **Purpose**: Generates the final rack price report, using the sorted and processed data from `BB951S` and additional files for company, location, container, tax, and historical data.

10. **Clean Up Temporary Files**:
    - `// GSDELETE BB951X,BB951P,BB9511,BB951S,BB9518,BB951U,BB951T,,?9?`: Deletes all temporary files created during the process to free up resources.

---

### External Programs Called

The OCL program explicitly calls the following external programs:
1. **#GSORT**: System sort utility, used twice to sort the `BBPRCE` and `BB9511` files into temporary files `BB951T` and `BB951S`, respectively.
2. **BB9511**: Program to build a temporary file for rack price processing, likely an RPG or CL program.
3. **BB9518**: Program to build a temporary tax print file, likely an RPG or CL program.
4. **BB951**: Main program to print the rack price report, likely an RPG or CL program (possibly the same as `BB951P.rpg36.txt`).

---

### Tables/Files Used

The OCL program references the following files (some are tables, others are data or temporary files):
1. **BBPRCE** (`?9?BBPRCE`): Rack price file, input for sorting.
2. **BB951T** (`?9?BB951T`): Temporary file, output from the first sort.
3. **BB951U** (`?9?BB951U`): Indexed temporary file, built from `BB951T`.
4. **BB9511** (`?9?BB9511`): Temporary file, output from `BB9511` program.
5. **BB951S** (`?9?BB951S`): Temporary file, output from the second sort.
6. **BB9518** (`?9?BB9518`): Temporary tax print file, output from `BB9518` program.
7. **BBPRCEC** (`?9?BBPRCEC`): Rack price exception file, cleared and used in the final report.
8. **BICONT** (`?9?BICONT`): Company control file, used for company data.
9. **INLOC** (`?9?INLOC`): Location file, used for location data.
10. **GSTABL** (`?9?GSTABL`): Table file, used for reference data (e.g., tax rates, product groups).
11. **GSPROD** (`?9?GSPROD`): Product file, used for product data.
12. **GSPRD4** (`?9?GSPRD4`): Additional product file, used in tax processing.
13. **BIPRT1** (`?9?BIPRT1`): Print file, used in tax processing.
14. **BISLTX** (`?9?BISLTX`): Sales tax file, used in tax processing.
15. **ICSUMHY** (`?9?ICSUMHY`): Summary history file, used in tax and final report processing.
16. **GLCONT** (`?9?GLCONT`): General ledger control file, used in the final report.
17. **GSCNTR1** (`?9?GSCNTR1`): Container file, used for container data.

**Note**: The `?9?` prefix indicates a library or version placeholder, resolved at runtime.

---

### Summary

- **Process Overview**: The `BB951.ocl36.txt` OCL program orchestrates the generation of a rack price report by:
  - Checking for concurrent execution to prevent conflicts.
  - Setting selection criteria based on user input (location, product, container, date).
  - Clearing and creating temporary files.
  - Sorting and indexing the rack price file (`BBPRCE`).
  - Building temporary files (`BB9511`, `BB951S`, `BB9518`) with processed data.
  - Generating the final report via the `BB951` program.
  - Cleaning up temporary files.
- **External Programs**: `#GSORT`, `BB9511`, `BB9518`, `BB951`.
- **Files/Tables**: `BBPRCE`, `BB951T`, `BB951U`, `BB9511`, `BB951S`, `BB9518`, `BBPRCEC`, `BICONT`, `INLOC`, `GSTABL`, `GSPROD`, `GSPRD4`, `BIPRT1`, `BISLTX`, `ICSUMHY`, `GLCONT`, `GSCNTR1`.

If you need further details (e.g., specifics of `BB951`, `BB9511`, or `BB9518` programs, or file structures), please provide them, or I can search for related information if desired!