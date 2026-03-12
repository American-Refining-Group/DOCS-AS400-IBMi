The call stack provided includes the following RPGLE programs called from the OCL procedure `BB943INQ.ocl36.txt`: `BB943`, `BB943V`, `BB9433`, `BB944`, `GSDTCLC1`, and `GB730P`. These programs collectively implement a customer sales agreement maintenance system, handling creation, validation, copying/updating, deletion/reactivation, and history inquiry for sales agreements in an IBM System/36 (or AS/400 compatibility mode) environment. Below, I’ll identify the use cases implemented by this call stack and then provide a concise function requirement document for each use case, assuming they are large functions that process inputs programmatically rather than interactively via screens.

---

### List of Use Cases

Based on the analysis of the provided programs (`BB943`, `BB943V`, `BB9433`, `BB944`, `GSDTCLC1`, `GB730P`), the following distinct use cases are implemented within the call stack:

1. **Create or Update Sales Agreement (BB943, BB943V, BB9433)**:
   - **Description**: Allows creation of new sales agreements or updating existing ones, including copying agreements to other locations, ship-tos, or customer-ship-to combinations.
   - **Programs Involved**: 
     - `BB943`: Handles user interaction via display file `bb943d`, collects input, and calls `BB943V` and `BB9433`.
     - `BB943V`: Validates input data (customer, container, products, dates).
     - `BB9433`: Processes the creation or update, writes to `bicuag`, expires original records, and logs to `bicuagh`.
     - `GSDTCLC1`: Performs date calculations (e.g., adjusting expiration dates).

2. **Delete or Reactivate Sales Agreement (BB944)**:
   - **Description**: Marks a sales agreement as deleted or reactivates a previously deleted agreement, logging changes to the history file.
   - **Programs Involved**: 
     - `BB944`: Handles user interaction via display file `bb944d`, updates `bicuag` (sets/clears `badel`), and writes to `bicuagh`.
     - `GSDTCLC1`: May be used for date validation or calculations (if called in truncated logic).

3. **View Sales Agreement History (GB730P)**:
   - **Description**: Displays historical data for sales agreements and related entities (e.g., pricing, freight, orders) for inquiry purposes.
   - **Programs Involved**: 
     - `GB730P`: Retrieves and displays historical records from multiple files (e.g., `bicuaghx`, `bbordhsx`) via display file `gb730pd` with subfile `SFL1`.
     - `GSDTCLC1`: May be used for date formatting or calculations (if called in truncated logic).

These three use cases cover the core functionalities of the customer sales agreement maintenance system: creating/updating agreements, deleting/reactivating agreements, and viewing historical changes. Each use case is treated as a large function that accepts inputs programmatically and processes them without screen interaction.

---

### Function Requirement Documents

Below are the function requirement documents for each use case, presented concisely with business requirements, process steps, and necessary calculations. Each function assumes programmatic input and output, bypassing interactive screen handling.

#### Use Case 1: Create or Update Sales Agreement

<xaiArtifact artifact_id="7565e6ab-97f9-4deb-bc39-41a0650d1ec2" artifact_version_id="f99503b5-7e85-46da-8d04-8485fc130ed6" title="Create_Update_Sales_Agreement_Requirements.md" contentType="text/markdown">

# Function Requirement: Create or Update Sales Agreement

## Purpose
Create a new customer sales agreement or update an existing one, including copying to other locations, ship-tos, or customer-ship-to combinations, expiring original records if needed, and logging changes to history.

