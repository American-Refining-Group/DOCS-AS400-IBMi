The provided document is an RPGLE program named `GB730P`, called from the OCL procedure `BB943INQ.ocl36.txt` in the IBM System/36 (or AS/400 compatibility mode) environment. This program, titled "Global File Tracking History Inquiry," is designed to display historical tracking data for various files related to customer sales agreements, pricing, freight, and order details. It uses a display file (`gb730pd`) with a subfile (`SFL1`) to present data interactively. Due to the significant truncation of the source code (579,116 characters), critical sections like the main processing logic and subroutines are missing, limiting the ability to fully describe the process steps. However, based on the available code, file definitions, and context from related programs (`BB943`, `BB943V`, `BB9433`, `BB944`, `GSDTCLC1`), I’ll provide a reconstructed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the RPGLE Program

The `GB730P` program is an interactive inquiry module that retrieves and displays historical data from multiple files, focusing on tracking changes to customer sales agreements (`bicuaghx`), pricing, freight, and order-related data. It likely allows users to filter and view records via a subfile in the `gb730pd` display file. The following process steps are inferred from the provided code and context:

1. **Initialization (`*inzsr` Subroutine, Not Shown)**:
   - **Receive Parameters**: The program likely accepts parameters via the `*ENTRY` plist, such as:
     - Company number (`p$cono`, inferred from related programs like `BB944`).
     - Selection criteria (e.g., customer, sequence number, date range).
     - File group (`p$fgrp`, `'G'` or `'Z'` for file overrides).
     - Return flag (`p$flag`, e.g., `'1'` for success, `'0'` for failure).
   - **Set System Information**: Uses the program status data structure (implied, similar to `psds##` in `BB944`) to retrieve:
     - Program name.
     - User ID.
     - Job date (`ps#mdy`, MMDDYY).
     - Job time (`ps#hms`, HHMMSS).
   - **Set Date/Time**: Uses a date conversion data structure (`d#cymd`, etc.) to handle date formatting (e.g., converting CCYYMMDD to MMDDYY for display).
   - **Initialize Display File**: Prepares `gb730pd` and subfile `SFL1` (relative record number `RRN1`) for displaying historical records.
   - **Open Database Tables**: Calls a subroutine (likely `opntbl`, similar to other programs) to apply file overrides and open files.

2. **Open Database Tables (Implied `opntbl` Subroutine)**:
   - **File Overrides**: Applies overrides based on `p$fgrp` (`'G'` or `'Z'`) using `OVG` or `OVZ` arrays (e.g., `OVRDBF FILE(bicuag) TOFILE(*LIBL/gbicuag)` for `'G'`). Executes overrides with `QCMDEXC`.
   - **Open Files**: Opens all files defined in the F-specs with `USROPN`:
     - Master files: `glcont`, `shiptz`, `shipthx`, `arcup31`, `arcuphx`, `cuadrx`, `cuadrhx`, `arcusz`, `arcushx`, `arcuspx`, `arcusphx`, `bicuag`, `bicuaghx`, `bbprce`, `bbprcehx`, `bbndfi`, `bbndfihx`, `bbcfsh`, `bbcfshhx`, `bbcfsd`, `bbcfsdhx`, `bicuf2`, `bicufrhx`, `gsprod`, `gsprodhx`, `gshazm`, `gshazmhx`, `gsumcv`, `gsumcvhx`, `gsctum`, `gsctumhx`, `gsctwt`, `gsctwthx`, `biprt2`, `biprtxhx`, `gscntr1`, `gscntrhx`, `bbordh`, `bborhhsx`, `bbordd`, `bbordhsx`, `bbordm`, `bbormhsx`, `gbfmsg`.
     - Display file: `gb730pd` (workstation file with subfile `SFL1`).
   - **Purpose**: Ensures access to the correct file library (`G` or `Z`) for retrieving historical data.

3. **Process User Input (Main Loop, Truncated)**:
   - **Display Screen**: Writes to `gb730pd`, likely showing a selection panel for users to input criteria (e.g., company, customer, date range) and a subfile (`SFL1`) for historical records.
   - **Handle Input**: Processes user input, such as:
     - Selection criteria to filter records (e.g., customer number, moved to screen per `jk01`).
     - Function keys (e.g., `F12` to exit, `ENTER` to display results, `PAGEDN` to scroll subfile).
   - **Retrieve Data**: Chains or reads from relevant files based on user input, populating `SFL1` with historical records (e.g., from `bicuaghx` for sales agreements, `bbordhsx` for order headers).
   - **Display Headers**: Uses `HDR` array (22 elements, e.g., "Customer Sales Agreement Tracking", "Open Order Header Tracking") to label different file types.

