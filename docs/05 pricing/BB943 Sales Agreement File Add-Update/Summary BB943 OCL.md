Below, I will first identify the use cases implemented by the call stack of RPG programs (`BB943V`, `BB944`, `BB9433`, and `GB730P`) based on the provided source code. These programs collectively form part of the Customer Sales Agreement Maintenance system, handling validation, deletion/reactivation, copying/updating, and history tracking of sales agreements. Then, I will provide a concise Function Requirement Document for each use case, focusing on business requirements, process steps, and necessary calculations, assuming the use cases are implemented as functions that process inputs programmatically rather than through screen interactions.

---

### Use Cases Implemented in the Call Stack

The call stack (`BB943V`, `BB944`, `BB9433`, `GB730P`) supports the Customer Sales Agreement Maintenance system, which manages sales agreement records in the `bicuag` file. Based on the program functionalities, the following use cases are implemented:

1. **Validate Sales Agreement Data** (`BB943V`):
   - Validates input fields for adding or updating a sales agreement record, ensuring data integrity before storage in `bicuag`.
   - Handles "copy to other" validation for new records.

2. **Delete or Reactivate Sales Agreement** (`BB944`):
   - Deletes an active sales agreement record (sets `badel = 'D'`) or reactivates a deleted record (sets `badel = 'A'`).
   - Logs actions to the history file (`bicuagh`).

3. **Copy or Update Sales Agreement** (`BB9433`):
   - Copies an existing sales agreement to new locations, ship-tos, or customer ship-tos, or updates an existing record.
   - Optionally expires the original record and logs actions to `bicuagh`.

4. **Track Sales Agreement History** (`GB730P`):
   - Queries and displays historical changes to sales agreement records (and other related files) from history files like `bicuaghx`.

These use cases collectively cover the creation, validation, modification, deletion, and history tracking of customer sales agreements.

---

### Function Requirement Document

Below, I provide a Function Requirement Document for each use case, describing the purpose, inputs, outputs, process steps, business rules, and calculations (where applicable) in a concise manner. Each use case is treated as a programmatic function that processes inputs without screen interaction.

<xaiArtifact artifact_id="79d2973c-e7d6-4577-9b23-d54a731b8dfa" artifact_version_id="0ceb9076-a663-4f8f-ab0d-282191c4f120" title="SalesAgreementFunctionRequirements.md" contentType="text/markdown">

# Function Requirement Document: Customer Sales Agreement Management

## 1. Validate Sales Agreement Data

**Purpose**: Validates input data for a customer sales agreement to ensure it meets business rules before adding or updating in the database.

**Inputs**:
- **Data Structure** (`CUAGDS`):
  - Company (`cono`, 2-digit numeric)
  - Customer (`cust`, 6-digit numeric)
  - Location (`aloc`, 3-char)
  - Container (`kcntr`, 3-char)
  - Unit of Measure (`unms`, 3-char)
  - All Ship-Tos (`alsh`, 'Y'/'N')
  - Ship-To (`ship`, 3-digit numeric)
  - Product Codes (`s3pr`, array of 10 4-char codes)
  - Purchase Order (`pord`, 15-char)
  - Start Date (`stdt`, 6-digit MMDDYY), Start Time (`sttm`, 4-digit HHMM)
  - End Date (`endt`, 6-digit MMDDYY), End Time (`entm`, 4-digit HHMM)
  - Sequence Number (`seq`, 9-digit numeric)
  - Expire Flag (`s3expr`, 'Y'/'N')
  - Price (`prce`, 9.4 packed), Off-Price (`offp`, 9.4 packed)
  - Freight Code (`frcd`, 1-char)
  - Delivery (`delv`, 1-char)
  - Prepaid (`ppd`, 'P' or blank)
  - Primary Flag (`prim`, 'I'/'M')
  - Min/Max Quantities (`mnqy`, `mxqy`, 7-digit numeric)
  - Other Locations (`s3lo`, array of 10 3-char)
  - Other Ship-Tos (`s3sh`, array of 10 3-char)
  - Other Customer Ship-Tos (`s3cs`, array of 4 9-char)
  - Agreement Password (`kagpw`, 8-char)
  - Add Record Flag (`addrec`, 'Y'/'N')
- **File Group** (`fgrp`, 'G'/'Z')

**Outputs**:
- **Error Message** (`@msg`, 40-char): Error description if validation fails.
- **Cursor Position** (`@pc`, 8-char): Field to highlight for correction.
- **Validation Status**: Boolean indicating success (true) or failure (false).

