The provided call stack (`BB715.ocl36.txt`, `BB715P.rpg36.txt`, `BB713.rpg36.txt`, `BB714.rpg36.txt`, `BB714A.rpg36.txt`, and `BB715.rpg36.txt`) orchestrates the generation of a "Daily Requirements Report" on an IBM System/36 or AS/400 system. Below, I will identify the use cases implemented by this call stack and then provide a function requirement document for a large function that processes inputs to produce the report without screen interaction, focusing on business requirements and necessary calculations.

### List of Use Cases Implemented

The call stack implements the following use cases, each representing a distinct business function within the process of generating the Daily Requirements Report:

1. **Input Validation for Report Parameters**:
   - **Description**: Validates user-provided parameters (company number, start date, job queue option, and number of report copies) to ensure they meet the requirements for report generation.
   - **Program**: `BB715P.rpg36.txt`.
   - **Details**: 
     - Validates the company number against `BICONT`, ensuring it exists and is not marked as deleted.
     - Ensures the start date is valid (MMDDYY format, month 01–12, day 01–31, not a weekend unless allowed).
     - Verifies the job queue parameter is ' ', 'N', or 'Y'.
     - Defaults the number of copies to 1 if zero.
     - Displays errors on a screen (`BB715PFM`) if validation fails, looping until corrected or canceled.

2. **Preprocessing Multi-Load and Non-Multi-Load Orders**:
   - **Description**: Processes order data to handle multi-load orders (splitting loads across days) and non-multi-load orders, creating an intermediate file for further processing.
   - **Program**: `BB713.rpg36.txt`.
   - **Details**:
     - Reads order headers (`BBORDR`) and details (`BBORDR2`).
     - For multi-load orders with a request date, calculates daily loads, skips weekends unless allowed, and writes daily records to `BB713M`.
     - For multi-load orders without a request date, writes a single record.
     - For non-multi-load orders, copies all detail lines to `BB713M`.

3. **Unit Conversion (Pounds to Gallons)**:
   - **Description**: Converts order quantities from pounds to gallons for specific products and container codes, updating the unit of measure and container code.
   - **Program**: `BB714.rpg36.txt`.
   - **Details**:
     - Processes `BB713M` records with `BDUM = 'LBS'` and `BDCNTR = '001'`, converting quantities using `GSUMCV` conversion factors.
     - Updates `BDUM` to 'GAL' and `BDCNTR` to '001'.
     - For `BDUM = 'GAL'` and `BDCNTR = '002'`, updates `BDCNTR` to '001' without conversion.
     - Flags product '4317' with 49,000 LBS for special handling.

4. **Adding Sort Codes for Report Organization**:
   - **Description**: Enhances `BB713M` records by adding container type (bulk/packaged) and customer sort codes for sorting and grouping in the report.
   - **Program**: `BB714A.rpg36.txt`.
   - **Details**:
     - Retrieves container type (`TCCNTY`, 'B' or 'P') from `GSCNTR1` using `BDCNTR`.
     - Retrieves product class (`TPPRCL`) from `GSPROD` and customer sort code (`TBCSRT`) from `GSTABL` using company and product class.
     - Updates `BB713M` with `CNTY` (position 521) and `CSRT` (position 523).

5. **Sorting Order Data**:
   - **Description**: Sorts the preprocessed order data by specified fields to organize the report by company, location, and other criteria.
   - **Program**: `#GSORT` (called in `BB715.ocl36.txt`).
   - **Details**:
     - Sorts `BB713M` into `BB715S` by company, status code, location, bulk/packaged, container, unit of measure, and product.
     - Filters out deleted records and applies range checks (positions 10–12 between '000' and '900') and company code matches.

6. **Generating the Daily Requirements Report**:
   - **Description**: Produces a formatted report showing daily and weekly order quantities, tank inventory, and available balance, grouped by company, location, and other criteria.
   - **Program**: `BB715.rpg36.txt`.
   - **Details**:
     - Reads sorted `BB715S` (mapped as `BBORDR`), retrieves inventory from `INTKRY`/`INTKRW` and `INCHRT`, and company/location names from `BICONT`/`INLOC`.
     - Calculates daily quantities (five days), weekly totals, and overage.
     - Computes tank inventory and available balance using tank measurements and capacities.
     - Outputs a printer file (`LIST`) with headers, detail lines, and totals.

### Function Requirement Document

Below is a function requirement document for a large function that encapsulates the entire report generation process, taking inputs programmatically (instead of via screen interaction) and producing the report. The function consolidates the use cases into a single process, focusing on business requirements and calculations.

<xaiArtifact artifact_id="b5ad8986-a374-45d2-bc8c-a1da1d3552fe" artifact_version_id="bc6d13ae-bcd5-4b25-b0d5-3c208c9fcccd" title="Daily_Requirements_Report_Function_Requirements.md" contentType="text/markdown">

