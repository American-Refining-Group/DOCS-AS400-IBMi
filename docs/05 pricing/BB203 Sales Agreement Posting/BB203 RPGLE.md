The `BB203.rpgle` program is an RPGLE (RPG IV) program for the IBM System i (AS/400) environment, called from the main OCL script (`BB203.ocl36.txt`). Its purpose is to automate the posting of customer sales agreement pricing by processing records from a temporary work file, updating or expiring existing pricing records, creating new ones, and logging historical data. Below, I’ll explain the process steps, business rules, tables used, and external programs called.

### Process Steps of the BB203 RPGLE Program

The program orchestrates a sequence of operations to validate input, process pricing records, and produce output. Here’s a step-by-step breakdown of the process, based on the provided code:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Parameter Reception**:
     - Accepts a 1-character parameter `p$fgrp` (file group, e.g., 'G' or 'Z'), which determines file overrides.
   - **File and Printer Setup**:
     - Calls `openprtf` to override and open the printer file `QSYSPRT` for output with specific formatting (e.g., page size, lines per inch).
     - Calls `opntbl` to apply database file overrides (based on `p$fgrp`) and open files `BICUAG`, `BB203W`, `BICONT`, and `BICUAGH`.
   - **Message Handling**:
     - Initializes message handling fields (e.g., `m@pgmq`, `m@key`) for error messaging via a subfile.
   - **Date and Time Setup**:
     - Captures the current date and time into `t#time` and formats it into `t#cymd` (CCYYMMDD).
   - **Miscellaneous**:
     - Sets `fmtagn` to `*ON` to control the display loop.
     - Defines work fields and clears the message subfile via `clrmsg`.

2. **Main Processing Loop (`Process` Subroutine)**:
   - **Display Screen**:
     - Enters a loop (`fmtagn = *ON`) to display the `pstscn` screen format (via `EXFMT`) for user interaction.
     - If messages are pending (`dspmsg = *ON`), writes the message subfile (`wrtmsg`); otherwise, clears the screen (`clrscr`).
   - **Handle Function Keys**:
     - If F12 is pressed, exits the loop and terminates the program.
   - **Validate Start Date**:
     - Calls `editstrdate` to validate the user-entered start date (`s1stdt`).
     - Converts the date to `p#cymd` (CCYYMMDD format) if valid; otherwise, sets error indicators (`*IN50`, `*IN51`, `*IN53`) and adds an error message (`ERR0020`) to the message subfile.
     - Ensures the start date matches the start date in the work file `BB203W` (field `new_banstd`). If mismatched, sets an error (`ERR0000`) with the expected start date.
   - **Calculate Expiry Date**:
     - If the start date is valid, computes the expiry date for existing records as one day before the new start date using the `GSDTCLC1` program (via `pldtclc1` parameter list).
     - Stores the result in `w1endcymd` (CCYYMMDD) and `w1endymd` (YYMMDD).
   - **Post Records**:
     - If F09 is pressed and no errors exist, calls the `postit` subroutine to process records.

