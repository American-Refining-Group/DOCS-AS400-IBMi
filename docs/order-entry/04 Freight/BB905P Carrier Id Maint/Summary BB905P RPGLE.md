The call stack for the Billing and Invoicing system consists of four RPG programs: `BB905`, `BB9054`, `BB9055`, and `BB9059`. These programs collectively handle various aspects of carrier code management and reporting. Below is a comprehensive list of use cases implemented by these programs, followed by a function requirement document for each use case, reimagined as a large function that processes inputs programmatically rather than through screen interactions.

---

### List of Use Cases Implemented in the Call Stack

Based on the provided RPG programs (`BB905`, `BB9054`, `BB9055`, `BB9059`), the following use cases are implemented:

1. **Carrier Code Entry Maintenance (BB905)**:
   - **Description**: Allows the creation or update of carrier ID records, including validation of inputs and synchronization with related tables. Handles both maintenance (`MNT`) and inquiry (`INQ`) modes.
   - **Details**: Validates carrier description (must not be blank), temporarily allows multiple carrier IDs to share the same Fuel Facs ID (per revision `JB02`), populates creation date for new records (per `JK01`), and supports replacement carrier ID capture by calling `BB9059`. Updates `bbcaid` and, for company `10`, `gstabl`.

2. **Carrier Code Inactivation/Reactivation (BB9054)**:
   - **Description**: Manages the inactivation or reactivation of existing carrier ID records.
   - **Details**: Toggles the status (`cidel`) between active (`A`) and inactive (`I`), updates `bbcaid` and `gstabl` (for company `10`), and allows inactivation even if the carrier is linked to a vendor (per revision `jb01`). Uses F22 for reactivation and F23 for inactivation.

3. **Carrier Code Listing Report (BB9055)**:
   - **Description**: Generates a printed report listing all carrier ID records for a specified file group.
   - **Details**: Reads all records from `bbcaid`, formats a report with headers and detail lines (company number, carrier ID, carrier name, EIN, Fuel Facs number, status), and handles page overflows. Configures output to be held and saved in the job’s output queue.

4. **Capture Replacing Carrier ID (BB9059)**:
   - **Description**: Captures a "From" carrier ID when creating a new carrier ID, storing the relationship in `bbfx62w`.
   - **Details**: Validates the "From" carrier ID (must exist in `bbcaid` and not be linked to a vendor in `apveny`), allows blank entries, and sends confirmation messages to users `JLBIT` and `CAPO` upon successful update.

---

### Function Requirement Documents

Each use case is reimagined as a large function that takes inputs programmatically and returns outputs, focusing on business requirements and calculations where applicable. The documents are concise, detailing inputs, outputs, process steps, and business rules.

<xaiArtifact artifact_id="e675d4da-9124-45d2-b628-db952608588a" artifact_version_id="8e86a941-ed18-4259-8a9b-dad83b3b1762" title="Carrier_Code_Management_Requirements.md" contentType="text/markdown">

# Function Requirement Documents for Carrier Code Management

## 1. Carrier Code Entry Maintenance Function

### Purpose
Create or update carrier ID records in the `bbcaid` and `gstabl` tables, validating inputs and handling replacement carrier relationships.

### Inputs
- **Company Number** (`p$co`): Integer, company identifier.
- **Carrier ID** (`p$caid`): String, carrier identifier.
- **Mode** (`p$mode`): String (`MNT` for maintenance, `INQ` for inquiry).
- **File Group** (`p$fgrp`): String (`Z` or `G` for database overrides).
- **Carrier Name** (`cicanm`): String, carrier description.
- **EIN** (`ciein`): Numeric, Employer Identification Number.
- **Fuel Facs ID** (`ciffid`): Numeric, Fuel Facility ID.
- **Delete Flag** (`cidel`): String (`A` for active, `I` for inactive).

### Outputs
- **Return Flag** (`p$flag`): String (`1` for success, `2` for no creation, `E` for error).
- **Error Messages**: Array of strings, validation errors if any.

### Process Steps
1. **Validate Inputs**:
   - Ensure `p$co` exists in `bicont`.
   - In `MNT` mode, ensure `cicanm` is not blank (else return error `ERR0012`).
   - (Disabled per `JB02`) Ensure `ciffid` is unique in `bbcaid1` unless it matches `p$caid`.
