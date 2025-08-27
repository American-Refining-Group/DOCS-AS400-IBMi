The RPGLE program `BB9055`, titled "Print Carrier Code Listing," is a component of a billing and invoicing system designed to generate a printed report of carrier ID records. It is called by the main program `BB905P` (as described in the first query) to produce a listing of carrier IDs sorted by company number and carrier ID. Below, I’ll explain the process steps, business rules, database tables used, and external programs called by `BB9055`.

### Process Steps of the RPGLE Program (`BB9055`)

The program reads the `bbcaid` file and outputs a formatted report to the `qsysprt` printer file, including headers and detail lines for each carrier ID record. It handles file overrides and overflow conditions to ensure proper report formatting. Here’s a detailed breakdown of the process steps:

1. **Program Initialization (`*inzsr` Subroutine)**:
   - **Receives Input Parameter**: Accepts one parameter, `p$fgrp`, which specifies the file group (`Z` or `G`) for database file overrides.
   - **Sets Headers**: Initializes the report header (`c$hdr1`) with the first element of the `hdr` array ("Carrier Id Listing By Co#/Carrier Id").
   - **Sets Overflow Flag**: Sets `prtovr = *on` to trigger the printing of headers at the start.
   - **Opens Database Tables**: Calls `opntbl` to apply file overrides and open the `bbcaid` file.
   - **Initializes Date/Time**: Uses a time conversion data structure (`t#time`) to capture the current date and time for the report header.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Applies File Overrides**: Based on the `p$fgrp` parameter (`Z` or `G`), executes `OVRDBF` commands to override the `bbcaid` file to the appropriate library (e.g., `gbbcaid` or `zbbcaid`).
   - **Opens File**: Opens the `bbcaid` file for input.

3. **Print Listing (`prtlist` Subroutine)**:
   - **Opens Printer File**: Calls `openprtf` to set up and open the `qsysprt` printer file.
   - **Main Processing Loop**:
     - Reads records from `bbcaid` sequentially until the end of the file (`*inlr = *on`).
     - For each record:
       - Checks for overflow by calling `ovrflo`.
       - Prints a detail line (`dtl01`) containing the company number (`cico`), carrier ID (`cicaid`), carrier name (`cicanm`), EIN (`ciein`), Fuel Facs ID (`ciffid`), and deletion status (`cidel`).
   - **Closes Printer File**: Calls `closprtf` to close `qsysprt` and clean up overrides.

4. **Process Overflow (`ovrflo` Subroutine)**:
   - **Checks Overflow Indicator**: If `*inof` (overflow indicator) is `*on`, sets `prtovr = *on` and activates indicators `*in81` to `*in85` for header formatting.
   - **Prints Headers**: If `prtovr = *on`, prints the header format (`hdr01`) and resets `prtovr = *off`.

5. **Open Printer File (`openprtf` Subroutine)**:
   - **Sets Printer Overrides**: Combines the first two elements of the `ovr` array to configure the printer file with specific attributes (e.g., page size of 68 lines by 164 characters, 8 lines per inch, 15 characters per inch, overflow at line 62, output queue `*JOB`, hold and save options).
   - **Executes Override**: Calls `QCMDEXC` to apply the `OVRPRTF` command.
   - **Opens Printer File**: Opens `qsysprt` for output.

6. **Close Printer File (`closprtf` Subroutine)**:
   - **Closes Printer File**: Closes `qsysprt`.
   - **Deletes Override**: Applies the `DLTOVR` command from `ovr(03)` using `QCMDEXC` to remove the printer file override.

7. **Program Termination**:
   - The program ends after printing all records, with no explicit file closure in the mainline (handled in `closprtf`).

### Report Format
- **Header (`hdr01`)**:
  - Line 1: "American Refining Group" (left), report title (`c$hdr1`, centered), page number (right).
  - Line 2: Job name (`ps#jobn`), program name (`ps#pgm`), date (`t#mdcy`, right).
  - Line 3: User ID (`ps#usr`), file group (`p$fgrp`), time (`t#hms`, right).
  - Line 4: Separator line (`str`, 164 characters).
  - Line 5: Column headers ("Co", "Carr Id", "Carrier Name", "Ein #", "Fuel Facs #", "Del").
  - Line 6: Separator line (`str`).
- **Detail (`dtl01`)**:
  - Prints one line per `bbcaid` record with fields: `cico` (company), `cicaid` (carrier ID), `cicanm` (carrier name), `ciein` (EIN), `ciffid` (Fuel Facs ID), `cidel` (deletion status).

### Business Rules

1. **Report Content**:
   - The report lists all carrier ID records from the `bbcaid` file, sorted by company number and carrier ID (implied by the file’s key structure).
   - Includes all records, regardless of deletion status (`cidel`), allowing visibility of both active and inactive carriers.

2. **Formatting**:
   - The report is formatted for a page size of 68 lines by 164 characters, with 8 lines per inch and 15 characters per inch.
   - Overflow occurs at line 62, triggering header reprinting.
   - The output is held (`HOLD(*YES)`) and saved (`SAVE(*YES)`) in the job’s output queue (`OUTQ(*JOB)`).

3. **File Overrides**:
   - The `bbcaid` file is overridden to the appropriate library (`gbbcaid` or `zbbcaid`) based on the `p$fgrp` parameter.
   - Printer file overrides ensure consistent formatting and output handling.

4. **No User Interaction**:
   - The program is non-interactive, running to completion without user input after being called with the `p$fgrp` parameter.

5. **Error Handling**:
   - No explicit error handling is implemented for file operations or printing, assuming the calling program (`BB905P`) handles errors.
   - Relies on the system’s default error handling for file I/O and printer operations.

### Database Tables Used

The program interacts with the following database file:
1. **bbcaid**:
   - Input-only file containing carrier ID records.
   - Key fields: Likely `cico` (company) and `cicaid` (carrier ID), though not explicitly used in the program (sequential read).
   - Fields printed: `cico` (company), `cicaid` (carrier ID), `cicanm` (carrier name), `ciein` (EIN), `ciffid` (Fuel Facs ID), `cidel` (deletion status).

### External Programs Called

The program calls the following external program:
1. **QCMDEXC**:
   - Executes file override commands (`OVRDBF` for `bbcaid`, `OVRPRTF` and `DLTOVR` for `qsysprt`) in the `opntbl`, `openprtf`, and `closprtf` subroutines.
   - Parameters: `dbov##` (override command string), `dbol##` (length of command, 80 or 160).

### Additional Notes
- **Non-Interactive Nature**: Unlike `BB905P`, `BB905`, and `BB9054`, which use display files for user interaction, `BB9055` is a batch-style program focused solely on generating a printed report.
- **Printer File Configuration**: The use of `PAGESIZE(68 164)`, `LPI(8)`, `CPI(15)`, and `OVRFLW(62)` ensures a compact, readable report format suitable for wide paper or spooled output.
- **No Revisions**: The program has no listed revisions, indicating stable functionality since its creation in 2013.
- **Minimal Indicators**: Unlike other programs in the suite, `BB9055` uses only the overflow indicator (`*inof`) and indicators `*in81` to `*in85` for header formatting, reflecting its simple output-driven design.

This program complements the carrier ID management suite by providing a reporting function, enabling users to review all carrier ID records in a printed format. Let me know if you need further clarification or additional details!