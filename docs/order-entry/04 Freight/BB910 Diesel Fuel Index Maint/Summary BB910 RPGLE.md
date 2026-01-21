The call stack involves two RPG programs: **BB910** (main program for National Diesel Fuel Index maintenance and inquiry) and **GB730P** (called for history inquiry). Below, I identify the use cases implemented by this call stack and provide a function requirement document for each, assuming they are large functions that process inputs programmatically rather than through screen interactions .

---

### Use Cases Implemented

Based on the functionality of **BB910** and **GB730P**, the following use cases are implemented:

1. **Maintain National Diesel Fuel Index Records**:
   - Allows users to create, update, or delete National Diesel Fuel Index records for a given company, effective date, time, and region, with changes logged in the history file.
   - Implemented in **BB910** (subroutines `srsfl1`, `srsfl2`, `sf1prc`, `sf2prc`, `sf2upd`).

2. **Inquire National Diesel Fuel Index Records**:
   - Enables users to view National Diesel Fuel Index records without modification, displaying records by company, effective date, time, and region.
   - Implemented in **BB910** (subroutines `srsfl1`, `srsfl2`, `sf1s05`).

3. **View History of National Diesel Fuel Index Records**:
   - Displays historical changes (create, update, delete) for National Diesel Fuel Index records, including details like change date, time, user, region, and price.
   - Implemented in **GB730P** (called via F09 in `srsfl2` of **BB910**).

---

### Function Requirement Documents

Each use case is reimagined as a large function that takes inputs programmatically and returns outputs, focusing on business requirements and calculations where applicable. The documents are concise, outlining process steps, inputs, outputs, and business rules.

---

#### Use Case 1: Maintain National Diesel Fuel Index Records

<xaiArtifact artifact_id="8c2480bf-045c-4368-80af-a3f2945488f5" artifact_version_id="010f1fa9-0efa-4994-b52f-8fae5cf21807" title="MaintainDieselFuelIndex.md" contentType="text/markdown">

# Function Requirement Document: MaintainDieselFuelIndex

## Purpose
Create, update, or delete National Diesel Fuel Index records for a company, effective date, time, and region, logging changes to the history file.

## Inputs
- **CompanyCode**: 2-digit numeric company code.
- **FileGroup**: Single character (`Z` or `G`) for file overrides.
- **Records**: Array of records, each containing:
  - **EffectiveDate**: 8-digit numeric date (YYYYMMDD).
  - **EffectiveTime**: 4-digit numeric time (HHMM).
  - **Region**: 3-character region code.
  - **Price**: 9.4 packed decimal price (or 0 for deletion).
  - **Action**: String (`CREATE`, `UPDATE`, `DELETE`).
- **UserID**: 8-character user identifier.

## Outputs
- **Status**: String (`SUCCESS`, `ERROR`).
- **ErrorMessages**: Array of error messages (if any).
- **UpdatedRecords**: Array of updated records with occurrence count.

## Process Steps
1. **Validate Inputs**:
   - Verify `CompanyCode` exists in `bicont` file.
   - Validate `EffectiveDate` using date validation logic.
   - Ensure `EffectiveTime` is valid (00:00–23:59).
   - Check `Region` exists in `gstabl` (for `CREATE`/`UPDATE`).
   - For `CREATE`, ensure record does not exist in `bbndfi`.
   - For `UPDATE`/`DELETE`, ensure record exists in `bbndfi`.
2. **Apply File Overrides**:
   - Use `FileGroup` to override files (`gbbndfi`/`zbbndfi`, `gbicont`/`zbicont`, etc.).
3. **Process Records**:
   - For each record in `Records`:
     - **CREATE**: Write new record to `bbndfi` with `CompanyCode`, `EffectiveDate`, `EffectiveTime`, `Region`, `Price`.
     - **UPDATE**: Update existing record in `bbndfi` with new `Price`.
     - **DELETE**: Delete record from `bbndfi` (set price to 0).
     - Write history record to `bbndfih` with action details (deletion flag, company, date, time, region, price, change date, change time, `UserID`).
     - Update occurrence count for the effective date/time.
