### Explanation of the Process Steps and Business Rules in the OCL Program (AP765.ocl36.txt)

The provided OCL (Operation Control Language) program, `AP765.ocl36.txt`, is designed for processing vendor 1099 forms on an IBM System/36 or AS/400 system. It is called by the main OCL program (`AP765P.ocl36.txt`) and handles sorting, data processing, and printing of 1099 forms for a specified year. Below is a detailed explanation of the process steps, business rules, tables/files used, and external programs called.

---

### Process Steps

1. **Program Initialization**:
   - **GSY2K**: Calls a system routine to ensure Year 2000 compliance for date handling.
   - **Parameters**:
     - `?10?`: Represents the four-digit year for the 1099 file (e.g., 2012).
     - `?13?`: Specifies the 1099 file name (e.g., `APVN2012`), which contains vendor data.
   - **Programmer Note**: The file `APVNYYYY` (e.g., `APVN2012`) is created during the period-end process (`AP300`) and is a snapshot of the `GAPVEND` file before monthly and yearly totals are cleared.

2. **Delete Temporary File (if exists)**:
   - `IF DATAF1-?9?AP766 DELETE ?9?AP766,F1`: Checks if the temporary file `?9?AP766` (e.g., `XXAP766` where `?9?` is a prefix) exists. If it does, the file is deleted to ensure a clean start for processing.

3. **Sort Input File**:
   - `LOAD #GSORT`: Loads the system sort utility (`#GSORT`).
   - **Input File**:
     - `FILE NAME-INPUT,LABEL-?13?,DISP-SHR`: Specifies the input file as `?13?` (e.g., `APVN2012`), opened in shared mode (`DISP-SHR`).
   - **Output File**:
     - `FILE NAME-OUTPUT,LABEL-?9?AP765S,RECORDS-999000,EXTEND-999000,RETAIN-J`: Creates a sorted output file named `?9?AP765S` (e.g., `XXAP765S`), with a capacity of 999,000 records, extendable by another 999,000, and retained as a job file (`RETAIN-J`).
   - **Sort Specifications**:
     - `HSORTA 22A 3X N`: Defines an ascending sort (`A`) on a 22-character field starting at position 3, with no sequence checking (`N`).
     - `O C 264EQC`: Includes records where the field at position 264 equals a constant value (likely a flag or type indicator).
     - `I*`: Indicates inclusion of all records.
     - `I C 1NECD`: Includes records where position 1 equals `N`, `E`, `C`, or `D` (likely 1099 types: Non-Employee Compensation, etc.).
     - `IAC 264 264EQC?L'110,1'?`: Conditionally includes records based on the value of `?L'110,1'?` (e.g., `M` or `N` from the main OCL, determining 1099 type).
     - **Sort Fields**:
       - `FNC 264 264`: Sorts on the 1099 type field (position 264, 1 character).
       - `FNC 4 8`: Sorts on the vendor field (positions 4-8, 5 characters).
       - `FNC 2 3`: Sorts on the company field (positions 2-3, 2 characters).
   - `RUN`: Executes the sort, producing the sorted file `?9?AP765S`.
   - **Purpose**: This step sorts the vendor data by 1099 type, vendor, and company to organize records for subsequent processing.

4. **Process Sorted Data**:
   - `LOAD AP766`: Loads the program `AP766` (likely an RPG program for data processing).
   - **Files Used**:
     - `FILE NAME-APVEND,LABEL-?13?,DISP-SHR`: Input file `?13?` (e.g., `APVN2012`), shared mode.
     - `FILE NAME-AP765S,LABEL-?9?AP765S`: Sorted input file `?9?AP765S` from the previous step.
     - `FILE NAME-AP766,LABEL-?9?AP766,RECORDS-1000,EXTEND-500`: Output file `?9?AP766`, with an initial capacity of 1,000 records, extendable by 500.
     - `FILE NAME-PA1099X,LABEL-?9?PA1099X,DISP-SHR`: Additional file `?9?PA1099X`, likely a cross-reference or configuration file, opened in shared mode.
   - `RUN`: Executes `AP766`, which processes the sorted vendor data and produces the output file `?9?AP766`.
   - **Purpose**: This step likely aggregates or transforms the sorted data into a format suitable for printing 1099 forms.

5. **Print 1099 Forms**:
   - `LOAD AP765`: Loads the program `AP765` (likely an RPG program for printing).
   - **File Used**:
     - `FILE NAME-AP766,LABEL-?9?AP766,DISP-SHR`: Input file `?9?AP766`, the processed data from the previous step, shared mode.
   - **Printer Configuration**:
     - `OVRPRTF FILE(AP1099) FORMTYPE(1099) CPI(10) LPI(6)`: Overrides printer file `AP1099` to use form type `1099`, with 10 characters per inch (`CPI`) and 6 lines per inch (`LPI`).
     - The commented line `PRINTER NAME-AP1099,FORMSNO-1099,ALIGN-YES,LPI-6,CPI-10` suggests similar printer settings, possibly for compatibility with older syntax.
   - `RUN`: Executes `AP765`, which prints the 1099 forms using the data in `?9?AP766`.
   - **Purpose**: This step generates the physical or electronic 1099 forms for vendors.