2. **Check Record Existence**:
   - Chain to `bbcaid` using `p$co` and `p$caid`.
   - If found, mark as existing; else, treat as new.
3. **Handle Replacement Carrier**:
   - If new record, call `CaptureReplacingCarrierID` function with `p$co`, `p$caid`, `p$fgrp`.
   - If returned `p$flag = '2'`, return error `ERR0000` ("New Carrier Not Created").
4. **Update Database**:
   - For existing records in `MNT` mode:
     - If fields changed, update `bbcaidpf` with `cicanm`, `ciein`, `ciffid`, `cidel`.
     - Set `p$flag = '1'`.
   - For new records:
     - Set creation date (`cidate`) to current date (per `JK01`).
     - Write to `bbcaidpf` with `p$co`, `p$caid`, `cicanm`, `ciein`, `ciffid`, `cidel`.
     - Set `p$flag = '1'`.
   - For company `10`, update `gstabl`:
     - Chain to `gstabl` with `type = 'BBCAID'` and `p$caid`.
     - If found, update `tbdel = cidel`, `tbdesc = cicanm`, `tbein = ciein`.
     - If not found, write new record with `tbtype = 'BBCAID'`, `tbcode = p$caid`, `tbdel = cidel`, `tbdesc = cicanm`, `tbein = ciein`.
5. **Return Outputs**: Return `p$flag` and any error messages.

### Business Rules
- **Mode Restriction**: In `INQ` mode, no updates are allowed; return existing record data.
- **Validation**:
  - Carrier name must not be blank in `MNT` mode.
  - (Disabled per `JB02`) Fuel Facs ID must be unique unless updating the same carrier.
- **Creation Date**: Set `cidate` for new `bbcaid` records using the current date.
- **Replacement**: New records require checking for a replacement carrier via `CaptureReplacingCarrierID`.
- **GSTABL Sync**: For company `10`, synchronize `gstabl` with `bbcaid` updates.

---

## 2. Carrier Code Inactivation/Reactivation Function

### Purpose
Toggle the active/inactive status of a carrier ID in `bbcaid` and `gstabl` tables.

### Inputs
- **Company Number** (`p$co`): Integer, company identifier.
- **Carrier ID** (`p$caid`): String, carrier identifier.
- **File Group** (`p$fgrp`): String (`Z` or `G` for database overrides).
- **Action** (`action`): String (`INACTIVATE` or `REACTIVATE`).

### Outputs
- **Return Flag** (`p$flag`): String (`A` for reactivated, `I` for inactivated, `E` for error).
- **Error Messages**: Array of strings, validation errors if any.

### Process Steps
1. **Validate Inputs**:
   - Ensure `p$co` and `p$caid` exist in `bbcaid` (else return error).
2. **Check Current Status**:
   - Chain to `bbcaid` using `p$co` and `p$caid`.
   - If `action = 'REACTIVATE'`, ensure `cidel = 'I'` (else return error).
   - If `action = 'INACTIVATE'`, ensure `cidel ≠ 'I'` (else return error).
3. **Update Database**:
   - For `REACTIVATE`: Set `cidel = 'A'`, update `bbcaidpf`, set `p$flag = 'A'`.
   - For `INACTIVATE`: Set `cidel = 'I'`, update `bbcaidpf`, set `p$flag = 'I'`.
   - For company `10`, update `gstabl`:
     - Chain to `gstabl` with `type = 'BBCAID'` and `p$caid`.
     - If found, update `tbdel = p$flag`.
4. **Return Outputs**: Return `p$flag` and any error messages.

### Business Rules
- **Status Transition**:
  - Only inactive records (`cidel = 'I'`) can be reactivated.
  - Only active records (`cidel ≠ 'I'`) can be inactivated.
- **Vendor Linkage**: (Disabled per `jb01`) Inactivation is allowed even if the carrier is linked to a vendor in `apveny`.
- **GSTABL Sync**: For company `10`, synchronize `tbdel` in `gstabl` with `cidel`.

---

## 3. Carrier Code Listing Report Function

