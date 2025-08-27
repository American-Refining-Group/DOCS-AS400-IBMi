The call stack consists of the following programs, all part of the "Bradford Order Entry / Invoices" system for "Rack Price File Maintenance":

1. **BB955.ocl36.txt**: The OCL (Operation Control Language) program that initiates the process, invoking `BB955`.
2. **BB955.rpgle (parts 1 and 2)**: The main RPGLE program that handles interactive rack price file maintenance, including viewing, updating, adding, and deleting records.
3. **BB955AC.clp**: A CLP program called by `BB955` to clear or create the work file `BB955W` in `QTEMP`.
4. **GB730P.rpgle**: An RPGLE program called by `BB955` (via F9) for displaying historical data of various files, including `bbprcehx` for rack price history.

Below, I’ll identify the use cases implemented by this call stack, then provide a function requirement document for each use case, assuming they are non-interactive functions that process inputs programmatically.

### List of Use Cases

Based on the analysis of the provided programs, the call stack implements the following use cases:

1. **Maintain Rack Price Records** (`BB955.rpgle`):
   - **Description**: Allows for viewing, updating, adding, and deleting rack price records in the `bbprce` file, with support for up to 10 additional locations and validation against master files. Changes are staged in `BB955W` and logged to `bbprceh`.
   - **Components**: Main logic in `BB955` (`srsfl1`, `sf1rep`, `sf1prc`, `sf1upd`, `sf1f07`, `updadlloc`, `addadlloc`, `sf1cte`, `sf1ctenew`, `sf1lodall`), with `BB955AC` clearing/creating `BB955W`.
   - **Key Features**:
     - Filter records by company, location, product, container, unit of measure, product class, status, dates, inactive status, rack price flag, and division.
     - Update existing records or add new ones with a new effective date.
     - Support for additional locations (`loc` array).
     - Toggle display of deleted records and update permissions.
     - Validate inputs against master files (`bicont`, `inloc`, `gsprod`, `gscntr1`, `gstabl`).
     - Log changes to `bbprceh`.

2. **View Rack Price History** (`GB730P.rpgle`, called by `BB955`):
   - **Description**: Displays historical changes to rack price records (`bbprcehx`) for a specific company, location, product, container, unit of measure, date, and time.
   - **Components**: Invoked via F9 in `BB955`, using `GB730P`’s subfile processing and parameter handling.
   - **Key Features**:
     - Query historical data based on parameters passed from `BB955`.
     - Display history with formatted dates and user information.
     - Support for multiple file types (e.g., `BBPRCE`, `SHIPTO`), with specific handling for `BBPRCE`.

3. **Manage Work File for Rack Price Processing** (`BB955AC.clp`):
   - **Description**: Clears or creates the `BB955W` work file in `QTEMP` to stage rack price data before updating `bbprce`.
   - **Components**: Called by `BB955` (`sf1f07`) to ensure a clean work file.
   - **Key Features**:
     - Clear existing `BB955W` or create it from `DATA` (production) or `DATADEV` (development) based on the file group parameter.
     - Ensure session-specific data isolation in `QTEMP`.

### Function Requirement Documents

Each use case is transformed into a non-interactive function that processes inputs programmatically, avoiding screen-based interaction. The requirements focus on business needs, process steps, and calculations, presented concisely.

---

<xaiArtifact artifact_id="c7404748-f953-405c-a00a-4f5c5784ce30" artifact_version_id="a1911881-89eb-446b-bbde-9c885c9c20e1" title="MaintainRackPriceRecords.md" contentType="text/markdown">

# Function Requirement Document: MaintainRackPriceRecords

## Purpose
Programmatically maintain rack price records in the `bbprce` file, allowing updates, additions, or deletions based on provided inputs, with changes staged in `BB955W` and logged to `bbprceh`.

