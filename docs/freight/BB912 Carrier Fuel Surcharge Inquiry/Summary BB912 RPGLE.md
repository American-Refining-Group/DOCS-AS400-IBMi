The call stack provided consists of four RPG programs within the Brandford Order Entry/Invoices system: `BB912`, `BB910`, `GB730P`, and `MGSTABL`. These programs collectively manage carrier fuel surcharge data, national diesel fuel index data, historical inquiry, and table value lookups, primarily for region codes. Below is a comprehensive list of use cases implemented by this call stack, followed by a function requirement document for each use case, reimagined as large functions that process inputs programmatically rather than through screen interactions.

### List of Use Cases

Based on the analysis of the programs `BB912`, `BB910`, `GB730P`, and `MGSTABL`, the following use cases are implemented:

1. **Carrier Fuel Surcharge Maintenance** (`BB912`):
   - Allows users to create, update, copy, delete, or inquire about carrier fuel surcharge header and detail records, including validation of company, carrier, and region codes, and effective dates.
   - Supports maintenance (`MNT`) and inquiry (`INQ`) modes.
   - Manages header records (company, carrier, effective date) and detail records (price ranges and surcharge percentages).

2. **National Diesel Fuel Index Maintenance** (`BB910`, called from `BB912` via F07):
   - Enables users to maintain or inquire about national diesel fuel index records, including adding, updating, or deleting price data for specific regions, effective dates, and times.
   - Supports maintenance (`MNT`) and inquiry (`INQ`) modes, with region code validation and history logging.

3. **Historical Inquiry of File Changes** (`GB730P`, called from `BB912` and `BB910` via F09):
   - Provides a read-only view of historical changes for multiple files, including `BBCFSH` (fuel surcharge headers), `BBCFSD` (fuel surcharge details), and `BBNDFI` (diesel fuel index).
   - Displays audit trails with change dates, times, users, and modified fields.

4. **Table Value Lookup for Region Codes** (`MGSTABL`, called from `BB912` via F04 for `S1REGN`):
   - Facilitates the selection of valid region codes from the `gstabl` table for use in fuel surcharge maintenance.
   - Supports filtering by table type and search string, returning the selected code to the calling program.

### Function Requirement Documents

Each use case is reimagined as a programmatic function that processes inputs directly, eliminating screen interactions. The documents below outline the business requirements, process steps, and calculations concisely, focusing on business logic and necessary computations.

---

#### Function Requirement Document: Carrier Fuel Surcharge Maintenance

**Function Name**: `MaintainCarrierFuelSurcharge`

**Purpose**: Programmatically create, update, copy, delete, or retrieve carrier fuel surcharge records for a specified company, carrier, and effective date, including header and detail records.

**Inputs**:
- `company_code`: 2-digit numeric company code (default: 10 if blank/zero).
- `carrier_code`: 6-character carrier ID.
- `effective_date`: 8-digit numeric date (YYYYMMDD).
- `file_group`: 1-character group ('G' or 'Z') for file library selection.
- `mode`: 3-character mode ('MNT' for maintenance, 'INQ' for inquiry).
- `operation`: String ('CREATE', 'UPDATE', 'COPY', 'DELETE', 'RETRIEVE').
- `header_data`: Structure with region code (3-digit numeric).
- `detail_data`: Array of structures with:
  - `from_price`: 9.4 packed decimal.
  - `to_price`: 9.4 packed decimal.
  - `fuel_surcharge_percentage`: 5.2 packed decimal (updated per jb07/jb08).
- `copy_effective_date`: 8-digit numeric date (for COPY operation).

**Outputs**:
- `status`: String ('SUCCESS', 'ERROR').
- `error_message`: String (if applicable).
- `records`: Array of retrieved header and detail records (for RETRIEVE operation).

