The RPG program `GB730P.rpgle` is part of the Brandford Order Entry / Invoices system and is designed for Global File Tracking History Inquiry. It is an interactive RPGLE program that displays historical data across multiple files, allowing users to inquire about changes to various records such as ship-to, customer, product, and order data. It is called from the OCL program `BI890.ocl36.txt`, likely as part of the broader ship-to master maintenance workflow involving `BI890P`, `BI8903`, and `BI890`. Due to the truncation of the source code (571,894 characters omitted), some logic is inferred based on the provided declarations, field definitions, partial code, and context from the OCL and related programs. Below, I’ll explain the process steps, business rules, tables used, and external programs called.

---

### **Process Steps of the RPG Program**

The `GB730P` program is an interactive application that uses a workstation file (`gb730pd`) with a subfile (`SFL1`) to display historical tracking data for various files. It supports inquiries based on parameters like company, customer, and file type, with headers tailored to the specific file being queried. Here’s a step-by-step breakdown of the process based on the provided code and context:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Parameters**: Receives a parameter data structure (`p$elist`) containing:
     - `p$file` (10 characters): Specifies the file to query (e.g., `SHIPTO`, `ARCUPR`, `CUADR`, `ARCUST`, `ARCUSP`, `BICUAG`, `BBPRCE`, `BBNDFI`, `BBCFSH`, `BBCFSD`, `BICUFR`, `GSPROD`, `GSHAZM`, `GSUMCV`, `GSCTUM`, `GSCTWT`, `BIPRT2`, `GSCNTR1`, `BBORDH`, `BBORDD`, `BBORDM`).
     - `p$fgrp` (1 character): File group (`G` or `Z`) for file overrides.
     - `p$co` (2 digits): Company number.
   - **Additional Parameters**: Defines data structures for specific file types, including:
     - `stparm` (for `SHIPTO`): `stco#` (company), `stcust` (customer), `stship` (ship-to).
     - `acparm` (for `ARCUPR`): `acco#`, `accust`, `acship`, `acprod` (product), `accnty` (container type).
     - `caparm` (for `CUADR`): `caco#`, `cacust`, `caship`, `caedic` (EDI code).
     - `arparm` (for `ARCUST`): `arco#`, `arcus`.
     - `arpparm` (for `ARCUSP`): `apco#`, `apcust`.
     - `bicuparm` (for `BICUAG`): `bico#`, `bicust`, `biloc` (location), `bicntr` (container), `biunms` (unit measure), `biship`, `bipr01`–`bipr10` (products), `bipord` (purchase order), `bistd8` (date), `bisttm` (time), `biseqn` (sequence).
     - `bbprparm` (for `BBPRCE`): `bbcono`, `bbloc`, `bbprod` (partial, truncated).
   - **Header Setup**: Sets the subfile header (`C$HDR1`) based on `p$file` using the `HDR` array (e.g., "Ship-To Information Tracking" for `SHIPTO`, "Customer Product Code Tracking" for `ARCUPR`, etc.).
   - **Indicator Setup**: Sets `*in75` for specific files (`ARCUPR`, `BICUFR`, potentially `BICUAG`) to control display or processing logic.
   - **Field Definitions**: Defines fields for date conversion (`d#cymd`, etc.), message handling (`m@msgf`, `m@data`, etc.), and subfile control (`RRN1`, `PAGRRN`, `CSRLOC`).
   - **Environment Data**: Uses `PSDS##` for job details (`JOB##`, `USER##`, `JBDT##`) and defines fields like `r$fdat`, `r$tdat`, `h$fdat`, `h$tdat`, `h$user`, `r$user` for tracking history dates and users.