# Daily Requirements Report Function Requirements

## Function Overview
The `GenerateDailyRequirementsReport` function generates a formatted report detailing daily and weekly order quantities, tank inventory, and available balance for a specified company and start date. It processes order data, converts units, adds sort codes, sorts records, and produces a printer-compatible output without screen interaction.

## Inputs
- **Company Number** (`KYCO`, 2 characters): A valid company code from the `BICONT` file.
- **Start Date** (`KYSTDT`, 6 digits, MMDDYY): The start date for the report, in Gregorian format.
- **Job Queue Option** (`KYJOBQ`, 1 character): ' ', 'N', or 'Y' to indicate job queue handling.
- **Number of Copies** (`KYCOPY`, numeric): Number of report copies (defaults to 1 if 0).
- **File Inputs**:
  - `BBORDR`, `BBORDR2`: Order header and detail files.
  - `BICONT`: Company data.
  - `INLOC`: Location data.
  - `INTKRY`, `INTKRW`: Inventory transaction files.
  - `GSCTUM`: Unit of measure data.
  - `INCHRT`: Tank chart data.
  - `GSCNTR1`: Container type data.
  - `GSPROD`: Product data.
  - `GSUMCV`: Unit conversion factors.
  - `GSTABL`: Table data for sort codes.

## Outputs
- **Report File** (`LIST`, printer format, 164 bytes): A formatted report with:
  - Headers: Company name, location, date, time, page number.
  - Detail Lines: Container, product, description, unit of measure, daily quantities (5 days), weekly total, overage, tank inventory, balance.
  - Totals: Unit of measure totals and differences.
- **Temporary Files**: `BB713M` (intermediate), `BB715S` (sorted).

## Process Steps
1. **Validate Inputs**:
   - Check `KYCO` exists in `BICONT` and is not deleted (`BCDEL ≠ 'D'`).
   - Validate `KYSTDT` is a valid date (month 01–12, day 01–31, considering leap years).
   - Ensure `KYSTDT` is not a Saturday or Sunday (unless allowed).
   - Verify `KYJOBQ` is ' ', 'N', or 'Y'.
   - Set `KYCOPY` to 1 if 0.
   - Return error if validation fails (e.g., "INVALID COMPANY NUMBER", "INVALID DATE ENTERED", "CANNOT START ON SATURDAY OR SUNDAY", "INVALID PARAMETER ENTERED").

2. **Preprocess Orders**:
   - Read `BBORDR` (headers) and `BBORDR2` (details).
   - For multi-load orders (`BOMULO = 'Y'`):
     - If `BORQDT ≠ 0`, calculate daily loads (`BOTOLO / BOLODA`, round up if remainder), skip weekends unless `BOWEPU = 'Y'`, and write daily records to `BB713M` with calculated volume (`BOLOVO * loads`) and dates.
     - If `BORQDT = 0`, write a single record with `BORQD8`.
   - For non-multi-load orders (`BOMULO ≠ 'Y'`), write all matching detail lines to `BB713M`.
   - Account for shipped loads (`(BOTOLO * BOLOVO - BDQTY) / BOLOVO`).

3. **Convert Units**:
   - For `BB713M` records with `BDUM = 'LBS'` and `BDCNTR = '001'`:
     - Chain `GSUMCV` with key (`BDCO`, `BDPROD`, 'LBSGAL') to get conversion factor (`UCCVFA`).
     - Convert quantity (`BDQTY / UCCVFA`) to gallons, update `BDUM = 'GAL'`, `BDCNTR = '001'`.
   - For `BDUM = 'GAL'` and `BDCNTR = '002'`, update `BDCNTR = '001'`.
   - Flag product '4317' with `BDQTY = 49000` and `BDUM = 'LBS'` for special handling (no conversion).

4. **Add Sort Codes**:
   - Chain `GSCNTR1` with `BDCNTR` to get container type (`TCCNTY`, 'B' or 'P'), write to position 521 (`CNTY`).
   - Chain `GSPROD` with `BDCO` and `BDPROD` to get product class (`TPPRCL`).
   - Chain `GSTABL` with `BDCO` and `TPPRCL` (type 'PRODCL') to get customer sort code (`TBCSRT`), write to position 523 (`CSRT`).

5. **Sort Data**:
   - Sort `BB713M` into `BB715S` by:
     - Company (2–3).
     - Status code (522–523).
     - Location (22–24).
     - Bulk/packaged (521).
     - Container (121–123).
     - Unit of measure (51–53).
     - Product (25–28).
   - Exclude deleted records (`position 1 ≠ 'D'`) and records outside range (10–12 not between '000' and '900') or non-matching company.

