The provided document, `BI947.ocl36.txt`, is an Operation Control Language (OCL) program for an IBM System/36 or AS/400 environment. It is designed to print a "Freight Table Master File by Carrier ID" and is likely the main processing step called by the previously analyzed `BI947P.ocl36.txt` and `BI947P.rpg36.txt`. Below, I’ll explain the process steps, business rules, external programs called, and tables/files used.

---

### Process Steps of the OCL Program

The OCL program orchestrates the preparation, sorting, and processing of data to generate a freight table report sorted by carrier ID. Here’s a step-by-step breakdown of the process:

1. **Initial File Cleanup**:
   - `IF DATAF1-?9?FRTRAT DELETE ?9?FRTRAT,F1`: Deletes the file `?9?FRTRAT` (likely a temporary or output file) if it exists, where `?9?` is a parameter (e.g., a library name).
   - `IF DATAF1-?9?BICUFRO DELETE ?9?BICUFRO,F1`: Deletes the file `?9?BICUFRO` if it exists.
   - `CLRPFM ?9?FR947XL`: Clears the physical file `?9?FR947XL`, preparing it for new data.

2. **Set Local Variables for Sorting**:
   - `IF ?L'111,3'?/CO LOCAL OFFSET-1,DATA-'IAC'`: If the parameter at position 111 (length 3) equals 'CO', sets a local variable at offset 1 to 'IAC' (likely a sort control value for company-specific processing).
   - `ELSE LOCAL OFFSET-1,DATA-'I*C'`: Otherwise, sets the variable to 'I*C' (for all companies or a different sort mode).
   - `IF ?L'123,3'?/SEL LOCAL OFFSET-4,DATA-'O COAC'`: If the parameter at position 123 (length 3) equals 'SEL', sets a local variable at offset 4 to 'O COAC' (indicating selective carrier processing).
   - `ELSE LOCAL OFFSET-4,DATA-'O*CO*C'`: Otherwise, sets it to 'O*CO*C' (for all carriers).

3. **Sort Input Data**:
   - `LOAD #GSORT`: Loads the system sort utility (`#GSORT`) to sort the input file.
   - `FILE NAME-INPUT,LABEL-?9?BICUFR,DISP-SHR`: Specifies the input file `?9?BICUFR` (shared read mode).
   - `FILE NAME-OUTPUT,LABEL-?9?BI947S,RECORDS-999000,EXTEND-999000,RETAIN-J`: Defines the output file `?9?BI947S` with a capacity of 999,000 records, extensible by 999,000, and retained as a job file.
   - `RUN` with sort specifications:
     - `HSORTR 20A 3X 640 N`: Defines a sort header with a 20-byte ascending sort key, 3 control fields, and a maximum record length of 640 bytes, with no sequence checking (`N`).
     - Sort keys:
       - `?L'4,3'? 16 21NEC?L'126,6'?`: Sorts on a 3-byte field (likely company number) and a 6-byte field (carrier ID) if non-blank.
       - Multiple similar lines check carrier IDs at positions 132, 138, 144, 150, 156, 162, 168, 174, and 180.
     - Inclusion criteria (`I*`):
       - `I C 1NECD`: Includes records where position 1 is not 'D' (not deleted).
       - `?L'1,3'? 2 3EQC?L'114,2'?`, etc.: Includes records matching company numbers at positions 114, 116, or 118.
     - Output control (`O*`):
       - `IOC 1NECD`: Outputs records where position 1 is not 'D'.
       - Matches company numbers as above.
     - Field definitions (`FNC`, `FDC`):
       - `FNC 2 3 COMPANY #`: Company number (positions 2–3).
       - `FNC 16 21 CARRIER ID`: Carrier ID (positions 16–21).
       - `FNC 4 9 CUSTOMER #`: Customer number (positions 4–9).
       - `FNC 10 12 SHIP TO #`: Ship-to number (positions 10–12).
       - `FNC 13 15 LOCATION`: Location (positions 13–15).
       - `FDC 1 256`, `FDC 257 512`, `FDC 513 640`: Copies full record segments (1–256, 257–512, 513–640).
   - `END`: Completes the sort operation.

