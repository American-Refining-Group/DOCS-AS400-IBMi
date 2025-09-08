The provided document, `EDI811.rpg36.txt`, is an RPG (Report Program Generator) program for IBM midrange systems (e.g., AS/400 or IBM i), named `EDI811`. It is called by the `EDI811.ocl36.txt` program as part of the EDI (Electronic Data Interchange) outbound Bill of Lading (BOL) tracking maintenance process. The program creates EDI 404-compliant records for BOLs, using input from the `EDIBOL` and `EDIBOLTX` files, and writes output to the `EDIOUT` and `EDIRSI` files. Below, I’ll explain the process steps, business rules, tables (files) used, and any external programs called, integrating context from the related `EDI811P` and `EDI811.ocl36` programs.

---

### Process Steps of the EDI811 RPG Program

The `EDI811` RPG program processes BOL data to generate EDI 404 transaction sets, which are standard formats for rail carrier shipment information. It reads validated BOL records from `EDIBOL`, retrieves additional details from `EDIBOLTX`, and writes formatted EDI records to `EDIOUT` and `EDIRSI`. Here’s a detailed breakdown of the process steps, inferred from the file definitions and output specifications (noting that the program logic is truncated, but the structure provides sufficient insight):

1. **File and Data Structure Definitions** (Lines 0003–0170):
   - **Files**:
     - `EDIBOL` (Input, Primary, 12 bytes, 12-byte key): Contains validated BOL records with `KYCO#` (company number, 2 bytes), `KYBOL#` (BOL number, 7 bytes), and `KYSRN#` (serial number, 3 bytes).
     - `EDIBOLTX` (Input, Full Procedural, 1340 bytes, 18-byte key, externally described): Contains detailed BOL transaction data, with multiple record types identified by indicators (41–48) and condition names (C0–C8).
     - `EDIOUT` (Output, 197 bytes): Stores EDI 404-compliant records for transmission.
     - `EDIRSI` (Output, 1970 bytes): Likely stores additional routing or shipment information, possibly for internal tracking or secondary EDI processing.
   - **Data Structures**:
     - `RE` (Array, 2 elements, 40 characters): Likely used for error messages or constants, though not explicitly used in the output specs.
     - `EDIBOLTX` Input Formats (NS 41–48, C0–C8): Defines multiple record types with fields like:
       - `KEY02`, `KEY03` (5 bytes): Record type or key prefix (e.g., "40402", "40403").
       - `BOL#` (7 bytes): BOL number.
       - `ZERO3` (3 bytes): Padding or zeros.
       - EDI-specific fields (e.g., `GS03` for purchase order, `SYSDT8` for record type, `EBX01–EBX08` for business data, `EN9011–EN9063` for addresses, `ELH101–ELH601` for shipment details).
       - `SRN#` (3 bytes): Serial number, aligning with `KYSRN#` in `EDIBOL`.
     - Each record type (C0–C8) corresponds to different EDI segments (e.g., header, address, shipment details).

2. **Input Processing** (Inferred from File Definitions and Context):
   - The program reads `EDIBOL` as the primary file (`IP`), processing each record sequentially.
   - For each `EDIBOL` record (`KYCO#`, `KYBOL#`, `KYSRN#`), it chains to `EDIBOLTX` using a constructed key (likely `KEY02` or `KEY03` + `BOL#` + `ZERO3`) to retrieve detailed transaction data.
   - The `EDIBOLTX` file has multiple record formats (indicators 41–48, condition names C0–C8), each corresponding to different EDI 404 segments:
     - **C0 (Header, NS 41)**: Contains purchase order (`GS03`), sequence number (`GS02`), routing details (`SYSHM`, `GS06–GS08`).
     - **C2 (NS 42)**: Business data (e.g., `EBX01–EBX08`, `EM301–EM302`).
     - **C3 (NS 43)**: Address details (e.g., `EN9011–EN9063`, `CUST`, `SHIP`).
     - **C4 (NS 44)**: Shipment details (e.g., `EN701–EN705`, `EF902–EF903`).
     - **C5 (NS 45)**: Additional BOL details (e.g., `EN1011–EN4033`).
     - **C6 (NS 46)**: Reference numbers (e.g., `ER2011–ER203C`).
     - **C7 (NS 47)**: Line item details (e.g., `EH311`, `EL511–EL572`, `TPPROD`, `BDPROD`).
     - **C8 (NS 48)**: Hazardous material or special instructions (e.g., `ELH101–ELH105`).
   - The program matches `EDIBOL` records with `EDIBOLTX` records based on `BOL#` and `SRN#` to build complete EDI transactions.

3. **Output Processing** (Lines starting at OEDIOUT):
   - The program writes multiple record types to `EDIOUT` using exception output (`EADD`) with labels like `ADD01`, `ADD02`, ..., `ADD24`, `TRAILR`. Each corresponds to an EDI 404 segment:
     - **ADD01–ADD05** (40401–40405): Header records with fields like `KYCO#`, `KYBOL#`, `GS03`, `SYSDT8`, and routing data.
     - **ADD06–ADD10** (40406–40410): Business and address data (e.g., `EBX01`, `EN9011`, `CUST`).
     - **ADD11–ADD14C** (40411–40414): Reference numbers (e.g., `ER2011–ER203C`).
     - **ADD151–ADD176** (40415–40417): Line item and shipment details (e.g., `EH311`, `EL511–EL567`).
     - **ADD17A–ADD24** (40417–40424): Additional shipment and hazardous material data (e.g., `ELH101`, `ELHT01`).
     - **TRAILR** (Line 1481): Writes a trailer record with `#EOT` to mark the end of the EDI transaction set.
   - Records in `EDIRSI` are likely written similarly, though specific output specs are not shown in the provided code. The larger record size (1970 bytes) suggests it stores detailed or aggregated shipment data.
   - Each output record includes:
     - A record type identifier (e.g., `40401`, `40417`).
     - `KYBOL#` (BOL number).
     - `KYSRN#` (serial number).
     - Specific EDI fields from `EDIBOLTX` (e.g., `GS03`, `EL511`).
     - Constants like `000`, `TKR`, or `LH1` for EDI compliance.

