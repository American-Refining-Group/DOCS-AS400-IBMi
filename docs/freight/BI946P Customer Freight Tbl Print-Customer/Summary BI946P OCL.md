The provided call stack consists of OCL and RPG programs (`BI946P.ocl36.txt`, `BI946P.rpgle.txt`, `BI945.ocl36.txt`, `BI945.rpgle.txt`, `BI946.ocl36.txt`, `BI946.rpgle.txt`) that together form a system for processing and reporting customer freight tables. Below, I’ll identify the use cases implemented by this call stack and then provide a concise function requirement document for one of the primary use cases, adapted as a large function that processes inputs directly rather than using screen interactions.

---

### Use Cases Implemented by the Call Stack

The call stack implements the following distinct use cases, each representing a significant business function within the freight table processing workflow:

1. **Validate User Inputs for Freight Table Report**:
   - **Description**: Collects and validates user inputs to specify parameters for generating a freight table report, such as company selection, customer numbers, and job queue options.
   - **Program Involved**: `BI946P.ocl36.txt`, `BI946P.rpgle.txt`
   - **Details**:
     - Prompts for inputs like company selection (`KYALCO`: ALL or CO), company numbers (`KYCO1`, `KYCO2`, `KYCO3`), customer selection (`KYALCS`: ALL or SEL), customer numbers (`CS`), job queue flag (`KYJOBQ`), number of copies (`KYCOPY`), and current records flag (`KYCUR`).
     - Validates inputs against the `BICONT` file, ensuring valid company numbers and appropriate values for other fields.
     - Outputs validated parameters to the `SCREEN` file or proceeds to call `BI946` for report generation.

2. **Summarize Sales Gallons for Pricing**:
   - **Description**: Aggregates sales data (gallons and orders) over a one-year period to create a pricing work file for freight calculations.
   - **Program Involved**: `BI945.ocl36.txt`, `BI945.rpgle.txt`
   - **Details**:
     - Filters sales data (`SA5FILD`, `SA5MOVD`) by date (today to one year ago) and excludes deleted records.
     - Sorts data by company, customer, ship-to, location, product, container, and unit of measure.
     - Summarizes gallons and order counts by freight code (collect, prepaid, prepaid & add) and calculates average gallons per order.
     - Outputs results to `PRICWK` for use in freight calculations.

3. **Generate Freight Table Report**:
   - **Description**: Produces a detailed freight table report, including freight rates, surcharges, and carrier costs, with outputs to a printed report and disk files for analysis.
   - **Program Involved**: `BI946.ocl36.txt`, `BI946.rpgle.txt`
   - **Details**:
     - Uses sorted freight data (`BICUFR`/`BI946S`), customer data (`ARCUST`), ship-to data (`SHIPTO`), and pricing data (`PRICWK`).
     - Filters records by current status (`KYCUR`) and excludes deleted records.
     - Calculates freight costs (via `MBBFRT`), including rates per mile, hundredweight, gallon, unit of measure, and additional fees (e.g., cleaning, pump, tolls).
     - Outputs a printed report (`list164`), a logistics analysis file (`FR946XL`), and a pricing report file (`BICUFRC`).

4. **Calculate Freight Costs**:
   - **Description**: Computes freight costs, including customer and carrier rates, surcharges, and insurance percentages, for each freight table record.
   - **Program Involved**: `BI946.rpgle.txt` (via implied call to `MBBFRT`)
   - **Details**:
     - Uses inputs from `BICUFR`, `BICONT`, `ARCUST`, `SHIPTO`, `GSCNTR1`, and `PRICWK`.
     - Applies constants (6500 for bulk, 25000 for rail) for freight calculations.
     - Calculates totals for billed freight (`f$btot`) and carrier freight (`f$ctot`), including line haul, fuel surcharges, and insurance amounts.

---

### Function Requirement Document: Generate Freight Table Report

Below is a concise function requirement document for the primary use case, **Generate Freight Table Report**, reimagined as a large function that takes inputs directly rather than using screen interactions. This document focuses on business requirements and explains calculations as needed.

<xaiArtifact artifact_id="2efef58c-16fb-4d76-9a33-fadd2de0b8c3" artifact_version_id="8cb0f216-94ef-4bb3-9031-161739764b46" title="FreightTableReportFunction.md" contentType="text/markdown">

