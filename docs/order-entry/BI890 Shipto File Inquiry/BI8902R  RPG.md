The RPG program `BI8902R.rpg.txt` is part of the Brandford Order Entry / Invoices system and is designed for Customer Ship-to Addresses File Inquiry, with limited maintenance capabilities (specifically for adding new records). It is called from the main OCL program `BI890.ocl36.txt`, integrating with the ship-to master maintenance workflow alongside programs like `BI890`, `BI890P`, `BI8903`, `GB730P`, `BB800E`, `BI907AC`, `BI907`, and `BI9078`. The program uses a display file (`BI8902D`) with a subfile (`SCRSUBF`) to display and manage customer ship-to address records from the `CUADR` file. Due to the truncation of the source code (16,648 characters omitted), some logic is inferred from the provided declarations, partial code, and context from related programs. Below, I’ll explain the process steps, business rules, tables used, and external programs called.

---

### **Process Steps of the RPG Program**

The `BI8902R` program is an interactive RPGLE application that allows users to inquire about customer ship-to addresses and add new address records to the `CUADR` file. It uses a display file with a subfile to present data and supports function keys for navigation and record creation. Here’s a step-by-step breakdown of the process based on the provided code and context:

1. **Initialization (`*inzsr` Subroutine, Not Shown)**:
   - **Parameters**: Receives a parameter data structure (`@CUADR`, 20 characters) containing:
     - `@CDCO` (2 characters): Company number.
     - `@CDCUS` (6 characters): Customer number.
     - `@CDSHP` (3 characters): Ship-to number.
     - `@CDVLD` (1 character): Valid flag (not used in provided code).
   - **Subfile Control**:
     - Initializes `RRN` (relative record number) for the subfile (`SCRSUBF`).
     - Uses `WRKDS` (display file information data structure) to track cursor location (`CSRLOC`) and current record number (`CRRN`).
   - **Message Handling**: Initializes `IDS` (message subfile data structure) with `STKCNT` (stack count), `DTALEN` (data length), and `ERRCOD` (error code).
   - **Environment Data**: Uses `SDS` (program status data structure) for job details (`PGMQ##`, `JOB##`, `USER##`, `JOBN##`, `JBDT##`, `TIME##`).
   - **Indicators**: Sets up indicators for subfile control (`*IN31`, `*IN32`, `*IN33`, `*IN34`) and function keys (`*IN03`, `*IN06`, `*IN12`).

2. **Open Database Tables**:
   - **File Overrides**: Applies an override for `CUADRRD` to `ZCUADR` (noted in commented code, `OVRDBF FILE(CUADRRD) TOFILE(*LIBL/ZCUADR) SHARE(*NO)`), suggesting support for the `Z` file group. The override is likely executed via `QCMDEXC` (implied, not shown).
   - **Open Files**:
     - `BI8902D` (Workstation File, `CF`): Display file with subfile `SCRSUBF` for inquiry and add operations.
     - `CUADRRD` (Input, `IF`, renamed `CUADRP1`): Customer address file for reading records to build the subfile.
     - `CUADR` (Input, `IF`, renamed `CUADRPF`, commented update mode): Customer address file for inquiry (originally update/add, but update mode is commented out per `JB` revisions).

3. **Main Processing Loop (`*IN03 DOWEQ *OFF`)**:
   - **Subfile Display**:
     - Writes the subfile (`SCRSUBF`) using `*IN31`.
     - Writes the message subfile control (`MSGCTL`) using `*IN32`.
     - Clears cursor position (`ROW`, `COL`) and indicators (`*IN40-*IN42`, `*IN29`).
     - Clears the message subfile (`CLRMSG` subroutine, not shown).
   - **Display Screen**: Executes the `DISPLY` subroutine (not shown) to display the subfile and control format (`exfmt`, implied), allowing user interaction via function keys:
     - **F3**: Exits the program (`*IN03=*ON`, ends loop).
     - **F6**: Initiates Add mode (`*IN25`, inferred from `IF *IN25` condition).
     - **F12**: Cancels the operation (returns to previous screen, implied).
   - **Load Subfile (`LOAD` Subroutine, Not Shown)**:
     - Reads `CUADRRD` using key list `KL02` (likely `CO`, `CUST`, `SHIP`) to populate the subfile with address records (e.g., `EDIC`, `EDYN`, `NAME`, `ADR1`, `ADR2`, `CITY`, `ST`, `ZIP`, `CTY`).
     - Sets `*IN34` if at the end of the file (`*Bottom`, no more records to page forward).

