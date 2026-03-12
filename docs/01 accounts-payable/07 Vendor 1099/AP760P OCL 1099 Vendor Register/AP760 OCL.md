The provided document is an OCL (Operation Control Language) program, `AP760.ocl36.txt`, which is called by the main OCL program `AP760P.ocl36.txt` as part of the vendor 1099 register process on an IBM AS/400 or iSeries system. This OCL program orchestrates the sorting, processing, and reporting of vendor 1099 data. Below, I’ll explain the process steps, list the external programs called, and identify the tables (files) used, based on the provided OCL code and its context within the 1099 register process.

---

### Process Steps of the AP760 OCL Program

The `AP760.ocl36.txt` OCL program is responsible for sorting the 1099 data file, processing it to generate an intermediate file, and then producing the final 1099 register report. It relies on parameters passed from the main OCL (`AP760P.ocl36.txt`) and the validated inputs from the RPG program `AP760P.rpg36.txt`. Here’s a step-by-step breakdown of the process:

1. **Set Year in Local Data Area (LDA)**:
   - The program sets the 1099 year (`?10?`, e.g., `2025`) into the Local Data Area (LDA) at offset 142 with the command `LOCAL OFFSET-142,DATA-'?10?'`. This makes the year available to subsequent programs.

2. **Delete Existing Temporary File**:
   - If the temporary file `?9?AP761` (e.g., `PRODAP761` if `?9?` is `PROD`) exists, it is deleted using `IF DATAF1-?9?AP761 DELETE ?9?AP761,F1`. This ensures a clean slate for the intermediate file created later.

3. **Set Company and Type Selection in LDA**:
   - **Company Selection**:
     - If `?L'111,3'?` (likely `KYALCO` from `AP760P.rpg36.txt`) equals `'CO'`, set LDA offset 1 to `'IAC'` (indicating specific companies).
     - Otherwise, set LDA offset 1 to `'I*C'` (indicating all companies).
   - **Type Selection**:
     - If `?L'135,3'?` (likely `KYALTY` from `AP760P.rpg36.txt`) equals `'TYP'`, set LDA offset 4 to `'IAC'` (indicating specific 1099 types).
     - Otherwise, set LDA offset 4 to `'I*C'` (indicating all 1099 types).
   - These settings control filtering in the subsequent sort operation.

4. **Sort the 1099 File**:
   - **Load Sort Program**: The program loads the system sort utility `#GSORT`.
   - **File Definitions**:
     - Input file: `?13?` (e.g., `APVN2025`, the 1099 data file created by `AP300`).
     - Output file: `?9?AP760S` (e.g., `PRODAP760S`), a temporary sorted file with up to 999,000 records, extendable by 999,000, retained as a job file (`RETAIN-J`).
   - **Sort Specifications**:
     - Sort sequence: Ascending (`HSORTA 22A`).
     - Primary sort field: 1099 type (field at positions 264–264, 1 byte).
     - Secondary sort field: Vendor (field at positions 4–8, 5 bytes).
     - Tertiary sort field: Company (field at positions 2–3, 2 bytes).
   - **Record Selection**:
     - Exclude records marked as deleted (position 1 ≠ `'D'`).
     - Apply conditional filters based on company and type selections:
       - If specific companies (`'CO'`), include records where company (positions 2–3) matches `KYCO1`, `KYCO2`, or `KYCO3` (from `?L'114,2'`, `?L'116,2'`, `?L'118,2'`).
       - If specific types (`'TYP'`), include records where 1099 type (position 264) matches `KYTY1`, `KYTY2`, or `KYTY3` (from `?L'138,1'`, `?L'139,1'`, `?L'140,1'`).
     - If `'ALL'` is selected for companies or types, no specific matching is applied (handled by `I*C` in LDA).
   - **Execution**: The `RUN` command executes the sort, producing the sorted file `?9?AP760S`.

5. **Process Sorted Data**:
   - **Load Program `AP761`**:
     - Input files:
       - `APVEND`: The original 1099 file (`?13?`, e.g., `APVN2025`), shared access.
       - `AP760S`: The sorted file (`?9?AP760S`).
     - Output file: `AP761` (`?9?AP761`), a temporary file with 1,000 records, extendable by 500.
     - **Purpose**: The `AP761` program processes the sorted data to create an intermediate file (`?9?AP761`) for reporting. It likely aggregates or formats the 1099 data based on the sorted input.
   - **Execution**: The `RUN` command executes `AP761`.

6. **Generate 1099 Register Report**:
   - **Load Program `AP760`**:
     - Input files:
       - `AP761`: The intermediate file (`?9?AP761`), shared access.
       - `GSTABL`: Table file (`?9?GSTABL`, e.g., `PRODGSTABL`), shared access, likely used for 1099 type descriptions.
     - **Purpose**: The `AP760` program generates the final 1099 register report, using the processed data from `?9?AP761` and reference data from `GSTABL`.
   - **Execution**: The `RUN` command executes `AP760`.

7. **Clean Up Temporary File**:
   - After processing, if the temporary file `?9?AP761` exists, it is deleted (`IF DATAF1-?9?AP761 DELETE ?9?AP761,F1`) to free up resources.

---

### External Programs Called

The OCL program calls the following external programs:

1. **#GSORT**:
   - System sort utility used to sort the 1099 data file (`?13?`) into a temporary sorted file (`?9?AP760S`).
   - Purpose: Sorts records by 1099 type, vendor, and company, applying filters based on user selections.

2. **AP761**:
   - An RPG or CL program (not provided) that processes the sorted file (`?9?AP760S`) and the original 1099 file (`?13?`) to produce an intermediate file (`?9?AP761`).
   - Purpose: Likely aggregates or formats 1099 data for reporting.

3. **AP760**:
   - An RPG or CL program (not provided, but likely the main report generator) that uses the intermediate file (`?9?AP761`) and table file (`?9?GSTABL`) to produce the final 1099 register report.
   - Purpose: Generates the formatted 1099 register output, possibly a printed report or file.

---

### Tables Used

The OCL program references the following files (tables):

1. **INPUT (labeled ?13?)**:
   - Name: `?13?` (e.g., `APVN2025`, the 1099 data file created by `AP300`).
   - Type: Disk file, shared access (`DISP-SHR`).
   - Purpose: Input to the `#GSORT` program, containing raw vendor 1099 data (e.g., `GAPVEND` before monthly/yearly totals are cleared).

2. **OUTPUT (labeled ?9?AP760S)**:
   - Name: `?9?AP760S` (e.g., `PRODAP760S`).
   - Type: Disk file, temporary, up to 999,000 records, extendable by 999,000, retained as a job file (`RETAIN-J`).
   - Purpose: Sorted output from `#GSORT`, used as input to `AP761`.

3. **APVEND (labeled ?13?)**:
   - Name: `?13?` (e.g., `APVN2025`).
   - Type: Disk file, shared access (`DISP-SHR`).
   - Purpose: Input to `AP761`, providing the original 1099 data for processing.

4. **AP760S (labeled ?9?AP760S)**:
   - Name: `?9?AP760S` (e.g., `PRODAP760S`).
   - Type: Disk file, used as input to `AP761`.
   - Purpose: Contains sorted 1099 data from `#GSORT`.

5. **AP761 (labeled ?9?AP761)**:
   - Name: `?9?AP761` (e.g., `PRODAP761`).
   - Type: Disk file, temporary, 1,000 records, extendable by 500.
   - Purpose: Output from `AP761`, input to `AP760`, containing processed 1099 data for the final report.

6. **GSTABL (labeled ?9?GSTABL)**:
   - Name: `?9?GSTABL` (e.g., `PRODGSTABL`).
   - Type: Disk file, shared access (`DISP-SHR`).
   - Purpose: Reference table used by `AP760`, likely containing 1099 type descriptions or other lookup data.

---

### Integration with Main OCL and RPG Program

The `AP760.ocl36.txt` program is called by `AP760P.ocl36.txt` after the RPG program `AP760P.rpg36.txt` validates user inputs. Here’s how it integrates:

- **Parameters from AP760P**:
  - The main OCL passes parameters `?9?` (library prefix), `?10?` (year), and `?13?` (1099 file name, e.g., `APVN2025`).
  - The RPG program `AP760P` sets fields in the LDA (e.g., `KYALCO`, `KYCO1–3`, `KYALTY`, `KYTY1–3`, `KYCRLS`, `KYJOBQ`, `KYCOPY`, `KYCYYR`) at offsets like 111, 114–118, 135, 138–140, etc., which are referenced in the sort logic (`?L'111,3'?`, `?L'114,2'?`, etc.).

- **Job Queue Execution**:
  - The main OCL (`AP760P.ocl36.txt`) decides whether to run `AP760` (this OCL or its final program) interactively or via a job queue based on `?L'120,1'?` (`KYJOBQ` from `AP760P`).

- **File Usage**:
  - The 1099 file (`?13?`, e.g., `APVN2025`) is created by `AP300` (period-end processing) and used as input.
  - The `GSTABL` file is shared with `AP760P.rpg36.txt` for validating 1099 types and used here for report generation.

- **Cancellation**:
  - If `AP760P.rpg36.txt` sets `KYCANC` to `'CANCEL'` (offset 129–134), the main OCL cancels the job before calling this OCL.

---

### Summary

- **Process Steps**: The OCL program sets the 1099 year in the LDA, deletes any existing temporary file, configures company and type selections, sorts the 1099 data by type, vendor, and company, processes the sorted data into an intermediate file, generates the final 1099 register report, and cleans up the temporary file.
- **External Programs Called**:
  - `#GSORT`: System sort utility for sorting the 1099 file.
  - `AP761`: Processes sorted data into an intermediate file.
  - `AP760`: Generates the final 1099 register report.
- **Tables Used**:
  - `?13?` (e.g., `APVN2025`): Input 1099 data.
  - `?9?AP760S` (e.g., `PRODAP760S`): Sorted output.
  - `APVEND` (same as `?13?`): Input to `AP761`.
  - `AP760S` (same as `?9?AP760S`): Input to `AP761`.
  - `?9?AP761` (e.g., `PRODAP761`): Intermediate file.
  - `?9?GSTABL` (e.g., `PRODGSTABL`): Reference table for 1099 types.

This OCL program orchestrates the backend processing of the 1099 register, transforming raw vendor data into a sorted, processed, and report-ready format, building on the validated inputs from `AP760P.rpg36.txt`.