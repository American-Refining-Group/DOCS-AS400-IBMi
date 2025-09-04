The provided document is an RPGLE program named `BB943P`, called from the OCL procedure `BB943INQ.ocl36.txt` in the IBM System/36 (or AS/400 compatibility mode) environment. This program is designed to "Work With Customer Sales Agreements," providing functionality for inquiring, maintaining, copying, or deleting customer sales agreement records. Below, I’ll explain the process steps, business rules, tables used, and external programs called, based on the provided RPGLE source code and the OCL context.

---

### Process Steps of the RPGLE Program

The `BB943P` program is an interactive RPGLE program that manages a subfile-based display for customer sales agreements. It supports inquiry (`INQ`) and maintenance (`MNT`) modes, allowing users to view, add, update, copy, or delete sales agreement records. Here’s a detailed breakdown of the process steps:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Receive Parameters**: The program accepts two parameters via the `*ENTRY` plist:
     - `p$mode` (3 characters): Specifies the run mode (`INQ` for inquiry or `MNT` for maintenance).
     - `p$fgrp` (1 character): Specifies the file group (`G` or `Z`) for file overrides.
   - **Set Up Work Fields**: Initializes variables, such as `rrn1` (subfile relative record number), `pagsz1` (page size set to 28), and message handling fields (`dspmsg`, `m@pgmq`).
   - **Set Current Date/Time**: Uses the `TIME` operation to populate `t#time` and converts it to a date format (`t#cymd`) for validations.
   - **Define Key Lists**: Sets up key lists (`kls1s1`, `kls1s2`, etc.) for file access based on fields like company (`c1cono`), customer (`c1cust`), location (`c1loc`), and container (`c1cntr`).
   - **Set Default Values**: Initializes fields like `c1stat` to `'CF '` (likely a default status) and `o$file` to `'BICUAG'` (primary file).

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **File Overrides**: Based on `p$fgrp` (`G` or `Z`), applies file overrides using `ovg` or `ovz` arrays (e.g., `ovrdbf file(arcust) tofile(*libl/garcust)` for `G` group). This ensures the program accesses the correct file library (`G` or `Z`).
   - **Open Files**: Opens all defined files (`arcust`, `bicont`, `bicuag`, `gscntr1`, `gsprod`, `gstabl`, `inloc`, `shipto`, `bicuax`, `bicua10`) with `usropn` for controlled access.

3. **Process Subfile (`srsfl1` Subroutine)**:
   - **Clear Message Subfile**: Calls `clrmsg` to clear any existing messages in the message subfile.
   - **Initialize Subfile Mode**: Sets `sfmod1` to `'1'` (folded mode) and `*in45` (subfile fold indicator) to `*on` for the initial subfile load.
   - **Set Global Protection**: If `p$mode` is `'INQ'`, sets `*in70` to `*on` to protect input fields (read-only mode). In `'MNT'` mode, fields are editable (`*in70` is `*off`).
   - **Initialize Filters**:
     - Sets `w$expd` to `*off` (exclude expired entries by default) and assigns `F8=Incl Del` or `F8=Excl Del` to `s1f08d` based on the `fky` array.
     - Sets `f13sel` to `'D'` (descending order) and assigns `F13=Descend` to `s1f13s` (updated by `jk02` revision for ascending/descending toggle).
   - **Position File**: Calls `sf1rep` to position the file based on user input (e.g., company, customer, location).
   - **Main Loop (`sf1agn`)**:
     - Displays the command line (`sflcmd1`) and message subfile (`msgctl` or `msgclr`).
     - Checks if subfile records exist (`rrn1 > 0`) to set `*in41` (subfile display indicator).
     - Sets fold/unfold mode (`*in45`) based on `sfmod1`.
     - Displays the subfile control format (`sflctl1`) using `EXFMT`.
     - Processes user input based on function keys:
       - **F3 (Exit)**: Exits the loop (`sf1agn = *off`).
       - **F4 (Prompt)**: Calls `prompt` to display selection windows for fields like customer (`C1CUST`), ship-to (`C1SHIP`), location (`C1LOC`), container (`C1CNTR`), or product code (`C1PRCD`).
       - **F5 (Refresh)**: Sets `repsfl` to `*on` to reload the subfile.
       - **F8 (Toggle Expired)**: Toggles `w$expd` between include/exclude expired entries and reloads the subfile.
       - **F9 (History Inquiry)**: Calls `histinq` to invoke `GB730P` for customer history.
       - **F13 (Toggle Ascend/Descend)**: Toggles `f13sel` between `'A'` (ascending) and `'D'` (descending) and reloads the subfile (added by `jk02`).
       - **Page Down**: Calls `sf1lod` to load more subfile records.
       - **Enter**: Validates password (`validatepw`) and processes subfile selections (`sf1prc`).
       - **F6 (Add)**: Validates password and calls `sf1add` to add a new record.
       - **F10 (Position Cursor)**: Clears cursor position (`row1`, `col1`).
       - **User Repositioning**: If control fields (e.g., `c1cono`, `c1cust`) change, calls `sf1rep` to reposition the subfile.

