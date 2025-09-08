The call stack consists of the **EDI810.ocl36** OCL program and the **EDI810** RPG program, which together implement an **EDI Outbound Bill of Lading Tracking Maintenance** system for generating and maintaining EDI 404 (Rail Carrier Shipment Information) transactions. Below, I will first identify the **use cases** implemented by this call stack, followed by a **functional requirements document** for a non-interactive version of the primary use case, assuming inputs are provided programmatically rather than via a screen.

---

### Use Cases Implemented by the Call Stack

The call stack supports multiple use cases related to managing EDI Bill of Lading (BOL) transactions. Based on the OCL and RPG code, the following use cases are implemented:

1. **Create New EDI Bill of Lading Transaction**:
   - **Description**: Allows users to create a new EDI 404 transaction by entering details such as BOL number, order number, serial number, routing, equipment, party identification, hazardous materials, and contact information.
   - **Details**: 
     - Users interact with screen formats (`EDI810S1` to `EDI810S9`, `EDI810SA` to `EDI810SG`) to input data for various EDI segments (BX, N9, N7, N1, R2, L5, LH1, PER).
     - Data is validated (e.g., valid company number, customer, BOL) and written to the `EDIBOLTX` file with record types like `40402` (BX), `40403` (N9), etc.
     - Supported by `ADD02` to `ADD0A` operations in the RPG program.
   - **Trigger**: User initiates via the `SCREEN` file, starting with `EDI810S1` to input key fields (e.g., `BOL#`, `SRN#`).

2. **Update Existing EDI Bill of Lading Transaction**:
   - **Description**: Allows users to modify existing EDI 404 transactions by updating fields in the `EDIBOLTX` file.
   - **Details**:
     - Users retrieve an existing BOL record (using `BOL#`, `SRN#`, or other keys) and modify fields across the same screen formats as above.
     - Updates are written to `EDIBOLTX` using `UPD02` to `UPD0A` operations.
     - Validation ensures data integrity (e.g., Y/N fields, non-zero free days).
   - **Trigger**: User selects an existing BOL via `EDI810S1` and navigates through screens to update data.

3. **Validate Bill of Lading Data**:
   - **Description**: Validates user-entered data against business rules to ensure compliance with EDI 404 standards and internal requirements.
   - **Details**:
     - Checks include valid company number, customer existence (via `SHIPTO`), valid BOL (via `BOLEDIY` or `BBBOLY`), Y/N field constraints, and non-zero free days.
     - Error messages (e.g., "INVALID COMPANY NUMBER ENTERED", "CUSTOMER NOT FOUND") are displayed on the screen.
   - **Trigger**: Occurs during data entry in any screen format before writing to `EDIBOLTX`.

4. **Rekey BOL Data**:
   - **Description**: Allows users to re-enter or correct data by returning to the initial screen (`EDI810S1`) without exiting the program.
   - **Details**:
     - Triggered by the **KA** function key, which resets the input process to `EDI810S1`.
     - Useful for correcting errors or starting a new transaction without restarting the program.
   - **Trigger**: User presses **KA** during screen interaction.

5. **End of Job (EOJ)**:
   - **Description**: Terminates the program cleanly, resetting switches and clearing local variables.
   - **Details**:
     - Triggered by the **KG** function key or conditional logic in the OCL (`SWITCH2-1` set to 1 skips execution).
     - The OCL script resets switches (`SWITCH 00000000`) and clears local variables (`LOCAL BLANK-*ALL`).
   - **Trigger**: User presses **KG** or system condition met in OCL.

6. **Handle Hazardous Materials Information**:
   - **Description**: Captures and processes hazardous material data for compliance with transportation regulations (e.g., CHEMTREC contract).
   - **Details**:
     - Uses the `GSHAZM` file and `EDI810S8` screen format for LH1 segment data (e.g., `LH101` to `LH321`).
     - Outputs hazardous material details to `EDIBOLTX` with record type `40408`.
   - **Trigger**: User enters hazardous material data in `EDI810S8`.

---

### Functional Requirements Document

Below is a **functional requirements document** for a non-interactive version of the primary use case: **Create New EDI Bill of Lading Transaction**. This assumes the function receives all necessary inputs programmatically (e.g., via parameters or a data structure) rather than through interactive screens. The document focuses on business requirements, process steps, and necessary calculations, presented concisely.

<xaiArtifact artifact_id="0d91abf1-a3fe-4494-a114-0b616d39f024" artifact_version_id="6e85b49c-5754-428a-b1eb-e211cf5e3152" title="EDI810_CreateBOL_Function_Requirements.md" contentType="text/markdown">

# Functional Requirements: Create EDI Bill of Lading Transaction

## Overview
This function creates a new EDI 404 (Rail Carrier Shipment Information) transaction for outbound Bill of Lading (BOL) tracking. It accepts structured input data, validates it against business rules, and writes EDI segment records to the `EDIBOLTX` file.

