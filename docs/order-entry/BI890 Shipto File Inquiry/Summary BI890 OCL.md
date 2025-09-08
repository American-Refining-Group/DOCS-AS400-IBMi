The call stack consists of the following programs, all part of the Brandford Order Entry / Invoices system on an IBM AS/400 (iSeries) platform:

1. **BI890.rpg36.txt**: An RPG/400 program for ship-to master file maintenance and inquiry, providing an interactive interface for managing ship-to records, including customer details, addresses, product codes, and accessorial/marks data.
2. **BI907AC.clp.txt**: A CLP program that prepares a temporary work file (`QTEMP/BI907W`) and calls `BI907` for customer/ship-to product description and freight terms maintenance.
3. **BI907.rpgle.txt**: An RPGLE program for maintaining and inquiring about customer/ship-to product descriptions and freight terms, using a subfile interface.
4. **BB800E.rpgle.txt**: An RPGLE program for customer order instructions inquiry, displaying accessorial and marks instructions for orders or ship-to records.
5. **BI8903.rpgle.txt**: An RPGLE program for copying ship-to records from one customer/ship-to to another, updating related files.

Based on the analysis of these programs, I will identify the use cases implemented in the call stack and then provide a function requirement document for each use case, assuming they are implemented as large functions that process inputs programmatically without screen interaction.

### List of Use Cases

The call stack implements the following distinct use cases:

1. **Ship-to Master File Maintenance and Inquiry** (`BI890`):
   - Allows users to view, add, update, or delete ship-to master records, including customer details, addresses, product codes, accessorial/marks, and freight terms.
   - Validates inputs against customer, company, and product master files.
   - Supports inquiry (`INQ`) and maintenance (`MNT`) modes.
   - Calls `BB800E` for accessorial/marks inquiry, `BI907AC` for product/freight terms maintenance, and `BI9002` for address maintenance.

2. **Customer/Ship-to Product Description and Freight Terms Maintenance** (`BI907AC` and `BI907`):
   - Manages customer/ship-to product pricing records (`arcupr`), including product codes, gallons to bill, stock numbers, freight codes, and freight terms.
   - Supports adding, updating, deleting, or reactivating records in inquiry or maintenance modes.
   - Validates product and freight codes, with special handling for freight collect scenarios (e.g., non-Bradford locations, service fees).
   - Uses a temporary work file (`BI907W`) for intermediate data.

3. **Customer Order/Ship-to Instructions Inquiry** (`BB800E`):
   - Displays accessorial and marks instructions for customer orders (`bbora1`) or ship-to master records (`bbshsa1`) in inquiry mode.
   - Supports order mode (`p$ship = 0`) and ship-to mode (`p$ship ≠ 0`).
   - Validates customer and order/ship-to data, displaying up to 32 records in a subfile.

4. **Copy Ship-to Records** (`BI8903`):
   - Copies ship-to records and related data (pricing, addresses, accessorial/marks) from a source customer/ship-to to a target customer/ship-to within the same company.
   - Ensures the target ship-to does not already exist and clears specific fields in the new `shipto` record.

### Function Requirement Documents

Below are concise function requirement documents for each use case, designed as programmatic functions that take inputs and produce outputs without screen interaction. Each document outlines the process steps, business rules, and calculations (if applicable) in a business-focused manner.

---

<xaiArtifact artifact_id="b7e57883-f470-4060-b439-df94c282c69f" artifact_version_id="3e9356f0-f9ff-4fbd-8b0b-93b6b2e47fbe" title="ShipToMasterMaintenanceFunction.md" contentType="text/markdown">

# Function Requirement Document: ShipToMasterMaintenance

## Purpose
Programmatically maintain or inquire about ship-to master records, including customer details, addresses, product codes, accessorial/marks, and freight terms, for the Brandford Order Entry / Invoices system.

