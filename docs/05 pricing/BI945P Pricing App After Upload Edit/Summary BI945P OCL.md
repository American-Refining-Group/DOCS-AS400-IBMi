Based on the provided documents (`BI945P.ocl36.txt`, `BI945P.rpgle.txt`, `BI9445.ocl36.txt`, `BI944X.rpg.txt`, and `BI9445.rpgle.txt`), the call stack represents a process for managing and reporting customer sales agreements on an IBM midrange system (e.g., AS/400 or iSeries). The stack involves OCL scripts orchestrating RPG programs to process, validate, and report sales agreement data. Below, I’ll identify the use cases implemented by this call stack and then provide a concise function requirement document for the primary use case, treating it as a large function that processes inputs programmatically.

---

### List of Use Cases

The call stack implements the following use cases, each addressing a distinct business need within the customer sales agreement process:

1. **Prepare Sales Agreement Data for Processing**:
   - **Description**: Extracts customer and product data from the `PRSABLX` file and populates the Local Data Area (LDA) with up to 30 unique product codes and a customer number for downstream processing.
   - **Components**: 
     - `BI945P.ocl36.txt`: Loads `PRSABLX` and conditionally calls `BI9445` (via job queue or directly).
     - `BI945P.rpgle.txt`: Processes `PRSABLX` records, assigns customer number to `kycs01`, and stores unique product codes in `kypd01`–`kypd30`.
   - **Purpose**: Prepares filtered data (customer and products) to be used by subsequent programs for reporting or updates.

2. **Generate Customer Sales Agreement Price Change Report**:
   - **Description**: Produces a detailed report of customer sales agreements, including product codes, customer details, pricing, freight terms, and start/end dates, with options for printed and Excel-compatible outputs. Validates data and flags errors.
   - **Components**:
     - `BI9445.ocl36.txt`: Clears files, updates LDA, sorts data, and calls `BI944X` and `BI9445`.
     - `BI944X.rpg.txt`: Updates `BICUAG` with salesman, product class, and division, filtering out expired agreements.
     - `BI9445.rpgle.txt`: Processes sorted data, retrieves reference data, computes new prices, and generates reports (`prtdown`, `prtexcel`).
   - **Purpose**: Provides a comprehensive report for auditing or managing sales agreements, including price changes and validation errors.

3. **Update CRM System with Sales Agreement Data**:
   - **Description**: Updates the `BICUAGC` file with sales agreement data for hourly uploads to a CRM system, including customer, product, pricing, and freight details.
   - **Components**:
     - `BI9445.ocl36.txt`: Configures the process and calls `BI9445`.
     - `BI9445.rpgle.txt`: Writes to `BICUAGC` when indicator `U3` is on (hourly run).
   - **Purpose**: Ensures CRM system has up-to-date sales agreement data for external integration.

4. **Update Work File for New Start Dates**:
   - **Description**: Writes new start dates and agreement data to the `BB204T` work file for use in subsequent processes (e.g., agreement table updates).
   - **Components**:
     - `BI9445.ocl36.txt`: Clears `BB204T` and calls `BI9445`.
     - `BI9445.rpgle.txt`: Outputs agreement data to `BB204T` when `kyadda = 'Y'`.
   - **Purpose**: Supports downstream processes that update agreement tables with new start dates.

---

### Function Requirement Document

The primary use case, **Generate Customer Sales Agreement Price Change Report**, is the most comprehensive, encompassing data preparation, validation, price calculations, and reporting. Below is a function requirement document for this use case, treated as a large function that accepts inputs programmatically and produces outputs without screen interaction.

<xaiArtifact artifact_id="014086f1-1e25-4d11-991d-c8abc8c5c115" artifact_version_id="9544d56e-a6ba-4117-abef-b017f6241e1e" title="Customer_Sales_Agreement_Report_Requirements.md" contentType="text/markdown">

# Customer Sales Agreement Price Change Report Function

## Purpose
Generate a report of customer sales agreements, including product codes, customer details, pricing, freight terms, and start/end dates, while validating data and optionally updating work and CRM files. The function processes input data programmatically and produces printed and Excel-compatible reports.

## Inputs
- **Sales Agreement Data (`PRSABLX`)**: File with customer number, product codes, start/end dates, prices, and contract details.
- **Reference Files**:
  - `BICUAG`: Raw sales agreement data.
  - `ARCUST`: Customer master data (name, salesman).
  - `GSTABL`: Product class and division descriptions.
  - `GSCNTR1`: Container codes.
  - `SHIPTO`: Ship-to addresses (city, state).
  - `ARCUPR`: Previous pricing data.
  - `ARCUSP`, `BBPRCY`, `INLOC`, `GSUMCV`, `BICONT`, `BICUA7`: Additional reference data for validation and enrichment.
- **LDA Parameters**:
  - `kycs01`–`kycs20`: Customer numbers for inclusion/exclusion.
  - `kypd01`–`kypd30`: Product codes for filtering.
  - `kysprd`: Special pricing flag ('Y'/'N').
  - `kyindc`: Price index for adjustments.
  - `kydlch`: Price delta change (packed decimal).
  - `kyadda`: Add/update flag ('Y'/'N').
  - `kystd8`: System date for comparison (CYMD).
  - `kydiv`, `kylosl`, `kycssl`, `kypdsl`, `kyumsl`, `kydvno`, `kyjobq`: Filtering parameters (division, location, etc.).

