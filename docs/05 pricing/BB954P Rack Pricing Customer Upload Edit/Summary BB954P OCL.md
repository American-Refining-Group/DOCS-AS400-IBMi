The provided call stack consists of an OCL program (`BB954P.ocl36.txt`) and three RPG programs (`BB9541.rpg36.txt`, `BB954P.rpg36.txt`, `BB9542.rpg36.txt`) that together implement a rack pricing update process for "DA Lubricants" on an IBM System/3x or AS/400 system. Below, I will identify the use cases implemented by this call stack and then provide a concise function requirement document for one of the use cases, assuming it operates as a large function with inputs rather than screen interaction.

### List of Use Cases

The call stack implements the following use cases, each representing a distinct business function within the rack pricing update process:

1. **Generate Temporary Customer List**:
   - **Description**: Creates a temporary file (`BB954S`) containing unique, active customer records (company and customer numbers) from the customer master file (`PRCTUM`).
   - **Program Involved**: `BB9541`
   - **Purpose**: Prepares a filtered list of valid customers for subsequent validation and pricing updates, ensuring only relevant customers are processed.
   - **Key Actions**:
     - Reads `PRCTUM` and writes unique records to `BB954S`, marking them active ('A').
     - Avoids duplicates by checking existing records in `BB954S`.

2. **Validate User Inputs for Pricing Update**:
   - **Description**: Validates user-provided inputs (company number, customer number, date, job queue, copy count, and password) and prepares customer data for display and further processing.
   - **Program Involved**: `BB954P`
   - **Purpose**: Ensures that only valid data is used for pricing updates, preventing errors in downstream processing.
   - **Key Actions**:
     - Populates a list of up to 10 customer numbers and names from `BB954S` and `ARCUST`.
     - Validates company number (`KYCO`) against `BICONT`, customer number (`KYCUST`) against the customer list, month (`KYMO`) between 01–12, job queue (`KYJOBQ`) as 'Y', 'N', or blank, copy count (`KYCOPY`) > 0, and password (`KYPASS`) as 'CUSTPROD' or blank.
     - Handles cancellation by setting `KYCANC` to 'CANCEL'.
     - Adjusts dates for Y2K compliance.

3. **Update Rack Pricing and Generate Report**:
   - **Description**: Processes new pricing data from `DAPRCUP`, updates the customer master (`PRCTUM`) and permanent pricing file (`BBPRCE`), and generates a report summarizing the updates.
   - **Program Involved**: `BB9542`
   - **Purpose**: Applies new pricing data to the system, ensures data integrity, and provides an audit trail via a printed report.
   - **Key Actions**:
     - Validates company numbers against `BICONT`.
     - Checks for prior prices in `BBPRCEP`.
     - Updates `PRCTUM` with new customer records if needed.
     - Updates `BBPRCE` with new prices if the password is valid.
     - Generates a report with pricing details and totals.

4. **Orchestrate Pricing Update Workflow**:
   - **Description**: Coordinates the execution of the above use cases, managing file operations, program calls, and cleanup.
   - **Program Involved**: `BB954P.ocl36`
   - **Purpose**: Ensures the entire pricing actualización process runs smoothly, with proper sequencing and error handling.
   - **Key Actions**:
     - Sets processing mode based on `?1?` parameter ('POST' or default).
     - Deletes temporary file `BB954S` if it exists.
     - Calls `BB9541`, `BB954P`, and `BB9542` in sequence.
     - Checks for cancellation (`KYCANC = 'CANCEL'`).
     - Cleans up `BB954S` and resets variables.

### Function Requirement Document

**Function Name**: UpdateRackPricing

**Description**: Updates the rack pricing data for DA Lubricants by processing new pricing inputs, validating them, updating customer and pricing files, and generating a report, without requiring screen interaction.

**Inputs**:
- `company_number` (2 characters): Company identifier.
- `customer_number` (6 digits): Customer identifier.
- `month` (2 digits): Month of the pricing update (01–12).
- `job_queue` (1 character): Job queue indicator ('Y', 'N', or blank).
- `copy_count` (2 digits): Number of report copies (default 01 if 00).
- `password` (10 characters): Authorization password ('CUSTPROD' or blank).
- `pricing_data` (list of records): New pricing data with fields:
  - `company` (2 characters)
  - `location` (3 characters)
  - `product_code` (4 characters)
  - `container` (3 characters)
  - `unit_of_measure` (3 characters)
  - `date` (8 digits, CYMD format)
  - `time` (4 digits, HHMM)
  - `price` (9.4 numeric)
  - `quantity_description` (10 characters)

