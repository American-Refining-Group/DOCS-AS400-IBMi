The `BB1015.rpgle.txt` document is an RPGLE (RPG IV) program called from the `BB101.ocl36.txt` OCL program in an IBM System/36 or AS/400 (now IBM i) environment. It is a component of the `BB101` order entry program, designed to retrieve and process remarks (accessorials and marks) from the `SHIPTO` and `BBSHSA1` files and update the `BBOTA1` file with order-specific accessorials/marks. The program includes revisions to enhance functionality, such as adding dispatch marks (JB02) and replacing the `BBSHSP` table with `SHIPTO` (DC01). Below is a detailed explanation of the process steps, business rules, tables (files) used, and external programs called.

---

### Process Steps of the RPGLE Program

The `BB1015` program retrieves remarks from the `SHIPTO` file and accessorials/marks from the `BBSHSA1` file, then builds or updates corresponding records in the `BBOTA1` file for an order. The steps are as follows:

1. **Program Initialization**:
   - The program defines three files:
     - `SHIPTO`: Input-only file (2048 bytes, indexed, key length 11, starting at position 2) for ship-to data, replacing `BBSHSP` (DC01).
     - `BBSHSA1`: Input-only file (512 bytes, indexed, key length 16, starting at position 1) for ship-to accessorials/marks (JB01).
     - `BBOTA1`: Update-capable file (512 bytes, indexed, key length 13, starting at position 1) for order transaction accessorials/marks (JB01).
   - A data structure `MARKS` is defined to handle input and output parameters:
     - `MKSKEY` (positions 2-12, 11 digits): Key combining company, customer, and ship-to.
     - `MKCONO` (positions 2-3, 2 digits): Company number.
     - `MKCUST` (positions 4-9, 6 digits): Customer number.
     - `MKSHIP` (positions 10-12, 3 digits): Ship-to number.
     - `MKOMK1` to `MKOMK4` (60 characters each): Order marks 1-4.
     - `MKIMK1` to `MKIMK2` (60 characters each): Invoice marks 1-2.
     - `MKDSP1` to `MKDSP4` (60 characters each): Dispatch information 1-4 (MKDSP3, MKDSP4 added later).
     - `MKBMK1` to `MKBMK4` (60 characters each): Bill of lading marks 1-4.
     - `MKFRNM` (30 characters): Freight bill name.
     - `MKFRA1` to `MKFRA3` (30 characters each): Freight bill address lines 1-3.
     - `MKORDN` (positions 973-978, 6 digits): Order number.
   - Input specifications for `SHIPTO` (DC01):
     - `CSDEL` (position 1): Delete code.
     - `CSCONO` (positions 2-3): Company number.
     - `CSCUST` (positions 4-9): Customer number.
     - `CSSHIP` (positions 10-12): Ship-to number.
     - `CSOMK1` to `CSOMK4`, `CSIMK1` to `CSIMK2`, `CSDSP1` to `CSDSP4`, `CSBMK1` to `CSBMK4`, `CSFRNM`, `CSFRA1` to `CSFRA3`: Corresponding remarks fields.

2. **Parameter Input**:
   - Receives the `MARKS` data structure via `*ENTRY PLIST`, containing company (`MKCONO`), customer (`MKCUST`), ship-to (`MKSHIP`), and order number (`MKORDN`).

3. **Retrieve Ship-to Remarks**:
   - Chains to the `SHIPTO` file using `MKSKEY` (company/customer/ship-to, line DC01).
   - If a record is found (`N99`):
     - Checks if the record is not deleted (`CSDEL ≠ 'D'`).
     - If not deleted, moves remark fields (`CSOMK1` to `CSOMK4`, `CSIMK1` to `CSIMK2`, `CSDSP1` to `CSDSP4`, `CSBMK1` to `CSBMK4`, `CSFRNM`, `CSFRA1` to `CSFRA3`) to corresponding `MARKS` fields (`MKOMK1` to `MKFRA3`).
   - If no record is found or the record is deleted, skips to the `OUT` tag and proceeds to the `SHSA` subroutine.