## Inputs
- **Company Number** (`cono`, 2 digits): Identifies the company.
- **Customer Number** (`cust`, 6 digits): Identifies the customer.
- **Ship-to Number** (`ship`, 3 digits): Identifies the ship-to (optional for inquiry).
- **Mode** (`mode`, 3 characters): 'INQ' for inquiry, 'MNT' for maintenance.
- **Ship-to Data** (`shipData`, structure): Includes fields like name (`csname`, 30 chars), address lines (`csadr1-4`, 30 chars each), city (`csctst`, 30 chars), state (`csstat`, 2 chars), zip (`cszip5`, 5 digits; `cscszp`, 15 chars), miles (`csmils`, 3 digits), product codes (`cspr01-24`, 4 chars each), backup ship-to (`cssbkp`, 15 chars), delivery location (`csdloc`, 3 chars), billing customer (`csbcus`, 6 digits), billing ship-to (`csbcsh`, 3 digits), memo customer (`csbmcu`, 10 chars), rail company (`csrrco`, 2 digits), country (`cscty`, 3 chars), export flag (`csexpo`, 1 char), FOB code (`csfob`, 1 char), contract owner (`cscoon`, 1 char), alternate owner (`csalto`, 1 char), longitude (`cslong`, 13 chars), latitude (`cslatt`, 13 chars), order marks (`csomk1-4`, 60 chars each), invoice marks (`csimk1-2`, 60 chars each), dispatch info (`csdsp1-4`, 60 chars each), bill of lading marks (`csbmk1-4`, 60 chars each), freight bill name (`csfrnm`, 30 chars), freight bill address (`csfra1-3`, 30 chars each), note status (`csnote`, 6 chars), customer ship-to ID (`cscsid`, 9 chars), max railcar weight (`csrcmx`, 7 digits), service fee flag (`cssvfe`, 1 char).
- **File Group** (`fgrp`, 1 char): 'G' or 'Z' for file overrides.

## Outputs
- **Status** (`status`, string): 'SUCCESS', 'ERROR', or specific error message (e.g., 'INVALID CUSTOMER NUMBER').
- **Ship-to Data** (`shipData`, structure): Updated or retrieved ship-to record (if applicable).
- **Product Data** (`productData`, array): Product codes (`prcd`, 4 chars), descriptions (`prds`, 30 chars), gallons to bill (`glcd`, 1 char), stock numbers (`stno`, 20 chars), freight codes (`pfrc`, 1 char), separate freight (`psfr`, 1 char), calculate freight (`pcaf`, 1 char), container type (`cnty`, 1 char).
- **Messages** (`messages`, array): Error or status messages (e.g., 'SHIP TO NOT FOUND').

## Process Steps
1. **Validate Inputs**:
   - Check `cono` against `bicont`. If invalid, return 'INVALID COMPANY NUMBER'.
   - Check `cust` against `arcust`. If invalid, return 'INVALID CUSTOMER NUMBER'.
   - If `ship` provided, check against `shipto` using `cskey` (`cono`, `cust`, `ship`). If not found, return 'SHIP TO NOT FOUND'.
   - If `ship = '999'`, return 'SHIPTO # 999 MAY NOT BE USED'.
2. **Inquiry Mode (`mode = 'INQ'`)**:
   - Retrieve `shipto` record using `cskey`.
   - If deleted (`csdel = 'D'`), return 'THIS SHIP TO WAS PREVIOUSLY DELETED' and `shipData`.
   - Retrieve product data from `arcupr` for `cono`, `cust`, `ship`.
   - Retrieve descriptions from `gsprod` and freight terms from `arcupr`.
   - Return `shipData`, `productData`, and 'SUCCESS'.
3. **Maintenance Mode (`mode = 'MNT'`)**:
   - If `ship` exists, update `shipto` with `shipData` fields, clearing `csdel` if reactivating.
   - If `ship` does not exist, create new `shipto` record with `shipData`.
   - Validate product codes (`cspr01-24`) against `gsprod`.
   - Update or add `arcupr` records with product data.
   - Write history to `arcuphs`.
   - Return updated `shipData`, `productData`, and 'SUCCESS' or error (e.g., 'INVALID CODE').
4. **Call External Functions**:
   - Invoke `CustomerOrderInstructionsInquiry` for accessorial/marks data (via `BB800E`).
   - Invoke `CustomerShipToProductMaintenance` for product/freight terms (via `BI907AC`/`BI907`).
   - Invoke `CustomerShipToAddressMaintenance` for address details (via `BI9002`).

## Business Rules
- **Validation**:
  - Company, customer, and ship-to must exist in `bicont`, `arcust`, and `shipto`.
  - Ship-to number 999 is reserved and cannot be used.
  - Product codes must exist in `gsprod` with `tpsell = 'Y'`.