2. **Open Database Tables**:
   - **File Overrides**: Applies overrides from `OVG` (for `G` group) or `OVZ` (for `Z` group) arrays using `QCMDEXC` (implied, not shown in truncated code) to redirect files (e.g., `gshiptz` or `zshiptz` for `shiptz`).
   - **Open Files**: Opens all declared files in user-controlled mode (`USROPN`) for read-only access:
     - `glcont`, `shiptz`, `shipthx`, `arcup31`, `arcuphx`, `cuadrx`, `cuadrhx`, `arcusz`, `arcushx`, `arcuspx`, `arcusphx`, `bicuag`, `bicuaghx`, `bbprce`, `bbprcehx`, `bbndfi`, `bbndfihx`, `bbcfsh`, `bbcfshhx`, `bbcfsd`, `bbcfsdhx`, `bicuf2`, `bicufrhx`, `gsprod`, `gsprodhx`, `gshazm`, `gshazmhx`, `gsumcv`, `gsumcvhx`, `gsctum`, `gsctumhx`, `gsctwt`, `gsctwthx`, `biprt2`, `biprtxhx`, `gscntr1`, `gscntrhx`, `bbordh`, `bborhhsx`, `bbordd`, `bbordhsx`, `bbordm`, `bbormhsx`, `gbfmsg`.
     - Note: `empmas` and `emphisz` are commented out (per `jb07`), indicating they are no longer used.

3. **Subfile Processing** (Inferred from `SFL1` and Truncated Code):
   - **Subfile Setup**: Uses `gb730pd` with subfile `SFL1` (relative record number `RRN1`) to display historical data.
   - **File Selection**: Based on `p$file`, queries the corresponding file (e.g., `shipthx` for `SHIPTO`, `arcuphx` for `ARCUPR`, etc.) using appropriate key lists (defined in data structures like `stparm`, `acparm`).
   - **Data Retrieval**: Reads historical records (e.g., `shipthx` for ship-to history, `arcuphx` for customer product history) and populates the subfile with fields like dates (`r$fdat`, `r$tdat`), user (`r$user`), and record-specific data.
   - **Header Display**: Sets `C$HDR1` to a descriptive header from the `HDR` array, reflecting the file being queried.

4. **Message Handling**:
   - Uses message subfile data structures (`m@msgf`, `m@data`, etc.) to display errors or status messages (e.g., "Cannot Enter both Sys And Program Id" from `COM` array).
   - Likely calls `QMHSNDPM` and `QMHRMVPM` (implied, common in RPGLE for message handling) to manage message subfile.

5. **Program Termination** (Inferred):
   - Closes all files (`close *all`, implied).
   - Sets `*inlr` to end the program and returns control to the OCL.

---

### **Business Rules**

The program enforces the following business rules, inferred from the code and context:
1. **File Selection**:
   - The file to query (`p$file`) determines which historical file is accessed (e.g., `shipthx` for `SHIPTO`, `arcuphx` for `ARCUPR`). The program supports 21 file types, each with a specific header.
   - Only one file can be queried at a time, and both system and program IDs cannot be entered simultaneously (error message: "Cannot Enter both Sys And Program Id").

2. **History Tracking**:
   - Displays historical changes for records, including user (`r$user`), from/to dates (`r$fdat`, `r$tdat`), and file-specific fields (e.g., customer, ship-to, product, container).
   - Supports both current (`g*`) and alternate (`z*`) file groups, controlled by `p$fgrp`.

3. **Field Expansions** (Revisions):
   - `jk02`: Expanded `BAPRCE` and `BAOFFP` fields in `BBPRCE` to 9.4 packed for pricing accuracy.
   - `jk03`: Added `BFTSEQ` in `BICUFR` for sequence tracking and included `GSCNTR1` for container codes.
   - `jk05`: Renamed `BDDDEL` to `BDDEL` in `BBORDD` for consistency.
   - `jk06`: Added `BAPOOR` in `BICUAG` for purchase order tracking (live June 2024).
   - `jb07`/`jb08`: Expanded `BKFSPC` in `BBCFSD` from 3.1 to 5.2 packed for fuel surcharge precision and removed `EMPMAS`/`EMPHISZ`.

4. **Validation**:
   - Validates input parameters (e.g., `p$co`, `stcust`, `stship`) against files like `glcont`, `arcusz`, or `shiptz`.
   - Ensures historical data is retrieved only for valid keys (e.g., company, customer, ship-to).

5. **Display Control**:
   - Uses `*in75` for specific files (`ARCUPR`, `BICUFR`, possibly `BICUAG`) to control display or processing logic (e.g., enabling/disabling fields or formats).

---

### **Tables (Files) Used**

