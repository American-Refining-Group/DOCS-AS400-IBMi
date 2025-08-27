The provided call stack consists of an IBM System/36 Operator Control Language (OCL) program (`BB932.ocl36.txt`) and an RPG program (`BB932.rpg36.txt`) that together implement a process to print a freight table code file. Below, I’ll identify the use cases implemented by this call stack, followed by a function requirement document that describes the process steps and business rules for the primary use case, treating it as a large function that processes inputs programmatically rather than via screen interaction.

### List of Use Cases

The call stack implements the following use cases:

1. **Print Freight Table Report**:
   - **Description**: Generates a formatted report listing freight table data, including freight codes, rates (per gallon, pound, and unit of measure), unit of measure descriptions, and company information, with pagination and error handling for invalid unit of measure codes.
   - **Inputs**: Freight table data (`BBFRTB`), company control data (`BICONT`), and system table data for unit of measure descriptions (`GSTABL`).
   - **Outputs**: A printed report (`PRINT`) with headers (company name, date, time, page number) and detail lines (freight codes, rates, and descriptions).
   - **Purpose**: Provides a readable summary of freight rates for business users, such as logistics or accounting teams, to review shipping cost structures.

2. **Validate Unit of Measure Codes**:
   - **Description**: Validates the unit of measure codes (`BFFRUM`) in the freight table against a system table (`GSTABL`) to ensure only valid codes are included in the report, with error messages for invalid codes.
   - **Inputs**: Freight table unit of measure codes (`BFFRUM`) and system table data (`GSTABL`).
   - **Outputs**: Unit of measure descriptions or error messages in the report.
   - **Purpose**: Ensures data integrity by flagging invalid unit of measure codes, aiding in data quality control.

3. **Retrieve Company Information**:
   - **Description**: Retrieves the company name associated with each freight table record’s company code (`BFCONO`) for inclusion in the report header.
   - **Inputs**: Company code (`BFCONO`) from `BBFRTB` and company control data (`BICONT`).
   - **Outputs**: Company name (`BCNAME`) in the report header or blanks if not found.
   - **Purpose**: Contextualizes the freight table data by associating it with the relevant company, useful for multi-company environments.

**Note**: The primary use case is “Print Freight Table Report,” as it encompasses the core functionality of the program. The other two use cases (validation and company lookup) are supporting functions within the primary use case. No additional independent use cases are evident from the code, as the program focuses on a single report generation process.

### Function Requirement Document

<xaiArtifact artifact_id="69720622-7fe4-45ee-9edf-fc9bebdaeec9" artifact_version_id="b87fa349-ac2c-40ce-aac1-a708b60129d3" title="Freight_Table_Report_Function_Requirements.md" contentType="text/markdown">

# Freight Table Report Function Requirements

## Purpose
The `PrintFreightTableReport` function generates a formatted report of freight table data, including freight codes, rates (per gallon, pound, and unit of measure), unit of measure descriptions, and company information, with pagination and error handling for invalid unit of measure codes.

## Inputs
- **Freight Table Data** (`BBFRTB`):
  - `BFDEL` (1 char): Delete code (indicates if record is marked for deletion).
  - `BFCONO` (2 chars): Company code.
  - `BFFRCD` (2 chars): Freight code.
  - `BFRATE` (6 chars): Freight rate per gallon.
  - `BFFRLB` (6 chars): Freight rate per pound.
  - `BFFRUM` (3 chars): Unit of measure code.
  - `BFFRPU` (6 chars): Freight rate per unit of measure.
- **Company Control Data** (`BICONT`):
  - `BCNAME` (30 chars): Company name, indexed by company code.
- **System Table Data** (`GSTABL`):
  - `TBTYPE` (6 chars): Table type (fixed to `'BIUNMS'`).
  - `TBCODE` (6 chars): Table code (matches `BFFRUM`).
  - `TBDESC` (30 chars): Unit of measure description.
- **System Date and Time**: Current date and time for report header.
- **Output Destination**: Printer or file for report output.

## Outputs
- A formatted report containing:
  - **Header**: Company name, page number, date (MMDDYY), time (HHMMSS).
  - **Detail Lines**: For each freight record:
    - Delete code (pos 1).
    - Freight code (pos 8).
    - Freight rate per gallon (pos 21).
    - Freight rate per pound (pos 34).
    - Unit of measure code (pos 39).
    - Unit of measure description or “INVALID U/M CODE” (pos 70).
    - Freight rate per unit of measure (pos 83).
  - **Formatting**: Column headers, separator lines, and pagination.

## Process Steps
1. **Initialize**:
   - Set table type to `'BIUNMS'` for unit of measure lookup.
   - Retrieve system date and time.
   - Initialize page number to 0.
2. **Process Each Freight Record**:
   - Read freight table record (`BBFRTB`).
   - Lookup company name in `BICONT` using `BFCONO`. If not found, use blank name.
   - If `BFFRUM` is non-blank, lookup description in `GSTABL` using composite key (`'BIUNMS'`, `BFFRUM`). If not found, set description to “INVALID U/M CODE”.
3. **Generate Report**:
   - Print header with company name, page number, date, and time.
   - Print column headers: “FREIGHT TABLE CODE”, “FREIGHT RATE PER GAL”, “FREIGHT RATE PER LBS”, “U/M”, “U/M DESCRIPTION”, “FREIGHT RATE PER U/M”.
   - Print detail line for each freight record with fields as specified.
   - Handle pagination, incrementing page number as needed.

## Business Rules
1. **Company Name Lookup**:
   - Retrieve company name from `BICONT` using `BFCONO`. If no match, output blank name.
2. **Unit of Measure Validation**:
   - For non-blank `BFFRUM`, validate against `GSTABL` using key (`'BIUNMS'`, `BFFRUM`). Output “INVALID U/M CODE” if no match.
3. **Include All Records**:
   - Process all `BBFRTB` records, including those with `BFDEL` set, to show active and deleted records.
4. **Report Formatting**:
   - Use fixed positions for fields and headers.
   - Include separator lines (asterisks) for readability.
   - Format date as MMDDYY, time as HHMMSS.
5. **Pagination**:
   - Increment page number for each new page and include in header.

## Calculations
- **Field Formatting**: Rates (`BFRATE`, `BFFRLB`, `BFFRPU`) are output as-is (assumed pre-formatted in input data). No explicit calculations are performed.
- **Page Number**: Incremented per page, zero-suppressed in output.
- **Date/Time**: Extracted from system, formatted as 6-character strings (MMDDYY, HHMMSS).

## Constraints
- Input files (`BBFRTB`, `BICONT`, `GSTABL`) must be accessible and indexed correctly.
- Output report is fixed at 132 characters per line, designed for printer output.
- No user interaction; function processes all records sequentially.

## Error Handling
- If `BICONT` lookup fails, output blank company name.
- If `GSTABL` lookup fails, output “INVALID U/M CODE” for unit of measure description.

</xaiArtifact>

### Notes
- **Scope**: The function requirement document focuses on the primary use case (“Print Freight Table Report”) as it encapsulates the supporting use cases (company lookup and unit of measure validation).
- **Assumptions**: The document assumes programmatic input (files) rather than screen interaction, aligning with the request. Calculations are minimal, as the RPG code performs no explicit arithmetic.
- **Further Details**: If you need requirements for additional use cases as separate functions or more technical details (e.g., file formats), please clarify!

Let me know if you need further refinements or additional artifacts!