4. **SHSA Subroutine (Get Ship-to Accessorials/Marks, JB01)**:
   - **Check for Existing Order Accessorials**:
     - Constructs an 8-character key `COORD` (company/order, `MKCONO` + `MKORDN`).
     - Builds a 13-character key `XXK13` by appending '00000' to `COORD`.
     - Sets the file pointer to `BBOTA1` using `XXK13` (`SETLL`).
     - Reads `BBOTA1` to check for existing accessorials/marks (indicator 74).
     - If a record is found with matching `BACOOR` (company/order, `N74`), skips further processing (`GOTO ENDA`).
   - **Build Accessorials/Marks**:
     - Constructs keys for `BBSHSA1`:
       - `SHKY8`: Company/customer (`MKCONO` + `MKCUST`).
       - `SHKY11`: Company/customer/ship-to (`SHKY8` + `MKSHIP`).
       - `SHKY16`: Full key with '00000' appended.
     - Sets the file pointer to `BBSHSA1` using `SHKY16` (`SETLL`).
     - Reads `BBSHSA1` records in a loop (`BLDAAG` tag, indicator 74).
     - For each non-end-of-file record (`N74`):
       - Verifies the record matches `SHKY11` (company/customer/ship-to, `COCUSH`).
       - Constructs a 13-character key `AKEY` (company/order + `BARCID`).
       - Chains to `BBOTA1` using `AKEY`:
         - If no record exists (`78`), writes a new record (`EXCPT BLDA`).
         - If a record exists (`N78`), releases it (`EXCPT RELA`).
       - Continues reading `BBSHSA1` until no more matching records (`GOTO ENDA`).

5. **Output Operations**:
   - **BLDA (Add Record to BBOTA1, JB01)**:
     - Writes a new record to `BBOTA1` with:
       - `BADEL` (position 1): Delete code (not set, assumed blank).
       - `MKCONO` (positions 2-3): Company number.
       - `MKORDN` (positions 4-9): Order number.
       - Positions 10-19: Blanks.
       - `BARCID` (positions 20-24): Accessorial ID.
       - `MKCONO` (positions 25-26): Company number (repeated).
       - `MKORDN` (positions 27-32): Order number (repeated).
       - `BARCTY` (positions 33-51): Accessorial type.
       - `BARCCD` (positions 52-57): Accessorial code.
       - `BADESC` (positions 58-107): Description.
       - `BAQTY` (position 108-110): Quantity.
       - `BAPICK` (position 111): Pick list flag.
       - `BABOL` (position 112): Bill of lading flag.
       - `BAINV` (position 113): Invoice flag.
       - `BAACCS` (position 114): Accessorial flag.
       - `BATCST` (positions 115-123): Cost.
       - `BADSPH` (position 124, JB02): Dispatch print flag.
   - **RELA (Release Record in BBOTA1, JB01)**:
     - Releases an existing `BBOTA1` record (no fields specified, likely clears locks or updates status).

6. **Program Termination**:
   - Sets the last record indicator (`LR`) to exit the program.
   - Returns the updated `MARKS` data structure to the calling program (`BB101`).

---

### Business Rules

The program enforces the following business rules:

1. **Ship-to Remarks Retrieval**:
   - Remarks are retrieved from `SHIPTO` only if the record exists and is not deleted (`CSDEL ≠ 'D'`).
   - Retrieved fields include order marks, invoice marks, dispatch information, bill of lading marks, and freight bill details, which are copied to the `MARKS` data structure.

2. **Accessorials/Marks Processing**:
   - If the order already has accessorials/marks in `BBOTA1` for the company/order (`BACOOR`), no new records are added to avoid duplication (JB01).
   - Accessorials/marks are retrieved from `BBSHSA1` for the company/customer/ship-to combination and added to `BBOTA1` if no existing records are found.
   - Each `BBSHSA1` record generates a new `BBOTA1` record with accessorial details (e.g., ID, type, code, description, quantity, flags).

