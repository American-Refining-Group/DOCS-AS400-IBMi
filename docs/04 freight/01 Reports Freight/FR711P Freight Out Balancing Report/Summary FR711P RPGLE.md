Below is a comprehensive analysis of the use cases implemented by the provided call stack of programs (`FR712C`, `FR712A`, `FR712B`, `FR711A`, `FR711B`) and a function requirement document for one of the use cases, adapted to operate as a large function that processes inputs directly without screen interaction.

---

### List of Use Cases Implemented by the Call Stack

The call stack supports two primary use cases for generating the Freight Out Reconciliation Report, differing primarily in the sorting criteria (location vs. carrier). These use cases are executed based on the sort option (`P$SORT`) passed from the main program (`FR711P` via `FR711PC` to `FR711C` or `FR712C`). Below are the identified use cases:

1. **Generate Freight Out Reconciliation Report Sorted by Location**:
   - **Description**: Produces a report that reconciles freight charges for a specified company, date, location, and carrier, sorted hierarchically by company, location, and state. The report includes detail lines for each order and totals at the location and company levels, with an option for Excel-compatible output.
   - **Programs Involved**:
     - `FR711C`: Coordinates the creation and population of the temporary work file `FR711W` and calls the processing programs.
     - `FR711A`: Builds the `FR711W` work file by extracting, filtering, and calculating freight data.
     - `FR711B`: Prints the report from `FR711W`, sorted by location, with company and location names and totals.
   - **Trigger**: Invoked when `P$SORT = 'L'` in `FR711PC`, which calls `FR711C`.
   - **Key Features**:
     - Filters records based on company, location, carrier, and month selection.
     - Calculates billed, effective, and actual freight amounts, and their differences.
     - Supports month selection options (`M`, `O`, `C`, `A`, blank).
     - Outputs to `LIST164` (standard) and `LIST378` (Excel-compatible).

2. **Generate Freight Out Reconciliation Report Sorted by Carrier**:
   - **Description**: Produces a report that reconciles freight charges for a specified company, date, location, and carrier, sorted hierarchically by company, location, and routing group (carrier). The report includes detail lines with carrier names and totals at the location and company levels, with an option for Excel-compatible output.
   - **Programs Involved**:
     - `FR712C`: Coordinates the creation and population of the temporary work file `FR712W` and calls the processing programs.
     - `FR712A`: Builds the `FR712W` work file by extracting, filtering, and calculating freight data.
     - `FR712B`: Prints the report from `FR712W`, sorted by carrier, with company, location, and carrier names and totals.
   - **Trigger**: Invoked when `P$SORT = 'C'` in `FR711PC`, which calls `FR712C`.
   - **Key Features**:
     - Similar filtering and calculations as the location-sorted report.
     - Includes carrier name (`CICANM`) from the `bbcaid` file for enhanced readability.
     - Supports the same month selection options as the location-sorted report.
     - Outputs to `LIST164` (standard) and `LIST378` (Excel-compatible).

---

### Function Requirement Document: Generate Freight Out Reconciliation Report Sorted by Carrier

This document outlines the requirements for a function that implements the "Generate Freight Out Reconciliation Report Sorted by Carrier" use case, adapted to process inputs directly without screen interaction. The function encapsulates the logic of `FR712C`, `FR712A`, and `FR712B`, focusing on business requirements and key calculations.

<xaiArtifact artifact_id="a5f6d185-4ee9-4f10-bff7-89cc052fa646" artifact_version_id="4f0efbd7-f17d-4cad-982e-52728b810827" title="FreightOutReconciliationByCarrier.md" contentType="text/markdown">

# Function Requirement Document: Generate Freight Out Reconciliation Report Sorted by Carrier

## Overview
The function generates a Freight Out Reconciliation Report sorted by carrier, reconciling billed, effective, and actual freight charges for a specified company, date, location, and carrier. It processes input parameters, applies filtering and calculations, and produces a structured report in both standard and Excel-compatible formats.

## Inputs
- **Company Code** (`P$CO`, 2 characters): Identifies the company for which the report is generated.
- **Report Date** (`P$RDAT`, 8 digits, CYMD format): Specifies the end date of the reporting period (typically the last day of the selected month).
- **Location Code** (`P$LOC`, 3 characters): Filters records by location; blank for all locations.
- **Carrier ID** (`P$CAID`, 6 characters): Filters records by carrier; blank for all carriers.
- **Sort Option** (`P$SORT`, 1 character): Set to 'C' for carrier-sorted report.
- **Month Selection** (`P$SELMO`, 1 character): Determines record filtering:
  - `M`: Selected month, all records (open and closed).
  - `O`: Selected month, open records only.
  - `C`: Selected month, closed records only.
  - `A`: All open records from previous and selected months.
  - Blank: Previous month open records and selected month all records.
- **File Group** (`P$FGRP`, 1 character): Specifies data source:
  - `G`: Uses `gfrbinh`, `gfrbinf`, `gfrcinhj1`, `gfrbind`, `GINLOC`, `GGLCONT`, `Gbbcaid`.
  - `Z`: Uses `zfrbinh`, `zfrbinf`, `zfrcinhj1`, `zfrbind`, `ZINLOC`, `ZGLCONT`, `Zbbcaid`.

## Outputs
- **Standard Report** (`LIST164`, 164 characters wide): A formatted report with headers, detail lines, and totals, sorted by company, location, and routing group (carrier).
- **Excel-Compatible Report** (`LIST378`, 378 characters wide): A wider-format report for Excel compatibility.
- Both outputs include:
  - Header with company, location, carrier, date, job, user, and month selection details.
  - Detail lines with carrier name, shipping destination, state, ship date, customer, order details, freight amounts, and status.
  - Totals at location and company levels, including selected month and all-dates summaries.

