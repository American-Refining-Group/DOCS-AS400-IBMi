The **SA801** RPGLE program is a Customer Invoice Inquiry application within the Customer Invoicing and Sales system, designed to display detailed information about customer invoices, including header and detail data, using subfiles. It is called from the main program **BB802** and supports inquiries for invoices tied to specific customers, orders, and sales/accounts file groups. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### **Process Steps of the SA801 RPG Program**

The program follows a structured flow to retrieve and display invoice details based on input parameters, handle user interactions via function keys, and manage subfile displays. The main steps are executed in the `srfmt` subroutine, with supporting subroutines handling specific tasks. Here’s a breakdown of the key process steps:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Sets up the program environment and initializes variables.
   - **Actions**:
     - Receives input parameters: `p$co` (company), `p$cust` (customer), `p$inv` (invoice number), `p$ord` (order number), `p$sagr` (sales/accounts file group: e.g., 'BCM', 'COP', 'DBB', or blank), `p$pmov` (product move: 'Y' or 'N'), `p$fgrp` (file group: 'G' or 'Z'), and `p$flag` (return flag).
     - Moves input parameters to format fields (`c$co`, `c$cust`, `c$inv`, `c$ord`) and sets `w$mode = 'INQ'` for inquiry mode.
     - Initializes subfile control fields (`rrn1`, `rrn2`, `rrnsv1`, `rrnsv2`) to zero and sets page sizes (`pagsz1 = 12`, `pagsz2 = 12`).
     - Sets up date fields using the program status data structure (`psds##`) to derive current date (`ps#dat`) in century-year-month-day format.
     - Configures message handling fields (`dspmsg`, `m@pgmq`, `m@key`) for error messaging.
     - Sets subfile modes to folded (`sfmod1 = '1'`, `sfmod2 = '1'`, `*in45 = *on`) for initial display.
     - Defines key lists (`klist`) for database access (e.g., `kl5shz`, `klcust`, `klslsman`, `klarterm`, `klincotm`, `klc1s1`, `klc2s1`).
     - Sets the header (`c$hdr1`) to 'Customer Invoice Inquiry'.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Purpose**: Opens required database files with appropriate overrides based on `p$fgrp` and `p$sagr`.
   - **Actions**:
     - Applies file overrides using arrays `ovg1`, `ovz1` (general files: `arcust`, `gstabl`, `sa5shz`) and `ovg2`, `ovz2` (invoice-specific files: `sa5find`, `sa5fixm`, `sa5movdl`, `sa5movml`).
     - Overrides `sa5find` and `sa5fixm` based on `p$sagr` (e.g., `gsa5bcxm` for 'BCM', `gsa5coxm` for 'COP', `gsa5dbxm` for 'DBB', or `gsa5find`/`gsa5fixm` for blank).
     - Executes `QCMDEXC` to apply overrides dynamically.
     - Opens files: `arcust`, `gstabl`, `sa5shz`, `sa5find`, `sa5fixm`, `sa5movdl`, `sa5movml`.

3. **Retrieve Data (`rtvdta` Subroutine)**:
   - **Purpose**: Fetches invoice header and customer data for the specified invoice.
   - **Actions**:
     - Chains to `sa5shz` using `kl5shz` (`p$co`, `p$cust`, `p$inv`, `p$ord`) to retrieve invoice header details.
     - Chains to `arcust` using `klcust` to retrieve customer details (`arcust`).
     - Chains to `gstabl` for salesman (`klslsman`) and incoterms (`klincotm`) descriptions.
     - If `p$pmov = 'Y'`, reads `sa5movml` to retrieve salesman (`saslmn`); otherwise, uses `sa5fixm`.
     - Formats data for display (e.g., `f$smnm`, `f$inct`).

4. **Process Formats (`srfmt` Subroutine)**:
   - **Purpose**: Manages the main loop for displaying and processing panel formats and subfiles.
   - **Actions**:
     - Clears the screen (`clrscr`).
     - Initializes format fields (`f01mov`) and sets `w$fmt = 'SFL1'` for subfile display (changed from `FMT01` due to jk03).
     - Enters a loop (`fmtagn`) to:
       - Display the message subfile if needed (`wrtmsg`).
       - Select and display the appropriate format based on `w$fmt`:
         - `FMT01`: Invoice header format (`exfmt fmt01`, added by jk03).
         - `SFL1`: Invoice detail subfile (`srsfl1`).
         - `SFL2`: Miscellaneous charges subfile (`srsfl2`).
         - Default: Displays `FMT01`.
       - Clears error indicators (`*in50`–`*in69`).
       - Clears the message subfile (`clrmsg`) if displayed.
       - Processes user input for `FMT01` (`f01sr`).

