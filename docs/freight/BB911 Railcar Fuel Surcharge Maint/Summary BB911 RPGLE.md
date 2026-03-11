Below is a detailed analysis of the use cases implemented by the **BB911** and **GB730P** RPG programs, followed by a function requirement document for a non-interactive version of the primary use cases, focusing on business requirements and calculations .

---

### **List of Use Cases Implemented**

The call stack consists of two programs: **BB911** (National Diesel Fuel Index Maintenance and Inquiry) and **GB730P** (Global File Tracking History Inquiry). Together, they support multiple use cases related to managing and querying fuel index data. The use cases are derived from the functionality described in the program specifications and code.

1. **Use Case 1: Inquire National Diesel Fuel Index Records**
   - **Description**: Allows users to view existing National Diesel Fuel Index records for a specified company and effective date in inquiry mode ('INQ').
   - **Program**: BB911
   - **Details**:
     - Displays records in subfile SFL1, showing effective date, time, retail price, and occurrence count.
     - Supports direct access to specific records by entering company code, effective date, and time.
     - Allows navigation through records using page down and refresh (F5).
     - Validates company code against `bicont` and effective date using `GSDTEDIT`.

2. **Use Case 2: Maintain National Diesel Fuel Index Records**
   - **Description**: Enables users to create, update, or delete National Diesel Fuel Index records in maintenance mode ('MNT').
   - **Program**: BB911
   - **Details**:
     - Create (Option 1): Adds new records to `bbrcsc` with validated company code, effective date, time, and retail price.
     - Update (Option 2): Modifies existing records in `bbrcsc`, updating fields like retail price.
     - Delete: Sets retail price to zero to mark records for deletion.
     - Logs all changes to the history file `bbrcsch` using the `writehist` subroutine.
     - Validates inputs (e.g., non-zero retail price, valid date/time) and checks for record existence to prevent duplicates or invalid updates.
     - Uses subfile SFL2 for detailed record maintenance, supporting All or Review modes.

3. **Use Case 3: View History of National Diesel Fuel Index Changes**
   - **Description**: Allows users to inquire about historical changes to National Diesel Fuel Index records, including change date, time, user, and modified fields.
   - **Program**: BB911 and GB730P
   - **Details**:
     - Initiated via F9 in BB911’s SFL2, calling GB730P with parameters (`x$hist`) specifying the file (`BBNDFI`), company code, effective date, and time.
     - GB730P displays historical records from `bbndfihx` in a subfile, showing change details like date (`lhchd8`), time (`lhchtm`), user (`lhuser`), and retail price (`lhrprc`).
     - Supports navigation through historical records and displays the most recent change date/time initially.

4. **Use Case 4: Validate Input Data for Fuel Index Records**
   - **Description**: Ensures that input data (company code, effective date, time, retail price) is valid before creating or updating records.
   - **Program**: BB911
   - **Details**:
     - Validates company code against `bicont` (error `ERR0021` if invalid).
     - Validates effective date using `GSDTEDIT` (error `ERR0020` if invalid).
     - Validates effective time for valid hours and minutes (error `ERR0019` if invalid).
     - Ensures retail price is non-zero for create/update operations (error `ERR0012` if zero).
     - Checks for record existence to prevent duplicates on create (`ERR0101`) or missing records on update (`ERR0102`).

5. **Use Case 5: Log Changes to Fuel Index History**
   - **Description**: Automatically logs all create, update, and delete operations on National Diesel Fuel Index records to a history file.
   - **Program**: BB911
   - **Details**:
     - Writes history records to `bbrcsch` with details like company code, effective date, time, retail price, change date, change time, user, and delete flag (`lhdel = 'D'` for deletions).
     - Ensures auditability of all changes for tracking purposes.

---

### **Function Requirement Document**

**Function Name**: ManageNationalDieselFuelIndex

**Purpose**: To provide a non-interactive function that manages (creates, updates, deletes) and inquires about National Diesel Fuel Index records, including logging changes to a history file and retrieving historical data, without requiring screen interaction.