3. **Data Integrity**:
   - Only non-deleted `SHIPTO` records are processed.
   - `BBSHSA1` records must match the company/customer/ship-to key (`COCUSH`) to be included.
   - Existing `BBOTA1` records are released rather than overwritten if found (`RELA`).

4. **No Error Messaging**:
   - The program does not define or display error messages, relying on the calling program (`BB101`) to handle validation and errors.

5. **Dispatch Marks**:
   - Includes a dispatch print flag (`BADSPH`) in `BBOTA1` records (JB02).

---

### Tables (Files) Used

The program interacts with the following files:

1. **SHIPTO**: Input-only file for ship-to data (2048 bytes, indexed, key length 11, starting at position 2, replaced `BBSHSP` in DC01).
   - Fields:
     - `CSDEL` (position 1): Delete code.
     - `CSCONO` (positions 2-3): Company number.
     - `CSCUST` (positions 4-9): Customer number.
     - `CSSHIP` (positions 10-12): Ship-to number.
     - `CSOMK1` to `CSOMK4` (60 characters each): Order marks.
     - `CSIMK1` to `CSIMK2` (60 characters each): Invoice marks.
     - `CSDSP1` to `CSDSP4` (60 characters each): Dispatch information.
     - `CSBMK1` to `CSBMK4` (60 characters each): Bill of lading marks.
     - `CSFRNM` (30 characters): Freight bill name.
     - `CSFRA1` to `CSFRA3` (30 characters each): Freight bill address lines.
2. **BBSHSA1**: Input-only file for ship-to accessorials/marks (512 bytes, indexed, key length 16, JB01).
   - Fields include:
     - `COCUSH` (positions 1-11): Company/customer/ship-to key.
     - `BARCID` (5 characters): Accessorial ID.
     - `BARCTY` (19 characters): Accessorial type.
     - `BARCCD` (6 characters): Accessorial code.
     - `BADESC` (50 characters): Description.
     - `BAQTY` (3 digits): Quantity.
     - `BAPICK` (1 character): Pick list flag.
     - `BABOL` (1 character): Bill of lading flag.
     - `BAINV` (1 character): Invoice flag.
     - `BAACCS` (1 character): Accessorial flag.
     - `BATCST` (9 digits): Cost.
     - `BADSPH` (1 character, JB02): Dispatch print flag.
3. **BBOTA1**: Update-capable file for order transaction accessorials/marks (512 bytes, indexed, key length 13, JB01).
   - Fields include:
     - `BADEL` (position 1): Delete code.
     - `BACOOR` (positions 2-9): Company/order key.
     - `BARCID` (positions 20-24): Accessorial ID.
     - `BARCTY` (positions 33-51): Accessorial type.
     - `BARCCD` (positions 52-57): Accessorial code.
     - `BADESC` (positions 58-107): Description.
     - `BAQTY` (positions 108-110): Quantity.
     - `BAPICK` (position 111): Pick list flag.
     - `BABOL` (position 112): Bill of lading flag.
     - `BAINV` (position 113): Invoice flag.
     - `BAACCS` (position 114): Accessorial flag.
     - `BATCST` (positions 115-123): Cost.
     - `BADSPH` (position 124, JB02): Dispatch print flag.

---

### External Programs Called

The `BB1015` RPGLE program does not call any external programs. All processing is handled internally through file operations and the `SHSA` subroutine.

---

### Summary

The `BB1015` RPGLE program, called from the `BB101.ocl36.txt` OCL program, retrieves remarks from the `SHIPTO` file and accessorials/marks from the `BBSHSA1` file, then builds or updates corresponding records in the `BBOTA1` file for order entry in `BB101`. It processes ship-to remarks (order, invoice, dispatch, and bill of lading) and adds accessorials/marks if none exist for the order. Business rules ensure non-deleted records are processed, duplicates are avoided, and dispatch print flags are included (JB02). The program interacts with `SHIPTO`, `BBSHSA1`, and `BBOTA1` files and does not call external programs, relying on internal logic for all operations.