## Process Steps
1. **Initialize Environment**:
   - Validate input parameters and set up data structures for date parsing and totals.
   - Apply file overrides based on `P$FGRP` to access the correct input files (`gfrbinh`/`zfrbinh`, etc.).

2. **Create Temporary Work File**:
   - Create or clear `FR712W` in `QTEMP` by copying the file structure from `DATA` (`P$FGRP = 'G'`) or `DATADEV` (`P$FGRP = 'Z'`).

3. **Process Freight Data**:
   - Read header records from `frbinh` (freight billing header) for the specified company.
   - Filter records based on:
     - Matching `P$CAID` (if non-blank) against `bocaid`.
     - Matching `P$LOC` (if non-blank) against `boloc`.
     - Excluding records where customer-owned product is 'Y' (`bocoon`) or where calculate freight is 'N' (`bocafr`) and freight total is zero (`bofrto`).
     - Month selection (`P$SELMO`):
       - Previous month open: `boshd8 < first day of selected month` and `boclos ≠ 'C'` for `P$SELMO = 'A'` or blank.
       - Previous month closed after report date: `boshd8 < first day of selected month`, `boclos = 'C'`, `bocld8 > P$RDAT` for `P$SELMO = 'A'` or blank.
       - Selected month: `boshd8` between first and last day of selected month; open (`boclos ≠ 'C'`) for `P$SELMO = 'M'`, `'O'`, `'A'`, or blank; closed (`boclos = 'C'`) for `P$SELMO = 'M'`, `'C'`, or blank.
   - For each valid header record:
     - Retrieve freight invoice amount (`ficfam`) from `frcinhj1`.
     - Calculate freight amounts:
       - Billed freight (`w$bfrt`): `bofrto`.
       - Effective freight (`w$efrt`): `boefr`.
       - Actual freight (`w$cfrt`): `ficfam - bobfot` (freight balancing order override total).
       - Difference (`w$diff`): If `w$efrt ≠ 0`, then `w$efrt - w$cfrt`; else `w$bfrt - w$cfrt`. Set to 0 if closed in the selected month (`boclos = 'C'` and `bocld8` matches month).
     - Read detail records from `frbind` for the header.
     - For each detail record:
       - Prorate amounts using `bdpctt` (percentage total): `w$bfrtdtl = w$bfrt * bdpctt`, `w$efrtdtl = w$efrt * bdpctt`, `w$cfrtdtl = w$cfrt * bdpctt`.
       - Calculate detail difference (`w$diffdtl`): If `w$efrtdtl ≠ 0`, then `w$efrtdtl - w$cfrtdtl`; else `w$bfrtdtl - w$cfrtdtl`. Set to 0 if closed in the selected month.
       - Write detail record to `FR712W` with fields: company, customer, name, GL account, routing code, ship date, order number, sequence, invoice, type, unit of measure, product, quantity, closed status, freight amounts, totals, location, state, routing group, city, ship number, freight code.

4. **Generate Report**:
   - Read `FR712W` records, sorted by company, location, and routing group.
   - Retrieve company name from `GLCONT`, location name from `INLOC`, and carrier name from `bbcaid`.
   - Print report headers with company, location, carrier, date, job, user, and month selection description.
   - Print detail lines with carrier name, shipping destination, state, ship date, customer, order details, freight amounts, and status.
   - Accumulate totals at routing group, location, and company levels for billed, effective, actual freight, differences, and carrier charges greater/less.
   - Print totals for selected month and all dates at location and company levels.
   - Output to `LIST164` (standard) and `LIST378` (Excel-compatible).

## Business Rules
- **Filtering**:
  - Only include records matching specified company, location (if provided), and carrier (if provided).
  - Exclude customer-owned products (`bocoon = 'Y'`) and non-calculated freight with zero amount (`bocafr = 'N'`, `bofrto = 0`).
  - Apply month selection filters based on ship date and closed status (see Inputs and Process Steps).
- **Calculations**:
  - Actual freight: Subtract freight balancing order override (`bobfot`) from invoice amount (`ficfam`).
  - Difference: Use effective freight if non-zero; otherwise, use billed freight, minus actual freight. Zero if closed in the selected month.
  - Prorate detail-level amounts using percentage total (`bdpctt`).
- **Sorting**: Hierarchical by company, location, and routing group (carrier).
- **File Group**: Use 'G' or 'Z' files based on `P$FGRP` for data source consistency.
- **Multi-User Support**: Use `QTEMP/FR712W` to ensure job-specific data isolation.
- **Output Formats**: Generate both standard (164 characters) and Excel-compatible (378 characters) reports.

## Error Handling
- If company, location, or carrier records are not found, use blank names in the report.
- Rely on system-level error handling for file access or printing issues.

## Assumptions
- Input parameters are validated by the calling program (`FR711P`).
- Input files (`frbinh`, `frbinf`, `frcinhj1`, `frbind`, `INLOC`, `GLCONT`, `bbcaid`) are available and correctly structured.
- Output printer files (`LIST164`, `LIST378`) are configured for printing.

</xaiArtifact>

---

### Notes on Other Use Case
The "Generate Freight Out Reconciliation Report Sorted by Location" use case (implemented by `FR711C`, `FR711A`, `FR711B`) is nearly identical in functionality, differing only in:
- Sorting by state (`W1SST`) instead of routing group (`W1RTG1`) at the third level of hierarchy.
- Using `FR711W` instead of `FR712W` as the work file.
- Omitting the carrier name (`CICANM`) and `bbcaid` file access, as it is not needed for location sorting.
A separate function requirement document for this use case would be similar but would adjust the sorting hierarchy and exclude the carrier name retrieval. Since the question requests a single document, the carrier-sorted use case is prioritized due to its explicit mention in the latest query (`FR712B`).