- **Modes**:
  - Inquiry mode retrieves data without modifications.
  - Maintenance mode allows add, update, delete, or reactivation of records.
- **Data Integrity**:
  - Deleted records (`csdel = 'D'`) can be reactivated.
  - Address fields include city, state, and zip (DC01).
  - Maximum 24 product codes per ship-to.
- **Freight Terms**:
  - Gallons to bill (`glcd`) is 'G' (gross), 'N' (net), or blank.
  - Freight codes (`pfrc`) and terms (`psfr`, `pcaf`) validated against `gstabl`.

## Calculations
- None explicitly required; data is validated and copied without computational transformations.

</xaiArtifact>

---

<xaiArtifact artifact_id="8732544d-04ea-47af-bd5f-6d6d5ff2af7f" artifact_version_id="1e94ae79-3e84-4aac-b648-b6d2a3668d12" title="CustomerShipToProductMaintenanceFunction.md" contentType="text/markdown">

# Function Requirement Document: CustomerShipToProductMaintenance

## Purpose
Programmatically maintain or inquire about customer/ship-to product descriptions and freight terms, including product codes, stock numbers, and freight settings, for the Bradford Order Entry / Invoices system.

## Inputs
- **Company Number** (`cono`, 2 digits): Identifies the company.
- **Customer Number** (`cust`, 6 digits): Identifies the customer.
- **Ship-to Number** (`ship`, 3 digits): Identifies the ship-to.
- **Mode** (`mode`, 3 characters): 'INQ' for inquiry, 'MNT' for maintenance.
- **File Group** (`fgrp`, 1 char): 'G' or 'Z' for file overrides.
- **Product Data** (`productData`, array): Product code (`cpprod`, 4 chars), container type (`w$cnty`, 1 char), gallons to bill (`cpglcd`, 1 char), stock number (`cpcstk`, 20 chars), freight code (`cpfrcd`, 1 char), separate freight (`cpsfrt`, 1 char), calculate freight (`cpcafr`, 1 char).

## Outputs
- **Status** (`status`, string): 'SUCCESS', 'ERROR', or specific error message (e.g., 'INVALID CODE').
- **Product Data** (`productData`, array): Retrieved or updated product records.
- **Messages** (`messages`, array): Error or status messages (e.g., 'Record has been Copied').

## Process Steps
1. **Validate Inputs**:
   - Check `cono` against `bicont`. If invalid, return 'INVALID COMPANY NUMBER'.
   - Check `cust` against `arcust`. If invalid, return 'INVALID CUSTOMER NUMBER'.
   - Check `ship` against `shipto`. If invalid, return 'INVALID SHIP TO NUMBER'.
   - Validate `cpprod` against `gsprod`. If invalid, return 'Invalid Code, Prompt For Valid Codes'.
   - Validate `cpglcd` as 'G', 'N', or ' '. If invalid, return 'Invalid Code, Valid Code are: "G" " "'.
   - Validate `cpfrcd` against `gstabl` (`BBFRCD`). If invalid, return 'Invalid Freight Code'.
2. **Prepare Temporary File**:
   - Clear or create `QTEMP/BI907W` (handled by `BI907AC`).
3. **Inquiry Mode (`mode = 'INQ'`)**:
   - Retrieve `arcupr` records for `cono`, `cust`, `ship` using `kls1s1`.
   - Return `productData` with product codes, descriptions (from `gsprod`), and freight terms.
4. **Maintenance Mode (`mode = 'MNT'`)**:
   - **Add**: If no `arcupr` record exists for `cpprod` and `w$cnty`, add new record to `arcupr`. Require at least one product code (`err(01)`). If record exists or is deleted, return 'Record Already exist, Or Marked as Deleted, Cannot Add Record'.
   - **Update**: Update existing `arcupr` records with `productData`.
   - **Delete/Reactivate**: Mark records as deleted or reactivate them, updating `bi907w`.
   - **Copy**: Copy records to `bi907w` and `arcuphs` (history).
   - Write history to `arcuphs`.
   - Return updated `productData` and 'SUCCESS' or error.
5. **Freight Terms Validation**:
   - For `cpfrcd = 'C'`, ensure `cpsfrt` and `cpcafr` are ' ', 'Y', or 'N' (`err(07)`, `err(08)`).
   - For `cpfrcd = 'A'`, ensure `cpsfrt = 'Y'` (freight collect with $100 service fee, `err(09)`).

