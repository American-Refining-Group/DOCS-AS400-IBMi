The call stack provided includes the following programs from the Brandford Order Entry / Invoices system: `BI890.ocl36.txt` (main OCL), `GB730P.rpgle`, `BB800E.rpgle`, `BI907AC.clp`, `BI907.rpgle`, `BI9078.rpgle`, `BI907C2.clp`, and `BI8902R.rpgle`. These programs collectively form part of the ship-to master maintenance workflow, handling various aspects of customer, ship-to, product, and order data management. Below, I’ll identify the distinct use cases implemented by these programs and provide a concise function requirement document for each, focusing on business requirements and necessary calculations, assuming the functions process inputs programmatically rather than interactively via screens.

---

### **List of Use Cases**

Based on the analysis of the call stack, the following distinct use cases are implemented:

1. **Global File Tracking History Inquiry** (`GB730P`):
   - Displays historical changes to records across multiple files (e.g., ship-to, customer, product, order) for auditing purposes.
2. **Customer Order Instructions Inquiry** (`BB800E`):
   - Displays accessorial and marks information for open orders or ship-to master records.
3. **Customer/Ship-to Product Description and Freight Terms Maintenance** (`BI907`, `BI907AC`, `BI907C2`):
   - Manages (add, update, delete, inquire) customer/ship-to product descriptions and freight terms, including special freight conditions.
4. **Customer/Ship-to Product Description Copy** (`BI9078`):
   - Copies product description records from one customer/ship-to to another, with reporting.
5. **Customer Ship-to Address Inquiry and Add** (`BI8902R`):
   - Displays and adds customer ship-to address records.

These use cases are derived from the distinct functionalities of each program, as coordinated by the OCL (`BI890`). The OCL serves as the orchestrator, calling these programs based on user selections or workflow requirements, while `BI907AC` and `BI907C2` support `BI907` by managing a temporary work file.

---

### **Function Requirement Documents**

Each use case is documented as a programmatic function that processes inputs to produce outputs, focusing on business requirements and calculations. The documents are concise, emphasizing the process steps and rules without interactive screen details.

<xaiArtifact artifact_id="02874725-1037-4649-9b24-e6fcc843f2bc" artifact_version_id="71062bc7-2213-45b4-9a7d-5a5a5ab8c3a5" title="Global_File_Tracking_History_Inquiry.md" contentType="text/markdown">

# Function Requirement: Global File Tracking History Inquiry

## Description
Retrieves and returns historical change records for specified files (e.g., ship-to, customer, product, order) for a given company and key fields.

## Inputs
- `company` (2 characters): Company number.
- `file_type` (10 characters): File to query (e.g., `SHIPTO`, `ARCUPR`, `ARCUST`, `BBORDH`).
- `file_group` (1 character): File group (`G` or `Z`).
- `key_fields` (structure): Key fields specific to `file_type` (e.g., customer, ship-to, product).

## Outputs
- `history_records` (array): List of historical records with fields: change date, user, from/to values, and file-specific data.
- `header` (string): Descriptive header for the file type (e.g., "Ship-To Information Tracking").
- `status` (string): Success or error message (e.g., "Invalid file type").

## Process Steps
1. **Validate Inputs**: Confirm `company` exists in `GLCONT` and `file_type` is valid (one of 21 supported files). Validate `key_fields` against `ARCUST` or `SHIPTO` if applicable.
2. **Apply File Overrides**: Redirect file access to `G*` or `Z*` files based on `file_group` (e.g., `GSHIPTZ` or `ZSHIPTZ`).
3. **Retrieve Header**: Set `header` based on `file_type` (e.g., "Customer Product Code Tracking" for `ARCUPR`).
4. **Query History**: Read history file (e.g., `SHIPTHX` for `SHIPTO`, `ARCUPHX` for `ARCUPR`) using `key_fields`. Collect records with change date, user, and field values.
5. **Return Results**: Output `history_records`, `header`, and `status`.

## Business Rules
- Supports 21 file types (e.g., `SHIPTO`, `ARCUPR`, `BBORDH`) with specific key fields.
- Cannot query both system and program IDs simultaneously.
- Validates `company` and `key_fields` against master files (e.g., `ARCUST`, `SHIPTO`).
- History records are read-only, retrieved from files like `SHIPTHX`, `ARCUPHX`.
- Supports `G` or `Z` file groups for data segregation.