**Outputs**:
- Updated `PRCTUM` file with new customer records.
- Updated `BBPRCE` file with new pricing records.
- Printed report summarizing pricing updates, including totals.
- Return status: Success, Failure (with error message), or Cancelled.

**Process Steps**:
1. **Validate Inputs**:
   - Check if `company_number` exists in `BICONT`. If not, return "INVALID COMPANY NUMBER".
   - Verify `customer_number` exists in `PRCTUM` or `BB954S`. If not, return "INVALID CUSTOMER NUMBER".
   - Ensure `month` is between 01 and 12. If not, return "INVALID DATE".
   - Confirm `job_queue` is 'Y', 'N', or blank. If not, return "INVALID JOB QUEUE CODE".
   - Set `copy_count` to 01 if 00.
   - Validate `password` as 'CUSTPROD' or blank. If invalid, return "INVALID PASSWORD".
   - If any validation fails, return the error message and halt.

2. **Generate Temporary Customer List**:
   - Read `PRCTUM` to create `BB954S`, including only unique, non-deleted records (`PCDEL ≠ 'D'`) with `company_number` and `customer_number`.
   - Mark records in `BB954S` as active (`PSDEL = 'A'`).

3. **Process Pricing Updates**:
   - For each record in `pricing_data`:
     - Validate `company` against `BICONT`. If invalid, set company name to blank and use default date (20791231).
     - Check for prior price in `BBPRCEP` using key (`company`, `location`, `product_code`, `container`, `unit_of_measure`, `date`, `time`).
     - If no record exists in `PRCTUM` for the key (`company`, `customer_number`, `product_code`, `container`, `unit_of_measure`), add a new record with `PCDEL = 'A'`.
     - If `password = 'CUSTPROD'`, add a new record to `BBPRCE` with:
       - `RKDEL = 'A'`
       - Fields from `pricing_data` (`company`, `location`, `product_code`, `container`, `unit_of_measure`, `date`, `time`, `price`)
       - Zero values for additional prices (`RKPR02`–`RKPR05`) and quantities (`RKMINQ`, `RKQT01`–`RKQT05`)
       - Hard-coded quantity '9999999' for one field

4. **Generate Report**:
   - Produce a report for `copy_count` copies, including:
     - Header: Company name, "DA LUBRICANTS RACK PRICING EDIT/POST", system date, time, page number.
     - Columns: Company, Location, Product, Container, Unit of Measure, Date, Time, Quantity Range, New Price.
     - Detail lines for each `pricing_data` record.
     - Footer: Total record count and total new price (`DAPRTL`).

5. **Cleanup**:
   - Delete `BB954S` file.
   - Reset all variables to blanks or zeros.

**Business Rules**:
- **Validation**:
  - Company number must exist in `BICONT`.
  - Customer number must exist in `PRCTUM` or `BB954S`.
  - Month must be 01–12.
  - Job queue must be 'Y', 'N', or blank.
  - Copy count must be > 0 (default to 1 if 0).
  - Password must be 'CUSTPROD' or blank for `BBPRCE` updates.
- **Uniqueness**:
  - `BB954S` contains unique customer records (company + customer number).
  - `BBPRCEP` is checked for prior prices to avoid duplicates.
- **Data Integrity**:
  - New records in `PRCTUM` and `BBPRCE` are marked active ('A').
  - `BBPRCE` updates require valid password.
- **Calculations**:
  - Total record count (`COUNT`) increments by 1 per processed record.
  - Total new price (`DAPRTL`) sums `price` from `pricing_data`.
  - Prior price total (`PRVPTL`) sums `RKPRCE` from `BBPRCEP` if found.
- **Reporting**:
  - Report includes all pricing records and totals, formatted with headers and separators.

**Error Handling**:
- Return specific error messages for validation failures.
- Halt processing if cancellation is detected (`KYCANC = 'CANCEL'`).

**Tables Used**:
- `PRCTUM`: Customer master (input/update).
- `BB954S`: Temporary customer list (input/output).
- `BICONT`: Company control (input).
- `ARCUST`: Customer master (input).
- `DAPRCUP`: New pricing data (input).
- `BBPRCEP`: Existing pricing data (input).
- `BBPRCE`: Updated pricing data (output).
- `PRTDOWN`: Report output (printer).

