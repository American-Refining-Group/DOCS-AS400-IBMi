### List of Use Cases Implemented in the Call Stack

The call stack (BB114P.ocl36 → BB110E → BB114 → BB114SC → BB114S → BB114O → BB1143 → BB110) implements several use cases related to processing and printing customer order confirmations and proforma invoices within a Customer Orders system on an IBM midrange system. Each use case reflects a distinct business function, derived from the program interactions, file manipulations, and output generation. Below are the identified use cases:

1. **Interactive Order Confirmation Inquiry**:
   - **Description**: Allows users to interactively query and view open order headers, filter by criteria (e.g., company, customer, order number, date), and select orders for confirmation processing.
   - **Programs Involved**: BB114P (initiates), BB110E (busy screen), BB114 (main inquiry and subfile).
   - **Key Features**: Dynamic filtering, subfile display, prompting for field values, and selection for further processing (e.g., option 6 for confirmation).

2. **Customer Order Confirmation Generation**:
   - **Description**: Builds work files for a selected order to generate a customer order confirmation, preparing data for printing or further processing.
   - **Programs Involved**: BB114 (triggers via option 6), BB114SC (file overrides), BB114S (builds work files).
   - **Key Features**: Segregates order records (header, detail, miscellaneous) into work files (BBPROH, BBPROD, etc.) based on sequence numbers.

3. **Proforma Invoice Printing**:
   - **Description**: Processes order data, sorts it by specified criteria, and prints proforma invoices (with Spoolflex conversion to PDF, no emailing).
   - **Programs Involved**: BB114P (initiates), BB114O (sorting and printing), BB1143 (preprocessing sort fields), BB110 (main printing).
   - **Key Features**: Sorting by company/order/sequence, specialized grouping for customer-owned products or packaging plants, and output to LIST4 printer file with post-processing (splitting, PDF conversion).

4. **Picking Ticket Printing for Customer-Owned Products (e.g., Viscosity EDI856)**:
   - **Description**: Generates picking tickets tailored for customer-owned products, with specific sorting and formatting (e.g., by product or size/container).
   - **Programs Involved**: BB114O (initiates), BB1143 (sort preprocessing), BB110 (prints to LIST or LIST2).
   - **Key Features**: Custom sorting (product or size-based), adjusted layouts to avoid off-page printing, and Viscosity-specific formatting.

5. **Picking Ticket Printing for Packaging Plant (PP/PKGP)**:
   - **Description**: Produces picking tickets for packaging plant orders, with unique sorting and product descriptions.
   - **Programs Involved**: BB114O, BB1143, BB110 (prints to LIST2).
   - **Key Features**: Sorting by product or size/container, right-justified brand, and packaging plant-specific descriptions.

6. **Order Confirmation Printing**:
   - **Description**: Prints order confirmations for manual selection, including detailed order information and notes.
   - **Programs Involved**: BB114O, BB1143, BB110 (prints to LIST3).
   - **Key Features**: Outputs confirmations with status (e.g., on-hold), weights, freight, and export compliance notes.

7. **Spool File Post-Processing for Proforma Invoices**:
   - **Description**: Splits, converts, and archives spool files for proforma invoices into PDF-like electronic documents, with specific naming and output queue handling.
   - **Programs Involved**: BB114O (calls SFASPLIT, FFAEDOC, SFACOPY).
   - **Key Features**: No emailing, moves output to designated queues (e.g., PROFOROQSV), and uses standardized naming.

### Function Requirement Document: Proforma Invoice Generation

<xaiArtifact artifact_id="fce7e3d9-e732-4a46-b547-5ca28ba1e9c6" artifact_version_id="9fbecb16-af2c-49c8-9e5c-cae4e20b672e" title="Proforma_Invoice_Generation_Requirements.md" contentType="text/markdown">

# Proforma Invoice Generation Function Requirements

## Purpose
Generate proforma invoices for customer orders, producing formatted spool files (converted to PDF) without emailing, based on input order data, with sorting, grouping, and detailed calculations for printing.

## Inputs
- **Company Number** (2-digit decimal): Identifies the company.
- **Order Number** (6-digit decimal): Specifies the order to process.
- **File Group** (1-character, e.g., 'G' or 'Z'): Determines library/file set (e.g., GBBPROH vs. ZBBPROH).
- **Order Data Files**: Pre-populated files (e.g., BBORDR, BBPROH, BBPROD) containing order headers, details, and miscellaneous records.
- **Supporting Files**: Customer (ARCUST), product (GSPROD), container (GSCNTR), and other reference data.

