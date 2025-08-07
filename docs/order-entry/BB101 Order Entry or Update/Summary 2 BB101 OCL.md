### Summary of Use Cases Implemented by the Programs

The provided programs (`BB101`, `BB001`, `BB003`, `BB005`, `BB215`, and `IN805BC`) form part of an order entry system on an IBM AS/400 (iSeries) platform, called by the main OCL program `BB101.ocl36.txt`. Below is a summary of the use cases implemented by each program, as previously detailed:

1. **BB101 (Order Entry and Management)**:
   - Manages the creation, validation, updating, and finalization of sales orders, handling header, detail, and miscellaneous line items, including freight and tax calculations, and ensuring compliance with business rules.

2. **BB001 (Batch Management for Order Entry)**:
   - Creates, selects, updates, or deletes order batches, ensuring proper batch number assignment, user-specific locking, and compatibility with the order entry mode (e.g., standard or viscosity entry).

3. **BB003 (Batch Transaction Record Deletion)**:
   - Deletes records from order transaction files (`BBORCL`, `BBOTHS1`, `BBOTDS1`, `BBOTA1`) associated with a deleted batch, maintaining data consistency by removing all records linked to a company and order number.

4. **BB005 (Batch Status Update for Order Processing)**:
   - Updates the status of an order batch to release, mark for pick list, mark as bill of lading printed, or post the batch, facilitating progression through the order entry workflow.

5. **BB215 (Order Lockout Removal)**:
   - Clears lock status and workstation ID fields in order header records (`BBORDRH`) for orders identified in transaction records (`BBORTR`), ensuring orders are accessible after batch deletion or lock-inducing activities.

6. **IN805BC (Work File Preparation for Tank Total Accumulation)**:
   - Creates and populates temporary work files (`INWZHW`, `INWZ10W`) in `QTEMP` by copying data from S/36 files, enabling tank total accumulation for inventory processing.

### Function Requirement Document: Integrated Order Entry and Batch Processing

<xaiArtifact artifact_id="30d7b131-07f5-45ca-8c92-fd4a5d81f62a" artifact_version_id="52bf4640-cf03-420a-9811-13e494a05e96" title="Integrated_Order_Entry_Requirements.md" contentType="text/markdown">

# Integrated Order Entry and Batch Processing Function Requirements

## Overview
The Integrated Order Entry and Batch Processing function manages the end-to-end order entry process in an IBM AS/400 system, including batch creation, order entry, transaction cleanup, batch status updates, lockout removal, and inventory work file preparation. It ensures accurate order processing, data consistency, and inventory alignment.

## Inputs
- **Batch Inputs**:
  - Batch Number (1-99), Program Mode ('O', 'L', 'B', 'P'), Batch Source (e.g., 'PP'), User ID, Workstation ID, Record Count, Delete Flag ('D' or blank).
- **Order Inputs**:
  - Company Number (2 chars), Order Number (6 digits), Customer Number, Ship-to Number, Order/PO/Delivery Dates, Freight Code ('C', 'P', 'A', 'CYY'), Carrier ID, Product Code, Container Code, Quantity, Unit Price, Miscellaneous Line Details, Tax Codes/Amounts (up to 10), Remarks, Customer-Owned Product Flag ('Y' or blank), Hand Ticket Number, Incoterms, Country Code.
- **Inventory Inputs**:
  - File Group (1 char, e.g., 'G'), Company Number (2 chars).
- **Lockout Removal Inputs**:
  - Company Number, Order Number.

## Process Steps
1. **Create or Select Batch**:
   - Generate new batch number (1-99, reset at 99) or select existing batch.
   - Set batch source, user, workstation, and creation date; update record count.
   - Validate batch availability (not locked by another user or deleted).

2. **Enter and Validate Order**:
   - Create/update order header (`BBORTR`) with customer, ship-to, dates, freight, and carrier details.
   - Validate company, customer, ship-to, dates (not >30 days past or holidays), carrier (active, not 'CC', 'CT', 'SCO'), and bill-to PO#.
   - Add line items with product, container, quantity, and price; validate against master files (`GSPROD`, `GSCNTR1`, `GSCTWT`).
   - For customer-owned products (`COON='Y'`), bypass inventory/credit checks, force no-charge, and zero miscellaneous quantities.
   - Process miscellaneous lines (e.g., surcharges, hide seq 940+/950).
   - Validate tax codes/amounts and allow overrides.

