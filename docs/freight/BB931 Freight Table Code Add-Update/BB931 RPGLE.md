The provided document is an RPGLE (RPG IV) program named `BB931`, designed for Flat Freight Code Maintenance/Inquiry within the Brandford Order Entry/Invoices system. Below, I’ll explain the process steps of the program, list the external programs called, and identify the tables (files) used. The explanation will focus on the program's high-level flow and key operations, maintaining clarity and conciseness.

### Process Steps of the BB931 RPGLE Program

The `BB931` program is a maintenance and inquiry application for managing flat freight codes. It uses a display file with subfiles to interact with users, allowing them to view, add, update, or delete freight code records. The program operates in two modes: Maintenance (`MNT`) or Inquiry (`INQ`), determined by an input parameter. Here’s a breakdown of the process steps based on the program’s structure:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Sets up the program environment.
   - **Actions**:
     - Receives input parameters: `p$mode` (run mode: `MNT` or `INQ`) and `p$fgrp` (file group: `Z` or `G`).
     - Initializes subfile control fields (`rrn1`, `rrnsv1`), page size (`pagsz1 = 30`), and various flags (e.g., `w$frst`, `dspmsg`).
     - Defines key lists (`kls1s1`, `kls1r1`, `klsfl1`, `klfrum`) for file access.
     - Sets default values for message handling and cursor positioning.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Purpose**: Opens required files with appropriate overrides based on the file group (`G` or `Z`).
   - **Actions**:
     - Executes override commands (`ovrdbf`) stored in arrays `ovg` or `ovz` to point to the correct files (`gbicont`, `ggstabl`, `gbbfrtb` for `G`; `zbicont`, `zgstabl`, `zbbfrtb` for `Z`).
     - Opens files: `bicont` (input), `gstabl` (input), `bbfrtbrd` (input, renamed record format), and `bbfrtb` (update/add).
     - Uses `QCMDEXC` to execute override commands dynamically.

3. **Process Subfile (`srsfl1` Subroutine)**:
   - **Purpose**: Manages the main user interface loop, displaying and processing a subfile (`sfl1`) for freight code records.
   - **Actions**:
     - **Header and Protection Setup**:
       - Sets header (`c$hdr1`) based on mode (`MNT`: "Flat Freight Code Maintenance"; `INQ`: "Flat Freight Code Inquiry").
       - Enables/disables field protection (`*IN71`) based on mode (`ON` for `INQ`, `OFF` for `MNT`).
     - **Message Subfile Initialization**:
       - Clears the message subfile (`clrmsg`) and writes it (`wrtmsg`) to ensure no residual messages.
     - **Subfile Initialization**:
       - Sets default company code (`c1cono = 10`) and initializes subfile mode to `UPDATE` (`s1updt = *ON`, `s1f06d = "F6=Entry Mode"`, `c1mode = "Update Mode"`).
       - Repositions the subfile (`sf1rep`) to load initial records.
     - **Suppress Initial Errors**:
       - On the first display (`w$frst = *ON`), clears error indicators and messages to present a clean screen.
     - **Main Loop (`sf1agn`)**:
       - Displays the command line (`sflcmd1`) and message subfile if needed.
       - Checks if subfile records exist to set display control indicators (`*IN41` for `SFLDSP`).
       - Displays the subfile control format (`sflctl1`) using `EXFMT`.
       - Processes user input based on function keys and subfile changes:
         - **F03**: Exits the program.
         - **F04**: Prompts for field input (e.g., `S1FRUM`) by calling `MGSTABL`.
         - **F05**: Refreshes the subfile by clearing positioning fields and repositioning.
         - **F06**: Toggles between Update and Entry modes, updating labels (`s1f06d`, `c1mode`) and repositioning the subfile.
         - **F12**: Cancels the operation, exiting the loop.
         - **F23**: Initiates deletion of a subfile record.
         - **PAGEDN**: Loads additional subfile records (`sf1lod`).
         - **ENTER**: Processes subfile changes (`sf1prc`) or repositions based on user input (`c1cono`, `c1frcd`).
         - **F10**: Positions the cursor to the control record.
       - Handles cursor location (`csrloc`) and subfile record number (`rcdnb1`) for redisplay.

