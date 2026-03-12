The **BB801** RPGLE program is a Customer Order Inquiry application within the Customer Orders and Billing system, designed to display detailed information about customer orders, including open and canceled orders. It is called from the main program **BB802** and operates interactively, using subfiles to present order details. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### **Process Steps of the BB801 RPG Program**

The program follows a structured flow to retrieve and display order details based on input parameters, handle user interactions via function keys, and manage subfile displays. The main steps are executed in the `srfmt` subroutine, with supporting subroutines handling specific tasks. Here’s a breakdown of the key process steps:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Sets up the program environment and initializes variables.
   - **Actions**:
     - Receives input parameters: `p$co` (company), `p$ord#` (order number), `p$mode` (mode: 'OPN' for open orders, 'CNL' for canceled orders), `p$fgrp` (file group: 'G' or 'Z'), and `p$flag` (return flag).
     - Moves input parameters to format fields (`f$co`, `f$ord#`) and sets `w$mode = 'INQ'` for inquiry mode.
     - Initializes subfile control fields (`rrn1`, `rrn2`, `rrnsv1`, `rrnsv2`) to zero and sets page sizes (`pagsz1 = 12`, `pagsz2 = 10`).
     - Sets up date fields using the program status data structure (`psds##`) to derive current date (`ps#dat`) in century-year-month-day format.
     - Configures message handling fields (`dspmsg`, `m@pgmq`, `m@key`) for error messaging.
     - Defines key lists (`klist`) for database access (e.g., `klordh`, `klordd`, `klcust`, `klcstship`).
     - Sets the header (`c$hdr1`) based on `p$mode` ('Customer Order Inquiry' for 'OPN', 'Customer Canceled Order Inquiry' for 'CNL').

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Purpose**: Opens required database files with appropriate overrides based on the file group (`p$fgrp`) and mode (`p$mode`).
   - **Actions**:
     - Applies file overrides using arrays `ovg`, `ovz` (general files), `ovgopn`, `ovzopn` (open orders), or `ovgcnl`, `ovzcnl` (canceled orders).
     - Executes `QCMDEXC` to apply overrides dynamically (e.g., `arcust` to `garcust` or `zarcust`).
     - Opens files: `arcust`, `gsctum`, `gstabl`, `shipto`, `inloc`, `bbordb`, `bbordd`, `bbordh`, `bbordi`, `bbordm`, `bbordo`, `bbords1`, `bborcl`, `bbcnb`, `bbcnd`, `bbcnh`, `bbcni`, `bbcnm`, `bbcno`, `bbcnor`, `bborhs1`.

3. **Retrieve Data (`rtvdta` Subroutine)**:
   - **Purpose**: Fetches order header and customer data for the specified order.
   - **Actions**:
     - Chains to `bbordh` (open orders) or `bbcnh` (canceled orders) using `klordh` (`f$co`, `f$ord#`) to retrieve order header details.
     - Chains to `arcust` using `klcust` to retrieve customer details (`bocust`).
     - Chains to `bborcl` using `klorcl` to retrieve order control data.
     - Chains to `gstabl` for terms (`klarterm`), salesman (`klslsman`), order status (`klbborst`), and cancel reason (`klbborcn`, for canceled orders).
     - Calls `rtvshipto` to retrieve ship-to address details.
     - Calls `clcordtot` to calculate order totals (product, miscellaneous, freight).
     - Sets `w$fmt = 'SFL1'` to display the order detail subfile.

4. **Process Formats (`srfmt` Subroutine)**:
   - **Purpose**: Manages the main loop for displaying and processing panel formats and subfiles.
   - **Actions**:
     - Clears the screen (`clrscr`).
     - Initializes format fields (`f01mov`) and sets `w$fmt = 'SFL1'` for subfile display.
     - Enters a loop (`fmtagn`) to:
       - Display the message subfile if needed (`wrtmsg`).
       - Select and display the appropriate format based on `w$fmt`:
         - `FMT01`: Main order header format (`exfmt fmt01`).
         - `FMT02`: Secondary format (`exfmt fmt02`).
         - `SFL1`: Order detail subfile (`srsfl1`).
         - `SFL2`: Miscellaneous charges subfile (`srsfl2`).
       - Clears error indicators (`*in50`–`*in69`) and cursor position (`row`, `col`).
       - Clears the message subfile (`clrmsg`) if displayed.
       - Processes user input based on the current format (`f01sr` for `FMT01`, `f02sr` for `FMT02`).

