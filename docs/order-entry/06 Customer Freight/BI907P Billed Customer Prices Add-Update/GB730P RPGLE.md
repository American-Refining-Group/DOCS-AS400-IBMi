The `GB730P` RPGLE program is part of the Bradford Order Entry/Invoices system and is designed for global file tracking history inquiry. It allows users to view historical data for various files related to customer, ship-to, pricing, and order information. The program is called from the `BI907` RPGLE program and uses a subfile-based interface to display history records. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called, based on the provided source code and context from related programs.

---

### Process Steps of the GB730P RPGLE Program

The `GB730P` program is an interactive workstation program that uses a display file (`gb730pd`) with a subfile (`SFL1`) to present historical data for a specified file and key fields. It supports inquiries across multiple files, with the file to query determined by an input parameter. The process steps are as follows:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Receive Parameters**: The program accepts a parameter data structure (`p$elist`) containing:
     - `p$file`: File name to query (e.g., `SHIPTO`, `ARCUPR`, `CUADR`, `ARCUST`, `ARCUSP`, `BICUAG`, `BBPRCE`, `BBNDFI`, `BBCFSH`, `BBCFSD`, `BICUFR`, `GSPROD`, `GSHAZM`, `GSUMCV`, `GSCTUM`, `GSCTWT`, `BIPRT2`, `GSCNTR1`, `BBORDH`, `BBORDD`, `BBORDM`).
     - `p$fgrp`: File group (`Z` or `G`, 1-character string).
     - `p$co`: Company number (2-digit numeric).
     - Additional fields depending on the file (e.g., `c1cust`, `c1ship`, `s1prod`, `s1cnty` for `ARCUPR`).
   - **Set Header**: Sets the screen header (`C$HDR1`) based on the file being queried using the `HDR` array (e.g., "Customer Product Code Tracking" for `ARCUPR`).
   - **Set Indicators**: Sets `*in75` for specific files (`ARCUPR`, `BICUFR`) to control display behavior.
   - **Date and Time**: Captures the current date and time using the `JBDT##` field from the program status data structure (`PSDS##`) and formats it into the `d#cymd` data structure for potential use in filtering or display.
   - **User Information**: Captures the user ID (`USER##8`) for logging or display purposes.
   - **File Overrides**: Calls `opntbl` to apply file overrides based on `p$fgrp` (`Z` or `G`) and open database files.
   - **Initialize Fields**: Defines work fields (`r$fdat`, `r$tdat`, `h$fdat`, `h$tdat`, `h$user`, `r$user`, `r$prod`, `h$prod`) for date, user, and product filtering.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - Applies file overrides for all supported files using the `QCMDEXC` API, based on the `p$fgrp` parameter (`Z` or `G`). Overrides are defined in the `OVG` (G group) and `OVZ` (Z group) arrays.
   - Opens all files with the `usropn` option for read-only access (`if`):
     - Master files: `glcont`, `shiptz`, `arcup31`, `cuadrx`, `arcusz`, `arcuspx`, `bicuag`, `bbprce`, `bbndfi`, `bbcfsh`, `bbcfsd`, `bicuf2`, `gsprod`, `gshazm`, `gsumcv`, `gsctum`, `gsctwt`, `biprt2`, `gscntr1`, `bbordh`, `bbordd`, `bbordm`, `gbfmsg`.
     - History files: `shipthx`, `arcuphx`, `cuadrhx`, `arcushx`, `arcusphx`, `bicuaghx`, `bbprcehx`, `bbndfihx`, `bbcfshhx`, `bbcfsdhx`, `bicufrhx`, `gsprodhx`, `gshazmhx`, `gsumcvhx`, `gsctumhx`, `gsctwthx`, `biprtxhx`, `gscntrhx`, `bborhhsx`, `bbordhsx`, `bbormhsx`.
     - Note: `empmas` and `emphisz` were removed in revision `jb07` (06/19/2024).

