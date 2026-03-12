The call stack for the rack price list processing consists of four programs: `BB951P.rpg36.txt`, `BB951.ocl36.txt`, `BB9511.rpg36.txt`, and `BB9518.rpg36.txt`. These programs collectively generate a rack price report that displays the last two price changes for products, incorporating tax and cost information, and store the output in a table for salesmen’s use. Below, I’ll identify the **use cases** implemented by this call stack and then provide a **function requirement document** for a non-interactive function that encapsulates the primary use case, focusing on business requirements and necessary calculations.

---

### Use Cases Implemented by the Call Stack

The call stack implements the following use cases, derived from the functionality of the programs and their interactions:

1. **Generate Rack Price Report**:
   - **Description**: Produces a printed report listing the last two price changes for products by company, location, product group, container, and unit of measure, including tax rates, delivery flags, and cost data, formatted for easy customer quoting (two lines per product/container/unit of measure). The report is hierarchical (company, location, product group) and includes separators and headers.
   - **Components Involved**:
     - `BB951P.rpg36`: Collects and validates user input (company, location range, product range, container range, date range, job queue flag, copy count).
     - `BB951.ocl36`: Orchestrates the process, performing sorts, building temporary files, and invoking preprocessing (`BB9511`, `BB9518`) and printing (`BB951`).
     - `BB9511.rpg36`: Preprocesses rack price data, filtering by company, enriching with product descriptions and sort codes, and handling quantities/prices.
     - `BB9518.rpg36`: Gathers tax information for products by company and product group, including tax codes, descriptions, and rates.
     - `BB951.rpg36`: Prints the final report with hierarchical headers, price changes, tax details, and stores data in `BBPRCEC` for salesmen.
   - **Inputs**: Company number, location range, product range, container range, date range, job queue flag, copy count.
   - **Outputs**: Printed report (`LIST`), data table (`BBPRCEC`).

2. **Validate User Input for Report Parameters**:
   - **Description**: Validates user-specified parameters (company, location, product, container, date ranges, job queue flag) against master files to ensure data integrity before generating the report.
   - **Components Involved**:
     - `BB951P.rpg36`: Performs input validation using `BICONT` (company), `INLOC` (location), `GSPROD` (product), `GSCNTR1` (container), and checks job queue flag (`KYJOBQ`).
   - **Inputs**: User input via screen (company number, location from/to, product from/to, container from/to, date from/to, job queue flag, copy count).
   - **Outputs**: Validated parameters or error messages displayed on the screen.

3. **Store Rack Price Data for Salesmen**:
   - **Description**: Stores rack price report data (company, location, product, prices, quantities, descriptions, tax, cost) in a table (`BBPRCEC`) for use in salesmen’s pricing reports.
   - **Components Involved**:
     - `BB951.rpg36`: Writes processed data to `BBPRCEC` during report generation.
     - `BB9511.rpg36`: Preprocesses data, providing enriched records used in `BBPRCEC`.
     - `BB9518.rpg36`: Provides tax data included in `BBPRCEC`.
   - **Inputs**: Preprocessed rack price and tax data.
   - **Outputs**: `BBPRCEC` table with report data.

4. **Prevent Concurrent Report Execution**:
   - **Description**: Ensures that only one instance of the rack price report process runs at a time to avoid data conflicts or resource issues.
   - **Components Involved**:
     - `BB951.ocl36`: Checks if `BB951` is active and cancels the job if another workstation is running it, displaying a message to try again later.
   - **Inputs**: System status of `BB951` program.
   - **Outputs**: Message and job cancellation if active.

---

### Function Requirement Document

Below is a **function requirement document** for a non-interactive function that encapsulates the primary use case: **Generate Rack Price Report**. The function takes inputs programmatically (instead of via a screen) and produces the report and table output. The document focuses on business requirements, process steps, and calculations, presented concisely.

<xaiArtifact artifact_id="e9b05255-32c7-4560-9d37-d345fa6fc5e1" artifact_version_id="48e37ea8-243e-4f32-a718-112749d8b67d" title="RackPriceReportFunction.md" contentType="text/markdown">

# Function Requirement Document: Generate Rack Price Report

## Function Name
`GenerateRackPriceReport`

