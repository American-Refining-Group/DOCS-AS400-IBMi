The provided document, `AR745.ocl36.txt`, is an Operation Control Language (OCL) script used in an IBM midrange system (e.g., AS/400 or iSeries) to generate an Electronic Funds Transfer (EFT) Customer Accounts Receivable Due Report. This script is invoked from the main OCL script (`AR745P.ocl36.txt`) under certain conditions (e.g., job queue submission or direct execution). Below, I’ll explain the process steps, business rules, tables (files) used, and external programs called, ensuring clarity and conciseness while adhering to the provided guidelines.

### Process Steps of the OCL Program

The `AR745` OCL script orchestrates the creation of a work file, sorts accounts receivable data, and generates the EFT report. The steps are as follows:

1. **Y2K Processing Initialization**:
   - `// GSY2K`:
     - Invokes Year 2000 (Y2K) compliance processing to handle date-related calculations or conversions, ensuring two-digit year formats are correctly interpreted.

2. **Conditional Work File Creation for Batch Processing**:
   - `// IF ?L'123,1'?/Y ...`:
     - Checks if the value at location 123 (1 character, `KYBATC` from `AR745P`) is 'Y' (indicating batch processing).
     - If true, performs the following:
       - Deletes existing files (if they exist) named `?9?E?L'244,6'?` (settlement date-based file) and `?9?X?L'208,6'?` (settlement date-based file) from library `QS36F`.
       - Creates two physical files (`CRTPF`) in `QS36F` using the source member `AREFTD`:
         - File named `?9?E?L'244,6'?` (e.g., `EMMDDYY` based on settlement date at location 244).
         - File named `?9?X?L'208,6'?` (based on settlement date at location 208).
         - Options `*NOLIST *NOSOURCE` suppress listing and source output during file creation.
     - These files are temporary work files for EFT data processing.

3. **Set Processing Mode**:
   - `// IF ?L'238,6'?/000000 LOCAL OFFSET-13,DATA-'I*C' ELSE LOCAL OFFSET-13,DATA-'IAC'`:
     - Checks if the customer number at location 238 (`KYCUST`) is zero (indicating all customers).
     - If true, sets location 13 to `I*C` (likely indicating "include all customers").
     - If false, sets location 13 to `IAC` (likely indicating a specific customer).
     - This controls whether the report processes all customers or a single customer.

4. **Sort Accounts Receivable Details**:
   - `// IF DATAF1-?9?AR745S DELETE ?9?AR745S,F1`:
     - Deletes the existing `AR745S` file (if present) in library `?9?` to ensure a fresh work file.
   - `// LOAD #GSORT`:
     - Loads the sort utility program `#GSORT` to sort accounts receivable detail records.
   - File Definitions:
     - `FILE NAME-INPUT,LABEL-?9?ARDETL,DISP-SHR`: Input file `ARDETL` (accounts receivable details) in shared mode.
     - `FILE NAME-OUTPUT,LABEL-?9?AR745S,RECORDS-999000,EXTEND-999000,RETAIN-J`: Output file `AR745S` with capacity for 999,000 records, extendable, and retained as a job file.
   - `RUN` and Sort Specifications:
     - `HSORTR 18A 3X 128 N`: Defines a sort header with an 18-byte key, excluding 3 bytes, sorting 128-byte records, no sequence check.
     - Input Conditions (`I*`):
       - `I C 1NECD`: Excludes records where position 1 is a delete flag (not equal to 'C').
       - `IAC 2 3EQC?L'114,2'?`: Includes records where positions 2–3 (company number) match location 114 (`KYCO1`).
       - `IAC 17 17EQCI`: Includes records where position 17 equals 'I' (likely invoice type).
       - `IAC 96 103GEC?L'214,8'?`: Includes records where positions 96–103 (date field) are greater than or equal to location 214 (`KFCYMD`, from-date).
       - `IAC 96 103LEC?L'222,8'?`: Includes records where positions 96–103 are less than or equal to location 222 (`KTCYMD`, to-date).
       - `?L'13,3'? 4 9EQC?L'238,6'?`: Includes records where positions 4–9 (customer number) match location 238 (`KYCUST`) based on the mode set at location 13 (`I*C` or `IAC`).
     - Field Specifications (`FNC`, `FDC`):
       - `FNC 2 3 COMPANY`: Sorts by company number (positions 2–3).
       - `FNC 4 19 CUST,INV,TYPE,SEQ`: Sorts by customer, invoice, type, and sequence (positions 4–19).
       - `FDC 1 128`: Copies the entire 128-byte record to the output.
     - Output: Creates a sorted file `AR745S` containing filtered accounts receivable details.