# Function Requirement Document: Generate Freight Table Report

## Purpose
The `GenerateFreightTableReport` function processes freight table data to produce a detailed report of freight rates, surcharges, and carrier costs, along with disk files for logistics and pricing analysis. It summarizes sales gallons for pricing and applies user-specified filters to generate tailored outputs.

## Inputs
- **CompanySelection** (String): `'ALL'` or `'CO'` to process all companies or specific companies.
- **CompanyNumbers** (Array of Integers, max 3): Company numbers to filter (if `CompanySelection = 'CO'`).
- **CustomerSelection** (String): `'ALL'` or `'SEL'` to process all customers or specific customers.
- **CustomerNumbers** (Array of Integers, max 10): Customer numbers to filter (if `CustomerSelection = 'SEL'`).
- **CurrentRecordsOnly** (String): `'CUR'` or blank to filter for current records (within start/end dates) or all records.
- **JobQueueFlag** (String): `'Y'`, `'N'`, or blank to submit to job queue or run interactively.
- **NumberOfCopies** (Integer): Number of report copies (default: 1 if 0).
- **TodayDate** (String, CCYYMMDD): Current date for filtering.
- **OneYearAgoDate** (String, CCYYMMDD): Date one year ago for sales data filtering.

## Outputs
- **PrintedReport** (File: `list164`): A printed report detailing freight table data, including company, customer, ship-to, rates, and surcharges.
- **LogisticsFile** (File: `FR946XL`): Disk file with detailed freight data for logistics analysis.
- **PricingFile** (File: `BICUFRC`): Disk file with condensed data for salesmen’s pricing reports.
- **ReturnStatus** (String): Success or error message (e.g., “Invalid company number”).

## Process Steps
1. **Validate Inputs**:
   - Ensure `CompanySelection` is `'ALL'` or `'CO'`.
   - If `'CO'`, validate `CompanyNumbers` exist in `BICONT` and are not deleted (`BCDEL ≠ 'D'`).
   - Ensure `CustomerSelection` is `'ALL'` or `'SEL'`.
   - If `'SEL'`, ensure at least one `CustomerNumbers` is non-zero.
   - Ensure `CurrentRecordsOnly` is `'CUR'` or blank.
   - Ensure `JobQueueFlag` is `'Y'`, `'N'`, or blank.
   - Set `NumberOfCopies` to 1 if 0.
   - Return error messages for invalid inputs.

2. **Summarize Sales Gallons**:
   - Filter sales data (`SA5FILD`, `SA5MOVD`) from `OneYearAgoDate` to `TodayDate`, excluding deleted records (`byte 1 ≠ 'D'`).
   - Sort by company (`SACO`), customer (`SACUST`), ship-to (`SASHIP`), location (`SALOC`), product (`SAPROD`), container (`SACNTR`), unit of measure (`SAUM`).
   - Accumulate gallons (`SANGAL`) and order counts by freight code (`SHFRCD`: `'C'` for collect, `'P'` for prepaid, `'A'` for prepaid & add).
   - Calculate average gallons per order (`pwavgl = pwtoga / pwnoor`) for each group.
   - Write to `PRICWK` with fields: `pwcono`, `pwcust`, `pwship`, `pwloc`, `pwprod`, `pwcntr`, `pwunms`, `pwfrcd`, `pwnoor`, `pwtoga`, `pwavgl`.

3. **Sort Freight Data**:
   - Sort `BICUFR` by company, customer, ship-to, and location, filtering by:
     - `CompanyNumbers` (if `CompanySelection = 'CO'`) or all companies.
     - `CustomerNumbers` (if `CustomerSelection = 'SEL'`) or all customers.
     - Current records (if `CurrentRecordsOnly = 'CUR'`, `BFSTD8 ≤ TodayDate ≤ BFEXD8`).
     - Exclude deleted records (`BFDEL ≠ 'D'`).
   - Output sorted data to `BI946S`.