## Inputs
- **Company Number** (`cono`, 2 digits): Identifies the company.
- **Location** (`loc`, 3 chars): Primary location code.
- **Product** (`prod`, 4 chars): Product code.
- **Container** (`cntr`, 3 chars): Container code (000–999).
- **Unit of Measure** (`unms`, 3 chars): Unit of measure code.
- **Product Class** (`prcl`, 3 chars): Product class code.
- **Status** (`stat`, 3 chars): Filter status (ALL, CUR, FUT, CF).
- **From Date** (`fmdy`, 6 digits, MMDDYY): Start date filter.
- **To Date** (`tmdy`, 6 digits, MMDDYY): End date filter.
- **Inactive Status** (`inac`, 1 char): Blank (active), I (inactive, no orders), B (inactive, orders allowed).
- **No Rack Price** (`rkrq`, 1 char): Blank or N (no rack price).
- **Division** (`dvsn`, 1 char): Blank (all), R (refinery), L (blended lube).
- **Effective Date** (`emdy`, 6 digits, MMDDYY): Effective date for new records.
- **Additional Locations** (`loc_array`, array of 10 x 3 chars): Additional location codes.
- **Existing Quantities** (`qt`, array of 4 x 7 digits): Quantities for existing records.
- **Existing Prices** (`pr`, array of 4 x 9.4 packed): Prices for existing records.
- **New Quantities** (`n_qt`, array of 4 x 7 digits): Quantities for new records.
- **New Prices** (`n_pr`, array of 4 x 9.4 packed): Prices for new records.
- **New Minimum Quantity** (`n_minq`, 7 digits): Minimum quantity for new records.
- **New Inactive Status** (`n_inac`, 1 char): Inactive status for new records.
- **New Rack Price Flag** (`n_rkrq`, 1 char): Rack price flag for new records.
- **Action** (`action`, char): UPDATE, ADD, DELETE, RESTORE.
- **File Group** (`fgrp`, 1 char): G (production) or Z (development).

## Outputs
- **Status Code** (char): SUCCESS or ERROR.
- **Error Messages** (array of strings): List of validation errors, if any.
- **Updated Records** (array of records): Details of updated/added/deleted records.

## Process Steps
1. **Validate Inputs**:
   - Verify `cono` exists in `bicont`, retrieving company name.
   - Verify `loc` exists in `inloc`, retrieving location name.
   - Verify `prod` exists in `gsprod`, retrieving product description.
   - Verify `cntr` is 000–999 and exists in `gscntr1`, retrieving description.
   - Verify `unms` exists in `gstabl` (type `BIUNMS`), retrieving description.
   - Verify `prcl` (with `cono`) exists in `gstabl` (type `PRODCL`), retrieving description.
   - Validate `stat` as ALL, CUR, FUT, or CF.
   - Validate `fmdy` and `tmdy` using `GSDTEDIT`, converting to CYMD format.
   - Validate `inac` as blank, I, or B.
   - Validate `rkrq` as blank or N.
   - Validate `dvsn` as blank, R, or L.
   - Validate `loc_array` entries exist in `inloc` and are not equal to `loc`.
   - Validate `emdy` (if provided) using `GSDTEDIT` and ensure non-zero if `loc_array` is non-empty.
   - Validate `qt`, `pr`, `n_qt`, `n_pr`, `n_minq`, `n_inac`, `n_rkrq` for numeric/format consistency.
   - Return ERROR with specific messages (e.g., "Invalid Company", "Must Enter Effective Date") if any validation fails.

2. **Clear/Create Work File**:
   - Call `BB955AC` with `fgrp` to clear `BB955W` in `QTEMP` or create it from `DATA` (if `G`) or `DATADEV` (if `Z`).

3. **Build Work File**:
   - Filter `bbprce` records based on `cono`, `loc`, `prod`, `cntr`, `unms`, `prcl`, `stat`, `fmdy`, `tmdy`, `inac`, `rkrq`, `dvsn`.
   - Exclude inactive records in `gsctum`, `gsctwt`, `gsumcv` (treated as deleted per `JB05`).
   - Write matching records to `BB955W`.