5. **Process Format FMT01 (`f01sr` Subroutine)**:
   - **Purpose**: Handles user input on the main order header format.
   - **Actions**:
     - Processes function keys:
       - **F03**: Exits the program by setting `fmtagn = *off`.
       - **F10**: Resets cursor position to home (`row`, `col` cleared).
       - **ENTER**: Validates input (`f01edt`), and if no errors (`*in50 = *off`), proceeds to the next format (`f01nxt`).
     - Calls `f01pro` to set display attributes (e.g., highlight order process status 'F' in red).

6. **Process Format FMT02 (`f02sr` Subroutine)**:
   - **Purpose**: Handles user input on the secondary format.
   - **Actions**:
     - Processes function keys:
       - **F03**: Exits the program (`fmtagn = *off`).
       - **F10**: Resets cursor position to home.
       - **F12**: Returns to `SFL2` (`w$fmt = 'SFL2'`).
       - **ENTER**: Validates input (`f02edt`), and if no errors, proceeds to the next format (`f02nxt`).

7. **Determine Next Format for FMT01 (`f01nxt` Subroutine)**:
   - **Purpose**: Sets the next format to display.
   - **Actions**:
     - Sets `w$fmt = 'SFL1'` to transition to the order detail subfile.

8. **Determine Next Format for FMT02 (`f02nxt` Subroutine)**:
   - **Purpose**: Handles transition from `FMT02`.
   - **Actions**:
     - Calls `BB800E` for order access/marks inquiry with parameters `a$co`, `a$ord#`, `a$ship = '000'`, and `p$fgrp`.
     - Returns to `SFL1` (`w$fmt = 'SFL1'`) and reinitializes `FMT01` fields (`f01mov`).

9. **Process Subfile SFL1 (`srsfl1` Subroutine)**:
   - **Purpose**: Manages the order detail subfile display and user interactions.
   - **Actions**:
     - Initializes subfile mode to folded (`sfmod1 = '1'`, `*in45 = *on`).
     - Clears and writes the message subfile (`clrmsg`, `wrtmsg`).
     - Positions the subfile (`sf1rep`) and enters a loop (`sf1agn`):
       - Handles repositioning if `repsfl = *on` by updating `c1seq` and calling `sf1rep`.
       - Displays the command line (`sflcmd1`) and message subfile if needed.
       - Sets display indicator (`*in41`) based on subfile records (`rrn1 > 0`).
       - Toggles folded/unfolded mode (`*in45`) based on `sfmod1`.
       - Displays subfile control (`exfmt sflctl1`).
       - Processes function keys:
         - **F03**: Exits subfile and program (`sf1agn`, `fmtagn = *off`).
         - **F06**: Calls `historddInq` for history inquiry if a subfile record is selected.
         - **F12**: Returns to `FMT01` (`w$fmt = 'FMT01'`).
         - **F15**: Clears subfile and reloads (`sf1clr`, `sf1lod`).
         - **F16**: Displays order instructions (`sflcmd2`, chains to `bbordi` or `bbcni`).
         - **F20**: Displays miscellaneous charges subfile (`w$fmt = 'SFL2'`).
         - **ENTER**: Processes subfile selections (`sf1prc`).
         - **PAGEDN**: Loads additional subfile records (`sf1lod`).
       - Updates cursor position and subfile record number (`rcdnb1`).

10. **Process Subfile SFL2 (`srsfl2` Subroutine)**:
    - **Purpose**: Manages the miscellaneous charges subfile.
    - **Actions**:
      - Similar to `srsfl1`, initializes subfile mode, clears/writes message subfile, and positions records (`sf2rep`).
      - Processes function keys:
        - **F03**: Exits subfile and program.
        - **F12**: Returns to `SFL1` (`w$fmt = 'SFL1'`).
        - **F15**: Clears and reloads subfile.
        - **ENTER**: Processes selections (`sf2prc`).
        - **PAGEDN**: Loads additional records (`sf2lod`).