6. **Generate Report**:
   - Read `BB715S` (as `BBORDR`), group by company (`BDCO`), location (`BDLOC`), sort code (`REFBLD`), bulk/packaged (`BLKPKG`), container (`BDCNTR`), unit of measure (`BDUM`), and product (`BDPROD`).
   - Retrieve company name (`BCNAME`) from `BICONT` and location name (`ILNAME`) from `INLOC`.
   - For each group:
     - Chain `INTKRY`/`INTKRW` with `IXCO`, `IXLOC`, `IXPRCD`, `IXTANK`, `IXCNTR` to get inventory quantities (`IXQTFT`, `IXQTSE`) and measurements (`IXTOFT`, `IXTOIN`).
     - Chain `INCHRT` to get tank capacity (`TCSP1`).
     - Calculate available inches (`AVALIN = IXTOFT - TCSP1`).
     - Calculate available product (`FACT2 = (AVALIN * IXQTTA) / IXTOFT`).
     - If `INCHRT` lookup fails, use `IXQTSE` for inventory.
     - Accumulate daily quantities (`D1QTY` to `D5QTY`) for five days from `KYSTDT` based on `BDRQD8`.
     - Compute weekly total (`L1QTY`, `L2QTY`) and overage (`OVQTY`, `L2VQTY`).
     - Calculate balance (`L1DIFF = L1XINV - L1QTY - OVQTY`, `L2DIFF = L2XINV - L2QTY - L2VQTY`).
   - Output to `LIST`:
     - Headers: Company, location, date, time, page, day names, and column labels.
     - Detail Lines: Container, product, description, unit of measure, daily quantities, weekly total, overage, tank inventory, balance.
     - Totals: Unit of measure totals and differences.

7. **Cleanup**:
   - Delete temporary files `BB713M` and `BB715S`.

## Business Rules
1. **Input Validation**:
   - Company must exist in `BICONT` and not be deleted.
   - Start date must be valid (MMDDYY, month 01–12, day 01–31, not Saturday/Sunday unless allowed).
   - Job queue must be ' ', 'N', or 'Y'.
   - Copies default to 1 if 0.

2. **Order Processing**:
   - Multi-load orders (`BOMULO = 'Y'`) with a request date are split into daily records, skipping weekends unless `BOWEPU = 'Y'`.
   - Multi-load orders without a request date produce one record.
   - Non-multi-load orders include all detail lines.
   - Account for shipped loads in calculations.

3. **Unit Conversion**:
   - Convert `LBS` to `GAL` for `BDCNTR = '001'` using `GSUMCV` factor.
   - Standardize `BDCNTR = '002'` to '001' for gallons.
   - Flag product '4317' with 49,000 LBS for special handling.

4. **Sorting**:
   - Group by company, status, location, bulk/packaged, container, unit of measure, and product.
   - Exclude deleted records and invalid ranges.

5. **Report Generation**:
   - Group by company, location, sort code, bulk/packaged, container, unit of measure, and product.
   - Include inventory from `INTKRY`/`INTKRW` and `INCHRT`.
   - Calculate available product based on tank capacity and measurements.
   - Report daily quantities (5 days), weekly totals, overage, tank inventory, and balance.

## Calculations
- **Daily Loads**: `REMDYS = BOTOLO / BOLODA`, round up if remainder.
- **Load Volume**: `LOVO = BOLODA * BOLOVO`.
- **Shipped Loads**: `SHLO = (BOTOLO * BOLOVO - BDQTY) / BOLOVO`.
- **Gallons Conversion**: `QTY = BDQTY / UCCVFA` (from `GSUMCV`).
- **Available Inches**: `AVALIN = IXTOFT - TCSP1` (from `INCHRT`).
- **Available Product**: `FACT2 = (AVALIN * IXQTTA) / IXTOFT`.
- **Balance**: `L1DIFF = L1XINV - L1QTY - OVQTY`, `L2DIFF = L2XINV - L2QTY - L2VQTY`.

## Error Handling
- Return specific error messages for invalid inputs (company, date, job queue).
- Skip records with missing reference data (`GSCNTR1`, `GSPROD`, `GSTABL`, `INCHRT`) or use defaults (e.g., `IXQTSE` for inventory).
- Log failures for missing files or records.

## Assumptions
- Input files are pre-populated and accessible.
- Dates are processed in Gregorian and Julian formats for calculations.
- The report covers a five-day period starting from `KYSTDT`.
- Inventory data is reliable; fallback to `IXQTSE` if `INCHRT` is unavailable.

</xaiArtifact>

### Notes
- The function assumes programmatic input, bypassing the interactive screen (`BB715PFM`) used in `BB715P`.
- The call stack’s modularity (`BB713`, `BB714`, `BB714A`, `#GSORT`, `BB715`) is consolidated into a single function for simplicity.
- The `MINCHT` program (implied in OCL) is assumed to be part of inventory processing within `BB715`, but its specifics are not detailed due to missing code.
- The report format is preserved from `BB715.rpg36.txt`, with headers and totals as specified.

If you need further clarification, additional use cases, or specific code implementations (e.g., in a modern language), let me know!