4. **Return Results**:
   - If successful, return `SUCCESS` and updated records.
   - If errors occur, return `ERROR` with error messages.

## Business Rules
- **Company Validation**: `CompanyCode` must exist in `bicont`, else return error `ERR0021`.
- **Date Validation**: `EffectiveDate` must be valid (YYYYMMDD format), else return error `ERR0020`.
- **Time Validation**: `EffectiveTime` must be valid (HHMM, 00:00–23:59).
- **Record Existence**:
  - `CREATE` fails if record exists (`ERR0101`).
  - `UPDATE`/`DELETE` fails if record does not exist (`ERR0102`).
- **Region Validation**: For `CREATE`/`UPDATE`, `Region` must exist in `gstabl`.
- **History Logging**: Every action (create, update, delete) must log to `bbndfih` with change date, time, and `UserID`.
- **Price Handling**: Price is 9.4 packed decimal; deletion sets price to 0.

## Calculations
- **Occurrence Count**: Increment count for each unique `EffectiveDate`/`EffectiveTime` combination after processing.
- **Change Date/Time**: Use system date (YYYYMMDD) and time (HHMMSS) for history records.

</xaiArtifact>

---

#### Use Case 2: Inquire National Diesel Fuel Index Records

<xaiArtifact artifact_id="14a53111-025f-471b-85eb-5e6d36ddd6aa" artifact_version_id="13527ce3-f5d1-4226-8229-34e46cb1d3e6" title="InquireDieselFuelIndex.md" contentType="text/markdown">

# Function Requirement Document: InquireDieselFuelIndex

## Purpose
Retrieve and return National Diesel Fuel Index records for a company, optionally filtered by effective date, time, or region.

## Inputs
- **CompanyCode**: 2-digit numeric company code.
- **FileGroup**: Single character (`Z` or `G`) for file overrides.
- **EffectiveDate**: 8-digit numeric date (YYYYMMDD, optional).
- **EffectiveTime**: 4-digit numeric time (HHMM, optional).
- **Region**: 3-character region code (optional).

## Outputs
- **Status**: String (`SUCCESS`, `ERROR`).
- **ErrorMessages**: Array of error messages (if any).
- **Records**: Array of records, each containing:
  - **EffectiveDate**: 8-digit numeric date (YYYYMMDD).
  - **EffectiveTime**: 4-digit numeric time (HHMM).
  - **Region**: 3-character region code.
  - **Price**: 9.4 packed decimal price.
  - **OccurrenceCount**: Numeric count of records for the date/time.

## Process Steps
1. **Validate Inputs**:
   - Verify `CompanyCode` exists in `bicont` file.
   - If provided, validate `EffectiveDate` using date validation logic.
   - If provided, ensure `EffectiveTime` is valid (00:00–23:59).
   - If provided, check `Region` exists in `gstabl`.
2. **Apply File Overrides**:
   - Use `FileGroup` to override files (`gbbndfi`/`zbbndfi`, `gbicont`/`zbicont`, `gbbndfird`/`zbndfird`).
3. **Retrieve Records**:
   - Query `bbndfi` using `CompanyCode` and optional filters (`EffectiveDate`, `EffectiveTime`, `Region`).
   - For each matching record, retrieve `EffectiveDate`, `EffectiveTime`, `Region`, `Price`, and calculate `OccurrenceCount`.
   - If `Region` is provided, filter by region using `bbndfird`.
4. **Return Results**:
   - Return `SUCCESS` and retrieved records, or `ERROR` with error messages if validation fails.

