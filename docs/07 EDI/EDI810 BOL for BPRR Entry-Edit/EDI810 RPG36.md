The provided **EDI810.rpg36.txt** is an RPG/400 (or RPG III) program named **EDI810**, designed for **EDI Outbound Bill of Lading Tracking** (likely for EDI 404 transaction sets, as noted in the file comments). It is called by the **EDI810.ocl36** OCL program previously discussed. Below, I’ll explain the **process steps**, **business rules**, **tables (files) used**, and **external programs called**, based on the RPG source code provided.

---

### Process Steps of the EDI810 RPG Program

The RPG program is structured to handle interactive screen processing and file operations for maintaining EDI Bill of Lading (BOL) data. It uses a workstation file (`SCREEN`) for user interaction and multiple database files for data storage and retrieval. The program processes various EDI segments (e.g., BX, N9, N1, N7, R2, L5, LH1, PER) and supports adding and updating BOL records. Here’s a step-by-step breakdown of the process:

1. **Program Initialization**:
   - The **H-spec** (`H P064`) defines program options, likely related to debugging or environment settings (P064 is a program identifier or option code).
   - The program defines several files (see "Tables Used" below) and arrays (`MSG` for error messages, `ARG` for static data like company information).
   - The **F-specs** declare the workstation file (`SCREEN`) and database files, specifying their record lengths, key lengths, and access modes (e.g., `UF` for update, `IF` for input).
   - The **I-specs** define multiple screen formats (`EDI810S1` to `EDI810S9`, `EDI810SA` to `EDI810SG`) and file record formats (`NS 41` to `NS 47`, `NS 51`) for input, mapping fields to specific positions in the records.

2. **Screen Processing (Interactive Input)**:
   - The program uses the `SCREEN` file (a workstation file, likely for a 5250 terminal) with formats `EDI810S1` to `EDI810SG`. Each format corresponds to a specific screen for user input or display:
     - **EDI810S1**: Captures key fields like `EMTY#` (empty number), `KYORD#` (order number), `KYSRN#` (serial number), `KYRULE`, `KYEMTY`, and `KYFRDY` (freight ready date).
     - **EDI810S2**: Handles BX segment data (e.g., `BX01` to `BX08`, `BNX01`, `BNX03`, `BNX04`, `M301`, `M302`) for transaction set details.
     - **EDI810S3**: Handles N9 segment data (e.g., `N9011` to `N9063`) for reference information (up to three reference identifiers).
     - **EDI810S4**: Handles N7 segment data (e.g., `N701` to `D903`) for equipment details.
     - **EDI810S5**: Handles N1 segment data (e.g., `N1011` to `N4033`) for party identification (up to three parties, e.g., shipper, consignee).
     - **EDI810SD**: Similar to `EDI810S5`, but for additional party details (e.g., `N1211` to `N4233`).
     - **EDI810S6**: Handles R2 segment data (e.g., `R2011` to `R203C`) for routing information (up to 12 routing segments, extended by `MG02` modifications).
     - **EDI810S7, EDI810SB, EDI810SC, EDI810SE, EDI810SF, EDI810SG**: Handle L5 segment data (e.g., `H311` to `L065`) for line item descriptions (multiple iterations for different commodities or items).
     - **EDI810S8**: Handles LH1 segment data (e.g., `LH101` to `LH321`) for hazardous material information.
     - **EDI810S9**: Handles PER segment data (e.g., `PER011` to `PER042`) for contact information.
     - **EDI810SA**: Handles additional L5 data (e.g., `L502A` to `L502N`) for extended line item details.
   - The program supports function keys:
     - **KA**: Rekey, returns to screen `EDI810S1`.
     - **KG**: End of Job (EOJ), terminates the program.