## Outputs
- **Spool File (LIST4)**: Formatted proforma invoice output, routed to PROFOROUTQ or TESTOUTQ, with form type PROF, CPI 15.
- **PDF-like Electronic Document**: Converted via Spoolflex, named 'ARG PROFORMA INVOICE', archived in PROFOROQSV.
- **Work Files**: Populated BBPROH (headers), BBPROD (details), BBPROO/BBPROI/BBPROB (miscellaneous), BBPROM (remarks).

## Process Steps
1. **Validate Input Data**:
   - Check if BBPROH exists and has records for the company/order; exit if empty.
2. **Prepare Work Files**:
   - Read BBORDR records for the specified company/order.
   - Segregate by sequence number:
     - Seq# 000 → BBPROH (header).
     - Seq# 900-959 → BBPROM (remarks).
     - Seq# 960-962 → BBPROO/BBPROI/BBPROB (miscellaneous).
     - Others → BBPROD (details).
   - Write records verbatim to respective files.
3. **Sort and Preprocess Data**:
   - Sort BBPROR by company, order, sequence number (header first, then marks/details).
   - Create temporary file (QTEMP/BB1143, 554 bytes) for preprocessing.
   - Assign sort fields for details based on:
     - Customer-owned (BOCOON = 'Y'): Group by product (BDPROD) or size (from GSCNTR); sort by product, container, or size/product.
     - Packaging plant (BORACD = 'PP', BOMLCD = 'PKGP'): Similar to customer-owned.
     - Other: Group by product or container; sort by product/container or container/product.
   - Output extended records with sort fields (513-553) and BOORPR (554).
   - Sort again by company, order, header priority, custom sort fields (from BB1101), sequence number.
4. **Generate Print Output**:
   - Read sorted data (BB1143S) and reference files (e.g., ARCUST, GSPROD, BBCAID).
   - Skip orders with delete code 'E' (error).
   - Output to LIST4:
     - Header: Order#, customer, ship-to, incoterms, route code, order status.
     - Details: Quantity, gross weight (from BB101), product descriptions (GSPROD), container types (GSCNTR1), prices (ARCUPR).
     - Freight: Description, bill-to address (internal: customer; external: processor).
     - Totals: Net quantity, gross/net weight (in LBS), price total, miscellaneous charges.
     - Notes: ARG terms, safety instructions, freight estimates, export compliance.
5. **Post-Process Spool File**:
   - Split LIST4 spool (named 'PROFORMA INV LIST4') in PROFOROUTQ.
   - Convert to e-doc 'ARG PROFORMA INVOICE' via Spoolflex, move to PROFOROQSV.
   - Copy spool with name 'PROFORMA NAMING EMAL' to PROFOROQSV.

## Business Rules
- **Order Eligibility**: Process only non-deleted orders (BODEL ≠ 'D'); skip if BOORPR = 'E' (error). Mark BOORPR = 'H' as 'ON HOLD - DO NOT SHIP'.
- **Sorting/Grouping**:
  - Customer-owned (Viscosity): Group by product (BOGPBY = 'C') or size; sort by product/container or size/product/container.
  - Packaging plant: Similar to customer-owned.
  - Other: Group by product or container; sort by product or container/product.
  - BOORPR = 'C', 'S', 'R' uses sorted order; blank uses line entry order.
- **Calculations**:
  - Gross weight: From BB101 (order entry).
  - Net weight: Includes product weight.
  - Extended price: Quantity × unit price (from ARCUPR, container-specific).
  - Totals: Sum net quantity, misc charges, price total.
- **Freight Handling**:
  - Internal processor: Use customer address for freight bill.
  - External processor: Use freight processor address.
  - Suppress freight total amount printing.
- **Formatting**:
  - Use CPI 15, form type PROF for LIST4.
  - Right-justify brand, adjust columns to avoid off-page printing.
  - Include incoterms, route code, loading location, and export compliance notes.
- **File Groups**: Use file group ('G' or 'Z') to select library (e.g., GBBPROH vs. ZBBPROH).
- **Post-Processing**: No emailing; archive spool files in PROFOROQSV with standardized names.

## Calculations
- **Net Quantity (L2CQT)**: Sum of BDQTY across detail lines (non-deleted).
- **Gross Weight (L2GWT)**: Pre-calculated in BB101, output in LBS.
- **Net Weight (L2NWT)**: Sum of product weights (BDGWT) from detail lines.
- **Price Total (TOTPRCJB)**: Sum of extended prices (BDQTY × unit price from ARCUPR).
- **Miscellaneous Total (L2TOT)**: Sum of BMAMT from miscellaneous lines (Seq# 960-962, 900-959).

</xaiArtifact>