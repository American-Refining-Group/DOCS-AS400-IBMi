The RPG program `GS9293` is designed to copy product load records within an IBM i (AS/400) environment. It is called from the main program `GS929P` to create a new product load record by duplicating an existing record with new key values specified by the user. The program does not have a user interface (no display file) and focuses solely on database operations. Below, I outline the process steps, business rules, database tables used, and external programs called.

---

### Process Steps of GS9293

The program follows a straightforward flow to copy a product load record from one set of key values to another. The steps are organized around the mainline logic and subroutines:

#### 1. **Initialization (*INZSR Subroutine)**
   - **Purpose**: Sets up initial parameters and variables.
   - **Steps**:
     - Receives a single input parameter, `p$elist`, a 125-byte data structure containing:
       - `p$cono`: Company number (2 bytes, numeric).
       - `p$floc`: From location (3 bytes, character).
       - `p$fprod`: From product code (4 bytes, character).
       - `p$fcntr`: From container code (3 bytes, character).
       - `p$fprty`: From priority (1 byte, numeric).
       - `p$fseq#`: From sequence number (3 bytes, numeric).
       - `p$fracd`: From responsibility area code (5 bytes, character).
       - `p$fmlcd`: From major location code (4 bytes, character).
       - `p$ftype`: From product type (30 bytes, character).
       - `p$tloc`: To location (3 bytes, character).
       - `p$tprod`: To product code (4 bytes, character).
       - `p$tcntr`: To container code (3 bytes, character).
       - `p$tprty`: To priority (1 byte, numeric).
       - `p$tseq#`: To sequence number (3 bytes, numeric).
       - `p$tracd`: To responsibility area code (5 bytes, character).
       - `p$tmlcd`: To major location code (4 bytes, character).
       - `p$ttype`: To product type (30 bytes, character).
       - `p$fgrp`: File group ('Z' or 'G', 1 byte, character).
     - Defines key lists (`klFrmProd` and `klToProd`) for accessing the `prdlod1` file based on the "from" and "to" key fields.
     - Initializes a data structure (`wkds01`) to mirror the `prdlod` file record format and a save data structure (`svds`) to hold the record being copied.
     - Sets up time and date conversion data structures (`time12`, `d#cymd`) and program status data structure (`psds##`) for environment values.

#### 2. **Open Database Tables (opntbl Subroutine)**
   - **Purpose**: Opens the `prdlod1` file with the appropriate file override.
   - **Steps**:
     - Checks if `p$fgrp` is 'G' or 'Z'.
     - Applies the corresponding file override from the `ovg` (for 'G') or `ovz` (for 'Z') array using the `QCMDEXC` system API to override `prdlod1` to `gprdlod1` or `zprdlod1`.
     - Opens the `prdlod1` file (update/add mode, `UF A`).

#### 3. **Write New Product Load Record (WriteProd Subroutine)**
   - **Purpose**: Copies an existing product load record to a new record with the specified "to" key values.
   - **Steps**:
     - Clears the save data structure (`svds`).
     - Chains to `prdlod1` using the "from" key list (`klFrmProd`) to retrieve the source record.
     - If the source record is found (`*in99 = *off`):
       - Saves the source record into `svds` using `wkds01`.
       - Chains to `prdlod1` using the "to" key list (`klToProd`) to check if a record with the target keys already exists.
       - If no target record exists (`*in99 = *on`):
         - Clears the `prdlodpf` record format.
         - Restores the source record from `svds` to `wkds01`.
         - Updates key fields with the "to" values (`p$tloc`, `p$tprod`, `p$tcntr`, `p$tprty`, `p$tseq#`, `p$tracd`, `p$tmlcd`, `p$ttype`, `p$cono`).
         - Writes the new record to `prdlod1`.
     - If the source record is not found or the target record already exists, no action is taken (the program silently skips the write operation).

#### 4. **Program Termination**
   - Closes all files (`prdlod1`) and sets `*inlr` to `*on` to end the program.

---

### Business Rules

The program enforces the following business rules during the copy operation:

1. **Source Record Existence**:
   - The source record, identified by the "from" key fields (`p$cono`, `p$floc`, `p$fprod`, `p$fcntr`, `p$fprty`, `p$fseq#`, `p$fracd`, `p$fmlcd`, `p$ftype`), must exist in `prdlod1`. If it does not, no copy is performed.

2. **Target Record Non-Existence**:
   - The target record, identified by the "to" key fields (`p$cono`, `p$tloc`, `p$tprod`, `p$tcntr`, `p$tprty`, `p$tseq#`, `p$tracd`, `p$tmlcd`, `p$ttype`), must not already exist in `prdlod1`. If it does, the copy operation is skipped to prevent overwriting existing records.

3. **Key Field Consistency**:
   - The company number (`p$cono`) is retained from the source to the target record, ensuring the copy operation stays within the same company context.
   - The "to" key fields replace the corresponding "from" key fields in the new record, while all other fields (e.g., descriptions, scheduling data) are copied verbatim from the source record.

4. **File Group Handling**:
   - The file group (`p$fgrp`) determines whether the program accesses `gprdlod1` ('G') or `zprdlod1` ('Z'), ensuring the correct file is used based on the input parameter.

5. **No User Interaction**:
   - The program operates without a user interface, relying on input parameters provided by the calling program (`GS929P`). No validation errors are displayed, and the program assumes the input parameters have been validated by the caller.

---

### Database Tables Used

The program interacts with the following database file:
1. **prdlod1**: Product load file (update/add mode, `UF A`, user-opened).
   - Overrides to `gprdlod1` or `zprdlod1` based on `p$fgrp` ('G' or 'Z').

---

### External Programs Called

The program calls the following external program:
1. **QCMDEXC**: Executes the file override command for `prdlod1` in the `opntbl` subroutine.

---

### Summary

`GS9293` is a concise RPG program called by `GS929P` to copy a product load record from one set of key values to another in the `prdlod1` file. It retrieves the source record, ensures the target record does not already exist, and writes a new record with updated key fields while preserving other data. The program enforces business rules to prevent overwriting existing records and ensures the correct file is accessed based on the file group. With no user interface, it relies on the calling program for input validation and user interaction, making it a focused backend utility for record duplication.