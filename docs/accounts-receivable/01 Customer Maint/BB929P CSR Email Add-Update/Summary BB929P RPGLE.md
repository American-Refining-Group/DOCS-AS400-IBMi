### List of Use Cases Implemented by the Program Call Stack

The RPGLE programs (`BB929P`, `BB929`, `BB9294`, `BB9295`) collectively form an interactive application for managing Customer Service Representative (CSR) IDs within the Billing and Invoicing system. The following use cases are implemented:

1. **List and Select CSR Records**:
   - **Description**: Users can view a subfile list of CSR records, filtered to include all records (active and inactive), and select records for further actions (create, change, inactivate/reactivate, display, or print).
   - **Program**: `BB929P`
   - **Details**: Displays a subfile with company number, CSR ID, name, email, and status. Users can position the list by company or CSR ID, refresh the list, or toggle between folded/unfolded views (though F8 is disabled in Profound UI). Supports direct access to create or modify records.

2. **Create a New CSR Record**:
   - **Description**: Users can create a new CSR record by specifying company number, CSR ID, name, and email address.
   - **Program**: `BB929` (called by `BB929P` with option 1)
   - **Details**: Validates that the company exists and the record does not already exist, then writes a new record with status `A` (active).

3. **Modify an Existing CSR Record**:
   - **Description**: Users can update the name or email address of an existing, active CSR record.
   - **Program**: `BB929` (called by `BB929P` with option 2)
   - **Details**: Ensures the record is not inactive or deleted before allowing changes. Key fields (company, CSR ID) are protected.

4. **Inactivate or Reactivate a CSR Record**:
   - **Description**: Users can toggle a CSR record's status between active (`A`) and inactive (`I`).
   - **Program**: `BB9294` (called by `BB929P` with option 4)
   - **Details**: Allows reactivation (F22) if the record is inactive or inactivation (F23) if the record is active. Includes a placeholder for checking dependencies (e.g., vendor table), but no validation is implemented.

5. **Display a CSR Record**:
   - **Description**: Users can view the details of a CSR record in read-only mode.
   - **Program**: `BB929` (called by `BB929P` with option 5)
   - **Details**: Displays record details without allowing modifications.

6. **Print a CSR ID Listing**:
   - **Description**: Users can generate a printed report of all CSR records, including company, CSR ID, name, email, and status.
   - **Program**: `BB9295` (called by `BB929P` with F15)
   - **Details**: Produces a formatted report with headers and detail lines, sent to a spooled printer file.

---

### Function Requirement Document: CSR Record Management Function



# CSR Record Management Function Requirements

## Overview
The `manageCSRRecord` function handles the creation, modification, inactivation, reactivation, and retrieval of Customer Service Representative (CSR) records in the Billing and Invoicing system. It processes inputs programmatically without screen interaction, updating or retrieving data from the `bbcsr` file and validating against the `bicont` file.

## Inputs
- **company** (numeric): Company number.
- **csrId** (string): CSR ID.
- **mode** (string, enum: `CREATE`, `UPDATE`, `INACTIVATE`, `REACTIVATE`, `RETRIEVE`): Operation mode.
- **fileGroup** (string, enum: `Z`, `G`): File group for database library selection.
- **csrName** (string, optional): CSR name (required for `CREATE` and `UPDATE`).
- **email** (string, optional): Email address (required for `CREATE` and `UPDATE`).

## Outputs
- **status** (string, enum: `SUCCESS`, `ERROR`): Operation outcome.
- **flag** (string, enum: `1`, `A`, `I`, `E`, null): Return flag (`1` for create/update success, `A` for reactivation, `I` for inactivation, `E` for error, null for retrieval or failure).
- **message** (string): Success or error message.
- **record** (object, optional): Retrieved CSR record details (for `RETRIEVE` mode).

## Process Steps
1. **Validate File Group**:
   - Ensure `fileGroup` is `Z` or `G`. If invalid, return `status=ERROR`, `message="Invalid file group"`.

