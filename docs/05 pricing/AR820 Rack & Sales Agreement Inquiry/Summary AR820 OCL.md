The call stack consists of three programs: `AR820.ocl36.txt` (OCL program), `AR820.rpg36.txt` (RPG program for customer search), and `AR822R.rpgle.txt` (RPG program for price lookup). These programs work together to facilitate a customer search and pricing inquiry system on an IBM System/36 or AS/400 environment. Below, I identify the use cases implemented by this call stack and provide a function requirement document for one of the use cases, reimagined as a non-interactive function that processes inputs directly.

---

### Use Cases Implemented by the Call Stack

The call stack implements two distinct use cases, each addressing a specific business need:

1. **Customer Search for Pricing Inquiry**:
   - **Description**: Allows a Customer Service Representative (CSR) to search for customers based on company number, customer number, product code, container code, and a partial customer name. The system retrieves and displays customer details (name, address, open balance) to help identify the correct customer for a pricing inquiry.
   - **Program Involved**: `AR820.rpg36.txt` (with setup from `AR820.ocl36.txt`).
   - **Key Features**:
     - Validates inputs (company, customer, product, container) against `ARCONT`, `ARCUST`, `GSPROD`, and `GSCNTR1`.
     - Supports partial name search using `ARCUSX` for cross-reference.
     - Displays up to 15 customer records per screen, with scrolling (roll up/down).
     - Shows open balance indicator (`OPENYN`) if total due (`ARTOTD`) is non-zero.
     - Handles function keys: F3 (rekey), F12 (exit), Roll Up/Down (scrolling).
   - **Purpose**: Enables CSRs to locate a customer before proceeding to pricing details, ensuring accurate customer selection.

2. **Pricing Lookup for Customer Orders**:
   - **Description**: Retrieves and displays pricing information (rack pricing and sales agreements) for a specific customer, product, container, and company, including freight terms and purchase order details.
   - **Program Involved**: `AR822R.rpgle.txt` (called by `AR820.ocl36.txt` after `AR820.rpg36.txt`).
   - **Key Features**:
     - Retrieves rack pricing from `BBPRCY` and sales agreements from `BICUA6`.
     - Validates inputs using `ARCUST`, `GSPROD`, `GSCNTR1`, and `SHIPTO`.
     - Displays pricing details in subfiles, including price type (standard or percent off), quantities, freight terms, and purchase order fields.
     - Handles special freight scenarios: collect (`CNY`), collect with service fee (`CYY`, $100 charge), prepaid, third-party, or PPD & ADD.
     - Supports function keys: F3/F7 (exit), F5 (sales agreement search).
     - Skips deleted or expired records and allows zero end date for sales agreements.
   - **Purpose**: Provides CSRs with accurate pricing and freight terms to quote customers or process orders.

---

### Function Requirement Document: Pricing Lookup Function

This document reimagines the **Pricing Lookup for Customer Orders** use case as a non-interactive function that takes inputs and returns pricing results without screen interaction. The function processes customer, product, container, and company inputs to retrieve rack pricing, sales agreements, and freight terms.

<xaiArtifact artifact_id="fe3fe2c0-70ba-4ba6-a5de-dbb743142df9" artifact_version_id="7e2fbd13-43b8-449d-b125-8c74fb23c8e1" title="Pricing_Lookup_Function_Requirements.md" contentType="text/markdown">

# Pricing Lookup Function Requirements

## Overview
The **Pricing Lookup Function** retrieves rack pricing and sales agreement details for a specified customer, product, container, and company, including freight terms and purchase order information. It returns structured pricing data for use in order processing or customer quoting.

## Inputs
- **Company Number (CO)**: 2-digit numeric (required).
- **Customer Number (CUST)**: 6-character alphanumeric (required).
- **Product Code (PROD)**: 4-character alphanumeric (required).
- **Container Code (CNTR)**: 3-character alphanumeric (optional, defaults to blank).
- **Current Date (DATE)**: 8-digit numeric (YYYYMMDD, defaults to system date).

## Outputs
- **Status**: Success or error code (e.g., "INVALID_COMPANY", "INVALID_CUSTOMER", etc.).
- **Customer Name (S1NAME)**: 30-character customer name.
- **Product Description (S1DESC)**: 30-character product description.
- **Container Description (S1CTDS)**: 30-character container description (if `CNTR` provided).
- **Rack Pricing Records** (array):
  - Product Code, Location, Unit of Measure, Minimum Quantity, Quantities (1–4), Prices (1–4).
- **Sales Agreement Records** (array):
  - Product Code, Ship-to Code, Location, Unit of Measure, Price, Price Type ("P" or "O"), Minimum/Maximum Quantity, Start Date, PO Number, Freight Terms.
- **Error Message**: Description of any validation failure.

## Process Steps
1. **Validate Inputs**:
   - Check `CO` against `ARCONT`. If invalid, return error "INVALID COMPANY NUMBER".
   - Check `CUST` against `ARCUST` using `CO + CUST`. If invalid, return error "INVALID CUSTOMER NUMBER".
   - Check `PROD` against `GSPROD`. If invalid, return error "INVALID PRODUCT CODE".
   - If `CNTR` non-blank, check against `GSCNTR1`. If invalid, return error "INVALID CONTAINER CODE".
   - Retrieve `S1NAME` from `ARCUST`, `S1DESC` from `GSPROD`, and `S1CTDS` from `GSCNTR1` (if applicable).

