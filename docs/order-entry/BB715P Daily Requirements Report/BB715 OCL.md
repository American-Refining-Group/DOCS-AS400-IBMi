The provided `BB715.ocl36.txt` is an Operator Control Language (OCL) program for an IBM System/36 or AS/400, designed to orchestrate the generation of a "Daily Requirements Report" through a series of preprocessing, sorting, and reporting steps. Below, I’ll explain the process steps, list the external programs called, and identify the tables (files) used, ensuring clarity and conciseness.

### Process Steps of the OCL Program

The OCL program coordinates multiple programs and file operations to prepare and generate the Daily Requirements Report. Here’s a step-by-step breakdown:

1. **Header and Initialization**:
   - The program starts with a header indicating it is a "Daily Requirements Report."
   - `GSY2K` suggests a configuration or version, possibly related to Y2K compliance.
   - `GSDELETE BB713M,BB715S,,,,,,,?9?`: Deletes existing files `BB713M` and `BB715S` (likely temporary or intermediate files) with a label including the parameter `?9?` (a dynamic value, e.g., a company code or date).

2. **Preprocessing with BB713**:
   - `LOAD BB713`: Loads the program `BB713`.
   - **File Specifications**:
     - `BBORDR` (input, labeled `?9?BBORDR`, shared read mode `DISP-SHR`).
     - `BBORDR2` (likely an alias for `BBORDR`, same label, shared read mode).
     - `BB713M` (output, labeled `?9?BB713M`, new file with 999,000 records, extendable by 1,000 records, `DISP-NEW`).
   - `RUN`: Executes `BB713`, which likely processes order data from `BBORDR` to create an intermediate file `BB713M`.

3. **Processing with BB714**:
   - `LOAD BB714`: Loads the program `BB714`.
   - **File Specifications**:
     - `BB713M` (input, shared read mode, from previous step).
     - `GSUMCV` (input, labeled `?9?GSUMCV`, shared read mode, likely a summary or conversion file).
   - `RUN`: Executes `BB714`, which likely refines or aggregates data from `BB713M` using `GSUMCV`.

4. **Additional Processing with BB714A**:
   - `LOAD BB714A`: Loads the program `BB714A`.
   - **File Specifications**:
     - `BB713M` (input, shared read mode).
     - `GSTABL` (input, labeled `?9?GSTABL`, shared read mode, likely a table file).
     - `GSCNTR1` (input, labeled `?9?GSCNTR1`, shared read mode, possibly a container file).
     - `GSPROD` (input, labeled `?9?GSPROD`, shared read mode, likely a product file).
   - `RUN`: Executes `BB714A`, which further processes `BB713M` using additional reference files (`GSTABL`, `GSCNTR1`, `GSPROD`).

5. **Sorting with #GSORT**:
   - `LOAD #GSORT`: Loads the system sort utility `#GSORT`.
   - **File Specifications**:
     - `INPUT` (mapped to `BB713M`, shared read mode).
     - `OUTPUT` (mapped to `BB715S`, new file with 999,000 records, extendable by 999,000, retained temporarily with `RETAIN-T`).
   - **Sort Specifications**:
     - `HSORTR 18A 3X 530`: Defines a sort with an 18-byte ascending key and a 530-byte record length, repeated 3 times (possibly for multiple sort passes).
     - **Include Conditions (I)**:
       - `I C 1 1NECD`: Includes records where position 1 is not 'D' (not deleted).
       - `IAC 2 3EQC?L'101,2'?`: Includes records where positions 2–3 equal a parameter from position 101 (length 2, likely a company code).
       - `IAC 10 12GTC000`: Includes records where positions 10–12 are greater than '000'.
       - `IAC 10 12LTC900`: Includes records where positions 10–12 are less than '900'.
     - **Sort Fields (FNC)**:
       - `FNC 2 3 COMPANY`: Sorts by company (positions 2–3).
       - `FNC 522 523 00 OR 50`: Sorts by a field at 522–523 (possibly a status code).
       - `FNC 22 24 LOCATION`: Sorts by location (22–24).
       - `FNC 521 521 BULK OR PACKAGED`: Sorts by bulk/packaged flag (521).
       - `FNC 121 123 CONTAINER`: Sorts by container (121–123).
       - `FNC 51 53 UNIT OF MEASURE`: Sorts by unit of measure (51–53).
       - `FNC 25 28 PRODUCT`: Sorts by product (25–28).
     - **Data Fields (FDC)**:
       - `FDC 1 256`, `FDC 257 512`, `FDC 513 530`: Copies data from positions 1–256, 257–512, and 513–530 to the output file.
   - `RUN`: Executes the sort, producing `BB715S` from `BB713M` with the specified order and filtering.

