### List of Use Cases Implemented by the AP910 Program Suite

The `AP910` program suite, comprising `AP910.rpgle` (vendor master maintenance/inquiry) and `AP9104.rpgle` (vendor master delete/restore), called via `AP910P` from the OCL program `AP910P99.ocl36.txt`, implements the following use cases in the accounts payable system on IBM AS/400 or IBM i:

1. **Vendor Master Maintenance**:
   - **Description**: Allows users to create, update, or view vendor master records in the `APVEND` file, with support for 1099 processing. Includes validations for mandatory fields, reference data, and 1099-specific requirements (e.g., IRS Name Control, EIN, Box 1).
   - **Program**: `AP910.rpgle`
   - **Details**: Supports maintenance (`MNT`) mode for adding/updating vendor records and inquiry (`INQ`) mode for viewing. Validates fields like vendor name, sort field, hold status, GL account, carrier, terms, category, and 1099 codes. Optionally synchronizes with an SQL vendor table (commented out).

2. **Vendor Master Inactivation/Reactivation**:
   - **Description**: Enables marking a vendor as inactive (`I`) or reactivating it (`A`) in the `APVEND` file, with support for 1099 processing via year-specific file overrides.
   - **Program**: `AP9104.rpgle`
   - **Details**: Provides a window interface to toggle the `vndel` flag. Originally included balance checks (invoices, monthly balances, inventory history) to prevent inactivation, but these are disabled, allowing unrestricted status changes.

---

### Function Requirement Document: Vendor Master Management Function



# Vendor Master Management Function Requirements

## Overview
The Vendor Master Management function handles the creation, update, inquiry, inactivation, and reactivation of vendor master records in the accounts payable system. It supports 1099 processing with year-specific file overrides and enforces data integrity through validations.

## Inputs
- **Company Code** (`p$co`): Identifier for the company (numeric).
- **Vendor Number** (`p$vend`): Identifier for the vendor (numeric).
- **Run Mode** (`p$mode`, 3A): `MNT` (maintenance) or `INQ` (inquiry).
- **File Name** (`p$file`, 10A): Optional year-specific vendor file (e.g., `APVN2025`) for 1099 processing.
- **File Group** (`p$fgrp`, 1A): `G` or `Z` for file overrides.
- **Vendor Data** (for `MNT` mode, structure mirroring `APVEND`):
  - Name (`vnname`): Vendor name (non-blank).
  - Sort Field (`vnsort`): Short name for sorting (non-blank).
  - Name Overflow (`vnnovf`): `Y` or `N`.
  - Hold (`vnhold`): `H` (check), `A` (ACH), `W` (wire), `E`, `U` (utility auto-pay), or blank.
  - Single Payee (`vnsngl`): `S` or blank.
  - Expense GL Account (`vnexgl`): General ledger account number.
  - Carrier (`vncaid`): Carrier ID.
  - Terms (`vnterm`): Payment terms code.
  - Category (`vncatg`): Vendor category code.
  - 1099 Code (`vn1099`): 1099 reporting code.
  - IRS Name Control (`vnnmct`): Required if `vn1099` is `M` or `N`.
  - IRS EIN (`vnidno`): Required if `vn1099` is `M` or `N`.
  - IRS Box 1 (`vnbox1`): Required if `vn1099` is `M` or `N`.
  - Country (`vnctry`): Country code (e.g., `US`).
  - Zip Code (`vnzip5`): Required if `vnctry = 'US'`, `vn1099 ≠ 'X'`, and `vnhold ≠ 'E'`.
  - Payee 1 (`vnpyn1`): Primary payee name.
  - Payee 2 (`vnpyn2`): Secondary payee name (requires `vnpyn1` if non-blank).
  - ACH Fields (optional, commented out):
    - Bank Account (`vnabk#`): Bank account number.
    - Account Type (`vnacos`): `C` (checking) or `S` (savings).
    - ACH Class (`vnacls`): `CCD`.
    - Routing Code (`vnarte`): Bank routing number.
- **Action** (for inactivation/reactivation): `REACTIVATE` or `INACTIVATE`.

## Outputs
- **Vendor Data**: Updated or retrieved vendor record (structure mirroring `APVEND`).
- **Return Flag** (`p$flag`, 1A): Indicates result:
  - `1`: Successful create/update (maintenance).
  - `A`: Successful reactivation.
  - `D`: Successful inactivation.
- **Error Messages**: List of validation errors (if any).

## Process Steps
1. **Validate Inputs**:
   - Verify `p$co` exists in `APCONT`.
   - For `MNT` or `INQ`, check if `p$co` and `p$vend` exist in `APVEND` (optional for `MNT` if creating new).
   - Apply file overrides for `APVEND`, `APCONT`, `GLMAST`, `GSTABL`, `BBCAID`, `APOPNH`, `INFIL4` based on `p$fgrp` (`G` or `Z`) or `p$file` (e.g., `APVN2025`).