3. **Posting Records (`postit` Subroutine)**:
   - **Read Work File**:
     - Reads all records from `BB203W` sequentially (`setll *loval`, `read`, `dow not %eof`).
   - **Update/Expire Existing Record**:
     - For each record, chains to `BICUAG` using keys `new_bacono` (company) and `new_baseqn` (sequence number).
     - If a matching record is found (`%found`):
       - Saves old values (price, off-price, start/end dates, contract number) for history and printing.
       - Checks if the existing record’s expiry date (`baend8`) is greater than the new start date (`w1strcymd`) or is zero (per `jk03` revision).
       - If true, sets the expiry date to one day before the new start date (`w$endt`, `w1endcymd`, time 23:59) and updates `BICUAG`.
       - Writes the expired record to the history file `BICUAGH` via `writehistorg`.
     - If no match is found, clears old values and sets `*IN25` to `*OFF` for reporting.
   - **Create New Price Record**:
     - Calls `rtvnxtsqky` to retrieve and increment the sequence number from `BICONT` for the new record’s key (`w$sq`).
     - Chains to `BICUAG` with the new sequence number to ensure no conflict.
     - If no existing record is found, populates a new `BICUAG` record with fields from `BB203W` (e.g., `new_badel`, `new_bacono`, `new_bacust`, etc.).
     - Sets the start date (`bastdt`, `bastd8`, time 00:01) and clears expiry date fields (`baendt`, `baend8`, `baentm = 0`).
     - Writes the new record to `BICUAG` and the history file `BICUAGH` via `writehist`.
   - **Print Output**:
     - Calls `ovrflo` to handle printer overflow and prints a detail line (`except dtl01`) with old and new pricing data.
   - **Loop Completion**:
     - Exits the loop when all `BB203W` records are processed and sets `fmtagn` to `*OFF` to exit the display loop.

4. **Retrieve Next Sequence Number (`rtvnxtsqky` Subroutine)**:
   - Chains to `BICONT` using `new_bacono` to retrieve the last sequence number (`bcseqn`).
   - Increments it by 1 (`w$sq`) and updates `BICONT` with the new sequence number.
   - Handles errors if no record is found (`*IN99`).

5. **Write to History File (`writehist` and `writehistorg` Subroutines)**:
   - **writehist**:
     - Clears the `BICUAGH` record and populates it with fields from the new `BB203W` record (e.g., `hhdel`, `hhcono`, `hhcust`, etc.).
     - Writes the record to the history file `BICUAGH`.
   - **writehistorg** (not fully shown but referenced in `jk03`):
     - Writes the expired `BICUAG` record to `BICUAGH` when an existing record is updated with a new expiry date.

6. **Validate Start Date (`editstrdate` Subroutine)**:
   - Validates the user-entered start date (`s1stdt`) for correct format (MM/DD/CC/YY) and logical validity (e.g., valid month, day, leap year checks).
   - Converts the date to `p#cymd` (CCYYMMDD) if valid.
   - Compares it with the start date in `BB203W` (`new_banstd`) to ensure consistency.
   - Sets error indicators (`*IN50`, `*IN51`) and adds messages (`ERR0000`, `ERR0020`) if invalid or mismatched.

7. **Message Handling (`addmsg`, `wrtmsg`, `clrmsg` Subroutines)**:
   - **addmsg**: Sends error messages to the program message queue using `QMHSNDPM` with message ID, data, and type.
   - **wrtmsg**: Displays the message subfile (`msgctl`) when errors exist.
   - **clrmsg**: Clears the message subfile using `QMHRMVPM` and restores the screen state.

8. **Printer Overflow (`ovrflo` Subroutine)**:
   - Checks the overflow indicator (`*INOF`) and prints header lines (`except hdr01`) when needed.
   - Resets the overflow flag (`prtovr`).

9. **Program Termination**:
   - Closes all files (`close *all`).
   - Sets `*INLR` to `*ON` and returns, ending the program.

### Business Rules

The program enforces the following business rules:
- **Data Validation**:
  - The start date entered by the user must match the start date in `BB203W` (`new_banstd`). If not, an error (`ERR0000`) is displayed.
  - The start date must be in a valid format (MM/DD/CC/YY) and pass logical checks (e.g., valid days per month, leap year rules).
- **Expiry Logic (`jk03` Revision)**:
  - An existing `BICUAG` record is only expired if its expiry date (`baend8`) is greater than the new record’s start date (`w1strcymd`) or is zero.
  - The expiry date for existing records is set to one day before the new start date (at 23:59).
- **Sequence Number Management**:
  - New records use a unique sequence number (`w$sq`) retrieved and incremented from `BICONT` for the company (`new_bacono`).
