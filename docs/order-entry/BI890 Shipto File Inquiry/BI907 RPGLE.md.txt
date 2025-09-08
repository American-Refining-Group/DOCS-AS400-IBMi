The RPG program `BI907.rpgle` is part of the Brandford Order Entry / Invoices system and is designed for Customer and Ship-to File Maintenance/Inquiry, specifically managing product descriptions and freight terms in the `ARCUPR` file. It is called from the CLP program `BI907AC.clp.txt`, which is invoked by the main OCL program `BI890.ocl36.txt`, as referenced in `BI890.rpg36.txt` (revision JB05). The program supports interactive maintenance (add, update, delete) and inquiry of customer/ship-to product records, using a display file with a subfile. Due to the truncation of the source code (70,923 characters omitted), some logic is inferred from the provided declarations, partial code, and context from related programs (`BI890`, `BI890P`, `BI8903`, `GB730P`, `BB800E`, `BI907AC`). Below, I’ll explain the process steps, business rules, tables used, and external programs called.

---

### **Process Steps of the RPG Program**

The `BI907` program is an interactive RPGLE application that uses a display file (`bi907d`) with a subfile (`sfl1`) to manage and inquire about customer/ship-to product records in the `ARCUPR` file. It supports three modes (Add, Update, All) and uses a temporary work file (`BI907W`) for processing. Here’s a step-by-step breakdown of the process based on the provided code and context:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Parameters**: Receives five input parameters via `*entry` (passed from `BI907AC`):
     - `p$cono` (2 characters): Company number.
     - `p$cust` (6 characters): Customer number.
     - `p$ship` (3 characters): Ship-to number.
     - `p$mode` (3 characters): Processing mode (`INQ` for inquiry, `UPD` for update, etc.).
     - `p$fgrp` (1 character): File group (`G` or `Z`) for file overrides.
   - **Output Parameters**: Likely initializes `o$` fields (e.g., `o$mode`, `o$flag`, not shown in truncated code) to return status to the caller.
   - **Subfile Control**:
     - Initializes `rrn1` (relative record number) for `sfl1`.
     - Defines `pagsz1` (page size, not shown but implied, typically 32 as in `BB800E`).
     - Sets `updmode=*on` to enable update mode by default.
   - **Mode Determination**:
     - Chains to `arcupr` using `kls1s1` (company, customer, ship-to) to check for existing records.
     - If a record exists (`*in99=*off`):
       - Sets Update mode: `*in87=*on`, `s1updt='1'`, `s1f10d='F10=All Mode'` (`fky(02)`), `c1mode='Update Mode'` (`com(02)`).
     - If no record exists (`*in99=*on`):
       - If not in inquiry mode (`*in71=*off`), sets All mode: `s1updt='1'`, `*in87=*off`, `s1f10d='F10=Add Mode'` (`fky(03)`), `c1mode='All Mode'` (`com(03)`).
       - If in inquiry mode (`*in71=*on`), sets inquiry settings: `s1updt='1'`, `*in87=*off`, `s1f10d='F10=Update Mode'` (`fky(01)`), `c1mode='All Mode'` (`com(03)`).
   - **Date/Time**: Captures the current date and time (`time` to `t#time`, mapped to `t#cymd` for timestamping records).
   - **Key Lists**: Defines key lists for file access:
     - `klbi907`: `cpcono`, `cpcust`, `cpship`, `cpprod`, `w$cnty` (for `arcupr`).
     - `kl2bi907`: `c1cono`, `c1cust`, `c1ship`, `tpprod`, `w$cnty` (for `arcupr` with product).
     - `kls1r1`: `c1cono`, `c1cust`, `c1ship` (for `arcupr` positioning).
     - `klsfl1`: `c1cono`, `c1cust`, `c1ship`, `s1prod`, `s1cnty` (for subfile).
     - `klsfl2`: `c1cono`, `s1prod` (for product validation).
     - `klcust`: `c1cono`, `c1cust` (for `arcust`).
     - `klship`: `c1cono`, `c1cust`, `c1ship` (for `shipto`).
     - `klctyp`: `k$ctyp='CNTRTY'` (for container type in `gstabl`).
   - **Header Setup**: Sets `c$hdr` to either "Customer & Ship To File Maint" (`hd1(01)`) or "Customer & Ship To File Inquiry" (`hd1(02)`) based on mode.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **File Overrides**: Applies overrides from `ovg` (for `G` group) or `ovz` (for `Z` group) arrays using `QCMDEXC` (implied, not shown in truncated code) for seven files: `gstabl`, `arcupr`, `bicont`, `gsprod`, `arcust`, `shipto`, `arcuphs`. Redirects to `g*` (e.g., `ggstabl`, `garcupr`) or `z*` (e.g., `zgstabl`, `zarcupr`) files based on `p$fgrp`.
   - **Open Files**: Opens files in user-controlled mode (`USROPN`):
     - Input files: `gstabl`, `bicont`, `gsprod`, `arcust`, `shipto` (read-only).
     - Update file: `arcupr` (update/add mode).
     - Output file: `arcuphs` (write mode for history).
     - Update work file: `bi907w` (update/add, temporary file in `QTEMP`).

