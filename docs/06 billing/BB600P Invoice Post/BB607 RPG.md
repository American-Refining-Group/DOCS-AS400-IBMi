The `BB607.rpg.txt` RPG program is called by the main OCL program (likely `BB600.ocl36.txt`) within the invoice posting workflow on an IBM System/36 environment, following the execution of `BB607.ocl36.txt`. Its primary function is to delete processed records from the shipment container file (`VISSHCN`) and create corresponding history records in the shipment header history file (`VISSHCH`). It uses the order master file (`BBORDR`) to validate records before deletion. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the BB607 RPG Program

The `BB607` program processes records in `VISSHCN`, deletes those that are processed, and writes history records to `VISSHCH`, using `BBORDR` for validation.

1. **File Initialization**:
   - Opens the following files:
     - **Input/Update File**:
       - `VISSHCN`: Update file (`UP`), 249 bytes, disk-based. Primary input file for shipment container records.
       - `BBORDR`: Input file (`IF`), 512 bytes, 11-byte alternate index, 2-byte key, disk-based. Used for order validation.
     - **Output File**:
       - `VISSHCH`: Output file (`O`), 249 bytes, 11-byte alternate index, 2-byte key, add capability (`A`), disk-based. Stores shipment history records.
   - Defines record format for `VISSHCN` (`NS 01`):
     - `VDCO` (1–2): Company number.
     - `VDORDR` (3–8): Order number.
     - `BCOORD` (1–8): Combined company and order number.
     - `VDSRN` (9–11): Sales representative number.
     - `COORDS` (1–11): Combined company, order, and sales representative number.
     - `VDSHDT` (12–19): Ship date.
     - `VDCARR` (20–39): Carrier name.
     - `VDCAR1` (20–29): Carrier code (short form).
     - `VDSQTY` (40–46): Shipment quantity.
     - `VDSHTM` (47–52): Shipment time.
     - `VDPICK` (53–67): Pick ticket number.
     - `VDUPDT` (68–75): Update date.
     - `VDUPTM` (76–81): Update time.
     - `VDPROD` (82–85): Product code.
     - `VDCNTR` (86–88): Container code.
     - `VDCPDS` (89–118): Product description.
     - `VDSEQ` (119–121): Sequence number.
     - `VDFILL` (122–249): Filler or additional data.
     - `RECORD` (1–249): Entire record.

2. **System Date and Time Initialization**:
   - Executes once (`N09 DO`, indicator `09` ensures single execution):
     - Retrieves system time (`TIME` to `TIMDAT`, 12 digits).
     - Converts to system date (`SYSDAT`, 6 digits) and formats as YMD (`SYSYMD = SYSDAT * 10000.01`).
     - Constructs 8-digit date (`SYSD8`) by combining `20` (for century) and `SYSYMD`.
     - Extracts time (`TIME06`, 6 digits) from `TIMDAT`.
     - Sets `SYDT` (14 digits) with `SYSD8` (date) and `TIME06` (time).
     - Sets indicator `09` to prevent re-execution.

3. **Record Processing (VISSHCN)**:
   - Reads `VISSHCN` records sequentially using the RPG cycle (`NS 01`).
   - For each record:
     - Constructs a key (`KEY11`, 11 bytes) by combining `BCOORD` (company + order number) and `VDSEQ` (sequence number).
     - Chains to `BBORDR` using `KEY11` to validate the order (`CHAINBBORDR 99`).
     - If the record is not found in `BBORDR` (`99` on):
       - Deletes the `VISSHCN` record (`EXCPTDELETE`, `EDEL DELETE`).
       - Writes a history record to `VISSHCH` (`EXCPTHIST`, `EADD HIST`).

4. **History Record Creation**:
   - Writes to `VISSHCH` (`EADD HIST`):
     - `RECORD` (1–249): Entire `VISSHCN` record.
     - `SYSD8` (129–136): System date (8 digits).
     - `TIME06` (135–140): System time (6 digits, overlaps with `SYSD8`).

5. **Cycle Completion**:
   - Processes all `VISSHCN` records.
   - Deletes processed records from `VISSHCN` and creates corresponding history records in `VISSHCH`.
   - Terminates after processing, closing all files.

---

### Business Rules

1. **Record Deletion**:
   - Deletes `VISSHCN` records only if they are not found in `BBORDR` (indicator `99` on), indicating they are processed and no longer needed in the active shipment file.

