The call stack provided consists of the following programs for generating a **Pending Orders Report** on the IBM System/36:

1. **BB705P.ocl36.txt**: Orchestrates the initial job, calling `BB705P.rpg36.txt` for input validation.
2. **BB705P.rpg36.txt**: Validates user input parameters (company, location, date range, etc.) via a workstation screen.
3. **BB705.ocl36.txt**: Manages data preparation (`BB7051`), sorting (`#GSORT`), and report generation (`BB705`).
4. **BB7051.rpg36.txt**: Builds a work file (`BB705O`) by filtering and enhancing order data from `BBORDR`.
5. **BB705.rpg36.txt**: Generates the final formatted report using sorted data and additional files.

Below, I’ll identify the **use cases** implemented by this call stack and then provide a **Function Requirement Document** for one use case, adapted to process inputs programmatically rather than via a screen.

---

### Use Cases Implemented by the Call Stack

The call stack implements the following use cases for generating a Pending Orders Report, each representing a distinct business need or reporting variation:

1. **Generate Pending Orders Report by Company, Location, Date Range, and Carrier**:
   - **Description**: Produces a report listing pending orders filtered by company, location, date range, and carrier, sorted hierarchically (company, location, request date, carrier, order, sequence).
   - **Details**: Users specify company (`KYCO`), location (`KYLOC`), from/to dates (`KYFRDT`, `KYTODT`), zero date flag (`KYZEYN`), job queue flag (`KYJOBQ`), and number of copies (`KYCOPY`). The report includes order details (e.g., customer, ship-to, quantities, prices) and remarks (order, invoice, dispatch, BOL, freight).
   - **Components**:
     - `BB705P.rpg36.txt`: Validates input parameters.
     - `BB7051.rpg36.txt`: Filters `BBORDR` and builds `BB705O` with sort fields.
     - `#GSORT` (via `BB705.ocl36.txt`): Sorts data by specified fields.
     - `BB705.rpg36.txt`: Generates the formatted report with data from `BB705S`, `ARCUST`, `SHIPTO`, etc.

2. **Generate Pending Orders Report with Zero Date Option**:
   - **Description**: Produces a report listing pending orders with a zero request date (special case for orders without a scheduled date), filtered by company and location.
   - **Details**: Enabled when `KYZEYN = 'Y'`, ensuring the report includes only orders with a request date of zero, overriding date range inputs. This is useful for identifying unscheduled orders.
   - **Components**:
     - `BB705P.rpg36.txt`: Validates `KYZEYN = 'Y'` and ensures zero dates (`KYFRDT`, `KYTODT = 0`).
     - `BB705.ocl36.txt`: Sets sort criteria (`I*CIAC` vs. `IACI*C`) based on `KYZEYN`.
     - `BB7051.rpg36.txt` and `#GSORT`: Filter and sort records with zero request dates.
     - `BB705.rpg36.txt`: Generates the report.

3. **Validate Input Parameters for Pending Orders Report**:
   - **Description**: Ensures user inputs (company, location, date range, zero date flag, job queue flag, copies) are valid before processing the report.
   - **Details**: Checks company and location against `BICONT` and `INLOC`, validates date ranges, ensures `KYZEYN` and `KYJOBQ` are 'Y', 'N', or blank, and defaults copies to 1 if zero. Errors are displayed on the screen.
   - **Components**:
     - `BB705P.rpg36.txt`: Handles all validation logic and screen interaction.

4. **Run Pending Orders Report in Batch Mode**:
   - **Description**: Submits the report generation job to a job queue for background processing, allowing asynchronous execution.
   - **Details**: Enabled when `KYJOBQ = 'Y'`, submitting `BB705` to the job queue specified by `?CLIB?` in `BB705P.ocl36.txt`.
   - **Components**:
     - `BB705P.ocl36.txt`: Conditionally submits the job (`JOBQ ?CLIB?,BB705`).
     - `BB705.ocl36.txt`: Executes the full workflow (`BB7051`, `#GSORT`, `BB705`).

---

### Function Requirement Document

**Function Name**: GeneratePendingOrdersReport

**Purpose**: Generate a Pending Orders Report based on provided company, location, date range, and zero date parameters, producing a formatted report of pending orders sorted by company, location, request date, carrier, order, and sequence. This function assumes programmatic input rather than screen interaction, processing both standard and zero-date reports.

**Inputs**:
- `company` (2 characters): Company number (e.g., '10').
- `location` (3 characters): Location code (e.g., '001').
- `fromDate` (8 digits, CCYYMMDD): Start of date range (e.g., '20250101').
- `toDate` (8 digits, CCYYMMDD): End of date range (e.g., '20251231').
- `zeroDateFlag` (1 character): 'Y' for zero date report, 'N' or blank for date range report.
- `copies` (2 digits): Number of report copies (default to 1 if 0).

**Outputs**:
- A formatted report written to a file or printer, containing:
  - Headers: Company, location, date range, request date, order details.
  - Details: Product, quantity, price, unit of measure, description, totals.
  - Remarks: Order, invoice, dispatch, bill of lading, freight information.
  - Totals: Order-level totals.

<xaiArtifact artifact_id="acba42cb-c2d0-47d0-af62-4d226f416d00" artifact_version_id="2ac6d740-a6eb-4434-8058-22a4cae43aca" title="PendingOrdersReportRequirements.md" contentType="text/markdown">

# Function Requirement Document: GeneratePendingOrdersReport

## Purpose
Generate a Pending Orders Report listing pending orders filtered by company, location, and either a date range or zero request date, sorted by company, location, request date, carrier, order, and sequence. The function processes inputs programmatically and produces a formatted report with order details, remarks, and totals.

