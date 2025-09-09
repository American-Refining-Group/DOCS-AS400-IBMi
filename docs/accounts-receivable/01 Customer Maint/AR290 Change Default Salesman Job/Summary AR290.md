### List of Use Cases Implemented by the AR290 OCL and RPG Programs

The `AR290.ocl36.txt` OCL program, along with its associated RPG programs (`AR290`, `AR291`, `AR294`, `AR295`, `AR299`), implements a single primary use case:

1. **Update Salesman Codes for Selected Customers Across Multiple Files**:
   - **Description**: This use case updates the salesman code for specified customers (identified by company and customer numbers) from an old salesman code to a new salesman code in various accounts receivable (A/R) system files. It processes customer master, order, billing, sales analysis, freight, and invoice transaction files, logs changes, and generates a report summarizing the updates.
   - **Scope**: The process ensures consistent salesman code updates across all relevant files, validates customer data, tracks the number of updates, and provides an audit trail via a report.

### Function Requirement Document

<xaiArtifact artifact_id="ed716420-7dbc-4097-8103-386890ecf9e5" artifact_version_id="1daeea7b-4542-471f-9059-216a23541ddb" title="Salesman_Code_Update_Requirements.md" contentType="text/markdown">

# Salesman Code Update Function Requirements

## Overview
The Salesman Code Update function updates salesman codes for selected customers across multiple accounts receivable system files, validates customer data, logs changes, and generates a summary report.

## Inputs
- **Salesman Change Transaction File (`ARSLST`)**:
  - Company Number (`ASCO`, 2 chars): Identifies the company.
  - Customer Number (`ASCUST`, 6 chars): Identifies the customer.
  - Old Salesman Code (`ASSLSO`, 2 chars): Current salesman code (optional, may be zero).
  - New Salesman Code (`ASSLSN`, 2 chars): Target salesman code.
  - Delete Flag (`ASDEL`, 1 char): 'D' indicates record to skip.
- **Customer Master File (`ARCUST`)**: Contains customer details (name, salesman code).
- **General Table File (`GSTABL`)**: Contains salesman descriptions.
- **Catalog File (`AR29WS`)**: Lists invoice transaction file names.
- **Data Files**:
  - Order Header (`BBORDH`), Bill of Lading (`BBBOLX`), Batch Orders (`BBOR01–BBOR98`), Invoice Transactions (`BBTRXX`), Sales Analysis Files (`SA5FIXD`, `SA5FIXM`, `SA5DBDX`, `SA5DBMX`, `SA5BCDX`, `SA5BCMX`, `SA5CODX`, `SA5COMX`), Freight Bill History (`FRBINH2`), Shipping Units (`SA5SHU`).

## Outputs
- **Updated Files**: Updated salesman codes in `ARCUST`, `BBORDH`, `BBBOLX`, `BBORXX`, `BBTRXX`, `SA5FIXD`, `SA5FIXM`, `SA5DBDX`, `SA5DBMX`, `SA5BCDX`, `SA5BCMX`, `SA5CODX`, `SA5COMX`, `FRBINH2`, `SA5SHU`.
- **Customer History File (`ARCUSHS`)**: Logs each salesman code change with timestamp and user ID.
- **Updated `ARSLST`**: Includes counts of updated records per file.
- **Report (`ARPRINT`)**: Summarizes updates by customer, including company, customer number, name, old/new salesman codes, salesman description, and update counts for each file.
- **Cleared Files**: `SLSCHG` deleted, `ARSLST` cleared after processing.

## Process Steps
1. **Validate Input**:
   - Check if `ARSLST` exists and is non-empty; skip if empty or `SLSCHG` exists.
2. **Update Customer and Related Files**:
   - Read `ARSLST` records (non-deleted).
   - Validate customer in `ARCUST` using company (`ASCO`) and customer (`ASCUST`) numbers.
   - Update salesman code to `ASSLSN` in:
     - `ARCUST` (customer master).
     - `BBORDH`, `BBBOLX` (orders and bills of lading).
     - `SA5FIXD`, `SA5FIXM`, `SA5DBDX`, `SA5DBMX`, `SA5BCDX`, `SA5BCMX`, `SA5CODX`, `SA5COMX` (sales analysis).
     - `FRBINH2`, `SA5SHU` (freight and shipping).
   - Log change in `ARCUSHS` with timestamp, user ID ('GAIL'), and change type ('DSPM2').
   - Count updates per file (`ASBBO`, `ASBBB`, `ASS5F`, `ASS5D`, `ASS5B`, `ASS5C`, `ASS5S`, `ASSFR`, `ASORT`, `ASBBT`) and store in `ARSLST`.
3. **Update Batch Order Files**:
   - Loop through `BBOR01–BBOR98`.
   - Update salesman code in matching records (company, customer).
   - Increment `ASORT` counter in `ARSLST`.
4. **Update Invoice Transaction Files**:
   - Generate catalog (`AR29WS`) of invoice transaction files (`BBTRXX`).
   - For each unprocessed file in `AR29WS`:
     - Retrieve file name, mark as processed ('Y').
     - Update salesman code in matching `BBTRAN` records.
     - Increment `ASBBT` counter in `ARSLST`.
   - Delete `AR29WS` after processing.
5. **Generate Report**:
   - Read `ARSLST`, retrieve customer name from `ARCUST`, salesman description from `GSTABL`.
   - Print report with headers (page, date, time, columns) and details (company, customer, name, old/new salesman, description, update counts).
   - Handle invalid customers/salesmen by displaying 'INVALID'.
6. **Cleanup**:
   - Delete `SLSCHG`.
   - Clear `ARSLST`.

## Business Rules
1. **Update Criteria**:
   - Update records where company (`CO`) and customer (`CUST`) match `ASCO` and `ASCUST`.
   - Salesman code comparison (`SLSM` vs. `ASSLSO`) is bypassed (per MG04, 09/26/2017); updates rely on customer/company match.
   - New salesman code (`ASSLSN`) replaces existing code (`SLSM`).
2. **Validation**:
   - Skip `ARSLST` records with `ASDEL` = 'D'.
   - Validate customers against `ARCUST`; skip updates if not found (except for history logging).
3. **Audit and Tracking**:
   - Log changes in `ARCUSHS` with timestamp, user, and change type.
   - Track update counts per file in `ARSLST`.
   - Report includes all non-deleted `ARSLST` records, with 'INVALID' for missing customer/salesman data.
4. **File Processing**:
   - Process batch orders (`BBOR01–BBOR98`) sequentially.
   - Process invoice transaction files (`BBTRXX`) via catalog (`AR29WS`), ensuring no duplicates.
5. **Data Integrity**:
   - Use shared access (`DISP-SHRMM`) for concurrent file access.
   - Clear `ARSLST` and delete `SLSCHG` to prevent reprocessing.

## Calculations
- **Update Counts**:
  - Increment counters (`ASBBO`, `ASBBB`, `ASORT`, `ASBBT`, `ASS5F`, `ASS5D`, `ASS5B`, `ASS5C`, `ASS5S`, `ASSFR`) for each updated record in respective files.
  - Store counts in `ARSLST` fields (positions 14–133).
- **Timestamp**:
  - Capture system time (`TIMEX`) and format as `TIMEOF` (HHMMSS) and `DATE` (MMDDYY or YYMMDD) for logging/reporting.

## Constraints
- Files must exist and be accessible with shared memory access.
- `AR29WS` catalog must not contain duplicate keys.
- Report output directed to `QUSRSYS/SLSMCHANGE` (production) or `QUSRSYS/TESTOUTQ` (testing).

</xaiArtifact>