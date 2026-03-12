The call stack for the Freight Out Reconciliation Report system consists of the programs `FR713P`, `FR713PC`, `FR713C`, `FR714C`, `FR715C`, `FR713A`, `FR714A`, `FR715A`, `FR713B`, `FR714B`, and `FR715B`. Based on the provided RPGLE source code for `FR713B`, `FR714A`, `FR714B`, `FR715A`, and `FR715B`, and inferred behavior from the call stack, the system implements multiple use cases to generate freight reconciliation reports sorted by different criteria. Below, I will list the use cases implemented by this call stack and then provide a concise function requirement document for one of the primary use cases, treating it as a large function that processes inputs programmatically without screen interaction.

### List of Use Cases

The call stack supports the generation of Freight Out Reconciliation Reports with different sorting criteria, each representing a distinct use case. The use cases are driven by the sort option (`p$sort`) passed to `FR713PC`, which determines whether the report is sorted by location (`FR713C` → `FR713A` → `FR713B`), carrier (`FR714C` → `FR714A` → `FR714B`), or product code (`FR715C` → `FR715A` → `FR715B`). The use cases are:

1. **Generate Freight Out Reconciliation Report Sorted by Location**:
   - **Description**: Produces a report that reconciles freight charges (booked, estimated, and actual) for shipments, sorted by company, location, state, city, routing code, customer, and order/sequence number.
   - **Programs Involved**: `FR713P` (screen input), `FR713PC` (controller), `FR713C` (location sort driver), `FR713A` (builds `FR713W`), `FR713B` (prints report).
   - **Key Features**:
     - Filters by company, location, carrier, customer, date range, and zero-dollar freight inclusion.
     - Generates hierarchical report with totals at company and location levels.
     - Outputs to `LIST164` (standard report), `LIST378` (Excel-compatible, per `mg01`), and `FR714W2` (secondary work file, likely for further processing).
     - Retrieves company and location names from `GLCONT` and `INLOC`.

2. **Generate Freight Out Reconciliation Report Sorted by Carrier/Routing**:
   - **Description**: Produces a report that reconciles freight charges, sorted by company, location, routing code, state, city, customer, and order/sequence number.
   - **Programs Involved**: `FR713P` (screen input), `FR713PC` (controller), `FR714C` (carrier sort driver), `FR714A` (builds `FR714W`), `FR714B` (prints report).
   - **Key Features**:
     - Similar filtering and output options as the location-sorted report.
     - Emphasizes carrier/routing (`W1RTG1`) in the sorting hierarchy.
     - Uses `frcinhj1` for freight billed balance headers (per `jb02`).
     - Outputs to `LIST164`, `LIST378`, and `FR714W2`.

3. **Generate Freight Out Reconciliation Report Sorted by Product Code**:
   - **Description**: Produces a report that reconciles freight charges, sorted by company, location, product code, state, city, customer, and order/sequence number.
   - **Programs Involved**: `FR713P` (screen input), `FR713PC` (controller), `FR715C` (product code sort driver), `FR715A` (builds `FR715W`), `FR715B` (prints report).
   - **Key Features**:
     - Similar filtering and output options as the other reports.
     - Emphasizes product code (`W1PROD`) in the sorting hierarchy.
     - Outputs to `LIST164`, `LIST378`, and `FR714W2` (likely a typo for `FR715W2`).

### Notes on Use Cases
- **Commonalities**: All three use cases share the same input parameters, filtering logic, and freight calculation rules, differing only in the primary sort field (`W1LOC`, `W1RTG1`, or `W1PROD`). They produce reports with identical data fields and totals, and all support Excel output (`LIST378`) and secondary work file output.
- **File Group Handling**: The `p$fgrp` parameter ('G' or 'Z') determines whether the system uses files like `gfrbinh` or `zfrbinh`, allowing flexibility for different data sets.
- **Potential Discrepancy**: The location code (`p$loc`) is defined as LEN(3) in `FR713B`, `FR714A`, `FR714B`, and `FR715A`, but LEN(2) in `FR714C` and `FR715C`, suggesting a potential inconsistency in the system design.
- **Output File Issue**: `FR715B` references `FR714W2` as the output work file, which is likely a typo for `FR715W2`, as it should align with `FR715W`.

