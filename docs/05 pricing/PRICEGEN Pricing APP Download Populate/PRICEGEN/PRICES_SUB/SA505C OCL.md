The provided document, `SA505C.ocl36.txt`, is a System/36 OCL (Operation Control Language) script used in an IBM System/36 environment (or AS/400 in compatibility mode). It is part of the pricing generation process for blended lubes, invoked by `PRICES.ocl36.txt` within the `PRICEGEN.clp` workflow, following `BB953B` and preceding other `SA505*` programs. The script generates a Customer Shipping Analysis Report by sorting sales data, processing it, and producing a report or file output. Below, I will explain the process steps, business rules, external programs called, and tables (files) used, despite the truncation in the document .

### Process Steps of the OCL Script

1. **Delete Temporary File (Conditional)**:
   - `// IF DATAF1-?9?S5505S DELETE ?9?S5505S,F1`: If the temporary file `?9?S5505S` (e.g., `AS5505S`) exists, deletes it to ensure a clean state for new data.

2. **Clear Local Data Area**:
   - `// LOCAL BLANK-*ALL`: Clears the Local Data Area (LDA) to reset all parameters.

3. **Run GSY2K Procedure**:
   - `// GSY2K`: Calls the System/36 procedure `GSY2K`, which likely initializes the environment or handles Y2K date configurations (also used in `PRICEGEN.clp`, `BI944B`, `BI942E`, and `BB953B`).

4. **Set Up LDA for Dates and Report Type**:
   - `// LOCAL OFFSET-1,DATA-'IACI*CI*CI*CI*CI*CI*CI*CI*CI*CI*CI*C'`: Sets inclusion/exclusion flags for filtering (e.g., `'I'` for include, `'C'` for exclude) at offset 1.
   - `// LOCAL OFFSET-100,DATA-'510   ALL'`: Sets report parameters, possibly company or division filters (`510`, `ALL`).
   - `// LOCAL OFFSET-124,DATA-'             N01   N0000000'`: Sets additional filters, possibly for customer or product selection.
   - `// LOCAL OFFSET-151,DATA-'00000000000000000000000P                Y'`: Sets flags, including report type (`'P'` for printer) and other options.
   - `// LOCAL OFFSET-509,DATA-'1980'`: Sets Y2K pivot year for date calculations.

5. **Run GSDT12 Procedure**:
   - `// GSDT12 ,,,,,,,,G`: Executes the `GSDT12` procedure with parameter `'G'`, likely for date or data setup related to sales analysis.

6. **Delete DCS File (Conditional)**:
   - `// IF DATAF1-DCS?WS? DELETE DCS?WS?,F1`: If the file `DCS?WS?` (e.g., `DCSA` if `?WS? = 'A'`) exists, deletes it to clear previous data.

7. **Set Conditional LDA Flags for Sorting**:
   - Series of conditional `LOCAL` statements based on LDA values:
     - `// IF ?L'101,2'?/00 LOCAL OFFSET-1,DATA-'I*C' ELSE LOCAL OFFSET-1,DATA-'IAC'`: Sets offset 1 to `'I*C'` if LDA positions 101–102 are `'00'`, else `'IAC'`.
     - Similar conditions for offsets 4, 7, 10, 13, 16, 19 (checking LDA positions 106, 192, 196, 200, 204, 208) to set `'I*C'` or `'IAC'` for filtering (e.g., include/exclude customers, products, or locations).

