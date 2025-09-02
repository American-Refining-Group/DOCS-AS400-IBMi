The RPG program `BI8903.rpgle.txt` is part of the Brandford Order Entry / Invoices system and is designed to copy ship-to records from one customer and ship-to combination to another. It is called by the `BI890P` program (as referenced in the previous context) and operates non-interactively to duplicate records across multiple files. Below, I’ll explain the process steps, business rules, tables used, and external programs called, integrating its role within the context of the OCL (`BI890.ocl36.txt`) and `BI890P` programs.

---

### **Process Steps of the RPG Program**

The `BI8903` program is a straightforward RPGLE program that performs the task of copying records from a source customer/ship-to to a target customer/ship-to across four files: `shipto`, `arcupr`, `cuadr`, and `bbshsa`. Here’s a step-by-step breakdown of the process:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Parameters**: Receives six input parameters:
     - `p$co` (2 digits, company number).
     - `p$fcust` (6 digits, source customer number).
     - `p$fshp` (3 digits, source ship-to number).
     - `p$tcust` (6 digits, target customer number).
     - `p$tshp` (3 digits, target ship-to number).
     - `p$fgrp` (1 character, file group: 'G' or 'Z').
   - **Key Fields**: Populates a composite key (`key11`) and a data structure (`ds`) with:
     - `dsco` (company, from `p$co`).
     - `dstcust` (target customer, from `p$tcust`).
     - `dstshp` (target ship-to, from `p$tshp`).
   - **Key Lists**: Defines two key lists:
     - `klFrmShipto`: For source records (`p$co`, `p$fcust`, `p$fshp`).
     - `KlToShipto`: For target records (`p$co`, `p$tcust`, `p$tshp`).
   - **Test Data**: Includes commented-out test values for parameters, indicating a debugging or development setup (e.g., `p$co='10'`, `p$fcust='006700'`, etc.).

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **File Overrides**: Based on `p$fgrp` ('G' or 'Z'), applies file overrides (`ovg` or `ovz`) for seven files using the `QCMDEXC` system program. The overrides redirect file access to either `g*` (e.g., `gshipto`, `garcupr`) or `z*` (e.g., `zshipto`, `zarcupr`) files.
   - **Open Files**: Opens the following files in user-controlled mode (`usropn`):
     - Input files: `arcuprrd`, `cuadrrd`, `bbshsa1rd` (read-only, renamed record formats).
     - Update file: `shipto` (update/add mode).
     - Output files: `arcupr`, `cuadr`, `bbshsa` (write mode).

3. **Write New Ship-to Records (`WriteShipto` Subroutine)**:
   - **Copy to `shipto`**:
     - Checks if the source record exists in `shipto` using `klFrmShipto` (`chain(n)`). If found (`*in99=*off`), stores the record in `svds` (a 2048-byte save data structure).
     - Checks if the target record does not exist in `shipto` using `KlToShipto` (`chain`). If it doesn’t exist (`*in99=*on`):
       - Clears the `shiptopf` record format.
       - Copies fields from `svds` to `wkds01` (external data structure for `shipto`).
       - Updates key fields: `csship` (target ship-to, `p$tshp`), `cscust` (target customer, `p$tcust`).
       - Clears specific fields (e.g., `csadr1`, `csadr2`, `csadr3`, `csadr4`, `cszip5`, `cszip9`, `csstat`, `csmils`, `csctst`, `cscszp`, `cslong`, `cslatt`) to ensure only relevant data is copied.
       - Writes the new record to `shipto`.
   - **Copy to `arcupr`**:
     - Positions to source records in `arcuprrd` using `klFrmShipto` (`setll`).
     - Reads all matching records (`reade`) until end-of-file (`*in99=*on`).
     - For each record:
       - Stores in `svds` and clears `arcuprpf` format.
       - Copies fields to `wkds02` (external data structure for `arcupr`).
       - Updates key fields: `cpship` (`p$tshp`), `cpcust` (`p$tcust`).
       - Writes the new record to `arcupr`.
   - **Copy to `cuadr`**:
     - Similar to `arcupr`, positions to source records in `cuadrrd` (`setll`), reads (`reade`), and for each record:
       - Stores in `svds`, clears `cuadrpf`, and copies to `wkds03`.
       - Updates key fields: `cdship` (`p$tshp`), `cdcust` (`p$tcust`).
       - Writes to `cuadr`.
   - **Copy to `bbshsa`**:
     - Positions to source records in `bbshsa1rd` (`setll`), reads (`reade`), and for each record:
       - Stores in `svds`, clears `bbshsapf`, and copies to `wkds04`.
       - Updates key fields: `baship` (`p$tshp`), `bacust` (`p$tcust`), `bahkey` (from `key11`).
       - Writes to `bbshsa`.

