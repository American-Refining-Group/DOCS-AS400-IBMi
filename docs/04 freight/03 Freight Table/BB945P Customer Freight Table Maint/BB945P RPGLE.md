The RPG program `BB945P.rpgle` is designed to manage customer freight entries within a customer freight system. It provides an interactive interface for users to work with freight records, allowing operations such as viewing, adding, changing, copying, and deleting entries. Below is an explanation of the process steps, the external programs called, and the tables (files) used by the program.

### Process Steps of `BB945P.rpgle`

The program follows a structured workflow to handle customer freight entries through a subfile-based display interface. The main steps are:

1. **Initialization (`*inzsr` Subroutine)**:
   - Receives input parameters: `p$mode` (run mode: maintenance or inquiry) and `p$fgrp` (file group: 'Z' or 'G').
   - Sets global protection mode (`*in70`) based on `p$mode` ('MNT' for maintenance allows editing; otherwise, read-only).
   - Calls the external program `BB945C` to create a work file for `BB945P`.
   - Initializes work fields, subfile control fields, message handling fields, and key lists for file access.
   - Sets up the current date and time for use in filtering and record processing.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - Applies file overrides based on the file group (`p$fgrp`) to use either 'Z' or 'G' library files.
   - Opens input files (`arcust`, `bicont`, `bicuf1`, `gscntr1`, `gsprod`, `gstabl`, `inloc`, `shipto`, `bbcaid`) and update files (`bicuf2`, `bicuf3`, `bicufrh`) for data access.

3. **Process Subfile (`srsfl1` Subroutine)**:
   - **Clear Message Subfile**: Clears any existing messages to prepare for new user interactions.
   - **Initialize Subfile**: Sets initial subfile mode to folded (`sfmod1 = '1'`) and initializes control fields.
   - **Position File**: Calls `sf1rep` to position the file cursor based on user input filters (company, customer, ship-to, etc.).
   - **Main Loop**:
     - Displays the command line and message subfile.
     - Checks for existing subfile records to enable/disable display indicators (`*in41` for SFLDSP).
     - Determines folded/unfolded mode for subfile display (`*in45`).
     - Displays the subfile control format (`sflctl1`) using `EXFMT`.
     - Processes user input based on function keys or Enter:
       - **F03**: Exits the program.
       - **F04**: Prompts for field input (calls external programs for selection).
       - **F05**: Refreshes the subfile by repositioning (`sf1rep`).
       - **F08**: Toggles include/exclude expired/deleted entries filter (`w$expd`).
       - **F09**: Calls `GB730P` for history inquiry based on selected subfile record.
       - **F06**: Initiates adding a new freight entry by calling `BB945`.
       - **F10**: Positions the cursor to the control record.
       - **Enter**: Processes subfile changes (`sf1prc`).
     - Handles user repositioning requests by checking control fields (`c1cono`, `c1cust`, etc.) and repositioning if needed (`sf1rep`).
     - Loads subfile records (`sf1lod`) to populate the display.

4. **Reposition Subfile (`sf1rep` Subroutine)**:
   - Clears the subfile and resets the relative record number (`rrn1`).
   - Validates control fields (`sf1cte`) to ensure valid company, customer, ship-to, location, container code, carrier ID, carrier type, and product code.
   - Positions the file cursor using key lists (`kls1s1` or `kls1s2`) based on user input (e.g., company, customer, carrier ID).
   - Loads the subfile with records (`sf1lod`).
   - Retains control field values for subsequent repositioning.

5. **Edit Subfile Control Input (`sf1cte` Subroutine)**:
   - Validates input fields:
     - **Company (`c1cono`)**: Chains to `bicont` to verify and retrieve company name.
     - **Customer (`c1cust`)**: Chains to `arcust` to verify and retrieve customer name.
     - **Ship-to (`c1ship`)**: Ensures customer is specified before ship-to; chains to `shipto` for name.
     - **Location (`c1loc`)**: Chains to `inloc` for location name.
     - **Container Code/Type (`c1cncd`)**: Validates as container type (1 char) or code (>1 char) using `gstabl` or `gscntr1`.
     - **Carrier ID (`c1caid`)**: Chains to `bbcaid` for carrier name.
     - **Carrier Type (`c1cacd`)**: Chains to `gstabl` for description.
     - **Product Code (`c1prcd`)**: Chains to `gsprod` for description.
   - Sets error indicators (`*in50`–`*in58`) and adds error messages if validation fails.

6. **Load Subfile Records (`sf1lod` Subroutine)**:
   - Reads records from `bicuf1` or `bicuf3` based on user filters (company, customer, ship-to, carrier ID, etc.).
   - Filters out expired (`bfexdt < t#cymd`) or deleted (`bfdel = 'D'`) records unless `w$expd` is on.
   - Applies additional filters for customer, ship-to, location, container code, carrier ID, carrier type, and product code.
   - Formats each subfile line (`sf1fmt`) and applies color coding (`sf1col`).
   - Writes records to the subfile and updates the relative record number (`rrn1`).

7. **Format Subfile Line (`sf1fmt` Subroutine)**:
   - Populates subfile fields (`s1sqky`, `s1exdt`, `s1cust`, `s1ship`, `s1tseq`, etc.) from the database record.
   - Retrieves ship-to name from `shipto` and container description from `gstabl` or `gscntr1`.
   - Formats start and end dates (`bfstdt`, `bfexdt`) into MMDDYY format.
   - Sets the `s1more` indicator if more than four product codes exist.
   - Calls `calcfrt` to calculate freight totals (`s1$btot`, `s1$ctot`).
   - Sets protection mode (`*in70`) based on `p$mode`.

