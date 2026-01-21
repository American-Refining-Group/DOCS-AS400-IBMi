The RPG program `GB730P.rpgle` is a Global File Tracking History Inquiry program within the Bradford Order Entry/Invoices system. It is designed to display historical changes to various master and transaction files, allowing users to review updates, additions, or deletions for records such as customer data, pricing, fuel surcharges, and order details. The program is called by other programs (e.g., `BB910.rpgle`, `BB911.rpgle`) and uses a subfile (`SFL1`) to present historical data in a user-friendly format. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called, based on the provided source code and context.

---

### Process Steps of `GB730P.rpgle`

The program operates through a series of subroutines to initialize, open files, display historical data in a subfile, and handle user interactions. The main steps are:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Receives Input Parameters** (via `*entry` plist):
     - `p$file`: File name (e.g., `BBNDFI`, `BBCFSD`, `BBORDH`) to determine which history file to query.
     - `p$fgrp`: File group ('Z' or 'G' for library selection).
     - `p$co`: Company number.
     - `p$efdt`: Effective date (CCYYMMDD).
     - `p$eftm`: Effective time (HHMM, for files requiring time-based keys).
     - Additional parameters (e.g., `p$regn` for region, `p$rprc` for retail price, `p$ord#` for order number) depending on the file.
   - Initializes the display file (`gb730pd`) and subfile (`SFL1`) fields, setting `rrn1` (relative record number) to zero.
   - Sets the program message queue (`m@pgmq = '*'`) and initializes message handling fields.
   - Converts the current system date (`ps#mdy`) to CCYYMMDD format (`ps#dat`) for comparison.
   - Applies file overrides based on `p$fgrp` ('Z' for `z*` libraries, `G` for `g*` libraries) using the `OVG` or `OVZ` arrays.
   - Sets the report header (`c$hdr1`) based on the file name (e.g., "National Diesel Fuel Index Tracking" for `BBNDFI`).
   - Initializes key lists (e.g., `klsfl1`, `klhist`) for accessing history files.

2. **Open Database Tables**:
   - Executes file overrides for all files using `QCMDEXC` based on `p$fgrp`.
   - Opens all input files (e.g., `glcont`, `shiptz`, `bbndfi`, `bbndfihx`) in user-open mode (`USROPN`).
   - Opens the display file `gb730pd` for user interaction.

3. **Main Processing Loop (`srsfl1` Subroutine)**:
   - Clears the subfile (`SFL1`) and message subfile (`clrmsg`).
   - Positions the file cursor (`sf1rep`) to load historical records into `SFL1`.
   - Enters a loop (`sf1agn`) to display and process the subfile:
     - Writes the command line (`sflcmd1`) and message subfile (`wrtmsg`) if errors exist.
     - Sets subfile display indicators (`*in41` for `SFLDSPCTL`, `*in42` for `SFLDSP`) based on whether records exist.
     - Displays the subfile control format (`sflctl1`) using `EXFMT`.
     - Processes user input:
       - **F03/F12**: Exits the program or returns to the previous screen.
       - **F05**: Refreshes the subfile, clearing positioning fields (e.g., `c1cust`, `c1efdt`).
       - **Page Down**: Loads additional records (`sf1lod`).
       - **Enter**: Processes subfile selections (`sf1prc`).
     - Handles repositioning requests based on control fields (e.g., `c1co`, `c1cust`, `c1efdt`) and repositions the subfile (`sf1rep`).
     - Sets the cursor position (`row1`, `col1`) for the next display.

4. **Reposition Subfile (`sf1rep` Subroutine)**:
   - Clears `SFL1` and resets `rrn1`.
   - Validates control fields (e.g., company, customer, effective date) using the appropriate file (e.g., `glcont` for company, `arcusz` for customer).
   - Positions the cursor in the history file (e.g., `bbndfihx` for `BBNDFI`) using a key list (`klhist`) based on the file’s key fields (e.g., company, effective date, time, region).
   - Loads historical records into `SFL1` (`sf1lod`).