3. **Subfile Processing (`srsfl1` Subroutine, Implied)**:
   - Although the provided code is truncated, the program uses a subfile (`SFL1`) in the display file (`gb730pd`) with relative record number `RRN1`.
   - **Clear Subfile**: Clears the subfile and message subfile using `clrmsg` and `wrtmsg` subroutines (implied from `BI907` structure).
   - **Load Subfile**: Reads history records from the appropriate history file (e.g., `arcuphx` for `ARCUPR`) based on the input parameters (e.g., company, customer, ship-to, product, container type).
   - **Positioning**: Uses key lists to position the file cursor (e.g., `kls1s1` for `ARCUPR` with fields `c1cono`, `c1cust`, `c1ship`, `s1prod`, `s1cnty`).
   - **Display**: Writes the subfile control record (`sflctl1`) using `exfmt` to display records to the user.
   - **User Interaction**: Processes function keys (e.g., F03 to exit, F05 to refresh) and allows navigation through history records.

4. **Message Handling**:
   - **Add Message (`addmsg`, Implied)**: Sends error messages to the program message queue using `QMHSNDPM` (implied from data structures `m@msgf`, `m@data`, etc.).
   - **Write Message Subfile (`wrtmsg`, Implied)**: Displays the message subfile.
   - **Clear Message Subfile (`clrmsg`, Implied)**: Clears messages using `QMHRMVPM`.

5. **Program Termination**:
   - Closes all open files.
   - Sets `*inlr = *on` and returns.

---

### Business Rules

The `GB730P` program enforces the following business rules:

1. **File-Specific Inquiry**:
   - The program displays history records for the file specified in `p$file`, using the appropriate history file (e.g., `arcuphx` for `ARCUPR`).
   - Supports a wide range of files, each with specific key fields (e.g., company, customer, ship-to, product for `ARCUPR`; company, location, product for `BBPRCE`).

2. **File Group Flexibility**:
   - Supports different file sets (`Z` or `G`) via overrides, allowing the program to work with different data environments.

3. **History Tracking**:
   - Displays historical changes (e.g., additions, updates, deletions) for the specified file, including details like change date, time, user, and modified fields.
   - Ensures history files are accessed for inquiry, not master files.

4. **Field Revisions**:
   - **jk02 (10/05/2014)**: Expanded fields `BAPRCE` and `BAOFFP` to 9.4 packed in `bbprce` and `bbprcehx`.
   - **jk03 (06/01/2015)**: Added field `BFTSEQ` to `bicuf2` and `bicufrhx` for sequence tracking.
   - **jk03 (08/24/2015)**: Added support for `gscntr1` and `gscntrhx` for container code tracking.
   - **jk04 (03/19/2017)**: Added support for order files `bbordh`, `bbordd`, `bbordm` and their history files.
   - **jk05 (04/15/2017)**: Renamed field `BDDDEL` to `BDDEL` in `HISTBBORDD` data structure.
   - **jk06 (08/04/2022)**: Added field `BAPOOR` to `bicuag` and `bicuaghx` for purchase order tracking (live 06/2024).
   - **jb07/jb08 (06/19/2024, 07/01/2024)**: Revised `BKFSPC` in `bbcfsd` and `bbcfsdhx` from 3.1 to 5.2 packed; removed `empmas` and `emphisz` support.

5. **Error Handling**:
   - Validates input parameters to ensure the file name (`p$file`) is supported.
   - Displays error messages (e.g., "Cannot Enter both Sys And Program Id") using the message subfile.

6. **Security**:
   - The program is proprietary to WCI International Company, with unauthorized use prohibited.

---

### Tables Used

The program uses the following database files, all opened with the `usropn` option and overridden based on the file group (`Z` or `G`):
1. **Master Files** (read-only, `if`):
   - `glcont`: General ledger control (override: `gglcont`/`zglcont`).
   - `shiptz`: Ship-to master (override: `gshiptz`/`zshiptz`).
   - `arcup31`: Customer/ship-to cross-reference (override: `garcup3`/`zarcup3`).
   - `cuadrx`: Customer ship-to address (override: `gcuadrx`/`zcuadrx`).
   - `arcusz`: A/R customer master (override: `garcusz`/`zarcusz`).
   - `arcuspx`: A/R customer supplemental (override: `garcuspx`/`zarcuspx`).
   - `bicuag`: Customer sales agreement (override: `gbicuag`/`zbicuag`).
   - `bbprce`: Rack pricing (override: `gbbprce`/`zbbprce`).
   - `bbndfi`: National diesel fuel index (override: `gbbndfi`/`zbbndfi`).
   - `bbcfsh`: Carrier fuel surcharge header (override: `gbbcfsh`/`zbbcfsh`).
   - `bbcfsd`: Carrier fuel surcharge detail (override: `gbbcfsd`/`zbbcfsd`).
   - `bicuf2`: Customer freight (override: `gbicuf2`/`zbicuf2`).
   - `gsprod`: Product codes (override: `ggsprod`/`zgsprod`).
   - `gshazm`: Hazmat/shipping descriptions (override: `ggshazm`/`zgshazm`).
   - `gsumcv`: Product unit of measure conversion (override: `ggsumcv`/`zgsumcv`).
   - `gsctum`: Container unit of measure conversion (override: `ggsctum`/`zgsctum`).
   - `gsctwt`: Container weight (override: `ggsctwt`/`zgsctwt`).
   - `biprt2`: Product tax codes (override: `gbiprt2`/`zbiprt2`).
   - `gscntr1`: Container codes (override: `ggscntr1`/`zgscntr1`).
   - `bbordh`: Open order header (override: `gbbordh`/`zbbordh`).
   - `bbordd`: Open order detail (override: `gbbordd`/`zbbordd`).
   - `bbordm`: Open order miscellaneous (override: `gbbordm`/`zbbordm`).
   - `gbfmsg`: Message file (override: `ggbfmsg`/`zgbfmsg`).