4. **Program Termination**:
   - Closes all files (`close *all`).
   - Sets `*inlr` to end the program and returns.

---

### **Business Rules**

The program enforces the following business rules:
1. **Record Existence**:
   - The source ship-to record (`p$co`, `p$fcust`, `p$fshp`) must exist in `shipto` for copying to proceed.
   - The target ship-to record (`p$co`, `p$tcust`, `p$tshp`) must not exist in `shipto` to avoid duplication. If it exists, no write occurs for that file.

2. **Data Copying**:
   - Copies all relevant fields from source records to target records, preserving the structure and most data except for specific fields in `shipto` (e.g., addresses, ZIP codes, status) which are cleared to ensure the new record is initialized appropriately.
   - Updates key fields (company, customer, ship-to) to reflect the target values in all files.

3. **File Group Handling**:
   - Supports two file groups ('G' or 'Z'), determined by `p$fgrp`. This allows the program to work with different sets of files (`g*` or `z*`) based on the environment or configuration.

4. **Multi-File Consistency**:
   - Ensures related records in `arcupr`, `cuadr`, and `bbshsa` are copied only if corresponding source records exist, maintaining data integrity across files.

5. **Non-Interactive Operation**:
   - The program does not interact with a display file or user input, relying entirely on parameters passed from `BI890P` to perform the copy operation.

---

### **Tables (Files) Used**

The program uses the following files, as declared in the RPGLE code and overridden in the OCL:
1. **shipto** (Update/Add, `uf a`):
   - Ship-to master file (`gshipto` or `zshipto` based on `p$fgrp`).
   - Used to read source records and write new target records.
2. **arcuprrd** (Input, `if`):
   - Customer pricing/profile file (`garcupr` or `zarcupr`), renamed to `arcuprpr`.
   - Read-only for source records.
3. **arcupr** (Output, `o`):
   - Same file as `arcuprrd` but used for writing new target records.
4. **cuadrrd** (Input, `if`):
   - Customer address file (`gcuadr` or `zcuadr`), renamed to `cuadrpr`.
   - Read-only for source records.
5. **cuadr** (Output, `o`):
   - Same file as `cuadrrd` but used for writing new target records.
6. **bbshsa1rd** (Input, `if`):
   - Shipping data file (`gbbshsa1` or `zbbshsa1`), renamed to `bbshsapr`.
   - Read-only for source records.
7. **bbshsa** (Output, `o`):
   - Same file as `bbshsa1rd` but used for writing new target records.

The OCL (`BI890.ocl36.txt`) declares additional files (e.g., `arcust`, `bicont`, `gsprod`, `gstabl`, `trrtcd`, `shipths`, `arcuphs`), but only the above are used by `BI8903`.

---

### **External Programs Called**

The program calls the following external program:
1. **QCMDEXC**:
   - System program used in the `opntbl` subroutine to execute file override commands (`ovg` or `ovz`) for redirecting file access based on `p$fgrp`.

No other external programs are called directly by `BI8903`. The OCL references `BI9002` and `BB800E`, but they are not invoked by this program.

---

### **Summary**

The `BI8903` program is a non-interactive RPGLE program that copies ship-to records from a source customer/ship-to to a target customer/ship-to across four files (`shipto`, `arcupr`, `cuadr`, `bbshsa`). It:
- Initializes with parameters from `BI890P`, sets up key lists, and applies file overrides.
- Copies records by reading source data, updating keys, and writing to target files, ensuring the target ship-to does not already exist.
- Enforces business rules for record existence and data integrity.
- Integrates with the OCL (`BI890.ocl36.txt`) and `BI890P` by using shared files and parameters.

**Tables Used**:
- `shipto` (`gshipto` or `zshipto`)
- `arcuprrd`/`arcupr` (`garcupr` or `zarcupr`)
- `cuadrrd`/`cuadr` (`gcuadr` or `zcuadr`)
- `bbshsa1rd`/`bbshsa` (`gbbshsa1` or `zbbshsa1`)

**External Programs Called**:
- `QCMDEXC`

If you need further analysis of specific subroutines, business rules, or interactions with `BI890P` or the OCL, let me know!