- **History Tracking**:
  - Both new and expired records are written to the history file `BICUAGH` for audit purposes.
- **File Overrides**:
  - The file group parameter (`p$fgrp`, 'G' or 'Z') determines which database files (`GBICUAG` or `ZBICUAG`, etc.) are used via overrides.
- **Error Handling**:
  - Errors (e.g., invalid dates, missing records) are displayed via a message subfile and prevent posting until corrected.
- **Printing**:
  - Outputs a report to `QSYSPRT` with details of old and new pricing, including company, customer, location, pricing codes, prices, and sequence numbers.

### Tables (Files) Used

The program interacts with the following files:
1. **BICUAG** (Update, Keyed, `uf a e k disk usropn`):
   - Primary file for customer sales agreement pricing records.
   - Renamed to `bicuagrd` internally.
   - Updated with new expiry dates for existing records and written with new pricing records.
2. **BB203W** (Input, Keyed, `if e k disk usropn prefix(new_)`):
   - Temporary work file in `QTEMP` containing pricing records to be posted.
   - Fields are prefixed with `new_` (e.g., `new_bacono`, `new_banprc`).
3. **BICONT** (Update, Keyed, `uf a e k disk usropn`):
   - Stores the last sequence number (`bcseqn`) for each company (`new_bacono`).
   - Updated with incremented sequence numbers.
4. **BICUAGH** (Output, Keyed, `o e k disk usropn`):
   - History file for logging new and expired pricing records.
5. **BB203D** (Workstation, `cf e workstn infds(dspf_ds)`):
   - Display file for user interaction (screen format `pstscn`, message subfile `msgctl`).
6. **QSYSPRT** (Output, Printer, `o f 184 printer infds(prtf_opn) usropn oflind(*inof)`):
   - Printer file for generating reports with headers (`hdr01`) and detail lines (`dtl01`).

### External Programs Called

The program calls the following external program:
1. **GSDTCLC1**:
   - Called via the `pldtclc1` parameter list to calculate the date one day before the new start date.
   - Parameters:
     - `p#dat1`: Input date (CCYYMMDD).
     - `p#dat2`: Output date (CCYYMMDD, result of subtraction).
     - `p#fmt`: Format ('D' for days).
     - `p#diff`: Difference (-1 for one day prior).
     - `p#err`: Error flag ('Y' if an error occurs).

### Additional Notes

- **Revisions**:
  - `JB01` (12/03/2012): Renamed work file, replacing `IAGNEW` with `BB203W`.
  - `JK01` (12/10/2012): Added logic to retrieve the next sequence number from `BICONT`.
  - `JK03` (08/07/2014): Added conditional expiry logic and history file writes for expired records.
  - Field renames (10/01/2013): Changed `BAMNGL`/`BAMXGL` to `BAMNQY`/`BAMXQY`.
- **Potential Issues**:
  - The hardcoded file name `BB203W` in `BB203AC` (from the previous CLP) may conflict with the dynamic naming in the OCL script (`?9?BB203?WS?`). This program assumes `BB203W` in `QTEMP`, aligning with the OCL’s `CRTDUPOBJ`.
  - The `writehistorg` subroutine is referenced but not fully shown in the truncated code, but its purpose is clear from the `jk03` revision.

### Summary

The `BB203` RPGLE program automates the posting of customer sales agreement pricing by:
- Validating user-entered start dates against `BB203W`.
- Expiring existing `BICUAG` records if their expiry date is greater than the new start date or zero.
- Creating new `BICUAG` records with incremented sequence numbers from `BICONT`.
- Logging new and expired records to `BICUAGH`.
- Generating a printed report via `QSYSPRT`.
- Handling user interaction and errors via a display file and message subfile.

**Tables Used**: `BICUAG`, `BB203W`, `BICONT`, `BICUAGH`, `BB203D` (display), `QSYSPRT` (printer).
**External Programs Called**: `GSDTCLC1`.