## Inputs
- **Company Number** (`cono`, 2.0): Identifies the company.
- **Sequence Number** (`seqn`, numeric): Unique identifier for the agreement (0 for new).
- **Customer Number** (`cust`, 6.0): Customer ID.
- **Location** (`loc`, 3A): Primary location code.
- **Container Code** (`cntr`, 3A): Container identifier.
- **Unit of Measure** (`unms`, 3A): Unit of measure for the agreement.
- **All Ship-tos Flag** (`alsh`, 1A): `'Y'` to apply to all ship-tos, `'N'` otherwise.
- **Ship-to Number** (`ship`, 3.0): Specific ship-to location.
- **Product Codes** (`pr01`–`pr10`, 4A each): Up to 10 product codes.
- **Bill-to Purchase Order** (`pord`, 15A): Purchase order number.
- **Price** (`prce`, 9.4 packed): Agreement price.
- **Off-Price** (`offp`, 9.4 packed): Discounted price.
- **Freight Code** (`frcd`, 1A): Freight terms.
- **Delivery Code** (`delv`, 3A): Delivery method.
- **Prepaid Flag** (`ppd`, 1A): `'P'` for prepaid, blank otherwise.
- **Pricing Type** (`prim`, 1A): `'I'` (item) or `'M'` (miscellaneous).
- **Minimum Quantity** (`mnqy`, numeric): Minimum order quantity.
- **Maximum Quantity** (`mxqy`, numeric): Maximum order quantity.
- **Contract Number** (`cntn`, numeric): Contract identifier.
- **PO/Order Code** (`poor`, 15A): Purchase order or order code.
- **Start Date** (`std8`, 8.0, CCYYMMDD): Agreement start date.
- **Start Time** (`sttm`, 6.0, HHMMSS): Agreement start time.
- **End Date** (`end8`, 8.0, CCYYMMDD): Agreement end date (0 if none).
- **End Time** (`entm`, 6.0, HHMMSS): Agreement end time.
- **Add/Update Flag** (`addupd`, 1A): `'A'` for add, `'U'` for update.
- **Expire Flag** (`expr`, 1A): `'Y'` to expire original record, `'N'` otherwise.
- **Other Locations** (`locs`, 30A): Array of up to 10 location codes (3A each).
- **Other Ship-tos** (`shp`, 30A): Array of up to 10 ship-to codes (3A each).
- **Other Customer-Ship-tos** (`cshp`, 36A): Array of up to 4 customer/ship-to pairs (9A each: 6A customer + 3A ship-to).
- **Original Key Fields** (`o@cono`, `o@cust`, `o@loc`, `o@cntr`, `o@unms`, `o@ship`, `o@pr01`–`o@pr10`, `o@pord`, `o@mnqy`, `o@mxqy`, `o@std8`, `o@sttm`): Key for expiring record.
- **File Group** (`fgrp`, 1A): `'G'` or `'Z'` for file library selection.

## Outputs
- **Return Flag** (`flag`, 1A): `'1'` for success, `'0'` for failure.
- **Error Messages** (array): List of validation errors, if any.

## Process Steps
1. **Validate Inputs**:
   - Verify `cono` exists in `bicont`.
   - Validate `cust` in `arcusz`, `loc` in `cuadrx`, `ship` in `shipto`, `cntr` in `gscntr1`, `unms` in `gsctum`, `pr01`–`pr10` in `gsprod`, `frcd` in `gstabl`, `delv` in `gstabl`.
   - Ensure `std8` and `end8` (if non-zero) are valid dates using `GSDTCLC1`.
   - Check for duplicate agreements in `bicuag` (same key fields and `std8`, unless `frcd` differs).
   - Validate `locs`, `shp`, `cshp` arrays for non-blank entries and valid codes.
2. **Process Primary Agreement**:
   - If `addupd = 'A'`, create new record in `bicuag` with input fields (`bacono`, `bacust`, `baloc`, `bacntr`, `baunms`, `baship`, `bapr01`–`bapr10`, `bapord`, `baprce`, `baoffp`, `bafrcd`, `badelv`, `bappd`, `baprim`, `bamnqy`, `bamxqy`, `bacntn`, `bapoor`, `bastd8`, `basttm`, `baend8`, `baentm`).
   - If `addupd = 'U'`, update existing record in `bicuag` using `cono` and `seqn`.
3. **Copy to Other Locations/Ship-tos/Customer-Ship-tos**:
   - For each non-blank `locs` entry, create/update `bicuag` record with `baloc = locs[i]`.
   - For each non-blank `shp` entry, create/update `bicuag` record with `baship = shp[i]`.
   - For each non-blank `cshp` entry, create/update `bicuag` record with `bacust = cshp[i].cust`, `baship = cshp[i].ship`.