4. **Process Subfile Selections (`sf1prc` Subroutine)**:
   - Reads subfile records (`READC sfl1`) and processes each selected record by calling `sf1chg` if `*in81` is `*off` (record found).

5. **Process Subfile Record Changes (`sf1chg` Subroutine)**:
   - Processes user selections based on `s1opt` (subfile option field):
     - **Option 2 (Update)**: Calls `sf1s02` if in `'MNT'` mode and record is not deleted (`s1del ≠ 'D'`).
     - **Option 3 (Copy)**: Calls `sf1s03` if in `'MNT'` mode and record is not deleted.
     - **Option 4 (Delete)**: Calls `sf1s04` if in `'MNT'` mode.
     - **Option 5 (Display)**: Calls `sf1s05` to display the customer order.
   - Updates the subfile record after processing by chaining to `bicuag`, formatting fields (`sf1fmt`, `sf1col`), and updating `sfl1`.

6. **Reposition Subfile (`sf1rep` Subroutine)**:
   - Clears the subfile (`sf1clr`) and resets `rrn1`.
   - Validates control fields (`sf1cte`) for company (`c1cono`), customer (`c1cust`), etc.
   - Positions the file (`bicuax` or `bicua10`) based on `f13sel` (ascending/descending, per `jk02`) using key lists (`kls1s1` or `kls1s2`).
   - Handles partial PO data entry (`c1pord`) by calculating its length (`w$slen`, per `jk06`).
   - Loads the subfile (`sf1lod`) and retains control fields for future repositioning.

7. **Edit Subfile Control Input (`sf1cte` Subroutine)**:
   - Validates input fields:
     - **Company (`c1cono`)**: Chains to `bicont`. If invalid or zero, sets error `ERR0010` and `*in50`, `*in51`.
     - **Customer (`c1cust`)**: Chains to `arcust`. If invalid, sets an error (not fully shown due to truncation).
   - Additional validations are likely performed for other fields, but the code is truncated.

8. **Copy Record (`sf1s03` Subroutine)**:
   - Calls `BB943` to copy a record, passing parameters like `c1cono`, `s1seqn` (sequence number), `w$validpw` (password validation), and `o$mode` (`'MNT'`).
   - Processes return flag (`o$flag`):
     - `'1'`: Record copied, displays message from `com(04)` and repositions subfile.
     - `'0'`: Record not copied, displays message from `com(09)` and sets `*in60`.
   - If `o$MsgFlag = '1'` (per `jk03`), displays an expiration message (`com(11)`).

9. **Delete Record (`sf1s04` Subroutine)**:
   - Calls `BB944` to delete a record, passing `c1cono`, `s1seqn`, `p$fgrp`, and `o$flag`.
   - Processes return flag:
     - `'D'`: Record marked deleted, displays message from `com(05)`.
     - `'A'`: Record marked active, displays message from `com(06)`.

10. **Display Customer Order (`sf1s05` Subroutine)**:
    - Calls `BB943` in `'INQ'` mode to display the record, passing similar parameters as in `sf1s03`.

11. **History Inquiry (`histinq` Subroutine)**:
    - Calls `GB730P` with `x$custhist` data structure to display customer history.

12. **Field Prompting (`prompt` Subroutine)**:
    - Handles field prompting for input fields using external programs:
      - `C1CUST`: Calls `LARCUST` to select a customer.
      - `C1SHIP`: Calls `LCSTSHP` to select a ship-to address.
      - `C1LOC`: Calls `LINLOC` to select a location.
      - `C1CNTR`: Calls `LGSCNCD` to select a container code.
      - `C1PRCD`: Calls `LGSPROD` to select a product code.
    - Updates control fields with selected values.

