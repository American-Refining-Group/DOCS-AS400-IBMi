The provided call stack consists of several programs (`BB750P.ocl36.txt`, `BB750P.rpg36.txt`, `BB750.ocl36.txt`, `BB750.rpg36.txt`, `BB751.ocl36.txt`, `BB751.rpg36.txt`) that collectively implement a system for generating **Open Railcar Bill of Lading (BOL) Listings** in either **detailed** or **summary** formats. Below, I’ll identify the use cases implemented by this call stack and then provide a **Function Requirement Document** for a large function that encapsulates these use cases, assuming inputs are provided programmatically rather than via a screen.

---

### **List of Use Cases**

The call stack implements the following use cases for generating Open Railcar BOL Listings:

1. **Generate Detailed BOL Listing by Order**:
   - **Description**: Produces a detailed report listing all open railcar BOLs by order, including header information (e.g., company, order number, customer, ship-to, railcar, dates), detail lines (e.g., items, quantities, prices), and remarks (e.g., order, invoice, dispatch, freight).
   - **Components Involved**:
     - `BB750P.ocl36.txt`: Determines if a detailed report is requested (`KYDETL = 'Y'`) and calls `BB750`.
     - `BB750P.rpg36.txt`: Validates input parameters (company number, detail flag, job queue flag, copies).
     - `BB750.ocl36.txt`: Sorts BOL data (`BBBOL`) into `BB750S` and calls `BB750` with additional files (`ARCUST`, `SHIPTO`, `GSCONT`, `BICONT`, `GSCTUM`).
     - `BB750.rpg36.txt`: Generates the detailed report, including header, detail, and remark records, using data from multiple files and calling `MSHPADR` for ship-to address formatting.
   - **Key Features**:
     - Filters out deleted records and validates company/customer data.
     - Sorts by company, order, and sequence.
     - Includes detailed item information and remarks.
     - Outputs to a 164-byte printer file (`LIST`).

2. **Generate Summary BOL Listing by Order**:
   - **Description**: Produces a summarized report listing open railcar BOLs by order, including only header information (e.g., company, order number, customer, railcar, dates, PO number) without detail lines or remarks.
   - **Components Involved**:
     - `BB750P.ocl36.txt`: Determines if a summary report is requested (`KYDETL ≠ 'Y'`) and calls `BB751`.
     - `BB750P.rpg36.txt`: Validates input parameters (company, detail flag, job queue flag, copies).
     - `BB751.ocl36.txt`: Sorts BOL data (`BBBOL`) into `BB751S`, filtering for sequence `'000'`, and calls `BB751` with files (`ARCUST`, `BICONT`, `GSCONT`).
     - `BB751.rpg36.txt`: Generates the summary report with header-level data only.
   - **Key Features**:
     - Filters for header records (sequence `'000'`) and excludes deleted records.
     - Sorts by company and order.
     - Outputs to a 132-byte printer file (`LIST`).
     - More concise than the detailed report, omitting item details and remarks.

3. **Validate Input Parameters for BOL Listing**:
   - **Description**: Validates user-provided parameters (company number, detail flag, job queue flag, number of copies) before generating either report.
   - **Components Involved**:
     - `BB750P.ocl36.txt`: Checks conditions and routes to `BB750` or `BB751`.
     - `BB750P.rpg36.txt`: Performs validation of company number (against `BICONT`), detail flag (`'Y'`, `'N'`, or blank), job queue flag (`'Y'`, `'N'`, or blank), and sets default copies to 1 if zero.
   - **Key Features**:
     - Ensures valid company number exists in `BICONT` and is not deleted (`BCDEL ≠ 'D'`).
     - Validates flags and provides error messages for invalid inputs.
     - Sets default values if initial screen is displayed.

4. **Sort BOL Data for Reporting**:
   - **Description**: Sorts BOL data by company, order, and sequence, filtering out deleted records and applying company selection criteria, to prepare data for either report.
   - **Components Involved**:
     - `BB750.ocl36.txt`: Sorts `BBBOL` into `BB750S` for the detailed report.
     - `BB751.ocl36.txt`: Sorts `BBBOL` into `BB751S` for the summary report, restricting to sequence `'000'`.
   - **Key Features**:
     - Excludes deleted records (`BODEL ≠ 'D'`).
     - Filters by company code based on input parameters (`'O COAC'` or `'O*CO*C'`).
     - Sorts by multiple keys (order number, sequence fields).

---

### **Function Requirement Document**

<xaiArtifact artifact_id="369816c5-cdaa-45a2-b1f6-b3fb220c9f03" artifact_version_id="397550b3-c2b6-4779-a41f-76f042c94bf0" title="BOL_Listing_Function_Requirements.md" contentType="text/markdown">

# Function Requirement Document: Generate Open Railcar BOL Listing

