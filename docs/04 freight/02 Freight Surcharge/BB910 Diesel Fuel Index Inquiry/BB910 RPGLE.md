The RPG program **BB910** is designed for the maintenance and inquiry of the National Diesel Fuel Index within the Brandford Order Entry/Invoices system. It uses subfiles (SFL1 and SFL2) to display and manage data interactively, supporting both inquiry (INQ) and maintenance (MNT) modes. Below is a detailed explanation of the process steps, followed by a list of external programs called and tables used.

---

### Process Steps of the BB910 RPG Program

1. **Program Initialization (*INZSR)**:
   - **Receives Entry Parameters**:
     - `a$co`: Company code (2 characters).
     - `p$mode`: Run mode, either 'MNT' (maintenance) or 'INQ' (inquiry).
     - `p$fgrp`: File group, either 'Z' or 'G', determining which file overrides to apply.
   - **Sets Initial Values**:
     - Moves the company code to the control field `c1co` if valid.
     - Initializes subfile relative record numbers (`rrn1`, `rrn2`, `rrnsv1`, `rrnsv2`) to zero.
     - Sets subfile page sizes (`pagsz1`, `pagsz2`) to 28 records.
     - Configures the program header (`c$hdr1`) based on the mode ('Maintenance' or 'Inquiry').
     - Sets global protection indicator (*IN70) based on mode (on for INQ, off for MNT).
     - Initializes date and time fields using the system date (`ps#mdy`) and time (`time12`).

2. **Open Database Tables (opntbl)**:
   - Applies file overrides based on the file group (`p$fgrp`):
     - For 'G', overrides point to `gbicont`, `ggstabl`, `gbbndfi`, `gbbndfi1`, `gbbndfih`.
     - For 'Z', overrides point to `zbicont`, `zgstabl`, `zbbndfi`, `zbbndfi1`, `zbbndfih`.
   - Executes the `QCMDEXC` system command to apply overrides.
   - Opens files: `bicont`, `gstabl`, `bbndfi`, `bbndfi1`, `bbndfird`, and `bbndfih`.

3. **Process Subfile 1 (srsfl1)**:
   - **Clear Message Subfile**: Calls `clrmsg` to clear any existing messages and `wrtmsg` to display the message subfile.
   - **Default Company**: Sets `c1co` to 10 if zero.
   - **Initial Display**:
     - Clears error indicators and suppresses errors on the first display (`w$frst`).
     - Repositions the subfile (`sf1rep`) to load initial records.
   - **Main Loop**:
     - Writes the command line (`sflcmd1`) and displays the message subfile if needed.
     - Checks if subfile records exist to enable display control (`*IN41`).
     - Displays the subfile control record (`sflctl1`) using `exfmt`.
     - Determines cursor location (`csrloc`) and sets the subfile record number (`rcdnb1`) to the lowest displayed record (`pagrrn`).
     - Processes user input:
       - **F03 (Exit)**: Exits the loop and program.
       - **F05 (Refresh)**: Resets positioning fields (`r$emdy`, `c1emdy`, `d1opt`, `d1emdy`, `d1eftm`) and repositions the subfile.
       - **Direct Access**: If `d1opt`, `d1emdy`, or `d1eftm` is non-zero, calls `sf1dir` for direct processing.
       - **Page Down**: Loads more subfile records (`sf1lod`).
       - **Enter**: Processes subfile changes (`sf1prc`).
       - **User Positioning**: If the company (`c1co`) or effective date (`c1emdy`) changes, repositions the subfile.
       - **F10**: Clears cursor position (`row1`, `col1`) to reposition to the control record.
   - Continues until the user exits (`sf1agn = *off`).

4. **Subfile 1 Processing (sf1prc)**:
   - Reads changed subfile records (`readc sfl1`) and processes them (`sf1chg`).
   - **sf1chg**:
     - Copies selected values (`s1efdt`, `s1eftm`, `s1emdy`) to control fields (`c2efdt`, `c2eftm`, `c2emdy`).
     - Processes options:
       - Option 2 (Change): Calls `sf1s02` for maintenance mode.
       - Option 5 (Display): Calls `sf1s05` for inquiry mode.
     - Updates the subfile record after processing, clearing the option field and updating occurrence count (`s1occr`).

