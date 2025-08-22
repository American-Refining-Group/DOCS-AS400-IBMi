The provided call stack consists of the following programs in the Bradford Order Entry/Invoices system:

1. **BI907P** (Main RPGLE program, assumed as the entry point based on context).
2. **BI907C** (CLP program, prepares temporary work file and calls BI907).
3. **BI907** (RPGLE program, handles customer/ship-to file maintenance and inquiry).
4. **BI9078** (RPGLE program, copies customer/ship-to product descriptions).
5. **GB730P** (RPGLE program, provides history inquiry for various files).
6. **BI907C2** (CLP program, clears temporary work file).

These programs collectively support a system for managing customer and ship-to product descriptions, freight terms, and related historical data. Below, I will identify the use cases implemented by this call stack and then provide a function requirement document for one of the primary use cases, reimagined as a non-interactive function that processes inputs programmatically.

---

### Use Cases Implemented by the Call Stack

Based on the analysis of the programs, the call stack implements the following use cases:

1. **Maintain Customer/Ship-To Product Descriptions and Freight Terms**:
   - **Description**: Allows users to add, update, or delete customer/ship-to-specific product descriptions and freight terms, including freight codes, container types, and related flags (e.g., separate freight, calculate freight).
   - **Programs Involved**: `BI907P`, `BI907C`, `BI907`, `BI907C2`.
   - **Details**:
     - `BI907P` (assumed) provides the main user interface for entering company, customer, ship-to, mode (`MNT` or `INQ`), and file group (`Z` or `G`).
     - `BI907C` prepares the temporary work file `BI907W` in `QTEMP` by clearing or creating it and calls `BI907`.
     - `BI907` uses a subfile-based interface to display and manage records in `arcupr`, validate inputs (e.g., freight codes, container types), and log changes to `arcuphs`.
     - `BI907C2` clears `BI907W` to reset the temporary file for new operations.
     - Supports modes: Maintenance (`MNT`) for editing and Inquiry (`INQ`) for viewing.
     - Validates freight codes (e.g., `C` requires `s1sfrt` and `s1cafr` to be `Y`, `N`, or blank; `A` requires `s1sfrt = 'Y'`; `CYY` indicates freight collect with a $100 service fee).
     - Provides options to prompt for valid codes (e.g., customer, ship-to, product, freight code, container type) and toggle between Add, Update, and All modes.

2. **Copy Customer/Ship-To Product Descriptions**:
   - **Description**: Copies product descriptions from one customer/ship-to pair to another within the same company, updating existing records or adding new ones, and generates a printed report.
   - **Programs Involved**: `BI907P`, `BI907C`, `BI9078`.
   - **Details**:
     - Initiated from `BI907P` (via F8 key in `BI907`) with source and target customer/ship-to parameters.
     - `BI907C` sets up `BI907W` in `QTEMP` for temporary processing.
     - `BI9078` reads source records from `arcup3`, updates or adds records to `arcupr`, logs changes to `arcuphs`, and prints a report to `qsysprt`.
     - The report includes details like company, customer, ship-to, product, container type, product description, and whether the record is new or updated.
     - Supports file groups `Z` and `G` for different data environments.

3. **View Historical Changes for Customer/Ship-To Data**:
   - **Description**: Displays historical changes for customer/ship-to data and other related files, allowing users to review past modifications (e.g., additions, updates, deletions).
   - **Programs Involved**: `BI907P`, `BI907C`, `BI907`, `GB730P`.
   - **Details**:
     - Initiated from `BI907` (via F9 key for history inquiry) with parameters for the file to query (e.g., `ARCUPR`) and key fields (e.g., company, customer, ship-to, product, container type).
     - `BI907C` ensures `BI907W` is available in `QTEMP`.
     - `GB730P` uses a subfile-based interface to display history records from the corresponding history file (e.g., `arcuphx` for `ARCUPR`).
     - Supports multiple files (e.g., `SHIPTO`, `CUADR`, `ARCUST`, `BICUAG`, `BBPRCE`, `BBORDH`, etc.) with specific headers and key fields.
     - Enforces file group flexibility (`Z` or `G`) and proprietary security restrictions.