4. **Process Action**:
   - **UPDATE**:
     - Chain to `BB955W` using `cono`, `prod`, `cntr`, `unms`, `loc`, `date`, `time`.
     - Update fields (`pr`, `qt`, `minq`, `rkrq`, `inac`) if different from existing values.
     - Log changes to `bbprceh` with user and timestamp (revision `JK01`).
     - Update `bbprce` and additional locations (`loc_array`).
   - **ADD**:
     - Validate `emdy` and non-empty fields (`n_pr`, `n_qt`, `n_minq`, `n_rkrq`, `n_inac`).
     - Add new records to `bbprce` for `cono`, `loc`, `prod`, `cntr`, `unms`, `emdy`, `time` (default 0001 if not provided).
     - Add records for `loc_array` entries.
     - Log changes to `bbprceh`.
   - **DELETE**:
     - Mark records in `bbprce` as deleted (`rkdel = 'D'`) for specified keys.
     - Log to `bbprceh`.
   - **RESTORE**:
     - Restore deleted records (`rkdel = 'A'`) in `bbprce`.
     - Log to `bbprceh`.

5. **Return Results**:
   - Return SUCCESS with updated records or ERROR with error messages.

## Business Rules
- All input fields must be validated against master files (`bicont`, `inloc`, `gsprod`, `gscntr1`, `gstabl`).
- Effective date (`emdy`) is mandatory for ADD with additional locations.
- Inactive records in `gsctum`, `gsctwt`, `gsumcv` are treated as deleted (`JB05`).
- Changes are logged to `bbprceh` with user and timestamp (`JK01`).
- Prices are in U.S. dollars.
- Work file (`BB955W`) is session-specific in `QTEMP` (`jk04`).
- Support two file groups: `G` (production, `DATA`) and `Z` (development, `DATADEV`).

## Calculations
- **Date Conversion**: Use `GSDTEDIT` to convert `fmdy`, `tmdy`, `emdy` (MMDDYY) to CYMD format for storage.
- **Timestamp**: Use system time (`ps#hms`) for new records if not provided, defaulting to 0001 for ADD.
- **Quantity/Price Arrays**: Ensure `qt` and `n_qt` are 7 digits, `pr` and `n_pr` are 9.4 packed, and validate for ascending order or non-zero constraints (per `BB955`’s `com` array).

</xaiArtifact>

---

<xaiArtifact artifact_id="da92b046-464e-40f6-8edc-e9a4475853ea" artifact_version_id="01039661-09d2-46eb-a5f1-20160eef4180" title="ViewRackPriceHistory.md" contentType="text/markdown">

# Function Requirement Document: ViewRackPriceHistory

## Purpose
Retrieve and return historical changes to rack price records from `bbprcehx` based on provided keys.

## Inputs
- **File Name** (`file`, 10 chars): Must be `BBPRCE`.
- **File Group** (`fgrp`, 1 char): G (production) or Z (development).
- **Company Number** (`cono`, 2 digits): Company identifier.
- **Location** (`loc`, 3 chars): Location code.
- **Product** (`prod`, 4 chars): Product code.
- **Container** (`cntr`, 3 chars): Container code.
- **Unit of Measure** (`unms`, 3 chars): Unit of measure code.
- **Date** (`date`, 8 digits, CYMD): Specific date of change (optional).
- **Time** (`time`, 4 digits): Specific time of change (optional).

## Outputs
- **Status Code** (char): SUCCESS or ERROR.
- **Error Messages** (array of strings): List of validation errors, if any.
- **History Records** (array of records): Historical data including prices, quantities, user, and timestamp.

## Process Steps
1. **Validate Inputs**:
   - Ensure `file = 'BBPRCE'`.
   - Validate `fgrp` as G or Z.
   - Verify `cono` exists in `bicont`.
   - Verify `loc` exists in `inloc`.
   - Verify `prod` exists in `gsprod`.
   - Verify `cntr` exists in `gscntr1`.
   - Verify `unms` exists in `gstabl` (type `BIUNMS`).
   - Validate `date` (if provided) using `GSDTEDIT`.
   - Validate `time` (if provided) as 4 digits.
   - Return ERROR with messages (e.g., "Cannot Enter both Sys And Program Id") if validation fails.