3. **File Input and Validation**:
   - The program reads from input files (`BOLEDIY`, `TRRTCD`, `GSHAZM`, `GSPROD`, `SHIPTO`, `SHPADR`, `BBBOLY`) to retrieve BOL, transportation, hazardous material, product, customer, and address data.
   - The `EDIBOLTX` file is used for both input and update (`UF`), storing EDI transaction data with keys like `KEY02`, `KEY03`, `BOL#`, `SRN#`, and `SEQ#`.
   - The **I-specs** (`NS 41` to `NS 47`, `NS 51`) define record formats for `EDIBOLTX`, mapping fields to specific EDI segments (e.g., BX, N9, N1, N7, R2, L5).
   - Validation checks are implied by the `MSG` array, which contains error messages like:
     - "INVALID COMPANY NUMBER ENTERED"
     - "CUSTOMER NOT FOUND"
     - "INVALID BILL OF LADING ENTERED"
     - "ENTRY MUST BE 'Y' OR 'N'"
     - "FREE DAYS CANNOT BE ZERO"
   - These messages suggest the program validates user input (e.g., company number, customer, BOL number) and enforces business rules (see below).

4. **File Output and Updates**:
   - The **O-specs** define output operations for the `SCREEN` file (displaying data and messages) and the `EDIBOLTX` file (adding or updating records).
   - **Screen Output**:
     - Formats `EDI810S1` to `EDI810SG` display fields and messages (`MSG1`, `MSG2`) to the user.
     - Fields like `SAVBOL` (saved BOL number) and `SAVSRN` (saved serial number) are used to maintain context across screens.
   - **EDIBOLTX Output**:
     - **ADD Operations** (`ADD02` to `ADD09`, `ADD0A`): Write new records to `EDIBOLTX` for different EDI segments (e.g., `40402` for BX, `40403` for N9, `40404` for N7, etc.).
     - **UPDATE Operations** (`UPD02` to `UPD09`, `UPD0A`): Update existing records in `EDIBOLTX` with modified data.
     - Each output record includes a record type (e.g., `40402`, `40403`), `SAVBOL`, `SAVSRN`, and segment-specific fields.
   - Additional fields like `TPPROD` (top product), `BDPROD` (bulk product), and `BDNGAL` (bulk gallons) are written for specific L5 segments (`ADD07`).

5. **Program Termination**:
   - The program terminates when the user presses the **KG** function key (EOJ) or when an error condition (e.g., end of file, invalid input) triggers an exit.
   - The `END` tag in the OCL script (from the previous query) is reached after the RPG program completes, resetting switches and clearing local variables.

---

### Business Rules

The RPG program enforces several business rules, inferred from the error messages, screen formats, and file operations:

1. **Company Number Validation**:
   - The program checks for a valid company number. If invalid, it displays "INVALID COMPANY NUMBER ENTERED" (MSG1).
   - Likely tied to fields in `SHIPTO` or `SHPADR` files, ensuring the entered company exists.

2. **Customer Validation**:
   - The program verifies that the customer exists in the `SHIPTO` file. If not found, it displays "CUSTOMER NOT FOUND" (MSG2).
   - The `DC01` comment indicates that the `BBSHSP` table was replaced by `SHIPTO`, suggesting customer data is critical.

3. **Bill of Lading Validation**:
   - The BOL number (`BOL#`) must be valid. An invalid BOL triggers "INVALID BILL OF LADING ENTERED" (MSG5).
   - The program likely checks `BOLEDIY` or `BBBOLY` for existing BOL records.

4. **Y/N Field Validation**:
   - Certain fields (e.g., flags like `KYEMTY` or `BX06`) must be 'Y' or 'N'. Invalid entries trigger "ENTRY MUST BE 'Y' OR 'N'" (MSG7).
   - The rule "BOTH OPTIONS CANNOT BE 'Y'" (MSG8) suggests mutually exclusive flags (e.g., two options cannot both be enabled).

5. **Free Days Validation**:
   - The program ensures that "free days" (likely a field like `KYFRDY` or related to freight terms) is not zero, triggering "FREE DAYS CANNOT BE ZERO" (MSG9).

6. **File Navigation**:
   - The program handles file navigation with messages like "END OF FILE HAS BEEN REACHED" (MSG3) and "BEGINNING OF FILE HAS BEEN REACHED" (MSG4), indicating sequential or keyed access to files like `EDIBOLTX` or `BOLEDIY`.