8. **Calculate Freight (`calcfrt` Subroutine)**:
   - Initializes `FRGHT` and `FRTTBL` data structures with record values (company, customer, ship-to, carrier ID, etc.).
   - Sets default values for freight calculation (e.g., quantity, pickup date).
   - Calls `MBBFR1` to compute freight totals.
   - Stores billed (`f$famt`) and carrier (`f$cfamt`) freight amounts in subfile fields (`s1$btot`, `s1$ctot`).

9. **Color Coding (`sf1col` Subroutine)**:
   - Applies color indicators to subfile records:
     - Red (`*in71`): Expired records (`s1exdt < t#cymd`).
     - Turquoise (`*in72`): Deleted records (`s1del = 'D'`).
     - Blue (`*in73`): Expired but not deleted records.

10. **Process Subfile on Enter (`sf1prc` Subroutine)**:
    - Reads changed subfile records using `READC`.
    - Processes user-selected options (`s1opt`):
      - **Option 2**: Change record (`sf1s02`).
      - **Option 3**: Copy record (`sf1s03`).
      - **Option 4**: Delete record (`sf1s04`).
      - **Option 5**: Display customer order (`sf1s05`).
      - Updates tender sequence (`s1tseq`) if changed, calls `bicuf2Upd`, and writes to history file (`writehist`).
    - Updates the subfile record after processing.

11. **Subfile Option Processing**:
    - **Change (`sf1s02`)**:
      - Checks if the record is not deleted.
      - Calls `BB945` in maintenance mode with the selected sequence key (`s1sqky`).
      - Displays a confirmation message if successful.
    - **Copy (`sf1s03`)**:
      - Calls `BB945` with the source sequence key (`s1sqky`) for copying.
      - Displays a confirmation or error message based on the return flag.
    - **Delete (`sf1s04`)**:
      - Calls `BB946` to mark the record as deleted or reactivated.
      - Displays a confirmation message based on the return flag ('D' or 'A').
    - **Display (`sf1s05`)**:
      - Calls `BB945` in inquiry mode to display the selected record.

12. **Update `bicuf2` (`bicuf2Upd` Subroutine)**:
    - Updates the tender sequence (`bftseq`) in `bicuf2` if changed.
    - Saves the record to a data structure (`svds`) and updates the file.

13. **Write History (`writehist` Subroutine)**:
    - Writes a history record to `bicufrh` with the current timestamp and user ID.

14. **Field Prompting (`prompt` Subroutine)**:
    - Calls external programs to prompt for field values (e.g., customer, ship-to, location) based on cursor position:
      - `LCSTSHP` for customer/ship-to (`c1cust`, `c1ship`).
      - `LINLOC` for location (`c1loc`).
      - `LGSCNCD` for container code (`c1cncd`).
      - `LBBCAID` for carrier ID (`c1caid`).
      - `LGSTABL` for carrier type (`c1cacd`).
      - `LGSPROD` for product code (`c1prcd`).
    - Updates control fields with selected values.

15. **Message Handling**:
    - **Add Message (`addmsg`)**: Sends error or confirmation messages to the program message queue.
    - **Write Message (`wrtmsg`)**: Displays messages in the message subfile.
    - **Clear Message (`clrmsg`)**: Clears the message subfile.

16. **Program End**:
    - Calls `BB9451` to print invalid freight GL accounts.
    - Closes all files and terminates the program.

### External Programs Called

The program interacts with the following external programs:
1. **BB945C**: Creates a work file for `BB945P` (called during initialization and before program end).
2. **BB9451**: Prints invalid freight GL accounts (called at program end).
3. **BB945**: Handles add, change, copy, and display operations for freight entries.
4. **BB946**: Manages deletion/reactivation of freight entries.
5. **GB730P**: Displays history inquiry for a selected freight entry.
6. **MBBFR1**: Calculates freight totals (billed and carrier amounts).
7. **LCSTSHP**: Prompts for customer and ship-to selection.
8. **LINLOC**: Prompts for location selection.
9. **LGSCNCD**: Prompts for container code selection.
10. **LBBCAID**: Prompts for carrier ID selection.
11. **LGSTABL**: Prompts for carrier type selection.
12. **LGSPROD**: Prompts for product code selection.
13. **QMHSNDPM**: Sends messages to the program message queue.
14. **QMHRMVPM**: Removes messages from the program message queue.
15. **QCMDEXC**: Executes file override commands.

### Tables (Files) Used

The program uses the following files, with overrides applied based on the file group (`p$fgrp`):
1. **bb945pd**: Display file (workstation file with subfile `sfl1`).
2. **arcust**: Customer master file (input only).
3. **bicont**: Company master file (input only).
4. **bicuf1**: Freight file (input only, replaces `bicufr`).
5. **bicuf2**: Freight file (update, renamed record format `bicufrp2`).
6. **bicuf3**: Freight file (input only, renamed record format `bicufrp3`).
7. **bicufrh**: Freight history file (output only).
8. **gscntr1**: Container code file (input only, replaces `gscntr`).
9. **gsprod**: Product code file (input only).
10. **gstabl**: Table file for container types, carrier types, etc. (input only).
11. **inloc**: Location master file (input only).
12. **shipto**: Ship-to master file (input only).
13. **bbcaid**: Carrier ID file (input only).
14. **gscont**: Container file (input only, fixed-length format).

### Summary

The `BB945P` program provides a comprehensive interface for managing customer freight entries, supporting operations like viewing, adding, modifying, copying, and deleting records. It uses a subfile to display records, validates user input, applies filters, and calculates freight totals. The program integrates with multiple external programs for specific tasks (e.g., prompting, history inquiry, freight calculation) and accesses various files for data retrieval and updates. The revisions indicate ongoing enhancements, such as adding freight calculations, fixing record locking issues, and updating file usage.