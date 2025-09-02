The RPG program `BI890P.rpgle.txt` is a Ship-to Master Inquiry Prompt program, part of the Customer Orders and Invoicing system, designed to interact with a display file for user inquiries and maintenance of ship-to records. Below, I’ll explain the process steps, business rules, tables used, and external programs called, based on the provided RPGLE code and its interaction with the OCL program `BI890.ocl36.txt`.

---

### **Process Steps of the RPG Program**

The `BI890P` program is an interactive RPGLE program that manages a subfile-based interface for inquiring, updating, or copying ship-to records. It uses a display file (`bi890pd`) with a subfile (`sfl1`) and handles user input via function keys and direct access options. Here’s a step-by-step breakdown of the process:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Parameters**: Receives `p$fgrp` (file group: 'G' or 'Z') and `p$mode` (mode: 'INQ' for inquiry or 'MNT' for maintenance).
   - **Data Area**: Reads the Local Data Area (LDA) to retrieve company (`l$co`), customer (`l$cust`), ship-to (`l$ship`), and function key (`l$fkey`) values.
   - **Field Setup**: Initializes subfile control fields (e.g., `rrn1`, `rrnsv1`, `pagsz1`=32 for page size), work fields, and key lists (`kls1s1`, `kls1s2`, `klcust`, `klcust2`, `klshipto`) for file access.
   - **Date Handling**: Converts system date (`ps#mdy`) to various formats (e.g., `d#cymd`, `d#ymd`) for consistent date processing.
   - **Message Handling**: Initializes message handling fields (`dspmsg`, `m@pgmq`) for error and status messages.
   - **Mode Check**: If `p$mode` is 'MNT', sets indicator `*in78` to enable maintenance functions.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **File Overrides**: Based on `p$fgrp` ('G' or 'Z'), applies file overrides (`ovg` or `ovz`) to redirect file access to `garcust`, `gbicont`, `gshipto` (for 'G') or `zarcust`, `zbicont`, `zshipto` (for 'Z') using `QCMDEXC`.
   - **Open Files**: Opens `arcust`, `bicont`, and `shipto` files in user-controlled mode (`usropn`) for shared access.

3. **Process Subfile (`srsfl1` Subroutine)**:
   - **Subfile Initialization**:
     - Sets `sfmod1` to '1' (folded mode) and `*in45` for initial subfile display.
     - Clears message subfile (`clrmsg`) and writes it (`wrtmsg`).
     - Sets default company number (`c$co`=10).
   - **Main Loop** (`sf1agn`):
     - Displays command line (`sflcmd1`) and message subfile if needed.
     - Checks if subfile has records (`rrn1 > 0`) to set `*in41` (SFLDSP).
     - Toggles folded/unfolded mode based on `sfmod1`.
     - Displays subfile control (`sflctl1`) using `exfmt`.
     - Processes user input based on function keys or direct access:
       - **F03 (Exit)**: Sets `l$fkey` to 'F03', updates LDA, and exits loop.
       - **F04 (Prompt)**: Calls `prompt` subroutine to assist with field input.
       - **F05 (Refresh)**: Triggers subfile repositioning (`repsfl`).
       - **User Positioning**: If company (`c$co`), customer (`c$cust`), or ship-to (`c$ship`) changes, repositions subfile (`sf1rep`).
       - **Direct Access**: If `d1opt` or `d1ship` is specified, processes direct access (`sf1dir`).
       - **Page Down**: Loads additional subfile records (`sf1lod`).
       - **Enter**: Processes subfile selections (`sf1prc`).
       - **F10**: Moves cursor to control record.
     - Updates cursor location (`csrloc`) and subfile record number (`rcdnb1`).

4. **Subfile Processing (`sf1prc` Subroutine)**:
   - Reads changed subfile records (`readc sfl1`) and processes each via `sf1chg` if not empty.

5. **Subfile Record Change (`sf1chg` Subroutine)**:
   - Processes subfile options (`s1opt`):
     - **Option 2 (Update)**: If `s1ship` is non-zero and maintenance mode (`*in78`), calls `sf1s02` to update ship-to.
     - **Option 3 (Copy)**: Calls `sf1s03` to copy ship-to record.
     - **Option 5 (Inquiry)**: If not in maintenance mode, calls `sf1s05` to display ship-to details.
   - Clears and updates subfile record after processing.

6. **Subfile Repositioning (`sf1rep` Subroutine)**:
   - Clears subfile (`sf1clr`) and validates control fields (`sf1cte`).
   - Positions `shipto` file using key list `kls1s1` (company, customer, ship-to).
   - Loads subfile records (`sf1lod`) and retains control fields for repositioning.