2. **Retrieve Rack Pricing**:
   - Query `BBPRCY` using `CO`, `PROD`, `CNTR`, and any location (`RKLOC`).
   - Filter out deleted records (`RKDEL = 'D'`) and records with `RKDATE > DATE`.
   - Collect up to 10 records with fields: `RKPROD`, `RKLOC`, `RKUNMS`, `RKMINQ`, `RKQT01–04`, `RKPRCE`, `RKPR02–05`.

3. **Retrieve Sales Agreements**:
   - Query `BICUA6` using `CO`, `CUST`, `PROD`, `CNTR`, and any ship-to (`BASHIP`).
   - Filter out:
     - Deleted records (`BADEL = 'D'`).
     - Non-matching records (`BACONO ≠ CO`, `BACUST ≠ CUST`, `BAPR01 ≠ PROD`, `BACNTR ≠ CNTR`).
     - Expired records (`BASTD8 > DATE` or `BAEND8 < DATE` unless `BAEND8 = 0`).
   - For each record:
     - Retrieve ship-to city (`S4DESC`) from `SHIPTO` using `CO`, `CUST`, `BASHIP`.
     - Set price (`S4PRCE`) to `BAPRCE` (if non-zero, `S4TYPE = 'P'`) or `BAOFFP` (if non-zero, `S4TYPE = 'O'`).
     - Set PO fields: `S4POY = 'PO'`, `S4PORD = BAPORD` if `BAPORD` non-blank.
     - Collect fields: `BAPR01`, `BASHIP`, `BALOC`, `BAUNMS`, `S4PRCE`, `S4TYPE`, `BAMNQY`, `BAMXQY`, `BASTD8`, `S4POY`, `S4PORD`.

4. **Determine Freight Terms**:
   - If `BAFRCD` (from `BICUA6`) is blank:
     - Query `ARCUPR` using `CO`, `CUST`, `BASHIP`, `PROD`, and container type (`TCCNTY` from `GSCNTR1`, or blank/`'P'` for non-fluid products).
     - If not found, query `ARCUSP` using `CO`, `CUST`.
   - Set `S4TERM` based on freight code (`BAFRCD`, `CPFRCD`, or `CSFRCD`):
     - `'C'`: "COLLECT" (or "COLLECT." if `CPCAFR = 'Y'` for freight collect calculation).
     - `'A'`: "PPD & ADD".
     - `'P'`: "PREPAID" (if `CPSFRT = 'Y'` or `CPCAFR = 'Y'`) or "3RD PARTY".
     - `'CYY'`: "COLLECT" with $100 service fee (freight arranged by ARG, billed by carrier).
   - Return up to 100 sales agreement records.

5. **Return Results**:
   - If rack pricing found, return array of rack pricing records.
   - If sales agreements found, return array of sales agreement records with freight terms.
   - If no pricing found, return empty arrays with appropriate status.

## Business Rules
- **Validation**: All inputs must exist in respective master files (`ARCONT`, `ARCUST`, `GSPROD`, `GSCNTR1`).
- **Rack Pricing**: Only include non-deleted records with valid dates. Display quantities and prices for multiple tiers.
- **Sales Agreements**:
  - Include non-deleted, non-expired records (zero end date allowed).
  - Display standard price (`BAPRCE`) or percent-off price (`BAOFFP`) with type indicator.
  - Include purchase order number if provided.
- **Freight Terms**:
  - Prioritize `BAFRCD`, then `CPFRCD`, then `CSFRCD`.
  - Handle special cases: `CNY` (freight collect from non-Bradford location), `CYY` (freight collect with $100 service fee).
- **Container Type**: For `ARCUPR` lookups, try container type from `GSCNTR1`, then blank, then `'P'` for non-fluid products.
- **Error Handling**: Return specific error messages for invalid inputs.

## Calculations
- **Price Selection**: Use `BAPRCE` if non-zero; otherwise, use `BAOFFP` (no additional calculation for percent-off price, per JK01).
- **Freight Service Fee**: Add $100 to customer cost for `CYY` freight terms (JB10).
- **Date Comparison**: Convert system date to YYYYMMDD for comparison with `RKDATE`, `BASTD8`, and `BAEND8`.

## Dependencies
- **Files**: `ARCONT`, `ARCUST`, `GSPROD`, `GSCNTR1`, `BBPRCY`, `BICUA6`, `ARCUPR`, `ARCUSP`, `SHIPTO`.
- **Assumptions**: Input values are trimmed and uppercase. System date is provided or defaults to current date.

</xaiArtifact>

---

### Notes
- The **Customer Search for Pricing Inquiry** use case was not converted to a function requirement document, as the pricing lookup is the primary function called by the OCL and builds on the customer search output. However, a similar function could be created for customer search, taking similar inputs and returning customer records.
- The function requirement document focuses on business requirements (validation, data retrieval, freight terms) and concisely describes calculations (price selection, freight fee). It assumes direct input processing to align with the non-interactive requirement.