2. **Apply Database Overrides**:
   - Redirect `bbcsr` to `gbbcsr` (if `fileGroup=G`) or `zbcsr` (if `fileGroup=Z`).
   - Redirect `bicont` to `gbicont` or `zbicont` accordingly.

3. **Validate Company**:
   - Check if `company` exists in `bicont`. If not, return `status=ERROR`, `message="Invalid company number"`.

4. **Process by Mode**:
   - **CREATE**:
     - Validate `csrName` and `email` are non-blank. If blank, return `status=ERROR`, `message="CSR name and email required"`.
     - Check if record (`company`, `csrId`) exists in `bbcsr`. If exists, return `status=ERROR`, `message="CSR ID already exists"`.
     - Write new record to `bbcsr` with `crdel=A` (active), `crco=company`, `crcrid=csrId`, `crcrnm=csrName`, `cremal=email`.
     - Return `status=SUCCESS`, `flag=1`, `message="CSR <csrId> created"`.
   - **UPDATE**:
     - Validate `csrName` and `email` are non-blank. If blank, return `status=ERROR`, `message="CSR name and email required"`.
     - Chain to `bbcsr` using `company`, `csrId`. If not found or `crdel` is `D` or `I`, return `status=ERROR`, `message="Cannot modify inactive or deleted record"`.
     - Update `bbcsr` with `crcrnm=csrName`, `cremal=email`.
     - Return `status=SUCCESS`, `flag=1`, `message="CSR <csrId> updated"`.
   - **INACTIVATE**:
     - Chain to `bbcsr`. If not found or `crdel=I`, return `status=ERROR`, `message="Record not found or already inactive"`.
     - Check for dependencies (placeholder, not implemented). If dependencies exist, return `status=ERROR`, `flag=E`, `message="CSR <csrId> cannot be inactivated due to dependencies"`.
     - Update `bbcsr` with `crdel=I`.
     - Return `status=SUCCESS`, `flag=I`, `message="CSR <csrId> inactivated"`.
   - **REACTIVATE**:
     - Chain to `bbcsr`. If not found or `crdel<>I`, return `status=ERROR`, `message="Record not found or not inactive"`.
     - Update `bbcsr` with `crdel=A`.
     - Return `status=SUCCESS`, `flag=A`, `message="CSR <csrId> reactivated"`.
   - **RETRIEVE**:
     - Chain to `bbcsr`. If not found, return `status=ERROR`, `message="Record not found"`.
     - Return `status=SUCCESS`, `record={company, csrId, csrName, email, status}`.

## Business Rules
1. **Data Validation**:
   - Company number must exist in `bicont`.
   - CSR name and email must be non-blank for `CREATE` and `UPDATE`.
   - CSR record must not exist for `CREATE`.
   - CSR record must be active (`crdel=A`) for `UPDATE` or `INACTIVATE`.
   - CSR record must be inactive (`crdel=I`) for `REACTIVATE`.

2. **Status Management**:
   - New records are created with `crdel=A` (active).
   - Inactivation sets `crdel=I`. Reactivation sets `crdel=A`.
   - Deleted records (`crdel=D`) cannot be modified.

3. **File Group**:
   - `fileGroup` determines the database library (`Z` for `zbcsr`/`zbicont`, `G` for `gbbcsr`/`gbicont`).

4. **Dependency Check**:
   - Inactivation should check for dependencies (e.g., vendor table usage), but this is not implemented (returns `flag=E` if added).

5. **Error Handling**:
   - Returns descriptive error messages for invalid inputs, non-existent records, or invalid status changes.

## Calculations
- No complex calculations are performed. The function primarily manages data validation, record retrieval, and updates to the `crdel` field.

## Notes
- The function assumes programmatic input instead of interactive screen input, consolidating the logic of `BB929`, `BB9294`, and part of `BB929P`.
- The printing use case (`BB9295`) is excluded, as it requires a separate function for report generation.
- The placeholder dependency check in `INACTIVATE` mode should be implemented to validate against related tables (e.g., vendor table).

