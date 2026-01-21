The `BB117.rpgle.txt` RPG program, part of the Customer Order Entry system on an IBM System/36 environment, is called by the main OCL program (likely `BB600.ocl36.txt`) or directly by `BB201.rpg36.txt` within the invoice posting workflow. Its primary function is to update or delete records in the order history file (`BBOH`) and its marks supplemental file (`BBOHMS`) to manage historical order data used for duplicate order checking. The program accepts input parameters to identify specific order records and processes them based on a mode indicator (`p$mode`). Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the BB117 RPG Program

The `BB117` program processes input parameters to add, update, or delete records in `BBOH` and `BBOHMS`, ensuring accurate order history for duplicate checking.

1. **File Initialization**:
   - Opens two files (both user-opened, `USROPN`):
     - **BBOH**: Update file (`UF A E`), keyed, disk-based. Stores order history records for duplicate checking.
     - **BBOHMS**: Input file with add capability (`IF A E`), keyed, disk-based. Stores marks supplemental data for order history.
   - Defines file overrides (`ovg`, `ovz`) to map `BBOH` and `BBOHMS` to libraries `gbboh`, `gbbohms` (for `ovg`) or `zbboh`, `zbbohms` (for `ovz`), with `SHARE(*NO)` to prevent file sharing.
   - Defines key lists:
     - `kloh`: Key for `BBOH` using `p$co` (company), `p$rdno` (order number), `p$rseq` (sequence number).
     - `klohms`: Key for `BBOHMS` using `p$co`, `p$rdno`, `p$rseq`, `k$sdad` (ship date), `k$stad` (status date).
     - `klohhdr`: Key for `BBOH` header using `p$co`, `p$rdno`.
   - Defines a data structure `p$elist` for input parameters:
     - `p$co` (1–2): Company number (numeric).
     - `p$rdno` (3–8): Order number (numeric).
     - `p$rseq` (9–11): Sequence number (numeric).
     - `p$cust` (12–17): Customer number (numeric).
     - `p$ship` (18–20): Ship-to code (numeric).
     - `p$pord` (21–35): Purchase order number.
     - `p$ord8` (36–43): Order date (numeric, 8 digits).
     - `p$prod` (44–47): Product code.
     - `p$cntr` (48–51): Container code.
     - `p$qty` (52–56): Quantity (numeric).
     - `p$btch` (57–60): Batch number.
     - `p$dpok` (61): Duplicate OK flag.
     - `p$mode` (62–64): Processing mode (e.g., add, update, delete).
     - `p$fgrp` (65): Freight group.
     - `p$flag` (66): Additional flag.

2. **Parameter Processing**:
   - Receives input parameters via `p$elist` to identify the order record to process.
   - Uses `p$mode` to determine the action (e.g., add, update, delete).
   - Example (commented code): Processes order with `p$rdno = 145669`, `p$cust = 003693`, `p$ship = 001`, `p$prod = '4111'`, `p$pord = 'CAPO-TEST01'`, `p$fgrp = 'Z'`, `p$flag = '0'`.

3. **Record Processing (BBOH)**:
   - Chains to `BBOH` using `kloh` (`p$co`, `p$rdno`, `p$rseq`) to locate the order history record.
   - Based on `p$mode`:
     - **Add**: Writes a new record to `BBOH` with fields from `p$elist` (e.g., `p$cust`, `p$ship`, `p$pord`, `p$ord8`, `p$prod`, `p$cntr`, `p$qty`, `p$btch`, `p$dpok`, `p$fgrp`, `p$flag`).
     - **Update**: Updates the existing record with new values from `p$elist`.
     - **Delete**: Deletes the record if `p$mode` indicates deletion (likely when delete code is `'D'`, per `BB201 JB12`).

4. **Marks Supplemental Processing (BBOHMS)**:
   - Chains to `BBOHMS` using `klohms` (`p$co`, `p$rdno`, `p$rseq`, `k$sdad`, `k$stad`) to locate supplemental records.
   - Performs similar actions (add, update, delete) based on `p$mode`.
   - Adds new supplemental records if needed (e.g., for marks or additional order metadata).

