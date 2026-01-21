The call stack provided consists of the OCL program `BB953.ocl36.txt` and the RPG programs `BB953P.rpg36.txt`, `BB9531.rpg36.txt`, `BB9534.rpg.txt`, and `BB953.rpg36.txt`. Together, these programs implement a workflow for generating a customer rack pricing list report on an IBM System/36 or AS/400 environment. Below, I will first identify the use cases implemented by this call stack and then provide a function requirement document for the primary use case, reimagined as a large function that processes inputs programmatically rather than via screen interaction.

---

### **Use Cases Implemented by the Call Stack**

The call stack supports the generation of a rack price list report with filtering and formatting capabilities. Based on the analysis of the programs, the following use cases are implemented:

1. **Generate a Comprehensive Rack Price List**:
   - **Description**: Produces a report listing the latest price changes for all products, containers, and units of measure across all locations, divisions, and product classes, without specific filtering.
   - **Details**:
     - Users can select "ALL" for locations, product classes, containers, products, and dates to include all relevant records.
     - The report includes company, location, division, product class, product, container, unit of measure, quantities, prices, effective date, time, minimum quantity, and rack/inactive flags.
     - Output is printed (`LIST`) and saved to a disk file (`OUTFILE`).

2. **Generate a Filtered Rack Price List**:
   - **Description**: Produces a report filtered by specific locations, product classes, containers, products, and/or date ranges, based on user input.
   - **Details**:
     - Users can specify individual locations (`KYLOC1-5`), product classes (`KYPC01-10`), containers (`KYCT01-5`), products (`KYPD01-10`), and a date range (`KYFRDT`, `KYTODT`).
     - Validations ensure selected values exist in reference files (`INLOC`, `GSPROD`, `GSCNTR1`).
     - The report is formatted similarly to the comprehensive list but includes only records matching the filters.

3. **Validate User Input for Rack Price List Parameters**:
   - **Description**: Validates user-specified parameters (division, locations, product classes, containers, products, date range, job queue, and copy count) before generating the report.
   - **Details**:
     - Ensures division is valid (`'R'`, `'L'`, or `'B'`), locations exist in `INLOC`, product classes and products exist in `GSPROD`, containers exist in `GSCNTR1`, and dates are non-zero when required.
     - Provides error messages for invalid inputs (e.g., "INVALID LOCATION ENTERED").
     - Sets defaults (e.g., `KYCOPY = 1`, `KYJOBQ = 'N'`) when no input is provided.

4. **Generate a Background Job for Rack Price List**:
   - **Description**: Allows the report generation to be queued as a background job rather than executed interactively.
   - **Details**:
     - Users can specify `KYJOBQ = 'Y'` to queue the job or `'N'` for immediate execution.
     - The OCL program checks for concurrent execution (`IF ACTIVE-BB953`) to prevent conflicts.

---

### **Function Requirement Document**

The primary use case, **Generate a Filtered Rack Price List**, is reimagined as a programmatic function that accepts inputs directly rather than through screen interaction. This function encapsulates the entire workflow, including validation, preprocessing, filtering, sorting, and report generation. Below is a concise function requirement document outlining the process steps, business rules, and calculations.

<xaiArtifact artifact_id="6e5c1c16-6187-497a-a040-b86ccccbf95f" artifact_version_id="b84b2a17-2153-4e82-ba81-adbfed1b101f" title="RackPriceListFunctionRequirements.md" contentType="text/markdown">

# Rack Price List Function Requirements

## **Function Name**: `generateRackPriceList`

## **Purpose**
Generate a rack price list report for customer quoting, filtered by user-specified parameters (company, division, locations, product classes, containers, products, date range), with output to a printer and disk file.

## **Inputs**
- **Company Code** (`KYCO`, 2 bytes): Company identifier (e.g., `10`).
- **Division** (`KYDIV`, 1 byte): `'R'` (Refinery), `'L'` (Blended Lubes), or `'B'` (Both).
- **Location Selection** (`KYLOSL`, 3 bytes): `'ALL'` or `'SEL'`.
- **Locations** (`KYLOC1-5`, array of 5 x 3 bytes): Specific location codes if `KYLOSL = 'SEL'`.
- **Product Class Selection** (`KYPCSL`, 3 bytes): `'ALL'` or `'SEL'`.
- **Product Classes** (`KYPC01-10`, array of 10 x 3 bytes): Specific product class codes if `KYPCSL = 'SEL'`.
- **Container Selection** (`KYCTSL`, 3 bytes): `'ALL'` or `'SEL'`.
- **Containers** (`KYCT01-5`, array of 5 x 3 bytes): Specific container codes if `KYCTSL = 'SEL'`.
- **Product Selection** (`KYPDSL`, 3 bytes): `'ALL'` or `'SEL'`.
- **Products** (`KYPD01-10`, array of 10 x 4 bytes): Specific product codes if `KYPDSL = 'SEL'`.
- **Date Selection** (`KYDTSL`, 3 bytes): `'ALL'` or `'SEL'`.
- **From Date** (`KYFRDT`, 6 bytes, YYMMDD): Start date if `KYDTSL = 'SEL'`.
- **To Date** (`KYTODT`, 6 bytes, YYMMDD): End date if `KYDTSL = 'SEL'`.
- **Job Queue** (`KYJOBQ`, 1 byte): `'Y'` (queue job), `'N'` (run immediately), or blank.
- **Copy Count** (`KYCOPY`, 2 bytes, numeric): Number of report copies (default: 1).