3. **Calculate Freight and Quantities**:
   - Compute freight based on freight code:
     - Collect (`C`): Customer/Product price = 1.00, Freight = 0.00.
     - PPD&ADD (`A`): Customer/Product price = 1.00, Freight = 0.25.
     - Prepaid (Rack): Customer/Product price = 1.00, Freight = 0.25.
     - Prepaid (Sales Agreement): Customer price = 1.00, Product price = 0.75, Freight = 0.25.
     - Collect with Service Fee (`CYY`): $100 fee, freight by carrier.
   - Auto-populate carrier ID from top 5 preferences if blank.
   - Calculate gross/net gallons, shipping weight, and unit measure quantities; ensure load volume ≤ 7/0.
   - Warn if freight calculation requested but none calculated.

4. **Prepare Inventory Work Files**:
   - Create `QTEMP/INWZHW` and `QTEMP/INWZ10W` from `DATA` (production, `P$FGRP='G'`) or `DATADEV` (development).
   - Copy S/36 files (e.g., `GINWZH`, `GINWZ10`) to work files, replacing records without validation.

5. **Finalize Order and Batch**:
   - Validate no override codes ('X') on detail lines.
   - Check for duplicate orders/hand tickets (include customer, ignore deleted).
   - Update supplemental files (`BBOTHS1`, `BBOTDS1`), remarks, and totals.
   - Update batch status (`BBBTCH`):
     - Release (`O`): Clear locks.
     - Pick List (`L`): Clear locks.
     - Bill of Lading (`B`): Clear locks, set BOL printed ('Y').
     - Post (`P`): Mark deleted ('D'), set lock to 'P'.

6. **Delete Batch Transactions (if deleted)**:
   - Remove records from `BBORCL`, `BBOTHS1`, `BBOTDS1`, `BBOTA1` matching company/order number.

7. **Remove Order Lockouts**:
   - Clear lock status and workstation ID in `BBORDRH` for orders matching company/order number from `BBORTR`.

## Business Rules
- **Batch Management**:
  - Batch numbers 1-99, reset at 99; creation restricted to order entry mode (`PGM='O'`).
  - Batch source must match mode (e.g., 'PP'); locked batches accessible only by locking user.
  - Deleted batches skip selection; deletion requires confirmation.
- **Order Validation**:
  - Validate company, customer, ship-to, dates, carrier, and PO#.
  - Customer-owned products bypass inventory/credit checks, force no-charge, and zero miscellaneous quantities.
  - Freight codes consistent across line items; railcar orders prohibit multi-load, require 'C'/'L'.
  - Inactive rack prices block orders; stock depletion allows override.
- **Calculations**:
  - Freight calculated per code; warn if none calculated when requested.
  - Quantities/weights calculated for totals; load volume capped at 7/0.
- **Inventory Files**:
  - Work files in `QTEMP`; production uses `DATA`, development uses `DATADEV`.
  - Data copied without validation for S/36 compatibility.
- **Batch Status**:
  - Update status per mode; posting marks deletion.
- **Transaction Cleanup**:
  - Delete all transaction records for deleted batches.
- **Lockout Removal**:
  - Clear locks for matching order headers post-deletion.

## Outputs
- **Batch Records**: Updated `BBBTCH` with status, locks, and counts.
- **Order Records**: Updated `BBORTR`, `BBTRTX`, `BBOTHS1`, `BBOTDS1` with order data.
- **Work Files**: Populated `QTEMP/INWZHW`, `INWZ10W` for inventory.
- **Unlocked Orders**: Updated `BBORDRH` with cleared locks.
- **Deleted Transactions**: Removed records from `BBORCL`, `BBOTHS1`, `BBOTDS1`, `BBOTA1`.
- **Error/Warning Messages**: For invalid inputs, duplicates, or calculation issues.

## Dependencies
- **Files**:
  - `BBBTCH`, `BBORTR`, `BBORCL`, `BBTRTX`, `BBOTHS1`, `BBOTDS1`, `BBOTA1`, `BBORDRH`, `GSPROD`, `GSCNTR1`, `BBCAID`, `GSCTWT`, `GSCTUM`, `GSUMCV`, `GSTABL`, `ARCUST`, `ARCUSX`, `BBSHSA`, `BBPRXR`, `GLMAST`, `GSMLCD`, `ARCUPR`, `BICUAX`, `INWZHW`, `INWZ10W`.
- **Programs**:
  - `BB1011`, `BB1014`, `BB1015`, `BB1018`, `BB101F`, `BB106`, `BB1033`, `BB115`, `BB104A`, `BB811`, `AR822R`, `MBBQTY`, `IN805`, `LCSTSHP`, `MSHPADR`, `MGSTABL`.

</xaiArtifact>