4. **Add Record Processing (`IF *IN25`)**:
   - **Validate Input**:
     - Checks if a record exists in `CUADR` using `KL02` (company, customer, ship-to).
     - If a record exists (`*IN99=*OFF`), sets error indicators (`*IN29`, `*IN40`), assigns `MSGID='MSG0002'` ("Record already exists"), and sends the message via `SNDMSG` (not shown).
     - If the EDI code is invalid (`EDIC` not found in `GSTABL`, commented out), sets `MSGID='MSG0007'` (implied, not shown).
   - **Field Validation (`EDIT2` Subroutine)**:
     - Checks if the name field (`NAME`) is blank. If so, sets error indicators (`*IN29`, `*IN41`, `*IN55`), assigns `MSGID='MSG0009'` ("Name is blank"), and sends the message.
   - **Write Record**:
     - If no errors (`*IN55=*OFF`):
       - Clears the `CUADRPF` record.
       - Maps input fields to `CUADR` fields: `CO` to `CDCO`, `CUST` to `CDCUST`, `SHIP` to `CDSHIP`, `EDIC` to `CDEDIC`, `EDYN` to `CDEDYN`, `NAME` to `CDNAME`, `ADR1` to `CDADR1`, `ADR2` to `CDADR2`, `CITY` to `CDCITY`, `ST` to `CDST`, `ZIP` to `CDZIP`, `CTY` to `CDCTY`.
       - Writes the new record to `CUADR` (commented update mode suggests write is disabled in current version).
       - Sets `REBLYN=*ON` to trigger subfile rebuild (`REBLDS` subroutine, not shown).
       - Clears input fields (`EDIC`, `EDYN`, `NAME`, `ADR1`, `ADR2`, `CITY`, `ST`, `ZIP`, `CTY`) via `CLRREC` subroutine to prepare for the next add.
       - Sets `*IN55` and `*IN60` to stay in Add mode.
   - **Error Handling**:
     - If errors exist (`*IN40=*ON`), sets `*IN55=*ON` to indicate validation failure.

5. **Subfile Rebuild (`REBLDS` Subroutine, Not Shown)**:
   - If `REBLYN=*ON`, rebuilds the subfile by re-reading `CUADRRD` to reflect the newly added record.

6. **Message Handling (`SNDMSG`, `CLRMSG` Subroutines, Not Shown)**:
   - Sends error messages (e.g., `MSG0002`, `MSG0009`) to the message subfile using `QMHSNDPM` (implied).
   - Clears the message subfile using `QMHRMVPM` (implied).

7. **Program Termination**:
   - Exits the main loop when `*IN03=*ON` (F3 pressed).
   - Closes all files (`close *all`, implied).
   - Sets `*inlr=*ON` and returns to the OCL.

---

### **Business Rules**

The program enforces the following business rules, based on the code and context:
1. **Inquiry Mode**:
   - Primarily designed for inquiry, displaying customer ship-to address records from `CUADRRD` in the subfile.
   - Uses `CUADR` in input mode (`IF`) instead of update/add (`UF A`, commented out per `JB` revisions), suggesting updates are disabled in the current version.

2. **Add Mode**:
   - Allows adding new ship-to address records via F6 (`*IN25`).
   - Prevents adding duplicate records (same company, customer, ship-to) with error `MSG0002` ("Record already exists").
   - Requires a non-blank name field (`NAME`), with error `MSG0009` if blank.
   - Clears input fields after a successful add and remains in Add mode (`*IN55`, `*IN60`).