5. **Generate EFT Report**:
   - `// LOAD AR745`:
     - Loads the `AR745` program (likely an RPG program) to generate the EFT report.
   - File Definitions:
     - `FILE NAME-ARDETL,LABEL-?9?AR745S,DISP-SHR`: Uses the sorted `AR745S` file as input.
     - `FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Customer master file for customer details.
     - `FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`: Control file for accounts receivable data.
     - `PRINTER NAME-LIST,FORMSNO-EFT,ALIGN-YES,CPI-10,LPI-6`: Defines the printer output with EFT form, 10 CPI, 6 LPI, and alignment required.
     - `IF ?9?/G OVRPRTF FILE(LIST1) OUTQ(QUSRSYS/AREFTOUTQ) CPI(10) LPI(6)`:
       - If `?9?` is 'G', overrides printer file `LIST1` to output queue `QUSRSYS/AREFTOUTQ`.
     - `IFF ?9?/G OVRPRTF FILE(LIST1) OUTQ(QUSRSYS/TESTOUTQ) CPI(10) LPI(6)`:
       - If `?9?` is not 'G', uses `QUSRSYS/TESTOUTQ` for testing.
     - `IF ?L'123,1'?/Y FILE NAME-AREFTD,LABEL-?9?E?L'244,6'?,DISP-SHR`:
       - If batch mode (`KYBATC = 'Y'`), includes the settlement date file `?9?E?L'244,6'?`.
     - `IFF ?L'123,1'?/Y FILE NAME-AREFTD,LABEL-?9?X?L'208,6'?,DISP-SHR`:
       - If batch mode, also includes the alternate settlement date file `?9?X?L'208,6'?`.
   - `RUN`: Executes the `AR745` program to print the EFT report.

6. **Check for Credit Invoices and Email Report**:
   - `// IF ?9?/G AR745A ,,,,,,,,?9?`:
     - If `?9?` is 'G', calls the `AR745A` program (with substitution variable `?9?`) to check for credit invoices and send an emailed report if found.
     - This step is conditional on the environment or library setting (`?9? = G`).

### Business Rules

1. **Batch Processing**:
   - If `KYBATC` (location 123) is 'Y', creates temporary work files named based on settlement dates (`?9?E?L'244,6'?` and `?9?X?L'208,6'?`) using the `AREFTD` source member.
   - Deletes existing work files before creation to avoid conflicts.

2. **Customer Selection**:
   - If `KYCUST` (location 238) is zero, processes all EFT customers (`I*C` mode).
   - If `KYCUST` is non-zero, processes a specific customer (`IAC` mode).

3. **Data Filtering and Sorting**:
   - Filters `ARDETL` records to include:
     - Non-deleted records (position 1 not 'C').
     - Matching company number (`KYCO1` at location 114).
     - Invoice type 'I' (position 17).
     - Dates within the range `KFCYMD` (from-date) to `KTCYMD` (to-date).
     - Matching customer number (if specified).
   - Sorts by company number and customer/invoice details.

4. **Report Output**:
   - Outputs the EFT report to a printer with EFT forms, 10 CPI, and 6 LPI.
   - Routes output to `AREFTOUTQ` (production) or `TESTOUTQ` (testing) based on `?9?`.
   - Includes settlement date files in batch mode for additional processing.

5. **Credit Invoice Handling**:
   - If in production mode (`?9? = G`), checks for credit invoices and emails the report using `AR745A`.

### Tables (Files) Used

1. **ARDETL**:
   - Type: Input (disk)
   - Label: `?9?ARDETL`
   - Disposition: Shared (`DISP-SHR`)
   - Purpose: Accounts receivable detail file containing invoice data.

2. **AR745S**:
   - Type: Output (disk)
   - Label: `?9?AR745S`
   - Disposition: Shared (`DISP-SHR`)
   - Attributes: 999,000 records, extendable, retained as job file (`RETAIN-J`)
   - Purpose: Sorted work file for EFT report data.

3. **ARCUST**:
   - Type: Input (disk)
   - Label: `?9?ARCUST`
   - Disposition: Shared (`DISP-SHR`)
   - Purpose: Customer master file for customer details.

4. **ARCONT**:
   - Type: Input (disk)
   - Label: `?9?ARCONT`
   - Disposition: Shared (`DISP-SHR`)
   - Purpose: Accounts receivable control file for configuration data.

5. **AREFTD (Settlement Date File 1)**:
   - Type: Input (disk, conditional)
   - Label: `?9?E?L'244,6'?` (e.g., `EMMDDYY`)
   - Disposition: Shared (`DISP-SHR`)
   - Purpose: Temporary work file for batch processing, named by settlement date.

6. **AREFTD (Settlement Date File 2)**:
   - Type: Input (disk, conditional)
   - Label: `?9?X?L'208,6'?`
   - Disposition: Shared (`DISP-SHR`)
   - Purpose: Alternate temporary work file for batch processing.

7. **LIST**:
   - Type: Printer output
   - Forms: EFT
   - Attributes: 10 CPI, 6 LPI, alignment required
   - Purpose: Output file for the EFT report.

### External Programs Called

1. **#GSORT**:
   - Loaded to sort `ARDETL` into `AR745S` based on specified criteria.
   - Utility program for sorting records.

2. **AR745**:
   - Main program to generate the EFT report using sorted data (`AR745S`), customer data (`ARCUST`), and control data (`ARCONT`).

3. **AR745A**:
   - Conditionally called if `?9? = G` to check for credit invoices and send an emailed report.
   - Invoked with substitution variable `?9?`.

### Summary

The `AR745` OCL script, called from the main `AR745P` OCL script, generates an EFT Customer Accounts Receivable Due Report by:
- Initializing Y2K processing.
- Creating temporary work files for batch mode based on settlement dates.
- Setting processing mode for all or specific customers.
- Sorting accounts receivable details into `AR745S` using `#GSORT`.
- Generating the report via `AR745` with input from `AR745S`, `ARCUST`, and `ARCONT`.
- Optionally emailing the report via `AR745A` if credit invoices are found in production mode.

**Business Rules** ensure proper filtering, sorting, and output routing, with conditional batch file creation and customer selection.

**Tables Used**:
- `ARDETL` (input)
- `AR745S` (sorted output)
- `ARCUST` (customer master)
- `ARCONT` (control)
- `AREFTD` (two settlement date files, conditional)
- `LIST` (printer output)

**External Programs**:
- `#GSORT` (sorting utility)
- `AR745` (report generation)
- `AR745A` (credit invoice check and email)

If you need further details (e.g., specific behavior of `AR745` or `AR745A`, or integration with `AR745P`), let me know!