The program uses the following files, as declared in the RPGLE code and aligned with the OCL (`BI890.ocl36.txt`):
1. **gb730pd** (Workstation File, `CF`):
   - Interactive display file with subfile `SFL1` for history inquiry.
2. **glcont** (`IF`, `gglcont` or `zglcont`):
   - General ledger control file, likely for company validation.
3. **shiptz** (`IF`, `gshiptz` or `zshiptz`):
   - Ship-to master file (current).
4. **shipthx** (`IF`, `gshipthx` or `zshipthx`):
   - Ship-to history file.
5. **arcup31** (`IF`, `garcup3` or `zarcup3`):
   - Customer product file (current).
6. **arcuphx** (`IF`, `garcuphx` or `zarcuphx`):
   - Customer product history file.
7. **cuadrx** (`IF`, `gcuadrx` or `zcuadrx`):
   - Customer address file (current).
8. **cuadrhx** (`IF`, `gcuadrhx` or `zcuadrhx`):
   - Customer address history file.
9. **arcusz** (`IF`, `garcusz` or `zarcusz`):
   - Customer master file (current).
10. **arcushx** (`IF`, `garcushx` or `zarcushx`):
    - Customer master history file.
11. **arcuspx** (`IF`, `garcuspx` or `zarcuspx`):
    - Customer supplemental file (current).
12. **arcusphx** (`IF`, `garcusphx` or `zarcusphx`):
    - Customer supplemental history file.
13. **bicuag** (`IF`, `gbicuag` or `zbicuag`):
    - Customer sales agreement file (current).
14. **bicuaghx** (`IF`, `gbicuaghx` or `zbicuaghx`):
    - Customer sales agreement history file.
15. **bbprce** (`IF`, `gbbprce` or `zbbprce`):
    - Rack pricing file (current).
16. **bbprcehx** (`IF`, `gbbprcehx` or `zbbprcehx`):
    - Rack pricing history file.
17. **bbndfi** (`IF`, `gbbndfi` or `zbbndfi`):
    - National diesel fuel index file (current).
18. **bbndfihx** (`IF`, `gbbndfihx` or `zbbndfihx`):
    - National diesel fuel index history file.
19. **bbcfsh** (`IF`, `gbbcfsh` or `zbbcfsh`):
    - Carrier fuel surcharge header file (current).
20. **bbcfshhx** (`IF`, `gbbcfshhx` or `zbbcfshhx`):
    - Carrier fuel surcharge header history file.
21. **bbcfsd** (`IF`, `gbbcfsd` or `zbbcfsd`):
    - Carrier fuel surcharge detail file (current).
22. **bbcfsdhx** (`IF`, `gbbcfsdhx` or `zbbcfsdhx`):
    - Carrier fuel surcharge detail history file.
23. **bicuf2** (`IF`, `gbicuf2` or `zbicuf2`):
    - Customer freight file (current).
24. **bicufrhx** (`IF`, `gbicufrhx` or `zbicufrhx`):
    - Customer freight history file.
25. **gsprod** (`IF`, `ggsprod` or `zgsprod`):
    - Product master file (current).
26. **gsprodhx** (`IF`, `ggsprodhx` or `zgsprodhx`):
    - Product master history file.
27. **gshazm** (`IF`, `ggshazm` or `zgshazm`):
    - Hazmat/shipping description file (current).
28. **gshazmhx** (`IF`, `ggshazmhx` or `zgshazmhx`):
    - Hazmat/shipping description history file.
29. **gsumcv** (`IF`, `ggsumcv` or `zgsumcv`):
    - Product unit of measure conversion file (current).
30. **gsumcvhx** (`IF`, `ggsumcvhx` or `zgsumcvhx`):
    - Product unit of measure conversion history file.
31. **gsctum** (`IF`, `ggsctum` or `zgsctum`):
    - Container unit of measure conversion file (current).
32. **gsctumhx** (`IF`, `ggsctumhx` or `zgsctumhx`):
    - Container unit of measure conversion history file.
33. **gsctwt** (`IF`, `ggsctwt` or `zgsctwt`):
    - Container weight file (current).
34. **gsctwthx** (`IF`, `ggsctwthx` or `zgsctwthx`):
    - Container weight history file.
