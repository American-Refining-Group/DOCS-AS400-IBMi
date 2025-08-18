### List of Use Cases Implemented in the GS929 Program Call Stack

The call stack consists of five RPG programs (`GS929P`, `GS929`, `GS9293`, `GS9294`, `GS9295`) that collectively manage product load records in an IBM i (AS/400) environment. Below is a comprehensive list of use cases implemented across these programs, derived from their functionality and interactions. Each use case represents a distinct user-driven or system-driven process for managing product load data.

1. **Browse Product Load Records (GS929P)**:
   - **Description**: Allows users to view a list of product load records in a subfile, with options to filter by various criteria (e.g., company, location, product, container, priority, sequence number, responsibility area, major location, product type, hazardous material code, or responsible person) and toggle the inclusion of inactive records.
   - **Trigger**: Initiated when the user runs `GS929P` in inquiry ('INQ') or maintenance ('MNT') mode without specifying a direct record.
   - **Outcome**: Displays a scrollable list of records, allowing navigation and selection for further actions.

2. **Create a New Product Load Record (GS929P, GS929)**:
   - **Description**: Enables users to create a new product load record by entering key fields and detailed data, with validation to ensure the record does not already exist and meets data requirements.
   - **Trigger**: User selects option '1' in the subfile direct access field (`d1opt`) in `GS929P`, which calls `GS929` to capture detailed input.
   - **Outcome**: A new record is written to the `prdlod1` file, and the subfile is refreshed.

3. **Update an Existing Product Load Record (GS929P, GS929)**:
   - **Description**: Allows users to modify an existing product load record’s key fields (e.g., priority, sequence number) or non-key fields (e.g., descriptions, scheduling), ensuring the updated key combination is unique.
   - **Trigger**: User selects option '2' in the subfile (`s1opt`) in `GS929P`, which calls `GS929` for editing.
   - **Outcome**: The record is updated in `prdlod1`, and a confirmation message is displayed.

4. **Copy a Product Load Record (GS929P, GS9293)**:
   - **Description**: Permits users to duplicate an existing product load record to a new record with different key values, copying all non-key fields verbatim.
   - **Trigger**: User selects option '3' in the subfile (`s1opt`) in `GS929P`, which opens a copy window (`sflcpy1`) and calls `GS9293` to perform the copy.
   - **Outcome**: A new record is created in `prdlod1`, and the subfile is refreshed.

5. **Inactivate or Reactivate a Product Load Record (GS929P, GS9294)**:
   - **Description**: Allows users to toggle a product load record’s status between active ('A') and inactive ('I') via a confirmation window.
   - **Trigger**: User selects option '4' in the subfile (`s1opt`) in `GS929P`, which calls `GS9294` to display the confirmation window and update the status.
   - **Outcome**: The record’s `pddel` field is updated to 'A' or 'I', and a confirmation message is sent.

6. **Display a Product Load Record (GS929P, GS929)**:
   - **Description**: Enables users to view the details of a product load record in a read-only mode without modifying it.
   - **Trigger**: User selects option '5' in the subfile (`s1opt`) in `GS929P`, which calls `GS929` in inquiry mode.
   - **Outcome**: The record details are displayed across two panel formats (`FMT01`, `FMT02`) without allowing changes.

7. **Print a Product Load Listing (GS929P, GS9295)**:
   - **Description**: Generates a printed report of all product load records, including key fields and additional details, formatted for a printer.
   - **Trigger**: User presses F15 in `GS929P`, which calls `GS9295` to produce the report.
   - **Outcome**: A spool file is generated in the job’s output queue with the product load listing.

---

### Function Requirement Document: Product Load Management Function

#### Function Name
`manageProductLoad`

#### Purpose
To encapsulate the core functionality of the GS929 call stack for managing product load records, including browsing, creating, updating, copying, inactivating/reactivating, and printing records, using input parameters instead of screen interactions.