6. **Report Generation with BB715**:
   - `LOAD BB715`: Loads the program `BB715`.
   - **File Specifications**:
     - `BBORDR` (mapped to `BB715S`, the sorted output from `#GSORT`, shared read mode).
     - `BICONT` (labeled `?9?BICONT`, shared read mode, for company validation).
     - `GSTABL` (commented out, not used in this step).
     - `INLOC` (labeled `?9?INLOC`, shared read mode, likely location data).
     - `INTKRY`, `INTKRW` (labeled `?9?INTKRY`, `?9?INTKRW`, shared read mode, possibly transaction keys).
     - `INCHRT` (labeled `?9?INCHRT`, shared read mode, possibly chart or hierarchy data).
     - `GSCTUM` (labeled `?9?GSCTUM`, shared read mode, likely unit of measure data).
     - `GSCNTR1` (labeled `?9?GSCNTR1`, shared read mode, container data).
     - `GSPROD` (labeled `?9?GSPROD`, shared read mode, product data).
     - **For Called Program `MINCHT`**:
       - `MINCHRT` (mapped to `?9?INCHRT`, shared read mode with `DISP-SHRRM`).
       - `MINSTRP` (labeled `?9?INSTRP`, shared read mode with `DISP-SHRRM`, possibly strip data).
       - `ICSUMHY` (labeled `?9?ICSUMHY`, shared read mode, possibly summary history).
   - `RUN`: Executes `BB715`, which generates the final report using the sorted `BB715S` and reference files, potentially calling `MINCHT` for additional processing.

7. **Cleanup**:
   - `GSDELETE BB713M,BB715S,,,,,,,?9?`: Deletes the temporary files `BB713M` and `BB715S` after processing, ensuring a clean environment.

### External Programs Called

- **BB713**: Preprocesses order data to create `BB713M`.
- **BB714**: Refines or aggregates data in `BB713M` using `GSUMCV`.
- **BB714A**: Further processes `BB713M` with additional reference files (`GSTABL`, `GSCNTR1`, `GSPROD`).
- **#GSORT**: System sort utility to sort `BB713M` into `BB715S` based on specified keys and filters.
- **BB715**: Generates the final report from `BB715S` and reference files.
- **MINCHT**: Potentially called by `BB715` (implied by file definitions `MINCHRT`, `MINSTRP`, `ICSUMHY`), likely for chart or hierarchy processing.

### Tables (Files) Used

- **Input Files**:
  - `BBORDR` (order data, used by `BB713`, mapped to `BB715S` in `BB715`).
  - `BBORDR2` (alias for `BBORDR` in `BB713`).
  - `GSUMCV` (summary/conversion data, used by `BB714`).
  - `GSTABL` (table data, used by `BB714A`, commented out in `BB715`).
  - `GSCNTR1` (container data, used by `BB714A` and `BB715`).
  - `GSPROD` (product data, used by `BB714A` and `BB715`).
  - `BICONT` (company data, used by `BB715` for validation, as seen in `BB715P.rpg36.txt`).
  - `INLOC` (location data, used by `BB715`).
  - `INTKRY`, `INTKRW` (transaction key files, used by `BB715`).
  - `INCHRT` (chart/hierarchy data, used by `BB715`, also as `MINCHRT` for `MINCHT`).
  - `GSCTUM` (unit of measure data, used by `BB715`).
  - `MINSTRP` (strip data, used by `MINCHT` in `BB715`).
  - `ICSUMHY` (summary history, used by `MINCHT` in `BB715`).

- **Output/Temporary Files**:
  - `BB713M` (intermediate file created by `BB713`, processed by `BB714`, `BB714A`, and sorted by `#GSORT`).
  - `BB715S` (sorted output from `#GSORT`, used as input for `BB715`).

### Notes

- **Parameter `?9?`**: Appears in file labels (e.g., `?9?BBORDR`), indicating a dynamic value (likely a company code or date) passed at runtime.
- **File Sharing**: Most files use `DISP-SHR` (shared read) or `DISP-SHRRM` (shared read with read-only mode), allowing concurrent access.
- **Sort Logic**: The `#GSORT` step filters out deleted records (`NECD`) and applies specific range checks (positions 10–12 between '000' and '900') and company code matches, then sorts by multiple fields for report organization.
- **Integration with BB715P**: The `BB715` program likely uses the `BB715P.rpg36.txt` logic for input validation (company, date, job queue) before processing `BB715S` to generate the report.
- **Commented Lines**: The commented `GSTABL` in `BB715` suggests it was used in a prior version or is optional.

### Summary

The `BB715.ocl36.txt` OCL program orchestrates the Daily Requirements Report by:
1. Deleting old temporary files.
2. Running preprocessing programs (`BB713`, `BB714`, `BB714A`) to prepare data in `BB713M`.
3. Sorting `BB713M` into `BB715S` using `#GSORT` with specific filters and sort keys.
4. Generating the report with `BB715`, using sorted data and reference files, potentially calling `MINCHT`.
5. Cleaning up temporary files.

**External Programs**: `BB713`, `BB714`, `BB714A`, `#GSORT`, `BB715`, `MINCHT`.
**Files**: `BBORDR`, `BBORDR2`, `BB713M`, `BB715S`, `GSUMCV`, `GSTABL`, `GSCNTR1`, `GSPROD`, `BICONT`, `INLOC`, `INTKRY`, `INTKRW`, `INCHRT`, `GSCTUM`, `MINCHRT`, `MINSTRP`, `ICSUMHY`.

If you need further analysis (e.g., details on `BB713`, `BB714`, or `MINCHT`), or have additional files, let me know!