## Business Rules
- **Validation**:
  - Product codes must exist in `gsprod`.
  - Freight codes must exist in `gstabl`.
  - Gallons to bill (`cpglcd`) must be 'G', 'N', or ' '.
  - Freight collect (`cpfrcd = 'C'`) supports non-Bradford locations (e.g., Anchor, JB01).
  - Freight code `A` requires separate freight (`cpsfrt = 'Y'`) with a $100 service fee (JB02).
- **Modes**:
  - Inquiry mode retrieves data without changes.
  - Maintenance mode supports add, update, delete, or reactivation.
- **Data Integrity**:
  - At least one product code required in Add mode.
  - Records marked deleted can be reactivated.
- **Temporary Storage**:
  - Uses `QTEMP/BI907W` for intermediate data and `arcuphs` for history.

## Calculations
- **Service Fee (JB02)**: For `cpfrcd = 'A'`, apply a fixed $100 service fee for freight collect arranged by ARG but billed by the carrier.

</xaiArtifact>

---

<xaiArtifact artifact_id="881ba587-8456-4d64-a99e-9b2ccc15732d" artifact_version_id="1bb22e4f-27ce-4563-925c-46b25283ecbd" title="CustomerOrderInstructionsInquiryFunction.md" contentType="text/markdown">

# Function Requirement Document: CustomerOrderInstructionsInquiry

## Purpose
Programmatically retrieve accessorial and marks instructions for customer orders or ship-to master records in the Brandford Order Entry / Invoices system.

## Inputs
- **Company Number** (`cono`, 2 digits): Identifies the company.
- **Customer/Order Number** (`csor`, 6 digits): Customer number (ship-to mode) or order number (order mode).
- **Ship-to Number** (`ship`, 3 digits): Ship-to number (0 for order mode).
- **File Group** (`fgrp`, 1 char): 'G' or 'Z' for file overrides.

## Outputs
- **Status** (`status`, string): 'SUCCESS', 'ERROR', or specific error message (e.g., 'INVALID CUSTOMER NUMBER').
- **Instructions Data** (`instructionsData`, array): Records containing record ID (`barcid`, 4 digits), quantity (`baqty`, numeric), description (`badesc`, 30 chars), total cost (`batcst`, numeric), reason code (`barccd`, 2 chars), pick flag (`bapick`, 'Y'/'N'), dispatch flag (`badsph`, 'Y'/'N'), bill of lading flag (`babol`, 'Y'/'N'), invoice flag (`bainv`, 'Y'/'N').
- **Customer Name** (`csnm`, 30 chars): Customer name from `arcust`.
- **Ship-to Name** (`shnm`, 30 chars): Ship-to name from `shipto` (ship-to mode only).
- **Messages** (`messages`, array): Error or status messages.

## Process Steps
1. **Validate Inputs**:
   - Check `cono` against `bicont`. If invalid, return 'INVALID COMPANY NUMBER'.
   - Check `csor` against `arcust` (ship-to mode) or `bbordh` (order mode). If invalid, return 'INVALID CUSTOMER NUMBER' or 'INVALID ORDER NUMBER'.
   - If `ship ≠ 0`, check against `shipto`. If invalid, return 'INVALID SHIP TO NUMBER'.
2. **Retrieve Data**:
   - **Order Mode (`ship = 0`)**:
     - Retrieve customer name (`arname`) from `arcust` using `klcus1`.
     - Retrieve order instructions from `bbora1` using `klc1r1`.
   - **Ship-to Mode (`ship ≠ 0`)**:
     - Retrieve customer name (`arname`) from `arcust` using `klcus2`.
     - Retrieve ship-to name (`csname`) from `shipto` using `klship`.
     - Retrieve ship-to instructions from `bbshsa1` using `klc1r2`.
   - Format each record with `barcid`, `baqty`, `badesc`, `batcst`, `barccd`, `bapick`, `badsph`, `babol`, `bainv`.
3. **Return Results**:
   - Return `instructionsData`, `csnm`, `shnm` (if applicable), and 'SUCCESS' or error.