**Process Steps**:
1. Apply file overrides (`bicont`, `arcust`, `shipto`, `inloc`, `gstabl`, `gscntr1`, `bicua3`, `bicua9`, `bbordh`, `gsprod`) based on `fgrp`.
2. Validate fields:
   - Check `cono` in `bicont`.
   - Verify `cust` in `arcust`.
   - Validate `aloc` in `inloc`.
   - Ensure `kcntr` exists in `gscntr1` and matches type in `gstabl`.
   - Confirm `unms` in `gstabl`.
   - Validate at least one `s3pr` in `gsprod`, no duplicates, fluid products require `kcntr`, non-fluid require blank `kcntr`.
   - Check `pord` in `bbordh` if `s3poor = 'O'`.
   - Ensure `alsh = 'Y'` implies `ship = 0`, `alsh = 'N'` implies valid `ship` in `shipto`.
   - Validate `s3lo`, `s3sh`, `s3cs` for no duplicates and existence in `inloc`, `shipto`, `arcust`.
   - Verify `stdt`, `endt` formats, `sttm`, `entm` (hours 00-23, minutes 00-59), non-zero times, `endt/entm > stdt/sttm`.
   - Check `frcd` ('C'/'P'/'A'/blank), `ppd` ('P'/blank), `prim` ('I'/'M'), `mxqy ≥ mnqy`.
   - Validate `kagpw` in `bicont`, required if `stdt` is >3 days past current date.
3. For `addrec = 'Y'`, check `bicua3` for no duplicates using key (`cono`, `cust`, `aloc`, `kcntr`, `unms`, `ship`, `s3pr`, `pord`, `mnqy`, `mxqy`, `stdt`, `sttm`, `frcd`).
4. For `addrec ≠ 'Y'`, verify `bicua9` has a matching record to update (end date = 12/31/2079 23:59 or no end date).
5. For `addrec = 'Y'` and copy operation, ensure `s3expr = 'Y'`, protect copy fields (`s3lo`, `s3sh`, `s3cs`) during updates.
6. Return error message and cursor position if validation fails, else return success.

**Business Rules**:
- Mandatory fields: `cono`, `cust`, `aloc`, at least one `s3pr`.
- No duplicate product codes, locations, ship-tos, or customer ship-tos.
- Fluid products require valid `kcntr`, non-fluid require blank `kcntr`.
- End date/time must follow start date/time; same start dates allowed if `frcd` differs.
- Password required for start dates >3 days past.
- Copy to other locations/ship-tos only allowed during add operations (F6 or option 3).

**Calculations**:
- Date validation: Ensure `stdt` and `endt` are valid MMDDYY, `sttm` and `entm` are valid HHMM.
- Date comparison: Check `endt/entm > stdt/sttm` using CCYYMMDD format conversion.
- Password check: Compare `kagpw` with `bcagpw` in `bicont`.

---

## 2. Delete or Reactivate Sales Agreement

**Purpose**: Deletes an active sales agreement record or reactivates a deleted one, logging the action to the history file.

**Inputs**:
- **Company** (`cono`, 2-digit numeric)
- **Sequence Number** (`seqn`, 9-digit numeric)
- **File Group** (`fgrp`, 'G'/'Z')
- **Password** (`agpw`, 8-char)
- **Action** (`action`, 'D'/'A'): 'D' for delete, 'A' for reactivate

**Outputs**:
- **Return Flag** (`flag`, 'D'/'A'): Indicates action taken (delete or reactivate)
- **Error Message** (`msg`, 40-char): Error description if action fails

**Process Steps**:
1. Apply file overrides (`gscntr1`, `gstabl`, `shipto`, `bicuag`, `bicuagh`) based on `fgrp`.
2. Chain to `bicuag` using `cono`, `seqn` to retrieve the record.
3. Validate `agpw` against stored password (`baagpw`).
4. If `action = 'D'` and `badel ≠ 'D'`:
   - Set `badel = 'D'`.
   - Update `bicuag`.
   - Write history record to `bicuagh` with user ID, date, time, `hhdel = 'D'`.
   - Set `flag = 'D'`.
5. If `action = 'A'` and `badel = 'D'`:
   - Set `badel = 'A'`.
   - Update `bicuag`.
   - Write history record to `bicuagh` with user ID, date, time, `hhdel = 'A'`.
   - Set `flag = 'A'`.
6. If validation fails or record not found, return error message.

**Business Rules**:
- Record must exist in `bicuag`.
- Valid password required for delete/reactivate.
- Delete only allowed for active records (`badel ≠ 'D'`).
- Reactivate only allowed for deleted records (`badel = 'D'`).
- All actions logged to `bicuagh` with user ID, date, time, and deletion status.

**Calculations**:
- Date/time stamping: Use current date (CCYYMMDD) and time (HHMMSS) for history logging.

---

## 3. Copy or Update Sales Agreement

**Purpose**: Copies a sales agreement to new locations, ship-tos, or customer ship-tos, or updates an existing record, with optional expiration of the original.

