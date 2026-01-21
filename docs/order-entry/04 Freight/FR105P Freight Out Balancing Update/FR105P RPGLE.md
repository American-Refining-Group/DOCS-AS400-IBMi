The RPG program `FR105P` is designed for managing freight out balancing customer invoices within a Traffic/Freight Invoicing system. It provides an interactive interface using a display file with a subfile to allow users to view, filter, and process freight invoice records. Below is a detailed explanation of the process steps, followed by a list of external programs called and database tables used.

---

### Process Steps of FR105P

1. **Program Initialization (`*inzsr` Subroutine)**:
   - Receives input parameters: `p$mode` (run mode: maintenance or inquiry) and `p$fgrp` (file group: 'Z' or 'G').
   - Initializes variables, including subfile control fields, message handling fields, and key lists.
   - Sets up work fields for date and time handling, using the current system date and time.
   - Defines key lists (`klsfl1`, `kls1s1`, etc.) for database access and sets default values for headers and function key labels.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - Applies file overrides based on the `p$fgrp` parameter ('G' or 'Z'), redirecting file access to appropriate libraries (e.g., `garcust` or `zarcust`).
   - Opens the input-only database files: `arcust`, `frbinh`, `frbinh1` (renamed record format), and `shipto`.

3. **Process Subfile (`srsfl1` Subroutine)**:
   - **Clear Message Subfile**: Clears any existing messages in the message subfile (`clrmsg`) and writes the message control (`wrtmsg`).
   - **Initialize Subfile Mode**: Sets the subfile to folded mode (`sfmod1 = '1'`, `*in45 = *on`) for the initial load.
   - **Initialize Control Fields**: Sets default values for subfile control fields (e.g., `c1co = 10`) and the include/exclude deleted entries filter (`w$del = *off`).
   - **Set Protection Mode**: Based on `p$mode`, enables (`*in70 = *on`) or disables (`*in70 = *off`) global input protection for inquiry or maintenance mode.
   - **Position File**: Calls `sf1rep` to position the file pointer based on user input or defaults.
   - **Main Loop**:
     - Displays the command line and message subfile.
     - Checks if subfile records exist to enable display (`*in41` for `SFLDSP`).
     - Sets folded/unfolded mode based on `sfmod1`.
     - Displays the subfile control (`sflctl1`) using `exfmt`.
     - Clears message subfile and format indicators after display.
     - Determines cursor location for the next display.
     - Initializes the subfile record number (`rcdnb1`) with the lowest subfile record number (`pagrrn`) for proper page redisplay.
     - Processes user input (function keys, enter, or repositioning):
       - **F03**: Exits the program.
       - **F04**: Prompts for field input (calls `prompt` subroutine).
       - **F05**: Refreshes the subfile by clearing positioning fields and repositioning (`repsfl = *on`).
       - **F08**: Toggles the include/exclude deleted entries filter (`w$del`) and updates the function key label (`s1f08d`).
       - **PAGEDN**: Loads additional subfile records (`sf1lod`).
       - **ENTER**: Processes subfile records (`sf1prc`).
       - **Repositioning**: If control fields (e.g., `c1co`, `c1rdno`) differ from saved values, repositions the subfile (`sf1rep`).
       - **F10**: Resets cursor to the control record.

4. **Process Subfile on ENTER (`sf1prc` Subroutine)**:
   - Reads changed subfile records (`readc sfl1`) if the subfile is not empty (`rrn1 > 0`).
   - Processes each changed record by calling `sf1chg`.

5. **Process Subfile Record Change (`sf1chg` Subroutine)**:
   - Saves selected subfile values (`s1co`, `s1rdno`, `s1srn`) into work fields (`s$co`, `s$ord#`, `s$srn`).
   - Constructs a descriptive string (`s$item`) for the selected entry.
   - Processes the subfile option (`s1opt`):
     - If `s1opt = 1`, calls `sf1s01` to handle the "Create" action.
   - Updates the subfile record:
     - Chains to `frbinh` using the key list `klsfl1` to retrieve the latest data.
     - Chains to the subfile record (`sfl1`) to update it.
     - Clears the option field (`s1opt`), formats the subfile line (`sf1fmt`), applies color coding (`sf1col`), and updates the subfile record.