4. **Reset Temporary Work File**:
   - **Description**: Clears the temporary work file `BI907W` to ensure no residual data affects subsequent operations.
   - **Programs Involved**: `BI907P`, `BI907C2`.
   - **Details**:
     - `BI907C2` is called from `BI907P` to execute the `CLRPFM` command on `BI907W` in `QTEMP`.
     - Ensures a clean slate for temporary data used by `BI907` or `BI9078`.

---

### Function Requirement Document: Maintain Customer/Ship-To Product Descriptions and Freight Terms

Below is a function requirement document for the primary use case, "Maintain Customer/Ship-To Product Descriptions and Freight Terms," reimagined as a non-interactive function that processes inputs programmatically rather than through a screen-based interface. The document focuses on business requirements, process steps, and calculations concisely.

<xaiArtifact artifact_id="f6ce07b7-0a09-4e8b-8579-2c25423ef9f8" artifact_version_id="345694b3-b4ef-477f-850f-5bdeef10d03b" title="CustomerShipToMaintenanceFunction.md" contentType="text/markdown">

# Function Requirement Document: Maintain Customer/Ship-To Product Descriptions and Freight Terms

## Purpose
To programmatically add, update, or delete customer/ship-to-specific product descriptions and freight terms in the Bradford Order Entry/Invoices system, ensuring data integrity and logging changes for audit purposes.

## Inputs
- **Company Number** (`cono`, 2-digit numeric): Identifies the company.
- **Customer Number** (`cust`, 6-digit numeric): Identifies the customer.
- **Ship-To Number** (`ship`, 3-digit numeric): Identifies the ship-to location.
- **File Group** (`fgrp`, 1-character): Specifies the data environment (`Z` or `G`).
- **Action** (`action`, string): Specifies the operation (`ADD`, `UPDATE`, `DELETE`).
- **Records** (array of records, each containing):
  - `product` (4-character): Product code.
  - `container_type` (1-character): Container type code.
  - `product_description` (20-character): Customer-specific product description.
  - `freight_code` (2-character): Freight code (e.g., `C`, `A`, `CYY`).
  - `separate_freight` (1-character): Separate freight flag (`Y`, `N`, or blank).
  - `calculate_freight` (1-character): Calculate freight flag (`Y`, `N`, or blank).
  - `gl_code` (10-character): General ledger code (optional for updates).
- **User ID** (`user`, 8-character): User performing the action for audit logging.
- **Timestamp** (`timestamp`, 14-digit numeric, format `CCYYMMDDHHMMSS`): Date and time of the operation.

## Outputs
- **Status** (string): `SUCCESS`, `ERROR`, or `PARTIAL_SUCCESS` (if some records fail).
- **Messages** (array of strings): Error or informational messages for each record processed.
- **Processed Records** (array of records): Returns the input records with updated status (e.g., added, updated, deleted, or failed).

## Process Steps
1. **Validate Inputs**:
   - Ensure `cono`, `cust`, `ship`, and `fgrp` are non-empty and valid.
   - Verify `fgrp` is `Z` or `G`.
   - For each record, validate:
     - `product` exists in `gsprod` (file group adjusted).
     - `container_type` exists in `gstabl` (table type `CNTRTY`).
     - `freight_code` exists in `gstabl` (table type `BBFRCD`).
     - `separate_freight` and `calculate_freight` are `Y`, `N`, or blank if `freight_code` is `C` or `CYY`.
     - `separate_freight` is `Y` if `freight_code` is `A`.
     - `product_description` is non-empty for `ADD` and `UPDATE`.

2. **Clear Temporary Work File**:
   - Clear `BI907W` in `QTEMP` to ensure no residual data.

3. **Apply File Overrides**:
   - Override `arcupr`, `arcuphs`, `gstabl`, `bicont`, `gsprod`, `arcust`, `shipto` to `g*` or `z*` files based on `fgrp`.