5. **Subfile 1 Repositioning (sf1rep)**:
   - Clears the subfile (`sf1clr`) and resets `rrn1` and `rrnsv1`.
   - Validates control input (`sf1cte`):
     - Checks if the company (`c1co`) exists in `bicont`. If not, sets error `ERR0021`.
     - Validates the effective date (`c1emdy`) using `GSDTEDIT`. If invalid, sets error `ERR0020`.
   - If no errors, positions the file (`bbndfi1`) using `kls1s1` (company and effective date) and loads the subfile (`sf1lod`).

6. **Subfile 1 Loading (sf1lod)**:
   - Sets the relative record number (`rrn1`) to the last saved number (`rrnsv1`).
   - Loads up to `pagsz1` (28) records:
     - Reads records from `bbndfi1` using `kls1r1` or `klnxt1`.
     - Formats each record (`sf1fmt`) with effective date (`s1efdt`), time (`s1eftm`), and occurrence count (`s1occr`).
     - Writes the record to `sfl1` and increments `rrn1`.
   - Sets the subfile end indicator (`*IN43`) if no more records are available.
   - Saves the last record number (`rrnsv1`).

7. **Direct Access Processing (sf1dir)**:
   - Validates the effective date (`d1emdy`) using `GSDTEDIT` and time (`d1eftm`) for valid hours and minutes.
   - Checks for required values in create mode (`d1opt = 1`). Sets errors if missing.
   - Verifies record existence in `bbndfi`:
     - For non-create options, errors if the record doesn’t exist (`ERR0102`).
     - For create, errors if the record already exists (`ERR0101`).
   - If no errors, processes options:
     - Option 1 (Create): Calls `sf1s01`.
     - Option 2 (Change): Calls `sf1s02`.
     - Option 5 (Display): Calls `sf1s05`.
   - Clears option and cursor fields if successful.

8. **Subfile 1 Options**:
   - **sf1s01 (Create)**: Saves indicators and subfile state, sets `s2mode` to 'MNT', calls `srsfl2`, and triggers repositioning.
   - **sf1s02 (Change)**: Similar to `sf1s01`, but for updating existing records.
   - **sf1s05 (Display)**: Sets `s2mode` to 'INQ' and calls `srsfl2` for inquiry.

9. **Process Subfile 2 (srsfl2)**:
   - Initializes the subfile based on mode:
     - In 'MNT', sets `s2all` to on and `F6=Review Mode`.
     - In 'INQ', sets `s2all` to off and `F6=All Mode`.
   - Retrieves the occurrence count (`rtvent#`) and sets `c2occr`.
   - Clears and repositions the subfile (`sf2rep`).
   - **Main Loop**:
     - Displays the command line (`sflcmd2`) and message subfile.
     - Checks for subfile records to enable display (`*IN41`).
     - Displays the subfile control record (`sflctl2`).
     - Processes user input:
       - **F03 (Exit)**: Exits both subfiles and the program.
       - **F05 (Refresh)**: Triggers repositioning.
       - **F06 (Toggle Mode)**: Switches between All and Review modes, updating the F6 label.
       - **F09 (History Inquiry)**: Calls `GB730P` with parameters for history inquiry.
       - **F12 (Cancel)**: Exits the subfile loop.
       - **Page Down**: Loads more records (`sf2lod`).
       - **Enter**: Processes subfile changes (`sf2prc`).
       - **User Positioning**: If `c2regn` is non-zero, repositions the subfile.
       - **F10**: Clears cursor position (`row2`, `col2`).

10. **Subfile 2 Processing (sf2prc)**:
    - Reads changed subfile records (`readc sfl2`) and processes them (`sf2chg`).
    - **sf2chg**:
      - Edits input (`sf2edt`).
      - Clears errors in inquiry mode.
      - Updates the database (`sf2upd`) if no errors and in maintenance mode.
      - Updates the subfile record (`sf2pro`).

11. **Subfile 2 Update (sf2upd)**:
    - Checks if the record exists in `bbndfi`:
      - **Create**: If no record exists and `s2rprc` is non-zero, writes a new record to `bbndfpf` with company, effective date, time, region, and price.
      - **Update**: If the record exists and `s2rprc` is non-zero, updates the price (`bnrprc`).
      - **Delete**: If `s2rprc` is zero, deletes the record and marks it in the history file.
    - Writes history record (`writehist`) for create, update, or delete actions.
    - Updates the occurrence count (`c2occr`).