## Calculations
- None required; retrieves and formats existing historical data.

</xaiArtifact>

<xaiArtifact artifact_id="c8b49abf-475d-4316-a401-712716ff8176" artifact_version_id="25dd0f2a-79cf-4800-94d4-255ccadcbabf" title="Customer_Order_Instructions_Inquiry.md" contentType="text/markdown">

# Function Requirement: Customer Order Instructions Inquiry

## Description
Retrieves accessorial and marks information for either open orders or ship-to master records based on input parameters.

## Inputs
- `company` (2 characters): Company number.
- `customer_or_order` (6 characters): Customer number (ship-to mode) or order number (order mode).
- `ship_to` (3 characters): Ship-to number (zero for order mode).
- `file_group` (1 character): File group (`G` or `Z`).

## Outputs
- `instructions` (array): List of accessorial/marks records with fields: record ID, description, and related data.
- `header` (string): Header based on mode (e.g., "Open Order Accessorials/Marks Inquiry").
- `status` (string): Success or error message (e.g., "Invalid customer").

## Process Steps
1. **Validate Inputs**: Confirm `company` exists in `GLCONT`, `customer_or_order` exists in `ARCUST` (ship-to mode) or `BBORDH` (order mode), and `ship_to` exists in `SHIPTO` if non-zero.
2. **Apply File Overrides**: Redirect file access to `G*` or `Z*` files (e.g., `GBBORA1`, `ZBBORA1`) based on `file_group`.
3. **Determine Mode**: If `ship_to=0`, use order mode (`BBORA1`); otherwise, use ship-to mode (`BBSHSA1`).
4. **Retrieve Data**: Query `BBORA1` (order mode) or `BBSHSA1` (ship-to mode) using `company`, `customer_or_order`, and `ship_to` (if applicable). Collect accessorial/marks records.
5. **Set Header**: Assign "Open Order Accessorials/Marks Inquiry" for order mode or "ShipTo Master Accessorials/Marks Inquiry" for ship-to mode.
6. **Return Results**: Output `instructions`, `header`, and `status`.

## Business Rules
- Order mode (`ship_to=0`): Queries `BBORA1` for order-specific accessorial/marks.
- Ship-to mode (`ship_to≠0`): Queries `BBSHSA1` for customer/ship-to accessorial/marks.
- Validates inputs against `ARCUST`, `BBORDH`, and `SHIPTO`.
- Read-only operation; no updates allowed.
- Supports `G` or `Z` file groups.

## Calculations
- None required; retrieves and formats existing data.

</xaiArtifact>

<xaiArtifact artifact_id="5ff7f497-886d-4e49-8350-6c6d93004283" artifact_version_id="418468db-59ad-45af-8393-d6240077a5b6" title="Customer_ShipTo_Product_Freight_Maintenance.md" contentType="text/markdown">

# Function Requirement: Customer/Ship-to Product Description and Freight Terms Maintenance

## Description
Manages (add, update, delete, inquire) customer/ship-to product descriptions and freight terms in the `ARCUPR` file, with history logging and temporary work file processing.

## Inputs
- `company` (2 characters): Company number.
- `customer` (6 characters): Customer number.
- `ship_to` (3 characters): Ship-to number.
- `mode` (3 characters): Operation mode (`ADD`, `UPD`, `ALL`, `INQ`).
- `file_group` (1 character): File group (`G` or `Z`).
- `records` (array, for `ADD`/`UPD`): List of records with fields: `product` (15 characters), `container_type` (3 characters), `description` (40 characters), `freight_code` (1 character), `separate_freight` (1 character), `calc_freight` (1 character).

## Outputs
- `updated_records` (array): List of processed records with status (added, updated, deleted).
- `status` (string): Success or error message (e.g., "Invalid product code").
- `history_records` (array): Logged changes in `ARCUPHS`.

