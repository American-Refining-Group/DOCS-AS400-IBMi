The call stack provided includes several programs (`BI941P.ocl36`, `BI941P.rpg36`, `BI941.ocl36`, `BI9413.rpg36`, `BI9411.rpg36`, `BI9412.rpg36`, `BI941.rpg36`, `BI941A.rpg36`) that work together to generate a customer sales agreement list and optionally check for duplicate agreements. Below, I’ll identify the use cases implemented by this call stack and then provide a concise function requirement document for the primary use case, assuming it operates as a large function with direct input parameters rather than screen interaction.

---

### **List of Use Cases**

The call stack implements two distinct use cases:

1. **Generate Customer Sales Agreement List**:
   - **Description**: Produces a detailed report (printed and Excel-compatible) of customer sales agreements based on user-specified criteria, such as company, customer, contract status, sort order, salesman, location, product, brand, product class, product group, and inventory group. The report includes company and customer details, product codes, pricing, contract details, and more, enriched with data from reference files.
   - **Components Involved**:
     - `BI941P.ocl36`: Initiates the process and calls the interactive RPG program.
     - `BI941P.rpg36`: Validates user input for filtering criteria and retrieves company data for display.
     - `BI941.ocl36`: Orchestrates preprocessing, sorting, and report generation.
     - `BI9413.rpg36`: Preprocesses agreement data, updating zero end dates and filtering by brand, product class, group, inventory group, and bill-to PO.
     - `BI9411.rpg36`: Adds salesman numbers from the customer master to agreement records (for salesman-sorted reports).
     - `BI9412.rpg36`: Clears salesman numbers for customer-sorted reports.
     - `BI941.rpg36`: Generates the final printed and Excel reports with enriched data.
   - **Inputs**:
     - Company (`ALL` or specific), customer (`ALL` or selected), contract (`ALL` or current), sort order (`N` for name, `S` for salesman), salesman (`ALL` or specific), location, product codes, brand name, product class, product group, inventory group, exclude PO pricing flag.
   - **Outputs**:
     - Printed report (`PRINTER`) and Excel-compatible report (`PRTEXCEL`) with agreement details.

2. **Identify Duplicate Current Customer Sales Agreements**:
   - **Description**: Detects and reports duplicate customer sales agreements that are active (current) based on the system date, using a composite key (company, product code, container code, ship-to). Outputs duplicate records to a file and generates a printed report.
   - **Components Involved**:
     - `BI941.ocl36`: Conditionally invokes `BI941A` (disabled by default due to key reference issues).
     - `BI941A.rpg36`: Processes agreement records, checks for duplicates, and outputs current duplicates to a file and printer.
   - **Inputs**:
     - Current system date, agreement data from `BICUAGXX`.
   - **Outputs**:
     - `BI941O` file with processed records, printed report of current duplicate agreements.
   - **Note**: This use case is optional and disabled by default, as the OCL program warns against running `BI941A` unless its key references are updated.

---

### **Function Requirement Document**

#### **Function Name**: GenerateCustomerSalesAgreementList

#### **Purpose**
Generate a comprehensive customer sales agreement report (printed and Excel-compatible) based on specified filtering and sorting criteria, enriched with company, customer, product, and container details.

#### **Inputs**
- **Company** (`KYALCO`, `KYCO1`): `ALL` or specific company number (2 digits, validated against `BICONT`).
- **Customer** (`KYALCS`, `CS`): `ALL` or array of up to 10 customer numbers (6 digits each, validated against `ARCUST`).
- **Contract** (`KYALCN`, `KYDAT8`): `ALL` or `CUR` (current contracts, based on system date).
- **Sort Order** (`KYSORT`): `N` (name) or `S` (salesman).
- **Salesman** (`KYALSL`, `KYFRSL`): `ALL` or specific salesman number (2 digits, validated against `GSTABL` type `SLSMAN`).
- **Location** (`KYLOC`): Blank (all) or specific location code (3 characters).
- **Product Codes** (`KYPROD`): Blank (all) or up to 10 product codes (4 characters each, validated against `GSPROD`).
- **Brand Name** (`KYBRND`): Blank (all) or specific brand name (20 characters, validated against `GSCNTR1`).
- **Product Class** (`KYCLCD`): Blank (all) or specific class code (3 characters, validated against `GSTABL` type `PRODCL`).
- **Product Group** (`KYGRCD`): Blank (all) or specific group code (2 characters, validated against `GSTABL` type `PRODGR`).
- **Inventory Group** (`KYIVGR`): Blank (all) or specific inventory group code (2 characters, validated against `GSTABL` type `INVGRP`).
- **Exclude PO Pricing** (`KYEXPO`): `Y` (exclude agreements with bill-to PO) or `N`.
- **System Date** (`UDATE`): Current date for validating current contracts.