## Purpose
Generate a rack price report displaying the last two price changes for products by company, location, product group, container, and unit of measure, including tax rates, delivery flags, and cost data, formatted within two lines per product/container/unit of measure for customer quoting. Store the data in a table for salesmen’s pricing reports.

## Inputs
- **Company Number** (`KYCO`, 2 bytes, numeric): Target company (required).
- **Location Range** (`KYLOFR`, `KYLOTO`, 3 bytes each, alphanumeric): From/to location codes (optional; blank for all).
- **Product Range** (`KYPRFR`, `KYPRTO`, 4 bytes each, alphanumeric): From/to product codes (optional; blank for all).
- **Container Range** (`KYCNFR`, `KYCNTO`, 3 bytes each, alphanumeric): From/to container codes (optional; blank for all).
- **Date Range** (`KYFRDT`, `KYTODT`, 6 bytes each, MMDDYY): From/to dates (optional; blank for all).
- **Job Queue Flag** (`KYJOBQ`, 1 byte, alphanumeric): `'Y'`, `'N'`, or blank (optional; default blank).
- **Copy Count** (`KYCOPY`, 2 bytes, numeric): Number of report copies (default 1 if 0).

## Outputs
- **Printed Report** (`LIST`): Hierarchical report (company, location, product group) with headers, last two price changes, quantities, tax details, and separators.
- **Data Table** (`BBPRCEC`): Stores report data (company, location, product, prices, quantities, descriptions, tax, cost) for salesmen.

## Process Steps
1. **Validate Inputs**:
   - Check `KYCO` exists in `BICONT` and is not deleted (`BCDEL ≠ 'D'`).
   - If `KYLOFR`/`KYLOTO` non-blank, verify in `INLOC` for `KYCO`.
   - If `KYPRFR`/`KYPRTO` non-blank, verify in `GSPROD` for `KYCO`.
   - If `KYCNFR`/`KYCNTO` non-blank, verify in `GSCNTR1`.
   - Ensure `KYJOBQ` is blank, `'Y'`, or `'N'`.
   - Set `KYCOPY` to 1 if 0.
   - Convert `KYFRDT`, `KYTODT` (MMDDYY) to YYYYMMDD (`KYFRD8`, `KYTOD8`): Prefix year with `19` if ≥ 80, else `20`.

2. **Check Concurrent Execution**:
   - Verify no other instance of the report process is running. If active, return error: “Report already running by another workstation.”

3. **Preprocess Rack Price Data**:
   - Filter `BBPRCE` by `KYCO`, non-deleted records (`RKDEL ≠ 'D'`), and optional ranges (`KYLOFR`/`KYLOTO`, `KYPRFR`/`KYPRTO`, `KYCNFR`/`KYCNTO`, `KYFRD8`/`KYTOD8`).
   - Retrieve product descriptions (`TPDES1`, `TPABDS`), product group (`TPPRGP`), and product class (`TPPRCL`) from `GSPROD`.
   - Retrieve inventory sort code (`TBCSRT`) from `GSTABL` using `TPPRCL`.
   - Store current prices (`RKPR`), quantities (`RKQT`), rack required (`RKRKRQ`), inactive flag (`RKINAC`) in temporary file `BB9511`.

4. **Gather Tax Information**:
   - Filter `GSPRD4` by `KYCO`, non-deleted records (`TBDEL ≠ 'D'`), and non-blank product group (`TBPRGP`).
   - Retrieve product group description (`TXDES1`) from `GSTABL`.
   - For each product code (`TBCODE`), retrieve tax codes (`TXC`), descriptions (`TXD`), rates (`TXR`), and delivery flag (`PTDELV`) from `BIPRT1` and `BISLTX` (exclude `TXC = 'T'`).
   - Store in temporary file `BB9518`.

