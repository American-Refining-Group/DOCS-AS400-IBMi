The **BB800E** RPGLE program is a Customer Order Instructions Inquiry application within the Customer Order Entry system, designed to display accessorials and marks information for either customer orders or ship-to records, using a subfile. It is called from the main program **BB802** (and referenced in **BB801** for order access/marks inquiry). Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### **Process Steps of the BB800E RPG Program**

The program retrieves and displays accessorials/marks data based on input parameters, handles user interactions via function keys, and manages a subfile for display. The main steps are executed in the `srsfl1` subroutine, with supporting subroutines handling specific tasks. Here’s a breakdown of the key process steps:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Sets up the program environment and initializes variables.
   - **Actions**:
     - Receives input parameters: `p$co` (company, 2 digits), `p$csor` (customer or order number, 6 digits), `p$ship` (ship-to code, 3 digits), and `p$fgrp` (file group: 'G' or 'Z').
     - Initializes output parameters: `o$mode = 'INQ'` (inquiry mode), `o$flag = '0'` (return flag).
     - Initializes subfile control fields: `rrn1 = 0`, `rrnsv1 = 0`, `pagsz1 = 32` (subfile page size).
     - Sets up message handling fields: `dspmsg = *blank`, `m@pgmq = '*'`, `m@key = *blanks`.
     - Defines key lists (`klist`) for database access: `klc1s1`, `klc1s2`, `klc1r1`, `klc1r2`, `klordh`, `klcus1`, `klcus2`, `klship`.
     - Sets subfile mode to folded (`sfmod1 = '1'`, `*in45 = *on`) for initial display.
     - Initializes header (`c$hdr`) based on mode: 'Open Order Accessorials/Marks Inquiry' (if `p$ship = '000'`) or 'ShipTo Master Accessorials/Marks Inquiry' (if `p$ship ≠ '000'`).

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Purpose**: Opens required database files with appropriate overrides based on `p$fgrp`.
   - **Actions**:
     - Applies file overrides using arrays `ovg` or `ovz` (for 'G' or 'Z' file groups) for files `arcust`, `bbora1`, `bbordh`, `bbshsa1`, and `shipto`.
     - Executes `QCMDEXC` to apply overrides dynamically (e.g., `arcust` to `garcust` or `zarcust`).
     - Opens files: `arcust`, `bbora1`, `bbordh`, `bbshsa1`, `shipto`.

3. **Process Parameters (`@parms` Subroutine)**:
   - **Purpose**: Validates and processes input parameters.
   - **Actions**:
     - Checks the number of parameters passed using `ps#prm` (from program status data structure).
     - Moves parameters to format fields: `p$co` to `c$co`, `p$csor` to `c$csor`, `p$ship` to `c$ship`.
     - If fewer than expected parameters are passed (`ps#err = 221`), sets return code to `*detc` and exits.
     - Includes commented-out test values for `c$co`, `c$csor`, and `c$ship` for debugging.

4. **Retrieve Data (`rtvdta` Subroutine)**:
   - **Purpose**: Fetches data for the subfile based on whether the inquiry is for a customer order or ship-to.
   - **Actions** (not shown in truncated code, but inferred from context and similar programs):
     - If `p$ship = '000'`, chains to `bbordh` using `klordh` (`c$co`, `c$csor`) to verify the order and reads `bbora1` using `klc1r1` to retrieve order accessorials/marks.
     - If `p$ship ≠ '000'`, chains to `arcust` using `klcus2` and `shipto` using `klship` to verify customer and ship-to, then reads `bbshsa1` using `klc1r2` to retrieve ship-to accessorials/marks.
     - Populates subfile fields (`s1*`) with retrieved data.

5. **Process Subfile (`srsfl1` Subroutine)**:
   - **Purpose**: Manages the subfile display and user interactions.
   - **Actions**:
     - Executes parameter processing (`@parms`) and error handling (`*pssr`).
     - Retrieves data (`rtvdta`).
     - Sets subfile mode to folded (`sfmod1 = '1'`, `*in45 = *on`).
     - Clears and writes the message subfile (`clrmsg`, `wrtmsg`).
     - Positions the subfile (`sf1rep`) to the first record.
     - Writes the assume-overlay record (`wdwovr`).
     - Enters a main loop (`sf1agn`) to:
       - Handle repositioning if `repsfl = *on` by updating `c$rcid` and calling `sf1rep`.
       - Display the message subfile if needed (`wrtmsg`).
       - Set subfile display indicator (`*in41`) based on records (`rrn1 > 0`).
       - Toggle folded/unfolded mode (`*in45`) based on `sfmod1`.
       - Display subfile control (`write sflcmd1`, `exfmt sflctl1`).
       - Clear message subfile if displayed (`clrmsg`, `write msgclr`).
       - Clear error indicators (`*in21`–`*in39`, `*in50`–`*in69`).
       - Update subfile record number (`rcdnb1 = pagrrn`) for redisplay.
       - Process function keys:
         - **F03**: Exits the subfile and program (`sf1agn = *off`).
         - **F12**: Returns to the calling program (`sf1agn = *off`).
         - **ENTER**: Processes subfile selections (not shown in truncated code, likely in `sf1prc`).
         - **PAGEDN**: Loads additional subfile records (commented out, suggesting a load-all technique).
     - Iterates until `sf1agn = *off`.