#### **Outputs**
- **Printed Report** (`PRINTER`): Formatted report with headers, grouped by company/customer/salesman, including:
  - Company number/name, customer number/name, location, product codes (up to 10) with descriptions, start/end dates and times, price, off-price, minimum/maximum gallons, prepaid flag, apply-to-all-ship-to flag, pricing method, container code, unit of measure, bill-to PO, ship-to number, contract number.
- **Excel Report** (`PRTEXCEL`): Spreadsheet-compatible output with similar details, plus metadata (system date/time, filter criteria like brand, product class, group, inventory group).
- **Intermediate Files**:
  - `BICUAGO`: Pre-processed agreement data.
  - `BI941S`: Sorted agreement data.

#### **Process Steps**
1. **Validate Inputs**:
   - Ensure company is `ALL` or valid (`BICONT`).
   - Ensure customer is `ALL` or valid non-zero numbers (`ARCUST`).
   - Ensure contract is `ALL` or `CUR` (set `KYDAT8` to current date for `CUR`).
   - Ensure sort order is `N` or `S`.
   - Ensure salesman is `ALL` or valid (`GSTABL` type `SLSMAN`); if `ALL`, no salesman number allowed; if `SEL`, non-zero number required.
   - Ensure location, product codes, brand, product class, group, and inventory group are blank or valid against respective files.
   - Ensure exclude PO pricing is `Y` or `N`.

2. **Preprocess Agreements**:
   - Read `BICUAG` records.
   - Update zero end dates (`BAENDT`, `BAEND8`) to `791231`/`20791231`.
   - Filter by brand (`KYBRND` matches `GSCNTR1.TCDES2`), exclude PO (`KYPORD = 'Y'` and `BAPORD` non-blank), end date (`BAEND8` vs. `KYENDT`), product class (`KYCLCD` vs. `GSPROD.TPPRCL`), group (`KYGRCD` vs. `GSPROD.TPPRGP`), inventory group (`KYIVGR` vs. `GSPROD.TPINGP`).
   - Write filtered records to `BICUAGO`.

3. **Sort Data**:
   - Sort `BICUAGO` into `BI941S` based on:
     - Company, customer, container code, unit of measure, bill-to PO, location, contract, sequence number.
     - Up to 10 product codes if specified.
   - If salesman sort (`KYSORT = 'S'`):
     - Add salesman number from `ARCUST` to `BI941S` (position 258).
   - If customer sort (`KYSORT = 'N'` and `KYALSL = 'SEL'`):
     - Clear salesman number in `BI941S` to `'00'`.

4. **Generate Report**:
   - Read `BI941S` records.
   - Enrich with:
     - Company name (`BICONT.BCNAME`).
     - Customer name (`ARCUST.ARNAME`, `ARNM20`).
     - Product descriptions (`GSPROD.TPDES1` or `GSTABL.TBDESC`).
     - Container description (`GSCNTR1.TCDESC`).
     - Salesman description (`GSTABL.TBDESC`, type `SLSMAN`).
   - Format output:
     - **Printed Report**: Group by company (`L1`), customer (`L2`), or salesman (`L3`); include headers, product codes (up to 10), dates (`NEVER` if no end date), pricing, gallons, and contract details.
     - **Excel Report**: Include metadata (date, time, filters) and agreement details in spreadsheet format.

