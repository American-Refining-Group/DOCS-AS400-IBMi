The call stack consists of three programs: `BB955.ocl36` (OCL), `BB955AC.clp` (CLP), and `BB955.rpgle` (RPGLE), which together implement the "Rack Price File Maintenance" functionality for the Bradford Order Entry/Invoices system on an IBM AS/400 (iSeries) platform. Below, I will identify the use cases implemented by this call stack and then provide a concise function requirements document for the primary use case, reimagined as a non-interactive function that processes inputs programmatically.

### List of Use Cases Implemented

Based on the analysis of the provided programs (`BB955.ocl36`, `BB955AC.clp`, and `BB955.rpgle`), the call stack supports the following use cases for maintaining rack price records:

1. **Create New Rack Price Records**:
   - **Description**: Allows users to add new pricing records for products, specifying attributes like company, location, product, container, unit of measure, effective date, quantities, and prices.
   - **Details**: Users enter data via the subfile interface (`sfl1` in `BB955.rpgle`). The program validates inputs against master files (`bicont`, `gsprod`, etc.) and writes new records to `bbprce` and history to `bbprceh`.
   - **Trigger**: Initiated through the subfile interface when users enter new quantities (`n$qt`) and prices (`n$pr`) and press Enter or F7 to accept changes.

2. **Update Existing Rack Price Records**:
   - **Description**: Enables modification of existing rack price records, such as updating quantities, prices, or effective dates, with an option to toggle update permissions (F6).
   - **Details**: Controlled by `s1alwupd` (F6 toggles allow/prevent updates). Changes are validated (`sf1edt`) and updated in `bbprce` with history logged to `bbprceh`.
   - **Trigger**: Users modify existing subfile records (`s1qt`, `s1pr`) and submit changes via Enter or F7.

3. **Delete Rack Price Records**:
   - **Description**: Allows users to mark rack price records as deleted, with an option to include or exclude deleted records from display (F8).
   - **Details**: Handled via F23 (`sf1del` subroutine), which marks records as deleted in `bbprce` and logs the action in `bbprceh`. Inactive records (`GSCTUM`, `GSCTWT`, `GSUMCV`) are treated as deleted (`JB05`).
   - **Trigger**: Users select a record and press F23 to delete.

4. **Restore Deleted Rack Price Records**:
   - **Description**: Permits users to restore previously deleted rack price records.
   - **Details**: Also handled via F23 (`sf1del` subroutine), which reverses the deletion flag in `bbprce` and logs the restoration in `bbprceh`.
   - **Trigger**: Users select a deleted record (when `s1showdel = *on` via F8) and press F23 to restore.

5. **View Rack Price Records**:
   - **Description**: Displays existing rack price records for inquiry, with options to filter by company, location, product, container, unit of measure, or status.
   - **Details**: The subfile (`sfl1`) displays records from `bbprce`, with positioning fields (`r$cono`, `r$loc`, etc.) and toggle for deleted records (F8). Users can view records in folded or unfolded mode (`sfmod1`).
   - **Trigger**: Initiated on program startup or via F5 (refresh).

6. **Prompt for Valid Input Values**:
   - **Description**: Provides lookup functionality to select valid values for fields like company, location, product, container, unit of measure, or product class.
   - **Details**: Triggered by F4, calling external programs (`LBICONT`, `LINLOC`, `LGSPROD`, etc.) to display valid options from master files.
   - **Trigger**: Users press F4 on a control or subfile field.

7. **View Customer History for Rack Price Records**:
   - **Description**: Allows users to view historical changes for a selected rack price record.
   - **Details**: Triggered by F9, calling `GB730P` with parameters (`x$custhist`) to display history from `bbprceh`.
   - **Trigger**: Users select a subfile record and press F9.

### Function Requirements Document