5. **Load Subfile (`sf1lod` Subroutine)**:
   - Reads records from the appropriate history file (e.g., `bbndfihx`, `bbcfsdhx`, `bbordhsx`) based on the input file name (`p$file`).
   - Filters records based on control fields (e.g., `c1co`, `c1efdt`, `c1regn`) and ensures only relevant records are loaded.
   - Formats each subfile line (`sf1fmt`) based on the file type, mapping history fields (e.g., `lhco`, `lhefdt`, `lhregn`, `lhrprc`) to subfile fields (`s1co`, `s1efdt`, `s1regn`, `s1rprc`).
   - Applies color coding (`sf1col`) to highlight deleted records (`lhdel = 'D'`) or other conditions.
   - Writes records to `SFL1` until the page size (typically 14 records) is reached or no more records are available.

6. **Format Subfile Line (`sf1fmt` Subroutine)**:
   - Maps history file fields to subfile fields, adjusting for file-specific formats (e.g., date conversion to MMDDYY, price formatting).
   - Retrieves descriptions (e.g., customer name from `arcusz`, region description from `gscntr1`) for display.
   - Handles file-specific fields (e.g., `lhrprc` for `BBNDFI`, `lhord#` for `BBORDH`).

7. **Color Coding (`sf1col` Subroutine)**:
   - Applies color indicators:
     - Turquoise (`*in73`): Deleted records (`lhdel = 'D'`).
     - Other indicators (e.g., `*in72` for expired records) may be used based on file-specific logic.

8. **Process Subfile on Enter (`sf1prc` Subroutine)**:
   - Reads changed subfile records (`READC`) and processes user-selected options (e.g., `s1opt`):
     - **5**: Displays detailed history record information in a window (`srwin`).
   - Updates the subfile if necessary (e.g., clearing options).

9. **Display Detail Window (`srwin` Subroutine)**:
   - Displays detailed field values for the selected history record in a pop-up window.
   - Formats data based on the file type (e.g., showing region, price, and deletion status for `BBNDFI`).
   - Allows users to review changes without modification.

10. **Message Handling**:
    - **Add Message (`addmsg`)**: Sends error messages (e.g., "Cannot Enter both Sys And Program Id") to the program message queue using `QMHSNDPM`.
    - **Write Message (`wrtmsg`)**: Displays messages in the message subfile.
    - **Clear Message (`clrmsg`)**: Clears the message subfile.

11. **Program End**:
    - Closes all open files and the display file (`gb730pd`).
    - Sets `*inlr = *on` and returns control to the calling program.

---

### Business Rules

The program enforces the following business rules to ensure accurate and secure history inquiry:
1. **File-Specific History Retrieval**:
   - The program dynamically adjusts its behavior based on the input file name (`p$file`), querying the corresponding history file (e.g., `bbndfihx` for `BBNDFI`, `bbcfsdhx` for `BBCFSD`).
   - Key fields (e.g., company, effective date, region, order number) are used to filter historical records.

2. **Validation of Control Fields**:
   - Company (`c1co`) must exist in `glcont`.
   - Customer (`c1cust`) must exist in `arcusz`.
   - Effective date (`c1efdt`) is validated using `GSDTEDIT` (assumed, not explicitly shown in the truncated code).
   - File-specific fields (e.g., region, order number) are validated against corresponding master files (e.g., `gscntr1` for regions).

3. **Deletion Handling**:
   - Deleted records (`lhdel = 'D'`) are displayed with turquoise coloring (`*in73`) to distinguish them from active records.

4. **Security and Proprietary Information**:
   - The program includes a security statement indicating that it is proprietary to WCI International Company, prohibiting unauthorized use.

5. **Display Modes**:
   - Users can filter records using control fields (e.g., company, customer, effective date) to narrow the history inquiry.
   - The subfile supports pagination (`Page Down`) and refresh (`F05`) to reload records based on updated filters.

6. **Revisions**:
   - **JK01 (01/09/2013)**: Moved customer number from a parameter to a screen field for better user interaction.
   - **JK02 (10/05/2014)**: Expanded pricing fields (`BAPRCE`, `BAOFFP`) to 9.4 packed for `bbprce` and `bbprcehx`.
   - **JK03 (06/01/2015, 08/24/2015)**: Added support for `BFTSEQ` in `bicufr` and the `gscntr1` table for contract codes.
   - **JK04 (03/19/2017)**: Added support for order-related files (`bbordh`, `bbordd`, `bbordm`) and their history files.
   - **JK05 (04/15/2017)**: Renamed `BDDDEL` to `BDDEL` in the `HISTBBORDD` data structure for consistency.
   - **JK06 (08/04/2022, live 06/2024)**: Added support for `BAPOOR` in `bicuag` for purchase order-related data.
   - **JB07/JB08 (06/19/2024, 07/01/2024)**: Revised `BKFSPC` in `bbcfsd` and `bbcfsdhx` from 3.1 to 5.2 packed for increased precision; removed `empmas` and `emphisz` tables.