4. **Calculate Freight Costs**:
   - For each `BI946S` record:
     - Retrieve company (`BICONT`), customer (`ARCUST`), ship-to (`SHIPTO`), and pricing (`PRICWK`) data.
     - Call `MBBFRT` with parameters (e.g., `f$FCO`, `f$FCUST`, `f$FSHIP`, `f$FCAID`, `f$FQTY`, `f$SVPROD`, `f$SVUM`).
     - Calculate:
       - Billed freight (`f$btot`): Sum of rates (`BFRPM`, `BFCWT`, `BFRPG`, `BFRPUM`), flat rate (`BFFLAT`), minimum (`BFMIN`), and fees (cleaning, pump, hose, pickup, tolls, detention).
       - Carrier freight (`f$ctot`): Sum of carrier rates (`CFRPM`, `CFCWT`, `CFRPG`, `CFRPUM`), flat rate (`CFFLAT`), minimum (`CFMIN`), and fees.
       - Fuel surcharge (`f$SCAM`): `BFRPM * f$SCPC` (fuel surcharge percentage).
       - Insurance amount (`f$INAM`): Based on `BFINSP` (insurance percentage).
       - Use constants: 6500 gallons for bulk, 25000 for rail.
     - Use average gallons (`pwavgl` from `PRICWK`) for calculations.

5. **Generate Outputs**:
   - **Printed Report (`list164`)**:
     - Include headers: program name, company name (`BCNAME`), customer number (`BFCUST`), customer name (`ARNAME`), date, time, page number.
     - Detail lines: `BFDEL`, `BFSHIP`, `BFCNTR`, `BFCAID`, `BFPR01–10`, rates (`BFRPM`, `BFCWT`, `BFRPG`, `BFRPUM`, `BFUM`), fees, surcharges (`BFSCHG`, `BFINSP`), and totals (`f$btot`).
     - If `BFACDF = 'Y'`, include carrier rates (`CFRPM`, `CFCWT`, etc.) and total (`f$ctot`).
     - Print `NumberOfCopies` copies.
   - **Logistics File (`FR946XL`)**:
     - Write detailed fields: keys (`BFDEL`, `BFCONO`, `BFCUST`, `BFSHIP`), rates, fees, totals (`f$btot`, `f$ctot`), customer name (`ARNAME`), etc.
   - **Pricing File (`BICUFRC`)**:
     - Write condensed fields: keys, product code (`PRODCD`), unit of measure (`BFUM`), average gallons (`pwavgl`), totals.

6. **Return Status**:
   - Return “Success” if completed, or an error message (e.g., “Invalid company number”) if validation fails.

## Business Rules
- **Input Validation**:
  - `CompanySelection`: Must be `'ALL'` or `'CO'`.
  - `CompanyNumbers`: Must exist in `BICONT`, not deleted.
  - `CustomerSelection`: Must be `'ALL'` or `'SEL'`.
  - `CustomerNumbers`: At least one non-zero if `'SEL'`.
  - `CurrentRecordsOnly`: Must be `'CUR'` or blank.
  - `JobQueueFlag`: Must be `'Y'`, `'N'`, or blank.
  - `NumberOfCopies`: Set to 1 if 0.
- **Data Filtering**:
  - Sales data: From `OneYearAgoDate` to `TodayDate`, exclude deleted records.
  - Freight data: Filter by `CompanyNumbers`, `CustomerNumbers`, `CurrentRecordsOnly`, exclude deleted records (`BFDEL ≠ 'D'`).
- **Calculations**:
  - Average gallons: `pwavgl = pwtoga / pwnoor` (rounded half-up).
  - Freight totals: Sum rates, flat rates, minimums, fees; apply fuel surcharge and insurance percentages.
  - Use 6500 (bulk) or 25000 (rail) gallons for calculations.
- **Output**:
  - Printed report: Detailed, with carrier costs if `BFACDF = 'Y'`.
  - Disk files: `FR946XL` for logistics, `BICUFRC` for pricing.

## Dependencies
- **Files**: `SA5FILD`, `SA5MOVD`, `SA5SHY`, `BICONT`, `ARCUST`, `SHIPTO`, `GSCNTR1`, `PRICWK`, `BICUFR`, `BI946S`, `list164`, `FR946XL`, `BICUFRC`.
- **External Program**: `MBBFRT` (freight calculations).

</xaiArtifact>

---

### Notes
- The **Generate Freight Table Report** use case was chosen as the primary function because it encapsulates the core business process of the call stack, integrating input validation, sales summarization, and freight calculations.
- The other use cases (input validation, sales summarization, freight calculations) are subsumed within this function as steps or dependencies.
- If you need a function requirement document for another use case (e.g., Summarize Sales Gallons), please specify, and I can provide a separate document.