## Business Rules
- **Modes**:
  - Order mode (`ship = 0`): Retrieves instructions for a customer order from `bbora1`.
  - Ship-to mode (`ship ≠ 0`): Retrieves instructions for a customer ship-to from `bbshsa1`.
- **Validation**:
  - Company, customer, and order/ship-to must exist in respective files.
- **Data Limits**:
  - Retrieves up to 32 records per query.
- **Inquiry Only**:
  - No modifications allowed; read-only access.

## Calculations
- None required; data is retrieved and formatted without transformations.

</xaiArtifact>

---

<xaiArtifact artifact_id="215ad8d6-98d1-4cd2-9e2e-bd0d0d46287c" artifact_version_id="4316b2fa-c56e-4c32-b710-2d41cb2493f9" title="CopyShipToRecordsFunction.md" contentType="text/markdown">

# Function Requirement Document: CopyShipToRecords

## Purpose
Programmatically copy ship-to records and related data (pricing, addresses, accessorial/marks) from a source customer/ship-to to a target customer/ship-to within the same company in the Brandford Order Entry / Invoices system.

## Inputs
- **Company Number** (`cono`, 2 digits): Identifies the company.
- **Source Customer Number** (`fcust`, 6 digits): Source customer.
- **Source Ship-to Number** (`fshp`, 3 digits): Source ship-to.
- **Target Customer Number** (`tcust`, 6 digits): Target customer.
- **Target Ship-to Number** (`tshp`, 3 digits): Target ship-to.
- **File Group** (`fgrp`, 1 char): 'G' or 'Z' for file overrides.

## Outputs
- **Status** (`status`, string): 'SUCCESS', 'ERROR', or specific error message (e.g., 'TARGET SHIP-TO EXISTS').
- **Messages** (`messages`, array): Status messages (e.g., 'Record has been Copied').

## Process Steps
1. **Validate Inputs**:
   - Check `cono` against `bicont`. If invalid, return 'INVALID COMPANY NUMBER'.
   - Check `fcust` and `fshp` against `shipto` using `klFrmShipto`. If not found, return 'SOURCE SHIP-TO NOT FOUND'.
   - Check `tcust` against `arcust`. If invalid, return 'INVALID TARGET CUSTOMER NUMBER'.
   - Check `tshp` against `shipto` using `klToShipto`. If exists, return 'TARGET SHIP-TO EXISTS'.
2. **Copy Ship-to Record**:
   - Retrieve source `shipto` record using `klFrmShipto`.
   - Create new `shipto` record with `tcust` and `tshp`, clearing fields: `cscsid`, `csadr1-4`, `cszip5`, `cszip9`, `csstat`, `csmils`, `csctst`, `cscszp`, `cslong`, `cslatt`.
3. **Copy Related Records**:
   - For each `arcuprrd` record (pricing) matching `cono`, `fcust`, `fshp`:
     - Create new `arcupr` record with `tcust`, `tshp`.
   - For each `cuadrrd` record (addresses) matching `cono`, `fcust`, `fshp`:
     - Create new `cuadr` record with `tcust`, `tshp`.
   - For each `bbshsa1rd` record (accessorial/marks) matching `cono`, `fcust`, `fshp`:
     - Create new `bbshsa` record with `tcust`, `tshp`, updating `bahkey`.
4. **Return Results**:
   - Return 'SUCCESS' and 'Record has been Copied'.

## Business Rules
- **Validation**:
  - Source ship-to must exist; target ship-to must not exist.
  - Company and target customer must be valid.
- **Data Integrity**:
  - Clears specific `shipto` fields to ensure clean data.
  - Copies all related records (`arcupr`, `cuadr`, `bbshsa`) without modification except for customer/ship-to fields.
- **File Overrides**:
  - Uses `fgrp` to select `G` or `Z` files with no shared access.

## Calculations
- None required; data is copied without transformations.

</xaiArtifact>

---

### Summary

The call stack implements four use cases: ship-to master maintenance, customer/ship-to product maintenance, customer order/ship-to instructions inquiry, and ship-to record copying. Each function requirement document describes a programmatic implementation that takes inputs, validates them, processes data according to business rules, and returns outputs without screen interaction. The documents focus on business requirements, including validation, data integrity, and specific freight term rules (e.g., $100 service fee for freight code `A`), with minimal calculations involved.

If you need further refinements or additional details (e.g., specific subroutine implementations from truncated code), please let me know!