2. **Apply File Overrides**:
   - If `fgrp = 'G'`, override `bbprcehx` to `gbbprcehx`.
   - If `fgrp = 'Z'`, override `bbprcehx` to `zbbprcehx`.

3. **Retrieve History**:
   - Query `bbprcehx` using `cono`, `loc`, `prod`, `cntr`, `unms`, `date`, `time` (if provided).
   - Filter out inactive records in `gsctum`, `gsctwt`, `gsumcv` (per `JB05`).
   - Retrieve associated descriptions from `bicont`, `inloc`, `gsprod`, `gscntr1`, `gstabl`.

4. **Format Output**:
   - Convert dates to MMDDYY format using `GSDTEDIT`.
   - Include user (`USER##8`) and timestamp for each change.
   - Return SUCCESS with history records.

## Business Rules
- Only `BBPRCE` history is retrieved (other files like `SHIPTO` are supported but irrelevant here).
- Input keys must match `bbprcehx` records.
- Inactive records in `gsctum`, `gsctwt`, `gsumcv` are excluded.
- File group determines library (`gbbprcehx` or `zbbprcehx`).
- Historical data includes audit fields (user, timestamp).

## Calculations
- **Date Conversion**: Use `GSDTEDIT` to convert `date` (CYMD) to MMDDYY for output.

</xaiArtifact>

---

<xaiArtifact artifact_id="cfe9a13c-1690-4790-9627-b760195a9757" artifact_version_id="20caa7c9-d862-4dfb-8bc3-8c19ece317fc" title="ManageWorkFile.md" contentType="text/markdown">

# Function Requirement Document: ManageWorkFile

## Purpose
Clear or create the `BB955W` work file in `QTEMP` to stage rack price data for processing.

## Inputs
- **File Group** (`fgrp`, 1 char): G (production) or Z (development).

## Outputs
- **Status Code** (char): SUCCESS or ERROR.
- **Error Messages** (array of strings): List of errors, if any.

## Process Steps
1. **Validate Input**:
   - Ensure `fgrp` is G or Z.
   - Return ERROR if invalid.

2. **Clear Work File**:
   - Attempt to clear `BB955W` in `QTEMP` using `CLRPFM`.

3. **Create Work File (if needed)**:
   - If `BB955W` not found:
     - If `fgrp = 'G'`, create `BB955W` in `QTEMP` from `DATA/BB955W` with no constraints or triggers.
     - If `fgrp = 'Z'`, create from `DATADEV/BB955W`.
   - Ignore errors for existing objects or constraints.

4. **Return Results**:
   - Return SUCCESS or ERROR with messages.

## Business Rules
- `BB955W` is session-specific in `QTEMP`.
- Source library is `DATA` (production) for `G`, `DATADEV` (development) for `Z` (`JK01`).
- Errors (e.g., file not found) are handled gracefully.

## Calculations
- None required.

</xaiArtifact>

---

### Additional Notes

- **Assumptions for Non-Interactive Functions**:
  - The functions replace `BB955`’s interactive subfile processing (`srsfl1`, `sf1lodall`) with programmatic input validation and processing.
  - User interaction (e.g., F6, F8, F23) is replaced by explicit `action` parameters (UPDATE, ADD, DELETE, RESTORE).
  - Screen-based error messages (`com` array) are returned as output arrays.
- **Missing Code**: The truncated `BB955` and `GB730P` code limits details on subroutines like `bldwrkf`, `writehist`, and `GB730P`’s main logic. The use cases assume standard RPG behavior for work file population and history retrieval.
- **Integration**:
  - `MaintainRackPriceRecords` calls `ManageWorkFile` to initialize `BB955W`.
  - `ViewRackPriceHistory` is invoked separately but uses the same file group and validation logic.
- **Business Context**: The system manages rack prices for products in U.S. dollars, with support for multiple locations, audit logging, and environment-specific data (production vs. development).

If you have additional code or need refinements to these requirements (e.g., specific calculations or validation rules), please provide them. I can also use DeepSearch to clarify AS/400 conventions or expand on specific aspects. Let me know how to proceed!