**Inputs**:
- **FileGroup**: String (1 char, 'G' or 'Z') – Specifies the library group for file overrides.
- **CompanyCode**: Integer (2 digits) – Company identifier.
- **Mode**: String (3 chars, 'INQ' or 'MNT') – Inquiry or maintenance mode.
- **Operation**: String (6 chars, 'INQUIRE', 'CREATE', 'UPDATE', 'DELETE') – Specifies the action to perform.
- **EffectiveDate**: Integer (8 digits, YYYYMMDD) – Effective date of the fuel index record.
- **EffectiveTime**: Integer (4 digits, HHMM) – Effective time of the record.
- **Region**: String (3 chars) – Region code for the fuel index (required for CREATE/UPDATE).
- **RetailPrice**: Decimal (9.4) – Retail price per gallon (required for CREATE/UPDATE).
- **UserID**: String (8 chars) – User performing the operation (for history logging).

**Outputs**:
- **Status**: String – 'SUCCESS' or 'ERROR' indicating operation outcome.
- **ErrorMessage**: String – Description of any error (e.g., 'Invalid company code').
- **Records** (for INQUIRE): Array of records containing:
  - EffectiveDate: Integer (8 digits, YYYYMMDD)
  - EffectiveTime: Integer (4 digits, HHMM)
  - Region: String (3 chars)
  - RetailPrice: Decimal (9.4)
  - OccurrenceCount: Integer (number of records for the effective date/time)
- **HistoryRecords** (for INQUIRE with history): Array of history records containing:
  - ChangeDate: Integer (8 digits, YYYYMMDD)
  - ChangeTime: Integer (6 digits, HHMMSS)
  - UserID: String (8 chars)
  - RetailPrice: Decimal (9.4)
  - DeleteFlag: String (1 char, 'D' for deleted, blank otherwise)

**Process Steps**:
1. **Validate Inputs**:
   - Check if `FileGroup` is 'G' or 'Z'; return error 'Invalid file group' if not.
   - Validate `CompanyCode` against `bicont` table; return error 'Invalid company code (ERR0021)' if not found.
   - If `Mode = 'MNT'` and `Operation = 'INQUIRE'`, return error 'Inquiry not allowed in maintenance mode'.
   - Validate `EffectiveDate` using date validation logic (similar to `GSDTEDIT`); return error 'Invalid effective date (ERR0020)' if invalid.
   - Validate `EffectiveTime` for valid hours (00-23) and minutes (00-59); return error 'Invalid effective time (ERR0019)' if invalid.
   - For `CREATE`/`UPDATE`, ensure `RetailPrice` is non-zero and `Region` is provided; return error 'Retail price cannot be zero (ERR0012)' or 'Region required' if invalid.
   - For `CREATE`, check if a record exists in `bbrcsc` with `CompanyCode`, `EffectiveDate`, `EffectiveTime`, `Region`; return error 'Record already exists (ERR0101)' if found.
   - For `UPDATE`/`DELETE`, check if a record exists; return error 'Record does not exist (ERR0102)' if not found.

2. **Apply File Overrides**:
   - Based on `FileGroup`, apply overrides to access `bbrcsc`, `bbrcsch`, `bbndfi`, `bbndfihx`, `bicont`, `gstabl`, and `bbrcscrd` in the appropriate library (e.g., `gbbndfi` for 'G', `zbbndfi` for 'Z').

3. **Process Operation**:
   - **INQUIRE**:
     - Retrieve records from `bbrcsc` matching `CompanyCode`, `EffectiveDate`, and optionally `EffectiveTime`.
     - For each record, fetch the occurrence count by counting matching records in `bbrcscrd` for the same `EffectiveDate` and `EffectiveTime`.
     - If history is requested, retrieve records from `bbndfihx` for the specified key fields.
     - Return an array of records with `EffectiveDate`, `EffectiveTime`, `Region`, `RetailPrice`, and `OccurrenceCount` (for main records) or `ChangeDate`, `ChangeTime`, `UserID`, `RetailPrice`, `DeleteFlag` (for history).
   - **CREATE**:
     - Insert a new record into `bbrcsc` with `CompanyCode`, `EffectiveDate`, `EffectiveTime`, `Region`, and `RetailPrice`.
     - Log the change to `bbrcsch` with `ChangeDate` (current date), `ChangeTime` (current time), `UserID`, `RetailPrice`, and `DeleteFlag` = blank.
     - Return 'SUCCESS'.
   - **UPDATE**:
     - Update the existing record in `bbrcsc` with the new `RetailPrice` for the specified `CompanyCode`, `EffectiveDate`, `EffectiveTime`, and `Region`.
     - Log the change to `bbrcsch` with current date, time, `UserID`, `RetailPrice`, and `DeleteFlag` = blank.
     - Return 'SUCCESS'.
   - **DELETE**:
     - Update the record in `bbrcsc` by setting `RetailPrice` to zero for the specified `CompanyCode`, `EffectiveDate`, `EffectiveTime`, and `Region`.
     - Log the change to `bbrcsch` with current date, time, `UserID`, `RetailPrice` = 0, and `DeleteFlag` = 'D'.
     - Return 'SUCCESS'.

