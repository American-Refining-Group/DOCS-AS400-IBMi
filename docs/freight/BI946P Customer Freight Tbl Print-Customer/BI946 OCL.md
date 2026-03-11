The provided document is an **OCL (Operation Control Language) program** named `BI946.ocl36.txt`, used on IBM midrange systems (e.g., AS/400 or IBM i) to control the execution of a job for printing a freight table master file. This OCL program is likely called after the `BI946P` RPG program validates user inputs, as referenced in the earlier context. Below, I’ll explain the process steps, business rules (where applicable), external programs called, and tables/files used.

---

### Process Steps of the OCL Program

The OCL program `BI946.ocl36.txt` orchestrates the process of summarizing sales gallons, sorting data, and generating a freight table report. Here’s a step-by-step breakdown of the process:

1. **Header and Initial Setup**:
   - The header `** PRINT FREIGHT TABLE MASTER FILE` indicates the purpose of the program: to generate a report for the freight table master file.
   - `// GSY2K`: A directive or comment, likely related to Year 2000 compliance or a system-specific configuration, ensuring proper date handling.

2. **Call to BI945 Program**:
   - `// BI945 ,,,,,,,,?9?`: Calls the program `BI945` with a parameter `?9?` (a dynamic placeholder, likely a library or prefix). The comment `** PRICING - SUMMARIZE SALES GALLONS` suggests `BI945` processes sales data, possibly aggregating gallon totals for freight calculations.

3. **Set Local Variables Based on Parameters**:
   - `// IF ?L'111,3'?/CO LOCAL OFFSET-1,DATA-'IAC'`:
     - Checks if the parameter at position 111 (3 bytes) equals `'CO'`.
     - If true, sets a local variable at offset 1 to `'IAC'` (likely a company-specific flag or code).
   - `// ELSE LOCAL OFFSET-1,DATA-'I*C'`:
     - If false, sets the variable to `'I*C'` (a wildcard or default code).
   - `// IF ?L'123,3'?/SEL LOCAL OFFSET-4,DATA-'O COAC'`:
     - Checks if the parameter at position 123 (3 bytes) equals `'SEL'`.
     - If true, sets a local variable at offset 4 to `'O COAC'` (possibly an output control code for selective customer processing).
   - `// ELSE LOCAL OFFSET-4,DATA-'O*CO*C'`:
     - If false, sets the variable to `'O*CO*C'` (a wildcard or default output code).

4. **Clear Physical Files**:
   - `CLRPFM ?9?FR946XL`: Clears the physical file `FR946XL` (prefixed by `?9?`, a dynamic library or identifier).
   - `CLRPFM ?9?BICUFRC`: Clears the physical file `BICUFRC`. These files are likely temporary or output files used for storing sorted or processed data.

5. **Sort Data Using #GSORT**:
   - `// LOAD #GSORT`: Loads the system sort utility (`#GSORT`) to sort input data.
   - **Input File**:
     - `// FILE NAME-INPUT,LABEL-?9?BICUFR,DISP-SHR`: Specifies `BICUFR` as the input file (shared read mode).
     - A commented line (`** FILE NAME-INPUT,LABEL-?9?FRTREC2,DISP-SHR`) suggests an alternative or deprecated input file `FRTREC2`.
   - **Output File**:
     - `// FILE NAME-OUTPUT,LABEL-?9?BI946S,RECORDS-999000,EXTEND-999000,RETAIN-J`: Defines `BI946S` as the output file with a capacity of 999,000 records, extensible by another 999,000, and retained as a job file (`RETAIN-J`).
   - **Run Sort**:
     - `// RUN` executes the sort with the following specifications:
       - `HSORTR 14A 3X 640 N`: Defines a sort with a 14-byte ascending key, 3 times (possibly multiple key fields), up to 640 bytes of data, no sequence checking (`N`).
       - Multiple sort conditions (`?L'4,3'? 4 9NEC?L'126,6'?`, etc.): Sorts records where bytes 4–9 (and other ranges like 126–131, 132–137, etc.) are not equal to values in the parameter list (e.g., customer or company numbers). These are likely filtering out specific records based on user input from `BI946P`.
       - `I C 1NECD`: Excludes records where byte 1 is not equal to `'D'` (deleted records).
       - `?L'1,3'? 2 3EQC?L'114,2'?`, etc.: Includes records where bytes 2–3 match company numbers from parameters (e.g., `KYCO1`, `KYCO2`, `KYCO3` from `BI946P`).
       - Output fields:
         - `FNC 2 3 COMPANY #`: Company number (bytes 2–3).
         - `FNC 4 9 CUSTOMER #`: Customer number (bytes 4–9).
         - `FNC 10 12 LOCATION #`: Location number (bytes 10–12).
         - `FNC 13 15 SHIP TO #`: Ship-to number (bytes 13–15).
         - `FDC 1 256`, `FDC 257 512`, `FDC 513 640`: Copies full data records (up to 640 bytes) to the output file.
   - **End Sort**:
     - `// END`: Completes the sort operation, producing the sorted file `BI946S`.