8. **Sort Sales Analysis File**:
   - `// LOAD #GSORT`: Loads the System/36 sort utility.
   - Files:
     - `INPUT,LABEL-?9?SAINVC,DISP-SHR`: Input file `?9?SAINVC` (e.g., `ASAINVC`), sales invoice data, shared mode.
     - `OUTPUT,LABEL-?9?S5505S,RECORDS-999000,EXTEND-999000,RETAIN-J`: Output file `?9?S5505S`, temporary, 999,000 records, job-level retention.
   - Sort specification (`HSORTR`):
     - `20A`: Sorts in ascending order on 20 fields.
     - `3X 1024 N`: Processes records of 1024 bytes.
     - `I C 1 1NESADEL`: Excludes records where position 1 ≠ `'D'` (non-deleted records).
     - Sort fields:
       - `FNC 2 3`: Company number.
       - `FNC 4 9`: Customer number.
       - `FNC 17 19`: Ship-to number.
       - `FNC 48 51`: Product code.
       - `FNC 193 195`: Container code.
       - `FNC 235 242`: Ship date (CYMD).
       - `FNC 10 16`: Invoice number.
     - Output fields:
       - `FDC 1 256`, `FDC 257 512`, `FDC 513 768`, `FDC 769 1024`: Includes all 1024 bytes.
       - `FDV 5`: Likely specifies 5 output files or divisions (context unclear due to truncation).
   - `// RUN`: Executes the sort, producing `?9?S5505S`.

9. **Run SA505X Program**:
   - `// LOAD SA505X`: Loads the `SA505X` program.
   - Files:
     - `SA5FILD,LABEL-?9?S5505S,DISP-SHR`: Sorted sales data, shared mode.
     - `SA5SHX,LABEL-?9?SA5SHX,DISP-SHR`: Sales history index, shared mode.
   - `// RUN`: Executes `SA505X`, likely preprocessing or indexing sales data for the report.

10. **Clear SA505C File (Conditional)**:
    - `// IF DATAF1-?9?SA505C CLRPFM ?9?SA505C`: If the output file `?9?SA505C` (e.g., `ASA505C`) exists, clears it.

11. **Run SA505C Program**:
    - `// LOAD SA505C`: Loads the `SA505C` program to generate the Customer Shipping Analysis Report.
    - Files:
      - `SA5FILD,LABEL-?9?S5505S,DISP-SHR`: Sorted sales data, shared mode.
      - `SA5SHX,LABEL-?9?SA5SHX,DISP-SHR`: Sales history index, shared mode.
      - `ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Customer master file, shared mode.
      - `SHIPTO,LABEL-?9?SHIPTO,DISP-SHR`: Ship-to file, shared mode.
      - `GSPRCL,LABEL-?9?GSPRCL,DISP-SHR`: Product class file, shared mode.
      - `BICONT,LABEL-?9?BICONT,DISP-SHR`: Contract file, shared mode.
      - `SAINVC,LABEL-?9?SAINVC,RECORDS-999000,EXTEND-999000,RETAIN-J`: Sales invoice file, job-level retention.
      - `SACSTC,LABEL-?9?SACSTC,RECORDS-999000,EXTEND-999000,RETAIN-J`: Customer summary file, job-level retention.
      - `EXCELOUT,LABEL-?9?SA505C,RETAIN-T`: Output file for report data, temporary retention.
    - Printer output:
      - `// IF ?L'174,1'?/P PRINTER NAME-LIST`: If LDA position 174 = `'P'`, outputs to printer `LIST`.
      - `// IF ?L'174,1'?/D PRINTER NAME-LIST,HOLD-YES,PRIORITY-0,FORMSNO-DCSA`: If LDA position 174 = `'D'`, outputs to printer `LIST` with hold and specific form (`DCSA`).
    - `// RUN`: Executes `SA505C`, generating the report or file output.

12. **Cleanup**:
    - `// LOCAL BLANK-*ALL`: Clears the LDA again.
    - `// IF DATAF1-DCS?WS? DELETE DCS?WS?,F1`: Deletes `DCS?WS?` if it exists.
    - `// IF DATAF1-?9?S5505S DELETE ?9?S5505S,F1`: Deletes `?9?S5505S` if it exists.

### Business Rules (Inferred)

1. **Purpose**: Generates a Customer Shipping Analysis Report by sorting sales invoice data (`?9?SAINVC`), preprocessing it with `SA505X`, and producing a report or file output (`?9?SA505C`) with customer, product, and shipping details, using `SA505C`.
2. **Sorting**:
   - Sorts `?9?SAINVC` by company, customer, ship-to, product, container, ship date, and invoice number to organize data for analysis.
   - Excludes deleted records (`ESADEL ≠ 'D'`).