12. **Subfile 2 Repositioning (sf2rep)**:
    - Clears the subfile (`sf2clr`) and resets `rrn2` and `rrnsv2`.
    - Validates control input (`sf2cte`).
    - Positions the file (`gstabl` or `bbndfird`) based on All or Review mode.
    - Loads the subfile (`sf2lod`).

13. **Subfile 2 Loading (sf2lod)**:
    - Loads up to `pagsz2` (28) records:
      - In All mode, reads from `gstabl` using `kls2a2`.
      - In Review mode, reads from `bbndfird` using `kls2r2`.
      - Formats records (`sf2fmt`) with region (`s2regn`), description (`s2desc`), and price (`s2rprc`).
      - Writes to `sfl2` and increments `rrn2`.
    - Sets the subfile end indicator (`*IN43`) if no more records are available.

14. **Message Handling**:
    - **addmsg**: Sends error messages to the program message queue using `QMHSNDPM`.
    - **wrtmsg**: Writes the message subfile (`msgctl`).
    - **clrmsg**: Clears the message subfile using `QMHRMVPM`.

15. **History Writing (writehist)**:
    - Clears the history record (`bbndfihpf`).
    - Sets fields: deletion flag (`lhdel`), company (`lhco`), effective date (`lhefdt`), time (`lheftm`), region (`lhregn`), price (`lhrprc`), date (`lhchd8`), time (`lhchtm`), and user (`lhuser`).
    - Writes the record to `bbndfih`.

16. **Program Termination**:
    - Closes all files.
    - Sets `*INLR` to `*ON` and returns.

---

### External Programs Called

1. **GSDTEDIT**:
   - Purpose: Validates the effective date (`p#mdy`) and returns a converted date (`p#cymd`) or an error flag (`p#err`).
   - Called in: `sf1dir` and `sf1cte`.

2. **GB730P**:
   - Purpose: Displays history inquiry for a selected record.
   - Parameters: `x$hist` data structure (file, file group, company, effective date, time, region).
   - Called in: `srsfl2` when F09 is pressed.

3. **QCMDEXC**:
   - Purpose: Executes file override commands (`ovg` or `ovz`).
   - Called in: `opntbl`.

4. **QMHSNDPM**:
   - Purpose: Sends error messages to the program message queue.
   - Called in: `addmsg`.

5. **QMHRMVPM**:
   - Purpose: Clears the message subfile.
   - Called in: `clrmsg`.

---

### Tables (Files) Used

1. **bb910d**:
   - Type: Display file (workstation).
   - Usage: Defines the user interface with subfiles `sfl1` and `sfl2`, and control records `sflctl1` and `sflctl2`.
   - Access: Input/Output.

2. **bicont**:
   - Type: Input file (database).
   - Usage: Stores company information, used to validate `c1co`.
   - Access: Read-only, user-opened.

3. **gstabl**:
   - Type: Input file (database).
   - Usage: Stores region codes and descriptions for All mode display in `s unofficially.
   - Access: Read-only, user-opened.

4. **bbndfi**:
   - Type: Update file (database).
   - Usage: Primary file for storing National Diesel Fuel Index data (company, effective date, time, region, price).
   - Access: Read/Write/Add, user-opened.

5. **bbndfi1**:
   - Type: Input file (database).
   - Usage: Renamed format of `bbndfi` for subfile 1 processing.
   - Access: Read-only, user-opened.

6. **bbndfird**:
   - Type: Input file (database).
   - Usage: Renamed format of `bbndfi` for subfile 2 processing.
   - Access: Read-only, user-opened.

7. **bbndfih**:
   - Type: Output file (database).
   - Usage: Stores history records for create, update, and delete actions.
   - Access: Write-only, user-opened.

---

### Summary

The **BB910** program provides an interactive interface for managing the National Diesel Fuel Index, allowing users to create, update, or delete records in maintenance mode, or view them in inquiry mode. It uses two subfiles (`sfl1` for effective date/time, `sfl2` for region/price) with navigation via function keys (F03, F05, F06, F09, F10, F12, Page Down, Enter). It validates inputs, manages database updates, logs history, and handles errors with a message subfile. The program interacts with multiple database files and calls external programs for date validation, history inquiry, and system commands.