#### **Business Rules**
1. **Input Validation**:
   - Company: `ALL` or valid number in `BICONT`.
   - Customer: `ALL` or non-zero numbers in `ARCUST`.
   - Contract: `ALL` or `CUR` (current date between start/end dates).
   - Sort: `N` (name) or `S` (salesman).
   - Salesman: `ALL` (no number) or valid number in `GSTABL` (type `SLSMAN`); mutually exclusive with `ALL`.
   - Location/Product/Brand/Class/Group/Inventory Group: Blank (all) or valid in respective files (`GSPROD`, `GSCNTR1`, `GSTABL`).
   - Exclude PO: `Y` (exclude non-blank `BAPORD`) or `N`.

2. **Data Filtering**:
   - Exclude deleted records (`BADEL = 'D'`).
   - Filter by brand, product class, group, inventory group, and bill-to PO as specified.
   - For `CUR` contracts, include only agreements where the current date is between `BASTD8` and `BAEND8`.

3. **Date Handling**:
   - Convert zero end dates to `791231`/`20791231`.
   - For `CUR` contracts, compute current date as `20YYMMDD` (from `UDATE`).

4. **Sorting**:
   - Sort by company, customer, and other fields (container, unit of measure, etc.).
   - For salesman sort (`S`), include salesman number; for customer sort (`N` with `SEL`), clear salesman number.

5. **Report Content**:
   - Include company/customer names, location, up to 10 product codes with descriptions, start/end dates (with `NEVER` for no end date), pricing (price, off-price, method), gallons (min/max), prepaid flag, apply-to-all-ship-to flag, container code, unit of measure, bill-to PO, ship-to, contract number.

#### **Calculations**
- **Date Conversion**:
  - Zero end dates: Set `BAENDT = 791231`, `BAEND8 = 20791231`.
  - Current date: `KYDAT8 = UDATE * 10000.01` with century `20` (e.g., `20YYMMDD`).
- **Pricing**: Format packed fields `BAPRCE`, `BAOFFP` for display.
- **Gallons**: Format `BAMNGL`, `BAMXGL` for display.

#### **Assumptions**
- Input files (`BICUAG`, `BICONT`, `ARCUST`, `GSTABL`, `GSPROD`, `GSCNTR1`) are populated and accessible.
- Output files (`BICUAGO`, `BI941S`, `PRINTER`, `PRTEXCEL`) are writable.
- Input parameters are pre-validated or default to `ALL`/`N` where applicable.

#### **Constraints**
- Up to 10 product codes per agreement.
- Maximum 999,000 records in sorted file (`BI941S`).
- Excel report format assumes compatibility with external spreadsheet tools.

---

### **Notes on Duplicate Agreement Check Use Case**
The duplicate agreement check (`BI941A`) is not included in the function requirement document as it is disabled by default in `BI941.ocl36` due to key reference issues. If needed, it could be implemented as a separate function with the following requirements:
- **Purpose**: Identify and report duplicate current agreements.
- **Inputs**: System date, `BICUAGXX` records.
- **Process**: Check for duplicates using a key (company, product code 1, container code, ship-to); output duplicates to `BI941O` and print current duplicates.
- **Business Rules**: Consider agreements current if the system date is between start/end dates; use composite key for duplicate detection.
- **Outputs**: `BI941O` file and printed report of duplicates.

This use case requires updating `BI941A`’s key structure before implementation, as noted in the OCL program.

---

<xaiArtifact artifact_id="b2c38d62-2f12-4f77-b021-0ee1efe3cc95" artifact_version_id="ad5b8330-dd96-4eae-8e74-7e7a19b6bd5d" title="CustomerSalesAgreementFunctionRequirements.md" contentType="text/markdown">

# Customer Sales Agreement List Function Requirements

## Function Name
GenerateCustomerSalesAgreementList

## Purpose
Generate a comprehensive customer sales agreement report (printed and Excel-compatible) based on specified filtering and sorting criteria, enriched with company, customer, product, and container details.