## **Outputs**
- **Printer Output** (`LIST`, 164 bytes): Formatted rack price list report with headers and detail lines.
- **Disk Output** (`OUTFILE`, 121 bytes): Structured report data for storage.
- **Return Value**: Status code (`SUCCESS`, `INVALID_INPUT`, `CONCURRENT_JOB`, `FILE_ERROR`).

## **Process Steps**
1. **Validate Inputs**:
   - Check if another instance is running (`BB953`). If true, return `CONCURRENT_JOB`.
   - Validate `KYDIV` is `'R'`, `'L'`, or `'B'`. If invalid, return `INVALID_INPUT` with error "DIVISION MUST BE 'R' 'L' OR 'B'".
   - If `KYLOSL = 'SEL'`, ensure at least one `KYLOC1-5` is non-blank and exists in `INLOC`. If invalid, return `INVALID_INPUT` with error "INVALID LOCATION ENTERED".
   - If `KYPCSL = 'SEL'`, ensure at least one `KYPC01-10` is non-blank and exists in `GSPROD`. If invalid, return `INVALID_INPUT` with error "INVALID PRODUCT CLASS ENTERED".
   - If `KYCTSL = 'SEL'`, ensure at least one `KYCT01-5` is non-blank and exists in `GSCNTR1`. If invalid, return `INVALID_INPUT` with error "INVALID CONTAINER CODE ENTERED".
   - If `KYPDSL = 'SEL'`, ensure at least one `KYPD01-10` is non-blank and exists in `GSPROD`. If invalid, return `INVALID_INPUT` with error "INVALID PRODUCT CODE ENTERED".
   - If `KYDTSL = 'SEL'`, ensure `KYFRDT` and `KYTODT` are non-zero. If invalid, return `INVALID_INPUT` with errors "FROM DATE CANNOT BE BLANK" or "TO DATE CANNOT BE BLANK".
   - If `KYLOSL`, `KYPCSL`, `KYCTSL`, or `KYPDSL` is `'ALL'`, ensure corresponding arrays (`KYLOC1-5`, `KYPC01-10`, `KYCT01-5`, `KYPD01-10`) are blank. If not, return `INVALID_INPUT` with error "CANNOT BE 'ALL' IF SELECTION IS MADE".
   - Validate `KYJOBQ` is blank, `'Y'`, or `'N'`. If invalid, return `INVALID_INPUT` with error "ENTER BLANK, N OR Y".
   - If `KYCOPY = 0`, set to 1.

2. **Preprocess Rack Price Data**:
   - Clear output file `RKPRCE`.
   - Read records from `BBPRCE` (rack price master).
   - Skip records where `RKDEL = 'D'` (deleted) or `RKCONO ≠ KYCO` (company mismatch).
   - If `KYDTSL = 'SEL'`, convert `KYFRDT` and `KYTODT` to 8-digit CCYYMMDD (`FRDAT8`, `TODAT8`):
     - If year > 80, prefix `19`; else, prefix `20`.
     - Filter records where `RKDATE` (CYMD) is within `FRDAT8` and `TODAT8`.
   - If `KYDIV ≠ 'B'`, filter records where division (`TBCSRT` from `GSTABL`) matches `KYDVNO` (`'00'` for `'R'`, `'50'` for `'L'`).
   - Retrieve product details (`TPDES1`, `TPABDS`, `TPPRCL`) from `GSPROD` using `RKCONO + RKPROD`.
   - Retrieve division sort code (`TBCSRT`) from `GSTABL` using `RKCONO + TPPRCL`.
   - Compare prices (`RKPR,1-5`), quantities (`RKQT`), rack required (`RKRKRQ`), and inactive (`RKINAC`) to previous values to detect changes.
   - Write valid records to temporary file `BB9531` (169 bytes) with fields: company, location, product group, product, container, unit of measure, quantities, prices, descriptions, date, time, minimum quantity, division, product class, rack required, inactive flags.

3. **Filter Records**:
   - Read records from `BB9531`.
   - If `KYLOSL = 'SEL'`, include only records where `RKLOC` matches one of `KYLOC1-5`.
   - If `KYPCSL = 'SEL'`, include only records where `RKPRCL` matches one of `KYPC01-10`.
   - If `KYCTSL = 'SEL'`, include only records where `RKCNTR` matches one of `KYCT01-5`.
   - If `KYPDSL = 'SEL'`, include only records where `RKPROD` matches one of `KYPD01-10`.
   - Write filtered records to temporary file `BB9532` (169 bytes, identical structure).