## **Overview**
The `GenerateBOLListing` function generates an Open Railcar Bill of Lading (BOL) Listing in either detailed or summary format, based on input parameters. It validates inputs, sorts BOL data, and produces a formatted report using data from multiple files, without screen interaction.

## **Inputs**
- **CompanyCode** (2 bytes, string): Company number for filtering BOLs.
- **DetailFlag** (1 byte, string): `'Y'` for detailed report, `'N'` or blank for summary report.
- **JobQueueFlag** (1 byte, string): `'Y'` to submit to job queue, `'N'` or blank for direct execution.
- **NumberOfCopies** (2 bytes, numeric): Number of report copies (default 1 if 0).
- **SelectionCriteria** (3 bytes, string): `'SEL'` for specific company (`'O COAC'`) or other for all companies (`'O*CO*C'`).

## **Outputs**
- **Report File**: A formatted text file (164 bytes for detailed, 132 bytes for summary) containing the BOL listing.
- **Error Message** (40 bytes, string): Returned if validation fails (e.g., invalid company, flag values).

## **Process Steps**
1. **Validate Inputs**:
   - Check `CompanyCode` against `BICONT` file. Return error ("INVALID COMPANY NUMBER ENTERED") if not found or marked deleted (`BCDEL = 'D'`).
   - Validate `DetailFlag` as `'Y'`, `'N'`, or blank. Return error ("INVALID PARAMETER ENTERED - ' ', N OR Y") if invalid.
   - Validate `JobQueueFlag` as `'Y'`, `'N'`, or blank. Return error ("INVALID PARAMETER ENTERED - ' ', N OR Y") if invalid.
   - Set `NumberOfCopies` to 1 if 0.

2. **Sort BOL Data**:
   - Read input BOL file (`BBBOL`, 512 bytes).
   - Filter out records where `BODEL = 'D'` (deleted).
   - Filter by `CompanyCode` (positions 2–3) matching input or wildcard (`'O*CO*C'` if `SelectionCriteria ≠ 'SEL'`).
   - For summary report (`DetailFlag ≠ 'Y'`), include only records with sequence number (`BORSEQ`, positions 10–12) equal to `'000'`.
   - Sort by:
     - Company number (positions 2–3, ascending).
     - Order number (positions 4–9, ascending).
     - Sequence fields (positions 106, 112, 118, etc., ascending).
   - Output sorted data to temporary file (`BB750S` for detailed, `BB751S` for summary).

3. **Generate Report**:
   - **For Detailed Report (`DetailFlag = 'Y'`)**:
     - Read sorted file (`BB750S`, renamed as `BBBOL`).
     - For each header record (`BORSEQ = '000'`):
       - Retrieve company name (`BCNAME`) from `BICONT` using `BOCO`.
       - Retrieve customer name (`ARNAME`) and address (`ARADR1`–`ARADR4`) from `ARCUST` using `BOCO` + `BOCUST`.
       - Retrieve ship-to name (`SNAM`) and address (`SAD1`–`SAD5`) from `SHIPTO` using `BOCO` + `BOCUST` + `BOSHIP` (or use customer address if `BOSHIP = 0`, special key for `900/999`).
       - Call `GetShipToAddress` function (equivalent to `MSHPADR`) with parameters: key (`BOCO`, `BOCUST`, `BOSHIP`), returning `SNAM`, `SAD1`–`SAD5`, `SCTY` (country).
       - Output header: company, order number, customer, ship-to, railcar (`BOCAR#`, `KRCCAR`), dates (`BORQDT`, `BOORDT`, `BOPODT`), PO numbers (`BOPORD`, `BOSHPO`), terms (`BOTERM`), salesman (`BOSLMN`), carrier (`BOCACD`), freight code (`BOFRCD`), delivery (`BODELV`), and user fields (`BOVAR1`, `BOVAR2`).
     - For each detail record (`NS 02`):
       - Output item details: location (`BDLOC`), product (`BDPROD`), quantity (`BDQTY`), price (`BDPRCE`), unit of measure (`BDUM`), no-charge code (`BDNOCH`), description (`BDDESC`), total (`BDTOT` = `BDQTY * BDPRCE`).
     - For remark records (`NS 04`, `NS 05`):
       - Output order remarks (`BXOMK1`–`BXOMK4`), invoice remarks (`BXIMK1`, `BXIMK2`), dispatch remarks (`BXDSP1`, `BXDSP2`), freight remarks (`BXFRNM`, `BXFRA1`–`BXFRA3`).
     - Output order total (`L1TOT`, sum of `BDTOT` for the order).
     - Format output to 164-byte `LIST` file with headers, separators (asterisks/hyphens), and page breaks.
   - **For Summary Report (`DetailFlag ≠ 'Y'`)**:
     - Read sorted file (`BB751S`, renamed as `BBBOL`).
     - For each header record (`BORSEQ = '000'`):
       - Retrieve company name (`BCNAME`) from `BICONT` using `BOCO`.
       - Retrieve customer name (`ARNAME`) from `ARCUST` using `BOCO` + `BOCUST`.
       - Output: order number (`BORDNO`), customer number (`BOCUST`), customer name (`ARNAME`), railcar (`KRCCAR`), order date (`BOORDT`), PO number (`BOPORD`), request date (`BORQDT`).
     - Format output to 132-byte `LIST` file with company headers, column labels, and asterisk separators.

