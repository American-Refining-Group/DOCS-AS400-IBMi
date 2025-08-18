The RPG program `GS9295` is designed to print a product load listing within an IBM i (AS/400) environment. It is called from the main program `GS929P` (via the F15 function key) to generate a report of product load records from the `prdlodx` file, outputting the results to a printer file (`qsysprt`). The program does not have a user interface and focuses solely on reading database records and formatting them for printing. Below, I outline the process steps, business rules, database tables used, and external programs called.

---

### Process Steps of GS9295

The program follows a straightforward flow to produce a printed report of product load records. The steps are organized around the mainline logic and subroutines:

#### 1. **Initialization (*INZSR Subroutine)**
   - **Purpose**: Sets up initial parameters and variables.
   - **Steps**:
     - Receives a single input parameter: `p$fgrp` (file group, 'Z' or 'G').
     - Sets the report header (`c$hdr1`) to "Product/Load Listing" from the `hdr` array.
     - Sets the print overflow flag (`prtovr`) to `*on` to ensure the header is printed initially.
     - Calls the `opntbl` subroutine to open the database file.
     - Initializes data structures:
       - Time conversion (`t#time`) for date and time formatting.
       - Date conversion (`d#cymd`) for handling date fields.
       - Program status data structure (`psds##`) for job, user, and timestamp information.
       - Printer file data structure (`prtf_opn`) for printer file details.
     - Defines a string array (`str`) for formatting report lines (likely used for separators or labels).

#### 2. **Open Database Tables (opntbl Subroutine)**
   - **Purpose**: Opens the `prdlodx` file with the appropriate file override.
   - **Steps**:
     - Checks if `p$fgrp` is 'G' or 'Z'.
     - Applies file overrides from the `ovg` (for 'G') or `ovz` (for 'Z') array using the `QCMDEXC` system API to override `prdlodx` to `gprdlodx` or `zprdlodx`.
     - Opens the `prdlodx` file (input-only, `IF`, user-opened).

#### 3. **Print Listing (prtlist Subroutine)**
   - **Purpose**: Reads records from `prdlodx` and prints them to `qsysprt`.
   - **Steps**:
     - Calls `openprtf` to open the printer file.
     - Enters a loop until the last record is read (`*inlr = *on`):
       - Reads the next record from `prdlodx`.
       - If a record is found (`*inlr = *off`):
         - Checks for overflow and prints headers if needed (`ovrflo`).
         - Prints the detail line (`dtl01`) using the `EXCEPT` operation.
     - Calls `closprtf` to close the printer file.

#### 4. **Process Overflow (ovrflo Subroutine)**
   - **Purpose**: Manages page overflow and prints report headers.
   - **Steps**:
     - Checks if the overflow indicator (`*inof`) is on.
     - If overflow occurs:
       - Sets `prtovr` to `*on` to trigger header printing.
       - Sets indicators `*in81-*in85` to '1' for header formatting.
     - If `prtovr` is on:
       - Prints the header (`hdr01`) using `EXCEPT`.
       - Resets `prtovr` to `*off`.

#### 5. **Open Print File (openprtf Subroutine)**
   - **Purpose**: Opens the `qsysprt` printer file with overrides.
   - **Steps**:
     - Constructs an override command by concatenating `ovr(01)` and `ovr(02)` (defines page size, lines per inch, characters per inch, overflow line, output queue, form type, hold, and save options).
     - Executes the override using `QCMDEXC`.
     - Opens the `qsysprt` file (output, 164 characters wide, user-opened).

#### 6. **Close Print File (closprtf Subroutine)**
   - **Purpose**: Closes the `qsysprt` file and removes overrides.
   - **Steps**:
     - Closes the `qsysprt` file.
     - Executes the `DLTOVR` command from `ovr(03)` using `QCMDEXC` to delete the printer file override.

#### 7. **Program Termination**
   - Implicitly closes any open files and ends the program (no explicit `close *all` or `*inlr = *on` in the mainline, but standard RPG behavior applies).

---

### Business Rules

The program enforces the following business rules:

1. **File Group Handling**:
   - The file group (`p$fgrp`) determines whether the program accesses `gprdlodx` ('G') or `zprdlodx` ('Z'), ensuring the correct product load file is used.

2. **Report Content**:
   - The report includes all records from `prdlodx` without filtering (e.g., no exclusion of inactive or deleted records).
   - Each detail line (`dtl01`) includes:
     - `pdcono`: Company number.
     - `pdloc`: Loading location.
     - `pdprod`: Product code.
     - `pdcntr`: Container code.
     - `pdseq#`: Sequence number.
     - `pdracd`: Responsibility area code.
     - `pdmlcd`: Major location code.
     - `pdtype`: Product type.
     - `pdresp`: Responsible person.
     - `pdhazm`: Hazardous material code.
     - `pdprty`: Loading priority.
   - The header (`hdr01`) includes:
     - Company name ("American Refining Group").
     - Report title (`c$hdr1`: "Product/Load Listing").
     - Job name, program name, user, file group, page number, date, and time.

3. **Printer File Configuration**:
   - The report is formatted with a page size of 68 lines by 164 characters, 8 lines per inch, 15 characters per inch, and an overflow at line 62.
   - The output is sent to the job’s output queue (`OUTQ(*JOB)`), with the spool file held (`HOLD(*YES)`) and saved (`SAVE(*YES)`).

4. **No User Interaction**:
   - The program operates without a user interface, relying on the input parameter `p$fgrp` from the calling program (`GS929P`) and producing a report without requiring user input.

5. **No Error Handling**:
   - The program does not implement explicit error checking or messaging for database or printer operations, assuming the input parameter and file access are valid.

---

### Database Tables Used

The program interacts with the following database file:
1. **prdlodx**: Product load file (input-only, `IF`, user-opened).
   - Overrides to `gprdlodx` or `zprdlodx` based on `p$fgrp` ('G' or 'Z').

---

### External Programs Called

The program calls the following external program:
1. **QCMDEXC**: Executes file override commands for `prdlodx` (`opntbl`) and `qsysprt` (`openprtf`, `closprtf`).

---

### Summary

`GS9295` is a simple RPG program called by `GS929P` to generate a printed report of all product load records from the `prdlodx` file. It reads records sequentially, formats them into a detailed listing with headers, and outputs to the `qsysprt` printer file. The program enforces minimal business rules, primarily ensuring the correct file is accessed based on the file group and formatting the report with predefined attributes. It relies on `QCMDEXC` for file and printer overrides and assumes valid input from the calling program, with no user interaction or error handling.