4. **Expire Original Record**:
   - If `expr = 'Y'`, locate original record in `bicuag` using `o@cono`, `o@cust`, `o@loc`, `o@cntr`, `o@unms`, `o@ship`, `o@pr01`–`o@pr10`, `o@pord`, `o@mnqy`, `o@mxqy`, `o@std8`, `o@sttm`.
   - If original `baend8 > std8` or `baend8 = 0`, set `baend8 = std8 - 1 minute` (using `GSDTCLC1`) and update `bicuag`.
   - Write expired record to `bicuagh` with audit fields (`hhchd8`, `hhchtm`, `hhuser`).
5. **Log to History**:
   - For each new/updated record, write to `bicuagh` with current date (`hhchd8`), time (`hhchtm`), and user (`hhuser`).
6. **Return Status**:
   - Set `flag = '1'` on success, `'0'` on validation failure with error messages.

## Business Rules
- **Validation**: All codes must exist in respective files; dates must be valid; no duplicate agreements unless freight codes differ.
- **Add vs. Update**: `'A'` creates new records with new `seqn`; `'U'` updates existing records by `cono` and `seqn`.
- **Copying**: Agreements can be copied to multiple locations, ship-tos, or customer-ship-to pairs, creating separate `bicuag` records.
- **Expiration**: Original record expires only if its end date is greater than the new start date or is zero; expiration sets end date to one minute before new start date.
- **History Logging**: All created/updated/expired records are logged to `bicuagh` with audit fields.
- **File Groups**: Use `G` or `Z` libraries based on `fgrp`.

## Calculations
- **Date Validation**: Use `GSDTCLC1` to validate `std8` and `end8` (YYYYMMDD format).
- **Expiration Date**: If expiring, calculate `baend8 = std8 - 1 minute` using `GSDTCLC1` with `DifFmt = 'D'`, `Diff = -1`.

</xaiArtifact>

#### Use Case 2: Delete or Reactivate Sales Agreement

<xaiArtifact artifact_id="08355b46-f7f7-409a-b94b-ae61d65a77c0" artifact_version_id="71ec674e-6714-4463-a23a-8a7d4e655dc7" title="Delete_Reactivate_Sales_Agreement_Requirements.md" contentType="text/markdown">

# Function Requirement: Delete or Reactivate Sales Agreement

## Purpose
Mark a customer sales agreement as deleted or reactivate a previously deleted agreement, logging changes to the history file.

## Inputs
- **Company Number** (`cono`, 2.0): Identifies the company.
- **Sequence Number** (`seqn`, numeric): Unique identifier for the agreement.
- **Action** (`action`, 1A): `'D'` for delete, `'R'` for reactivate.
- **File Group** (`fgrp`, 1A): `'G'` or `'Z'` for file library selection.

## Outputs
- **Return Flag** (`flag`, 1A): `'1'` for success, `'0'` for failure.
- **Error Messages** (array): List of validation errors, if any.

## Process Steps
1. **Validate Inputs**:
   - Verify `cono` exists in `bicont`.
   - Verify `seqn` exists in `bicuag` using key (`cono`, `seqn`).
   - Validate `action` as `'D'` or `'R'`.
2. **Retrieve Record**:
   - Chain to `bicuag` with `cono` and `seqn` to retrieve the agreement.
3. **Process Action**:
   - If `action = 'D'` and `badel ≠ 'Y'`, set `badel = 'Y'` to mark as deleted.
   - If `action = 'R'` and `badel = 'Y'`, set `badel = ' '` to reactivate.
4. **Log to History**:
   - Write updated record to `bicuagh` with audit fields (`hhchd8`, `hhchtm`, `hhuser`).
5. **Update Record**:
   - Update `bicuag` with modified `badel`.
6. **Return Status**:
   - Set `flag = '1'` on success, `'0'` on validation failure with error messages.

## Business Rules
- **Delete**: Only active records (`badel ≠ 'Y'`) can be deleted; sets `badel = 'Y'`.
- **Reactivate**: Only deleted records (`badel = 'Y'`) can be reactivated; clears `badel`.
- **History Logging**: All changes are logged to `bicuagh` with current date, time, and user.
- **Validation**: Agreement must exist; `action` must be valid.
- **File Groups**: Use `G` or `Z` libraries based on `fgrp`.

## Calculations
- None required (no date calculations; audit fields use current date/time).

</xaiArtifact>

#### Use Case 3: View Sales Agreement History