3. **Validation**:
   - Validates company (`CO`), customer (`CUST`), and ship-to (`SHIP`) against `CUADR`.
   - Optionally validates EDI codes (`EDIC`) against `GSTABL` (commented out, suggesting table validation is disabled).
   - Ensures mandatory fields (e.g., `NAME`) are populated.

4. **Function Keys**:
   - **F3**: Exits the program.
   - **F6**: Enters Add mode to create a new ship-to address record.
   - **F12**: Cancels the current operation (returns to previous screen or clears input).

5. **File Group Handling**:
   - Supports the `Z` file group for `CUADRRD` (override to `ZCUADR`), with potential support for `G` group (e.g., `GCUADR`, not shown but implied from related programs like `GB730P`).

6. **Data Integrity**:
   - Ensures new records are written only if they don’t exist in `CUADR`.
   - Rebuilds the subfile after adding a record to reflect changes.

---

### **Tables (Files) Used**

The program uses the following files, as declared in the RPGLE code and aligned with the OCL (`BI890.ocl36.txt`):
1. **BI8902D** (Workstation File, `CF`):
   - Interactive display file with subfile `SCRSUBF` for displaying and adding ship-to address records.
2. **CUADRRD** (Input, `IF`, renamed `CUADRP1`):
   - Customer address file (likely `ZCUADR` via override) used to read records for the subfile.
3. **CUADR** (Input, `IF`, renamed `CUADRPF`, originally Update/Add):
   - Customer address file used for inquiry and (in commented code) adding new records. Update mode is disabled per `JB` revisions.

The commented-out file `GSTABL` (general system table, 256 bytes, 12 key fields) suggests it was used for validating EDI codes but is no longer active. The OCL declares additional files (e.g., `arcupr`, `gsprod`, `arcust`, `shipto`, `bicon`, `bbshsa1`, `trrtcd`, `shipths`, `arcuphs`), but only the above are explicitly used in `BI8902R`.

---

### **External Programs Called**

The program likely calls the following external programs, inferred from the code and context:
1. **QMHSNDPM** (Implied):
   - Sends error messages (e.g., `MSG0002`, `MSG0009`) to the message subfile using the message file (likely `GSMSGF`, as in related programs).
2. **QMHRMVPM** (Implied):
   - Clears messages from the message subfile in the `CLRMSG` subroutine.
3. **QCMDEXC** (Implied):
   - Executes the file override for `CUADRRD` to `ZCUADR` (noted in commented code).

No other programs are directly called. The OCL references `BI9002`, `BI907AC`, `BI907`, `BI9078`, and `BB800E`, but `BI8902R` operates independently within the workflow.

---

### **Summary**

The `BI8902R` program is an interactive RPGLE application for Customer Ship-to Addresses File Inquiry, called from the OCL (`BI890.ocl36.txt`). It:
- Displays ship-to address records from `CUADRRD` in a subfile and allows adding new records to `CUADR` (add functionality disabled in current version).
- Supports F3 (exit), F6 (create), and F12 (cancel) for navigation and record creation.
- Validates inputs (e.g., non-blank `NAME`, no duplicate records) and rebuilds the subfile after adds.
- Enforces rules for inquiry mode, validation, and data integrity, with support for the `Z` file group.

**Tables Used**:
- `BI8902D` (workstation file with subfile `SCRSUBF`)
- `CUADRRD` (`ZCUADR`, input for subfile)
- `CUADR` (input, originally update/add, for inquiry/add)

**External Programs Called**:
- `QMHSNDPM` (implied, for message sending)
- `QMHRMVPM` (implied, for message clearing)
- `QCMDEXC` (implied, for file override)

Due to the truncation, some subroutines (e.g., `DISPLY`, `LOAD`, `REBLDS`, `SNDMSG`, `CLRMSG`) are not fully visible. If you have the complete code or need further analysis of its integration with the system (e.g., relation to `BI8903` for ship-to copying), let me know!