**Inputs**:
- **Data Structure** (`parmlist`): Same as `CUAGDS` (see Validate Sales Agreement Data), plus:
  - Add/Update Flag (`addupd`, 'A'/'U')
  - Date Flag (`dateflag`, 1-char)
  - Original Key Fields (`o@cono`, `o@cust`, `o@loc`, `o@cntr`, `o@unms`, `o@ship`, `o@pr01`–`o@pr10`, `o@pord`, `o@mnqy`, `o@mxqy`, `o@std8`, `o@sttm`, `o@frcd`)
- **File Group** (`fgrp`, 'G'/'Z')
- **Contract Number** (`cntn`, variable length)

**Outputs**:
- **Return Flag** (`flag`, 1-char): Indicates success ('S') or failure ('F')
- **Error Message** (`msg`, 40-char): Error description if action fails

**Process Steps**:
1. Apply file overrides (`bicont`, `bicuag`, `bicua3`, `bicuagh`) based on `fgrp`.
2. If `expr = 'Y'`:
   - Chain to `bicuag` using `cono`, `seqn`.
   - If `baend8 > std8` or `baend8 = 0`, set `baend8 = std8 - 1 minute`, update `bicuag`, write to `bicuagh`.
3. For each valid entry in `locs`, `shp`, or `cshp` arrays (if populated):
   - Update `wkcuag` with new `baloc`, `baship`, or `bacust/baship`.
   - Generate new sequence number (`w$sq`).
   - Check `bicua3` for no duplicates using key (`cono`, `cust`, `loc`, `cntr`, `unms`, `ship`, `pr01`–`pr10`, `pord`, `mnqy`, `mxqy`, `std8`, `sttm`, `frcd`).
   - Write new record to `bicuag` with `bacrdt`, `bacrtm`, `baludt`, `balutm`, `bacntn`.
   - Write history record to `bicuagh`.
4. If no arrays populated, add/update single record in `bicuag`.
5. Return success or error message.

**Business Rules**:
- Original record expires only if `expr = 'Y'` and `baend8 > std8` or `baend8 = 0`.
- Same start dates allowed if `frcd` differs.
- No duplicate records in `bicua3`.
- All actions logged to `bicuagh` with user ID, date, time.
- Min/max quantities, expanded price fields (9.4 packed), and contract number supported.

**Calculations**:
- Date adjustment: Set `baend8 = std8 - 1 minute` for expiration.
- Sequence number generation: Increment `w$sq` for each copied record.
- Date/time stamping: Use current date (CCYYMMDD) and time (HHMMSS) for `bacrdt`, `bacrtm`, `baludt`, `balutm`.

---

## 4. Track Sales Agreement History

**Purpose**: Retrieves and returns historical changes to customer sales agreement records.

**Inputs**:
- **Company** (`cono`, 2-digit numeric, optional)
- **Customer** (`cust`, 6-digit numeric, optional)
- **File Type** (`filetype`, string, e.g., 'bicuag')
- **File Group** (`fgrp`, 'G'/'Z')

**Outputs**:
- **History Records**: Array of records containing:
  - User ID (`hhuser`, 8-char)
  - Change Date (`hhchd8`, 8-digit CCYYMMDD)
  - Change Time (`hhchtm`, 6-digit HHMMSS)
  - Deletion Status (`hhdel`, 'D'/'A')
  - Agreement Fields (`hhcono`, `hhcust`, `hhloc`, etc.)
- **Error Message** (`msg`, 40-char): Error description if query fails

**Process Steps**:
1. Apply file overrides (`bicuaghx`, etc.) based on `fgrp`.
2. Query `bicuaghx` based on `cono`, `cust`, or `filetype`.
3. Format dates (`hhchd8` to MMDDYY) and populate history records with user, date, time, and agreement fields.
4. Return history records or error message if no records found.

**Business Rules**:
- Supports querying history for `bicuag` and other files (e.g., `bbordh`, `bbcfsd`).
- Cannot specify both system and program IDs simultaneously.
- Supports expanded fields (`BAPRCE`, `BAOFFP`, `BKFSPC`) and new fields (`BFTSEQ`, `BAPOOR`).

**Calculations**:
- Date conversion: Transform `hhchd8` (CCYYMMDD) to MMDDYY for output.

</xaiArtifact>

---

### Notes
- The use cases are derived from the explicit functionalities of `BB943V` (validation), `BB944` (delete/reactivate), `BB9433` (copy/update), and `GB730P` (history tracking).
- Each function assumes programmatic input processing, eliminating screen-based interactions (e.g., `bb944d` display file in `BB944` is replaced by direct parameter validation).
- The business rules and calculations are aligned with the program logic and revisions (e.g., `jk04` for expiration logic, `mg12` for freight code differentiation).
- If additional details or source code for `BB943P` or `BB943` are provided, I can refine the integration points or add more use cases (e.g., main entry point logic).

Let me know if you need further clarification, additional use cases, or modifications to the document!