The RPG program `GB730P` is designed for Global File Tracking History Inquiry within the Brandford Order Entry/Invoices system. It is called from the `BB912` and `BB910` programs (via the F09 key) to display historical data for various files, allowing users to view audit trails of changes made to records in inquiry mode. The program uses a display file (`gb730pd`) with a single subfile (`SFL1`) to present historical data based on the file specified in the input parameters. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called .

### Process Steps of the GB730P RPG Program

1. **Initialization (*INZSR Subroutine)**:
   - **Receive Parameters**: Accepts input parameters via a data structure (`p$elist`):
     - `p$file`: Specifies the file for which history is queried (e.g., 'BBNDFI', 'BBCFSH', 'BBCFSD', etc.).
     - `p$fgrp`: File group ('G' or 'Z') to determine the library for file overrides.
     - `p$co`: Company code.
     - Additional file-specific parameters (e.g., `pbbndco`, `pbbndefdt`, `pbbndeftm`, `pbbndregn` for `BBNDFI`).
   - **Set Defaults**: Initializes subfile control fields (`rrn1`, `rrnsv1`, `pagsz1` set to 10), message handling fields (`dspmsg`, `m@pgmq`, `m@key`), and flags (`frstrd`, `repsfl`, `fmtagn`, `movehdrflg`).
   - **Configure Header**: Selects a header (`c$hdr1`) from the `HDR` array based on the input file (`p$file`). For example:
     - 'BBNDFI' → "National Diesel Fuel Index Tracking"
     - 'BBCFSH' → "Carrier Fuel Surchrg Header Tracking"
     - 'BBCFSD' → "Carrier Fuel Surchrg Detail Tracking"
   - **Define Key Lists**: Sets up key lists for each supported file (e.g., `bbndfikey`, `bbcfshkey`, `bbcfsdkey`) to access historical records.
   - **Date and User Fields**: Defines work fields (`r$fdat`, `r$tdat`, `h$fdat`, `h$tdat`, `h$user`, `r$user`) for date and user tracking, derived from the job date (`jbdt##`) and user ID (`user##8`).

2. **Open Database Tables (opntbl Subroutine, implied)**:
   - **File Overrides**: Applies file overrides based on the file group (`p$fgrp`) to point to the correct library ('G' or 'Z') for files like `glcont`, `gbfmsg`, `bbndfi`, etc., using `OVG` or `OVZ` arrays. The overrides are executed via an external program (likely `QCMDEXC`, though not explicitly shown in the provided code snippet).
   - **Open Files**: Opens input files (all user-opened, read-only) for the relevant history and master files based on the input file (`p$file`). For example:
     - For `BBNDFI`, opens `bbndfi` and `bbndfihx`.
     - For `BBCFSH`, opens `bbcfsh` and `bbcfshhx`.
     - For `BBCFSD`, opens `bbcfsd` and `bbcfsdhx`.

3. **Process Subfile 1 (srsfl1 Subroutine, implied)**:
   - **Clear Message Subfile**: Clears the message subfile (`clrmsg`) and writes messages (`wrtmsg`) if needed.
   - **Position File**: Positions the history file based on the input parameters (e.g., `pbbndco`, `pbbndefdt`, `pbbndeftm`, `pbbndregn` for `BBNDFI`) using the appropriate key list (e.g., `bbndfikey`).
   - **Main Loop**:
     - Displays the command line and message subfile.
     - Checks for existing subfile records to enable/disable display control (`*IN41`, not explicitly shown but standard for subfile processing).
     - Displays the subfile control record (`sflctl1`) using `EXFMT`.
     - Processes user input:
       - **F03 (Exit)**: Exits the program.
       - **F05 (Refresh)**: Clears positioning fields and reloads the subfile.
       - **Page Down**: Loads additional subfile records (`sf1lod`, implied).
       - **Enter**: Processes user input for repositioning or filtering.
   - **Reposition Subfile**: Validates user input (e.g., date ranges using `GSDTEDIT`) and repositions the file if the company code or other key fields change.

4. **Load Subfile 1 Records (sf1lod Subroutine, implied)**:
   - Loads historical records from the relevant history file (e.g., `bbndfihx`, `bbcfshhx`, `bbcfsdhx`) into the subfile up to the page size (`pagsz1` = 10).
   - Formats each record based on the file type, including fields like change date, time, user, and specific data (e.g., price for `BBNDFI`, fuel surcharge percentage for `BBCFSD`).
   - Updates the relative record number (`rrn1`) and saves it (`rrnsv1`).

5. **Message Handling (clrmsg, wrtmsg, addmsg Subroutines, implied)**:
   - Clears the message subfile (`QMHRMVPM`) and writes messages (`msgctl`) to display errors or statuses.
   - Adds messages to the program message queue (`QMHSNDPM`) if validation fails (e.g., "Cannot Enter both Sys And Program Id").