7. **EDI Segment Structure**:
   - The program structures data according to EDI 404 standards, with segments like:
     - **BX**: Transaction set header (e.g., `BX01` to `BX08`).
     - **N9**: Reference information (up to three references, `N9011` to `N9063`).
     - **N7**: Equipment details (e.g., `N701` to `D903`).
     - **N1**: Party identification (up to three parties, `N1011` to `N4033`).
     - **R2**: Routing information (up to 12 routes, `R2011` to `R203C`).
     - **L5**: Line item descriptions (multiple iterations, `H311` to `L065`).
     - **LH1**: Hazardous material information (e.g., `LH101` to `LH321`).
     - **PER**: Contact information (e.g., `PER011` to `PER042`).
   - Each segment is written to `EDIBOLTX` with a specific record type (e.g., `40402` for BX, `40403` for N9).

8. **Data Persistence**:
   - The program supports both adding new records (`ADD02` to `ADD0A`) and updating existing records (`UPD02` to `UPD0A`) in `EDIBOLTX`, ensuring data consistency.
   - Fields like `SAVBOL` and `SAVSRN` maintain continuity across transactions.

9. **Hazardous Materials**:
   - The `GSHAZM` file and `LH1` segment (`EDI810S8`) indicate handling of hazardous material data, likely for compliance with transportation regulations (e.g., CHEMTREC contract noted in `ARG`).

10. **Customer and Address Data**:
    - The `ARG` array contains static data for "AMERICAN REFINING GROUP" (address, phone, contact), suggesting predefined customer or company details used in EDI output.

---

### Tables (Files) Used

The program uses the following files, as defined in the **F-specs** and consistent with the OCL program:

1. **SCREEN**: Workstation file for interactive display (1200 bytes, likely a 5250 display file).
2. **EDIBOLTX**: EDI Bill of Lading transaction file (1340 bytes, update mode, 18-byte key, externally described with `EXTK`).
3. **BOLEDIY**: Bill of Lading history or detail file (386 bytes, input mode, 13-byte key).
4. **TRRTCD**: Transportation or routing code file (324 bytes, input mode, 6-byte key, 2 indexes).
5. **GSHAZM**: Hazardous materials file (384 bytes, input mode, 6-byte key, 2 indexes).
6. **GSPROD**: Product or inventory file (512 bytes, input mode, 6-byte key, 8 indexes).
7. **SHIPTO**: Customer ship-to file (2048 bytes, input mode, 11-byte key, 2 indexes, replaced `BBSHSP` per `DC01` comment).
8. **SHPADR**: Shipping address file (448 bytes, input mode, 13-byte key, 2 indexes).
9. **BBBOLY**: Secondary Bill of Lading file (512 bytes, input mode, 14-byte key, externally described).

---

### External Programs Called

The provided RPG source code does not explicitly call any external programs (e.g., via `CALL` operations in RPG). The program operates self-contained, handling all logic through screen interactions and file operations. However:
- The **EDI810** program itself is called by the **EDI810.ocl36** OCL script, as discussed in the previous query.
- No additional external RPG, CL, or other programs are referenced in the code.

---

### Summary

The **EDI810** RPG program is an interactive application for maintaining EDI 404 Bill of Lading transactions. It uses a workstation file (`SCREEN`) to capture user input across multiple formats (`EDI810S1` to `EDI810SG`) and processes data from various files (`EDIBOLTX`, `BOLEDIY`, `TRRTCD`, `GSHAZM`, `GSPROD`, `SHIPTO`, `SHPADR`, `BBBOLY`). The program validates input, enforces business rules (e.g., valid company/customer, Y/N flags, non-zero free days), and writes/updates EDI segments to `EDIBOLTX`. No external programs are called within the RPG code.

- **Process Steps**: Initialize, display/input via screens, validate data, read from input files, add/update records in `EDIBOLTX`, and handle errors/termination.
- **Business Rules**: Validate company number, customer, BOL, Y/N fields, free days; structure EDI 404 segments; ensure data persistence.
- **Files Used**: SCREEN, EDIBOLTX, BOLEDIY, TRRTCD, GSHAZM, GSPROD, SHIPTO, SHPADR, BBBOLY.
- **External Programs Called**: None (called by EDI810.ocl36).

If you need a deeper analysis of specific RPG calculations (C-specs, which were truncated) or further details on EDI 404 segment mappings, please provide the complete code or specify the focus area!