**Process Steps**:
1. **Validate Inputs**:
   - Verify `company_code` exists in `bicont`. Return error `ERR0021` if invalid.
   - Validate `carrier_code` against `bbcaid`. Return error `ERR0012` if invalid.
   - Validate `effective_date` and `copy_effective_date` (if provided) using date validation logic (YYYYMMDD format). Return error `ERR0020` if invalid.
   - Verify `mode` is 'MNT' or 'INQ'. Return error if invalid.
   - Ensure `operation` is valid ('CREATE', 'UPDATE', 'COPY', 'DELETE', 'RETRIEVE').
   - Validate `header_data.region_code` against `gstabl`. Return error `ERR0010` if invalid.
   - For detail records, ensure `from_price`, `to_price`, and `fuel_surcharge_percentage` are either all provided or all blank, and validate price ranges (non-overlapping, `from_price <= to_price`). Return error if invalid.

2. **Apply File Overrides**:
   - Use `file_group` ('G' or 'Z') to select the appropriate library for files (`bicont`, `gstabl`, `bbcaid`, `bbcfsh`, `bbcfsd`, `bbcfshh`, `bbcfsdh`).

3. **Process Operation**:
   - **CREATE** (mode: 'MNT'):
     - Write new header record to `bbcfsh` with `company_code`, `carrier_code`, `effective_date`, and `header_data.region_code`.
     - Write detail records to `bbcfsd` with price ranges and surcharge percentages.
     - Log changes to history files (`bbcfshh`, `bbcfsdh`) with timestamp and user.
   - **UPDATE** (mode: 'MNT'):
     - Locate header record in `bbcfsh` using `company_code`, `carrier_code`, `effective_date`.
     - Update region code if changed, and log to `bbcfshh`.
     - Update or add detail records in `bbcfsd`, ensuring non-overlapping price ranges, and log to `bbcfsdh`.
   - **COPY** (mode: 'MNT'):
     - Copy header record from `effective_date` to `copy_effective_date` in `bbcfsh`.
     - Copy associated detail records from `bbcfsd` to the new effective date.
     - Log changes to `bbcfshh` and `bbcfsdh`.
   - **DELETE** (mode: 'MNT'):
     - Delete header record from `bbcfsh` and associated detail records from `bbcfsd`.
     - Log deletions to `bbcfshh` and `bbcfsdh` with delete flag 'D'.
   - **RETRIEVE** (mode: 'MNT' or 'INQ'):
     - Retrieve header record from `bbcfsh` and detail records from `bbcfsd` for the specified `company_code`, `carrier_code`, and `effective_date`.
     - Return records with region descriptions from `gstabl` and occurrence counts.

4. **Return Results**:
   - Set `status` to 'SUCCESS' if the operation completes without errors, else 'ERROR' with `error_message`.
   - For RETRIEVE, populate `records` with header and detail data.

**Business Rules**:
- Only valid company, carrier, and region codes are allowed.
- Effective dates must be in YYYYMMDD format and validated.
- In 'INQ' mode, only RETRIEVE operations are permitted; others return an error.
- Price ranges in detail records must not overlap and must satisfy `from_price <= to_price`.
- All changes (create, update, delete) are logged to history files with timestamps and user IDs.
- COPY operation creates identical records with a new effective date, preserving data integrity.

**Calculations**:
- **Occurrence Count**: For RETRIEVE, count the number of detail records for the header to populate `s1occr`.
- **Price Range Validation**: Ensure `from_price <= to_price` and check for overlaps by comparing new ranges against existing `bbcfsd` records.

<xaiArtifact artifact_id="10d88044-7344-4e6a-b05b-12e3f45d2f5a" artifact_version_id="f134f7a6-5aab-4ed6-82be-c2b408350bd8" title="CarrierFuelSurchargeMaintenance.md" contentType="text/markdown">

# Function Requirement: MaintainCarrierFuelSurcharge

## Purpose
Programmatically manage carrier fuel surcharge records (create, update, copy, delete, retrieve) for a specified company, carrier, and effective date.