#### Inputs
- **Operation**: String (e.g., 'BROWSE', 'CREATE', 'UPDATE', 'COPY', 'INACTIVATE', 'REACTIVATE', 'DISPLAY', 'PRINT').
- **FileGroup**: String ('G' or 'Z') – Specifies the file group (`gprdlod1` or `zprdlod1`).
- **RecordKey** (for all operations except BROWSE and PRINT):
  - `company`: Numeric (2) – Company number.
  - `location`: String (3) – Loading location.
  - `product`: String (4) – Product code.
  - `container`: String (3) – Container code.
  - `priority`: Numeric (1) – Loading priority (1-9).
  - `sequence`: Numeric (3) – Sequence number.
  - `respArea`: String (5) – Responsibility area code.
  - `majorLoc`: String (4) – Major location code.
  - `prodType`: String (30) – Product type ('BULK', 'PACKAGED', 'RAILCAR').
- **RecordData** (for CREATE, UPDATE, COPY):
  - `categoryDesc`: String – Category description (non-blank).
  - `commonNames`: String – Common names (non-blank).
  - `status`: String ('A' or 'I') – Record status.
  - `carrierType`: String ('TRUCK' or 'RAILCAR') – Carrier type.
  - `truckBlend`: String ('Y' or 'N') – Truck blend indicator.
  - `respPerson`: String – Responsible person (non-blank).
  - `hazmatCode`: String ('X' or blank) – Hazardous material code.
  - `schedule`: Array of String (7 days, 25 slots each, 'Y' or 'N') – Scheduling flags.
- **CopyToKey** (for COPY):
  - Same fields as `RecordKey` but for the target record.
- **FilterCriteria** (for BROWSE):
  - `company`, `location`, `product`, `container`, `priority`, `sequence`, `respArea`, `majorLoc`, `prodType`, `hazmatCode`, `respPerson`: Optional fields for filtering.
  - `includeInactive`: Boolean – Include inactive records.

#### Outputs
- **Status**: String – 'SUCCESS', 'ERROR', or specific status ('A' for reactivated, 'I' for inactivated).
- **Message**: String – Error or confirmation message (e.g., "Record created successfully").
- **RecordList** (for BROWSE): Array of records containing `RecordKey` and `RecordData` fields.
- **SpoolFile** (for PRINT): Reference to the generated spool file.

#### Process Steps
1. **Validate Inputs**:
   - Ensure `FileGroup` is 'G' or 'Z'.
   - For CREATE, UPDATE, COPY: Validate `RecordKey` fields (non-blank, valid ranges, `prodType` in ['BULK', 'PACKAGED', 'RAILCAR'], `priority` in 1-9).
   - For CREATE, UPDATE, COPY: Validate `RecordData` (`categoryDesc`, `commonNames`, `respPerson` non-blank; `status` in ['A', 'I']; `carrierType` in ['TRUCK', 'RAILCAR']; `truckBlend` in ['Y', 'N']; `hazmatCode` in ['X', '']; at least one `schedule` slot is 'Y').
   - For COPY: Validate `CopyToKey` fields similarly.
   - For BROWSE: Validate optional `FilterCriteria` fields for format.

2. **Open Database File**:
   - Apply file override for `prdlod1` (or `prdlodx` for PRINT) based on `FileGroup` using `QCMDEXC` (`gprdlod1` for 'G', `zprdlod1` for 'Z').
   - Open the file (`prdlod1` for CREATE, UPDATE, COPY, INACTIVATE, REACTIVATE, DISPLAY; `prdlodx` for PRINT).