## Inputs
- **company** (2 chars): Company number (e.g., '10').
- **location** (3 chars): Location code (e.g., '001').
- **fromDate** (8 digits, CCYYMMDD): Start date of range (e.g., '20250101').
- **toDate** (8 digits, CCYYMMDD): End date of range (e.g., '20251231').
- **zeroDateFlag** (1 char): 'Y' for zero date report, 'N' or blank for date range.
- **copies** (2 digits): Number of report copies (default 1 if 0).

## Outputs
- Formatted report (file/printer) with:
  - **Headers**: Company name, location, date range, request date, order details (order number, type, customer, ship-to, PO, carrier, etc.).
  - **Details**: Location, product, quantity, price, unit of measure, description, no-charge code, totals.
  - **Remarks**: Order, invoice, dispatch, bill of lading (BOL), freight details (if non-blank).
  - **Totals**: Order-level totals for quantities and amounts.

## Process Steps
1. **Validate Inputs**:
   - Verify `company` exists in `BICONT` and is not deleted.
   - Verify `company` + `location` exists in `INLOC`.
   - If `zeroDateFlag = 'Y'`, ensure `fromDate` and `toDate` are 0; else, ensure `fromDate ≤ toDate`.
   - Validate `zeroDateFlag` is 'Y', 'N', or blank.
   - If `copies = 0`, set to 1.

2. **Prepare Work File**:
   - Read `BBORDR` (order data) and filter records:
     - Exclude records with delete flag (`BODEL`, `BDDEL`, `BMDEL = 'D'`).
     - Match `company` (positions 2-3).
     - If `zeroDateFlag = 'Y'`, select records with request date = 0; else, select records where request date is within `fromDate` to `toDate`.
   - For each order:
     - Save location from first detail record, request date, and carrier code from header.
     - Write header, detail, and remark records to `BB705O` with appended fields (location: 513-515, request date: 516-523, carrier: 524-525).

3. **Sort Work File**:
   - Sort `BB705O` into `BB705S` by:
     - Company (2-3).
     - Location (513-515).
     - Request date (516-523).
     - Carrier code (524-525).
     - Order number (4-9).
     - Sequence (10-12).

4. **Generate Report**:
   - Read `BB705S` and retrieve additional data from `ARCUST` (customer), `SHIPTO` (ship-to), `BICONT` (company), `INLOC` (location), `GSCTUM` (unit of measure).
   - Call `MSHPADR` to get compressed ship-to address (name, address lines, country).
   - Format report with:
     - Headers: Company, location, date range, request date, order details (customer, ship-to, PO, carrier, terms, etc.).
     - Details: Product, quantity, price, unit of measure, description, totals.
     - Remarks: Order, invoice, dispatch, BOL, freight (if non-blank).
     - Totals: Order-level totals (`L1TOT`).
   - Write `copies` instances to output file/printer.

5. **Cleanup**:
   - Delete temporary files `BB705O` and `BB705S`.

## Business Rules
- **Validation**:
  - Company must exist in `BICONT` and not be deleted.
  - Location must exist in `INLOC` for the company.
  - If `zeroDateFlag = 'Y'`, dates must be 0; else, `fromDate ≤ toDate`.
  - `zeroDateFlag` must be 'Y', 'N', or blank.
  - Default `copies` to 1 if 0.
- **Filtering**:
  - Exclude deleted records (`BODEL`, `BDDEL`, `BMDEL ≠ 'D'`).
  - Match company and location.
  - If `zeroDateFlag = 'Y'`, select zero request date records; else, select records within date range.
- **Order Types**:
  - Include orders with type 'R' (regular) or 'M' (product move). Exclude legacy 'M' (cash return) and 'C' (cash sale).
- **Sorting**:
  - Sort by company, location, request date, carrier, order, sequence.
- **Report Content**:
  - Include customer/ship-to data from `ARCUST`/`SHIPTO` (via `MSHPADR`).
  - Print non-blank remarks (order, invoice, dispatch, BOL, freight).
  - Include multi-load, loads/day, load volume, and weekend pickup details.
- **Calculations**:
  - Detail total (`BDTOT`): `BDQTY * BDPRCE`.
  - Miscellaneous total (`BMTOT`): `BMQTY * BMAMT`.
  - Order total (`L1TOT`): Sum of `BDTOT` and `BMTOT` for the order.

## Dependencies
- **Files**:
  - `BBORDR`: Order data (header, detail, remarks).
  - `BB705O`, `BB705S`: Temporary work files.
  - `ARCUST`: Customer data.
  - `SHIPTO`: Ship-to data.
  - `BICONT`: Company data.
  - `INLOC`: Location data.
  - `GSCTUM`: Unit of measure data.
- **External Program**:
  - `MSHPADR`: Retrieves compressed ship-to address (name, address lines, country).

</xaiArtifact>

---

### Notes
- The **Function Requirement Document** focuses on the primary use case (Generate Pending Orders Report) but incorporates the zero-date option as a conditional behavior, as it’s a variation within the same workflow.
- The document assumes programmatic input, replacing the screen interaction in `BB705P.rpg36.txt` with direct parameter passing.
- The batch mode use case (`KYJOBQ = 'Y'`) is not explicitly included in the function, as it’s an execution mode rather than a functional difference, but it could be extended with a parameter to trigger job queue submission.
- Calculations (e.g., `BDTOT`, `BMTOT`, `L1TOT`) are inferred from `BB705.rpg36.txt`’s output structure, as `BB7051` does not perform calculations but passes data for later processing.

If you need a separate function document for another use case (e.g., zero-date report or batch mode), or if you want further details on file layouts or calculations, let me know!