---

### Tables (Files) Used

The program accesses the following files, with overrides applied based on `p$fgrp` ('Z' or 'G'):
1. **glcont**: Company master file (input only).
2. **shiptz**: Ship-to master file (input only).
3. **shipthx**: Ship-to history file (input only).
4. **arcup31**: Customer pricing file (input only).
5. **arcuphx**: Customer pricing history file (input only).
6. **cuadrx**: Customer address file (input only).
7. **cuadrhx**: Customer address history file (input only).
8. **arcusz**: Customer master file (input only).
9. **arcushx**: Customer master history file (input only).
10. **arcuspx**: Customer pricing extension file (input only).
11. **arcusphx**: Customer pricing extension history file (input only).
12. **bicuag**: Customer agreement file (input only).
13. **bicuaghx**: Customer agreement history file (input only).
14. **bbprce**: Rack pricing file (input only).
15. **bbprcehx**: Rack pricing history file (input only).
16. **bbndfi**: National Diesel Fuel Index file (input only).
17. **bbndfihx**: National Diesel Fuel Index history file (input only).
18. **bbcfsh**: Carrier fuel surcharge header file (input only).
19. **bbcfshhx**: Carrier fuel surcharge header history file (input only).
20. **bbcfsd**: Carrier fuel surcharge detail file (input only).
21. **bbcfsdhx**: Carrier fuel surcharge detail history file (input only).
22. **bicuf2**: Customer freight file (input only).
23. **bicufrhx**: Customer freight history file (input only).
24. **gsprod**: Product code file (input only).
25. **gsprodhx**: Product code history file (input only).
26. **gshazm**: Hazmat/shipping description file (input only).
27. **gshazmhx**: Hazmat/shipping description history file (input only).
28. **gsumcv**: Product unit of measure conversion file (input only).
29. **gsumcvhx**: Product unit of measure conversion history file (input only).
30. **gsctum**: Container unit of measure file (input only).
31. **gsctumhx**: Container unit of measure history file (input only).
32. **gsctwt**: Container weight file (input only).
33. **gsctwthx**: Container weight history file (input only).
34. **biprt2**: Product tax code file (input only).
35. **biprtxhx**: Product tax code history file (input only).
36. **gscntr1**: Container code file (input only).
37. **gscntrhx**: Container code history file (input only).
38. **bbordh**: Open order header file (input only).
39. **bborhhsx**: Open order header history file (input only).
40. **bbordd**: Open order detail file (input only).
41. **bbordhsx**: Open order detail history file (input only, renamed `bbordhsr`).
42. **bbordm**: Open order miscellaneous file (input only).
43. **bbormhsx**: Open order miscellaneous history file (input only, renamed `bbormhsr`).
44. **gbfmsg**: Message file for error messages (input only).
45. **gb730pd**: Display file with subfile `SFL1` for user interaction (workstation file).

*Note*: The `empmas` and `emphisz` files were removed in revision JB07 (06/19/2024).

---

### External Programs Called

The program calls the following external programs:
1. **QCMDEXC**: Executes file override commands for all files based on `p$fgrp`.
2. **QMHSNDPM**: Sends messages to the program message queue for error handling.
3. **QMHRMVPM**: Removes messages from the program message queue.
4. **GSDTEDIT** (assumed, not explicitly shown): Validates date fields (likely called for effective date validation).

---

### Summary

The `GB730P.rpgle` program, called by programs like `BB910.rpgle` and `BB911.rpgle`, provides a flexible interface for inquiring into historical changes across multiple files in the Bradford Order Entry/Invoices system. It uses a subfile (`SFL1`) to display records from history files (e.g., `bbndfihx`, `bbcfsdhx`, `bbordhsx`) based on the input file name (`p$file`) and key fields (e.g., company, effective date, region). The program supports filtering, pagination, and detailed record viewing, with strict validation and color-coded display for deleted records. Revisions (e.g., JK02, JB07, JB08) enhance field precision and add support for new files, while removing unused ones. The program ensures proprietary data security and integrates with external programs for file overrides and message handling.