4. **Run the Report Program**:
   - `LOAD BI947`: Loads the RPG program `BI947` for report generation.
   - File definitions:
     - `BICUFR,LABEL-?9?BI947S`: The sorted file from the previous step.
     - `ARCUST,LABEL-?9?ARCUST,DISP-SHRRM`: Customer master file (shared read mode).
     - `BICONT,LABEL-?9?BICONT,DISP-SHRRM`: Freight table master file.
     - `GSTABL,LABEL-?9?GSTABL,DISP-SHRRM`: General table file.
     - `BBCAID,LABEL-?9?BBCAID,DISP-SHRRM`: Carrier ID file.
     - `SHIPTO,LABEL-?9?SHIPTO,DISP-SHRRM`: Ship-to file.
     - `FR947XL,LABEL-?9?FR947XL,DISP-SHR`: Output file for the report.
     - Additional files for `MBBFR1`:
       - `GSUMCV`, `BBCFSH`, `BBCFSD1`, `BBNDFI2`, `BBRCSC2`: Likely used by `MBBFR1` for additional data (e.g., summary, freight, or customer data).
   - `RUN`: Executes the `BI947` program with the specified files.

5. **Cleanup**:
   - `SWITCH 00000000`: Resets all job switches to zero.
   - `LOCAL BLANK-*ALL`: Clears all local variables, ensuring a clean job exit.

---

### Business Rules

1. **File Preparation**:
   - Temporary/output files (`?9?FRTRAT`, `?9?BICUFRO`, `?9?FR947XL`) are cleared or deleted to ensure fresh data processing.
2. **Sort Criteria**:
   - Records are sorted by company number and carrier ID, with optional filtering by specific company numbers (`KYCO1`, `KYCO2`, `KYCO3`) and carrier IDs.
   - Excludes deleted records (`BCDEL ≠ 'D'`).
   - Supports selective (`SEL`) or all (`ALL`) carrier processing based on `KYALCS`.
3. **Company Selection**:
   - If `KYALCO = 'CO'`, processes specific companies; otherwise, processes all companies.
4. **Output File**:
   - The sorted data is written to `?9?BI947S`, which is used as input for the `BI947` program.
5. **Report Generation**:
   - The `BI947` program generates the final report, using multiple files for cross-referencing (e.g., customer, carrier, ship-to data).

---

### External Programs Called

1. **#GSORT**: The system sort utility, used to sort the `?9?BICUFR` file into `?9?BI947S` based on company number, carrier ID, and other fields.
2. **BI947**: The main RPG program that processes the sorted file (`?9?BI947S`) and generates the freight table report.
3. **MBBFR1** (implied): Not explicitly called in the OCL but referenced via file definitions, suggesting it may be invoked by `BI947` for additional processing (e.g., freight calculations or summaries).

---

### Tables/Files Used

1. **Input Files**:
   - `?9?BICUFR`: Input freight table file for sorting (shared read).
   - `?9?ARCUST`: Customer master file (shared read).
   - `?9?BICONT`: Freight table master file (shared read).
   - `?9?GSTABL`: General table file (shared read).
   - `?9?BBCAID`: Carrier ID file (shared read).
   - `?9?SHIPTO`: Ship-to file (shared read).
   - `?9?GSUMCV`, `?9?BBCFSH`, `?9?BBCFSD1`, `?9?BBNDFI2`, `?9?BBRCSC2`: Additional files for `MBBFR1` (shared read).

2. **Output Files**:
   - `?9?BI947S`: Sorted output file from `#GSORT` (999,000 records, job-retained).
   - `?9?FR947XL`: Output file for the report (shared).
   - `?9?FRTRAT`, `?9?BICUFRO`: Temporary files, deleted if they exist.

---

### Summary

- **Process Steps**:
  1. Delete/clear temporary files (`?9?FRTRAT`, `?9?BICUFRO`, `?9?FR947XL`).
  2. Set local variables for sort control based on `KYALCO` and `KYALCS`.
  3. Sort `?9?BICUFR` into `?9?BI947S` using `#GSORT`, filtering out deleted records and selecting by company/carrier.
  4. Run `BI947` with sorted and master files to generate the report.
  5. Reset switches and clear variables.
- **Business Rules**:
  - Clear temporary files for fresh processing.
  - Sort by company number and carrier ID, excluding deleted records.
  - Support selective or all company/carrier processing.
  - Generate a report using multiple cross-referenced files.
- **External Programs**:
  - `#GSORT` (sort utility).
  - `BI947` (report generator).
  - `MBBFR1` (implied, likely called by `BI947`).
- **Tables/Files**:
  - Input: `BICUFR`, `ARCUST`, `BICONT`, `GSTABL`, `BBCAID`, `SHIPTO`, `GSUMCV`, `BBCFSH`, `BBCFSD1`, `BBNDFI2`, `BBRCSC2`.
  - Output: `BI947S`, `FR947XL`, `FRTRAT`, `BICUFRO`.

This OCL program coordinates the sorting and reporting process, leveraging `BI947P` for input validation and `BI947` for final report generation. If you have the `BI947` RPG code or additional details, I can provide further insights!