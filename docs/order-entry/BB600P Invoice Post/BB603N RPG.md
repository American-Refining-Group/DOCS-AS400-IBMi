The RPG program `BB603N.rpg.txt` is called by the main OCL program (likely `BB600.ocl36.txt`) within the invoice posting workflow on an IBM System/36 or AS/400 environment. Its primary function is to create and update freight-related records for the Load Management System (LMS) by processing freight transaction records from the `BBTRF` file and writing them to `FRBINF`, `FRBINH`, `FRORST`, and other files. The program includes numerous revisions to handle specific freight scenarios, multi-loads, and tolerance tests, ensuring accurate freight data for invoices, including customer-owned product, third-party freight, and product movement transactions. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the BB603N RPG Program

The `BB603N` program reads freight transaction records from `BBTRF`, processes them based on specific conditions (e.g., freight processor code, transaction type, serial number), and writes to multiple freight-related files (`FRBINF`, `FRBINH`, `FRORST`, `FRCINH2`, `FRBINA`). It performs tolerance tests after each order and supports multi-load scenarios, ensuring accurate freight data for LMS integration.

#### Process Steps:
1. **File Initialization**:
   - The program opens the following files:
     - **Input Files**:
       - `BBTRF`: Primary input file (`IP`), 512 bytes per record, disk-based. Contains freight transaction records.
       - `BBIBCH`: Input file (`IF`), 512 bytes per record, 6-byte alternate index (`AI`), disk-based. Contains invoice batch header data.
       - `BBIBCHX`: Input file (`IF`), 512 bytes per record, disk-based. Contains additional invoice batch header data.
       - `BBTRA1`: Input file (`IF`), 512 bytes per record, disk-based. Contains additional transaction data.
       - `BBTRANU`: Input file (`IF`), 512 bytes per record, disk-based. Contains updated transaction data.
       - `BBTRTX`: Input file (`IF`), 512 bytes per record, disk-based. Contains transaction extension data.
     - **Output Files** (all disk-based, with add `A` or update capability):
       - `FRBINF`: Freight invoice file, 534 bytes per record, update capability (`UA`). Stores freight invoice details.
       - `FRBINH`: Freight invoice header file, 534 bytes per record, add capability (`A`). Stores freight invoice headers.
       - `FRBINA`: Freight invoice analysis file, add capability (`A`). Stores additional freight data (per `MG24`).
       - `FRORST`: Freight order status file, add capability (`A`). Stores order status updates.
       - `FRCINH2`: Freight control invoice header file, add capability (`A`). Stores control header updates.

2. **Record Processing (BBTRF)**:
   - The program reads each record from `BBTRF` sequentially using the RPG cycle.
   - Key fields extracted include:
     - `BODEL`: Delete flag (`'D'` for deleted records).
     - `BOCO`: Company number (3 bytes).
     - `BORDNO`: Order number (6 bytes).
     - `BORSEQ`: Header line sequence (3 bytes).
     - `BOFPCD`: Freight processor code (6 bytes).
     - `BFOTYP`: Order type (1 byte, e.g., `'R'` for returns).
     - `BFSRN`: Serial number (3 bytes).
     - `BFTSEQ`: Transaction sequence (3 bytes, per `JB16`).
     - `BFGGAL`: Total gross gallons (9 bytes, per `JB02`).
     - `BFOTFR`: Override freight amount (9 bytes, per `JB20`).
     - `BFMSFR`: Miscellaneous freight amount (9 bytes, per `JB21`).
     - Additional fields for freight calculations (e.g., `BFTWGT`, `BFTGAL`, `BFTQTY`, `BFLNHA`, `BFSCHA`, etc.).
   - The program processes each record to determine whether to write to `FRBINF`, `FRBINH`, `FRBINA`, `FRORST`, or `FRCINH2` based on conditions like freight processor code, transaction type, and serial number.

3. **Tolerance Tests**:
   - After processing each order (per revision 11/2010), the program performs tolerance tests to validate freight data, ensuring accuracy before writing to output files.
   - Specific details of the tests are not provided in the code snippet but likely involve checking quantities, weights, or freight amounts against predefined thresholds.

