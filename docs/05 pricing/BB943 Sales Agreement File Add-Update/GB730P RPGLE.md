The RPG program `GB730P.rpgle` is an IBM i (AS/400) program designed for "Global File Tracking History Inquiry." It provides an interactive interface to display historical changes to various database files related to sales, customer, and order data. The program is called from the main OCL and uses a display file with a subfile to present tracking history. Below, I will explain the process steps, business rules, tables used, and external programs called based on the provided RPG source code.

---

### Process Steps of the RPG Program

The `GB730P` program displays historical data for multiple files, allowing users to inquire about changes to records such as customer sales agreements, pricing, orders, and more. The key process steps are:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Parameters**: Receives input parameters (not fully detailed in the provided code due to truncation, but typically include selection criteria like company, customer, or file type).
   - **Field Setup**: Initializes:
     - Date conversion data structure (`d#cymd`) for formatting dates (e.g., CCYYMMDD, MMDDYY).
     - Message subfile data structures (`m@msgf`, `m@data`, `m@type`, etc.) for error and status messages.
     - Display file data structure (`INFDS`) to track record format (`RCDNAM`), function key pressed (`KEY`), cursor location (`CSRLOC`), and subfile relative record number (`PAGRRN`).
     - Program status data structure (`PSDS##`) to capture program name (`PGMQ##`), job details (`JOB##`, `USER##`), and date/time (`ERR`, `PARMS`).
     - Work fields like `r$prod`, `h$prod`, `h$user`, `r$user` for product codes and user IDs.
   - **Current Date/Time**: Captures the current date and time for comparison or display purposes.

2. **Open Database Tables (`opntbl` Subroutine, Implicit)**:
   - Applies file overrides (`OVG` or `OVZ`) based on a file group parameter (likely `p$fgrp`, 'G' or 'Z') using the `QCMDEXC` API, mapping files to appropriate libraries (e.g., `gglcont` or `zglcont`).
   - Opens multiple input files (e.g., `glcont`, `shiptz`, `bicuag`, `bbordh`) with `USROPN` for querying historical data.

3. **Main Processing (`main` Subroutine, Implicit)**:
   - **Display File Setup**: Uses `gb730pd` as the display file with a subfile (`SFL1`, relative record number `RRN1`) to present historical data.
   - **Retrieve Data**: Queries history files (e.g., `bicuaghx`, `bbordhsx`) based on user input (e.g., company, customer, or file type) to retrieve change records.
   - **Populate Subfile**: Loads historical data into the subfile (`SFL1`), including fields like user ID (`h$user`, `r$user`), change date (`h$fdat`, `h$tdat`), and product codes (`h$prod`, `r$prod`).
   - **User Interaction**:
     - Displays the subfile with headers (`HDR`) indicating the file being tracked (e.g., "Customer Sales Agreement Tracking").
     - Processes function keys (e.g., F3 to exit, page down to scroll subfile).
     - Handles errors by displaying messages from the `COM` array (e.g., "Cannot Enter both Sys And Program Id").
   - **Field Mapping**:
     - Maps fields like `BAPRCE` and `BAOFFP` (9.4 packed, per `jk02`) for sales agreement pricing.
     - Includes `BFTSEQ` for freight tracking (`jk03`) and `BAPOOR` for PO/Order code (`jk06`).
     - Adjusts `BKFSPC` in `bbcfsd` to 5.2 packed (`jb07`, `jb08`).

4. **Message Handling**:
   - **Add Message**: Uses `QMHSNDPM` to send diagnostic messages to the program message queue.
   - **Clear Message**: Uses `QMHRMVPM` to clear the message subfile.
   - **Display Messages**: Shows error or status messages in the subfile control record.

5. **Program Termination**:
   - Closes all files (`close *all`).
   - Sets `*inlr = *on` and returns.

---

### Business Rules

The program enforces the following business rules for tracking history inquiries:

1. **File Selection**:
   - Supports querying history for multiple files, including customer sales agreements (`bicuag`, `bicuaghx`), orders (`bbordh`, `bbordd`, `bbordm`), pricing (`bbprce`), freight (`bbcfsd`), and more, as listed in the `HDR` array.

2. **Data Validation**:
   - Prevents simultaneous entry of system and program IDs (`COM(01)`: "Cannot Enter both Sys And Program Id").
   - Validates input parameters (e.g., company, customer) against master files like `glcont`, `arcusz`, or `shiptz`.

3. **Field Formatting**:
   - Handles expanded fields like `BAPRCE` and `BAOFFP` (9.4 packed, per `jk02`).
   - Supports new fields like `BFTSEQ` for freight tracking (`jk03`) and `BAPOOR` for PO/Order code (`jk06`, live 06/2024).
   - Adjusts `BKFSPC` in `bbcfsd` to 5.2 packed (`jb07`, `jb08`).

