The `BI9078` RPGLE program is part of the Bradford Order Entry/Invoices system and is designed to copy customer and ship-to product description records from one customer/ship-to pair to another. It is called from the main RPGLE program `BI907P` via the `BI907C` CLP program and produces a printed report of the copied records. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the BI9078 RPGLE Program

The `BI9078` program performs a batch copy operation, reading records from a source customer/ship-to pair, copying them to a target customer/ship-to pair, updating or adding records as needed, and generating a printed report. The program does not use a display file or subfile, focusing instead on file operations and report generation. The process steps are as follows:

1. **Initialization (`*inzsr` Subroutine)**:
   - Receives input parameters:
     - `p$cono`: Company number (2-digit numeric).
     - `p$cust`: Target customer number (6-digit numeric).
     - `p$ship`: Target ship-to number (3-digit numeric).
     - `p$cpyfcst`: Source customer number (6-digit numeric).
     - `p$cpyfshp`: Source ship-to number (3-digit numeric).
     - `p$fgrp`: File group (`Z` or `G`, 1-character string).
   - Sets the report header (`c$hdr1`) to "Copy Customer Product Descriptions".
   - Captures the current date and time using the `TIME` operation, formatting it into the `t#cymd` data structure for use in history records and the report.
   - Defines key lists:
     - `klship`: For accessing source records in `arcup3` (company, source customer, source ship-to).
     - `klshipcpyf`: For accessing target records in `arcupr` (company, target customer, target ship-to, product, container type).
   - Initializes the print overflow flag (`prtovr = *on`) to ensure the report header is printed initially.
   - Calls `openprtf` to open the printer file (`qsysprt`) and `opntbl` to open database files.

2. **Open Printer File (`openprtf` Subroutine)**:
   - Constructs an override command for the printer file `qsysprt` using the `ovr` array (specifying page size, lines per inch, characters per inch, overflow line, output queue, hold, and save options).
   - Executes the override using the `QCMDEXC` API.
   - Opens the printer file `qsysprt` with the `usropn` option and overflow indicator (`*inof`).

3. **Open Database Tables (`opntbl` Subroutine)**:
   - Applies file overrides based on the `p$fgrp` parameter (`Z` or `G`) for files `arcup3`, `arcupr`, and `arcuphs` using the `QCMDEXC` API.
   - Opens these files with the `usropn` option:
     - `arcup3`: Input file for source records (read-only, `if`).
     - `arcupr`: Update file for target records (read/write, `uf a`).
     - `arcuphs`: Output file for history records (write-only, `o`).

4. **Copy Records (`WriteArcupr` Subroutine)**:
   - Positions the file cursor at the first record in `arcup3` matching the source key (`klship`: company, source customer, source ship-to) using `setll`.
   - If records exist (`%equal(arcup3)`), reads each record sequentially using `reade` until end-of-file (`%eof(arcup3)`).
   - For each source record:
     - Checks if a corresponding record exists in `arcupr` for the target customer/ship-to pair using `chain` with `klshipcpyf` (company, target customer, target ship-to, product, container type).
     - **If found** (record exists):
       - Sets `newrcd = 'N'` to indicate an update.
       - Updates the product description (`cpyt_cpcpds`) in `arcupr` with the source description (`cpyf_cpcpds`).
       - Calls `WriteHist` to log the update in `arcuphs`.
       - Calls `ovrflo` to handle report overflow and prints a detail line (`dtl01`).
     - **If not found** (new record):
       - Sets `newrcd = 'Y'` to indicate a new record.
       - Clears the `arcupr` record format.
       - Sets fields: `cpyt_cpdel = 'A'` (active), `cpyt_cpcono` (company), `cpyt_cpcust` (target customer), `cpyt_cpship` (target ship-to), `cpyt_cpcnty` (container type), `cpyt_cpprod` (product), `cpyt_cpcpds` (product description).
       - Writes the new record to `arcupr`.
       - Calls `WriteHist` to log the addition.
       - Calls `ovrflo` to handle report overflow and prints a detail line (`dtl01`).
   - Continues reading source records until all are processed.

5. **Write History Record (`writehist` Subroutine)**:
   - Clears the `arcuphs` record format.
   - Populates history fields:
     - `ahcono`: Company number (`p$cono`).
     - `ahcust`: Target customer number (`p$cust`).
     - `ahship`: Target ship-to number (`p$ship`).
     - `ahdel`: Set to `'A'` (active).
     - `ahprod`: Product code from source (`cpyf_cpprod`).
     - `ahcpds`: Product description from source (`cpyf_cpcpds`).
     - `ahcnty`: Container type from source (`cpyf_cpcnty`).
     - If updating an existing record (`newrcd = 'N'`), includes additional fields: `ahglcd` (GL code), `ahfrcd` (freight code), `ahsfrt` (separate freight), `ahcafr` (calculate freight).
     - `ahchd8`: Change date (`t#cymd`).
     - `ahchtm`: Change time (`t#hms`).
     - `ahuser`: User ID (`ps#usr8`).
   - Writes the history record to `arcuphs`.

