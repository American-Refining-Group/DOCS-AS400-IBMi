The call stack provided consists of three programs: `BI947P.ocl36.txt` (OCL), `BI947P.rpgle.txt` (RPG for input validation), and `BI947.ocl36.txt` with `BI947.rpgle.txt` (OCL and RPG for sorting and report generation). These programs work together to produce a "Customer Freight Table List by Carrier ID." Below, I will identify the use cases implemented by this call stack and then provide a function requirement document for one use case, assuming it operates as a large function with direct inputs instead of screen interaction.

---

### Use Cases Implemented by the Call Stack

The call stack implements the following use cases, each representing a distinct business process supported by the programs:

1. **Generate Freight Table Report by Carrier ID (All Companies, All Carriers)**:
   - **Description**: Produces a report listing all freight table records sorted by carrier ID for all companies, including customer details, rates, surcharges, and carrier-specific costs.
   - **Inputs**: Selection for all companies (`KYALCO='ALL'`), all carriers (`KYALCS='ALL'`), job queue flag (`KYJOBQ`), copy count (`KYCOPY`), and optional current records flag (`KYCUR`).
   - **Process**: Validates inputs, sorts the freight table (`BICUFR`) by carrier ID, retrieves related data (customer, ship-to, carrier), calculates freight amounts, and outputs a formatted report (`list164`) and optionally a disk file (`FR947XL`) for logistics analysis.
   - **Output**: Printed report and disk file with freight details.

2. **Generate Freight Table Report for Specific Companies (All Carriers)**:
   - **Description**: Produces a report for specific company numbers (`KYCO1`, `KYCO2`, `KYCO3`) and all carriers, including freight rates and surcharges.
   - **Inputs**: Specific company numbers (`KYALCO='CO'`, `KYCO1–3`), all carriers (`KYALCS='ALL'`), job queue flag, copy count, and current records flag.
   - **Process**: Validates company numbers against `BICONT`, sorts `BICUFR` for selected companies, retrieves related data, calculates freight, and generates report outputs.
   - **Output**: Printed report and disk file filtered by company.

3. **Generate Freight Table Report for Specific Carriers (All Companies)**:
   - **Description**: Produces a report for all companies but specific carrier IDs (`CS` array), focusing on selected carriers’ freight details.
   - **Inputs**: All companies (`KYALCO='ALL'`), specific carrier IDs (`KYALCS='SEL'`, `CS`), job queue flag, copy count, and current records flag.
   - **Process**: Validates carrier IDs, sorts `BICUFR` by carrier ID, retrieves related data, calculates freight, and generates report outputs.
   - **Output**: Printed report and disk file filtered by carrier.

4. **Generate Freight Table Report for Specific Companies and Carriers**:
   - **Description**: Produces a report for specific company numbers and specific carrier IDs, providing a targeted view of freight agreements.
   - **Inputs**: Specific company numbers (`KYALCO='CO'`, `KYCO1–3`), specific carrier IDs (`KYALCS='SEL'`, `CS`), job queue flag, copy count, and current records flag.
   - **Process**: Validates company and carrier inputs, sorts `BICUFR` for the selected subset, retrieves related data, calculates freight, and generates report outputs.
   - **Output**: Printed report and disk file for the specified companies and carriers.

5. **Generate Freight Table Report for Current Records Only**:
   - **Description**: Filters the freight table to include only records within valid start and end dates (`KYCUR='CUR'`), for any combination of companies and carriers.
   - **Inputs**: `KYALCO`, `KYCO1–3`, `KYALCS`, `CS`, `KYJOBQ`, `KYCOPY`, and `KYCUR='CUR'`.
   - **Process**: Validates inputs, filters `BICUFR` for current records (based on `BFSTD8` and `BFEXD8`), sorts, retrieves related data, calculates freight, and generates report outputs.
   - **Output**: Printed report and disk file with current records only.

6. **Batch Processing of Freight Table Report**:
   - **Description**: Submits the report generation to a job queue for batch processing, allowing asynchronous execution for large datasets.
   - **Inputs**: Same as above use cases, with `KYJOBQ='Y'`.
   - **Process**: Validates inputs, submits the sort and report generation (`BI947`) to the job queue, and produces outputs without immediate user interaction.
   - **Output**: Same as above, processed in batch mode.