5. **Generate Report and Table**:
   - Read `BB9511` records, grouping by company (`XXCO`), location (`XXLOC`), product group (`XXPRGR`).
   - Retrieve company name (`BCNAME`) from `BICONT`, location name (`ILNAME`) from `INLOC`, container description (`TCDESC`) from `GSCNTR1`, and cost data (`GICPGP`, `GICPGC`) from `ICSUMHY` using last closed month (`GCLMCC`) from `GLCONT`.
   - Retrieve tax data from `BB9518` (match on `XXCO`, `XXPRGR`).
   - Format quantities (`XXQT`) into ranges (5-digit, per FHL082198).
   - Print report (`LIST`):
     - Headers: Program ID, company, location, product group, date, time, column headers (container, quantities, prices, min qty).
     - Detail Lines: Product, description, container, unit of measure, current prices (`XXPR`), previous prices (`PRPR`), quantities, minimum quantity, rack required (`XXRKRQ`), inactive flag (`XXINAC`), effective date/time.
     - Tax: State, product group description, delivery flag, total tax rate, up to 10 tax codes/rates.
     - Separators: Lines between locations.
   - Write to `BBPRCEC`: Company, location, product, container, unit of measure, prices, quantities, descriptions, tax, cost, flags.

6. **Clean Up**:
   - Delete temporary files (`BB951T`, `BB951U`, `BB9511`, `BB951S`, `BB9518`).

## Business Rules
- **Validation**:
  - Company must exist, be non-deleted in `BICONT`.
  - Location, product, container ranges must exist in `INLOC`, `GSPROD`, `GSCNTR1` if specified.
  - Job queue flag must be blank, `'Y'`, or `'N'`.
  - Copy count defaults to 1 if 0.
- **Filtering**:
  - Exclude deleted records (`RKDEL`, `TBDEL`, `PTDEL ≠ 'D'`).
  - Process only specified company (`KYCO`).
  - Apply optional ranges for location, product, container, date.
- **Data Enrichment**:
  - Retrieve descriptions for products (`GSPROD`), containers (`GSCNTR1`), product groups (`GSTABL`), company (`BICONT`), location (`INLOC`).
  - Include up to 10 tax codes/rates, excluding sales tax (`TXC ≠ 'T'`).
  - Retrieve cost per gallon for last closed month.
- **Report Format**:
  - Hierarchical: Company, location, product group.
  - Two lines per product/container/unit of measure: Current prices/quantities, previous prices.
  - Include flags: Inactive (`XXINAC = 'I' or 'B'`), rack not required (`XXRKRQ`).
- **Output**:
  - Print report with headers, detail lines, tax info, separators.
  - Store data in `BBPRCEC` for salesmen.

## Calculations
- **Date Conversion**:
  - Convert MMDDYY (`KYFRDT`, `KYTODT`) to YYYYMMDD (`KYFRD8`, `KYTOD8`): Year ≥ 80 → `19xx`, else `20xx`.
  - Calculate today’s date and 30 days prior (YYYYMMDD) using Gregorian-to-Julian conversion:
    - Gregorian to Julian: `(Year - 1900) * 365.25 + (Month - 3) * 30.6 + Day`.
    - Subtract 31 days, convert back to Gregorian.
- **Total Tax Rate** (`TOTTXR`):
  - Sum up to 10 `TXR` values from `BB9518`.
- **Quantity Ranges**:
  - Format `XXQT` (5 elements, 7.0) into 5-digit ranges (e.g., `19999+1=20000`, per FHL082198).

## Error Handling
- Return error messages for invalid company, location, product, container, or job queue flag.
- Cancel if report is already running on another workstation.

## Dependencies
- **Files**:
  - Input: `BBPRCE`, `GSPRD4`, `BICONT`, `INLOC`, `GSTABL`, `GSPROD`, `GSCNTR1`, `BIPRT1`, `BISLTX`, `GLCONT`, `ICSUMHY`.
  - Temporary: `BB951T`, `BB951U`, `BB9511`, `BB951S`, `BB9518`.
  - Output: `LIST` (printer), `BBPRCEC` (table).
- **System**: IBM System/36 environment, `#GSORT` utility.

</xaiArtifact>

---

### Notes
- The primary use case (Generate Rack Price Report) encompasses the entire process, as it’s the main goal of the call stack. The other use cases (validation, data storage, concurrency check) are supportive but distinct functionalities.
- The function assumes programmatic input, replacing the interactive screen (`BB951P.rpg36`). Validation remains critical to ensure data integrity.
- Due to the truncation of `BB951.rpg36`, some logic (e.g., exact looping, cost calculations) was inferred from context and output specifications. If you provide the full code, I can refine the document further.
- If additional use cases need separate function documents or more details, please specify!