3. **Subfile Processing (Inferred from `sfl1` and Truncated Code)**:
   - **Subfile Setup**: Uses `bi907d` with subfile `sfl1` to display product records (`s1prod`, `s1cnty`) and freight terms (e.g., `s1frcd`, `s1sefr`, `s1clfr`).
   - **Data Retrieval**: Likely reads `arcupr` using `kls1r1` (company, customer, ship-to) to populate the subfile with product codes, descriptions (from `gsprod`), container types (`w$cnty` from `gstabl`), and freight terms.
   - **Modes**:
     - **Add Mode**: Allows adding new `arcupr` records, requiring at least one product code (`err(01)`).
     - **Update Mode**: Updates existing `arcupr` records, checking for deleted or existing records (`err(02)`, `err(10)`, `err(11)`).
     - **All Mode**: Displays all records for inquiry or selective maintenance.
     - **Inquiry Mode**: Protects fields (`*in71=*on`) for read-only access.
   - **Function Keys** (Inferred):
     - **F10**: Toggles between Update, All, and Add modes (based on `s1f10d` settings).
     - **F03/F12**: Exits or cancels (implied, common in RPGLE).
     - **Enter**: Processes subfile changes or selections.
     - **Page Down**: Loads additional subfile records.
   - **Validation**: Validates inputs against:
     - `arcust` (customer existence via `klcust`).
     - `shipto` (ship-to existence via `klship`).
     - `gsprod` (product codes via `klsfl2`).
     - `gstabl` (container types via `klctyp`, `k$ctyp='CNTRTY'`; freight codes via `k$frty='BBFRCD'`).

4. **Freight Terms Handling (Revisions JB01, JB02)**:
   - **JB01 (Calc Freight)**: Supports `CNY` freight code for calculating freight when shipped from non-Bradford locations (e.g., Anchor). Validates `s1clfr` (calculate freight) as `' '`, `'Y'`, or `'N'` when freight code is `'C'` (`err(08)`).
   - **JB02 (Freight Collect with Service Fee)**: Supports `CYY` freight code for freight collect with a $100 service fee charged by ARG (billed to customer by carrier). Validates `s1sefr` (separate freight) as `'Y'` when freight code is `'A'` (`err(09)`), and as `' '`, `'Y'`, or `'N'` when freight code is `'C'` (`err(07)`).

5. **Record Maintenance**:
   - **Add**: Writes new records to `arcupr` and `bi907w`, with history logged to `arcuphs`.
   - **Update**: Updates existing `arcupr` records, reactivating deleted records if needed (`err(11)`).
   - **Delete**: Marks records as deleted in `arcupr` (`err(10)`).
   - **Copy**: Supports copying records (`err(12)`), possibly integrating with `BI8903` for ship-to copying.
   - Uses `wkds01` (data structure for `arcupr`) to manage record fields.

6. **Message Handling**:
   - Displays errors using the message subfile (`m@msgf='GSMSGF'`) with messages from `err` array (e.g., "Must Enter At Least One Product Code in ADD Mode", "Invalid Freight Code").
   - Clears messages using `QMHRMVPM` and sends messages using `QMHSNDPM` (implied).

7. **Program Termination** (Inferred):
   - Closes all files (`close *all`).
   - Sets `*inlr` to `*on` and returns to `BI907AC`.

---

### **Business Rules**

The program enforces the following business rules, based on the code and context:
1. **Processing Modes**:
   - **Add Mode**: Requires at least one product code (`err(01)`). Prevents adding if the record exists or is marked as deleted (`err(02)`).
   - **Update Mode**: Allows updating existing `arcupr` records, reactivating deleted records (`err(11)`).
   - **All Mode**: Displays all records for inquiry or selective maintenance.
   - **Inquiry Mode**: Protects fields (`*in71=*on`) for read-only access (`p$mode='INQ'`).

2. **Validation**:
   - Validates company (`c1cono`), customer (`c1cust`), and ship-to (`c1ship`) against `arcust` and `shipto`.
   - Validates product codes (`s1prod`) against `gsprod`.
   - Validates container types (`s1cnty`) against `gstabl` (`k$ctyp='CNTRTY'`).
   - Validates freight codes (`s1frcd`) against `gstabl` (`k$frty='BBFRCD'`), with specific rules:
     - Freight code `'C'`: `s1sefr` (separate freight) and `s1clfr` (calculate freight) must be `' '`, `'Y'`, or `'N'` (`err(07)`, `err(08)`).
     - Freight code `'A'`: `s1sefr` must be `'Y'` (`err(09)`).
     - Invalid codes trigger `err(03)`, `err(05)`, or `err(06)`.