6. **Handle Report Overflow (`ovrflo` Subroutine)**:
   - Checks the printer overflow indicator (`*inof`).
   - If overflow occurs or `prtovr = *on`, prints the report header (`hdr01`) and resets `prtovr`.
   - The header includes:
     - Company name ("American Refining Group").
     - Page number.
     - Job name, program name, user ID, file group, date, and time.
     - Column headings for the detail lines (company, customer, ship-to, product, container type, customer product description, new record indicator).

7. **Report Generation**:
   - For each processed record, prints a detail line (`dtl01`) with:
     - `cpyt_cpcono`: Company number.
     - `cpyt_cpcust`: Target customer number.
     - `cpyt_cpship`: Target ship-to number.
     - `cpyt_cpprod`: Product code.
     - `cpyt_cpcnty`: Container type.
     - `cpyt_cpcpds`: Customer product description.
     - `newrcd`: Indicates whether the record was new (`Y`) or updated (`N`).

8. **Program Termination**:
   - Closes all open files (`arcup3`, `arcupr`, `arcuphs`, `qsysprt`).
   - Sets `*inlr = *on` and returns.

---

### Business Rules

The `BI9078` program enforces the following business rules:

1. **Record Copying**:
   - Copies product description records from a source customer/ship-to pair (`p$cpyfcst`, `p$cpyfshp`) to a target customer/ship-to pair (`p$cust`, `p$ship`) within the same company (`p$cono`).
   - If a target record already exists for the same product and container type, updates the product description (`cpyt_cpcpds`).
   - If no target record exists, creates a new record with the source data and sets the deletion flag to active (`cpyt_cpdel = 'A'`).

2. **History Logging**:
   - Every update or addition is logged in the `arcuphs` history file with details including company, customer, ship-to, product, container type, product description, change date, time, and user.
   - For updates, additional fields (GL code, freight code, separate freight, calculate freight) are included in the history record.

3. **File Group Flexibility**:
   - Supports different file sets (`Z` or `G`) via overrides, allowing the program to work with different data environments.

4. **Report Generation**:
   - Produces a printed report (`qsysprt`) with a header and detail lines for each copied or updated record.
   - The report is held in the output queue (`HOLD(*YES)`) and saved (`SAVE(*YES)`) for later review.
   - The report format includes 184 characters per line, 8 lines per inch, 15 characters per inch, and an overflow at line 62.

5. **Error Handling**:
   - Assumes valid input parameters (no explicit validation in the provided code).
   - Relies on the calling program (`BI907P` via `BI907C`) to provide valid company, customer, and ship-to values.

---

### Tables Used

The program uses the following database files, opened with the `usropn` option and overridden based on the file group (`Z` or `G`):
1. **arcup3**:
   - Purpose: Stores source customer/ship-to cross-reference data for copying.
   - Used in: `WriteArcupr` to read source records.
   - Prefix: `cpyf_` (e.g., `cpyf_cpcpds` for product description).
   - Record Format: Renamed to `arcuprlf`.
   - Override: `garcup3` (G group) or `zarcup3` (Z group).
   - Access: Input (`if`).

2. **arcupr**:
   - Purpose: Stores target customer/ship-to cross-reference data for updating or adding records.
   - Used in: `WriteArcupr` to check for existing records, update, or write new records.
   - Prefix: `cpyt_` (e.g., `cpyt_cpcpds` for product description).
   - Override: `garcupr` (G group) or `zarcupr` (Z group).
   - Access: Update (`uf a`).

3. **arcuphs**:
   - Purpose: Stores history records for changes to customer/ship-to data.
   - Used in: `writehist` to log updates or additions.
   - Override: `garcuphs` (G group) or `zarcuphs` (Z group).
   - Access: Output (`o`).

4. **qsysprt**:
   - Purpose: Printer file for generating the copy report.
   - Used in: `WriteArcupr` and `ovrflo` to print headers and detail lines.
   - Override: Configured with `OVRPRTF` for page size (68 lines, 184 characters), 8 LPI, 15 CPI, overflow at line 62, held and saved in the output queue.
   - Access: Output (`o`).

---

### External Programs Called

The `BI9078` program calls the following external program:
1. **QCMDEXC**:
   - Purpose: Executes file override commands for database and printer files.
   - Called in: `opntbl` and `openprtf` subroutines.
   - Parameters: `dbov##` (override command, 160 characters), `dbol##` (command length, 15.5).

---

### Summary

The `BI9078` RPGLE program is a specialized batch utility for copying customer/ship-to product description records from a source to a target customer/ship-to pair within the same company. It updates existing records or adds new ones, logs changes to a history file (`arcuphs`), and generates a detailed printed report of the operations performed. The program supports flexible file group handling (`Z` or `G`) and ensures all actions are documented for auditing. It integrates with the broader system via the `BI907P` and `BI907C` programs, relying on them to provide valid input parameters and manage the temporary work file context.