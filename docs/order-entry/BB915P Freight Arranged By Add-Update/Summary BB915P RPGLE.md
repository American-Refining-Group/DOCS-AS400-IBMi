### List of Use Cases Implemented in the Program Call Stack (BB915P, BB915, BB916)

The call stack consists of three RPGLE programs: **BB915P** (list-level "Work With Freight Arranged By Entries"), **BB915** (detail-level maintenance/inquiry), and **BB916** (delete/reactivate confirmation). Together, they implement a cohesive system for managing "Freight Arranged By" entries in a customer freight system. Below are the distinct use cases derived from the functionality across these programs:

1. **Browse Freight Entries (BB915P)**:
   - **Description**: Allows users to view a list of freight arranged by entries, filtered by company code and optionally by freight code, with the ability to toggle inclusion/exclusion of deleted/expired entries.
   - **Functionality**: Displays a subfile with records from the freight file (`bbfrpr`), supports scrolling (Page Down), repositioning by key fields, and filtering deleted records (F8). Users can select records for further actions.
   - **Modes**: Maintenance (MNT) or Inquiry (INQ).

2. **Create New Freight Entry (BB915P → BB915)**:
   - **Description**: Enables users to create a new freight arranged by entry for a specific company and freight code.
   - **Functionality**: From the list (BB915P, option 1), calls BB915 in MNT mode to display a detail screen (FMT01) for entering new data. Validates inputs and writes a new record to `bbfrpr`.
   - **Modes**: Maintenance only.

3. **Update Existing Freight Entry (BB915P → BB915)**:
   - **Description**: Allows users to modify an existing freight arranged by entry, provided it is not marked as deleted.
   - **Functionality**: From the list (BB915P, option 2), calls BB915 in MNT mode to edit fields on the detail screen (FMT01). Validates inputs and updates the record in `bbfrpr` if changed.
   - **Modes**: Maintenance only.

4. **Delete Freight Entry (BB915P → BB916)**:
   - **Description**: Permits users to mark an active freight entry as deleted (soft delete).
   - **Functionality**: From the list (BB915P, option 4), calls BB916 to display a confirmation window (delwdw). If confirmed (F23), sets the delete flag (`fpdel`) to 'D' and updates `bbfrpr`.
   - **Modes**: Maintenance only.

5. **Reactivate Freight Entry (BB915P → BB916)**:
   - **Description**: Allows users to reactivate a previously deleted freight entry.
   - **Functionality**: From the list (BB915P, option 4 on a deleted record), calls BB916 to display a confirmation window (delwdw). If confirmed (F22), sets the delete flag (`fpdel`) to 'A' and updates `bbfrpr`.
   - **Modes**: Maintenance only.

6. **View Freight Entry Details (BB915P → BB915)**:
   - **Description**: Enables users to view details of a freight entry in read-only mode.
   - **Functionality**: From the list (BB915P, option 5), calls BB915 in INQ mode to display the detail screen (FMT01) with fields protected. No updates are allowed.
   - **Modes**: Inquiry only.

7. **Direct Access to Freight Entry (BB915P)**:
   - **Description**: Allows users to directly access a freight entry by entering an option (1, 2, 4, or 5) and freight code in the list screen’s control fields to perform create, update, delete, or view actions.
   - **Functionality**: Validates the freight code and option in BB915P, then calls BB915 or BB916 as needed without requiring subfile selection.
   - **Modes**: Maintenance (options 1, 2, 4) or Inquiry (option 5).

### Function Requirement Document for Non-Interactive Freight Entry Management

Below is a function requirement document for a non-interactive version of the freight entry management system, consolidating the use cases into a single function that processes inputs programmatically without screen interactions. The function handles all operations (browse, create, update, delete, reactivate, view, direct access) based on input parameters, focusing on business requirements and necessary calculations.

<xaiArtifact artifact_id="4db6d8af-813e-424c-93e9-839430211642" artifact_version_id="8d9a12cb-483a-4e70-b074-24d011013ea5" title="FreightEntryManagementFunction.md" contentType="text/markdown">

# Freight Entry Management Function Requirements

## Purpose
The `ManageFreightEntry` function manages "Freight Arranged By" entries in the customer freight system, supporting browsing, creating, updating, deleting, reactivating, and viewing entries non-interactively. It processes inputs to perform the requested operation on the freight database (`bbfrpr`) and returns results, including status and error messages.

## Inputs
- **CompanyCode**: Numeric, required. Identifies the company (key field for `bicont` and `bbfrpr`).
- **FreightCode**: Alphanumeric, optional for browse, required for other operations. Unique freight arranged by code (key field for `bbfrpr`).
- **Operation**: String, required. One of: "BROWSE", "CREATE", "UPDATE", "DELETE", "REACTIVATE", "VIEW".
- **FileGroup**: String, required. 'Z' or 'G' to select file library (e.g., zbbfrpr or gbbfrpr).
- **IncludeDeleted**: Boolean, optional (BROWSE only). If true, includes deleted/expired entries; if false, excludes them. Default: false.
- **FreightData**: Object, required for CREATE/UPDATE. Contains:
  - Name: String, required. Freight entry name.
  - City: String, required. City of the freight entry.
  - State: String, required. State code.
  - Zip: String, required. Base zip code.
  - ZipPlus4: String, optional. Extended zip code.
  - Country: String, required. Country code.
  - ArrangedByType: String, required. Must be 'E' (External) or 'I' (Internal).
  - Other fields as defined in `bbfrpr` (e.g., address, contact details).