Below is a function requirements document for the primary use case, **Create/Update Rack Price Records**, reimagined as a non-interactive function that processes inputs programmatically rather than through a screen interface. This function consolidates the core functionality of creating and updating rack price records, as these are the most critical operations in the call stack. The document focuses on business requirements, process steps, and necessary calculations, presented concisely.

<xaiArtifact artifact_id="fb414278-d74d-474a-9c1b-71d76c3d6867" artifact_version_id="edb6f53c-fb69-433f-bdca-7fb2fca0d84d" title="RackPriceMaintenanceFunction.md" contentType="text/markdown">

# Rack Price Maintenance Function Requirements

## Purpose
The `RackPriceMaintenance` function creates or updates rack price records for products in the Bradford Order Entry/Invoices system, ensuring accurate pricing data for inventory and sales processes.

## Inputs
- **File Group (`fgrp`)**: Char(1), 'G' (production) or 'Z' (development/test), determines source library (`DATA` or `DATADEV`).
- **Company Code (`cono`)**: Char(2), identifies the company.
- **Location Code (`loc`)**: Char(3), specifies the pricing location.
- **Product Code (`prod`)**: Char(4), identifies the product.
- **Container Code (`cntr`)**: Char(3), specifies the container type.
- **Unit of Measure (`unms`)**: Char(3), defines the unit of measure.
- **Product Class (`prcl`)**: Char(3), categorizes the product.
- **Effective Date (`edat`)**: Num(8,0), format YYYYMMDD, when pricing takes effect.
- **Quantities (`qt`)**: Array of Num(7,0)[4], quantities in ascending order, last must be 9999999.
- **Prices (`pr`)**: Array of Num(9,4)[4], corresponding prices in U.S. dollars.
- **Status (`stat`)**: Char(3), 'ALL', 'CUR', 'FUT', or 'CF' (current & future).
- **Inactive Flag (`inac`)**: Char(1), blank (active), 'I', or 'B' (inactive).
- **No Rack Price Flag (`rkrq`)**: Char(1), blank or 'N' (no rack price).
- **Division (`dvsn`)**: Char(1), blank (all), 'R' (refinery), 'L' (blended lube).
- **Operation (`op`)**: Char(1), 'C' (create) or 'U' (update).
- **User ID (`usr`)**: Char(10), identifies the user making changes.

## Outputs
- **Status Code**: Char(1), 'S' (success), 'E' (error).
- **Error Message**: Char(80), describes validation errors, if any.
- **History Record ID**: Char(10), unique identifier for the history record written to `bbprceh`.

## Process Steps
1. **Validate Inputs**:
   - Verify `cono` exists in `bicont`.
   - Verify `loc` exists in `inloc`.
   - Verify `prod` exists in `gsprod`.
   - Verify `cntr` exists in `gscntr1`.
   - Verify `unms` exists in `gstabl` with `tbtype = 'BIUNMS'`.
   - Verify `prcl` exists in `gstabl` with `tbtype = 'PRODCL'`.
   - Ensure `stat` is 'ALL', 'CUR', 'FUT', or 'CF'.
   - Ensure `inac` is blank, 'I', or 'B'.
   - Ensure `rkrq` is blank or 'N'.
   - Ensure `dvsn` is blank, 'R', or 'L'.
   - Validate `qt` array: non-zero if `pr` is non-zero, in ascending order, last `qt` = 9999999.
   - Validate `pr` array: non-zero if corresponding `qt` is non-zero.
   - Check `edat` is a valid date (YYYYMMDD) using `GSDTEDIT`.
   - For create (`op = 'C'`), ensure no existing record in `bbprce` with same `cono`, `loc`, `prod`, `cntr`, `unms`, `edat`.
   - Return error message and status 'E' if any validation fails.

2. **Prepare Work File**:
   - Clear `QTEMP/BB955W` or create it from `DATA` (if `fgrp = 'G'`) or `DATADEV` (if `fgrp ≠ 'G'`) using `CRTDUPOBJ` with no constraints or triggers.
   - Write input data to `BB955W` for temporary storage.