2. **History Files** (read-only, `if`):
   - `shipthx`: Ship-to history (override: `gshipthx`/`zshipthx`).
   - `arcuphx`: Customer/ship-to cross-reference history (override: `garcuphx`/`zarcuphx`).
   - `cuadrhx`: Customer ship-to address history (override: `gcuadrhx`/`zcuadrhx`).
   - `arcushx`: A/R customer master history (override: `garcushx`/`zarcushx`).
   - `arcusphx`: A/R customer supplemental history (override: `garcusphx`/`zarcusphx`).
   - `bicuaghx`: Customer sales agreement history (override: `gbicuaghx`/`zbicuaghx`).
   - `bbprcehx`: Rack pricing history (override: `gbbprcehx`/`zbbprcehx`).
   - `bbndfihx`: National diesel fuel index history (override: `gbbndfihx`/`zbbndfihx`).
   - `bbcfshhx`: Carrier fuel surcharge header history (override: `gbbcfshhx`/`zbbcfshhx`).
   - `bbcfsdhx`: Carrier fuel surcharge detail history (override: `gbbcfsdhx`/`zbbcfsdhx`).
   - `bicufrhx`: Customer freight history (override: `gbicufrhx`/`zbicufrhx`).
   - `gsprodhx`: Product code history (override: `ggsprodhx`/`zgsprodhx`).
   - `gshazmhx`: Hazmat/shipping description history (override: `ggshazmhx`/`zgshazmhx`).
   - `gsumcvhx`: Product unit of measure conversion history (override: `ggsumcvhx`/`zgsumcvhx`).
   - `gsctumhx`: Container unit of measure conversion history (override: `ggsctumhx`/`zgsctumhx`).
   - `gsctwthx`: Container weight history (override: `ggsctwthx`/`zgsctwthx`).
   - `biprtxhx`: Product tax code history (override: `gbiprtxhx`/`zbiprtxhx`).
   - `gscntrhx`: Container code history (override: `ggscntrhx`/`zgscntrhx`).
   - `bborhhsx`: Open order header history (override: `gbborhhsx`/`zbborhhsx`).
   - `bbordhsx`: Open order detail history (override: `gbbordhsx`/`zbbordhsx`, renamed to `bbordhsr`).
   - `bbormhsx`: Open order miscellaneous history (override: `gbbormhsx`/`zbbormhsx`, renamed to `bbormhsr`).

---

### External Programs Called

The `GB730P` program calls the following external program:
1. **QCMDEXC**:
   - Purpose: Executes file override commands for database files.
   - Called in: `opntbl` subroutine (implied).
   - Parameters: `dbov##` (override command, 80 characters from `OVG` or `OVZ`), `dbol##` (command length).

---

### Summary

The `GB730P` RPGLE program is a versatile history inquiry tool that allows users to view historical changes for a wide range of files in the Bradford Order Entry/Invoices system. It uses a subfile-based interface to display records from history files (e.g., `arcuphx` for `ARCUPR`) based on input parameters like company, customer, ship-to, and product. The program supports flexible file group handling (`Z` or `G`), incorporates field revisions for accuracy, and enforces security restrictions. It integrates with the `BI907` program for customer/ship-to history inquiries and relies on a comprehensive set of database files to provide detailed tracking information.