6. **Program Termination**:
   - Closes all open files and sets `*INLR = *ON` to end the program.

### Business Rules
1. **Inquiry-Only Mode**:
   - The program operates in inquiry mode, displaying historical data without allowing modifications.
   - It supports tracking changes for multiple files, each with specific key fields (e.g., company, customer, order number, effective date).

2. **File-Specific Processing**:
   - The program dynamically adjusts based on the input file (`p$file`), using appropriate key lists and data structures to access the corresponding history file.
   - Headers are set dynamically to reflect the file being queried (e.g., "Carrier Fuel Surchrg Header Tracking" for `BBCFSH`).

3. **Validation**:
   - Validates date inputs using `GSDTEDIT` to ensure correct date formats.
   - Ensures key fields (e.g., company code, customer, product) match records in the corresponding master files (e.g., `glcont`, `bbcfsh`).

4. **Navigation**:
   - **F03**: Exits the program.
   - **F05**: Refreshes the subfile, clearing filters.
   - **Page Down**: Loads additional history records.
   - **Enter**: Repositions the subfile based on user input (e.g., date range, company code).

5. **History Tracking**:
   - Displays audit trails including change date, time, user, and specific field changes (e.g., price, fuel surcharge percentage) from history files (e.g., `bbndfihx`, `bbcfshhx`, `bbcfsdhx`).

### Tables Used
The program supports multiple files, each with a master and history file, all opened as input-only and user-opened:
1. **glcont**: Company code validation (read-only).
2. **gbfmsg**: Message file (read-only).
3. **shiptz**, **shipthx**: Ship-to information and history (read-only).
4. **arcup31**, **arcuphx**: Customer product code and history (read-only).
5. **cuadrx**, **cuadrhx**: Customer ship-to address and history (read-only).
6. **arcusz**, **arcushx**: A/R customer master and history (read-only).
7. **arcuspx**, **arcusphx**: A/R customer supplemental and history (read-only).
8. **bicuag**, **bicuaghx**: Customer sales agreement and history (read-only).
9. **bbprce**, **bbprcehx**: Rack pricing and history (read-only).
10. **bbndfi**, **bbndfihx**: National Diesel Fuel Index and history (read-only).
11. **bbcfsh**, **bbcfshhx**: Carrier Fuel Surcharge Header and history (read-only).
12. **bbcfsd**, **bbcfsdhx**: Carrier Fuel Surcharge Detail and history (read-only).
13. **bicuf2**, **bicufrhx**: Customer freight and history (read-only).
14. **gsprod**, **gsprodhx**: Product code and history (read-only).
15. **gshazm**, **gshazmhx**: Hazmat/shipping description and history (read-only).
16. **gsumcv**, **gsumcvhx**: Product unit of measure conversion and history (read-only).
17. **gsctum**, **gsctumhx**: Container unit of measure conversion and history (read-only).
18. **gsctwt**, **gsctwthx**: Container weight and history (read-only).
19. **biprt2**, **biprtxhx**: Product tax code and history (read-only).
20. **gscntr1**, **gscntrhx**: Container code and history (read-only, added per jk03).
21. **bbordh**, **bborhhsx**: Open order header and history (read-only, added per jk04).
22. **bbordd**, **bbordhsx**: Open order detail and history (read-only, added per jk04, renamed format `bbordhsr`).
23. **bbordm**, **bbormhsx**: Open order miscellaneous and history (read-only, added per jk04, renamed format `bbormhsr`).

*Note*: The `empmas` and `emphisz` files were removed per revision jb07 (06/19/2024).

### External Programs Called
1. **GSDTEDIT**: Validates date inputs, used in the `pldted` parameter list.
2. **QCMDEXC** (implied): Executes file override commands for the `OVG` and `OVZ` arrays.
3. **QMHSNDPM** (implied): Sends messages to the program message queue for error handling.
4. **QMHRMVPM** (implied): Removes messages from the program message queue.

### Additional Notes
- The program is designed to handle a wide range of files, making it a versatile history inquiry tool. It dynamically adjusts its behavior based on the input file (`p$file`), ensuring the correct history file and key fields are used.
- The revisions (e.g., jk01–jk06, jb07–jb08) indicate ongoing enhancements, such as field expansions (e.g., `BKFSPC` from 3.1 to 5.2 in `BBCFSD` per jb07/jb08) and support for new files (e.g., `BBORDH`, `BBORDD`, `BBORDM` per jk04).
- The program integrates with `BB912` and `BB910` by providing historical context for fuel surcharge and diesel fuel index changes, enhancing audit capabilities.

This program ensures users can efficiently review historical changes across multiple data entities with robust validation and user-friendly navigation.