2. **Maintenance Mode (`MNT`)**:
   - **Create/Update**:
     - Validate input fields:
       - Mandatory: `vnname`, `vnsort` non-blank.
       - `vnnovf`: `Y` or `N`.
       - `vnhold`: `H`, `A`, `W`, `E`, `U`, or blank.
       - `vnsngl`: `S` or blank.
       - `vnexgl`: Exists in `GLMAST`, not deleted or inactive, not special.
       - `vncaid`: Exists in `BBCAID`, not deleted.
       - `vnterm`: Exists in `GSTABL`, not deleted.
       - `vncatg`: Exists in `GSTABL`.
       - `vn1099`: Exists in `GSTABL`, not deleted, non-blank.
       - If `vn1099 = 'M'` or `'N'`:
         - `vnnmct`, `vnidno` non-blank.
         - `vnbox1` non-zero.
       - If `vnctry = 'US'` and `vn1099 ≠ 'X'` and `vnhold ≠ 'E'`, `vnzip5` non-zero.
       - If `vnpyn2` non-blank, `vnpyn1` non-blank.
       - ACH (if enabled): If `vnabk#` non-blank, `vnacos` (`C`/`S`), `vnacls` (`CCD`), `vnarte` non-zero; if `vnhold = 'A'` and `vnabk#` blank, error.
     - If valid, update or create record in `APVEND`:
       - If vendor exists, update `apvendpf` if changed.
       - If vendor does not exist, write new `apvendpf` with `vnco = p$co`, `vnvend = p$vend`.
       - Set `p$flag = '1'`.
     - (Optional, disabled) Synchronize with SQL vendor table via external call.

3. **Inquiry Mode (`INQ`)**:
   - Retrieve vendor record from `APVEND` using `p$co` and `p$vend`.
   - Return vendor data without modifications.

4. **Inactivation/Reactivation**:
   - **Reactivate**:
     - If vendor exists in `APVEND` and `vndel = 'I'`, set `vndel = 'A'`, update `apvendpf`, set `p$flag = 'A'`.
   - **Inactivate**:
     - If vendor exists in `APVEND` and `vndel ≠ 'I'`, set `vndel = 'I'`, update `apvendpf`, set `p$flag = 'D'`.
     - (Disabled) Prevent inactivation if open invoices (`APOPNH`), non-zero balances (`vnpurc`, `vnpay`, `vndmtd`), or inventory history (`INFIL4`).

5. **Error Handling**:
   - Return error messages for validation failures (e.g., “Invalid Response...H or A or W or blank”, “Field Cannot be Blank, If Vendor receives a 1099”).

## Business Rules
- **Data Integrity**:
  - Mandatory fields (`vnname`, `vnsort`, `vn1099`) must be non-blank.
  - Reference fields (`vnexgl`, `vncaid`, `vnterm`, `vncatg`, `vn1099`) must exist in respective files (`GLMAST`, `BBCAID`, `GSTABL`) and not be deleted (except `vncatg`).
  - 1099-specific fields (`vnnmct`, `vnidno`, `vnbox1`) required for `vn1099 = 'M'` or `'N'`.
  - Zip code (`vnzip5`) required for US vendors unless `vn1099 = 'X'` or `vnhold = 'E'`.
  - Payee hierarchy: `vnpyn2` requires `vnpyn1`.
- **1099 Processing**:
  - Supports year-specific vendor files (e.g., `APVN2025`) via `p$file`.
  - File overrides (`G` or `Z`) ensure access to correct data sets.
- **Status Management**:
  - Inactivation marks `vndel = 'I'`, reactivation marks `vndel = 'A'`.
  - (Disabled) Inactivation restricted if vendor has outstanding activity.
- **Mode Restrictions**:
  - `MNT` allows create/update, `INQ` allows view only.
  - Inactivation/reactivation is a maintenance action.

## Calculations
- **Message Data Length**:
  - For error messages, calculate length as `%len(%trim(m@data))` for `QMHSNDPM`.
- **No Other Calculations**: The function performs validations and status updates without numerical computations.

## Dependencies
- **Files**:
  - `APVEND`: Vendor master (update).
  - `APCONT`: Company data (input).
  - `GLMAST`: General ledger accounts (input).
  - `GSTABL`: Reference data (terms, category, 1099 codes) (input).
  - `BBCAID`: Carrier data (input).
  - `APOPNH`: Open invoices (input, used in disabled checks).
  - `INFIL4`: Inventory history (input, used in disabled checks).
- **External Programs** (for validations):
  - `LGLMAST`: GL account lookup.
  - `LBBCAID`: Carrier lookup.
  - `LGSTABL`: Terms, category, 1099 code lookup.
  - (Disabled) `AP915P`: Vendor contact maintenance/inquiry.
  - (Disabled) `STRPCOCLP`/`UpdateVendor.EXE`: SQL table sync.
- **System Programs**:
  - `QCMDEXC`: File overrides.
  - `QMHSNDPM`, `QMHRMVPM`: Message handling.

## Error Handling
- Return descriptive error messages for validation failures.
- Set `p$flag` to indicate success (`1`, `A`, `D`) or leave unset for errors.

## Assumptions
- Input data conforms to `APVEND` record format.
- File overrides are correctly configured for `p$fgrp` and `p$file`.
- Disabled features (ACH validation, balance checks, SQL sync) are not required.