13. **Message Handling (`addmsg`, `wrtmsg`, `clrmsg` Subroutines)**:
    - **Add Message (`addmsg`)**: Sends messages to the program message queue using `QMHSNDPM`.
    - **Write Message (`wrtmsg`)**: Displays the message subfile (`msgctl`).
    - **Clear Message (`clrmsg`)**: Clears the message subfile using `QMHRMVPM`.

14. **Program Termination**:
    - Closes all files (`close *all`).
    - Sets `*inlr = *on` and returns.

---

### Business Rules

The program enforces several business rules for managing customer sales agreements:

1. **Mode-Based Access**:
   - In `'INQ'` mode, fields are protected (`*in70 = *on`), allowing only viewing.
   - In `'MNT'` mode, fields are editable, enabling add, update, copy, and delete operations.

2. **Subfile Options**:
   - **Option 2 (Update)**: Allowed in `'MNT'` mode for non-deleted records (`s1del ≠ 'D'`).
   - **Option 3 (Copy)**: Allowed in `'MNT'` mode for non-deleted records, calls `BB943` to copy records.
   - **Option 4 (Delete)**: Allowed in `'MNT'` mode, calls `BB944` to mark records as deleted or active.
   - **Option 5 (Display)**: Displays record details via `BB943` in `'INQ'` mode.

3. **Validation**:
   - **Company (`c1cono`)**: Must exist in `bicont` file.
   - **Customer (`c1cust`)**: Must exist in `arcust` file.
   - **Password Validation**: Calls `validatepw` for add (`F6`) and enter operations to ensure valid access.
   - **Expired Records**: Toggles inclusion/exclusion via `F8` (`w$expd`). Displays expiration messages if `o$MsgFlag = '1'` (per `jk03`).
   - **Partial PO Data Entry**: Supports partial purchase order input (`c1pord`) with length calculation (per `jk06`).

4. **Sorting**:
   - Toggles between ascending and descending order via `F13` (`f13sel = 'A'` or `'D'`, per `jk02`).

5. **Error Handling**:
   - Displays error messages (e.g., `ERR0010` for invalid company) using the message subfile.
   - Uses `com` array for predefined messages (e.g., "Record has been copied," "Record has been deleted").

6. **Container Type Restriction**:
   - Per revision `jb07` (07/22/2024), the ability to enter container type in a sales agreement was removed.

7. **Price Display**:
   - If the off-price field (`BAOFFP`) is non-zero, the subfile price field is displayed in yellow (per `jk04`).

8. **Default Password**:
   - In add mode, defaults password flag to `'N'` if blank (per `jk01`).

---

### Tables Used

The program uses the following files, as defined in the File Specification (F-spec) section:

1. **bb943pd**: Display file (workstation file) with a subfile (`sfl1`) for interactive user interface.
2. **arcust**: Customer master file, input-only, keyed.
3. **bicont**: Company file, input-only, keyed.
4. **bicuag**: Customer sales agreement file, input-only, keyed.
5. **gscntr1**: Container file (replaced `gscntr` per `jk05`), input-only, keyed. Renames field `tpfil5` to `tcfil5` to avoid conflict with `gsprod`.
6. **gsprod**: Product file, input-only, keyed.
7. **gstabl**: Table file (likely for codes or configurations), input-only, keyed.
8. **inloc**: Location file, input-only, keyed.
9. **shipto**: Ship-to address file, input-only, keyed.
10. **bicuax**: Alternate sales agreement file, input-only, keyed, with renamed record format (`bicuagpf` to `bicuagp1`).
11. **bicua10**: Another alternate sales agreement file (added per `jk02`), input-only, keyed, with renamed record format (`bicuagpf` to `bicuagp2`).

**File Overrides**:
- The `ovg` and `ovz` arrays specify overrides for file groups `G` and `Z`, mapping files to libraries (e.g., `garcust`, `zbicuag`).

---

### External Programs Called

The program calls the following external programs:

1. **BB943**:
   - Called in `sf1s03` (copy) and `sf1s05` (display) subroutines.
   - Parameters include `c1cono`, `s1stat`, `w$validpw`, `s1seqn`, `o$frsq`, `c1f6flag`, `c1cust`, `c1loc`, `c1cntr`, `c1prcd`, `c1prim`, `c1ship`, `o$mode`, `p$fgrp`, `o$flag`, and `o$MsgFlag`.
   - Used for copying or displaying sales agreement records.

2. **BB944**:
   - Called in `sf1s04` (delete) subroutine.
   - Parameters include `c1cono`, `s1seqn`, `p$fgrp`, and `o$flag`.
   - Handles deletion or reactivation of records.

3. **GB730P**:
   - Called in `histinq` subroutine.
   - Parameter: `x$custhist` (data structure with company, customer, location, container, etc.).
   - Displays customer history.

4. **LARCUST**:
   - Called in `prompt` subroutine for `C1CUST`.
   - Parameters: `o$cono`, `o$cust`, `o$fgrp`.
   - Allows selection of a customer code.

5. **LCSTSHP**:
   - Called in `prompt` subroutine for `C1SHIP`.
   - Parameter: `x$cstshp` (data structure with company, search, customer, ship-to, flag, file group).
   - Allows selection of a ship-to address.

6. **LINLOC**:
   - Called in `prompt` subroutine for `C1LOC`.
   - Parameters: `c1cono`, `c1loc`, `o$fgrp`.
   - Allows selection of a location code.

7. **LGSCNCD**:
   - Called in `prompt` subroutine for `C1CNTR`.
   - Parameters: `c1cntr`, `o$fgrp`.
   - Allows selection of a container code.

8. **LGSPROD**:
   - Called in `prompt` subroutine for `C1PRCD`.
   - Parameters: `c1cono`, `c1prcd`, `o$fgrp`.
   - Allows selection of a product code.

9. **QCMDEXC**:
   - Called in `opntbl` to execute file override commands.
   - Parameters: `dbov##` (override command), `dbol##` (length).

10. **QMHSNDPM**:
    - Called in `addmsg` to send messages to the program message queue.
    - Parameters: `m@id`, `m@msgf`, `m@data`, `m@l`, `m@type`, `m@pgmq`, `m@scnt`, `m@key`, `m@errc`.

11. **QMHRMVPM**:
    - Called in `clrmsg` to clear messages from the message queue.
    - Parameters: `m@pgmq`, `m@scnt`, `m@rmvk`, `m@rmv`, `m@errc`.

---

### Summary

- **Process Steps**:
  1. Initialize parameters, work fields, and key lists.
  2. Apply file overrides and open database files.
  3. Process the subfile (`srsfl1`): clear messages, set modes, position file, and handle user input (F3, F4, F5, F6, F8, F9, F10, F13, Enter, Page Down).
  4. Process subfile selections (`sf1prc`, `sf1chg`) for update, copy, delete, or display.
  5. Reposition subfile (`sf1rep`) based on user input and sorting preferences.
  6. Validate control fields (`sf1cte`) for company and customer.
  7. Handle copy (`sf1s03`), delete (`sf1s04`), and display (`sf1s05`) operations.
  8. Support history inquiry (`histinq`) and field prompting (`prompt`).
  9. Manage messages (`addmsg`, `wrtmsg`, `clrmsg`).
  10. Close files and terminate.

- **Business Rules**:
  - Supports inquiry and maintenance modes with protected fields in `INQ`.
  - Validates company, customer, and other inputs.
  - Handles subfile options (update, copy, delete, display).
  - Toggles expired record inclusion and sort order.
  - Restricts container type entry (per `jb07`).
  - Displays price in yellow if `BAOFFP` is non-zero (per `jk04`).
  - Defaults password flag to `'N'` in add mode (per `jk01`).

- **Tables Used**:
  - `bb943pd` (display file), `arcust`, `bicont`, `bicuag`, `gscntr1`, `gsprod`, `gstabl`, `inloc`, `shipto`, `bicuax`, `bicua10`.

- **External Programs Called**:
  - `BB943`, `BB944`, `GB730P`, `LARCUST`, `LCSTSHP`, `LINLOC`, `LGSCNCD`, `LGSPROD`, `QCMDEXC`, `QMHSNDPM`, `QMHRMVPM`.

If you have additional files (e.g., `GSY2K`, display file `bb943pd`, or source for called programs like `BB943`), I can provide further details. Let me know if you need specific clarifications or analysis of related components!