## Process Steps
1. **Validate Inputs**: Confirm `company` exists in `BICONT`, `customer` in `ARCUST`, `ship_to` in `SHIPTO`. Validate `mode` as `ADD`, `UPD`, `ALL`, or `INQ`.
2. **Apply File Overrides**: Redirect to `G*` or `Z*` files (e.g., `GARCUPR`, `ZARCUPR`) based on `file_group`.
3. **Clear Work File**: Clear `QTEMP/BI907W` (handled by `BI907AC` or `BI907C2`).
4. **Process Records**:
   - **Inquiry Mode (`INQ`)**: Read `ARCUPR` using `company`, `customer`, `ship_to`. Return records.
   - **Add Mode (`ADD`)**: Validate at least one product code. Check for existing/deleted records. Write new records to `ARCUPR` and `BI907W`. Log to `ARCUPHS`.
   - **Update Mode (`UPD`)**: Update existing `ARCUPR` records. Reactivate deleted records if needed. Log to `ARCUPHS`.
   - **All Mode (`ALL`)**: Combine inquiry and selective update/delete. Process `records` as specified.
5. **Validate Fields**: Ensure `product` exists in `GSPROD`, `container_type` in `GSTABL` (`CNTRTY`), `freight_code` in `GSTABL` (`BBFRCD`). Validate:
   - `freight_code='C'`: `separate_freight` and `calc_freight` must be `' '`, `'Y'`, or `'N'`.
   - `freight_code='A'`: `separate_freight` must be `'Y'`.
6. **Log History**: Write changes (add, update, delete) to `ARCUPHS` with timestamp and user.
7. **Return Results**: Output `updated_records`, `status`, and `history_records`.

## Business Rules
- Supports `ADD`, `UPD`, `ALL`, `INQ` modes.
- Validates inputs against `BICONT`, `ARCUST`, `SHIPTO`, `GSPROD`, `GSTABL`.
- `ADD` requires at least one product code; prevents duplicates or adding deleted records.
- `freight_code='C'`: Supports `CNY` (calc freight for non-Bradford locations) and `CYY` (freight collect with $100 service fee).
- Logs all changes to `ARCUPHS`.
- Uses `QTEMP/BI907W` for temporary processing.
- Supports `G` or `Z` file groups.

## Calculations
- **Freight Service Fee** (`CYY`): Adds $100 fee to freight cost when `freight_code='C'` and `separate_freight='Y'`.

</xaiArtifact>

<xaiArtifact artifact_id="d807c08d-94b2-4de1-a57e-599e584ee0a5" artifact_version_id="5ee331bd-c647-4b18-9123-031dfef8a272" title="Customer_ShipTo_Product_Description_Copy.md" contentType="text/markdown">

# Function Requirement: Customer/Ship-to Product Description Copy

## Description
Copies product description records from a source customer/ship-to to a target customer/ship-to in the `ARCUPR` file, with history logging and a printed report.

## Inputs
- `company` (2 characters): Company number.
- `source_customer` (6 characters): Source customer number.
- `source_ship_to` (3 characters): Source ship-to number.
- `target_customer` (6 characters): Target customer number.
- `target_ship_to` (3 characters): Target ship-to number.
- `file_group` (1 character): File group (`G` or `Z`).

## Outputs
- `copied_records` (array): List of copied records with fields: `product`, `container_type`, `description`, `new_record_flag`.
- `report` (string): Formatted report content with copy details.
- `history_records` (array): Logged changes in `ARCUPHS`.
- `status` (string): Success or error message (e.g., "Invalid source customer").

## Process Steps
1. **Validate Inputs**: Confirm `company` exists in `BICONT`, `source_customer`/`source_ship_to` and `target_customer`/`target_ship_to` exist in `ARCUST` and `SHIPTO`.
2. **Apply File Overrides**: Redirect to `G*` or `Z*` files (e.g., `GARCUP3`, `ZARCUP3`) based on `file_group`.
3. **Read Source Records**: Query `ARCUP3` using `company`, `source_customer`, `source_ship_to` to retrieve product records.
4. **Copy Records**: For each source record:
   - Check if a record exists in `ARCUPR` for `company`, `target_customer`, `target_ship_to`, `product`, `container_type`.
   - If not, write to `ARCUPR` with target details, setting `new_record_flag='1'`.
   - Log to `ARCUPHS` with timestamp and user.