4. **History Tracking**:
   - Displays change records from history files (e.g., `bicuaghx`, `bbordhsx`, `bbormhsx`) with details like user ID, change date/time, and modified fields.
   - Renames `BDDDEL` to `BDDEL` in `bbordd` history data structure (`jk05`).

5. **File Group Overrides**:
   - Uses `p$fgrp` ('G' or 'Z') to apply appropriate file overrides, ensuring access to the correct dataset (e.g., `gbicuag` or `zbicuag`).

6. **Removed Tables**:
   - Tables `empmas` and `emphisz` were removed (`jb07`), indicating a shift away from employee-related history tracking.

---

### Tables Used

The program uses the following database files, with overrides applied based on `p$fgrp` ('G' or 'Z'):

1. **glcont**: Company master file (input, validates company data).
2. **shiptz**, **shipthx**: Ship-to files (input, for ship-to details).
3. **arcup31**, **arcuphx**: Customer price files (input, for pricing data).
4. **cuadrx**, **cuadrhx**: Customer address files (input, for address data).
5. **arcusz**, **arcushx**, **arcuspx**, **arcusphx**: A/R customer master and supplemental files (input, for customer data).
6. **bicuag**, **bicuaghx**: Sales agreement and history files (input, for agreement tracking).
7. **bbprce**, **bbprcehx**: Rack pricing files (input, for pricing data).
8. **bbndfi**, **bbndfihx**: National diesel fuel index files (input, for fuel index data).
9. **bbcfsh**, **bbcfshhx**, **bbcfsd**, **bbcfsdhx**: Carrier fuel surcharge header and detail files (input, for freight surcharge data).
10. **bicuf2**, **bicufrhx**: Customer freight files (input, for freight data).
11. **gsprod**, **gsprodhx**: Product files (input, for product data).
12. **gshazm**, **gshazmhx**: Hazmat/shipping description files (input, for hazmat data).
13. **gsumcv**, **gsumcvhx**: Product unit of measure conversion files (input, for U/M conversions).
14. **gsctum**, **gsctumhx**: Container unit of measure files (input, for container U/M data).
15. **gsctwt**, **gsctwthx**: Container weight files (input, for container weights).
16. **biprt2**, **biprtxhx**: Product tax code files (input, for tax codes).
17. **gscntr1**, **gscntrhx**: Container code files (input, alpha key, added per `jk03`).
18. **bbordh**, **bborhhsx**: Open order header files (input, for order headers, added per `jk04`).
19. **bbordd**, **bbordhsx**: Open order detail files (input, for order details, added per `jk04`).
20. **bbordm**, **bbormhsx**: Open order miscellaneous files (input, for misc. order data, added per `jk04`).
21. **gbfmsg**: Message file (input, for error/status messages).

Overrides map these files to specific libraries (e.g., `gbicuag` or `zbicuag`).

---

### External Programs Called

The program calls the following external programs:

1. **QCMDEXC**: System API to execute file override commands for the specified file group ('G' or 'Z').
2. **QMHSNDPM**: System API to send diagnostic messages to the program message queue.
3. **QMHRMVPM**: System API to clear messages from the message subfile.

No other user-defined programs are explicitly called in the provided source code.

---

### Summary

- **Process Steps**: Initializes parameters, applies file overrides, opens database files, retrieves historical data from multiple files, populates a subfile for display, handles user interactions via function keys, and manages error messages.
- **Business Rules**: Supports querying history for various files, validates input parameters, prevents simultaneous system/program ID entry, handles expanded fields (`BAPRCE`, `BAOFFP`, `BKFSPC`), supports new fields (`BFTSEQ`, `BAPOOR`), and logs changes with user and date/time details.
- **Tables Used**: `glcont`, `shiptz`, `shipthx`, `arcup31`, `arcuphx`, `cuadrx`, `cuadrhx`, `arcusz`, `arcushx`, `arcuspx`, `arcusphx`, `bicuag`, `bicuaghx`, `bbprce`, `bbprcehx`, `bbndfi`, `bbndfihx`, `bbcfsh`, `bbcfshhx`, `bbcfsd`, `bbcfsdhx`, `bicuf2`, `bicufrhx`, `gsprod`, `gsprodhx`, `gshazm`, `gshazmhx`, `gsumcv`, `gsumcvhx`, `gsctum`, `gsctumhx`, `gsctwt`, `gsctwthx`, `biprt2`, `biprtxhx`, `gscntr1`, `gscntrhx`, `bbordh`, `bborhhsx`, `bbordd`, `bbordhsx`, `bbordm`, `bbormhsx`, `gbfmsg`.
- **External Programs Called**: `QCMDEXC`, `QMHSNDPM`, `QMHRMVPM`.

If you need further analysis of specific subroutines, file structures, or integration with `BB943P`, `BB943`, `BB944`, or `BB9433`, please provide additional details or source code. Alternatively, I can perform a DeepSearch for related information if enabled. Let me know how to proceed!