## Inputs
- `company_code`: 2-digit numeric (default: 10 if blank/zero).
- `carrier_code`: 6-character carrier ID.
- `effective_date`: 8-digit YYYYMMDD.
- `file_group`: 'G' or 'Z' for library selection.
- `mode`: 'MNT' (maintenance) or 'INQ' (inquiry).
- `operation`: 'CREATE', 'UPDATE', 'COPY', 'DELETE', 'RETRIEVE'.
- `header_data`: Structure with `region_code` (3-digit numeric).
- `detail_data`: Array of `{from_price: 9.4, to_price: 9.4, fuel_surcharge_percentage: 5.2}`.
- `copy_effective_date`: 8-digit YYYYMMDD (for COPY).

## Outputs
- `status`: 'SUCCESS' or 'ERROR'.
- `error_message`: String (if applicable).
- `records`: Array of header and detail records (for RETRIEVE).

## Process Steps
1. **Validate Inputs**:
   - Check26: Check `company_code` (`bicont`), `carrier_code` (`bbcaid`), `region_code` (`gstabl`), `effective_date`, `mode`, `operation`.
   - Detail records: Ensure `from_price <= to_price`, non-overlapping ranges.
2. **File Overrides**: Select library ('G' or 'Z') for `bicont`, `gstabl`, `bbcaid`, `bbcfsh`, `bbcfsd`, `bbcfshh`, `bbcfsdh`.
3. **Process Operation**:
   - **CREATE** ('MNT'): Write header (`bbcfsh`), details (`bbcfsd`), log to history (`bbcfshh`, `bbcfsdh`).
   - **UPDATE** ('MNT'): Update header, details, log changes.
   - **COPY** ('MNT'): Copy header/details to new `copy_effective_date`, log changes.
   - **DELETE** ('MNT'): Delete header/details, log with 'D' flag.
   - **RETRIEVE** ('MNT'/'INQ'): Fetch header/details with region descriptions, occurrence counts.
4. **Return Results**: Set `status`, `error_message`, and `records` (if RETRIEVE).

## Business Rules
- Valid company, carrier, region codes required.
- Effective dates in YYYYMMDD, validated.
- 'INQ' mode restricts to RETRIEVE.
- Non-overlapping price ranges; `from_price <= to_price`.
- Log all changes with timestamps, user IDs.
- COPY preserves data with new effective date.

## Calculations
- **Occurrence Count**: Count detail records for header (`s1occr`).
- **Price Range Validation**: Ensure `from_price <= to_price`, no overlaps with existing ranges.

</xaiArtifact>

---

#### Function Requirement Document: National Diesel Fuel Index Maintenance

**Function Name**: `MaintainNationalDieselFuelIndex`

**Purpose**: Programmatically maintain or retrieve national diesel fuel index records for a specified company, effective date, time, and region, including price updates and history logging.

**Inputs**:
- `company_code`: 2-digit numeric company code (default: 10 if blank/zero).
- `effective_date`: 8-digit numeric date (YYYYMMDD).
- `effective_time`: 4-digit numeric time (HHMM, 00:00–23:59).
- `file_group`: 1-character group ('G' or 'Z').
- `mode`: 3-character mode ('MNT' or 'INQ').
- `operation`: String ('ADD', 'UPDATE', 'DELETE', 'RETRIEVE').
- `detail_data`: Array of structures with:
  - `region_code`: 3-digit numeric.
  - `price`: 9.4 packed decimal.

**Outputs**:
- `status`: String ('SUCCESS', 'ERROR').
- `error_message`: String (if applicable).
- `records`: Array of retrieved records (for RETRIEVE operation).

