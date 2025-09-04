### List of Use Cases Implemented in the Call Stack

The provided call stack consists of five programs: `BB945.rpgle`, `BB945C.clp`, `BB9451.rpgle`, `GB730P.rpgle`, and `MBBFR1.rpgle`, with `BB080.rpgle` called by `MBBFR1.rpgle`. These programs are part of the Bradford Order Entry/Invoices system and collectively implement several use cases related to customer freight maintenance, reporting, history tracking, and freight charge calculations. Below are the identified use cases, derived from the functionality of each program in the call stack:

1. **Customer Freight Maintenance (BB945.rpgle, BB945C.clp, BB946.rpgle)**:
   - **Description**: Allows users to maintain customer freight records in the `BICUFR` file, including adding, updating, deleting, or reactivating records. The process involves creating a temporary work file (`BB945W`), validating data, and logging changes to a history file (`bicufrh`).
   - **Components**:
     - `BB945C.clp`: Prepares the `BB945W` work file by clearing or creating it in `QTEMP`.
     - `BB945.rpgle`: Provides a user interface to add or update customer freight records, validate inputs, and populate `BB945W`.
     - `BB946.rpgle`: Handles deletion or reactivation of customer freight records, updating `BICUFR` and logging changes to `bicufrh`.

2. **Freight Maintenance Error Reporting (BB9451.rpgle)**:
   - **Description**: Generates a printed report listing customer freight records in `BB945W` with missing General Ledger (G/L) freight accounts, aiding in error identification and correction.
   - **Components**:
     - `BB9451.rpgle`: Reads `BB945W`, formats data, and prints the "Freight Maintenance GL Error Listing" report to the `FRTOUTQ` queue.

3. **Freight History Inquiry (GB730P.rpgle)**:
   - **Description**: Enables users to query and view historical changes to customer freight records (and other files) in a subfile, including details like effective dates, deletion status, and field changes.
   - **Components**:
     - `GB730P.rpgle`: Displays historical data from files like `bicufrh`, `bbndfihx`, and `bbordhsx`, with filtering and detailed record viewing capabilities.

4. **Freight Charge Calculation (MBBFR1.rpgle, BB080.rpgle)**:
   - **Description**: Calculates freight charges for orders or invoices, including line haul amounts, fuel surcharges, insurance, and accessorial charges, for use in order entry, bill of lading, or invoice processing.
   - **Components**:
     - `MBBFR1.rpgle`: Computes freight charges based on customer freight data (`BICUFR`), unit conversions (`GSUMCV`), and railcar surcharges (`BBRCSC2`).
     - `BB080.rpgle`: Retrieves the carrier fuel surcharge percentage (`FSPC`) from `BBCFSD`, `BBCFSH`, and `BBNDFI2` based on the National Diesel Fuel Index.

---

### Function Requirement Document: Customer Freight Maintenance and Calculation

#### Function Name: CustomerFreightMaintenanceAndCalculation

#### Purpose
To manage customer freight records, calculate freight charges, generate error reports, and track historical changes for orders or invoices in the Bradford Order Entry/Invoices system.