## Outputs
- **Status**: String. "SUCCESS", "ERROR", or "NO_ACTION" (e.g., canceled or invalid operation).
- **Flag**: String. '1' (record created/updated), 'D' (deleted), 'A' (reactivated), or blank (no change).
- **Records** (BROWSE only): Array of freight records with fields: FreightCode, Name, City, State, Zip, ZipPlus4, Country, ArrangedByType, DeleteFlag ('D' or 'A'), CompanyName (from `bicont`).
- **ErrorMessages**: Array of strings. Lists validation or processing errors (e.g., "Required field missing", "Invalid ArrangedByType").

## Process Steps
1. **Validate Inputs**:
   - Ensure CompanyCode is valid by chaining to `bicont`. If not found, return ERROR with "Invalid company code".
   - For non-BROWSE, ensure FreightCode is provided; else return ERROR with "Freight code required".
   - Validate Operation is one of the allowed values; else return ERROR with "Invalid operation".
   - For CREATE/UPDATE, validate FreightData:
     - Name, City, State, Zip, Country are non-blank.
     - If ZipPlus4 non-blank, Zip must be non-blank.
     - ArrangedByType must be 'E' or 'I'.
     - Return ERROR with specific messages (e.g., "Required field missing", "Invalid ArrangedByType: must be E or I") if invalid.

2. **Apply File Overrides**:
   - Based on FileGroup ('Z' or 'G'), override `bicont` and `bbfrpr` to the appropriate library (e.g., zbbfrpr or gbbfrpr).

3. **Execute Operation**:
   - **BROWSE**:
     - Read `bbfrpr` sequentially by CompanyCode (SETLL + READE).
     - If FreightCode provided, position to it (SETLL).
     - If IncludeDeleted=false, skip records with DeleteFlag='D'.
     - Collect up to a predefined limit (e.g., 28 records, simulating page size) with fields: FreightCode, Name, City, State, Zip, ZipPlus4, Country, ArrangedByType, DeleteFlag, and CompanyName (from `bicont`).
     - Return SUCCESS with Records array.
   - **CREATE**:
     - Chain to `bbfrpr` by CompanyCode + FreightCode. If found, return ERROR with "Freight code already exists".
     - Write new record to `bbfrpr` with FreightData, DeleteFlag='A', and key fields.
     - Return SUCCESS with Flag='1'.
   - **UPDATE**:
     - Chain to `bbfrpr` by CompanyCode + FreightCode. If not found or DeleteFlag='D', return ERROR with "Record not found or deleted".
     - Compare FreightData with existing record. If unchanged, return SUCCESS with Flag=blank.
     - Update record with FreightData, keeping DeleteFlag unchanged.
     - Return SUCCESS with Flag='1'.
   - **DELETE**:
     - Chain to `bbfrpr` by CompanyCode + FreightCode. If not found or DeleteFlag='D', return ERROR with "Record not found or already deleted".
     - Set DeleteFlag='D' and update record.
     - Return SUCCESS with Flag='D'.
   - **REACTIVATE**:
     - Chain to `bbfrpr` by CompanyCode + FreightCode. If not found or DeleteFlag<>'D', return ERROR with "Record not found or not deleted".
     - Set DeleteFlag='A' and update record.
     - Return SUCCESS with Flag='A'.
   - **VIEW**:
     - Chain to `bbfrpr` by CompanyCode + FreightCode. If not found, return ERROR with "Record not found".
     - Return SUCCESS with Records array containing one record (same fields as BROWSE).

4. **Handle Errors**:
   - Collect all validation errors in ErrorMessages.
   - If any errors, return ERROR with ErrorMessages and Flag=blank.

5. **Return Result**:
   - Return Status, Flag, Records (if applicable), and ErrorMessages.

## Business Rules
- **Required Fields (CREATE/UPDATE)**: Name, City, State, Zip, Country must be non-blank.
- **Zip Validation**: ZipPlus4 requires a non-blank Zip.
- **ArrangedByType**: Must be 'E' (External) or 'I' (Internal).
- **Soft Deletion**: Deletion sets DeleteFlag='D'; reactivation sets DeleteFlag='A'. No physical deletion.
- **Record Existence**:
  - CREATE fails if CompanyCode + FreightCode exists.
  - UPDATE/DELETE/REACTIVATE/VIEW fail if CompanyCode + FreightCode not found.
  - UPDATE/DELETE fail if record is deleted (DeleteFlag='D').
  - REACTIVATE fails if record is not deleted (DeleteFlag<>'D').
- **Company Validation**: CompanyCode must exist in `bicont`.
- **No Updates in VIEW**: VIEW operation is read-only.
- **BROWSE Filtering**: IncludeDeleted=false skips records with DeleteFlag='D'.
- **Output Flag**:
  - '1': Record created or updated.
  - 'D': Record marked deleted.
  - 'A': Record reactivated.
  - Blank: No change (e.g., VIEW, unchanged UPDATE, or invalid operation).
- **No Calculations**: No specific calculations (e.g., math or date computations) are performed; all operations are CRUD-based.

## Dependencies
- **Files**:
  - `bicont`: Input-only, for company name lookup.
  - `bbfrpr`: Update-capable, for freight entry CRUD operations.
- **External Programs** (for file overrides and messaging):
  - QCMDEXC: For file overrides.
  - QMHSNDPM: For error message queuing.
  - QMHRMVPM: For clearing messages.

</xaiArtifact>