4. **Freight Record Creation and Updates**:
   - **FRBINF Processing**:
     - For each `BBTRF` record (per `JB04`), the program writes a record to `FRBINF` unless the record is deleted (`BODEL = 'D'`) or a tax adjustment (per `JB08`).
     - Fields copied include `BOCO`, `BORDNO`, `BFSRN`, `BFTSEQ`, `BFGGAL`, `BFOTFR`, `BFMSFR`, and others (e.g., `BFTWGT`, `BFTGAL`, `BFTQTY`, etc.).
     - For multi-load orders (per `LT09`), the program handles multiple serial numbers (`BFSRN`) and updates `FRBINF` with appropriate freight amounts (`BGTFAM`, `BFTFAM`, `CFTFAM`, `CSTFAM`).
     - Override freight (`BFOTFR`, per `JB20`) and miscellaneous freight (`BFMSFR`, per `JB21`) are updated for both multi-load and non-multi-load orders (per `JB22`).
     - For freight-only invoices with `P N N` (prepaid, third-party, per `JB13`, `JB15`), records are created in `FRBINF`.
     - Serial number handling ensures `BFSRN = '001'` if zero (per `JB11`, `JB14`, `JB18`).
     - Quantities for type `'R'` (returns) have their sign reversed (per `JB12`).
   - **FRBINH Processing**:
     - Writes freight invoice header records, including carrier ID (`BFCARR`, per `LT06`).
     - Fields include `BOCO`, `BORDNO`, `BFSRN`, and others.
   - **FRBINA Processing** (per `MG24`):
     - Writes invoice number (`BOINV#`) and other freight analysis data to `FRBINA`.
   - **FRORST Processing**:
     - Writes order status records (`STADD`, `STUPD`) with fields like `BOCO`, `BORDNO`, `SRN#`, `SVSRNM`, `BOFPCD`, system date (`SYDT`), and test flags (`TEST2`, `TTE3`, `TTE4`).
     - Indicates active status with `'Y'` at position 19.
   - **FRCINH2 Processing**:
     - Writes control header updates (`ALSTUP`) with system date (`SYDT`) and active flag `'Y'` at position 91.

5. **Special Freight Handling**:
   - **Prepaid and Third-Party Freight** (per `JB01`, `JB13`):
     - Initially, records for prepaid third-party freight or collect freight were not created (per `JB01`), but this was revised to include `P N N` freight-only invoices (per `JB13`, `JB15`).
   - **Collect Freight from Non-Bradford Locations** (per `JB23`):
     - Handles special cases for freight collect (`CNY`) shipped from non-Bradford locations (e.g., Anchor), calculating freight appropriately.
   - **Multi-Load Support** (per `LT09`):
     - Manages multiple serial numbers (`BFSRN`) for multi-load orders, ensuring correct freight family amounts (`BFTFAM`, `CSTFAM`).
   - **Field Initialization** (per `JB05`):
     - Initializes new fields in `FRBINF` to ensure consistent data.
   - **Numeric FEGL** (per `JB19`):
     - Ensures the `FEGL` field is numeric for accurate processing.

6. **Cycle Completion**:
   - The RPG cycle processes all records in `BBTRF`, matching with `BBIBCH`, `BBIBCHX`, `BBTRA1`, `BBTRANU`, and `BBTRTX` as needed, until the end of the file.
   - Tolerance tests are performed after each order.
   - The program terminates after writing all records to the appropriate output files, closing all files.

---

### Business Rules

1. **Record Filtering**:
   - Deleted records (`BODEL = 'D'`) and tax adjustments are not written to `FRBINF` (per `JB08`).
   - Records are created for all `BBTRF` records regardless of freight processor code (per `JB04`, revising earlier restriction to `BOFPCD = 'I'`).
   - Prepaid third-party freight (`P N N`) and freight-only invoices are included (per `JB13`, `JB15`).

2. **Serial Number Handling**:
   - Serial numbers (`BFSRN`) are set to `'001'` if zero (per `JB11`, `JB14`, `JB18`) to ensure valid data.
   - Multi-load orders are supported by handling multiple serial numbers (`LT09`).