7. **Validate Control Input (`sf1cte` Subroutine)**:
   - Validates company (`c$co`) against `bicont`. If invalid or zero, sets error `ERR0010` and indicators `*in50`, `*in51`.
   - Validates customer (`c$cust`) against `arcust`. If invalid or zero, sets error `ERR0010` and indicators `*in50`, `*in52`.
   - Updates display fields (`c$conm`, `c$csnm`) with company and customer names.

8. **Load Subfile Records (`sf1lod` Subroutine)**:
   - Loads up to `pagsz1` (32) records into the subfile.
   - Reads `shipto` file using key list `kls1r1` (company, customer).
   - If no records are found (`*in43`), adds a "No Results" record (`com(01)`).
   - Formats each record (`sf1fmt`) and writes to subfile.

9. **Format Subfile Line (`sf1fmt` Subroutine)**:
   - Clears subfile record and populates fields (`s1ship`, `s1name`, `s1ctst`, `s1adr1`, `s1adr2`, `s1adr3`, `s1stat`, `s1cszp`) from `shipto` file.
   - Applies color coding (`sf1color`) based on delete flag (`s1del`).

10. **Color Coding (`sf1color` Subroutine)**:
    - Sets `*in77` if ship-to is flagged for deletion (`s1del='D'`), likely for visual highlighting.

11. **Direct Access Processing (`sf1dir` Subroutine)**:
    - Validates direct access input (`d1opt`, `d1ship`):
      - **Option 1 (Create)**: Ensures non-zero company, customer, and ship-to; checks if ship-to doesn’t exist (`*in99`). If it exists, sets error `ERR0101`.
      - **Option 2 (Update)**: Ensures ship-to exists (`*in99`). If not, sets error `ERR0102`.
      - **Option 3 (Copy)**: Processes copy operation.
      - **Option 5 (Inquiry)**: Displays ship-to details.
    - Calls corresponding subroutines (`sf1s01`, `sf1s02`, `sf1s03`, `sf1s05`) if no errors.

12. **Create (`sf1s01`), Update (`sf1s02`), Display (`sf1s05`) Subroutines**:
    - Sets LDA fields (`l$co`, `l$cust`, `l$ship`), clears `l$fkey`, updates LDA, and exits loop (`sf1agn=off`).
    - Updates cursor location.

13. **Copy (`sf1s03` and `sf1cpy` Subroutines)**:
    - Displays a copy window (`sflcpy1`) with customer name (`c$tcnm`) and temporary customer/ship-to fields (`w$tcst`, `w$tshp`).
    - Processes window input:
      - **F04**: Calls `prompt` for field assistance.
      - **F12**: Cancels copy operation.
      - **F22**: Processes copy (not fully implemented).
      - Validates customer (`w$tcst`) and ship-to (`w$tshp`) via `sf1cpyedt`. If valid, calls `BI8903` with parameters for copy operation.

14. **Field Prompting (`prompt` Subroutine)**:
    - For `SFLCTL1`:
      - Company (`C$CO`): Calls `LBICONT` to prompt company selection.
      - Customer (`C$CUST`): Calls `LARCUST` with company and customer parameters.
    - For `SFLCPY1`:
      - Customer (`W$TCST`): Calls `LARCUST` and updates customer name (`c$tcnm`).

15. **Message Handling (`addmsg`, `wrtmsg`, `clrmsg` Subroutines)**:
    - **addmsg**: Sends error messages to program message queue (`QMHSNDPM`) with message ID (`m@id`), file (`m@msgf`), and data (`m@data`).
    - **wrtmsg**: Displays message subfile (`msgctl`) with `*in49`.
    - **clrmsg**: Clears message subfile using `QMHRMVPM` and restores record format and page number.

16. **Program Termination**:
    - Closes all files (`close *all`), sets `*inlr` to end the program, and returns.

---

### **Business Rules**

The program enforces the following business rules:
1. **Validation**:
   - Company (`c$co`) must exist in `bicont` (error `ERR0010` if invalid or zero).
   - Customer (`c$cust`) must exist in `arcust` (error `ERR0010` if invalid or zero).
   - Ship-to (`d1ship`, `w$tshp`) cannot be zero (error `ERR0000`, message `com(02)`).
   - For create/copy (`d1opt=1` or `3`): Ship-to must not exist (`ERR0101` if it does).
   - For update (`d1opt=2`): Ship-to must exist (`ERR0102` if it doesn’t).
   - For copy, new ship-to must not exist (`ERR0000`, message `com(03)`).