3. **Execute Operation**:
   - **BROWSE**:
     - Read `prdlod1` records, applying `FilterCriteria`.
     - Exclude inactive records (`pddel = 'I'`) if `includeInactive` is false.
     - Return matching records in `RecordList`.
   - **CREATE**:
     - Check if the record with `RecordKey` exists in `prdlod1` using key list.
     - If not exists, create a new record with `RecordKey` and `RecordData`, set `pddel` to 'A', and write to `prdlod1`.
     - Return `Status = 'SUCCESS'`, `Message = 'Record created successfully'`.
   - **UPDATE**:
     - Chain to `prdlod1` with `RecordKey`.
     - If exists and not inactive (`pddel ≠ 'I'`), update with new `RecordKey` and `RecordData` if key fields changed (ensure new key is unique).
     - Return `Status = 'SUCCESS'`, `Message = 'Record updated successfully'`.
   - **COPY**:
     - Chain to `prdlod1` with `RecordKey` to retrieve source record.
     - If exists, chain to `prdlod1` with `CopyToKey` to ensure it does not exist.
     - Create a new record with `CopyToKey` and source `RecordData`, set `pddel` to 'A', and write to `prdlod1`.
     - Return `Status = 'SUCCESS'`, `Message = 'Record copied successfully'`.
   - **INACTIVATE**:
     - Chain to `prdlod1` with `RecordKey`.
     - If exists and not inactive (`pddel ≠ 'I'`), set `pddel` to 'I' and update the record.
     - Return `Status = 'I'`, `Message = 'Record inactivated'`.
   - **REACTIVATE**:
     - Chain to `prdlod1` with `RecordKey`.
     - If exists and inactive (`pddel = 'I'`), set `pddel` to 'A' and update the record.
     - Return `Status = 'A'`, `Message = 'Record reactivated'`.
   - **DISPLAY**:
     - Chain to `prdlod1` with `RecordKey`.
     - If exists, return `RecordKey` and `RecordData` in `RecordList` (single record).
     - Return `Status = 'SUCCESS'`, `Message = 'Record retrieved'`.
   - **PRINT**:
     - Open `qsysprt` with overrides (page size 68x164, 8 LPI, 15 CPI, overflow at line 62, hold and save spool file).
     - Read all `prdlodx` records sequentially.
     - Print headers (company name, report title, job/user info, date, time, page) and detail lines (`pdcono`, `pdloc`, `pdprod`, `pdcntr`, `pdseq#`, `pdracd`, `pdmlcd`, `pdtype`, `pdresp`, `pdhazm`, `pdprty`) with overflow handling.
     - Close `qsysprt` and delete overrides.
     - Return `Status = 'SUCCESS'`, `Message = 'Report generated'`, `SpoolFile` reference.

4. **Close Files**:
   - Close `prdlod1` or `prdlodx` and `qsysprt` (for PRINT).
   - Remove any file overrides using `QCMDEXC`.

#### Business Rules
1. **Record Existence**:
   - CREATE and COPY: The target record must not exist in `prdlod1`.
   - UPDATE, INACTIVATE, REACTIVATE, DISPLAY: The record must exist.
   - COPY: The source record must exist.

2. **Field Validation**:
   - Key fields: Non-blank, `priority` in 1-9, `prodType` in ['BULK', 'PACKAGED', 'RAILCAR'].
   - Data fields: `categoryDesc`, `commonNames`, `respPerson` non-blank; `status` in ['A', 'I']; `carrierType` in ['TRUCK', 'RAILCAR']; `truckBlend` in ['Y', 'N']; `hazmatCode` in ['X', '']; at least one `schedule` slot is 'Y'.

3. **Status Transitions**:
   - INACTIVATE: Only active records (`pddel ≠ 'I'`) can be inactivated (`pddel = 'I'`).
   - REACTIVATE: Only inactive records (`pddel = 'I'`) can be reactivated (`pddel = 'A'`).

4. **File Group**:
   - `FileGroup` ('G' or 'Z') determines the file (`gprdlod1` or `zprdlod1`/`gprdlodx` or `zprdlodx`).

5. **Report Formatting (PRINT)**:
   - Includes all records without filtering.
   - Formatted with headers (job, user, date, time, page) and detail lines, with overflow at line 62.

#### Calculations
- **Page Numbering (PRINT)**: Incremented automatically by the printer file (`page` field).
- **Date and Time Formatting**: Uses `ps#mdy` and `ps#hms` from `psds##` to format the current date (`t#mdcy`) and time (`t#hms`) in the report header.
- **Overflow Handling (PRINT)**: Triggers header reprint when the overflow indicator (`*inof`) is set at line 62.

#### Error Handling
- Return `Status = 'ERROR'` and a specific `Message` for:
  - Invalid `FileGroup`.
  - Missing or invalid key/data fields.
  - Record already exists (CREATE, COPY).
  - Record does not exist (UPDATE, INACTIVATE, REACTIVATE, DISPLAY).
  - Invalid status transitions (INACTIVATE, REACTIVATE).