3. **Freight Amount Updates**:
   - Override freight (`BFOTFR`, per `JB20`) and miscellaneous freight (`BFMSFR`, per `JB21`) are updated in `FRBINF` for both multi-load and non-multi-load orders (per `JB22`).
   - Freight family amounts (`BFTFAM`, `CSTFAM`) are updated for multi-loads (per `LT09`).

4. **Quantity Sign for Returns**:
   - Quantities for transaction type `'R'` (returns) have their sign reversed in `FRBINF` (per `JB12`).

5. **Tolerance Tests**:
   - Performed after each order to validate freight data (e.g., weights, quantities, amounts) against tolerances, ensuring accuracy (per 11/2010 revision).

6. **Special Freight Scenarios**:
   - Freight collect (`CNY`) from non-Bradford locations (e.g., Anchor) is calculated appropriately (per `JB23`).
   - Freight-only invoices (`P N N`) are processed to include freight data (per `JB15`).

7. **Field Enhancements**:
   - Total gross gallons (`BFGGAL`) are copied to `FRBINF` (per `JB02`).
   - Carrier ID (`BFCARR`) is output to `FRBINH` (per `LT06`).
   - Invoice number (`BOINV#`) is written to `FRBINA` for analysis (per `MG24`).
   - New fields are initialized (per `JB05`), and `FEGL` is ensured numeric (per `JB19`).

8. **No Error Handling**:
   - The program assumes input files (`BBTRF`, etc.) exist and contain valid data, and output files can be written to without issues.

9. **LMS Integration**:
   - The program creates and updates freight records for the Load Management System (LMS), ensuring compatibility with freight processing and invoice posting.

---

### Tables (Files) Used

1. **BBTRF**:
   - **Description**: Freight transaction file.
   - **Attributes**: 512 bytes per record, primary input file (`IP`), disk-based.
   - **Fields Used**:
     - `BODEL`: Delete flag.
     - `BOCO`: Company number (3 bytes).
     - `BORDNO`: Order number (6 bytes).
     - `BORSEQ`: Header line sequence (3 bytes).
     - `BOFPCD`: Freight processor code (6 bytes).
     - `BFOTYP`: Order type (1 byte, e.g., `'R'`).
     - `BFSRN`: Serial number (3 bytes).
     - `BFTSEQ`: Transaction sequence (3 bytes, per `JB16`).
     - `BFGGAL`: Total gross gallons (9 bytes, per `JB02`).
     - `BFOTFR`: Override freight amount (9 bytes, per `JB20`).
     - `BFMSFR`: Miscellaneous freight amount (9 bytes, per `JB21`).
     - `BFTWGT`, `BFTGAL`, `BFTQTY`, `BFLNHA`, `BFSCHA`, `BFTFAM`, `BFINSA`, etc.: Freight-related fields.
   - **Purpose**: Contains freight transaction data for invoice posting.
   - **Usage**: Read sequentially to create/update freight records.

2. **BBIBCH**:
   - **Description**: Invoice batch header file.
   - **Attributes**: 512 bytes per record, input file (`IF`), 6-byte alternate index, disk-based.
   - **Purpose**: Provides batch header data for matching with `BBTRF`.
   - **Usage**: Read to retrieve batch-related information.

3. **BBIBCHX**:
   - **Description**: Additional invoice batch header file.
   - **Attributes**: 512 bytes per record, input file (`IF`), disk-based.
   - **Purpose**: Provides supplemental batch header data.
   - **Usage**: Read to support freight processing.

4. **BBTRA1**:
   - **Description**: Additional transaction file.
   - **Attributes**: 512 bytes per record, input file (`IF`), disk-based.
   - **Purpose**: Contains additional transaction data for freight processing.
   - **Usage**: Read to match with `BBTRF` records.

5. **BBTRANU**:
   - **Description**: Updated transaction file.
   - **Attributes**: 512 bytes per record, input file (`IF`), disk-based.
   - **Purpose**: Contains updated transaction data.
   - **Usage**: Read to support freight record updates.

6. **BBTRTX**:
   - **Description**: Transaction extension file.
   - **Attributes**: 512 bytes per record, input file (`IF`), disk-based.
   - **Purpose**: Contains extended transaction data.
   - **Usage**: Read to provide additional freight-related data.