4. **Log History** (for CREATE/UPDATE/DELETE):
   - Write a history record to `bbrcsch` with:
     - `lhco`: `CompanyCode`
     - `lhefdt`: `EffectiveDate`
     - `lheftm`: `EffectiveTime`
     - `lhrprc`: `RetailPrice`
     - `lhchd8`: Current date (YYYYMMDD)
     - `lhchtm`: Current time (HHMMSS)
     - `lhuser`: `UserID`
     - `lhdel`: 'D' for DELETE, blank otherwise.

5. **Return Results**:
   - Return `Status`, `ErrorMessage` (if any), and `Records` or `HistoryRecords` arrays for INQUIRE operations.

**Business Rules**:
- Only valid company codes from `bicont` are allowed.
- Effective dates must be in YYYYMMDD format and pass date validation.
- Effective times must be in HHMM format with valid hours (00-23) and minutes (00-59).
- Retail price must be non-zero for CREATE and UPDATE operations.
- CREATE operations fail if a record with the same key (`CompanyCode`, `EffectiveDate`, `EffectiveTime`, `Region`) already exists.
- UPDATE and DELETE operations fail if the record does not exist.
- All changes (CREATE, UPDATE, DELETE) are logged to the history file for auditability.
- History inquiries retrieve all changes for the specified key fields, sorted by change date and time.
- File overrides ensure data is accessed from the correct library ('G' or 'Z').
- Inquiry mode ('INQ') restricts operations to read-only; maintenance mode ('MNT') allows CREATE, UPDATE, and DELETE.

**Calculations**:
- **Date Conversion**: For history inquiries, convert change date from YYYYMMDD to MMDDYY format for output (e.g., `datymd * 100.0001 = datmdy`, then `%editc(datmdy: 'Y')`).
- **Occurrence Count**: Count the number of records in `bbrcscrd` matching the `EffectiveDate` and `EffectiveTime` to provide the occurrence count for inquiry results.
- **Current Date/Time for History**: Use system date and time (e.g., `JBDT##` for date, `time12` for time) to populate `lhchd8` and `lhchtm` in history records.

**Dependencies**:
- **Tables**:
  - `bicont`: Validates company code.
  - `gstabl`: Provides region reference data.
  - `bbrcsc`: Stores fuel index records (update/add).
  - `bbrcsch`: Stores history records (output).
  - `bbrcsc1`: Input for inquiry (renamed format).
  - `bbrcscrd`: Input for detailed inquiry.
  - `bbndfi`/`bbndfihx`: Used by GB730P for history inquiries.
- **External Programs**:
  - `GSDTEDIT`: Validates dates (input: `p#mdy`, output: `p#cymd`, `p#err`).
  - `QCMDEXC`: Executes file override commands.
  - `QMHSNDPM`/`QMHRMVPM`: Handles error message queuing and removal.

**Error Handling**:
- Return specific error messages for validation failures (e.g., 'ERR0021', 'ERR0012').
- Ensure all database operations are atomic to maintain data integrity.
- Log errors to the message queue for traceability.

---

<xaiArtifact artifact_id="96af39f4-9eea-4348-a184-6c76b4725582" artifact_version_id="b1e553bf-4ae4-4445-a284-47ef94445d1e" title="ManageNationalDieselFuelIndex_Requirements.md" contentType="text/markdown">

# Function Requirement Document: ManageNationalDieselFuelIndex

## Purpose
To manage and inquire about National Diesel Fuel Index records (create, update, delete, inquire) and log changes to a history file without screen interaction.

## Inputs
- **FileGroup**: String (1 char, 'G' or 'Z') – Library group for file overrides.
- **CompanyCode**: Integer (2 digits) – Company identifier.
- **Mode**: String (3 chars, 'INQ' or 'MNT') – Inquiry or maintenance mode.
- **Operation**: String (6 chars, 'INQUIRE', 'CREATE', 'UPDATE', 'DELETE') – Action to perform.
- **EffectiveDate**: Integer (8 digits, YYYYMMDD) – Record effective date.
- **EffectiveTime**: Integer (4 digits, HHMM) – Record effective time.
- **Region**: String (3 chars) – Region code (required for CREATE/UPDATE).
- **RetailPrice**: Decimal (9.4) – Retail price per gallon (required for CREATE/UPDATE).
- **UserID**: String (8 chars) – User performing the operation.