## Business Rules
- **Company Validation**: `CompanyCode` must exist in `bicont`, else return error `ERR0021`.
- **Date Validation**: If provided, `EffectiveDate` must be valid (YYYYMMDD), else return error `ERR0020`.
- **Time Validation**: If provided, `EffectiveTime` must be valid (HHMM, 00:00–23:59).
- **Region Validation**: If provided, `Region` must exist in `gstabl`.
- **Read-Only**: No modifications to data are allowed.
- **Occurrence Count**: Count unique records for each `EffectiveDate`/`EffectiveTime` combination.

## Calculations
- **Occurrence Count**: Count records in `bbndfi` for the same `EffectiveDate` and `EffectiveTime`.

</xaiArtifact>

---

#### Use Case 3: View History of National Diesel Fuel Index Records

<xaiArtifact artifact_id="5acca8ed-db1f-4f1c-a32a-33ddbd3c516e" artifact_version_id="c4a7ffd5-8c77-4e88-82bd-7a3f70306b74" title="ViewDieselFuelIndexHistory.md" contentType="text/markdown">

# Function Requirement Document: ViewDieselFuelIndexHistory

## Purpose
Retrieve and return historical changes (create, update, delete) for National Diesel Fuel Index records for a company, effective date, time, and region.

## Inputs
- **CompanyCode**: 2-digit numeric company code.
- **FileGroup**: Single character (`Z` or `G`) for file overrides.
- **EffectiveDate**: 8-digit numeric date (YYYYMMDD).
- **EffectiveTime**: 4-digit numeric time (HHMM).
- **Region**: 3-character region code.

## Outputs
- **Status**: String (`SUCCESS`, `ERROR`).
- **ErrorMessages**: Array of error messages (if any).
- **HistoryRecords**: Array of history records, each containing:
  - **ChangeDate**: 8-digit numeric date (YYYYMMDD).
  - **ChangeTime**: 6-digit numeric time (HHMMSS).
  - **UserID**: 8-character user identifier.
  - **Region**: 3-character region code.
  - **Price**: 9.4 packed decimal price.
  - **Action**: String (`CREATE`, `UPDATE`, `DELETE`).

## Process Steps
1. **Validate Inputs**:
   - Verify `CompanyCode` exists in `bicont` file.
   - Validate `EffectiveDate` using date validation logic.
   - Ensure `EffectiveTime` is valid (00:00–23:59).
   - Check `Region` exists in `gstabl`.
2. **Apply File Overrides**:
   - Use `FileGroup` to override files (`gbbndfihx`/`zbbndfihx`, `gbicont`/`zbicont`).
3. **Retrieve History Records**:
   - Query `bbndfihx` using `CompanyCode`, `EffectiveDate`, `EffectiveTime`, and `Region`.
   - For each matching record, retrieve `ChangeDate`, `ChangeTime`, `UserID`, `Region`, `Price`, and determine `Action` based on deletion flag (`lhdel`).
4. **Return Results**:
   - Return `SUCCESS` and history records, or `ERROR` with error messages if validation fails.

## Business Rules
- **Company Validation**: `CompanyCode` must exist in `bicont`, else return error `ERR0021`.
- **Date Validation**: `EffectiveDate` must be valid (YYYYMMDD), else return error `ERR0020`.
- **Time Validation**: `EffectiveTime` must be valid (HHMM, 00:00–23:59).
- **Region Validation**: `Region` must exist in `gstabl`.
- **Read-Only**: No modifications to data are allowed.
- **Action Determination**: Map `lhdel` flag to `Action`:
  - `Y` → `DELETE`.
  - `N` → `CREATE` or `UPDATE` (based on context in `bbndfihx`).

## Calculations
- None required; data is retrieved directly from `bbndfihx`.

</xaiArtifact>

---

### Summary

The call stack implements three use cases: maintaining (create/update/delete) National Diesel Fuel Index records, inquiring records, and viewing their history. Each function requirement document reimagines the interactive processes in **BB910** and **GB730P** as programmatic functions, focusing on business rules like input validation, file overrides, and history logging. Calculations are minimal, primarily involving occurrence counts and date/time formatting. The functions ensure data integrity and provide clear outputs for integration into larger systems.