11. **Retrieve Ship-to Address (`rtvshipto` Subroutine)**:
    - **Purpose**: Populates ship-to address fields based on order data.
    - **Actions**:
      - If `boship = 0`, uses customer master (`arcust`) address (`arname`, `aradr1`, etc.).
      - If `boship` is 001–899 or 900–998, chains to `shipto` using `klcstship` or `klord999` to retrieve address (`csname`, `csadr1`, etc.).
      - Formats zip code (`f$cszp`) with hyphen if `arzip9` is non-zero.
      - Calls `movshipto` to move ship-to fields to format fields.

12. **Move Ship-to Values (`movshipto` Subroutine)**:
    - **Purpose**: Transfers ship-to data to display fields.
    - **Actions**:
      - Moves `csname`, `csadr1`, `csadr2`, `csadr3`, `csadr4`, `csctst`, `csstat`, `cscszp`, `cscty` to corresponding `f$` fields.

13. **Calculate Order Totals (`clcordtot` Subroutine)**:
    - **Purpose**: Computes product, miscellaneous, and freight totals for the order.
    - **Actions**:
      - Clears totals (`f$ptot`, `f$mtot`, `f$ftot`).
      - For order details (`bbordd` or `bbcnd` based on `p$mode`):
        - Reads records using `klordh`.
        - Converts container quantity (`bdqty`) to actual quantity using `gsctum` (`cuoper`, `cucvfa`).
        - Calculates product total (`f$ptot`) as `w$qty * bdprce`.
        - Adds freight (`bdofrr` or `bdfrrt * w$qty`) to `f$ftot`.
      - For miscellaneous charges (`bbordm` or `bbcnm`):
        - Reads records and adds `bmamt` to `f$mtot` (non-freight) or `f$ftot` (freight, `bmmsty = 'F'`).
      - Skips deleted records (`bddel`, `bmdel <> 'D'`).

14. **History Inquiry (`historddInq` Subroutine)**:
    - **Purpose**: Calls the order history inquiry program for a selected subfile record.
    - **Actions**:
      - Sets `o$file = 'BBORDH'`, `o$fgrp`, `f$co`, `bocust`, `f$ord#`, `o$seq`.
      - Calls `GB730P` with `x$ordrhist` data structure.

15. **Message Handling (`addmsg`, `wrtmsg`, `clrmsg` Subroutines)**:
    - **addmsg**: Sends error messages to the program message queue using `QMHSNDPM`.
    - **wrtmsg**: Displays the message subfile (`msgctl`).
    - **clrmsg**: Clears the message subfile using `QMHRMVPM`.

16. **Program Termination**:
    - Closes all open files (`close *all`).
    - Sets `*inlr = *on` and returns.

---

### **Business Rules**

1. **Order Mode Handling**:
   - Supports two modes: 'OPN' (open orders) and 'CNL' (canceled orders), determined by `p$mode`.
   - Different files are accessed based on mode (`bbord*` for open, `bbcn*` for canceled).
   - The header (`c$hdr1`) reflects the mode ('Customer Order Inquiry' or 'Customer Canceled Order Inquiry').

2. **File Overrides**:
   - Files are overridden based on `p$fgrp` ('G' or 'Z') to access different libraries (e.g., `garcust` vs. `zarcust`).
   - Additional overrides for open (`ovgopn`, `ovzopn`) or canceled orders (`ovgcnl`, `ovzcnl`) ensure correct file access.

3. **Ship-to Address Logic**:
   - If `boship = 0`, uses customer master address.
   - For `boship` 001–899 or 900–998, retrieves from `shipto` master; for 900+, uses one-time ship-to address.
   - Skips deleted ship-to records (`csdel <> 'D'`).

4. **Order Totals Calculation**:
   - Product total (`f$ptot`): Sum of `bdqty * bdprce` after converting container quantity.
   - Freight total (`f$ftot`): Sum of `bdofrr` or `bdfrrt * qty` from order details, plus freight-type miscellaneous charges (`bmmsty = 'F'`).
   - Miscellaneous total (`f$mtot`): Sum of non-freight miscellaneous charges (`bmamt`).
   - Excludes deleted records and handles special case for sequence 950 (automatic fuel surcharge, per jb01).