7. **Logistics Analysis Data Export**:
   - **Description**: Generates a disk file (`FR947XL`) with detailed freight data for logistics analysis, alongside the printed report.
   - **Inputs**: Same as above use cases, with output to `FR947XL` enabled.
   - **Process**: Includes additional output to `FR947XL` with fields like rates, surcharges, and freight totals for analysis.
   - **Output**: Disk file (`FR947XL`) alongside the printed report.

---

### Function Requirement Document

**Function Name**: GenerateFreightTableReport

**Purpose**: Generate a comprehensive freight table report sorted by carrier ID, including customer details, freight rates, surcharges, and carrier-specific costs, with optional filtering by company, carrier, and current records, and output to both a printed report and a disk file for logistics analysis.

**Inputs**:
- `company_selection` (string): 'ALL' or 'CO' (specific companies).
- `company_numbers` (array of integers): Up to three company numbers (if `company_selection='CO'`).
- `carrier_selection` (string): 'ALL' or 'SEL' (specific carriers).
- `carrier_ids` (array of strings): Up to 10 carrier IDs (if `carrier_selection='SEL'`).
- `current_records_only` (string): 'CUR' or blank (to filter by valid start/end dates).
- `job_queue` (string): 'Y' or 'N' (for batch processing).
- `copy_count` (integer): Number of report copies (minimum 1).

**Outputs**:
- Printed report (`list164`): Formatted report with headers and detail lines.
- Disk file (`FR947XL`): Structured data for logistics analysis.

<xaiArtifact artifact_id="6ca7c363-6d4a-4c59-acf1-e528a61b7bf9" artifact_version_id="09315ebf-3496-4ac2-ba24-a262b06422bc" title="GenerateFreightTableReport_Requirements.md" contentType="text/markdown">

# Function Requirement: GenerateFreightTableReport

## Purpose
Generate a freight table report sorted by carrier ID, including customer details, freight rates, surcharges, and carrier costs, with optional filtering by company, carrier, and current records, outputting to a printed report and a disk file for logistics analysis.

## Inputs
- **company_selection** (string): 'ALL' (all companies) or 'CO' (specific companies).
- **company_numbers** (array of integers): Up to three company numbers (if `company_selection='CO'`).
- **carrier_selection** (string): 'ALL' (all carriers) or 'SEL' (specific carriers).
- **carrier_ids** (array of strings): Up to 10 carrier IDs (if `carrier_selection='SEL'`).
- **current_records_only** (string): 'CUR' (filter by valid start/end dates) or blank.
- **job_queue** (string): 'Y' (batch processing) or 'N' (immediate).
- **copy_count** (integer): Number of report copies (minimum 1).

## Outputs
- **Printed Report** (`list164`): Formatted report with headers (report title, company name, date, time, carrier name) and detail lines (customer number, ship-to, location, container code, product codes, rates, surcharges, freight totals).
- **Disk File** (`FR947XL`): Structured data with freight rates, surcharges, and totals for logistics analysis.

## Process Steps
1. **Validate Inputs**:
   - Ensure `company_selection` is 'ALL' or 'CO'. If 'CO', validate `company_numbers` against `BICONT` (non-deleted records).
   - Ensure `carrier_selection` is 'ALL' or 'SEL'. If 'SEL', validate `carrier_ids` (non-blank).
   - Ensure `current_records_only` is 'CUR' or blank.
   - Ensure `job_queue` is 'Y' or 'N'.
   - Set `copy_count` to 1 if zero or invalid.
2. **Clear Temporary Files**:
   - Delete or clear `FRTRAT`, `BICUFRO`, `FR947XL` to ensure fresh data.
3. **Sort Freight Data**:
   - Sort `BICUFR` by company number and carrier ID, excluding deleted records (`BFDEL ≠ 'D'`).
   - Filter by `company_numbers` (if `company_selection='CO'`) and `carrier_ids` (if `carrier_selection='SEL'`).
   - If `current_records_only='CUR'`, include only records with valid start/end dates (`BFSTD8` to `BFEXD8`).
   - Output sorted data to `BI947S`.