3. **Filtering**:
   - Uses LDA flags (`I*C` or `IAC`) to include/exclude records based on company, customer, product, or other criteria (set at offsets 1, 4, 7, 10, 13, 16, 19).
   - Supports filtering by date range, company (`510 ALL`), and other parameters (LDA offset 124).
4. **Output**:
   - Produces a report on printer `LIST` (if `?L'174,1' = 'P'`) or a held report with form `DCSA` (if `'D'`).
   - Writes data to `?9?SA505C` (likely for Excel or further processing).
5. **Context**: Part of `PRICES.ocl36.txt`, complements rack pricing (`BB953B`) by analyzing customer shipping data, likely for pricing validation or sales reporting.
6. **Cleanup**: Ensures temporary files (`?9?S5505S`, `DCS?WS?`) are deleted to prevent data conflicts.

### External Programs Called

1. **GSY2K**: System/36 procedure for environment initialization or Y2K date handling.
2. **GSDT12**: Procedure for date or data setup, called with parameter `'G'`.
3. **#GSORT**: System/36 sort utility to sort `?9?SAINVC` into `?9?S5505S`.
4. **SA505X**: Preprocesses sorted sales data (`?9?S5505S`) with index (`?9?SA5SHX`).
5. **SA505C**: Generates the final Customer Shipping Analysis Report or file output.

### Tables (Files) Used

1. **SAINVC** (`?9?SAINVC`):
   - **Access**: Input, shared mode (`DISP-SHR`).
   - **Purpose**: Sales invoice data for sorting and reporting.
2. **S5505S** (`?9?S5505S`):
   - **Access**: Output (sort), input/output (`SA505X`, `SA505C`), temporary, job-level retention.
   - **Purpose**: Sorted sales data.
3. **SA5SHX** (`?9?SA5SHX`):
   - **Access**: Input, shared mode.
   - **Purpose**: Sales history index for `SA505X` and `SA505C`.
4. **ARCUST** (`?9?ARCUST`):
   - **Access**: Input, shared mode.
   - **Purpose**: Customer master data for customer details.
5. **SHIPTO** (`?9?SHIPTO`):
   - **Access**: Input, shared mode.
   - **Purpose**: Ship-to data for shipping details.
6. **GSPRCL** (`?9?GSPRCL`):
   - **Access**: Input, shared mode.
   - **Purpose**: Product class data for product categorization.
7. **BICONT** (`?9?BICONT`):
   - **Access**: Input, shared mode.
   - **Purpose**: Contract data for company or agreement details.
8. **SACSTC** (`?9?SACSTC`):
   - **Access**: Output, job-level retention.
   - **Purpose**: Customer summary data for reporting.
9. **SA505C** (`?9?SA505C`):
   - **Access**: Output, temporary retention.
   - **Purpose**: Output file for report data (e.g., Excel).
10. **DCS?WS?** (e.g., `DCSA`):
    - **Access**: Temporary, deleted if exists.
    - **Purpose**: Unknown, likely a work file.
11. **LIST**:
    - **Access**: Printer output.
    - **Purpose**: Customer Shipping Analysis Report.

### Additional Notes

- **Context**: Invoked by `PRICES.ocl36.txt` after `BB953B`, part of the pricing generation workflow, analyzing shipping data to complement rack pricing.
- **System/36 Environment**: Uses OCL and System/36 utilities, likely on AS/400.
- **Truncation**: The document is truncated, but key steps (sorting, program execution, file handling) are clear. Missing sort details likely include additional field mappings.
- **Error Handling**: Relies on System/36 and called programs for error handling.
- **Output Flexibility**: Supports printer (`LIST`) or file output (`?9?SA505C`), controlled by LDA position 174 (`'P'` or `'D'`).

If you have the RPG source code for `SA505X`, `SA505C`, or other `SA505*` programs, or need further analysis, please provide those details! Let me know if you have additional questions or files to share.