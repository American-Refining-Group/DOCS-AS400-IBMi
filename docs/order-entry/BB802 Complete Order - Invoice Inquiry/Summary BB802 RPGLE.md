### List of Use Cases Implemented by the BB802 Call Stack

The **BB802** call stack, comprising programs **BB802A**, **BB802B**, **BB802E**, **BB802F**, **BB802P**, **BB802H**, **BB802I**, **BB802G**, and **BB800E**, implements a Customer Order and Invoice Inquiry system. The system retrieves and displays data for customer orders, invoices, canceled orders, and related details, with additional functionality to display accessorials and marks. Below is a list of distinct use cases derived from these programs, focusing on their data processing and inquiry capabilities:

1. **Retrieve Open Order Header Data**:
   - **Description**: Queries and aggregates open order header data from `BBORDH` into the `BB802W` work file for inquiry display.
   - **Program**: BB802A
   - **Purpose**: Supports inquiries for open orders, providing header-level details like customer, order number, ship-to, and salesman.

2. **Retrieve Open Order Detail Data**:
   - **Description**: Queries and aggregates open order detail data from `BBORDD` and `BBORDH` (via `BB802BQF`) into the `BB802W` work file.
   - **Program**: BB802B
   - **Purpose**: Provides detailed order information, including products, quantities, and locations, for open order inquiries.

3. **Retrieve Sales History Header Data**:
   - **Description**: Queries and aggregates sales history header data from `SA5SHP` into the `BB802W` work file.
   - **Program**: BB802E
   - **Purpose**: Supports inquiries for invoiced orders, providing header-level details like invoice number, ship date, and tracking number.

4. **Retrieve Sales History Detail Data (Standard)**:
   - **Description**: Queries and aggregates sales history detail data from `SA5FILD` and `SA5SHP` (via `BB802FQF`) into the `BB802W` work file.
   - **Program**: BB802F
   - **Purpose**: Provides detailed invoice information, including products and quantities, for standard sales history inquiries.

5. **Retrieve Sales History Move Detail Data**:
   - **Description**: Queries and aggregates sales history move detail data from `SA5MOVD` and `SA5SHP` (via `BB802PQF`) into the `BB802W` work file when product move flag (`P$PMOV = 'Y'`) is set.
   - **Program**: BB802P
   - **Purpose**: Supports inquiries for move-specific sales history, focusing on product movement details.

6. **Retrieve Canceled Order Header Data**:
   - **Description**: Queries and aggregates canceled order header data from `BBCNH` and `BBCNOR` (via `BB802HQF`) into the `BB802W` work file.
   - **Program**: BB802H
   - **Purpose**: Supports inquiries for canceled orders, including cancel reasons, at the header level.

7. **Retrieve Canceled Order Detail Data**:
   - **Description**: Queries and aggregates canceled order detail data from `BBCND`, `BBCNH`, and `BBCNOR` (via `BB802IQF`) into the `BB802W` work file.
   - **Program**: BB802I
   - **Purpose**: Provides detailed information for canceled orders, including products and quantities.

8. **Enrich Work File with Ship-To Location Data**:
   - **Description**: Updates the `BB802W` work file with city and state data from the `SHIPTO` file based on ship-to codes.
   - **Program**: BB802G
   - **Purpose**: Enhances inquiry data with geographic details for display.

9. **Display Customer Order or Ship-To Accessorials and Marks**:
   - **Description**: Queries and displays accessorials and marks data for customer orders (from `BBORA1`, `BBORDH`) or ship-to records (from `BBSHSA1`, `SHIPTO`) using a subfile interface.
   - **Program**: BB800E
   - **Purpose**: Provides detailed inquiry for accessorials and marks, supporting both order-specific and ship-to-specific data.

---

### Function Requirement Document

<xaiArtifact artifact_id="f3317d73-bbbe-4809-83c4-d37945013737" artifact_version_id="17c173ce-e528-4e0e-82de-1c52a08661ad" title="CustomerOrderInquiryFunction.md" contentType="text/markdown">

# Customer Order and Invoice Inquiry Function

## Purpose
The Customer Order and Invoice Inquiry function retrieves and aggregates data for open orders, canceled orders, sales history (including move details), and accessorials/marks for customer or ship-to inquiries, consolidating data into a work file for processing and optionally displaying accessorials/marks details.

## Inputs
- **Company Code** (`p$co`, 2 digits): Identifies the company.
- **Customer or Order Number** (`p$csor`, 6 digits): Specifies the customer or order to query.
- **Ship-To Code** (`p$ship`, 3 digits): Specifies the ship-to location (optional, '000' for order-level inquiries).
- **File Group** (`p$fgrp`, 1 character): 'G' or 'Z' to select the appropriate library.
- **Product Move Flag** (`p$pmov`, 1 character): 'Y' to include sales history move details, else 'N'.

## Outputs
- **Consolidated Inquiry Data**: Aggregated data in the `BB802W` work file, including order, invoice, and canceled order details with city/state.
- **Accessorials/Marks Data**: Detailed accessorials and marks for orders or ship-to records (if requested).