### Function Requirement Document

Below is a concise function requirement document for the primary use case: **Generate Freight Out Reconciliation Report Sorted by Location**. This is treated as a large function that accepts inputs programmatically and produces the report and work file outputs without screen interaction. The document includes business requirements, process steps, and calculations, focusing on clarity and conciseness.

<xaiArtifact artifact_id="d44a50bc-4e5e-4c82-a862-616513152cfd" artifact_version_id="bbd20297-de7c-4586-acae-b0e1d02b4244" title="Freight_Out_Reconciliation_Location.md" contentType="text/markdown">

# Function Requirement Document: Generate Freight Out Reconciliation Report (Location-Sorted)

## Purpose
Generate a freight reconciliation report sorted by location, detailing booked, estimated, and actual freight charges for shipments, with totals at company and location levels, and supporting standard and Excel-compatible outputs.

## Inputs
- **Company Code** (`p$co`, 2 characters): Filters records by company.
- **From Date** (`p$fdat`, 8 digits, CYMD format): Start of shipment date range.
- **To Date** (`p$tdat`, 8 digits, CYMD format): End of shipment date range.
- **Location Code** (`p$loc`, 3 characters): Filters by location (optional).
- **Carrier ID** (`p$caid`, 6 characters): Filters by carrier (optional).
- **Customer Code** (`p$cust`, 6 characters): Filters by customer (optional).
- **Include Zero-Dollar Freight** (`p$car$`, 1 character, 'Y'/'N'): Includes records with zero freight if 'Y'.
- **File Group** (`p$fgrp`, 1 character, 'G'/'Z'): Selects file set (e.g., `gfrbinh` or `zfrbinh`).

## Outputs
- **Standard Report** (`LIST164`, 164 characters wide): Printed report with headers, detail lines, and totals.
- **Excel Report** (`LIST378`, 378 characters wide): Excel-compatible output.
- **Work File** (`FR714W2`): Secondary file for further processing or export.
- **Return Code**: Indicates success or failure.

## Process Steps
1. **Initialize**:
   - Validate inputs and move to work fields (`w$fdat`, `w$tdat`, `w$cust`).
   - Apply file overrides for `frbinh`, `frbinf`, `frcinhj1`, `frbind`, `inloc`, `glcont` based on `p$fgrp` ('G' or 'Z') using `QCMDEXC`.
   - Open input files (`frbinh`, `frbinf`, `frcinhj1`, `frbind`, `inloc`, `glcont`) and output files (`fr713w`, `fr714w2`, `list164`, `list378`).

2. **Build Work File (`FR713A`)**:
   - Read `frbinh` by company (`p$co`):
     - Filter by `p$caid` (if non-blank, match `bocaid`), `p$loc` (if non-blank, match `boloc`), `p$cust` (if non-zero, match `bocust`), and date range (`boshd8` in `[p$fdat, p$tdat]`).
     - For closed orders (`boclos = 'C'`) with close date (`bocld8`) ≤ `p$tdat`, set no-difference flag.
   - Read `frbinf` to accumulate booked freight (`w$bfrt = Σbftfam`).
   - Read `frcinhj1` to calculate actual/customer freight (`w$cfrt`), adjusting for freight balancing order override totals.
   - Read `frbind` to calculate prorated amounts:
     - Booked freight: `w$bfrtdtl = w$bfrt * bdpctt`.
     - Estimated freight: `w$efrtdtl = w$efrt * bdpctt`.
     - Actual freight: `w$cfrtdtl = w$cfrt * bdpctt`.
     - Difference: `w$diffdtl = (w$efrtdtl ≠ 0 ? w$efrtdtl - w$cfrtdtl : w$bfrtdtl - w$cfrtdtl)`.
     - If no-difference flag is set, `w$diffdtl = 0`.
   - Write to `fr713w` with fields: company, customer, name, GL account, routing code, shipment date, order number, sequence number, invoice, type, unit of measure, product code, quantity, close status, freight amounts, location, state, routing group, city, ship via, freight code.