4. **Program Termination**:
   - After processing all `EDIBOL` records, the program writes the trailer record (`#EOT`) to `EDIOUT` and likely closes all files.
   - The OCL program (`EDI811.ocl36`) then calls `EDI404SD` to transmit the `EDIOUT` file via FTP.

---

### Business Rules

The `EDI811` RPG program enforces the following business rules:
1. **EDI 404 Compliance**:
   - Generates records conforming to the EDI 404 Rail Carrier Shipment Information standard, including headers, addresses, shipment details, and trailer records.
   - Each record type (e.g., `40401` to `40424`) corresponds to specific EDI segments, with fields like purchase orders (`GS03`), routing (`SYSHM`), and line items (`EL511–EL567`).
2. **Data Validation**:
   - Uses `EDIBOL` records (from `EDI811P`) as the primary input, ensuring only validated BOLs (`KYCO#`, `KYBOL#`) are processed.
   - Matches `EDIBOL` records with `EDIBOLTX` records using `BOL#` and `SRN#` to retrieve detailed transaction data.
3. **Output Formatting**:
   - Writes fixed-format records to `EDIOUT` with specific field positions and lengths (e.g., `KYBOL#` at positions 6–12, `KYSRN#` at 195–197).
   - Includes constants (e.g., `000`, `TKR`, `LH1`) to meet EDI 404 requirements.
   - Writes a trailer record (`#EOT`) to signal the end of the transaction set.
4. **Multi-Record Processing**:
   - Handles multiple `EDIBOLTX` record types (C0–C8) to build comprehensive EDI transactions, covering headers, addresses, shipments, and references.
   - Supports up to 999,000 records in `EDIOUT` (per OCL definition), indicating large-scale processing capability.
5. **Integration with Workflow**:
   - Relies on `EDIBOL` data generated by `EDI811P.rpg36`, which validates company and BOL numbers.
   - Produces `EDIOUT` for FTP transmission via `EDI404SD` (called by OCL), completing the EDI workflow.

---

### Tables (Files) Used

The RPG program uses the following files:
1. **EDIBOL**:
   - Type: Input, Primary (`IP`, 12 bytes, 12-byte key).
   - Purpose: Contains validated BOL records (`KYCO#`, `KYBOL#`, `KYSRN#`) from `EDI811P`.
2. **EDIBOLTX**:
   - Type: Input, Full Procedural (`IF`, 1340 bytes, 18-byte key, externally described).
   - Purpose: Provides detailed BOL transaction data (e.g., purchase orders, routing, shipment details) for EDI 404 formatting.
3. **EDIOUT**:
   - Type: Output (`O`, 197 bytes).
   - Purpose: Stores EDI 404-compliant records for transmission, including headers, addresses, and trailer records.
4. **EDIRSI**:
   - Type: Output (`O`, 1970 bytes).
   - Purpose: Likely stores additional routing or shipment information, possibly for internal tracking or secondary EDI processing.

---

### External Programs Called

The RPG program itself does not explicitly call other programs via RPG operations (e.g., `CALL` or `PARM`). However:
- **Context from OCL**: The `EDI811.ocl36` program, which calls `EDI811`, subsequently calls `EDI404SD` to handle FTP transmission of the `EDIOUT` file.
- **No Direct Calls in RPG**: The `EDI811` RPG program focuses on data processing and file output, relying on the OCL to manage the broader workflow.

---

### Integration with EDI811P and EDI811 OCL Programs

- **EDI811P (OCL and RPG)**:
  - The `EDI811P.rpg36` program validates company numbers (`KYCO`) and BOL numbers against `BICONT` and `EDIBOLTX`, writing validated records to `EDIBOL`.
  - `EDI811` uses `EDIBOL` as its primary input, ensuring only pre-validated BOLs are processed.
- **EDI811 OCL**:
  - Defines the files (`EDIBOL`, `EDIBOLTX`, `EDIOUT`, `EDIRSI`) used by `EDI811`, with shared access for input and a temporary `EDIOUT` for output.
  - Calls `EDI811` to generate EDI 404 records and `EDI404SD` for FTP transmission, completing the workflow.
- **Workflow Continuity**:
  - `EDI811P` prepares the input (`EDIBOL`), `EDI811` formats it into EDI 404 records (`EDIOUT`), and `EDI404SD` transmits the data.
  - Shared files (`EDIBOL`, `EDIBOLTX`) ensure data consistency across programs.

---

### Summary

- **Process Overview**: The `EDI811` RPG program reads validated BOL records from `EDIBOL`, retrieves detailed transaction data from `EDIBOLTX`, and generates EDI 404-compliant records in `EDIOUT` and possibly `EDIRSI`. It writes multiple record types (headers, addresses, shipments, references, trailer) with fixed formats and constants. The OCL program then triggers FTP transmission via `EDI404SD`.
- **Business Rules**: Ensures EDI 404 compliance, validates input via `EDIBOL` and `EDIBOLTX`, formats output with specific field positions, and supports large-scale processing.
- **Tables/Files**: `EDIBOL` (input), `EDIBOLTX` (input), `EDIOUT` (output), `EDIRSI` (output).
- **External Programs**: None called directly in RPG; `EDI404SD` is called by the OCL for FTP.

If you need further analysis (e.g., the truncated RPG logic or details of `EDI404SD`), please provide additional code or context.