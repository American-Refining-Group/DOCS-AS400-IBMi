### List of Use Cases Implemented by the AR915P Call Stack

The `AR915P` program, along with its called programs `AR915` and `AR9154`, implements the following use case within the Customer Master Information system:

1. **Manage Customer Form Type Contacts**:
   - **Description**: This use case allows users to view, create, update, delete, or restore customer form type contact records. It provides an interactive interface to manage contact details (e.g., form type, contact name, email, flags) for a specific company and customer, supporting both maintenance (`MNT`) and inquiry (`INQ`) modes.
   - **Components**:
     - `AR915P`: Displays a subfile listing contact records, allowing users to select records for viewing, updating, deleting, or creating new ones.
     - `AR915`: Handles the creation or update of a single contact record, with validation of form type, contact name, email, and flags.
     - `AR9154`: Manages the deletion or restoration of a contact record via a confirmation window.

This is the primary use case, as the programs are tightly integrated to manage customer form type contacts comprehensively.

---

### Function Requirement Document: Manage Customer Form Type Contacts



# Function Requirement Document: Manage Customer Form Type Contacts

## Purpose
The `Manage Customer Form Type Contacts` function allows the system to retrieve, create, update, delete, or restore customer form type contact records for a specified company and customer without interactive screen input. It processes input parameters to perform the requested operation and returns a status flag to indicate success or failure.

## Inputs
- **Company Code (`cono`)**: 2-character code identifying the company (required).
- **Customer Code (`cust`)**: Customer identifier (required).
- **Sequence Number (`seq#`)**: Numeric identifier for the contact record; 0 for new records (required).
- **Mode (`mode`)**: Operation mode, either `MNT` (maintenance) or `INQ` (inquiry) (required).
- **File Group (`fgrp`)**: File group identifier, `Z` or `G`, for database overrides (required).
- **Operation (`oper`)**: Action to perform: `CREATE`, `UPDATE`, `DELETE`, `RESTORE`, or `VIEW` (required).
- **Form Type Code (`fmty`)**: Code for the form type (required for `CREATE`, `UPDATE`).
- **Contact Name (`cntc`)**: Name of the contact person (required for `CREATE`, `UPDATE`).
- **Email Address (`emla`)**: Email address for the contact (optional for `CREATE`, `UPDATE` if `fmyn = 'N'`).
- **Send Original Flag (`fmyn`)**: `Y` or `N`, indicates if the original form is sent (required for `CREATE`, `UPDATE`).
- **Send Reprint Flag (`rpyn`)**: `Y` or `N`, indicates if reprints are sent (required for `CREATE`, `UPDATE`).
- **Send by Mail Flag (`mlyn`)**: `Y` or `N`, indicates if sent by mail (required for `CREATE`, `UPDATE`).
- **Send Back Terms Flag (`bkyn`)**: `Y` or `N`, indicates if back terms are sent (required for `CREATE`, `UPDATE`).

## Outputs
- **Return Flag (`flag`)**: Status indicator:
  - `1`: Successful creation or update.
  - `D`: Successful deletion.
  - `A`: Successful restoration.
  - `E`: Error (e.g., validation failure or record not found).
- **Error Message (`errmsg`)**: Description of any error encountered (e.g., "Invalid form type code").

## Process Steps
1. **Validate Inputs**:
   - Verify `cono` exists in `bicont` and is not deleted; else, return `flag = 'E'`, `errmsg = "Invalid company code"`.
   - Verify `cust` exists in `arcust` and is not deleted; else, return `flag = 'E'`, `errmsg = "Invalid customer code"`.
   - For `CREATE`, `UPDATE`:
     - Verify `fmty` exists in `gstabl` with type `FRMTYP` and is not deleted; else, return `flag = 'E'`, `errmsg = "Invalid form type code"`.
     - Verify `cntc` is non-blank; else, return `flag = 'E'`, `errmsg = "Contact name required"`.
     - If `emla` is non-blank, validate via `VALMAILID`; if invalid, return `flag = 'E'`, `errmsg = "Invalid email address"`.
     - If `fmyn = 'Y'` or blank, `emla` must be non-blank; else, return `flag = 'E'`, `errmsg = "Email address required"`.
     - Verify `fmyn`, `rpyn`, `mlyn`, `bkyn` are `Y` or `N`; else, return `flag = 'E'`, `errmsg = "Invalid flag value"`.
   - For `UPDATE`, `DELETE`, `RESTORE`, `VIEW`, verify `seq#` exists in `arcufm` using `cono` and `seq#`; else, return `flag = 'E'`, `errmsg = "Record not found"`.