6. **Cleanup Temporary File**:
   - `IF DATAF1-?9?AP766 DELETE ?9?AP766,F1`: After processing, checks if the temporary file `?9?AP766` exists and deletes it to clean up.

---

### Business Rules

1. **Input File Validation**:
   - The input file `?13?` (e.g., `APVN2012`) must exist and contain vendor data from the period-end process (`AP300`).
   - The file is a snapshot of `GAPVEND` before monthly and yearly totals are cleared.

2. **Data Sorting**:
   - Vendor records are sorted by:
     - 1099 type (position 264, e.g., `M` for Miscellaneous, `N` for Non-Employee Compensation).
     - Vendor code (positions 4-8).
     - Company code (positions 2-3).
   - Only records matching specific 1099 types (`N`, `E`, `C`, `D`) or the value in `?L'110,1'?` (e.g., `M` or `N`) are included.

3. **Temporary Files**:
   - Temporary files `?9?AP765S` and `?9?AP766` are created during processing and deleted afterward to avoid conflicts in subsequent runs.
   - `?9?AP765S` is a sorted version of the input file.
   - `?9?AP766` contains processed data ready for printing.

4. **File Retention**:
   - The sorted file `?9?AP765S` is retained as a job file (`RETAIN-J`), ensuring it persists for the duration of the job.

5. **Printer Configuration**:
   - The 1099 forms are printed with specific formatting (10 CPI, 6 LPI) on form type `1099`, ensuring compliance with IRS requirements.

6. **Year 2000 Compliance**:
   - The `GSY2K` routine ensures dates are handled correctly, particularly for the year specified in `?10?`.

---

### Tables/Files Used

1. **Input File**:
   - `?13?` (e.g., `APVN2012`): The vendor file created by `AP300`, a snapshot of `GAPVEND` containing vendor data before totals are cleared. Used in the sort and processing steps.

2. **Temporary Files**:
   - `?9?AP765S` (e.g., `XXAP765S`): Sorted output file created by `#GSORT`, used as input for `AP766`.
   - `?9?AP766` (e.g., `XXAP766`): Processed output file created by `AP766`, used as input for `AP765` to print 1099 forms.

3. **Cross-Reference File**:
   - `?9?PA1099X` (e.g., `XXPA1099X`): Likely a configuration or cross-reference file used by `AP766` for 1099 processing, opened in shared mode.

4. **Printer File**:
   - `AP1099`: The printer file used by `AP765` to output 1099 forms, configured with specific formatting (form type `1099`, 10 CPI, 6 LPI).

---

### External Programs Called

1. **#GSORT**:
   - The system sort utility, used to sort the input file `?13?` into `?9?AP765S` based on 1099 type, vendor, and company.

2. **AP766**:
   - Likely an RPG program that processes the sorted file `?9?AP765S` and the original vendor file `?13?`, producing the processed file `?9?AP766`. It also uses the cross-reference file `?9?PA1099X`.

3. **AP765**:
   - Likely an RPG program that reads the processed file `?9?AP766` and prints 1099 forms to the printer file `AP1099`.

4. **AP300** (Referenced):
   - Mentioned in the programmer note as the period-end process that creates the `APVNYYYY` file (`?13?`). Not called directly in this program.

---

### Outputs

1. **Sorted File**:
   - `?9?AP765S`: A temporary file containing the sorted vendor data, created by `#GSORT`.

2. **Processed File**:
   - `?9?AP766`: A temporary file containing processed data ready for printing, created by `AP766`.

3. **Printed 1099 Forms**:
   - Output to the printer file `AP1099`, formatted as 1099 forms (form type `1099`, 10 CPI, 6 LPI). These are the final vendor 1099 forms for the specified year.

---

### Summary

- **Purpose**: The OCL program `AP765.ocl36.txt` orchestrates the processing of vendor 1099 forms by sorting vendor data, processing it, and printing the forms for a specified year.
- **Process**:
  - Deletes any existing temporary file (`?9?AP766`).
  - Sorts the input vendor file (`?13?`, e.g., `APVN2012`) into `?9?AP765S` using `#GSORT`.
  - Processes the sorted data using `AP766`, producing `?9?AP766`.
  - Prints 1099 forms using `AP765` and the printer file `AP1099`.
  - Cleans up by deleting `?9?AP766`.
- **Business Rules**:
  - Ensures valid input file and 1099 types.
  - Sorts data by 1099 type, vendor, and company.
  - Manages temporary files and printer formatting.
  - Ensures Year 2000 compliance.
- **Files Used**:
  - Input: `?13?` (`APVNYYYY`), `?9?PA1099X`.
  - Temporary: `?9?AP765S`, `?9?AP766`.
  - Output: `AP1099` (printer file).
- **External Programs**: `#GSORT`, `AP766`, `AP765`.

This program is a critical component of the 1099 processing workflow, integrating sorting, data transformation, and printing to produce IRS-compliant vendor forms.