## Inputs
The function accepts a data structure containing the following fields, grouped by EDI segment:
- **BX Segment** (Transaction Set Header):
  - `BX01` (2 chars): Transaction set purpose code.
  - `BX02` (1 char): Transportation method code.
  - `BX03` (2 chars): Shipment method of payment.
  - `BX05` (4 chars): Shipment qualifier.
  - `BX06` (1 char): Capacity load code.
  - `BX07` (1 char): Status report request code.
  - `BX08` (1 char): Transaction type code.
  - `BNX01` (1 char): Rail shipment type code.
  - `BNX03` (1 char): Billing code.
  - `BNX04` (5 chars): Repetitive pattern number.
  - `M301` (1 char): Yes/No flag for hazardous material.
  - `M302` (8 chars): Standard carrier alpha code.
- **N9 Segment** (Reference Information, up to 3 references):
  - `N901x` (2 chars): Reference identifier qualifier (x = 1, 2, 3).
  - `N902x` (50 chars): Reference identifier.
  - `N904x` (8 chars): Date.
  - `N905x` (8 chars): Time.
  - `N906x` (2 chars): Time code.
- **N7 Segment** (Equipment Details):
  - `N701` (4 chars): Equipment number prefix.
  - `N702` (15 chars): Equipment number.
  - `N703` (10 chars): Weight.
  - `N704` (1 char): Weight qualifier.
  - `N705` (8 chars): Tare weight.
  - `N711` (2 chars): Equipment type.
  - `F902` (30 chars): Equipment description.
  - `F903` (2 chars): Equipment status code.
  - `D902` (30 chars): Dunnage description.
  - `D903` (2 chars): Dunnage weight qualifier.
- **N1 Segment** (Party Identification, up to 3 parties):
  - `N101x` (3 chars): Entity identifier code (x = 1, 2, 3).
  - `N102x` (30 chars): Name.
  - `N301x` (55 chars): Address line 1.
  - `N302x` (55 chars): Address line 2.
  - `N401x` (30 chars): City.
  - `N402x` (2 chars): State/province code.
  - `N403x` (15 chars): Postal code.
- **N12 Segment** (Additional Party Details, up to 3 parties):
  - Same fields as N1 (`N121x` to `N423x`).
- **R2 Segment** (Routing, up to 12 routes):
  - `R201x` (4 chars): Standard carrier alpha code (x = 1 to C).
  - `R202x` (1 char): Routing sequence code.
  - `R203x` (30 chars): City name or carrier name.
- **L5 Segment** (Line Item Description, up to 6 iterations):
  - `H3x1` (3 chars): Commodity code qualifier (x = 1 to 6).
  - `LXx1` (1 char): Line item number.
  - `L5x1` (84 chars): Description.
  - `L5x2` (50 chars): Commodity code.
  - `L5x3` (30 chars): Packaging code.
  - `L5x4` (1 char): Marks and numbers qualifier.
  - `L5x6` (48 chars): Commodity code qualifier.
  - `L5x7` (2 chars): Compartment ID code.
  - `L0x4` (10 chars): Weight.
  - `L0x5` (2 chars): Weight qualifier.
- **LH1 Segment** (Hazardous Materials):
  - `LH101` (2 chars): Hazardous material code qualifier.
  - `LH102` (7 chars): Hazardous material class.
  - `LH103` (6 chars): Hazardous material code.
  - `LH105` (30 chars): Hazardous material description.
  - `LH110` (3 chars): Hazardous material contact.
  - `LH201` (30 chars): Hazardous placard notation.
  - `LH202` (1 char): Hazardous endorsement.
  - `LH301` (25 chars): Hazardous material shipment information.
  - `LH302` (1 char): Hazardous material shipment qualifier.
  - `LHT01` (30 chars): Packing group code.
  - `LHT02` (40 chars): Hazardous material weight.
  - `LH601` (60 chars): Freeform hazardous material information.
  - `LH321` (25 chars): CHEMTREC contract number.
- **PER Segment** (Contact Information, up to 2 contacts):
  - `PER01x` (2 chars): Contact function code (x = 1, 2).
  - `PER02x` (60 chars): Name.
  - `PER03x` (2 chars): Communication number qualifier.
  - `PER04x` (21 chars): Communication number.
- **Key Fields**:
  - `BOL#` (7 chars): Bill of Lading number.
  - `SRN#` (3 chars): Serial number.
  - `KYORD#` (6 chars): Order number.
  - `KYRULE` (1 char): Rule indicator.
  - `KYEMTY` (1 char): Empty indicator (Y/N).
  - `KYFRDY` (2 chars): Freight ready days.
- **Optional Fields**:
  - `TPPROD` (4 chars): Top product code.
  - `BDPROD` (4 chars): Bulk product code.
  - `BDNGAL` (7 chars): Bulk gallons.

## Process Steps
1. **Receive Input Data**:
   - Accept a data structure containing all required and optional fields listed above.