**Process Steps**:
1. **Validate Inputs**:
   - Verify `company_code` exists in `bicont`. Return error `ERR0021` if invalid.
   - Validate `effective_date` using date validation logic (YYYYMMDD). Return error `ERR0020` if invalid.
   - Validate `effective_time` for valid hours (00–23) and minutes (00–59). Return error `ERR0019` if invalid.
   - Verify `mode` is 'MNT' or 'INQ'. Return error if invalid.
   - Ensure `operation` is valid ('ADD', 'UPDATE', 'DELETE', 'RETRIEVE').
   - Validate `detail_data.region_code` against `gstabl`. Return error if invalid.

2. **Apply File Overrides**:
   - Use `file_group` to select library for `bicont`, `gstabl`, `bbndfi`, `bbndfi1`, `bbndfird`, `bbndfih`.

3. **Process Operation**:
   - **ADD** (mode: 'MNT'):
     - Add new record to `bbndfi` with `company_code`, `effective_date`, `effective_time`, `region_code`, and `price`.
     - Log to `bbndfih` with timestamp and user.
   - **UPDATE** (mode: 'MNT'):
     - Update existing `bbndfi` record’s `price` for the specified key.
     - Log update to `bbndfih`.
   - **DELETE** (mode: 'MNT'):
     - Delete `bbndfi` record and log to `bbndfih` with delete flag 'D'.
   - **RETRIEVE** (mode: 'MNT' or 'INQ'):
     - Retrieve records from `bbndfi` for `company_code`, `effective_date`, and optionally `effective_time`.
     - Include region descriptions from `gstabl` and occurrence counts.

4. **Return Results**:
   - Set `status` to 'SUCCESS' or 'ERROR' with `error_message`.
   - For RETRIEVE, populate `records` with data.

**Business Rules**:
- Valid company and region codes required.
- Effective date (YYYYMMDD) and time (HHMM) must be valid.
- 'INQ' mode restricts to RETRIEVE.
- All changes are logged to `bbndfih` with timestamps, user IDs, and delete flag for deletions.
- Supports toggling between all regions (`gstabl`) and existing records (`bbndfi`).

**Calculations**:
- **Occurrence Count**: Count records for a given `company_code`, `effective_date`, and `effective_time` to populate occurrence field.

<xaiArtifact artifact_id="6bc889de-6fbf-4f06-9351-4d04bdf3f18b" artifact_version_id="cb510e4e-e7b0-45d7-b406-c3b7d97ad8b9" title="NationalDieselFuelIndexMaintenance.md" contentType="text/markdown">

# Function Requirement: MaintainNationalDieselFuelIndex

## Purpose
Manage national diesel fuel index records (add, update, delete, retrieve) for a company, effective date, time, and region.

## Inputs
- `company_code`: 2-digit numeric (default: 10).
- `effective_date`: 8-digit YYYYMMDD.
- `effective_time`: 4-digit HHMM (00:00–23:59).
- `file_group`: 'G' or 'Z'.
- `mode`: 'MNT' or 'INQ'.
- `operation`: 'ADD', 'UPDATE', 'DELETE', 'RETRIEVE'.
- `detail_data`: Array of `{region_code: 3-digit numeric, price: 9.4}`.

## Outputs
- `status`: 'SUCCESS' or 'ERROR'.
- `error_message`: String (if applicable).
- `records`: Array of records (for RETRIEVE).

## Process Steps
1. **Validate Inputs**:
   - Check `company_code` (`bicont`), `effective_date`, `effective_time` (HHMM), `mode`, `operation`, `region_code` (`gstabl`).
2. **File Overrides**: Select library for `bicont`, `gstabl`, `bbndfi`, `bbndfi1`, `bbndfird`, `bbndfih`.
3. **Process Operation**:
   - **ADD** ('MNT'): Add record to `bbndfi`, log to `bbndfih`.
   - **UPDATE** ('MNT'): Update `price` in `bbndfi`, log to `bbndfih`.
   - **DELETE** ('MNT'): Delete record from `bbndfi`, log with 'D' flag.
   - **RETRIEVE** ('MNT'/'INQ'): Fetch records with region descriptions, occurrence counts.