### Purpose
Generate a report of all carrier ID records for a specified file group.

### Inputs
- **File Group** (`p$fgrp`): String (`Z` or `G` for database overrides).

### Outputs
- **Report Data**: Array of records containing company number, carrier ID, carrier name, EIN, Fuel Facs number, and status.
- **Report Metadata**: Object with job name, program name, user, date, time, and file group.

### Process Steps
1. **Initialize Report**:
   - Set report header with "American Refining Group", "Carrier Id Listing By Co#/Carrier Id", job name, program name, user, current date, time, and file group.
   - Set column headings: "Co", "Carr Id", "Carrier Name", "Ein #", "Fuel Facs #", "Del".
2. **Read Records**:
   - Sequentially read all records from `bbcaid` (overridden to `gbbcaid` or `zbbcaid` based on `p$fgrp`).
3. **Format Report**:
   - For each record, output:
     - `cico` (company number).
     - `cicaid` (carrier ID).
     - `cicanm` (carrier name).
     - `ciein` (EIN, 4 decimal places).
     - `ciffid` (Fuel Facs number, 4 decimal places).
     - `cidel` (delete/inactive flag).
   - Track page breaks at line 62, reprinting headers as needed.
4. **Return Outputs**: Return the formatted report data and metadata.

### Business Rules
- **Report Scope**: Include all `bbcaid` records without filtering.
- **Formatting**: Use 68 lines, 164 characters, 8 lines per inch, 15 characters per inch, with overflow at line 62.
- **Output**: Report is held and saved in the job’s output queue.

---

## 4. Capture Replacing Carrier ID Function

### Purpose
Capture a "From" carrier ID for a new carrier ID, storing the relationship in `bbfx62w` and notifying users.

### Inputs
- **Company Number** (`p$co`): Integer, company identifier.
- **To Carrier ID** (`p$caid`): String, new carrier ID.
- **File Group** (`p$fgrp`): String (`Z` or `G` for database overrides).
- **From Carrier ID** (`w$cifr`): String, optional replacement carrier ID.

### Outputs
- **Return Flag** (`p$flag`): String (`1` for success, `2` for cancellation).
- **Error Messages**: Array of strings, validation errors if any.

### Process Steps
1. **Validate Inputs**:
   - If `w$cifr` is blank, return `p$flag = '1'` without updating `bbfx62w`.
   - Chain to `bbcaid` using `p$co` and `w$cifr`. If not found, return error `ERR0000` ("Invalid Carrier ID").
   - Check `apveny` using `p$co` and `w$cifr`. If found, return error `ERR0000` ("Code [w$cifr] is linked to a vendor for ARGLMS. Contact IT to convert.").
2. **Update Database**:
   - If valid, write to `bbfx62w` with `w1cifr = w$cifr` and `w1cito = p$caid`.
   - Set `p$flag = '1'`.
3. **Notify Users**:
   - Send messages: "BBFX62W Entry Created. From Carrier [w$cifr] To Carrier [p$caid].)" to users `JLBIT` and `CAPO`.
4. **Return Outputs**: Return `p$flag` and any error messages.

### Business Rules
- **Optional Input**: Blank `w$cifr` is allowed, skipping `bbfx62w` update.
- **Validation**:
  - `w$cifr` must exist in `bbcaid`.
  - `w$cifr` must not be linked to a vendor in `apveny`.
- **Notification**: Successful updates trigger messages to `JLBIT` and `CAPO`.

</xaiArtifact>

---

### Summary

The call stack (`BB905`, `BB9054`, `BB9055`, `BB9059`) implements four distinct use cases: carrier code entry maintenance, inactivation/reactivation, listing report generation, and replacement carrier ID capture. Each use case has been reimagined as a programmatic function that processes inputs directly, focusing on business requirements and calculations. The functions validate inputs, update relevant tables (`bbcaid`, `gstabl`, `bbfx62w`), and enforce rules such as unique carrier descriptions, status transitions, and vendor linkage checks (some disabled per revisions). The report function generates a comprehensive listing, and notifications are sent for replacement carrier entries. These functions ensure data integrity and synchronization across related tables while providing clear outputs for downstream processing.