#### Inputs
- **Company Number** (`p$cono`, 2 digits, numeric): Identifies the company.
- **Customer Number** (`p$cust`, 6 digits, numeric): Identifies the customer.
- **Ship-to Number** (`p$ship`, 3 digits, numeric): Specifies the ship-to location.
- **Carrier ID** (`p$caid`, 6 characters): Identifies the carrier.
- **Effective Date** (`p$efdt`, 8 digits, CCYYMMDD): Date the freight record is effective.
- **Expiration Date** (`p$exdt`, 8 digits, CCYYMMDD): Date the freight record expires (optional).
- **Sequence Key** (`p$sqky`, numeric): Unique identifier for freight record (optional).
- **File Group** (`p$fgrp`, 1 character, 'Z' or 'G'): Specifies library (production or development/test).
- **Order/Invoice Mode** (`p$froei`, 1 character, 'O' or 'I'): Indicates order entry ('O') or invoice entry ('I').
- **Order Number** (`p$ordr`, 6 digits, numeric): Order number for freight calculation (if `p$froei = 'O'`).
- **Railcar Number** (`p$rcar`, 10 characters): Railcar identifier for invoice entry (if `p$froei = 'I'`).
- **Quantity** (`p$qty`, 7 digits, numeric): Order or invoice quantity.
- **Unit of Measure** (`p$um`, 3 characters): Unit of measure for quantity.
- **Freight Rate Override** (`p$ffpu`, 7.4 packed, optional): Override freight rate per unit (for `p$froei = 'I'`).
- **Pickup Date** (`p$rqdt`, 6 digits, YYMMDD): Pickup date for freight calculation.
- **Pickup Date Century** (`p$rqcn`, 2 digits, numeric): Century for pickup date.
- **Freight Code** (`p$frcd`, 1 character): Freight code for calculation.
- **Container Code** (`p$cncd`, 2 digits, numeric): Container code for validation.
- **Miles** (`p$mils`, 5 digits, numeric): Freight miles (optional).
- **Multi-Load Count** (`p$mlcnt`, 3 digits, numeric): Number of loads for invoice entry.
- **Product Code** (`p$prod`, 4 characters): Product code for unit conversion.
- **Weight** (`p$wt`, 9 digits, numeric): Total weight (optional).

#### Outputs
- **Freight Record Status** (`p$flag`, 1 character): Indicates success ('1') or failure ('0') of maintenance or calculation.
- **Total Freight Amount** (`p$famt`, 7.2 packed): Total calculated freight charge.
- **Line Haul Amount** (`p$lnha`, 7.2 packed): Base freight charge before surcharges.
- **Fuel Surcharge Percentage** (`p$scpc`, 5.2 packed): Fuel surcharge percentage applied.
- **Fuel Surcharge Amount** (`p$scam`, 7.2 packed): Fuel surcharge amount.
- **Insurance Amount** (`p$inam`, 7.2 packed): Insurance charge.
- **Other Freight Charges** (`p$otfr`, 7.2 packed): Sum of accessorial charges.
- **Error Report** (text file): Lists records with missing G/L freight accounts.
- **History Records** (list): Historical changes to freight records (company, customer, ship-to, effective date, deletion status, etc.).

#### Process Steps
1. **Initialize Work File**:
   - Clear or create `BB945W` in `QTEMP` from `DATA` (if `p$fgrp = 'G'`) or `DATADEV` (if `p$fgrp ≠ 'G'`).
2. **Maintain Freight Records**:
   - Validate inputs (`p$cono`, `p$cust`, `p$ship`, `p$cncd`) against `shipto`, `gscntr1`, and `gstabl`.
   - Add or update records in `BICUFR` with fields like freight rate (`bfrpum`), fuel surcharge percentage (`bfschg`), and accessorial charges.
   - Delete (`bfdel = 'D'`) or reactivate (`bfdel = *blank`) records in `BICUFR`, logging changes to `bicufrh`.
3. **Generate Error Report**:
   - Populate `BB945W` with freight records lacking valid G/L accounts (`qffrgl`).
   - Produce a formatted report listing company, customer, carrier, contract code, and other details.
4. **Track Freight History**:
   - Retrieve historical changes from `bicufrh` based on `p$cono`, `p$cust`, `p$efdt`, and other filters.
   - Return history records with fields like effective date, deletion status, and changed values.