4. **Retrieve Related Data**:
   - Fetch company names from `BICONT`.
   - Fetch customer names and zip codes from `ARCUST` and `SHIPTO`.
   - Fetch carrier names from `BBCAID`.
5. **Calculate Freight**:
   - Call `MBBFRT` with parameters (company, customer, ship-to, carrier ID, product codes, etc.) to compute freight amounts (`f$btot`, `f$ctot`), surcharges (e.g., fuel, insurance), and carrier costs.
6. **Generate Outputs**:
   - **Printed Report** (`list164`):
     - Print headers for each carrier ID, including report title, company name, date, time, and carrier name.
     - Print detail lines with customer number, ship-to, location, container code, product codes, rates (per mile, hundredweight, gallon, unit), surcharges (cleaning, pump, hose, etc.), and freight totals.
     - Handle page breaks on carrier change or overflow.
   - **Disk File** (`FR947XL`):
     - Write fields like rates, surcharges, carrier codes, customer name, and freight totals for logistics analysis.
7. **Batch Processing** (if `job_queue='Y'`):
   - Submit sorting and report generation to the job queue for asynchronous execution.
8. **Cleanup**:
   - Reset switches and clear local variables.

## Business Rules
- **Input Validation**:
  - `company_selection` must be 'ALL' or 'CO'.
  - `company_numbers` must exist in `BICONT` and not be deleted (`BCDEL ≠ 'D'`).
  - `carrier_selection` must be 'ALL' or 'SEL'.
  - `carrier_ids` must be non-blank if `carrier_selection='SEL'`.
  - `current_records_only` must be 'CUR' or blank.
  - `job_queue` must be 'Y' or 'N'.
  - `copy_count` defaults to 1 if invalid.
- **Data Filtering**:
  - Exclude deleted records (`BFDEL ≠ 'D'`).
  - Filter by `company_numbers` if `company_selection='CO'`.
  - Filter by `carrier_ids` if `carrier_selection='SEL'`.
  - Filter by current records if `current_records_only='CUR'` (within valid dates).
- **Freight Calculations**:
  - Use `MBBFRT` to calculate freight amounts (`f$btot`, `f$ctot`) based on rates (per mile, hundredweight, gallon, unit), surcharges (fuel, cleaning, pump, etc.), and insurance percentages.
  - Display surcharges separately on invoices if specified (`BFSCSP`).
- **Report Formatting**:
  - Group by carrier ID with headers.
  - Include customer, ship-to, location, container code, product codes, rates, and surcharges.
  - Show carrier-specific rates and cost differences (`BFACDF`).
- **Output**:
  - Generate `copy_count` printed reports.
  - Write logistics data to `FR947XL` if enabled.

## Calculations
- **Freight Amounts** (`f$btot`, `f$ctot`):
  - Computed by `MBBFRT` using inputs like company number, customer number, ship-to, carrier ID, quantity, and rates (e.g., `BFRPM`, `BFCWT`, `BFRPUM`).
  - Include surcharges (e.g., fuel: `f$SCAM`, insurance: `f$INAM`, carrier equivalents).
- **Surcharges**:
  - Fuel surcharge (`f$SCAM`): Based on `BFSCHG` and `f$SCPC` (percentage).
  - Insurance amount (`f$INAM`): Based on `BFINSP` (percentage).
  - Similar calculations for carrier-specific surcharges (`f$CSCAM`, `f$CINAM`).

</xaiArtifact>

---

### Notes
- The function assumes direct input parameters, replacing the interactive screen (`SCREEN` in `BI947P.rpgle.txt`) with a structured input format.
- The use cases cover all combinations of company and carrier filtering, current records, and batch processing, as implemented in the call stack.
- The truncation of `BI947.rpgle.txt` limits visibility into the exact calculation logic (e.g., `MBBFRT` call), but the data structures and context provide sufficient detail for the requirements.
- If additional details (e.g., full `BI947.rpgle.txt` or `MBBFRT` logic) are available, I can refine the calculations or process steps further.