3. **Print Report (`FR713B`)**:
   - Read `fr713w` hierarchically (company, location, state, city, routing code, customer, order/sequence):
     - Company break: Clear totals (`L7BFRT`, `L7EFRT`, `L7AFRT`, `L7DIFF`, `L7POST`, `L7NEGT`), retrieve company name from `glcont`.
     - Location break: Clear totals (`L6BFRT`, `L6EFRT`, `L6AFRT`, `L6DIFF`, `L6POST`, `L6NEGT`), retrieve location name from `inloc`.
     - Accumulate freight totals at company and location levels.
   - Print headers (`list164`, `list378`):
     - Job details, company, location, date/time.
     - Selection criteria: `p$loc`, `p$caid`, `p$cust`, `p$car$`, date range.
     - Columns: routing, product, destination, state, shipment date, customer, order#, sequence#, invoice, customer#, ship-to, type, quantity, unit of measure, freight code, billed/expected/actual freight, difference, close status.
   - Print detail lines from `fr713w`.
   - Print totals for location and company levels.
   - Write to `fr714w2` for further processing.

4. **Cleanup**:
   - Close all files.
   - Return success code.

## Business Rules
1. **Filtering**:
   - Include records matching `p$co`, `p$caid` (if specified), `p$loc` (if specified), `p$cust` (if specified), and shipment date in `[p$fdat, p$tdat]`.
   - Include zero-dollar freight records if `p$car$ = 'Y'`.
2. **Freight Calculations**:
   - Booked freight (`w$bfrt`): Sum of `bftfam` from `frbinf`.
   - Actual freight (`w$cfrt`): From `frcinhj1`, adjusted for override totals.
   - Prorated amounts: Multiply totals by `bdpctt` from `frbind`.
   - Difference: Use estimated freight if available, else booked freight, minus actual freight; zero for closed orders in period.
3. **Hierarchical Sorting**:
   - Sort by company, location, state, city, routing code, customer, order/sequence.
4. **Reference Data**:
   - Retrieve company name (`GCNAME`) from `glcont` and location name (`ILNAME`) from `inloc`, defaulting to blank if not found.
5. **Output Formats**:
   - Standard report (`list164`) for printing.
   - Wide report (`list378`) for Excel.
   - Work file (`fr714w2`) for external processing.
6. **File Group**:
   - Use `gfr*` files for `p$fgrp = 'G'`, `zfr*` for `p$fgrp = 'Z'`.

## Dependencies
- **Files**: `frbinh`, `frbinf`, `frcinhj1`, `frbind`, `inloc`, `glcont`, `fr713w`, `fr714w2`, `list164`, `list378`.
- **External Program**: `QCMDEXC` for file overrides.

## Notes
- Assumes input validation is handled externally (e.g., by `FR713P`).
- Location code length inconsistency (LEN(3) in `FR713B` vs. LEN(2) in `FR714C`/`FR715C`) may require resolution.

</xaiArtifact>

### Notes on Function Requirement Document
- **Scope**: Focused on the location-sorted report (use case 1) as the primary case, but the process and rules are nearly identical for carrier and product code sorting, differing only in the sorting hierarchy (`W1RTG1` for `FR714B`, `W1PROD` for `FR715B`).
- **Programmatic Approach**: The document assumes inputs are provided programmatically, eliminating screen interaction (`FR713P`’s role).
- **Output File Naming**: The document uses `FR714W2` as per `FR713B`’s code, consistent across `FR714B` and `FR715B` (though `FR715B`’s use of `FR714W2` is likely a typo for `FR715W2`).
- **Calculations**: Concisely described to clarify freight proration and difference logic.
- **Business Rules**: Emphasized filtering, sorting, and output requirements to meet business needs.

This document provides a clear, business-oriented specification for the location-sorted report, adaptable to the other use cases by adjusting the sorting field.