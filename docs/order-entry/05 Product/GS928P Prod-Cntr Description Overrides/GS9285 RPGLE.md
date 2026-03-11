### Process Steps of the RPG Program (GS9285)

The program **GS9285** is a batch-oriented RPG III program designed to print a listing of Product/Container entries from the GSPRCT file on an IBM i (AS/400) system. It is called from another program (e.g., GS928P, as referenced in the query) and generates a formatted report to the QSYSPRT printer file. The program is straightforward, reading all records from GSPRCT sequentially and printing them with headers, handling page overflows, and applying file overrides based on the input parameter `p$fgrp` ('G' or 'Z'). It does not involve interactive user input or a display file, focusing solely on report generation.

The high-level process flow is as follows:

1. **Initialization (*INZSR Subroutine)**:
   - Receives a single entry parameter: `p$fgrp` (file group: 'G' or 'Z' to determine file overrides).
   - Sets the report header (`c$hdr1`) to "Product/Container Listing by Co#/Prod/Cntr" from `hdr(01)`.
   - Sets the print overflow flag (`prtovr` = *ON) to ensure headers are printed initially.
   - Calls the OPNTBL subroutine to apply file overrides and open the GSPRCT file.
   - Initializes date/time fields (`t#time`, `t#mdcy`, `t#hms`) for use in report headers.

2. **Open Database Tables (OPNTBL Subroutine)**:
   - Applies file overrides for GSPRCT using QCMDEXC based on `p$fgrp`:
     - For 'G', overrides to GGS* files (e.g., GSPRCT to GGSPRCT).
     - For 'Z', overrides to ZGS* files (e.g., GSPRCT to ZGSPRCT).
   - Opens the GSPRCT file (input-only, USROPN).

3. **Print Listing (PRTLIST Subroutine - Main Processing Logic)**:
   - Calls OPENPRTF to open the printer file (QSYSPRT) with overrides.
   - Reads GSPRCT sequentially until end-of-file (*INLR = *ON):
     - For each record, checks for overflow (OVRFLO subroutine).
     - Prints detail line (EXCEPT DTL01) with fields: company (`ptcono`), product (`ptprod`), container (`ptcntr`), description 1 (`ptdes1`), description 2 (`ptdes2`), and delete flag (`ptdel`).
   - Calls CLOSPRTF to close the printer file and clean up overrides.

4. **Process Overflow (OVRFLO Subroutine)**:
   - Checks the overflow indicator (*INOF).
   - If overflow (*INOF = *ON), sets `prtovr` = *ON and indicators *IN81-*IN85 to trigger header printing.
   - If `prtovr` = *ON, prints the report header (EXCEPT HDR01) and resets `prtovr`.

5. **Open Print File (OPENPRTF Subroutine)**:
   - Constructs a printer override command by concatenating `ovr(01)` (PAGESIZE, LPI, CPI, OVRFLW) and `ovr(02)` (OUTQ, FORMTYPE, HOLD, SAVE).
   - Executes the override using QCMDEXC.
   - Opens the QSYSPRT printer file.

6. **Close Print File (CLOSPRTF Subroutine)**:
   - Closes the QSYSPRT printer file.
   - Executes a delete override command (`ovr(03)`: DLTOVR) using QCMDEXC to clean up.

7. **Program End**:
   - Closes all files (via CLOSPRTF and implicit GSPRCT closure).
   - Sets *INLR = *ON and returns.

The program is minimal, designed to produce a static report without user interaction or complex logic. It prints all records in GSPRCT, regardless of their active/inactive status (`ptdel`), sorted by company, product, and container (implied by the keyed file).

### Business Rules

- **Purpose**:
  - Generates a printed report of all Product/Container records from GSPRCT, including company, product, container, descriptions, and delete flag.
  - The report is spooled to the job’s output queue (OUTQ(*JOB)) with HOLD(*YES) and SAVE(*YES), allowing users to review/release later.

- **Report Format**:
  - **Header** (printed on first page and after overflow):
    - Line 1: "American Refining Group" (left), report title (`c$hdr1`, center), page number (right).
    - Line 2: Job name, program name (left), date in MM/DD/CCYY format (right).
    - Line 3: User ID, file group (`p$fgrp`, left), time in HH:MM:SS format (right).
    - Line 4: Underline (array `str`).
    - Line 5: Column headers (Co, Prod, Cntr, Description-1, Description-2, Del).
    - Line 6: Underline (array `str`).
  - **Detail Lines**:
    - Fields: `ptcono` (company), `ptprod` (product), `ptcntr` (container), `ptdes1` (description 1), `ptdes2` (description 2), `ptdel` (delete flag: 'A' or 'I').
    - Printed sequentially, one record per line.
  - Page size: 68 lines, 164 characters wide, 8 lines per inch, 15 characters per inch, overflow at line 62.

- **Data Selection**:
  - Reads all records from GSPRCT sequentially, with no filtering (e.g., includes both active and inactive records).
  - Sorted by key fields (company, product, container) as defined in the file’s key structure.

- **File Overrides**:
  - Uses `p$fgrp` to determine whether to access GGS* ('G') or ZGS* ('Z') files, ensuring data separation (e.g., for different companies or environments).

- **General**:
  - No validation or error handling for missing/invalid records; assumes GSPRCT is accessible.
  - No user interaction; the report is generated automatically upon call.
  - No auditing (e.g., no tracking of who ran the report beyond job/user in headers).
  - Printer file settings ensure the report is held and saved for later retrieval.

### Tables (Files) Used

The program uses the following files, overridden based on `p$fgrp` (to GGS* for 'G' or ZGS* for 'Z'):

- **GSPRCT**: Input file (IF - input full procedural, E - externally described, K - keyed). Primary data source containing Product/Container records (fields: `ptcono`, `ptprod`, `ptcntr`, `ptdes1`, `ptdes2`, `ptdel`).
- **QSYSPRT**: Printer file (O - output, F - fixed length 164 characters, USROPN). Used to generate the report with headers and detail lines.

Both files are opened with USROPN, and GSPRCT is overridden to GGSPRCT or ZGSPRCT based on `p$fgrp`.

### External Programs Called

The program calls only one system utility (via CALL):

- **QCMDEXC**: Executes override commands for:
  - GSPRCT file redirection (to GGSPRCT or ZGSPRCT).
  - QSYSPRT printer settings (PAGESIZE, LPI, CPI, OVRFLW, OUTQ, FORMTYPE, HOLD, SAVE) and cleanup (DLTOVR).

No external RPG or custom programs are called, making this a self-contained report generator.