4. **Handle Job Queue**:
   - If `JobQueueFlag = 'Y'`, submit the sort and report generation to a job queue in the specified library.
   - Otherwise, execute directly.

5. **Output Report**:
   - Write the report to a file (`LIST`) with `NumberOfCopies` copies.
   - Include system date (YY/MM/DD) and time (HH:MM:SS) in the report header.

## **Business Rules**
1. **Input Validation**:
   - `CompanyCode` must exist in `BICONT` and not be deleted (`BCDEL ≠ 'D'`).
   - `DetailFlag` must be `'Y'`, `'N'`, or blank.
   - `JobQueueFlag` must be `'Y'`, `'N'`, or blank.
   - `NumberOfCopies` defaults to 1 if 0.

2. **Data Filtering**:
   - Exclude records with `BODEL = 'D'` or `BDDEL = 'D'` (deleted).
   - Filter by `CompanyCode` (exact match or wildcard based on `SelectionCriteria`).
   - For summary report, include only header records (`BORSEQ = '000'`).

3. **Sorting**:
   - Sort by company (`BOCO`), order number (`BORDNO`), and sequence fields (positions 106, 112, etc.).
   - Ensure ascending order for consistent report grouping.

4. **Data Retrieval**:
   - Use `BICONT` for company name (`BCNAME`).
   - Use `ARCUST` for customer name (`ARNAME`) and address (`ARADR1`–`ARADR4`).
   - Use `SHIPTO` for ship-to name (`SNAM`) and address (`SAD1`–`SAD5`) in detailed report.
   - Use `GSCONT` and `GSCTUM` for control and unit of measure data in detailed report.

5. **Calculations**:
   - **Detail Line Total (`BDTOT`)**: `BDQTY * BDPRCE` (quantity * price, formatted with 2–4 decimal places based on `BDUM`).
   - **Order Total (`L1TOT`)**: Sum of `BDTOT` for all detail lines in an order (detailed report only).
   - **Prepaid Amount (`BOPAMT`)**: Formatted with 2 decimal places (packed field).

6. **Report Formatting**:
   - Detailed report (164 bytes): Includes header, detail lines, remarks, totals, with asterisk/hyphen separators.
   - Summary report (132 bytes): Includes header data only, with asterisk separators.
   - Include system date/time and page numbers in headers.

## **Dependencies**
- **Files**:
  - `BBBOL`: BOL data (512 bytes, header/detail/remarks).
  - `BB750S`/`BB751S`: Sorted temporary files.
  - `ARCUST`: Customer data (384 bytes).
  - `SHIPTO`: Ship-to data (2048 bytes, detailed report only).
  - `BICONT`: Company/inventory data (256 bytes).
  - `GSCONT`: Control data (512 bytes, detailed report only).
  - `GSCTUM`: Unit of measure data (64 bytes, detailed report only).
  - `LIST`: Output report file (164 bytes for detailed, 132 bytes for summary).
- **External Function**:
  - `GetShipToAddress` (equivalent to `MSHPADR`): Takes company, customer, ship-to number; returns formatted ship-to name, address, and country (detailed report only).

## **Error Handling**
- Return error messages for invalid inputs (company, flags).
- Skip invalid company/customer records, omitting names in the report.
- Log sort or file access failures if records are not found.

## **Performance Considerations**
- Use job queue (`JobQueueFlag = 'Y'`) for large datasets to avoid blocking execution.
- Optimize sort operation by leveraging indexed files (`ARCUST`, `BICONT`, etc.).
- Ensure temporary files (`BB750S`, `BB751S`) are retained only for the job duration.

</xaiArtifact>

---

### **Notes**
- The function assumes programmatic input, replacing the interactive screen (`SCREEN`, `BB750PFM`) in `BB750P.rpg36.txt`.
- The `GetShipToAddress` function encapsulates the logic of `MSHPADR` for ship-to address formatting.
- Calculations (e.g., `BDTOT`, `L1TOT`) are based on fields in `BB750.rpg36.txt`, with formatting inferred from output specifications.
- The report output format mirrors the original RPG programs’ printer files (`LIST`), maintaining column alignments and separators.

If you need specific file layouts or additional details on `MSHPADR`, I can refine the requirements further!