4. **Return Results**: Set `status`, `error_message`, `records`.

## Business Rules
- Valid company, region codes required.
- Valid date (YYYYMMDD), time (HHMM).
- 'INQ' restricts to RETRIEVE.
- Log changes with timestamps, user IDs.
- Support all regions or existing records.

## Calculations
- **Occurrence Count**: Count records for `company_code`, `effective_date`, `effective_time`.

</xaiArtifact>

---

#### Function Requirement Document: Historical File Inquiry

**Function Name**: `QueryFileHistory`

**Purpose**: Retrieve historical change records for a specified file, company, and key fields, returning audit trail data.

**Inputs**:
- `file_name`: String (e.g., 'BBNDFI', 'BBCFSH', 'BBCFSD', etc.).
- `file_group`: 1-character ('G' or 'Z').
- `company_code`: 2-digit numeric.
- `key_fields`: Structure specific to `file_name` (e.g., for `BBCFSD`: `carrier_code`, `effective_date`, `from_price`).

**Outputs**:
- `status`: String ('SUCCESS', 'ERROR').
- `error_message`: String (if applicable).
- `history_records`: Array of historical records with change date, time, user, and modified fields.

**Process Steps**:
1. **Validate Inputs**:
   - Verify `file_name` is supported (e.g., 'BBNDFI', 'BBCFSH', 'BBCFSD').
   - Validate `company_code` against `glcont`.
   - Validate `key_fields` based on file-specific requirements (e.g., `effective_date` for `BBCFSH`).
2. **Apply File Overrides**:
   - Use `file_group` to select library for master and history files (e.g., `bbndfi`, `bbndfihx`).
3. **Retrieve History**:
   - Query the history file (e.g., `bbndfihx`, `bbcfshhx`, `bbcfsdhx`) using file-specific key list (e.g., `bbndfikey`, `bbcfshkey`).
   - Fetch records matching `company_code` and `key_fields`, including change date, time, user, and modified fields (e.g., price, surcharge percentage).
4. **Return Results**:
   - Set `status` to 'SUCCESS' or 'ERROR' with `error_message`.
   - Populate `history_records` with retrieved data.

**Business Rules**:
- Inquiry-only; no modifications allowed.
- Valid company code and file-specific key fields required.
- History files provide audit trails with timestamps, user IDs, and changed values.

**Calculations**:
- None required; function retrieves and returns raw historical data.

<xaiArtifact artifact_id="d8a953aa-6d08-4826-9521-d06608a93406" artifact_version_id="a36e5666-ecca-4e45-af37-a633e73a3ec0" title="HistoricalFileInquiry.md" contentType="text/markdown">

# Function Requirement: QueryFileHistory

## Purpose
Retrieve historical change records for a specified file, company, and key fields.

## Inputs
- `file_name`: String (e.g., 'BBNDFI', 'BBCFSH', 'BBCFSD').
- `file_group`: 'G' or 'Z'.
- `company_code`: 2-digit numeric.
- `key_fields`: Structure (e.g., `carrier_code`, `effective_date`, `from_price` for `BBCFSD`).

## Outputs
- `status`: 'SUCCESS' or 'ERROR'.
- `error_message`: String (if applicable).
- `history_records`: Array with change date, time, user, modified fields.

## Process Steps
1. **Validate Inputs**: Verify `file_name`, `company_code` (`glcont`), `key_fields`.
2. **File Overrides**: Select library for master/history files.
3. **Retrieve History**: Query history file using key list, fetch matching records.
4. **Return Results**: Set `status`, `error_message`, `history_records`.

## Business Rules
- Inquiry-only.
- Valid company code, key fields required.
- History files provide audit trails.

## Calculations
- None; retrieves raw historical data.

</xaiArtifact>

---

#### Function Requirement Document: Table Value Lookup