3. **Freight Terms (JB01, JB02)**:
   - `CNY`: Calculate freight for non-Bradford locations (e.g., Anchor) (`err(08)`).
   - `CYY`: Freight collect with a $100 service fee, arranged by ARG but billed to the customer (`err(07)`, `err(09)`).

4. **File Group Handling**:
   - Supports `G` or `Z` file groups via `p$fgrp`, redirecting file access to `g*` or `z*` files.

5. **Data Integrity**:
   - Logs changes to `arcuphs` for history tracking.
   - Uses `bi907w` (`QTEMP/BI907W`) for temporary data processing.
   - Prevents adding duplicate records or reactivating deleted records without proper handling (`err(02)`, `err(10)`, `err(11)`).

---

### **Tables (Files) Used**

The program uses the following files, as declared in the RPGLE code and aligned with the OCL (`BI890.ocl36.txt`) and `BI907AC`:
1. **bi907d** (Workstation File, `CF`):
   - Interactive display file with subfile `sfl1` for product and freight terms maintenance/inquiry.
2. **gstabl** (Input, `IF`, `ggstabl` or `zgstabl`):
   - General system table, used for validating container types (`CNTRTY`) and freight codes (`BBFRCD`).
3. **bicont** (Input, `IF`, `gbicont` or `zbicont`):
   - Control file, likely for company-specific settings.
4. **gsprod** (Input, `IF`, `ggsprod` or `zgsprod`):
   - Product master file, used for product code validation and descriptions.
5. **shipto** (Input, `IF`, `gshipto` or `zshipto`):
   - Ship-to master file, used to validate ship-to numbers.
6. **arcust** (Input, `IF`, `garcust` or `zarcust`):
   - Customer master file, used to validate customer numbers.
7. **bi907w** (Update/Add, `UF A`, `QTEMP/BI907W`):
   - Temporary work file, managed by `BI907AC`, used for processing data.
8. **arcupr** (Update/Add, `UF A`, `garcupr` or `zarcupr`):
   - Customer product file, primary file for maintenance (add, update, delete).
9. **arcuphs** (Output, `O`, `garcuphs` or `zarcuphs`):
   - Customer product history file, used to log changes.

The OCL declares additional files (e.g., `cuadr`, `bbshsa1`, `trrtcd`, `shipths`), but only the above are explicitly used in `BI907`.

---

### **External Programs Called**

The program likely calls the following external programs, inferred from the code and context:
1. **QCMDEXC** (Implied):
   - Used in the `opntbl` subroutine to execute file override commands (`ovg` or `ovz`) for redirecting file access based on `p$fgrp`.
2. **QMHSNDPM** (Implied):
   - Sends error messages to the program message queue using `m@msgf='GSMSGF'`.
3. **QMHRMVPM** (Implied):
   - Clears messages from the message subfile.

No other programs are directly called by `BI907`. The CLP `BI907AC` calls `BI907`, and the OCL references `BI9002` and `BB800E`, but these are not invoked by `BI907`.

---

### **Summary**

The `BI907` program is an interactive RPGLE application for Customer and Ship-to File Maintenance/Inquiry, called via `BI907AC` from the OCL (`BI890.ocl36.txt`). It:
- Manages `arcupr` records for product descriptions and freight terms in Add, Update, All, or Inquiry modes.
- Validates inputs against `arcust`, `shipto`, `gsprod`, and `gstabl`, with specific freight term rules (`CNY`, `CYY`).
- Uses a temporary work file (`QTEMP/BI907W`) and logs history to `arcuphs`.
- Supports function keys (F10, F03, F12, Enter, Page Down) for navigation and maintenance.
- Enforces rules for mode selection, validation, freight terms, and data integrity.

**Tables Used**:
- `bi907d` (workstation file with subfile `sfl1`)
- `gstabl` (`ggstabl` or `zgstabl`)
- `bicont` (`gbicont` or `zbicont`)
- `gsprod` (`ggsprod` or `zgsprod`)
- `shipto` (`gshipto` or `zshipto`)
- `arcust` (`garcust` or `zarcust`)
- `bi907w` (`QTEMP/BI907W`)
- `arcupr` (`garcupr` or `zarcupr`)
- `arcuphs` (`garcuphs` or `zarcuphs`)

**External Programs Called**:
- `QCMDEXC` (implied, for file overrides)
- `QMHSNDPM` (implied, for message sending)
- `QMHRMVPM` (implied, for message clearing)

Due to the truncation, some subroutines (e.g., subfile processing, data retrieval) are not fully visible. If you have the complete code or need further analysis of its integration with the system, let me know!