## Inputs
- **Company** (`KYALCO`, `KYCO1`): `ALL` or 2-digit company number (validated against `BICONT`).
- **Customer** (`KYALCS`, `CS`): `ALL` or array of up to 10 6-digit customer numbers (validated against `ARCUST`).
- **Contract** (`KYALCN`, `KYDAT8`): `ALL` or `CUR` (current contracts, based on system date).
- **Sort Order** (`KYSORT`): `N` (name) or `S` (salesman).
- **Salesman** (`KYALSL`, `KYFRSL`): `ALL` or 2-digit salesman number (validated against `GSTABL` type `SLSMAN`).
- **Location** (`KYLOC`): Blank (all) or 3-character location code.
- **Product Codes** (`KYPROD`): Blank (all) or up to 10 4-character product codes (validated against `GSPROD`).
- **Brand Name** (`KYBRND`): Blank (all) or 20-character brand name (validated against `GSCNTR1`).
- **Product Class** (`KYCLCD`): Blank (all) or 3-character class code (validated against `GSTABL` type `PRODCL`).
- **Product Group** (`KYGRCD`): Blank (all) or 2-character group code (validated against `GSTABL` type `PRODGR`).
- **Inventory Group** (`KYIVGR`): Blank (all) or 2-character inventory group code (validated against `GSTABL` type `INVGRP`).
- **Exclude PO Pricing** (`KYEXPO`): `Y` (exclude agreements with bill-to PO) or `N`.
- **System Date** (`UDATE`): Current date for validating current contracts.

## Outputs
- **Printed Report** (`PRINTER`): Grouped by company/customer/salesman, including company/customer names, location, product codes (up to 10) with descriptions, start/end dates and times, price, off-price, minimum/maximum gallons, prepaid flag, apply-to-all-ship-to flag, pricing method, container code, unit of measure, bill-to PO, ship-to, contract number.
- **Excel Report** (`PRTEXCEL`): Spreadsheet with metadata (date, time, filters) and agreement details.
- **Intermediate Files**: `BICUAGO` (pre-processed data), `BI941S` (sorted data).

## Process Steps
1. **Validate Inputs**:
   - Ensure company, customer, contract, sort order, salesman, location, product, brand, class, group, inventory group, and PO flag are valid.
2. **Preprocess Agreements**:
   - Update zero end dates to `791231`/`20791231`.
   - Filter by brand, PO, end date, product class, group, inventory group.
   - Write to `BICUAGO`.
3. **Sort Data**:
   - Sort `BICUAGO` into `BI941S` by company, customer, container, etc.
   - Add salesman number (`ARCUST`) for salesman sort; clear for customer sort.
4. **Generate Report**:
   - Enrich `BI941S` records with company (`BICONT`), customer (`ARCUST`), product (`GSPROD`, `GSTABL`), and container (`GSCNTR1`) data.
   - Output printed and Excel reports with formatted details.

## Business Rules
1. **Validation**:
   - Company: `ALL` or valid in `BICONT`.
   - Customer: `ALL` or valid in `ARCUST`.
   - Contract: `ALL` or `CUR` (current date between start/end).
   - Sort: `N` or `S`.
   - Salesman: `ALL` (no number) or valid in `GSTABL`; exclusive with `ALL`.
   - Location/Product/Brand/Class/Group/Inventory Group: Blank or valid.
   - Exclude PO: `Y` (exclude non-blank `BAPORD`) or `N`.
2. **Filtering**:
   - Exclude deleted records (`BADEL = 'D'`).
   - Apply brand, product, PO, and date filters.
3. **Date Handling**:
   - Zero end dates: `791231`/`20791231`.
   - Current date: `20YYMMDD`.
4. **Sorting**:
   - Salesman sort: Include salesman number.
   - Customer sort: Clear salesman number if `SEL`.
5. **Report Content**:
   - Include enriched data, format dates (`NEVER` for no end date), and group by company/customer/salesman.

## Calculations
- **Date Conversion**: `KYDAT8 = UDATE * 10000.01` + century `20`; zero end dates set to `791231`/`20791231`.
- **Pricing/Gallons**: Format packed fields (`BAPRCE`, `BAOFFP`, `BAMNGL`, `BAMXGL`) for display.

## Assumptions
- Input files (`BICUAG`, `BICONT`, `ARCUST`, `GSTABL`, `GSPROD`, `GSCNTR1`) are populated.
- Output files (`BICUAGO`, `BI941S`, `PRINTER`, `PRTEXCEL`) are writable.

## Constraints
- Up to 10 product codes.
- Maximum 999,000 records in `BI941S`.
- Excel report assumes external spreadsheet compatibility.

</xaiArtifact>