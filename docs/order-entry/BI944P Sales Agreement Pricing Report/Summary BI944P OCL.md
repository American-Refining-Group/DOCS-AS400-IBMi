### List of Use Cases Implemented in the Program Call Stack

The call stack, consisting of the OCL program `BI944.ocl36.txt` and the RPG programs `BI944P.rpg36.txt`, `BI9443.rpg`, `BI9444.rpg`, and `BI944.rpg36`, implements a system for generating and managing a Customer Sales Agreement Master File Listing. The stack supports multiple use cases, each addressing specific business needs for filtering, processing, and reporting sales agreements. Below is a comprehensive list of use cases derived from the programs:

1. **Generate Customer Sales Agreement Report**:
   - **Description**: Produces a printed or Excel-compatible report listing customer sales agreements based on user-specified criteria, including division, location, product, customer, salesman, and unit of measure.
   - **Components Involved**: 
     - `BI944P.rpg36.txt`: Captures and validates user input parameters (e.g., division, locations, products, customers, start date, unit of measure).
     - `BI944.ocl36.txt`: Orchestrates file preparation, sorting, and program execution.
     - `BI9443.rpg`: Filters agreements to include only unexpired ones and appends salesman, product class, and division data.
     - `BI9444.rpg`: Filters records based on user-specified selection criteria.
     - `BI944.rpg36`: Generates the final report with enriched data (customer names, ship-to details, pricing, freight terms).
   - **Output**: Printed report (`PRTDOWN`) or Excel spreadsheet (`PRTEXCEL`) with agreement details, including product codes, descriptions, customer details, prices, freight terms, and quantities.

2. **Update Sales Agreement Work File with New Start Date**:
   - **Description**: Updates the work file `BB203W` with a new start date for agreements when the add/update option (`KYADDA = 'Y'`) is selected, typically for price change processing.
   - **Components Involved**:
     - `BI944P.rpg36.txt`: Validates the new start date (`KYSTDT`) to ensure it is valid and future-dated.
     - `BI944.ocl36.txt`: Copies `BB203W` to the appropriate library.
     - `BI944.rpg36`: Writes the new start date to `BB203W`.
   - **Output**: Updated `BB203W` file with new start dates for applicable agreements.

3. **Export Sales Agreement Data for CRM Integration**:
   - **Description**: Generates a file (`BICUAGC`) for hourly upload to CRM software when the `U3` indicator is on, containing sales agreement data synchronized with `BICUA7`.
   - **Components Involved**:
     - `BI944.ocl36.txt`: Manages the overall process flow.
     - `BI944.rpg36`: Writes enriched agreement data to `BICUAGC` when `U3` is on.
   - **Output**: `BICUAGC` file with agreement details for CRM integration.

4. **Filter and Sort Sales Agreements for Specific Criteria**:
   - **Description**: Filters and sorts sales agreement records based on user-specified criteria (e.g., specific locations, product classes, customers, products, salesmen, or units of measure) to produce a tailored dataset for reporting or further processing.
   - **Components Involved**:
     - `BI944P.rpg36.txt`: Captures and validates selection criteria (`KYLOSL`, `KYPCSL`, `KYCSSL`, `KYPDSL`, `KYSMSL`, `KYUMSL`).
     - `BI944.ocl36.txt`: Executes sorting via `#GSORT` with inclusion or exclusion logic based on `KYSTYP`.
     - `BI9443.rpg`: Filters out expired agreements.
     - `BI9444.rpg`: Applies user-specified filters for locations, product classes, products, and units of measure.
   - **Output**: Sorted and filtered dataset in `BICUAGXX` for final reporting.

### Function Requirement Document

The following function requirement document describes a consolidated function, `GenerateCustomerSalesAgreement`, that encapsulates the primary use case of generating the Customer Sales Agreement Master File Listing. It assumes the function receives all necessary inputs programmatically (rather than via interactive screens) and produces the required outputs. The document outlines the process steps and business rules concisely, focusing on business requirements and necessary calculations.

<xaiArtifact artifact_id="0bf58436-b8c3-4fc4-bb90-57053c538bec" artifact_version_id="fcf756e2-845a-4afd-8d17-9a1f7b3931cf" title="GenerateCustomerSalesAgreementRequirements.md" contentType="text/markdown">

# Function Requirement: GenerateCustomerSalesAgreement

## Purpose
Generate a Customer Sales Agreement Master File Listing report, filter and sort sales agreement records based on specified criteria, update a work file with new start dates (if applicable), and optionally export data for CRM integration.