6. **Reposition Subfile (`sf1rep` Subroutine)**:
   - Clears the subfile (`sf1clr`) and resets the relative record number (`rrn1`).
   - Validates subfile control input (`sf1cte`).
   - If no errors (`*in50 = *off`), positions the file pointer using `setll` on `frbinh1` and loads subfile records (`sf1lod`).
   - Saves control fields (`c1co`, `c1rdno`, etc.) for future repositioning.

7. **Edit Subfile Control Input (`sf1cte` Subroutine)**:
   - Currently commented out, but intended to validate the company number (`c1co`) against a control file (`prcontc`). If invalid, it would set an error message (`ERR0021`) and error indicators (`*in50`, `*in51`).

8. **Load Subfile Records (`sf1lod` Subroutine)**:
   - Sets the relative record number (`rrn1`) to the last saved value (`rrnsv1`) to append records.
   - Sets the subfile record number (`rcdnb1`) to ensure the new page is displayed correctly.
   - Loads up to `pagsz1` (32) records:
     - Reads the next record from `frbinh1` based on the company number (`c1co`) or sequentially.
     - Applies filters:
       - Skips deleted records (`bodel = 'D'`) if `w$del = *off`.
       - Skips records not matching user-specified filters (`c1co`, `c1rdno`, `c1cust`, `c1ship`, `c1invn`, `c1car`).
       - Skips customer-owned products (`bocoon = 'Y'`) [jb02].
       - Skips records with freight code 'C' (collect) and calculated freight 'N' (`bocafr = 'N'`) [jb03, jb04].
       - Applies open/closed filter (`c1clos`) [jb01].
     - Formats the subfile line (`sf1fmt`), applies color coding (`sf1col`), and writes the record to the subfile.
   - Saves the last relative record number (`rrnsv1`).

9. **Format Subfile Detail Line (`sf1fmt` Subroutine)**:
   - Clears the subfile record.
   - Populates subfile fields (`s1co`, `s1rdno`, `s1srn`, `s1cust`, `s1ship`, `s1invn`, `s1indt`, `s1shdt`, `s1car`, `s1caid`, `s1frcd`, `s1clos`, `s1del`) from the `frbinh` record.
   - Retrieves the customer name (`s1csnm`) from `arcust` using the key list `klcust`.
   - Retrieves the ship-to name (`s1shnm`) by calling `rtvshipto`.

10. **Retrieve Ship-to Name (`rtvshipto` Subroutine)**:
    - Handles different ship-to scenarios:
      - If `s1ship = 0`, uses the customer name (`arname`) from `arcust`.
      - If `s1ship < 900`, chains to `shipto` using `klshipcst` (company/customer/ship-to).
      - If `s1ship >= 900`, chains to `shipto` using `klship999` (company/customer/999) or falls back to `klshipcst` for `s1ship` between 900 and 998.
      - Sets the ship-to name (`s1shnm`) if the chain is successful.

11. **Subfile Color Coding (`sf1col` Subroutine)**:
    - Sets the color indicator (`*in72`) to blue for deleted records (`s1del = 'D'`).

12. **Clear Subfile (`sf1clr` Subroutine)**:
    - Resets `rrn1` and `rrnsv1` to zero.
    - Disables subfile display (`*in41`) and control (`*in40`), enables clear (`*in42`), writes the subfile control (`sflctl1`), and resets the clear indicator.

13. **Subfile Option 01 - Create (`sf1s01` Subroutine)**:
    - Saves the cursor location.
    - Calls the external program `FR105` with parameters for company, order, shipping reference, mode, file group, and a return flag.
    - If the return flag (`o$flag = '1'`) indicates success, sends a confirmation message (`ERR0000`) with the modified entry details and triggers a subfile reposition (`repsfl = *on`).

14. **Field Prompting (`prompt` Subroutine)**:
    - Handles prompting for customer (`c1cust`) or ship-to (`c1ship`) fields in the subfile control (`SFLCTL1`).
    - Calls `LCSTSHP` with parameters to retrieve customer and ship-to data.
    - Updates `c1cust` and `c1ship` if valid values are returned.
    - Sets `*in19` to indicate a panel format input change.