2. **Validate Input Data**:
   - Check company number exists in `SHIPTO` file.
   - Verify customer exists in `SHIPTO` file.
   - Validate `BOL#` is unique and not already in `BOLEDIY` or `BBBOLY`.
   - Ensure `KYEMTY` and other Y/N fields are 'Y' or 'N'.
   - Confirm no mutually exclusive Y/N flags are both 'Y'.
   - Ensure `KYFRDY` (freight ready days) is non-zero.
   - Validate hazardous material data against `GSHAZM` file if `M301` = 'Y'.
3. **Retrieve Supporting Data**:
   - Read `BOLEDIY` for BOL history.
   - Read `TRRTCD` for routing codes.
   - Read `GSPROD` for product details.
   - Read `SHPADR` for address information.
   - Read `BBBOLY` for additional BOL data.
4. **Generate EDI Segments**:
   - Map input fields to EDI 404 segments (BX, N9, N7, N1, N12, R2, L5, LH1, PER).
   - Assign record types (`40402` to `40409`) and keys (`BOL#`, `SRN#`, `SEQ#` = '000' or '001' for N12).
5. **Write to EDIBOLTX**:
   - Write records to `EDIBOLTX` for each segment using `ADD02` to `ADD0A` formats.
   - Include `SAVBOL` (BOL#), `SAVSRN` (SRN#), and static fields (e.g., `ZERO3` = '000').
6. **Return Status**:
   - Return success status or error codes with messages (e.g., "INVALID COMPANY NUMBER", "CUSTOMER NOT FOUND").

## Business Rules
1. **Company and Customer Validation**:
   - Company number must exist in `SHIPTO`.
   - Customer must be valid and not previously deleted.
2. **BOL Uniqueness**:
   - `BOL#` must be unique and not exist in `BOLEDIY` or `BBBOLY`.
3. **Y/N Field Constraints**:
   - Fields like `KYEMTY`, `BX06`, `BNX03` must be 'Y' or 'N'.
   - Mutually exclusive Y/N fields (e.g., two options) cannot both be 'Y'.
4. **Freight Ready Days**:
   - `KYFRDY` must be greater than zero.
5. **Hazardous Materials**:
   - If `M301` = 'Y', validate hazardous material fields against `GSHAZM` and include `LH321` (CHEMTREC contract number).
6. **EDI 404 Compliance**:
   - All segments must conform to EDI 404 standards (e.g., field lengths, mandatory fields).
   - Up to 3 N9 references, 3 N1/N12 parties, 12 R2 routes, 6 L5 iterations, 2 PER contacts.
7. **Data Persistence**:
   - All segments are written to `EDIBOLTX` with consistent keys (`BOL#`, `SRN#`).

## Calculations
- **Sequence Number (`SEQ#`)**:
  - Set to '000' for most segments, '001' for N12 segment (`ADD052`).
- **Record Type Assignment**:
  - Assign `40402` (BX), `40403` (N9), `40404` (N7), `40405` (N1/N12), `40406` (R2), `40407` (L5), `40408` (LH1), `40409` (PER) based on segment type.
- **Field Mapping**:
  - Map input fields directly to `EDIBOLTX` positions (e.g., `BX01` to position 24, `N9021` to 25-74).
- **Default Values**:
  - Set `ZERO3` to '000' for all records.
  - Include static data from `ARG` (e.g., "AMERICAN REFINING GROUP", CHEMTREC contract) for relevant fields.

## Outputs
- **EDIBOLTX Records**:
  - Multiple records written with record types `40402` to `40409`, containing mapped input data.
- **Status**:
  - Success indicator or error message (e.g., "INVALID BILL OF LADING ENTERED").

## Dependencies
- **Files**:
  - `EDIBOLTX` (update, output).
  - `BOLEDIY`, `TRRTCD`, `GSHAZM`, `GSPROD`, `SHIPTO`, `SHPADR`, `BBBOLY` (input).
- **No External Programs**:
  - Function operates self-contained, called by OCL or another process.

## Error Handling
- Return error codes and messages for:
  - Invalid company number.
  - Customer not found.
  - Duplicate or invalid BOL.
  - Invalid Y/N fields.
  - Zero free days.
  - Missing or invalid hazardous material data.

</xaiArtifact>

---

### Notes
- The functional requirements focus on the primary use case (**Create New EDI Bill of Lading Transaction**) as it encompasses the core functionality of the RPG program. Other use cases (e.g., update, validate, rekey, EOJ, hazardous materials) are secondary and often integrated into the create process (e.g., validation is part of creation).
- The non-interactive version eliminates the `SCREEN` file, assuming inputs are provided via a data structure or API call, making it suitable for batch processing or integration with modern systems.
- Calculations are minimal, primarily involving field mapping and static value assignments (e.g., `ZERO3`, record types). The RPG program’s C-specs (truncated) likely contain additional logic, but the provided code suggests straightforward data mapping.

If you need requirements for additional use cases (e.g., Update BOL, Handle Hazardous Materials) or further details on calculations, please let me know!