## Inputs
- **Division Selection** (`KYDIV`): `'R'` (Refinery), `'L'` (Blended Lubes), or `'B'` (Both).
- **Location Selection** (`KYLOSL`): `'ALL'` or `'SEL'` with up to 5 location codes (`KYLOC1`–`KYLOC5`, 3 bytes each).
- **Product Class Selection** (`KYPCSL`): `'ALL'` or `'SEL'` with up to 10 product class codes (`KYPC01`–`KYPC10`, 3 bytes each).
- **Customer Selection** (`KYCSSL`): `'ALL'` or `'SEL'` with up to 20 customer numbers (`KYCS01`–`KYCS20`, 6 bytes each) and type (`KYSTYP`): `'INCLUDE'` or `'EXCLUDE'`.
- **Product Selection** (`KYPDSL`): `'ALL'` or `'SEL'` with up to 10 product codes (`KYPD01`–`KYPD10`, 4 bytes each).
- **Salesman Selection** (`KYSMSL`): `'ALL'` or `'SEL'` with salesman range (`KYFRSM`, `KYTOSM`, 2 bytes each).
- **Unit of Measure Selection** (`KYUMSL`): `'ALL'` or `'SEL'` with unit of measure (`KYUM01`, 3 bytes) and type (`KYUMYP`): `'INCLUDE'` or `'EXCLUDE'`.
- **Add/Update Flag** (`KYADDA`): `'Y'` (update `BB203W`) or `'N'` (report only).
- **Job Queue Flag** (`KYJOBQ`): `'Y'` (run in batch) or `'N'` (interactive).
- **Copy Count** (`KYCOPY`): Integer (1–99, default 1).
- **Industry Code** (`KYINDC`): `'INCREASE'` or `'DECREASE'` (optional, for price change reporting).
- **Price Change Amount** (`KYDLCH`): Numeric (optional, for price change).
- **New Start Date** (`KYSTDT`): Date in MMDDYY format (optional, must be future-dated if provided).
- **Unit of Measure for Price Change** (`KYUOFM`): 3-byte code (optional, must exist in `GSTABL`).
- **CRM Export Flag** (`U3`): Boolean (true for hourly CRM export, false otherwise).

## Outputs
- **Printed Report** (`PRTDOWN`): Standard report with agreement details (164 bytes).
- **Excel Report** (`PRTEXCEL`): Spreadsheet-compatible report with additional fields (224 bytes).
- **Work File** (`BB203W`): Updated with new start dates if `KYADDA = 'Y'` (327 bytes).
- **CRM File** (`BICUAGC`): Sales agreement data for CRM upload if `U3 = true` (271 bytes).

## Process Steps
1. **Validate Inputs**:
   - Ensure `KYDIV` is `'R'`, `'L'`, or `'B'`.
   - For `KYLOSL`, `KYPCSL`, `KYCSSL`, `KYPDSL`, `KYSMSL`, `KYUMSL` = `'SEL'`, validate corresponding codes against `INLOC`, `GSTABL`, `ARCUST`, `GSPROD`, and `GSTABL` respectively.
   - If `KYCSSL` or `KYUMSL` = `'ALL'`, ensure `KYSTYP` or `KYUMYP` is blank.
   - Validate `KYADDA`, `KYJOBQ` as `'Y'` or `'N'`.
   - If `KYADDA = 'Y'`, ensure `KYSPRD ≠ 'Y'`.
   - If `KYUOFM` is non-blank, validate against `GSTABL` and ensure `KYSTDT` is non-zero, valid, and future-dated.
   - Validate `KYINDC` as `'INCREASE'` or `'DECREASE'` if non-blank.
   - Default `KYCOPY` to 1 if zero.

2. **Prepare Temporary Files**:
   - Clear `BICUAGX`, `BICUAG2`, `BI944S`, `BI944T`.
   - Copy `BB203W` to the appropriate library (`QS36F` or `QS36FTEST`).

3. **Filter Unexpired Agreements**:
   - Read `BICUAG`, include records where `BAEND8` (end date, CYMD) is zero or ≥ system date (`KYDAT8`).
   - Chain to `ARCUST` for salesman number (`ARSLS#`), `GSPROD` for product class (`TPPRCL`), and `GSTABL` for division (`TBCSRT`).
   - Write to `BICUAGX` with added fields: division (258), product class (259–261), salesman (262–263), and default dates for zero-end-date records.

4. **Sort and Filter Records**:
   - Sort `BICUAGX` into `BI944S` based on company, customer, container code, location, and contract number, applying inclusion (`KYSTYP = 'INCLUDE'`) or exclusion (`KYSTYP = 'EXCLUDE'`) for customers.
   - Filter `BI944S` into `BICUAG2` based on `KYLOSL`, `KYPCSL`, `KYPDSL`, `KYUMSL` criteria:
     - Locations: Match `BALOC` to `KYLOC1`–`KYLOC5`.
     - Product Classes: Match `BAPRCL` to `KYPC01`–`KYPC10`.
     - Products: Match `BAPR01`–`BAPR10` to `KYPD01`–`KYPD10`.
     - Units of Measure: Match `BAUNMS` to `KYUM01` (include/exclude based on `KYUMYP`).
   - Sort `BICUAG2` into `BI944T` by company, division, product class, product code, customer, and ship-to.