5. **Calculate Freight Charges**:
   - Retrieve freight parameters from `BICUFR` (e.g., `bfrpum`, `bfschg`, `bfinsp`).
   - Convert quantity (`p$qty`) using `GSUMCV` based on `p$um` and `p$prod`.
   - Compute line haul amount (`p$lnha = quantity * bfrpum` or `p$ffpu` if provided).
   - Calculate fuel surcharge:
     - If `bfschg ≠ 0`, use `bfschg` (`p$scam = p$lnha * bfschg`).
     - If `bfscca = 'M'`, add railcar surcharge from `BBRCSC2` based on `p$rcar` and `p$rqdt`.
     - Otherwise, retrieve carrier fuel surcharge (`FSPC`) from `BB080` using `p$cono`, `p$caid`, `p$rqdt`, and apply (`p$scam = p$lnha * FSPC / 100`).
   - Calculate insurance (`p$inam = p$lnha * bfinsp * 0.01` if `bfinsp ≠ 0`).
   - Sum accessorial charges (`p$otfr = bfclen + bfpump + bfhose + bfpuup + bftoll + bfdent + bfinta + bfstop + bfscal`).
   - For invoice mode (`p$froei = 'I'`), multiply accessorial charges by `p$mlcnt`.
   - Compute total freight (`p$famt = p$lnha + p$scam + p$inam + p$otfr`).
6. **Return Results**:
   - Set `p$flag = '1'` if processing succeeds, else `p$flag = '0'`.
   - Return calculated amounts, error report, and history records.

#### Business Rules
1. **Freight Record Maintenance**:
   - Records require valid company, customer, ship-to, and container code.
   - Deleted records are marked (`bfdel = 'D'`), reactivated records clear `bfdel`.
   - All changes are logged to `bicufrh` with user ID and timestamp.

2. **Error Reporting**:
   - Records in `BB945W` with missing G/L accounts are reported with company, customer, carrier, and date details.

3. **History Tracking**:
   - Historical records are filterable by company, customer, effective date, and deletion status.
   - Deleted records are marked visually distinct in history output.

4. **Freight Calculation**:
   - Line haul uses `bfrpum` or `p$ffpu` (invoice mode override).
   - Fuel surcharge uses `bfschg`, railcar surcharge (`BBRCSC2`), or carrier surcharge (`BB080`) based on `bfscca`.
   - Accessorial charges are summed, adjusted for negative quantities, and multiplied by multi-load count in invoice mode.
   - Fuel surcharge percentage is 5.2 packed (JB04, 06/17/2024).

5. **Data Isolation**:
   - Work file (`BB945W`) is job-specific in `QTEMP`.
   - File group (`p$fgrp`) determines library (`DATA` or `DATADEV`).

#### Calculations
- **Line Haul**: `p$lnha = quantity * (bfrpum or p$ffpu)`, adjusted for minimum quantities (`bfmin`, `bfminng`).
- **Fuel Surcharge**: 
  - If `bfschg ≠ 0`: `p$scam = p$lnha * bfschg`, `p$scpc = bfschg * 100`.
  - If `bfscca = 'M'`: `p$scam += RCSAM` from `BBRCSC2`.
  - Else: `p$scam = p$lnha * FSPC / 100`, `p$scpc = FSPC` (from `BB080`).
- **Insurance**: `p$inam = p$lnha * bfinsp * 0.01` if `bfinsp ≠ 0`.
- **Accessorial Charges**: `p$otfr = bfclen + bfpump + bfhose + bfpuup + bftoll + bfdent + bfinta + bfstop + bfscal`.
- **Total Freight**: `p$famt = p$lnha + p$scam + p$inam + p$otfr`.

#### Artifact: Function Requirement Document

<xaiArtifact artifact_id="14925a5b-0858-4a80-a45e-cae7c9751865" artifact_version_id="8f10925f-2c4f-45a5-b929-1ff0595118eb" title="CustomerFreightMaintenanceAndCalculation.md" contentType="text/markdown">

# Customer Freight Maintenance and Calculation Function Requirements

## Function Name
CustomerFreightMaintenanceAndCalculation

## Purpose
Manage customer freight records, calculate freight charges, generate error reports, and track historical changes for orders or invoices.