2. **Apply File Overrides**:
   - Use `fgrp` (`Z` or `G`) to override database files (`arcust`, `bicont`, `arcufm`, `gstabl`) to the appropriate library (e.g., `garcust` or `zarcust`).

3. **Process Operation**:
   - **VIEW**:
     - Retrieve record from `arcufm` using `cono` and `seq#`.
     - Return record fields (`fmty`, `cntc`, `emla`, `fmyn`, `rpyn`, `mlyn`, `bkyn`, `del`) and `flag = '1'`.
   - **CREATE**:
     - If `seq# = 0`, retrieve next sequence number from `bicont.bcseqn`, increment until unique in `arcufm`, and update `bicont.bcseqn`.
     - Create new `arcufm` record with `cono`, `seq#`, `cust`, `fmty`, `cntc`, `emla`, `fmyn`, `rpyn`, `mlyn`, `bkyn`, and `del = 'A'`.
     - Set `flag = '1'`.
   - **UPDATE**:
     - Update existing `arcufm` record with provided `fmty`, `cntc`, `emla`, `fmyn`, `rpyn`, `mlyn`, `bkyn`, retaining `del` status.
     - Set `flag = '1'`.
   - **DELETE**:
     - If `del != 'D'`, update `arcufm` record to set `del = 'D'`.
     - Set `flag = 'D'`.
   - **RESTORE**:
     - If `del = 'D'`, update `arcufm` record to set `del = 'A'`.
     - Set `flag = 'A'`.

4. **Return Results**:
   - Return `flag` and `errmsg` (if applicable).
   - For `VIEW`, include retrieved record fields.

## Business Rules
1. **Company and Customer Validation**:
   - `cono` must exist in `bicont` and not be deleted.
   - `cust` must exist in `arcust` and not be deleted.

2. **Form Type Validation**:
   - `fmty` must exist in `gstabl` with type `FRMTYP` and not be deleted.

3. **Contact Name**:
   - `cntc` must be non-blank for `CREATE` and `UPDATE`.

4. **Email Validation**:
   - If `emla` is provided, it must be valid per `VALMAILID`.
   - `emla` is optional if `fmyn = 'N'`; otherwise, it is required for `CREATE` and `UPDATE`.

5. **Flag Validation**:
   - `fmyn`, `rpyn`, `mlyn`, `bkyn` must be `Y` or `N`.

6. **Sequence Number for CREATE**:
   - If `seq# = 0`, generate a unique sequence number by incrementing `bicont.bcseqn` until no conflict exists in `arcufm`.

7. **Deletion and Restoration**:
   - Deletion sets `del = 'D'` only if the record is active (`del != 'D'`).
   - Restoration sets `del = 'A'` only if the record is deleted (`del = 'D'`).

8. **Inquiry Mode**:
   - In `INQ` mode, only `VIEW` operation is allowed; no database updates occur.

## Calculations
- **Sequence Number Generation**:
  - Retrieve `bcseqn` from `bicont` for the given `cono`.
  - Increment `bcseqn` and check `arcufm` for uniqueness using `cono` and `seq#`.
  - Repeat until a unique `seq#` is found, then update `bicont.bcseqn`.

## Database Tables
- **arcust**: Validates customer code (`cust`) and retrieves customer name.
- **bicont**: Validates company code (`cono`), retrieves company name, and manages sequence numbers (`bcseqn`).
- **arcufm**: Stores contact records; used for retrieval, creation, update, deletion, or restoration.
- **gstabl**: Validates form type code (`fmty`) with type `FRMTYP`.

## External Programs
- **VALMAILID**: Validates email addresses for `CREATE` and `UPDATE` operations.
- **QCMDEXC**: Applies file overrides based on `fgrp`.