4. **Process Each Record**:
   - **For ADD**:
     - Check if a record exists in `arcupr` for `cono`, `cust`, `ship`, `product`, `container_type`.
     - If exists and not marked deleted, return error ("Record Already Exists").
     - If marked deleted, reactivate by updating `cpdel = 'A'`.
     - If not exists, create new record with input fields.
     - Write to `arcupr` and log to `arcuphs` with `user`, `timestamp`, and fields.
   - **For UPDATE**:
     - Check if record exists in `arcupr`.
     - If not exists, return error ("Record Not Found").
     - Update `product_description`, `freight_code`, `separate_freight`, `calculate_freight`, `gl_code`.
     - Write to `arcupr` and log to `arcuphs`.
   - **For DELETE**:
     - Check if record exists in `arcupr`.
     - If not exists, return error ("Record Not Found").
     - Set `cpdel = 'D'` in `arcupr` and log to `arcuphs`.

5. **Log History**:
   - For each successful operation, write a history record to `arcuphs` with:
     - `ahcono`, `ahcust`, `ahship`, `ahprod`, `ahcnty`, `ahcpds`, `ahfrcd`, `ahsfrt`, `ahcafr`, `ahglcd`, `ahdel`, `ahchd8` (date), `ahchtm` (time), `ahuser`.

6. **Return Results**:
   - Compile status and messages for each record.
   - Return overall status and processed records.

## Business Rules
- **Data Integrity**:
  - Product codes must exist in `gsprod`.
  - Container types and freight codes must exist in `gstabl` (`CNTRTY` and `BBFRCD`).
  - Customer and ship-to must exist in `arcust` and `shipto`.
- **Freight Code Rules**:
  - For `freight_code = 'C'` (freight collect) or `CYY` (freight collect with $100 service fee):
    - `separate_freight` and `calculate_freight` must be `Y`, `N`, or blank.
  - For `freight_code = 'A'` (non-Bradford location, e.g., Anchor):
    - `separate_freight` must be `Y`.
  - `CYY` indicates shipping arranged by ARG but billed to customer by carrier with a $100 fee.
- **Record Status**:
  - Records marked deleted (`cpdel = 'D'`) cannot be added unless reactivated.
  - New records are marked active (`cpdel = 'A'`).
- **History Logging**:
  - All changes (add, update, delete) are logged to `arcuphs` with user and timestamp.
- **File Group**:
  - `fgrp` (`Z` or `G`) determines the database file set (`g*` or `z*`).

## Calculations
- **Freight Service Fee** (for `freight_code = 'CYY'`):
  - A fixed $100 fee is applied to the customer for freight arranged by ARG but billed by the carrier.
  - This is not programmatically calculated but is a business rule enforced during validation.
- **Timestamp Formatting**:
  - Input `timestamp` (14-digit `CCYYMMDDHHMMSS`) is split into `ahchd8` (8-digit `CCYYMMDD`) and `ahchtm` (6-digit `HHMMSS`) for history logging.

## Error Handling
- Return specific error messages for:
  - Invalid `product`, `container_type`, or `freight_code`.
  - Non-existent customer or ship-to.
  - Invalid `separate_freight` or `calculate_freight` for specific freight codes.
  - Record already exists (for `ADD`) or not found (for `UPDATE`, `DELETE`).
- Log errors in the output messages array.

</xaiArtifact>

---

### Notes
- The function requirement document focuses on the primary use case of maintaining customer/ship-to data, as it is the core functionality of the call stack (`BI907P`, `BI907C`, `BI907`, `BI907C2`). The other use cases (copying records, viewing history) could also be converted to non-interactive functions but are secondary in scope.
- The document assumes a programmatic interface, eliminating screen-based interactions and focusing on batch processing of input records.
- The business rules and validations are derived from the `BI907` program, with freight code rules (`C`, `A`, `CYY`) explicitly documented, including the $100 service fee for `CYY`.
- The temporary work file (`BI907W`) is included as a step to maintain consistency with the original call stack, though its use in a non-interactive function may be optional depending on implementation.