## Inputs
- **Company Number** (`p$cono`, 2 digits, numeric)
- **Customer Number** (`p$cust`, 6 digits, numeric)
- **Ship-to Number** (`p$ship`, 3 digits, numeric)
- **Carrier ID** (`p$caid`, 6 characters)
- **Effective Date** (`p$efdt`, 8 digits, CCYYMMDD)
- **Expiration Date** (`p$exdt`, 8 digits, CCYYMMDD, optional)
- **Sequence Key** (`p$sqky`, numeric, optional)
- **File Group** (`p$fgrp`, 1 char, 'Z'/'G')
- **Order/Invoice Mode** (`p$froei`, 1 char, 'O'/'I')
- **Order Number** (`p$ordr`, 6 digits, numeric)
- **Railcar Number** (`p$rcar`, 10 chars)
- **Quantity** (`p$qty`, 7 digits, numeric)
- **Unit of Measure** (`p$um`, 3 chars)
- **Freight Rate Override** (`p$ffpu`, 7.4 packed, optional)
- **Pickup Date** (`p$rqdt`, 6 digits, YYMMDD)
- **Pickup Date Century** (`p$rqcn`, 2 digits, numeric)
- **Freight Code** (`p$frcd`, 1 char)
- **Container Code** (`p$cncd`, 2 digits, numeric)
- **Miles** (`p$mils`, 5 digits, numeric, optional)
- **Multi-Load Count** (`p$mlcnt`, 3 digits, numeric)
- **Product Code** (`p$prod`, 4 chars)
- **Weight** (`p$wt`, 9 digits, numeric, optional)

## Outputs
- **Freight Record Status** (`p$flag`, 1 char): '1' (success), '0' (failure)
- **Total Freight Amount** (`p$famt`, 7.2 packed)
- **Line Haul Amount** (`p$lnha`, 7.2 packed)
- **Fuel Surcharge Percentage** (`p$scpc`, 5.2 packed)
- **Fuel Surcharge Amount** (`p$scam`, 7.2 packed)
- **Insurance Amount** (`p$inam`, 7.2 packed)
- **Other Freight Charges** (`p$otfr`, 7.2 packed)
- **Error Report** (text): Lists records with missing G/L accounts
- **History Records** (list): Historical freight record changes

## Process Steps
1. **Initialize Work File**: Clear/create `BB945W` in `QTEMP` from `DATA` (`p$fgrp = 'G'`) or `DATADEV` (`p$fgrp ≠ 'G'`).
2. **Maintain Freight Records**: Validate inputs, add/update/delete/reactivate `BICUFR` records, log changes to `bicufrh`.
3. **Generate Error Report**: List `BB945W` records with missing G/L accounts.
4. **Track Freight History**: Retrieve `bicufrh` records by `p$cono`, `p$cust`, `p$efdt`.
5. **Calculate Freight**:
   - Retrieve `BICUFR` data (`bfrpum`, `bfschg`, `bfinsp`).
   - Convert `p$qty` using `GSUMCV`.
   - Compute `p$lnha = quantity * (bfrpum or p$ffpu)`.
   - Fuel surcharge:
     - If `bfschg ≠ 0`: `p$scam = p$lnha * bfschg`.
     - If `bfscca = 'M'`: Add `RCSAM` from `BBRCSC2`.
     - Else: `p$scam = p$lnha * FSPC / 100` (from `BB080`).
   - Insurance: `p$inam = p$lnha * bfinsp * 0.01` if `bfinsp ≠ 0`.
   - Accessorial: `p$otfr = sum(bfclen, bfpump, bfhose, bfpuup, bftoll, bfdent, bfinta, bfstop, bfscal)`.
   - Total: `p$famt = p$lnha + p$scam + p$inam + p$otfr`.
6. **Return Results**: Set `p$flag`, return calculated amounts, report, and history.

## Business Rules
- Validate company, customer, ship-to, container code.
- Mark deleted records (`bfdel = 'D'`), reactivate by clearing `bfdel`.
- Log all changes to `bicufrh`.
- Report missing G/L accounts in `BB945W`.
- Filter history by company, customer, effective date.
- Calculate freight with customer-specific, railcar, or carrier surcharges.
- Use `QTEMP` for job-specific `BB945W`.
- Apply 5.2 packed precision for fuel surcharge (JB04).

</xaiArtifact>