3. **Update Rack Price File**:
   - If `op = 'C'`, write new record to `bbprce` with input fields.
   - If `op = 'U'`, update existing record in `bbprce` matching `cono`, `loc`, `prod`, `cntr`, `unms`, `edat`.
   - Fields updated: `qt(1-4)`, `pr(1-4)`, `stat`, `inac`, `rkrq`, `dvsn`.

4. **Log to History File**:
   - Create a history record in `bbprceh` with:
     - `ihcono` = `cono`, `ihloc` = `loc`, `ihprod` = `prod`, `ihcntr` = `cntr`, `ihunms` = `unms`, `ihprce` = `pr`, `ihedat` = `edat`.
     - `ihuser` = `usr`, `ihchdt` = current date, `ihchtm` = current time.
     - `ihdel` = 'A' (active) or existing deletion flag for updates.
   - Generate unique history record ID.

5. **Return Results**:
   - Return status 'S', empty error message, and history record ID on success.
   - Return status 'E' and error message on failure.

## Business Rules
- **Data Validation**:
  - All codes (`cono`, `loc`, `prod`, `cntr`, `unms`, `prcl`) must exist in respective master files.
  - Quantities must be in ascending order; last quantity must be 9999999.
  - Prices cannot be zero if corresponding quantity is non-zero, and vice versa.
  - Duplicate pricing records for the same `cono`, `loc`, `prod`, `cntr`, `unms`, `edat` are prohibited.
  - Additional locations cannot match the primary location.

- **File Management**:
  - Use `DATA` library for `fgrp = 'G'`, `DATADEV` for `fgrp ≠ 'G'`.
  - Work file `BB955W` is isolated in `QTEMP` and cleared or created per execution.
  - All changes are logged to `bbprceh` with user, date, and time stamps.

- **Error Handling**:
  - Return descriptive error messages for invalid inputs (e.g., "Invalid Company", "Cannot Be Zero If Price Is Entered").
  - Handle file creation errors (`CPF3142`, `CPF5813`, `CPF7302`) gracefully.

## Calculations
- **Date Validation**: Use `GSDTEDIT` to validate `edat` (YYYYMMDD format).
- **Timestamp Generation**: Capture current date (`ps#mdy`) and time (`ps#hms`) from program status data structure for `bbprceh` logging.
- **Quantity/Price Validation**:
  - For each `qt[i]` and `pr[i]` (i=1 to 4):
    - If `qt[i] > 0`, then `pr[i] > 0`.
    - If `pr[i] > 0`, then `qt[i] > 0`.
    - `qt[i] < qt[i+1]` for i=1 to 3.
    - `qt[4] = 9999999`.

## Dependencies
- **Files**:
  - Input: `bicont`, `inloc`, `gsprod`, `gscntr1`, `gstabl`, `bbprcw`.
  - Update: `bbprce`, `BB955W` (in `QTEMP`).
  - Output: `bbprceh`.
- **External Program**: `GSDTEDIT` for date validation.

</xaiArtifact>

### Additional Notes
- **Use Case Focus**: The function requirements document focuses on the create/update use case, as it encapsulates the core functionality of rack price maintenance. Other use cases (delete, restore, view, prompt, history) are secondary and typically require user interaction, making them less suitable for a non-interactive function.
- **Assumptions**: The function assumes all inputs are provided programmatically, bypassing the subfile interface. Validation logic is derived from `sf1edt` and related subroutines in `BB955.rpgle`.
- **Truncated Code**: The `BB955.rpgle` code was truncated, but key validations and processes were inferred from available logic and error messages in the `com` array.
- **No Search Performed**: The analysis is based on the provided code files, as requested. If further details are needed (e.g., `GSDTEDIT` specifications), I can search or analyze additional files.

If you need a function requirements document for another use case (e.g., delete or history inquiry) or further clarification, let me know!