**External Programs**: None called directly; function encapsulates logic from `BB9541`, `BB954P`, and `BB9542`.

<xaiArtifact artifact_id="a500f27c-b714-4ebd-a152-16e330b1c573" artifact_version_id="b17f9d6d-e077-43ff-986d-9e1972b7f2b1" title="UpdateRackPricing_Requirements.md" contentType="text/markdown">

# Function Requirement Document: UpdateRackPricing

## Description
Updates rack pricing for DA Lubricants by processing input pricing data, validating it, updating customer and pricing files, and generating a report, without screen interaction.

## Inputs
- `company_number`: 2 characters, company identifier.
- `customer_number`: 6 digits, customer identifier.
- `month`: 2 digits, 01–12.
- `job_queue`: 1 character, 'Y', 'N', or blank.
- `copy_count`: 2 digits, number of report copies (default 01 if 00).
- `password`: 10 characters, 'CUSTPROD' or blank.
- `pricing_data`: List of records with:
  - `company`: 2 characters
  - `location`: 3 characters
  - `product_code`: 4 characters
  - `container`: 3 characters
  - `unit_of_measure`: 3 characters
  - `date`: 8 digits (CYMD)
  - `time`: 4 digits (HHMM)
  - `price`: 9.4 numeric
  - `quantity_description`: 10 characters

## Outputs
- Updated `PRCTUM` and `BBPRCE` files.
- Printed report with pricing details and totals.
- Status: Success, Failure (with error), or Cancelled.

## Process Steps
1. **Validate Inputs**:
   - Company number in `BICONT`, else return "INVALID COMPANY NUMBER".
   - Customer number in `PRCTUM`/`BB954S`, else return "INVALID CUSTOMER NUMBER".
   - Month 01–12, else return "INVALID DATE".
   - Job queue 'Y', 'N', or blank, else return "INVALID JOB QUEUE CODE".
   - Copy count > 0 (set to 1 if 0).
   - Password 'CUSTPROD' or blank, else return "INVALID PASSWORD".
   - Halt on validation failure.
2. **Generate Customer List**:
   - Read `PRCTUM`, write unique, non-deleted (`PCDEL ≠ 'D'`) records to `BB954S` with `PSDEL = 'A'`.
3. **Process Pricing Updates**:
   - For each `pricing_data` record:
     - Validate company in `BICONT`; if invalid, use blank name and date 20791231.
     - Check prior price in `BBPRCEP` using key (`company`, `location`, `product_code`, `container`, `unit_of_measure`, `date`, `time`).
     - Add new `PRCTUM` record if key (`company`, `customer_number`, `product_code`, `container`, `unit_of_measure`) missing, with `PCDEL = 'A'`.
     - If `password = 'CUSTPROD'`, add `BBPRCE` record with `RKDEL = 'A'`, input fields, zeroed prices/quantities, and '9999999' for one quantity.
4. **Generate Report**:
   - Print `copy_count` reports with:
     - Header: Company name, title, date, time, page.
     - Columns: Company, Location, Product, Container, Unit of Measure, Date, Time, Quantity Range, New Price.
     - Footer: Total record count, total new price.
5. **Cleanup**:
   - Delete `BB954S`.
   - Reset variables.

## Business Rules
- **Validation**: Company in `BICONT`, customer in `PRCTUM`/`BB954S`, month 01–12, job queue 'Y'/'N'/blank, copy count > 0, password 'CUSTPROD'/blank.
- **Uniqueness**: `BB954S` records unique; `BBPRCEP` checked for prior prices.
- **Data Integrity**: New `PRCTUM`/`BBPRCE` records marked 'A'; `BBPRCE` updates require valid password.
- **Calculations**:
  - `COUNT`: Increment per record.
  - `DAPRTL`: Sum of `price` from `pricing_data`.
  - `PRVPTL`: Sum of `RKPRCE` from `BBPRCEP`.
- **Reporting**: Detailed report with headers, details, and totals.

## Error Handling
- Return specific error messages for validation failures.
- Halt if `KYCANC = 'CANCEL'`.

## Tables Used
- `PRCTUM`, `BB954S`, `BICONT`, `ARCUST`, `DAPRCUP`, `BBPRCEP`, `BBPRCE`, `PRTDOWN`.

## External Programs
None; encapsulates `BB9541`, `BB954P`, `BB9542` logic.

</xaiArtifact>