4. **Process Subfile on ENTER (`sf1prc` Subroutine)**:
   - **Purpose**: Processes changes in the subfile when the user presses ENTER.
   - **Actions**:
     - Reads changed subfile records (`readc sfl1`) and processes each (`sf1chg`).
     - Sets a flag (`s1chng`) if changes are detected.

5. **Process Subfile Record Change (`sf1chg` Subroutine)**:
   - **Purpose**: Validates and updates subfile record changes.
   - **Actions**:
     - Validates input (`sf1edt`).
     - In `INQ` mode, clears error indicators if errors exist.
     - If no errors, updates the database (`sf1upd`) in `MNT` mode.
     - Sets `SFLNXTCHG` (`*IN44`) if errors occur to allow correction.
     - Updates the subfile display (`sf1pro`, `update sfl1`).

6. **Edit Subfile Input (`sf1edt` Subroutine)**:
   - **Purpose**: Validates user input in the subfile.
   - **Actions**:
     - Clears error indicators (`*IN50`–`*IN69`).
     - Checks if the freight code (`s1frcd`) already exists in `bbfrtb` (`ERR0101` if it does and shouldn’t).
     - Ensures at least one value (`s1rate`, `s1frlb`, `s1frum`, `s1frpu`) is entered (`ERR0000` with message from `err(01)` if not).
     - Validates that unit of measure (`s1frum`) and rate (`s1frpu`) are both provided or neither (`ERR0000` with message from `err(02)` if inconsistent).
     - Verifies the unit of measure (`s1frum`) against `gstabl`, setting description (`s1umds`) or flagging an error (`ERR0010`) if invalid.

7. **Update Database (`sf1upd` Subroutine)**:
   - **Purpose**: Adds or updates records in `bbfrtb` based on subfile input.
   - **Actions**:
     - If the freight code (`s1frcd`) is non-blank, checks if it exists in `bbfrtb`.
     - For new records (`*IN99 = *ON`):
       - Clears the record format, sets company (`bfcono`) and freight code (`bffrcd`), moves subfile values (`sf1mov`), and writes to `bbfrtb`.
       - Marks the record as existing (`s1exis = 'Y'`).
     - For existing records:
       - Updates the record with subfile values (`sf1mov`) and marks as existing.

8. **Move Subfile Values to Table (`sf1mov` Subroutine)**:
   - **Purpose**: Transfers subfile fields to the database record.
   - **Actions**:
     - Moves `s1rate` to `bfrate`, `s1frlb` to `bffrlb`, `s1frum` to `bffrum`, and `s1frpu` to `bffrpu`.

9. **Delete Record (`sf1del` Subroutine)**:
   - **Purpose**: Handles deletion of a subfile record.
   - **Actions**:
     - Displays a confirmation window (`sfldel1`).
     - Processes user input:
       - **F12**: Cancels deletion.
       - **F23**: Deletes the record from `bbfrtb` if it exists and clears the subfile record.
     - Manages message display and clears the message subfile.

10. **Reposition Subfile (`sf1rep` Subroutine)**:
    - **Purpose**: Repositions the subfile based on user input (`c1cono`, `c1frcd`).
    - **Actions**:
      - Clears the subfile (`sf1clr`).
      - Validates control fields (`sf1cte`).
      - If no errors, positions the file (`bbfrtbrd`) using key list `kls1s1` and loads records (`sf1lod`).
      - Retains control fields for future repositioning.

11. **Edit Subfile Control Input (`sf1cte` Subroutine)**:
    - **Purpose**: Validates the company code in the control record.
    - **Actions**:
      - Checks `c1cono` against `bicont`, setting the company name (`c1conm`) or flagging an error (`ERR0011`) if invalid.