2. **Modes**:
   - **Inquiry Mode (`p$mode='INQ'`)**: Allows viewing ship-to details (option 5) without modification.
   - **Maintenance Mode (`p$mode='MNT'`)**: Enables create (option 1), update (option 2), and copy (option 3) operations.

3. **Subfile Behavior**:
   - Displays up to 32 records per page (`pagsz1`).
   - Supports folded/unfolded display modes (`sfmod1`, `*in45`).
   - Highlights deleted records (`s1del='D'`) with color coding (`*in77`).
   - Repositions based on user input (company, customer, ship-to) or direct access.

4. **Function Keys**:
   - **F03**: Exits program, updating LDA with `F03`.
   - **F04**: Prompts for company or customer selection.
   - **F05**: Refreshes subfile.
   - **F10**: Moves cursor to control record.
   - **F12**: Cancels copy operation.
   - **F22**: Processes copy (partially implemented).
   - **Page Down**: Loads next page of subfile records.
   - **Enter**: Processes subfile selections.

5. **Copy Operation**:
   - Validates new customer (`w$tcst`) and ship-to (`w$tshp`) before calling `BI8903`.
   - Ensures new ship-to doesn’t exist to avoid duplication.

---

### **Tables (Files) Used**

The program uses the following files, as declared in the RPGLE code and overridden in the OCL:
1. **bi890pd** (Display File, `cf`):
   - Work station file with subfile `sfl1` for interactive display.
2. **arcust** (Input File, `if`):
   - Customer master file (`garcust` or `zarcust` based on `p$fgrp`).
   - Used to validate customer numbers and retrieve names.
3. **bicont** (Input File, `if`):
   - Control file (`gbicont` or `zbicont`).
   - Used to validate company numbers and retrieve names.
4. **shipto** (Input File, `if`):
   - Ship-to master file (`gshipto` or `zshipto`).
   - Stores ship-to details (e.g., address, status, city/state/ZIP).

The OCL program declares additional files (`arcupr`, `gsprod`, `gstabl`, `trrtcd`, `cuadr`, `cuadrrd`, `bbora1`, `bbordh`, `bbshsa1`, `shipths`, `arcuphs`), but only `arcust`, `bicont`, and `shipto` are explicitly used in `BI890P`.

---

### **External Programs Called**

The program calls the following external programs:
1. **QCMDEXC**:
   - System program to execute file override commands (`ovg` or `ovz`) for `arcust`, `bicont`, and `shipto`.
2. **QMHSNDPM**:
   - Sends error messages to the program message queue.
3. **QMHRMVPM**:
   - Clears messages from the message subfile.
4. **LBICONT**:
   - Called in `prompt` subroutine to assist with company number selection.
   - Parameters: `c$co` (input/output), `p$fgrp` (input), `o$fgrp` (output).
5. **LARCUST**:
   - Called in `prompt` subroutine to assist with customer number selection.
   - Parameters: `c$co`/`o$co` (input/output), `c$cust`/`o$cust` or `w$tcst`/`o$cust` (input/output), `p$fgrp` (input), `o$fgrp` (output).
6. **BI8903**:
   - Called in `sf1cpy` to perform ship-to copy operation.
   - Parameters: `c$co`, `c$cust`, `s$ship`, `w$tcst`, `w$tshp`, `p$fgrp`.

The OCL references `BI9002` and `BB800E`, but they are not called directly in `BI890P`. They may be invoked by `BI8903` or other programs in the chain.

---

### **Summary**

The `BI890P` program is an interactive RPGLE application for managing ship-to master inquiries and maintenance. It:
- Displays a subfile with ship-to records, supporting inquiry, update, and copy operations.
- Validates input against `bicont`, `arcust`, and `shipto` files.
- Handles user input via function keys (F03, F04, F05, F10, F12, F22, Page Down, Enter).
- Enforces business rules for data validation and mode-specific operations.
- Integrates with the OCL program (`BI890.ocl36.txt`) by using shared files and parameters.

**Tables Used**:
- `bi890pd` (display file with subfile `sfl1`)
- `arcust` (`garcust` or `zarcust`)
- `bicont` (`gbicont` or `zbicont`)
- `shipto` (`gshipto` or `zshipto`)

**External Programs Called**:
- `QCMDEXC`
- `QMHSNDPM`
- `QMHRMVPM`
- `LBICONT`
- `LARCUST`
- `BI8903`

If you need a deeper analysis of specific subroutines, business rules, or interactions with the OCL, let me know!