5. **Display and Navigation**:
   - Highlights orders with process status 'F' in red (`*in76`).
   - Supports folded/unfolded subfile views (`*in45`).
   - Allows navigation between formats (`FMT01`, `FMT02`, `SFL1`, `SFL2`) and inquiries (e.g., order access/marks via `BB800E`).
   - Ensures proper subfile pagination and cursor positioning.

6. **Error Handling**:
   - Validates input in `f01edt` and `f02edt` (though currently empty, likely to be implemented).
   - Displays error messages for invalid inputs or credit hold status (`com` array).
   - Clears errors after processing to maintain a clean user interface.

7. **Credit Hold Check**:
   - Displays a message if the order is on credit hold (`com(01)`).

8. **Revision-Specific Rules**:
   - **jb01**: Includes sequence 950 records (automatic fuel surcharge) in totals.
   - **jk02**: Adds order process status to `FMT01` display.
   - **jk03**: Adds F06 for history inquiry via `GB730P`.
   - **jk04**: Expands screen to 132 columns and adds `inloc` file support.
   - **jk05**: Ensures `bborhs1` and `bbords1` are opened for canceled orders to avoid I/O errors.

---

### **Tables Used**

The program accesses the following database files, all defined with `usropn` for manual opening and overrides:

1. **arcust**: Customer master file (customer details).
2. **gsctum**: Container unit of measure file (quantity conversions).
3. **gstabl**: General table file (terms, salesman, order status, cancel reason).
4. **shipto**: Ship-to master file (ship-to addresses).
5. **inloc**: Location master file (added by jk04).
6. **bbordb**: Order billing file (open orders).
7. **bbordd**: Order detail file (open orders).
8. **bbordh**: Order header file (open orders).
9. **bbordi**: Order instructions file (open orders).
10. **bbordm**: Order miscellaneous charges file (open orders).
11. **bbordo**: Order file (open orders).
12. **bbords1**: Order detail file (open orders, additional).
13. **bborcl**: Order control file (open orders).
14. **bbcnb**: Canceled order billing file.
15. **bbcnd**: Canceled order detail file.
16. **bbcnh**: Canceled order header file.
17. **bbcni**: Canceled order instructions file.
18. **bbcnm**: Canceled order miscellaneous charges file.
19. **bbcno**: Canceled order file.
20. **bbcnor**: Canceled order reason file.
21. **bborhs1**: Order history file (open/canceled orders, added by jk04).

---

### **External Programs Called**

The program interacts with the following external programs:

1. **BB800E**: Order access/marks inquiry program, called from `f02nxt`.
   - Parameters: `a$co`, `a$ord#`, `a$ship`, `o$frgp`.
2. **GB730P**: Order history inquiry program, called via F06 from `historddInq`.
   - Parameters: `x$ordrhist` (data structure with `o$file`, `o$fgrp`, `f$co`, `bocust`, `f$ord#`, `o$seq`).
3. **QMHSNDPM**: Sends error messages to the program message queue.
   - Parameters: `m@id`, `m@msgf`, `m@data`, `m@l`, `m@type`, `m@pgmq`, `m@scnt`, `m@key`, `m@errc`.
4. **QMHRMVPM**: Removes messages from the program message queue.
   - Parameters: `m@pgmq`, `m@scnt`, `m@rmvk`, `m@rmv`, `m@errc`.
5. **QCMDEXC**: Executes override commands for file access.
   - Parameters: (implied, not explicitly defined in code snippet).

---

### **Additional Notes**

- **Indicator Usage**: Uses indicators (19, 21–39, 40–43, 49, 50–69, 70–79, 80, 88, 90–99) for screen control, subfile operations, and error handling.
- **Field Prefixes**: Organizes fields with prefixes (e.g., `f$` for display, `c$` for subfile control, `s1`/`s2` for subfile fields).
- **Revisions**: Includes updates for fuel surcharge (jb01), process status display (jk02), history inquiry (jk03), screen expansion (jk04), and file I/O fix (jk05).
- **Subfile Management**: Supports two subfiles (`SFL1` for order details, `SFL2` for miscellaneous charges) with pagination and folded/unfolded views.
- **Error Messaging**: Uses `com` array for messages like credit hold and handles message subfile display/clearing.

This program provides a comprehensive interface for querying customer order details, with robust navigation, data retrieval, and validation features, integrated with the broader system via calls from **BB802**.