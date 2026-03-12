The RPGLE program `BB9285` is a component of a billing and invoicing system designed to generate a printed listing of salesman ID records. It is called by the main program `BB928P` (as referenced in earlier documents) to produce a report of salesman ID data. The program reads records from a database file, formats them, and outputs them to a printer file (`qsysprt`). Below, I’ll explain the process steps, outline the business rules, list the database tables used, and identify any external programs called.

### Process Steps of the BB9285 Program

The program is structured to open files, read salesman ID records sequentially, and print a formatted report with headers and details. Here’s a detailed breakdown of the process steps:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Parameters**: Receives one input parameter:
     - `p$fgrp`: File group (`Z` or `G`), determining which database file to use.
   - **Header Setup**: Sets the report header (`c$hdr1`) to "Salesman Id Listing By Co#/Salesman Id" from the `hdr` array.
   - **Initial State**: Sets `prtovr` to `*on` to trigger printing of the report header on the first page.
   - **File Open**: Calls `opntbl` to open the database file.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **File Overrides**: Based on `p$fgrp` (`G` or `Z`), applies database overrides from `ovg` or `ovz` arrays to point to the correct file (`gbbslsm` or `zbbslsm`) using the `QCMDEXC` program.
   - **File Open**: Opens `bbslsm` in input mode (`if`) with `usropn`.

3. **Print Listing (`prtlist` Subroutine)**:
   - **Open Printer File**: Calls `openprtf` to set up and open the printer file (`qsysprt`).
   - **Read Records**: Reads `bbslsm` sequentially until the last record (`*inlr = *on`):
     - For each record, calls `ovrflo` to handle page overflow and print headers if needed.
     - Prints the detail line (`dtl01`) containing the salesman ID record fields.
   - **Close Printer File**: Calls `closprtf` to close `qsysprt` and clean up overrides.

4. **Handle Overflow (`ovrflo` Subroutine)**:
   - Checks if the printer file overflow indicator (`*inof`) is `*on`.
     - If true, sets `prtovr` to `*on` and sets indicators `*in81`–`*in85` to `*on` for header printing.
   - If `prtovr = *on`, prints the report header (`hdr01`) and resets `prtovr` to `*off`.

5. **Open Printer File (`openprtf` Subroutine)**:
   - Constructs a printer override command by combining `ovr(01)` and `ovr(02)` (defining page size, lines per inch, characters per inch, overflow line, output queue, form type, hold, and save options).
   - Executes the override using `QCMDEXC`.
   - Opens the printer file `qsysprt`.

6. **Close Printer File (`closprtf` Subroutine)**:
   - Closes `qsysprt`.
   - Applies the delete override command (`ovr(03)`) using `QCMDEXC` to remove the printer file override.

7. **Program Exit**:
   - Closes all files and returns to the calling program (`BB928P`).

### Business Rules

The program enforces the following business rules:
1. **Report Content**:
   - The report lists all salesman ID records from `bbslsm`, sorted by company number (`smco`) and salesman ID (`smsmid`).
   - Each detail line includes:
     - `smco`: Company number (position 2).
     - `smsmid`: Salesman ID (position 8).
     - `smsmnm`: Salesman name (position 44).
     - `smemal`: Email address (position 95).
     - `smdel`: Deletion status (position 100, e.g., `A` for active, `I` for inactive, `D` for deleted).
2. **Report Format**:
   - The report header (`hdr01`) includes:
     - Company name ("American Refining Group") at position 24.
     - Report title (`c$hdr1`) at position 65.
     - Page number at position 154.
     - Job name and program name at position 10/23.
     - Date (`t#mdcy`) at position 154 in `MM/DD/YYYY` format.
     - User ID (`ps#usr`) and file group (`p$fgrp`) at position 10/20.
     - Time (`t#hms`) at position 154 in `HH:MM:SS` format.
     - Column headers ("Co", "Slsm Id", "Salesman Name", "Email Address", "Del") at positions 2, 10, 27, 58, and 100.
   - Headers are printed at the start of the report and on overflow (line 62, as specified in `ovr(01)`).
3. **Printer Configuration**:
   - The printer file uses a page size of 68 lines and 164 characters, 8 lines per inch, 15 characters per inch, with overflow at line 62.
   - The output is sent to the job’s output queue (`OUTQ(*JOB)`), with the spool file held (`HOLD(*YES)`) and saved (`SAVE(*YES)`).
4. **Database Selection**:
   - Uses file overrides to select `gbbslsm` or `zbbslsm` based on `p$fgrp` (`G` or `Z`).
5. **Record Inclusion**:
   - Includes all records from `bbslsm`, regardless of status (`smdel`), unlike `BB928P`, which can filter inactive records.

### Database Tables Used

The program interacts with the following database file:
1. **bbslsm**: Input file for salesman ID records, opened with `usropn`. Read sequentially to generate the report.

### External Programs Called

The program calls the following external program:
1. **QCMDEXC**: Executes file override commands for `bbslsm` and `qsysprt`, and deletes the printer file override after printing.

### Additional Notes
- **Printer File**: `qsysprt` is a standard IBM i printer file, opened with `usropn` and configured for a 164-character wide report with overflow handling.
- **Indicators**: Uses `*inof` for printer overflow and `*in81`–`*in85` for header printing control, but does not use display-related indicators (e.g., 19, 21–79) since it is a print-only program.
- **Field Prefixes**: Uses `c$` for the report header, `p$` for the input parameter, and standard database fields (`smco`, `smsmid`, etc.) without additional prefixes like `f$` or `s1` used in `BB928P` and `BB928`.
- **Integration with BB928P**: Called by `BB928P` for option 15 (F15) to print a salesman ID listing, passing the file group (`p$fgrp`) to determine the database file.
- **Data Structures**: Includes time (`t#`) and date (`d#`) conversion structures, used to format the report’s date (`t#mdcy`) and time (`t#hms`) fields.

This program is a straightforward report generator, complementing the interactive maintenance and inquiry functions of `BB928P`, `BB928`, and `BB9284` by providing a printed summary of all salesman ID records.