5. **Time Conversion**:
   - Uses a data structure for time conversion:
     - `t#time` (1–14): Full timestamp.
     - `t#hms` (1–6): Hours, minutes, seconds.
     - `t#hrmn` (1–4): Hours and minutes.
   - Likely used to set system timestamps in `BBOH` or `BBOHMS` records.

6. **Cycle Completion**:
   - Processes the input parameters, updates or deletes records in `BBOH` and `BBOHMS`, and closes files (user-opened, so explicitly closed).
   - Terminates after processing.

---

### Business Rules

1. **Order History Management**:
   - Adds, updates, or deletes records in `BBOH` to maintain order history for duplicate checking.
   - Deletes records only when `p$mode` indicates deletion and the delete code is `'D'` (per `BB201 JB12`).

2. **Marks Supplemental Data**:
   - Manages supplemental data in `BBOHMS` for additional order details (e.g., marks, ship dates, status dates).
   - Supports add operations for new supplemental records.

3. **Input Parameters**:
   - Processes records based on `p$elist` parameters, including company (`p$co`), order number (`p$rdno`), sequence (`p$rseq`), and other order details.
   - Uses `p$mode` to determine the action (add, update, delete).

4. **Duplicate Order Checking**:
   - Maintains `BBOH` records to prevent duplicate orders, deleting them when no longer needed (e.g., when order is invoiced).

5. **File Overrides**:
   - Uses overrides (`ovg`, `ovz`) to map `BBOH` and `BBOHMS` to specific libraries (`gbboh`, `gbbohms` or `zbboh`, `zbbohms`) based on processing context.

6. **No Error Handling**:
   - Assumes input parameters are valid and files are accessible.
   - Includes a comment for “Possible Duplicates Accepted” but no explicit error handling logic.

7. **Integration with ARGLMS**:
   - Supports the Customer Order Entry system by maintaining order history for duplicate checking, called by `BB201` during invoice posting.

---

### Tables (Files) Used

1. **BBOH**:
   - **Description**: Order history file.
   - **Attributes**: Update file (`UF A E`), keyed, disk-based, user-opened (`USROPN`).
   - **Fields Used**: `p$co`, `p$rdno`, `p$rseq`, `p$cust`, `p$ship`, `p$pord`, `p$ord8`, `p$prod`, `p$cntr`, `p$qty`, `p$btch`, `p$dpok`, `p$fgrp`, `p$flag`.
   - **Purpose**: Stores order history for duplicate checking.
   - **Usage**: Added, updated, or deleted based on `p$mode`.

2. **BBOHMS**:
   - **Description**: Marks supplemental file for order history.
   - **Attributes**: Input file with add capability (`IF A E`), keyed, disk-based, user-opened (`USROPN`).
   - **Fields Used**: `p$co`, `p$rdno`, `p$rseq`, `k$sdad` (ship date), `k$stad` (status date).
   - **Purpose**: Stores supplemental order history data (e.g., marks, dates).
   - **Usage**: Added or updated based on `p$mode`.

---

### External Programs Called

The `BB117` RPG program does not call any external programs. It is called by `BB201` (per `DC02`, `JB12`) to handle deletions in `BBOH` and updates in `BBOHMS`.

---

### Summary

The `BB117` RPG program, called by `BB201` within the invoice posting workflow, manages order history by:
- Receiving input parameters via `p$elist` (e.g., `p$co`, `p$rdno`, `p$rseq`, `p$cust`, `p$mode`).
- Adding, updating, or deleting records in `BBOH` using key `kloh` based on `p$mode`.
- Managing supplemental data in `BBOHMS` using key `klohms`.
- Supporting duplicate order checking by maintaining or deleting history records (deletes only when code is `'D'`).
- Using file overrides (`ovg`, `ovz`) to map files to specific libraries.
- Terminating after processing.

**Tables Used**: `BBOH` (order history), `BBOHMS` (marks supplemental).
**External Programs Called**: None.

This program ensures accurate order history management in the ARGLMS system, supporting duplicate order checking during invoice posting.