## Process Steps
1. **Validate Inputs**:
   - Verify `p$co`, `p$csor`, `p$ship`, and `p$fgrp` are provided and valid.
   - Exit with error if parameters are missing or invalid.

2. **Apply File Overrides**:
   - Use `p$fgrp` ('G' or 'Z') to override file libraries (e.g., `GBBORDH`/`ZBBORDH`, `GSA5SHP`/`ZSA5SHP`).

3. **Aggregate Open Order Header Data**:
   - Read `BBORDH` records matching `p$co` and `p$csor`.
   - Write to `BB802W` with fields: company, customer, order number, ship-to, purchase order, salesman, carrier, freight details, and `w1oori = 'O'`.

4. **Aggregate Open Order Detail Data**:
   - Read `BB802BQF` (joining `BBORDD` and `BBORDH`) for matching records.
   - Write to `BB802W` with additional fields: location, product, container, quantity, unit of measure, and `w1oori = 'O'`.

5. **Aggregate Sales History Header Data**:
   - Read `SA5SHP` records matching `p$co` and `p$csor`.
   - Write to `BB802W` with fields: invoice number, ship date, tracking number, and `w1oori = 'I'`. Update to `w1oori = 'B'` if matching order exists.

6. **Aggregate Sales History Detail Data**:
   - If `p$pmov = 'Y'`, read `BB802PQF` (joining `SA5MOVD` and `SA5SHP`); else, read `BB802FQF` (joining `SA5FILD` and `SA5SHP`).
   - Write to `BB802W` with additional fields: location, product, container, quantity, unit of measure, order process, and `w1oori = 'I'`. Update to `w1oori = 'B'` if matching order exists.

7. **Aggregate Canceled Order Header Data**:
   - Read `BB802HQF` (joining `BBCNH` and `BBCNOR`) for matching records.
   - Write to `BB802W` with fields: cancel reason, and `w1oori = 'C'`.

8. **Aggregate Canceled Order Detail Data**:
   - Read `BB802IQF` (joining `BBCND`, `BBCNH`, and `BBCNOR`) for matching records.
   - Write to `BB802W` with additional fields: location, product, container, quantity, unit of measure, cancel reason, and `w1oori = 'C'`.

9. **Enrich with Ship-To Location**:
   - For each `BB802W` record, read `SHIPTO` using:
     - `w1co`, `w1cust`, `w1ship` for `w1ship` between `001`–`899` or `900`–`998` (if no order-specific record).
     - `w1co`, `w1ord#`, `999` for `w1ship ≥ 900` if order-specific record exists.
   - Update `BB802W` with `w1dstc` (city) and `w1dsts` (state).

10. **Retrieve Accessorials/Marks (Optional)**:
    - If `p$ship = '000'`, read `BBORA1` and `BBORDH` for order accessorials/marks.
    - If `p$ship ≠ '000'`, read `BBSHSA1` and `SHIPTO` for ship-to accessorials/marks.
    - Return data for display or further processing.

## Business Rules
- **Data Scope**: Queries open orders (`BBORDH`, `BBORDD`), sales history (`SA5SHP`, `SA5FILD`/`SA5MOVD`), canceled orders (`BBCNH`, `BBCND`, `BBCNOR`), and accessorials/marks (`BBORA1`, `BBSHSA1`) based on `p$co` and `p$csor`.
- **Uniqueness**: Ensures no duplicate `BB802W` records using keys (`w1co`, `w1cust`, `w1ord#`, `w1inv#`).
- **Origin Indicator**:
  - `w1oori = 'O'`: Open order.
  - `w1oori = 'I'`: Invoice (sales history).
  - `w1oori = 'C'`: Canceled order.
  - `w1oori = 'B'`: Combined order and invoice (updated in `BB802E`, `BB802F`, `BB802P`).
- **Ship-To Logic**:
  - For `w1ship` between `001`–`899`, use customer-specific ship-to (`w1co`, `w1cust`, `w1ship`).
  - For `w1ship` between `900`–`998`, prefer order-specific ship-to (`w1co`, `w1ord#`, `999`) if available, else customer-specific.
- **Product Move**: If `p$pmov = 'Y'`, process `SA5MOVD` instead of `SA5FILD` for sales history details.
- **Accessorials/Marks**:
  - Order-level (`p$ship = '000'`): Retrieve from `BBORA1` and validate with `BBORDH`.
  - Ship-to level (`p$ship ≠ '000'`): Retrieve from `BBSHSA1` and validate with `SHIPTO`, `ARCUST`.
- **File Overrides**: Use `p$fgrp` ('G' or 'Z') to select library (e.g., `GBBORDH` vs. `ZBBORDH`).
- **Data Integrity**: Clear `BB802W` records before writing to prevent residual data.

## Calculations
- No explicit calculations are performed; the function aggregates and maps data fields directly.
- Date fields (e.g., `w1sdat`, `w1indt`) are copied as-is (e.g., `borqd8`, `shimdy`).
- Quantities (`w1qty`) and totals (`w1tolo`, `w1inva`, `w1lovo`) are copied from source files without manipulation.

</xaiArtifact>