4. **Sort Records**:
   - Sort `BB9532` into `BB953S` using keys:
     - Company (pos 1-2, ascending)
     - Location (pos 3-5, ascending)
     - Division (pos 163-164, ascending)
     - Control field (pos 165-167, ascending)
     - Product (pos 8-11, ascending)
     - Product class (pos 12-14, ascending)
     - Container (pos 15-17, ascending)
   - Exclude records where pos 1 ≠ `'D'` (delete check).

5. **Generate Report**:
   - Read sorted records from `BB953S`.
   - Retrieve company name from `BICONT` using `XXCO`.
   - Retrieve location name from `INLOC` using `XXCO + XXLOC`, skipping deleted records (`ILDEL = 'D'`).
   - Retrieve product class description from `GSTABL` using `'PRODCL' + XXCO + XXPRCL`. If not found, use "NO PRODUCT CLASS LISTED".
   - Retrieve product group description from `GSTABL` using `'PRODGR' + XXCO + XXPRGR`. If not found, use blanks.
   - Retrieve container description from `GSCNTR1` using `XXCNTR`.
   - Format division: if `XXDIV = *ZEROS`, use "REFINERY"; else, use "BLENDED LUBES".
   - Format quantities (`XXQT,1-4`) by removing leading zeros.
   - Write to `OUTFILE` (121 bytes): company, location, product class, product, full description, container, unit of measure, effective date, time, quantities, prices, minimum quantity, rack required, inactive flags.
   - Print to `LIST` (164 bytes):
     - Headers: program name (`BB953`), page number, company name, report title, location, division, date, time, column headings (QUANTITY 1-4, PRICE 1-4, MIN QTY).
     - Detail lines: product class, product, description, container, container description, unit of measure, effective date, time, quantities, prices, minimum quantity, and flags (`RACK NOT REQ'D`, `INACTIVE`, `INACTIV BUT`).
     - Separators: `--` between locations.

6. **Clean Up**:
   - Delete temporary files (`BB953X`, `BB953P`, `BB9531`, `BB953S`, `BB9538`, `BB953U`, `BB953T`, `BB9532`).
   - Return `SUCCESS` or `FILE_ERROR` if file operations fail.

## **Business Rules**
1. **Input Validation**:
   - Division must be `'R'`, `'L'`, or `'B'`.
   - Specific selections (`KYLOSL`, `KYPCSL`, `KYCTSL`, `KYPDSL = 'SEL'`) require valid, non-blank entries in corresponding arrays, verified against `INLOC`, `GSPROD`, `GSCNTR1`.
   - If `'ALL'` is selected, corresponding arrays must be blank.
   - Date range requires non-zero `KYFRDT` and `KYTODT` if `KYDTSL = 'SEL'`.
   - Job queue must be blank, `'Y'`, or `'N'`.
   - Copy count defaults to 1 if zero.

2. **Data Filtering**:
   - Exclude deleted records (`RKDEL = 'D'`).
   - Match company (`RKCONO = KYCO`).
   - Filter by date range if `KYDTSL = 'SEL'`.
   - Filter by division (`KYDVNO`) if `KYDIV ≠ 'B'`.
   - Filter by location, product class, container, or product if `'SEL'` is specified.

3. **Report Formatting**:
   - Display up to 4 quantities and prices per record, with minimum quantity.
   - Include flags: `RKRKRQ = 'N'` ("RACK NOT REQ'D"), `RKINAC = 'I'` ("INACTIVE"), `RKINAC = 'B'` ("INACTIV BUT").
   - Fit last two price changes within two print lines.
   - Organize by company, location, division, product class.

4. **Calculations**:
   - Convert 6-digit dates (YYMMDD) to 8-digit (CCYYMMDD) using century logic (year > 80 → 19xx, else 20xx).
   - Calculate date 30 days prior to system date for default filtering.
   - Remove leading zeros from quantities for display.

## **Error Handling**
- Return `INVALID_INPUT` with specific error messages for validation failures.
- Return `CONCURRENT_JOB` if another instance is running.
- Return `FILE_ERROR` for file access issues.

## **Dependencies**
- Files: `BBPRCE`, `GSPROD`, `GSTABL`, `GSCNTR1`, `BICONT`, `INLOC`, `RKPRCE`, `BB9531`, `BB9532`, `BB953S`, `OUTFILE`, `LIST`.
- System date (`UDATE`) for calculations.

</xaiArtifact>

---

### **Summary**

The call stack implements four use cases: generating a comprehensive rack price list, a filtered rack price list, validating user inputs, and queuing the report as a background job. The function requirement document reimagines the filtered rack price list use case as a programmatic function (`generateRackPriceList`) that accepts inputs directly, validates them, processes and filters data, sorts records, and generates formatted output, adhering to the same business rules and calculations as the original programs.