## Outputs
- **Printed Report (`prtdown`)**: 164-byte report with headers, customer/product details, prices, freight terms, dates, and error messages.
- **Excel Report (`prtexcel`)**: 224-byte Excel-compatible report with additional fields (e.g., off-price).
- **Work File (`BB204T`)**: 327-byte file with new start dates and agreement data (if `kyadda = 'Y'`).
- **CRM File (`BICUAGC`)**: 271-byte file for CRM uploads (if `U3` is on).
- **Temporary Files**: `BICUAGX`, `BICUAG2`, `BI944S`, `BI944T` (cleared post-process).

## Process Steps
1. **Prepare Input Data**:
   - Read `PRSABLX` and store customer number in `kycs01` and up to 30 unique product codes in `kypd01`–`kypd30` in LDA.
   - Clear temporary files (`BICUAGX`, `BICUAG2`, `BI944S`, `BI944T`, `BB204T`).
2. **Update Sales Agreement Data**:
   - Process `BICUAG` to add salesman (`ARSLS#` from `ARCUST`), product class (`TBPRCL`), and division (`TBCSRT`) from `GSTABL`.
   - Filter out agreements with end date (`BAEND8`) before system date (`KYDAT8`) unless zero (perpetual).
   - Write updated records to `BICUAGX` (263 bytes, includes division, product class, salesman).
3. **Sort Data**:
   - Sort `BICUAGX` into `BI944S` by company, customer, container code, location, and contract, using inclusion (`kycs01`–`kycs20`) or exclusion logic based on LDA (`kystyp = 'EXCLUDE'`).
   - Copy `BI944S` to `BICUAG2`.
   - Sort `BICUAG2` into `BI944T` by company, division, product class, product code, customer, and ship-to.
4. **Generate Report**:
   - Read `BI944T` and validate against reference files:
     - Customer name from `ARCUST`.
     - Ship-to city/state from `SHIPTO`.
     - Product class description from `GSTABL`.
     - Container code from `GSCNTR1`.
     - Previous price from `ARCUPR` (try container code, then blank, then 'P' for non-fluid).
   - Compute new price: If `kysprd = 'Y'`, apply `kyindc` and `kydlch` to current price (`baprce`).
   - Output to `prtdown` and `prtexcel`:
     - Headers: Division, report mode, date, time, column titles (Product Code, Description, Customer No., Name, Ship-to, Location, Container, Prices, Freight Terms, Dates).
     - Details: Product code (`bapr01`), description, customer number, name, ship-to (`baship` or 'ALL'), city/state, container code, current price (`baprce`), previous price (`prvprc`), off-price (`baoffp`), location, unit of measure, freight terms, start/end dates, new price, quantities (`bamnqy`, `bamxqy`).
     - Total: Record count.
   - Write to `BB204T` (new start dates) if `kyadda = 'Y'`.
   - Write to `BICUAGC` (CRM data) if `U3` is on.
5. **Clean Up**:
   - Delete temporary files (`BI944S`, `BI944T`).
   - Reset LDA and switches.

## Business Rules
1. **Data Filtering**:
   - Include only unexpired agreements (`baend8 ≥ kystd8` or `baend8 = 0`).
   - Filter by customer numbers (`kycs01`–`kycs20`) and products (`kypd01`–`kypd30`) based on inclusion/exclusion logic (`kystyp`).
   - Apply division (`kydiv`), location (`kylosl`), and other LDA filters.
2. **Price Calculation**:
   - Retrieve previous price (`prvprc`) from `ARCUPR` using company, customer, product, and container code (priority: exact, blank, 'P' for non-fluid).
   - If `kysprd = 'Y'`, compute new price: `price = baprce + (baprce × kyindc) + kydlch`.
3. **Freight Terms**:
   - Use freight code (`bafrcd`) from `BICUAGXX` (e.g., COLLECT, PPD & ADD).
   - Compute freight difference (`frtdif`) for `CNY` (freight collect from non-Bradford locations).
4. **Date Handling**:
   - Display '00/00/00' for perpetual agreements (`baend8 = 0`).
   - Validate start/end dates against `kystd8`.
5. **Error Handling**:
   - Flag errors for missing product class, invalid product, or missing ship-to record.
6. **Output Modes**:
   - If `kyadda = 'Y'`, update `BB204T` and `BICUAGC` (if `U3` is on).
   - If `kyadda = 'N'`, generate report only.
7. **CRM Integration**:
   - Update `BICUAGC` only during hourly runs (`U3` on).

## Calculations
- **New Price**:
  - Formula: `price = baprce + (baprce × kyindc / 100) + kydlch` (if `kysprd = 'Y'`).
  - Example: If `baprce = 100.00`, `kyindc = 5` (5%), `kydlch = 2.00`, then `price = 100 + (100 × 0.05) + 2 = 107.00`.
- **Freight Difference**:
  - For `bafrcd = 'CNY'` and non-Bradford location, compute `frtdif` based on location-specific rules (not detailed in code).

## Assumptions
- Input files are pre-populated (`GPRSABLO` to `PRSABLX`).
- LDA parameters are set by upstream processes or defaults.
- Output files are accessible for writing.
- No screen interaction; inputs are file-based or LDA-based.

## Constraints
- File access uses shared read mode for reference files to ensure data integrity.
- `BICUAGC` updates require `U3` indicator (hourly run).
- Temporary files are cleared post-process to prevent data overlap.

</xaiArtifact>

---

### Notes
- The **Prepare Sales Agreement Data for Processing** use case is a prerequisite for the reporting and update use cases, as it populates the LDA with filtered data.
- The **Update CRM System** and **Update Work File** use cases are conditional, depending on `U3` and `kyadda`, respectively, and are executed within the reporting process.
- The function requirement document focuses on the primary reporting use case, as it integrates all components of the call stack. If separate documents for the other use cases are needed, please specify.