7. **FRBINF**:
   - **Description**: Freight invoice file.
   - **Attributes**: 534 bytes per record, update file (`UA`), disk-based.
   - **Fields Used**:
     - `BOCO`, `BORDNO`, `BFSRN`, `BFTSEQ`, `BFGGAL`, `BFOTFR`, `BFMSFR`, `BFTWGT`, `BFTGAL`, `BFTQTY`, `BFLNHA`, `BFSCHA`, `BFTFAM`, `BFINSA`, `CFSCHG`, `CFRPUM`, `BFCAC2`, `BFCAC3`, `BFINSP`, `CFINSP`, `BFFTCN`, `BFMILE`, etc.
   - **Purpose**: Stores freight invoice details for LMS.
   - **Usage**: Updated with `BINFU`, `RELFU` specifications for freight data.

8. **FRBINH**:
   - **Description**: Freight invoice header file.
   - **Attributes**: 534 bytes per record, output file (`O`), add capability (`A`), disk-based.
   - **Fields Used**:
     - `BOCO`, `BORDNO`, `BFSRN`, `BFCARR` (carrier ID, per `LT06`).
   - **Purpose**: Stores freight invoice header data.
   - **Usage**: Written with header records.

9. **FRBINA**:
   - **Description**: Freight invoice analysis file (per `MG24`).
   - **Attributes**: Output file (`O`), add capability (`A`), disk-based.
   - **Fields Used**:
     - `BOINV#`: Invoice number.
   - **Purpose**: Stores freight analysis data, including invoice numbers.
   - **Usage**: Written with analysis records.

10. **FRORST**:
    - **Description**: Freight order status file.
    - **Attributes**: Output file (`O`), add capability (`A`), disk-based.
    - **Fields Used**:
      - `BOCO`, `BORDNO`, `SRN#`, `SVSRNM`, `BOFPCD`, `SYDT`, `TEST2`, `TTE3`, `TTE4`, active flag (`'Y'`).
    - **Purpose**: Stores order status updates for freight processing.
    - **Usage**: Written with `STADD`, `STUPD` specifications.

11. **FRCINH2**:
    - **Description**: Freight control invoice header file.
    - **Attributes**: Output file (`O`), add capability (`A`), disk-based.
    - **Fields Used**:
      - `SYDT`, active flag (`'Y'`).
    - **Purpose**: Stores control header updates for freight processing.
    - **Usage**: Written with `ALSTUP` specification.

---

### External Programs Called

The `BB603N` RPG program does not explicitly call any external programs. It is a self-contained program that processes input from `BBTRF` and related files and writes to freight-related output files.

---

### Summary

The `BB603N` RPG program, called by `BB600.ocl36.txt`, supports the Load Management System (LMS) by:
- Reading freight transaction records from `BBTRF` and matching with `BBIBCH`, `BBIBCHX`, `BBTRA1`, `BBTRANU`, and `BBTRTX`.
- Writing records to `FRBINF`, `FRBINH`, `FRBINA`, `FRORST`, and `FRCINH2` based on conditions like transaction type (`BFOTYP`), serial number (`BFSRN`), and freight processor code (`BOFPCD`).
- Performing tolerance tests after each order to validate freight data.
- Handling special cases like prepaid third-party freight (`P N N`), freight collect from non-Bradford locations (`CNY`), and multi-load orders.
- Supporting enhancements for gross gallons, override freight, miscellaneous freight, and viscosity barcode data (per revisions `JB01`–`JB23`, `LT06`, `LT09`, `MG24`).
- Ensuring serial number accuracy and numeric fields (e.g., `FEGL`).

**Tables Used**: `BBTRF` (freight transactions), `BBIBCH`, `BBIBCHX`, `BBTRA1`, `BBTRANU`, `BBTRTX` (input files), `FRBINF` (freight invoices), `FRBINH` (freight headers), `FRBINA` (freight analysis), `FRORST` (order status), `FRCINH2` (control headers).
**External Programs Called**: None.

This program ensures accurate freight data for LMS integration, supporting invoice posting with robust handling of various freight scenarios and data validations.