12. **Load Subfile Records (`sf1lod` Subroutine)**:
    - **Purpose**: Loads records into the subfile.
    - **Actions**:
      - Starts from the last subfile record number (`rrnsv1`).
      - Reads records from `bbfrtbrd` (using `kls1r1` in Update mode) up to the page size (`pagsz1`).
      - Formats each record (`sf1fmt`) and writes to `sfl1`.
      - Updates the relative record number (`rrn1`) and saves it (`rrnsv1`).

13. **Format Subfile Line (`sf1fmt` Subroutine)**:
    - **Purpose**: Formats a subfile record for display.
    - **Actions**:
      - Clears the subfile record.
      - If a record exists, populates fields (`s1frcd`, `s1rate`, `s1frlb`, `s1frum`, `s1frpu`) and sets `s1exis = 'Y'`.
      - Retrieves the unit of measure description (`s1umds`) from `gstabl`.
      - Applies protection schemes (`sf1pro`).

14. **Subfile Protection Schemes (`sf1pro` Subroutine)**:
    - **Purpose**: Sets field protection based on mode and record status.
    - **Actions**:
      - In `INQ` mode, protects all fields (`*IN76`–`*IN78`).
      - For existing records (`s1exis = 'Y'`), protects the key field (`*IN76`).

15. **Field Prompting (`prompt` Subroutine)**:
    - **Purpose**: Handles field prompting for `S1FRUM` when F04 is pressed.
    - **Actions**:
      - Calls `MGSTABL` to prompt for the unit of measure, updating `s1frum` if a value is returned.
      - Sets `*IN19` to indicate a change.

16. **Message Handling (`addmsg`, `wrtmsg`, `clrmsg` Subroutines)**:
    - **Purpose**: Manages error and informational messages.
    - **Actions**:
      - `addmsg`: Sends messages to the program message queue using `QMHSNDPM`.
      - `wrtmsg`: Displays the message subfile (`msgctl`).
      - `clrmsg`: Clears the message subfile using `QMHRMVPM`.

17. **Program Termination**:
    - **Purpose**: Cleans up and exits.
    - **Actions**:
      - Closes all files.
      - Sets `*INLR = *ON` and returns.

### External Programs Called

The program calls the following external programs:
1. **QCMDEXC**: Executes override commands to set up file mappings dynamically.
2. **QMHSNDPM**: Sends messages to the program message queue for display.
3. **QMHRMVPM**: Removes messages from the message queue.
4. **MGSTABL**: Called for field prompting (specifically for `S1FRUM`) to select a unit of measure.
5. **DTP010R**: Referenced in a parameter list (`pld010`) for date validation, but not explicitly called in the provided code.

### Tables (Files) Used

The program interacts with the following files:
1. **BB931D**: Display file (workstation file) with subfile `sfl1` for user interaction.
2. **BICONT**: Input-only file for company code validation (used to retrieve company name).
3. **GSTABL**: Input-only file for unit of measure validation and description retrieval.
4. **BBFRTBRD**: Input-only file (renamed record format `bbfrtbpr`) for reading freight code records.
5. **BBFRTB**: Update/add file for maintaining freight code records.

Depending on the file group (`p$fgrp` = `G` or `Z`), the files are overridden to:
- **G Group**: `gbicont`, `ggstabl`, `gbbfrtb`.
- **Z Group**: `zbicont`, `zgstabl`, `zbbfrtb`.

### Summary

The `BB931` program is a robust RPGLE application for managing flat freight codes, supporting both maintenance and inquiry modes. It uses a subfile-based interface to display, add, update, or delete records, with validation against company codes and units of measure. The program leverages external programs for system functions (e.g., message handling, command execution) and field prompting, and it interacts with multiple files to store and retrieve data. The structured use of subroutines and indicators ensures clear control flow and error handling, making it a typical example of an AS/400 (IBM i) interactive application.