<xaiArtifact artifact_id="858b40a0-b97a-41c6-8902-fa103e2e3171" artifact_version_id="389b19d1-a5b6-4eb7-bdce-bf88a705cb36" title="View_Sales_Agreement_History_Requirements.md" contentType="text/markdown">

# Function Requirement: View Sales Agreement History

## Purpose
Retrieve and return historical data for customer sales agreements and related entities (e.g., pricing, freight, orders) based on selection criteria.

## Inputs
- **Company Number** (`cono`, 2.0): Identifies the company (optional, filters records).
- **Customer Number** (`cust`, 6.0): Customer ID (optional, per `jk01`).
- **File Type** (`filetype`, string): Specifies file to query (e.g., `bicuaghx`, `bbordhsx`, `bbcfsdhx`).
- **Date Range** (`start_date`, `end_date`, 8.0, CCYYMMDD): Filters records by change date (optional).
- **File Group** (`fgrp`, 1A): `'G'` or `'Z'` for file library selection.

## Outputs
- **Historical Records** (array): List of records with fields (e.g., `hhcono`, `hhcust`, `hhloc`, `hhprce`, `hhpoor`, `hhchd8`, `hhuser` for `bicuaghx`).
- **Return Flag** (`flag`, 1A): `'1'` for success, `'0'` for failure.
- **Error Messages** (array): List of validation errors, if any.

## Process Steps
1. **Validate Inputs**:
   - If `cono` provided, verify it exists in `glcont`.
   - If `cust` provided, verify it exists in `arcusz`.
   - Validate `filetype` against supported files (e.g., `bicuaghx`, `bbordhsx`).
   - Validate `start_date` and `end_date` using `GSDTCLC1` (if provided).
2. **Retrieve Records**:
   - Based on `filetype`, read records from the corresponding history file (e.g., `bicuaghx` for sales agreements).
   - Apply filters: `cono`, `cust`, `hhchd8` within `start_date` and `end_date`.
3. **Format Output**:
   - Map fields to output structure (e.g., for `bicuaghx`: `hhcono`, `hhcust`, `hhloc`, `hhcntr`, `hhpr01`–`hhpr10`, `hhprce`, `hhoffp`, `hhpoor`, `hhchd8`, `hhchtm`, `hhuser`).
   - Convert dates to display format (CCYYMMDD to MMDDYY) using `GSDTCLC1`.
4. **Return Results**:
   - Return array of records and set `flag = '1'` on success, `'0'` on validation failure with error messages.

## Business Rules
- **File Selection**: Supports multiple history files (e.g., sales agreements, orders, freight).
- **Filtering**: Allows filtering by company, customer, and date range; all optional.
- **Validation**: Ensures valid company, customer, and dates; `filetype` must be supported.
- **Field Support**: Includes expanded fields (`BAPRCE`, `BAOFFP`, `BFTSEQ`, `BAPOOR`, `BKFSPC`).
- **File Groups**: Uses `G` or `Z` libraries based on `fgrp`.
- **Error Handling**: Returns error if system and program IDs are both provided.

## Calculations
- **Date Conversion**: Uses `GSDTCLC1` to convert `hhchd8` (CCYYMMDD) to MMDDYY for output.
- **Date Validation**: Uses `GSDTCLC1` to validate `start_date` and `end_date`.

</xaiArtifact>

---

### Notes
- **Assumptions**: Due to the truncation of `BB943`, `BB944`, and `GB730P`, some process steps (e.g., main loop, subfile handling) were inferred based on context and patterns from `BB9433`, `BB944`, and related files. The function requirements assume programmatic input/output, removing interactive screen logic from `BB943`, `BB944`, and `GB730P`.
- **Integration**: The programs rely on `GSDTCLC1` for date calculations and `QCMDEXC`/`QMHRMVPM` for system operations (file overrides, message handling).
- **Completeness**: If you can provide the full source code for `BB943`, `BB944`, or `GB730P`, or additional files (e.g., display files `bb943d`, `bb944d`, `gb730pd`), I can refine the process steps and business rules further.
- **Artifact IDs**: Each use case has a unique UUID (`artifact_id`) as required, ensuring no overlap with historical artifacts.

Let me know if you need further clarification, additional details, or modifications to the function requirements!