5. **Process Format FMT01 (`f01sr` Subroutine)**:
   - **Purpose**: Handles user input on the invoice header format.
   - **Actions**:
     - Processes function keys:
       - **F03**: Exits the program (`fmtagn = *off`).
       - **F10**: Resets cursor position to home (`row`, `col` cleared).
       - **ENTER**: Validates input (`f01edt`), and if no errors (`*in50 = *off`), proceeds to the next format (`f01nxt`).
     - Calls `f01pro` to set display attributes (e.g., highlight order process status 'F' in red).

6. **Determine Next Format for FMT01 (`f01nxt` Subroutine)**:
   - **Purpose**: Sets the next format to display.
   - **Actions**:
     - Sets `w$fmt = 'SFL1'` to transition to the invoice detail subfile.

7. **Process Subfile SFL1 (`srsfl1` Subroutine)**:
   - **Purpose**: Manages the invoice detail subfile display and user interactions.
   - **Actions**:
     - Initializes subfile mode to folded (`sfmod1 = '1'`, `*in45 = *on`).
     - Clears and writes the message subfile (`clrmsg`, `wrtmsg`).
     - Positions the subfile (`sf1rep`) and enters a loop (`sf1agn`):
       - Handles repositioning if `repsfl = *on` by updating `c1seq` and calling `sf1rep`.
       - Displays the command line and message subfile if needed.
       - Sets display indicator (`*in41`) based on subfile records (`rrn1 > 0`).
       - Toggles folded/unfolded mode (`*in45`) based on `sfmod1`.
       - Displays subfile control (`exfmt sflctl1`).
       - Processes function keys:
         - **F03**: Exits subfile and program (`sf1agn`, `fmtagn = *off`).
         - **F06**: Calls `historddInq` for history inquiry (not shown in truncated code but implied from similar programs).
         - **F12**: Returns to `FMT01` (`w$fmt = 'FMT01'`).
         - **F15**: Clears subfile and reloads (`sf1clr`, `sf1lod`).
         - **F20**: Displays miscellaneous charges subfile (`w$fmt = 'SFL2'`).
         - **ENTER**: Processes subfile selections (`sf1prc`, not shown in truncated code).
         - **PAGEDN**: Loads additional subfile records (`sf1lod`, not shown in truncated code).
       - Updates cursor position and subfile record number.

8. **Process Subfile SFL2 (`srsfl2` Subroutine)**:
   - **Purpose**: Manages the miscellaneous charges subfile.
   - **Actions**:
     - Similar to `srsfl1`, initializes subfile mode, clears/writes message subfile, and positions records (`sf2rep`).
     - Processes function keys:
       - **F03**: Exits subfile and program.
       - **F12**: Returns to `SFL1` (`w$fmt = 'SFL1'`).
       - **F15**: Clears and reloads subfile.
       - **ENTER**: Processes selections (`sf2prc`, not shown in truncated code).
       - **PAGEDN**: Loads additional records (`sf2lod`, not shown in truncated code).

9. **Message Handling (`addmsg`, `wrtmsg`, `clrmsg` Subroutines)**:
   - **addmsg**: Sends error messages to the program message queue using `QMHSNDPM`.
   - **wrtmsg**: Displays the message subfile (`msgctl`).
   - **clrmsg**: Clears the message subfile using `QMHRMVPM`.

10. **Program Termination**:
    - Closes all open files (`close *all`).
    - Sets `*inlr = *on` and returns.

---

### **Business Rules**

1. **Invoice Inquiry Scope**:
   - Displays invoice details for a specific company (`p$co`), customer (`p$cust`), invoice number (`p$inv`), and order number (`p$ord`).
   - Supports different sales/accounts file groups (`p$sagr`: 'BCM', 'COP', 'DBB', or blank) for file overrides.

2. **File Overrides**:
   - Files are overridden based on `p$fgrp` ('G' or 'Z') to access different libraries (e.g., `garcust` vs. `zarcust`).
   - Additional overrides for `sa5find` and `sa5fixm` based on `p$sagr` to access group-specific files (e.g., `gsa5bcxm` for 'BCM').
   - Overrides for `sa5movdl` and `sa5movml` are applied when product moves are involved (`p$pmov = 'Y'`).