## Outputs
- **Status**: String – 'SUCCESS' or 'ERROR'.
- **ErrorMessage**: String – Error description (e.g., 'Invalid company code').
- **Records** (INQUIRE): Array of {EffectiveDate, EffectiveTime, Region, RetailPrice, OccurrenceCount}.
- **HistoryRecords** (INQUIRE with history): Array of {ChangeDate, ChangeTime, UserID, RetailPrice, DeleteFlag}.

## Process Steps
1. **Validate Inputs**:
   - Check `FileGroup` ('G' or 'Z'); error if invalid.
   - Validate `CompanyCode` in `bicont`; error 'ERR0021' if not found.
   - Ensure `Mode` and `Operation` compatibility; error if 'INQUIRE' in 'MNT' mode.
   - Validate `EffectiveDate` (YYYYMMDD) via date validation; error 'ERR0020' if invalid.
   - Validate `EffectiveTime` (HHMM, 00-23:00-59); error 'ERR0019' if invalid.
   - For CREATE/UPDATE, ensure `RetailPrice` > 0 and `Region` provided; error 'ERR0012' or 'Region required' if invalid.
   - For CREATE, check `bbrcsc` for existing record; error 'ERR0101' if found.
   - For UPDATE/DELETE, check `bbrcsc` for record; error 'ERR0102' if not found.

2. **Apply File Overrides**:
   - Override `bbrcsc`, `bbrcsch`, `bbndfi`, `bbndfihx`, `bicont`, `gstabl`, `bbrcscrd` to 'G' or 'Z' library based on `FileGroup`.

3. **Process Operation**:
   - **INQUIRE**:
     - Fetch records from `bbrcsc` matching `CompanyCode`, `EffectiveDate`, `EffectiveTime`.
     - Calculate occurrence count from `bbrcscrd`.
     - For history, fetch records from `bbndfihx`.
     - Return `Records` or `HistoryRecords`.
   - **CREATE**:
     - Insert record into `bbrcsc` with input fields.
     - Log to `bbrcsch` with current date/time, `UserID`, `DeleteFlag` = blank.
   - **UPDATE**:
     - Update `bbrcsc` record with new `RetailPrice`.
     - Log to `bbrcsch` with current date/time, `UserID`, `DeleteFlag` = blank.
   - **DELETE**:
     - Set `RetailPrice` = 0 in `bbrcsc`.
     - Log to `bbrcsch` with current date/time, `UserID`, `DeleteFlag` = 'D'.

4. **Log History** (CREATE/UPDATE/DELETE):
   - Write to `bbrcsch`: `lhco`, `lhefdt`, `lheftm`, `lhrprc`, `lhchd8` (current date), `lhchtm` (current time), `lhuser`, `lhdel`.

5. **Return Results**:
   - Return `Status`, `ErrorMessage`, and `Records`/`HistoryRecords`.

## Business Rules
- Valid `CompanyCode` required from `bicont`.
- `EffectiveDate` must be valid YYYYMMDD.
- `EffectiveTime` must be valid HHMM.
- `RetailPrice` must be non-zero for CREATE/UPDATE.
- No duplicate records for CREATE.
- Existing record required for UPDATE/DELETE.
- All changes logged to `bbrcsch`.
- History inquiries sorted by change date/time.
- File overrides applied based on `FileGroup`.
- 'INQ' mode is read-only; 'MNT' allows modifications.

## Calculations
- **Date Conversion**: YYYYMMDD to MMDDYY for output (`datymd * 100.0001 = datmdy`).
- **Occurrence Count**: Count `bbrcscrd` records for `EffectiveDate` and `EffectiveTime`.
- **History Date/Time**: Use system date (`JBDT##`) and time (`time12`).

## Dependencies
- **Tables**: `bicont`, `gstabl`, `bbrcsc`, `bbrcsch`, `bbrcsc1`, `bbrcscrd`, `bbndfi`, `bbndfihx`.
- **Programs**: `GSDTEDIT` (date validation), `QCMDEXC` (file overrides), `QMHSNDPM`/`QMHRMVPM` (messages).

## Error Handling
- Return specific error messages (e.g., 'ERR0021', 'ERR0012').
- Ensure atomic database operations.
- Log errors to message queue.

</xaiArtifact>