35. **biprt2** (`IF`, `gbiprt2` or `zbiprt2`):
    - Product tax code file (current).
36. **biprtxhx** (`IF`, `gbiprtxhx` or `zbiprtxhx`):
    - Product tax code history file.
37. **gscntr1** (`IF`, `ggscntr1` or `zgscntr1`):
    - Container code file (current).
38. **gscntrhx** (`IF`, `ggscntrhx` or `zgscntrhx`):
    - Container code history file.
39. **bbordh** (`IF`, `gbbordh` or `zbbordh`):
    - Open order header file (current).
40. **bborhhsx** (`IF`, `gbborhhsx` or `zbborhhsx`):
    - Open order header history file.
41. **bbordd** (`IF`, `gbbordd` or `zbbordd`):
    - Open order detail file (current).
42. **bbordhsx** (`IF`, `gbbordhsx` or `zbbordhsx`):
    - Open order detail history file.
43. **bbordm** (`IF`, `gbbordm` or `zbbordm`):
    - Open order miscellaneous file (current).
44. **bbormhsx** (`IF`, `gbbormhsx` or `zbbormhsx`):
    - Open order miscellaneous history file.
45. **gbfmsg** (`IF`, `ggbfmsg` or `zgbfmsg`):
    - Message file for error/status messages.

The OCL declares additional files (e.g., `shipths`, `arcuphs`), but only the above are explicitly used in `GB730P`. The commented-out files `empmas` and `emphisz` (per `jb07`) are no longer used.

---

### **External Programs Called**

The program likely calls the following external programs, inferred from the code structure and common RPGLE practices:
1. **QCMDEXC** (Implied):
   - Used to execute file override commands (`OVG` or `OVZ`) for redirecting file access based on `p$fgrp`.
2. **QMHSNDPM** (Implied):
   - Sends error messages to the program message queue, using `m@msgf` (`GSMSGF`).
3. **QMHRMVPM** (Implied):
   - Clears messages from the message subfile.

No other external programs are explicitly referenced in the provided code, and the OCL does not indicate direct calls to `GB730P` from `BI890P`, `BI8903`, or `BI890`. However, `GB730P` may be invoked indirectly via the OCL or another program in the workflow.

---

### **Summary**

The `GB730P` program is an interactive RPGLE application for Global File Tracking History Inquiry, called via the OCL (`BI890.ocl36.txt`). It:
- Displays historical data for 21 file types (e.g., ship-to, customer, product, order) in a subfile (`SFL1`) with tailored headers.
- Initializes with parameters (`p$file`, `p$fgrp`, `p$co`, etc.) and applies file overrides for `G` or `Z` groups.
- Retrieves and displays historical records using key lists (e.g., `stparm`, `acparm`) and manages messages via a message subfile.
- Enforces rules for file selection, validation, and field expansions (e.g., `BKFSPC`, `BAPOOR`).

**Tables Used**:
- `gb730pd` (workstation file with subfile `SFL1`)
- `glcont` (`gglcont` or `zglcont`)
- `shiptz`, `shipthx`, `arcup31`, `arcuphx`, `cuadrx`, `cuadrhx`, `arcusz`, `arcushx`, `arcuspx`, `arcusphx`, `bicuag`, `bicuaghx`, `bbprce`, `bbprcehx`, `bbndfi`, `bbndfihx`, `bbcfsh`, `bbcfshhx`, `bbcfsd`, `bbcfsdhx`, `bicuf2`, `bicufrhx`, `gsprod`, `gsprodhx`, `gshazm`, `gshazmhx`, `gsumcv`, `gsumcvhx`, `gsctum`, `gsctumhx`, `gsctwt`, `gsctwthx`, `biprt2`, `biprtxhx`, `gscntr1`, `gscntrhx`, `bbordh`, `bborhhsx`, `bbordd`, `bbordhsx`, `bbordm`, `bbormhsx`, `gbfmsg`

**External Programs Called**:
- `QCMDEXC` (implied, for file overrides)
- `QMHSNDPM` (implied, for message sending)
- `QMHRMVPM` (implied, for message clearing)

Due to the truncation, the full processing logic (e.g., subfile loading, user interaction) is not visible. If you have the complete code or need analysis of specific aspects, let me know!