3. **Product Move Handling (jk02)**:
   - If `p$pmov = 'Y'`, uses `sa5movml` (miscellaneous move lines) and `sa5movdl` (detail move lines) to retrieve data; otherwise, uses `sa5fixm`.
   - Ensures salesman data is retrieved from the appropriate file based on product move status.

4. **FOB Code Handling (jb02)**:
   - Supports FOB codes in `sa5shz` (e.g., 'T' for FOB Shipping Point/Tax Destination).
   - Displays FOB descriptions (e.g., 'F.O.B. DESTINATION', 'F.O.B. SHIPPING POINT', 'FOB SHP PNT/TAX DEST') from the `com` array.

5. **Display and Navigation**:
   - Highlights invoices with order process status 'F' in red (`*in76`).
   - Supports folded/unfolded subfile views (`*in45`) for `SFL1` (invoice details) and `SFL2` (miscellaneous charges).
   - Allows navigation between formats (`FMT01`, `SFL1`, `SFL2`) and supports inquiries via function keys.
   - Ensures proper subfile pagination and cursor positioning.

6. **Error Handling**:
   - Validates input in `f01edt` (currently empty, likely to be implemented).
   - Displays error or status messages (e.g., 'DELIVERY YES', 'DELIVERY NO') using the `com` array.
   - Clears errors after processing to maintain a clean user interface.

7. **Revision-Specific Rules**:
   - **jk01**: Renames field `saf005` to `s@f005` to resolve duplicate field issues.
   - **jb02**: Adds FOB code 'T' (FOB Shipping Point/Tax Destination) and includes it in `com` array.
   - **jk02**: Adds support for product moves with `sa5movdl` and `sa5movml` files.
   - **jk03**: Expands display to include `FMT01` for header information, shifting initial display to `SFL1`.

---

### **Tables Used**

The program accesses the following database files, all defined with `usropn` for manual opening and overrides:

1. **arcust**: Customer master file (customer details).
2. **gstabl**: General table file (salesman, terms, incoterms).
3. **sa5shz**: Sales history header file (invoice header details).
4. **sa5find**: Sales history detail file (invoice line items).
5. **sa5fixm**: Sales history miscellaneous charges file.
6. **sa5movdl**: Sales history move detail file (product moves, added by jk02).
7. **sa5movml**: Sales history move miscellaneous file (product moves, added by jk02).

---

### **External Programs Called**

The program interacts with the following external programs:

1. **QMHSNDPM**: Sends error messages to the program message queue.
   - Parameters: `m@id`, `m@msgf`, `m@data`, `m@l`, `m@type`, `m@pgmq`, `m@scnt`, `m@key`, `m@errc`.
2. **QMHRMVPM**: Removes messages from the program message queue.
   - Parameters: `m@pgmq`, `m@scnt`, `m@rmvk`, `m@rmv`, `m@errc`.
3. **QCMDEXC**: Executes override commands for file access.
   - Parameters: (implied, not explicitly defined in code snippet).
4. **GB730P** (implied, based on similar programs like BB801): Likely called for history inquiry via F06, though not shown in truncated code.
   - Parameters: Likely uses a data structure similar to `x$ordrhist` (e.g., `o$file`, `o$fgrp`, `f$co`, `p$cust`, `p$ord`, `o$seq`).

---

### **Additional Notes**

- **Indicator Usage**: Uses indicators (19, 21–39, 40–43, 49, 50–69, 70–79, 80, 88, 90–99) for screen control, subfile operations, and error handling.
- **Field Prefixes**: Organizes fields with prefixes (e.g., `f$` for display, `c$` for subfile control, `s1`/`s2` for subfile fields).
- **Revisions**: Includes updates for field renaming (jk01), FOB code support (jb02), product moves (jk02), and display expansion (jk03).
- **Subfile Management**: Supports two subfiles (`SFL1` for invoice details, `SFL2` for miscellaneous charges) with pagination and folded/unfolded views.
- **Error Messaging**: Uses `com` array for messages like delivery status and FOB descriptions, with dynamic message handling.

This program provides a comprehensive interface for querying customer invoice details, with robust navigation, data retrieval, and validation features, integrated with the broader system via calls from **BB802**.