6. **Message Handling (`wrtmsg`, `clrmsg` Subroutines)**:
   - **wrtmsg**: Displays the message subfile (`msgctl`) with `*in49 = *on`.
   - **clrmsg**: Clears the message subfile using `QMHRMVPM`, preserving the current record format (`rcdsav`) and page number (`pagsav`).

7. **Error Handling (`*pssr` Subroutine)**:
   - **Purpose**: Handles parameter-related errors.
   - **Actions**:
     - Checks for parameter errors (`ps#err = 221`) and sets return code to `*detc` if insufficient parameters are passed.
     - Returns to the caller.

8. **Program Termination**:
   - Closes all open files (`close *all`).
   - Sets `*inlr = *on` and returns.

---

### **Business Rules**

1. **Inquiry Mode**:
   - The program operates in inquiry mode (`o$mode = 'INQ'`), with fields protected (`*in70 = *on`) to prevent modifications.
   - Supports two inquiry types based on `p$ship`:
     - If `p$ship = '000'`, queries accessorials/marks for a customer order (`bbora1`, `bbordh`).
     - If `p$ship ≠ '000'`, queries accessorials/marks for a customer ship-to (`bbshsa1`, `shipto`).

2. **File Overrides**:
   - Files are overridden based on `p$fgrp` ('G' or 'Z') to access different libraries (e.g., `garcust` vs. `zarcust`, `gbbora1` vs. `zbbora1`).
   - Overrides are applied dynamically using `QCMDEXC` for files `arcust`, `bbora1`, `bbordh`, `bbshsa1`, and `shipto`.

3. **Data Retrieval**:
   - For customer orders (`p$ship = '000'`), retrieves data from `bbora1` (order accessorials) and validates the order in `bbordh`.
   - For ship-to inquiries (`p$ship ≠ '000'`), retrieves data from `bbshsa1` (ship-to accessorials) and validates customer and ship-to in `arcust` and `shipto`.
   - Uses `arcust` to retrieve customer details for display.

4. **Subfile Display**:
   - Displays accessorials/marks in a subfile (`SFL1`) with a page size of 32 records (`pagsz1`).
   - Supports folded/unfolded views (`*in45`) controlled by `sfmod1`.
   - Ensures proper pagination and cursor positioning using `pagrrn` and `rcdnb1`.

5. **User Interaction**:
   - Supports limited function keys: F03 (exit), F12 (return), and ENTER (process subfile selections).
   - PAGEDN is commented out, suggesting a load-all subfile approach where all records are loaded initially.
   - Displays different headers based on mode: 'Open Order Accessorials/Marks Inquiry' or 'ShipTo Master Accessorials/Marks Inquiry'.

6. **Error Handling**:
   - Validates input parameters, exiting with `*detc` if insufficient parameters are passed.
   - Uses message subfile for displaying errors or status messages.
   - Clears errors after processing to maintain a clean interface.

7. **Revisions**:
   - No specific revisions are noted, indicating the program is stable as of the last update.

---

### **Tables Used**

The program accesses the following database files, all defined with `usropn` for manual opening and overrides:

1. **arcust**: Customer master file (customer details).
2. **bbora1**: Order accessorials/marks file (used when `p$ship = '000'`).
3. **bbordh**: Order header file (validates order for customer order inquiries).
4. **bbshsa1**: Ship-to accessorials/marks file (used when `p$ship ≠ '000'`).
5. **shipto**: Ship-to master file (validates ship-to for ship-to inquiries).

---

### **External Programs Called**

The program interacts with the following external programs:

1. **QCMDEXC**: Executes override commands for file access.
   - Parameters: `dbov##` (override command, 80 characters), `dbol##` (length, 15.5).
2. **QMHRMVPM**: Removes messages from the program message queue.
   - Parameters: `m@pgmq`, `m@scnt`, `m@rmvk`, `m@rmv`, `m@errc`.

---

### **Additional Notes**

- **Indicator Usage**: Uses indicators (19, 21–39, 40–43, 49, 50–69, 70, 88, 90–99) for screen control, subfile operations, and error handling. Notably, `*in70` globally protects fields in inquiry mode.
- **Field Prefixes**: Organizes fields with prefixes (e.g., `f$` for display, `c$` for subfile control, `s1` for subfile fields).
- **Subfile Management**: Uses a single subfile (`SFL1`) for accessorials/marks data, with a load-all approach (PAGEDN commented out).
- **Error Messaging**: Uses a message subfile for dynamic error or status display, cleared via `QMHRMVPM`.
- **Testing Support**: Includes commented-out test values for `p$co`, `p$csor`, and `p$ship` for debugging.
- **Context with BB801**: As referenced in **BB801**, this program is called from `f02nxt` for order access/marks inquiry, passing `a$co`, `a$ord#`, `a$ship`, and `o$frgp`.

This program provides a focused interface for querying accessorials and marks data, integrated with the broader Customer Order Entry system via **BB802** and supporting **BB801** for detailed order inquiries.