2. **History Record Creation**:
   - Creates a history record in `VISSHCH` for each deleted `VISSHCN` record, copying the entire record and adding system date (`SYSD8`) and time (`TIME06`).

3. **Order Validation**:
   - Validates `VISSHCN` records against `BBORDR` using a composite key (`BCOORD` + `VDSEQ`) to ensure only processed records are deleted.

4. **System Date and Time**:
   - Generates a consistent system date (`SYSD8`, 8 digits, e.g., `20YYMMDD`) and time (`TIME06`, 6 digits, e.g., `HHMMSS`) for history records.

5. **No Error Handling**:
   - Assumes input files (`VISSHCN`, `BBORDR`) exist and are valid, and output file (`VISSHCH`) can be written without issues.

6. **Integration with ARGLMS**:
   - Part of the invoice posting workflow, maintaining shipment history and cleaning up processed shipment container records.

---

### Tables (Files) Used

1. **VISSHCN**:
   - **Description**: Shipment container file.
   - **Attributes**: Update file (`UP`), 249 bytes, disk-based.
   - **Fields Used**:
     - `VDCO` (1–2): Company number.
     - `VDORDR` (3–8): Order number.
     - `BCOORD` (1–8): Combined company and order number.
     - `VDSRN` (9–11): Sales representative number.
     - `COORDS` (1–11): Combined company, order, sales representative number.
     - `VDSHDT` (12–19): Ship date.
     - `VDCARR` (20–39): Carrier name.
     - `VDCAR1` (20–29): Carrier code.
     - `VDSQTY` (40–46): Shipment quantity.
     - `VDSHTM` (47–52): Shipment time.
     - `VDPICK` (53–67): Pick ticket number.
     - `VDUPDT` (68–75): Update date.
     - `VDUPTM` (76–81): Update time.
     - `VDPROD` (82–85): Product code.
     - `VDCNTR` (86–88): Container code.
     - `VDCPDS` (89–118): Product description.
     - `VDSEQ` (119–121): Sequence number.
     - `VDFILL` (122–249): Filler.
     - `RECORD` (1–249): Entire record.
   - **Purpose**: Stores active shipment container data.
   - **Usage**: Read and deleted (`EDEL DELETE`) for processed records.

2. **BBORDR**:
   - **Description**: Order master file.
   - **Attributes**: Input file (`IF`), 512 bytes, 11-byte alternate index, 2-byte key, disk-based.
   - **Fields Used**: Keyed access via `KEY11` (`BCOORD` + `VDSEQ`).
   - **Purpose**: Validates shipment records against order data.
   - **Usage**: Chained for validation (`CHAINBBORDR`).

3. **VISSHCH**:
   - **Description**: Shipment header history file.
   - **Attributes**: Output file (`O`), 249 bytes, 11-byte alternate index, 2-byte key, add capability (`A`), disk-based.
   - **Fields Used**:
     - `RECORD` (1–249): Entire `VISSHCN` record.
     - `SYSD8` (129–136): System date.
     - `TIME06` (135–140): System time.
   - **Purpose**: Stores historical shipment records.
   - **Usage**: Written with history records (`EADD HIST`).

---

### External Programs Called

The `BB607` RPG program does not explicitly call any external programs. It is called by the main OCL (e.g., `BB600.ocl36.txt`) and works in conjunction with `BB607.ocl36.txt`, which prepares the files (`CLRPFM VISSHSM`, `RGZPFM VISSHCN`).

---

### Summary

The `BB607` RPG program, called by the main OCL (e.g., `BB600.ocl36.txt`), processes shipment container records by:
- Initializing system date (`SYSD8`) and time (`TIME06`) for history records.
- Reading `VISSHCN` records and validating against `BBORDR` using `KEY11` (`BCOORD` + `VDSEQ`).
- Deleting processed `VISSHCN` records (`EDEL DELETE`) if not found in `BBORDR` (`99` on).
- Creating history records in `VISSHCH` (`EADD HIST`) with the entire record, system date, and time.
- Terminating after processing all records.

**Tables Used**: `VISSHCN` (shipment container), `BBORDR` (order master), `VISSHCH` (shipment header history).
**External Programs Called**: None.

This program ensures that processed shipment records are removed from active files and archived in the history file, supporting data integrity in the ARGLMS invoice posting workflow.