**Function Name**: `LookupTableValue`

**Purpose**: Retrieve and return a valid table value (e.g., region code) from the `gstabl` table based on a table type and optional search string.

**Inputs**:
- `table_type`: String (e.g., region code type).
- `search_string`: String (optional, for filtering descriptions).

**Outputs**:
- `status`: String ('SUCCESS', 'ERROR').
- `error_message`: String (if applicable).
- `selected_code`: String (selected table code, e.g., region code).

**Process Steps**:
1. **Validate Inputs**:
   - Verify `table_type` exists in `gstabl`. Return error "Invalid Table Type Code" if invalid.
   - Validate `search_string` (if provided) for non-empty content.
2. **Query Table**:
   - Retrieve records from `gstabl` for `table_type`.
   - If `search_string` is provided, filter records where `tbdesc` contains `search_string`.
3. **Select Value**:
   - Assume the first matching record is selected (or implement logic to prioritize based on criteria, e.g., exact match).
   - Set `selected_code` to the table code (`tbcode`).
4. **Return Results**:
   - Set `status` to 'SUCCESS' or 'ERROR' with `error_message`.
   - Return `selected_code`.

**Business Rules**:
- Inquiry-only; returns a single valid table code.
- `table_type` must exist in `gstabl`.
- `search_string` filters descriptions case-sensitively.

**Calculations**:
- **Search String Length**: Compute length of trimmed `search_string` for filtering.

<xaiArtifact artifact_id="00ea14bf-a4ef-4450-b7b0-8fb4f6794e96" artifact_version_id="55b7d8ee-7be6-4829-aebc-40bfa5cd08b6" title="TableValueLookup.md" contentType="text/markdown">

# Function Requirement: LookupTableValue

## Purpose
Retrieve a valid table value (e.g., region code) from `gstabl` based on table type and optional search string.

## Inputs
- `table_type`: String (e.g., region code type).
- `search_string`: String (optional).

## Outputs
- `status`: 'SUCCESS' or 'ERROR'.
- `error_message`: String (if applicable).
- `selected_code`: String (e.g., region code).

## Process Steps
1. **Validate Inputs**: Verify `table_type` (`gstabl`), `search_string`.
2. **Query Table**: Fetch `gstabl` records for `table_type`, filter by `search_string` if provided.
3. **Select Value**: Choose first matching record’s `tbcode`.
4. **Return Results**: Set `status`, `error_message`, `selected_code`.

## Business Rules
- Inquiry-only.
- Valid `table_type` required.
- Case-sensitive `search_string` filtering.

## Calculations
- **Search String Length**: Compute length of trimmed `search_string`.

</xaiArtifact>

---

### Notes
- **Programmatic Adaptation**: The functions eliminate interactive elements (e.g., subfile displays, function keys) by assuming inputs are provided upfront and results are returned programmatically. For `LookupTableValue`, the selection logic assumes the first matching record for simplicity, but could be extended to include prioritization criteria if needed.
- **Error Handling**: All functions include robust validation with specific error codes/messages derived from the original programs (e.g., `ERR0010`, `ERR0021`).
- **History Logging**: `MaintainCarrierFuelSurcharge` and `MaintainNationalDieselFuelIndex` log all changes to history files, preserving audit trails as in the original programs.
- **File Overrides**: The `file_group` parameter ensures correct library selection, mirroring the original override logic (`OVG`, `OVZ`).
- **Date Validation**: Date validation logic (originally via `GSDTEDIT`) is abstracted as internal validation for YYYYMMDD format.
- **Integration**: The functions maintain the interdependencies (e.g., `LookupTableValue` supporting `MaintainCarrierFuelSurcharge` for region code validation) but operate independently of screen-based navigation.

These function requirements provide a concise, business-focused translation of the interactive RPG programs into programmatic processes, suitable for integration into modern systems while preserving core business logic and data integrity.