6. **Load and Run BI946 Program**:
   - `// LOAD BI946`: Loads the program `BI946` (likely an RPG or CL program).
   - **Files Defined**:
     - `BICUFR` (`?9?BI946S`): The sorted file from the previous step, used as input.
     - `ARCUST` (`?9?ARCUST`, shared read mode): Customer master file.
     - `SHIPTO` (`?9?SHIPTO`, shared read mode): Ship-to address file.
     - `BICONT` (`?9?BICONT`, shared read mode): Freight table master file (also used in `BI946P`).
     - `GSCNTR1` (`?9?GSCNTR1`, shared read mode): Likely a control or configuration file.
     - `FR946XL` (`?9?FR946XL`, shared mode): Output or temporary file.
     - `PRICWK` (`?9?PRICWK`, shared mode): Pricing work file.
     - `BICUFRC` (`?9?BICUFRC`, shared mode): Freight calculation file.
     - Commented files (`SA5FJZD`, `GSUMCV`, `BBCFSH`, `BBCFSD1`, `BBNDFI2`, `BBRCSC2`): Likely used by a related program (`MBBFR1`) or deprecated, as they are not active in this run.
   - `// RUN`: Executes `BI946`, which likely generates the final freight table report using the sorted data and referenced files.

7. **Cleanup**:
   - `// SWITCH 00000000`: Resets all job switches to `0`.
   - `// LOCAL BLANK-*ALL`: Clears all local variables to ensure a clean state.

---

### Business Rules

The OCL program enforces the following business rules:
1. **Parameter-Based Filtering**:
   - Company selection (`KYALCO = 'CO'` or not) determines whether the variable at offset 1 is set to `'IAC'` (specific company) or `'I*C'` (all companies).
   - Customer selection (`KYALCS = 'SEL'` or not) determines whether the variable at offset 4 is set to `'O COAC'` (selective customers) or `'O*CO*C'` (all customers).
2. **Data Filtering in Sort**:
   - Excludes deleted records (`1NECD`, byte 1 ≠ `'D'`).
   - Filters records based on company numbers (`KYCO1`, `KYCO2`, `KYCO3`) from parameters.
   - Filters customer numbers (bytes 4–9) against up to 10 customer numbers from parameters (likely `CS` array from `BI946P`).
3. **File Clearing**:
   - Ensures `FR946XL` and `BICUFRC` are cleared before processing to avoid residual data.
4. **Sorted Output**:
   - The sort produces a file (`BI946S`) with key fields (company, customer, location, ship-to) for structured report generation.

---

### External Programs Called

1. **BI945**:
   - Called with `// BI945 ,,,,,,,,?9?`.
   - Likely an RPG or CL program that summarizes sales gallons for freight calculations, as indicated by the comment `** PRICING - SUMMARIZE SALES GALLONS`.

2. **#GSORT**:
   - System sort utility loaded with `// LOAD #GSORT`.
   - Used to sort the `BICUFR` file based on company, customer, location, and ship-to numbers, producing `BI946S`.

3. **BI946**:
   - Loaded with `// LOAD BI946` and executed with `// RUN`.
   - Likely an RPG or CL program that generates the final freight table report using the sorted data and other files.

---

### Tables/Files Used

1. **Input Files**:
   - `BICUFR` (`?9?BICUFR`, shared read mode): Input file for sorting, containing freight data.
   - `FRTREC2` (commented, `?9?FRTREC2`): Alternative input file, not currently used.
   - `ARCUST` (`?9?ARCUST`, shared read mode): Customer master file.
   - `SHIPTO` (`?9?SHIPTO`, shared read mode): Ship-to address file.
   - `BICONT` (`?9?BICONT`, shared read mode): Freight table master file (also used in `BI946P`).
   - `GSCNTR1` (`?9?GSCNTR1`, shared read mode): Control or configuration file.
   - `PRICWK` (`?9?PRICWK`, shared mode): Pricing work file.
   - `BICUFRC` (`?9?BICUFRC`, shared mode): Freight calculation file.

2. **Output/Temporary Files**:
   - `BI946S` (`?9?BI946S`): Sorted output file from `#GSORT`, with a capacity of 999,000 records.
   - `FR946XL` (`?9?FR946XL`, shared mode): Cleared and used as an output or temporary file.
   - `BICUFRC` (`?9?BICUFRC`, shared mode): Cleared and used for freight calculations.

3. **Commented Files** (not active but referenced):
   - `SA5FJZD`, `GSUMCV`, `BBCFSH`, `BBCFSD1`, `BBNDFI2`, `BBRCSC2`: Likely used by a related program (`MBBFR1`) or deprecated.

---

### Summary

- **Process Steps**: The OCL program calls `BI945` to summarize sales gallons, sets local variables based on parameters (`KYALCO`, `KYALCS`), clears temporary files (`FR946XL`, `BICUFRC`), sorts data from `BICUFR` into `BI946S` using `#GSORT`, runs `BI946` to generate the freight table report, and performs cleanup.
- **Business Rules**: Filters data by company and customer numbers, excludes deleted records, and uses parameters to control company/customer selection and output format.
- **External Programs**: `BI945` (summarizes sales), `#GSORT` (sorts data), `BI946` (generates report).
- **Tables/Files**: `BICUFR`, `ARCUST`, `SHIPTO`, `BICONT`, `GSCNTR1`, `PRICWK`, `BICUFRC` (input), `BI946S`, `FR946XL`, `BICUFRC` (output/temporary).

If you have the source code for `BI945` or `BI946`, or additional details about the file structures, I can provide a more detailed analysis of the end-to-end process.