4. **Populate Subfile (`SFL1`, Truncated)**:
   - **Load Records**: Iterates through historical files (e.g., `bicuaghx`, `bbprcehx`, `bbordhsx`) based on selection criteria, writing records to `SFL1` with relative record number `RRN1`.
   - **Field Mapping**: Maps file fields to subfile fields, including:
     - Sales agreement details (`bicuaghx`: `hhcono`, `hhcust`, `hhloc`, `hhcntr`, `hhpr01`–`hhpr10`, `hhprce`, `hhoffp` per `jk02`, `hhpoor` per `jk06`).
     - Order details (`bbordhsx`, `bbordd`, `bbormhsx`, added per `jk04`).
     - Freight details (`bbcfsdhx`, `BKFSPC` revised to 5.2 packed per `jb07`, `jb08`).
   - **Handle Errors**: Displays error messages from `COM` array (e.g., "Cannot Enter both Sys And Program Id") using message subfile (`GSMSGF`).

5. **Message Handling (Implied `clrmsg` Subroutine)**:
   - **Clear Messages**: Likely calls `QMHRMVPM` (as in `BB944`) to clear the message subfile.
   - **Display Messages**: Uses `m@msgf` (`GSMSGF`), `m@rmvk`, and other message fields to show errors or status messages.

6. **Program Termination (Truncated)**:
   - Closes all files (`close *all`).
   - Sets `*inlr = *on` to terminate.
   - Returns `p$flag` to indicate success or failure.

---

### Business Rules

The program enforces the following business rules for tracking history inquiry:

1. **File Selection**:
   - Supports inquiry into multiple historical files, including:
     - Customer sales agreements (`bicuaghx`).
     - Rack pricing (`bbprcehx`).
     - National diesel fuel index (`bbndfihx`).
     - Carrier fuel surcharge header/detail (`bbcfshhx`, `bbcfsdhx`).
     - Customer freight (`bicufrhx`).
     - Product codes (`gsprodhx`), hazmat/shipping (`gshazmhx`), unit of measure conversions (`gsumcvhx`, `gsctumhx`), weights (`gsctwthx`), tax codes (`biprtxhx`), containers (`gscntrhx`).
     - Order headers, details, and miscellaneous (`bborhhsx`, `bbordhsx`, `bbormhsx`, per `jk04`).
   - Users select which file’s history to view (based on `HDR` array).

2. **Customer Input (per `jk01`)**:
   - Customer number is input via the screen rather than passed as a parameter, allowing flexible filtering.

3. **Field Enhancements**:
   - Supports expanded price fields (`BAPRCE`, `BAOFFP`, 9.4 packed, per `jk02`).
   - Includes sequence number (`BFTSEQ`) in freight tracking (`bicuf2`, per `jk03`).
   - Supports container codes in `gscntr1` (per `jk03`).
   - Includes order-related files (`bbordh`, `bbordd`, `bbordm`, per `jk04`) with renamed deletion field (`BDDDEL` to `BDDEL`, per `jk05`).
   - Supports PO/order code (`BAPOOR`) in `bicuag` (per `jk06`).
   - Uses revised freight surcharge field (`BKFSPC`, 5.2 packed, per `jb07`, `jb08`).

4. **Error Handling**:
   - Prevents entering both system and program IDs simultaneously (`COM(1)`: "Cannot Enter both Sys And Program Id").
   - Displays errors in the message subfile using `GSMSGF`.

5. **File Overrides**:
   - Uses `G` or `Z` file groups (`OVG`, `OVZ`) to access different libraries (e.g., `gbicuag`, `zbicuag`).
   - Removed `empmas` and `emphisz` (per `jb07`).

6. **Display and Navigation**:
   - Uses subfile `SFL1` to display historical records.
   - Supports function keys (e.g., `F12` to exit, `ENTER` to refresh, `PAGEDN` to scroll).
   - Displays headers from `HDR` array to indicate the file being viewed.

---

### Tables Used

The program uses the following files, as defined in the File Specification (F-spec) section:

1. **gb730pd**: Display file (workstation file) with subfile `SFL1` for user interaction.
2. **glcont**: Company file, input-only, keyed.
3. **shiptz**, **shipthx**: Ship-to files, input-only, keyed.
4. **arcup31**, **arcuphx**, **cuadrx**, **cuadrhx**, **arcusz**, **arcushx**, **arcuspx**, **arcusphx**: Customer-related files, input-only, keyed.
5. **bicuag**, **bicuaghx**: Sales agreement and history files, input-only, keyed.
6. **bbprce**, **bbprcehx**: Rack pricing files, input-only, keyed.
7. **bbndfi**, **bbndfihx**: National diesel fuel index files, input-only, keyed.
8. **bbcfsh**, **bbcfshhx**, **bbcfsd**, **bbcfsdhx**: Carrier fuel surcharge header/detail files, input-only, keyed.
9. **bicuf2**, **bicufrhx**: Customer freight tracking files, input-only, keyed.
10. **gsprod**, **gsprodhx**: Product code files, input-only, keyed.
11. **gshazm**, **gshazmhx**: Hazmat/shipping description files, input-only, keyed.
12. **gsumcv**, **gsumcvhx**, **gsctum**, **gsctumhx**, **gsctwt**, **gsctwthx**: Unit of measure conversion and weight files, input-only, keyed.
13. **biprt2**, **biprtxhx**: Product tax code files, input-only, keyed.
14. **gscntr1**, **gscntrhx**: Container code files, input-only, keyed.
15. **bbordh**, **bborhhsx**, **bbordd**, **bbordhsx**, **bbordm**, **bbormhsx**: Order header, detail, and miscellaneous files, input-only, keyed (with renamed record formats `bbordhsr`, `bbormhsr`).
16. **gbfmsg**: Message file, input-only, keyed.

**Removed Files** (per `jb07`):
- `empmas`, `emphisz`.

**File Overrides**:
- `OVG` and `OVZ` arrays (46 elements each) map files to `G` or `Z` libraries (e.g., `gglcont`, `zbicuaghx`).

---

### External Programs Called

The program likely calls the following external program, based on patterns in related programs (`BB944`, etc.):

1. **QCMDEXC** (Implied):
   - Used in the `opntbl` subroutine (not shown due to truncation) to execute file override commands.
   - Parameters: Override command string (80 bytes), length (15.5).

2. **QMHRMVPM** (Implied):
   - Likely used in a `clrmsg` subroutine to clear the message subfile, as seen in `BB944`.
   - Parameters: Message file (`m@msgf = 'GSMSGF    *LIBL     '`), stack counter (`m@scnt`), remove key (`m@rmvk`), remove option (`m@rmv`), error code (`m@errc`).

No other external programs are explicitly referenced, though `GSDTCLC1` may be called for date calculations (e.g., converting or validating dates for display).

---

### Summary

- **Process Steps** (Inferred):
  1. Initialize by receiving parameters (e.g., `p$cono`, `p$fgrp`), setting date/time, and preparing `gb730pd`.
  2. Apply file overrides and open files (`opntbl`).
  3. Process user input (selection criteria, function keys) to filter historical records.
  4. Populate subfile `SFL1` with records from historical files (e.g., `bicuaghx`, `bbordhsx`).
  5. Handle errors via message subfile (`GSMSGF`).
  6. Terminate, closing files and returning `p$flag`.

- **Business Rules**:
  - Displays history for multiple files (sales agreements, pricing, freight, orders, etc.).
  - Allows customer number input via screen (per `jk01`).
  - Supports expanded fields (`BAPRCE`, `BAOFFP`, per `jk02`; `BFTSEQ`, per `jk03`; `BAPOOR`, per `jk06`; `BKFSPC`, per `jb07`, `jb08`).
  - Prevents simultaneous system and program ID entry (`COM(1)`).
  - Uses `G` or `Z` file groups for overrides.
  - Displays headers (`HDR`) to indicate file type.

- **Tables Used**:
  - `gb730pd`, `glcont`, `shiptz`, `shipthx`, `arcup31`, `arcuphx`, `cuadrx`, `cuadrhx`, `arcusz`, `arcushx`, `arcuspx`, `arcusphx`, `bicuag`, `bicuaghx`, `bbprce`, `bbprcehx`, `bbndfi`, `bbndfihx`, `bbcfsh`, `bbcfshhx`, `bbcfsd`, `bbcfsdhx`, `bicuf2`, `bicufrhx`, `gsprod`, `gsprodhx`, `gshazm`, `gshazmhx`, `gsumcv`, `gsumcvhx`, `gsctum`, `gsctumhx`, `gsctwt`, `gsctwthx`, `biprt2`, `biprtxhx`, `gscntr1`, `gscntrhx`, `bbordh`, `bborhhsx`, `bbordd`, `bbordhsx`, `bbordm`, `bbormhsx`, `gbfmsg`.

- **External Programs Called**:
  - `QCMDEXC` (implied for file overrides).
  - `QMHRMVPM` (implied for message clearing).

Due to the severe truncation of the source code, critical logic (e.g., main loop, subfile processing) is missing, limiting the detail of the process steps. If you can provide the complete `GB730P.rpgle` code or specific sections (e.g., main loop, subroutines), I can refine the analysis. Additionally, if you have the display file (`gb730pd`) or related programs, I can provide more context. Let me know how you’d like to proceed!