15. **Message Handling (`addmsg`, `wrtmsg`, `clrmsg` Subroutines)**:
    - **addmsg**: Sends a message to the program message queue using `QMHSNDPM`, setting the message ID, file, data, type, and other parameters.
    - **wrtmsg**: Writes the message subfile control (`msgctl`) with `*in49` enabled.
    - **clrmsg**: Clears the message subfile using `QMHRMVPM`, saving and restoring the current record format and page number.

16. **Program Termination**:
    - Closes all open files (`close *all`).
    - Sets the last record indicator (`*inlr = *on`) and returns.

---

### External Programs Called

1. **FR105**:
   - Called in the `sf1s01` subroutine to handle the "Create" action for a selected subfile record.
   - Parameters:
     - `o$co`: Company number.
     - `o$ord#`: Customer order number.
     - `o$srn`: Shipping reference number.
     - `o$mode`: Run mode ('MNT' for maintenance).
     - `o$fgrp`: File group ('Z' or 'G').
     - `o$flag`: Return flag indicating success ('1') or failure.

2. **LCSTSHP**:
   - Called in the `prompt` subroutine to provide prompting for customer and ship-to fields.
   - Parameters:
     - `x$cstshp`: Data structure containing company (`x$co`), search string (`x$srch`), customer (`x$cust`), ship-to (`x$ship`), flag (`x$flag`), and file group (`x$fgrp`).

3. **QCMDEXC**:
   - Called in the `opntbl` subroutine to execute file override commands (e.g., `ovrdbf`).
   - Parameters:
     - `dbov##`: Override command string (80 characters).
     - `dbol##`: Length of the command (15.5).

4. **QMHSNDPM**:
   - Called in the `addmsg` subroutine to send messages to the program message queue.
   - Parameters:
     - `m@id`: Message ID.
     - `m@msgf`: Message file name (`GSMSGF`).
     - `m@data`: Message data.
     - `m@l`: Message data length.
     - `m@type`: Message type (`*DIAG`).
     - `m@pgmq`: Program message queue (`*`).
     - `m@scnt`: Stack counter.
     - `m@key`: Message key.
     - `m@errc`: Error code.

5. **QMHRMVPM**:
   - Called in the `clrmsg` subroutine to clear messages from the program message queue.
   - Parameters:
     - `m@pgmq`: Program message queue.
     - `m@scnt`: Stack counter.
     - `m@rmvk`: Message key for removal.
     - `m@rmv`: Removal option (`*ALL`).
     - `m@errc`: Error code.

---

### Database Tables Used

1. **arcust**:
   - Input-only file containing customer master data.
   - Used to retrieve the customer name (`arname`) in the `sf1fmt` subroutine.
   - Keyed access using `klcust` (company and customer number).

2. **frbinh**:
   - Input-only file containing freight invoice header data.
   - Used to retrieve invoice details for subfile records.
   - Keyed access using `klsfl1` (company, order number, shipping reference number).

3. **frbinh1**:
   - Input-only file, a renamed version of `frbinh` (record format `frbinhp1`).
   - Used for reading invoice records during subfile loading (`sf1lod`) and positioning (`sf1rep`).
   - Keyed access using `kls1s1` or `kls1r3` (company number).

4. **shipto**:
   - Input-only file containing ship-to address data.
   - Used to retrieve the ship-to name (`csname`) in the `rtvshipto` subroutine.
   - Keyed access using `klshipcst` (company, customer, ship-to) or `klship999` (company, customer, 999).

---

### Additional Notes
- The program uses a display file (`fr105pd`) with a subfile (`sfl1`) and control (`sflctl1`) to present freight invoice data.
- Revisions (jb01–jb04) add filtering for open/closed status, skip customer-owned products, and handle freight code and calculated freight conditions.
- The program supports maintenance (`MNT`) and inquiry modes, with global input protection in inquiry mode.
- Function keys (F03, F04, F05, F08, F10, PAGEDN, ENTER) provide user interaction for navigation, prompting, refreshing, filtering, and repositioning.
- The subfile supports pagination, filtering (e.g., include/exclude deleted records), and color coding for deleted entries.

This program is a typical AS/400 RPG application, leveraging subfiles for interactive data display and processing, with robust error handling and database access.