5. **Generate Report**:
   - Read `BI944T`, enrich with:
     - Customer name (`ARNAME`) from `ARCUST`.
     - Ship-to details (`CSNAME`, `CSADR1`–`CSADR3`) from `SHIPTO` or `'ALL'` if `BAALSH = 'Y'`.
     - Product descriptions (`BAD`) from `GSPROD`.
     - Product class description (`PRCLDS`) from `GSTABL` or `GSPROD`.
     - Previous price (`PRVPRC`) from `BICUA7`, prioritizing container code (`BACNTR`), then blank, then `'P'` for non-fluid.
     - Freight description (`FRTDSC`) from `BAFRCD` (C=COLLECT, P=PPD & ADD, D=DELIVERED, A=3RD PARTY).
     - Quantities (`BAMNQY`, `BAMXQY`) converted to gallons using `GSCTWT` (preferred) or `GSUMCV`.
   - Write to `PRTDOWN` and `PRTEXCEL` with headers (company, division, date, time, criteria), detail lines (product, customer, ship-to, prices, freight, dates, quantities), and total count.
   - If `KYADDA = 'Y'`, write new start date (`KYSTDT`) to `BB203W`.
   - If `U3 = true`, write to `BICUAGC` for CRM.

## Business Rules
1. **Input Validation**:
   - `KYDIV` must be `'R'`, `'L'`, or `'B'`.
   - Selection fields (`KYLOSL`, `KYPCSL`, `KYCSSL`, `KYPDSL`, `KYSMSL`, `KYUMSL`) must be `'ALL'` or `'SEL'`.
   - For `'SEL'`, corresponding codes must exist in respective files; for `'ALL'`, selection arrays must be blank.
   - `KYSTYP`/`KYUMYP` must be `'INCLUDE'` or `'EXCLUDE'` for `'SEL'`, blank for `'ALL'`.
   - `KYADDA`, `KYJOBQ` must be `'Y'` or `'N'`; `KYADDA = 'Y'` and `KYSPRD = 'Y'` are mutually exclusive.
   - `KYSTDT` must be valid, future-dated, and paired with non-blank `KYUOFM`.
   - `KYINDC` must be `'INCREASE'` or `'DECREASE'` if provided.

2. **Agreement Filtering**:
   - Include agreements with `BAEND8 = 0` or `BAEND8 ≥ KYDAT8`.
   - Filter by location, product class, customer, product, salesman, and unit of measure as specified.
   - Exclude deleted records (`BADEL = 'D'`, `ARDEL = 'D'`, `CSDEL = 'D'`).

3. **Data Enrichment**:
   - Retrieve salesman from `ARCUST`, product class from `GSPROD`/`GSTABL`, division from `GSTABL`.
   - Retrieve ship-to details from `SHIPTO` unless `BAALSH = 'Y'`.
   - Retrieve previous price from `BICUA7` with container code priority.
   - Map freight codes (`BAFRCD`) to descriptions; handle non-Bradford freight collect cases.

4. **Quantity Calculation**:
   - Convert `BAMNQY` and `BAMXQY` to gallons using `GSCTWT` (gallons per container) if available, else `GSUMCV` (unit of measure conversion).

5. **Output Formatting**:
   - Reports include company, division, product, customer, ship-to, pricing, freight, dates, and quantities.
   - `PRTEXCEL` includes off-price, new price, and quantities.
   - Suppress end date if zero (`'00/00/00'`).
   - Include error messages for missing product classes, products, or ship-to records.

## Calculations
- **Date Validation**:
  - Convert `KYSTDT` to CYMD (`KYSTD8`) using century (`Y2KCEN`) and comparison year (`Y2KCMP`).
  - Ensure `KYSTDT > SYSDAT` (system date).
- **Quantity Conversion**:
  - If `GSCTWT` record exists for `BACNTR`, use gallons per container:  
    `ConvertedQty = Quantity * GallonsPerContainer`.
  - Else, use `GSUMCV` conversion factor for `BAUNMS`:  
    `ConvertedQty = Quantity * ConversionFactor`.
- **Price Change**:
  - If `KYINDC = 'INCREASE'`, `NewPrice = BAPRCE + KYDLCH`.
  - If `KYINDC = 'DECREASE'`, `NewPrice = BAPRCE - KYDLCH`.

## Error Handling
- Output error messages for invalid inputs, missing reference data, or failed validations.
- Skip invalid records and continue processing.

## Dependencies
- **Files**: `BICUAG`, `BICUAGX`, `BI944S`, `BICUAG2`, `BI944T`, `BICONT`, `ARCUST`, `SHIPTO`, `GSPROD`, `GSTABL`, `ARCUPR`, `ARCUSP`, `INLOC`, `BICUA7`, `GSCTWT`, `GSUMCV`, `GSCNTR1`, `BB203W`, `PRTDOWN`, `PRTEXCEL`, `BICUAGC`.

</xaiArtifact>

This function consolidates the call stack’s functionality into a single, programmatic process, focusing on business requirements and necessary calculations while eliminating interactive screen dependencies. Let me know if you need further refinements or additional details!