5. **Generate Report**: Create a report for `QSYSPRT` with header (company name, job details, date, time, file group) and details (`company`, `target_customer`, `target_ship_to`, `product`, `container_type`, `description`, `new_record_flag`).
6. **Return Results**: Output `copied_records`, `report`, `history_records`, and `status`.

## Business Rules
- Copies records only if they don’t exist in the target `ARCUPR`.
- Logs all copy operations to `ARCUPHS`.
- Generates a held and saved report in the job’s output queue.
- Validates source and target customer/ship-to against `ARCUST` and `SHIPTO`.
- Supports `G` or `Z` file groups.

## Calculations
- None required; copies existing data without modification.

</xaiArtifact>

<xaiArtifact artifact_id="ae2bb703-8a6a-4889-a19e-781b90c42516" artifact_version_id="75a577e5-9184-4065-a5cf-748f95421df3" title="Customer_ShipTo_Address_Inquiry_Add.md" contentType="text/markdown">

# Function Requirement: Customer Ship-to Address Inquiry and Add

## Description
Retrieves customer ship-to address records for inquiry and adds new address records to the `CUADR` file.

## Inputs
- `company` (2 characters): Company number.
- `customer` (6 characters): Customer number.
- `ship_to` (3 characters): Ship-to number (optional for inquiry).
- `mode` (3 characters): Operation mode (`INQ` or `ADD`).
- `address` (structure, for `ADD`): Fields: `edi_code` (10 characters), `edi_yn` (1 character), `name` (30 characters), `address1` (30 characters), `address2` (30 characters), `city` (20 characters), `state` (2 characters), `zip` (10 characters), `country` (3 characters).

## Outputs
- `address_records` (array): List of address records with fields: `edi_code`, `edi_yn`, `name`, `address1`, `address2`, `city`, `state`, `zip`, `country`.
- `status` (string): Success or error message (e.g., "Name is blank").

## Process Steps
1. **Validate Inputs**: Confirm `company` exists in `BICONT`, `customer` in `ARCUST`, and `ship_to` in `SHIPTO` (if provided). Validate `mode` as `INQ` or `ADD`.
2. **Apply File Overrides**: Redirect to `ZCUADR` for `CUADRRD` (and potentially `GCUADR` for `G` group).
3. **Process Mode**:
   - **Inquiry Mode (`INQ`)**: Read `CUADRRD` using `company`, `customer`, and `ship_to` (if provided). Return matching records.
   - **Add Mode (`ADD`)**: Validate `address`:
     - Ensure `name` is non-blank.
     - Check for existing record in `CUADR` using `company`, `customer`, `ship_to`.
     - If no record exists, write to `CUADR` with `address` fields.
4. **Return Results**: Output `address_records` and `status`.

## Business Rules
- Supports `INQ` and `ADD` modes (update mode disabled).
- Prevents duplicate records (same `company`, `customer`, `ship_to`).
- Requires non-blank `name` for `ADD`.
- Validates inputs against `ARCUST` and `SHIPTO` (EDI code validation disabled).
- Supports `Z` file group (and potentially `G`).

## Calculations
- None required; retrieves or writes address data without modification.

</xaiArtifact>

---

### **Notes on Call Stack Integration**

- **OCL Role**: `BI890.ocl36.txt` orchestrates the workflow, calling these programs based on user selections or batch processes. It passes parameters like company, customer, ship-to, and file group to invoke the appropriate use case.
- **Work File Management**: `BI907AC` and `BI907C2` support `BI907` by clearing and creating `QTEMP/BI907W`, ensuring data isolation.
- **Interdependencies**:
  - `BI9078` (copy) and `BI8902R` (address add) may complement `BI8903` (ship-to copy, not provided) for managing ship-to data.
  - `GB730P` provides audit history for changes made by `BI907`, `BI9078`, or `BI8902R`.
  - `BB800E` provides inquiry for accessorial/marks data, which may relate to freight terms managed by `BI907`.
- **File Groups**: All programs support `G` and `Z` file groups, ensuring flexibility in data access.

Each function requirement document is designed to be programmatic, accepting inputs and producing outputs without interactive screen dependencies, while preserving the core business logic and calculations from the original RPG/CLP programs. If you need further refinement or additional details, let me know!