<xaiArtifact artifact_id="1ce536cc-5eb5-4a19-bd58-c7396966d6ff" artifact_version_id="3072c7e7-4d68-4825-92c1-254032fae6c9" title="ProductLoadManagementRequirements.md" contentType="text/markdown">

# Function Requirement Document: Product Load Management

## Function Name
`manageProductLoad`

## Purpose
To manage product load records by browsing, creating, updating, copying, inactivating/reactivating, displaying, or printing records using input parameters.

## Inputs
- **Operation**: String ('BROWSE', 'CREATE', 'UPDATE', 'COPY', 'INACTIVATE', 'REACTIVATE', 'DISPLAY', 'PRINT').
- **FileGroup**: String ('G' or 'Z').
- **RecordKey** (except BROWSE, PRINT):
  - `company`: Numeric (2).
  - `location`: String (3).
  - `product`: String (4).
  - `container`: String (3).
  - `priority`: Numeric (1, 1-9).
  - `sequence`: Numeric (3).
  - `respArea`: String (5).
  - `majorLoc`: String (4).
  - `prodType`: String (30, 'BULK', 'PACKAGED', 'RAILCAR').
- **RecordData** (CREATE, UPDATE, COPY):
  - `categoryDesc`: String (non-blank).
  - `commonNames`: String (non-blank).
  - `status`: String ('A', 'I').
  - `carrierType`: String ('TRUCK', 'RAILCAR').
  - `truckBlend`: String ('Y', 'N').
  - `respPerson`: String (non-blank).
  - `hazmatCode`: String ('X', '').
  - `schedule`: Array of String (7 days, 25 slots, 'Y' or 'N').
- **CopyToKey** (COPY): Same as `RecordKey`.
- **FilterCriteria** (BROWSE): Optional `RecordKey` fields plus `hazmatCode`, `respPerson`, `includeInactive` (Boolean).

## Outputs
- **Status**: String ('SUCCESS', 'ERROR', 'A', 'I').
- **Message**: String (e.g., "Record created successfully").
- **RecordList** (BROWSE, DISPLAY): Array of `RecordKey` and `RecordData`.
- **SpoolFile** (PRINT): Spool file reference.

## Process Steps
1. **Validate Inputs**:
   - Validate `FileGroup`, `RecordKey`, `RecordData`, `CopyToKey`, `FilterCriteria`.
2. **Open File**:
   - Override and open `prdlod1` (or `prdlodx` for PRINT) based on `FileGroup`.
3. **Execute Operation**:
   - **BROWSE**: Filter and return records in `RecordList`.
   - **CREATE**: Write new record if unique.
   - **UPDATE**: Update existing record if unique.
   - **COPY**: Copy source record to new key if unique.
   - **INACTIVATE**: Set `pddel` to 'I' if active.
   - **REACTIVATE**: Set `pddel` to 'A' if inactive.
   - **DISPLAY**: Return record details.
   - **PRINT**: Print all records with headers.
4. **Close Files**: Close files and remove overrides.

## Business Rules
- Record must not exist for CREATE, COPY; must exist for UPDATE, INACTIVATE, REACTIVATE, DISPLAY.
- Key fields: Non-blank, `priority` 1-9, `prodType` in ['BULK', 'PACKAGED', 'RAILCAR'].
- Data fields: Non-blank `categoryDesc`, `commonNames`, `respPerson`; valid `status`, `carrierType`, `truckBlend`, `hazmatCode`; at least one `schedule` 'Y'.
- INACTIVATE: Only active records. REACTIVATE: Only inactive records.
- PRINT: All records, formatted with headers, overflow at line 62.

## Calculations
- **Page Numbering (PRINT)**: Auto-incremented by printer file.
- **Date/Time**: Formatted from `ps#mdy`, `ps#hms` for report header.

## Error Handling
- Return 'ERROR' for invalid inputs, record existence issues, or status transition violations.

</xaiArtifact>