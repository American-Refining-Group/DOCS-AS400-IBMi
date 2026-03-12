### List of Use Cases

Based on the entire call stack of programs (from the main OCL prompt through printing, freight integration, and Spoolflex), the following distinct use cases are implemented. These are derived from the interactive inquiry, batch selection/printing, preprocessing/sorting, freight handling, and distribution workflows. Note that some programs support multiple use cases via conditional logic (e.g., modes for picks vs. confirmations).

1. **Order Confirmation Inquiry and Selection**: Interactive search and viewing of open orders with filters (e.g., by company, customer, date range), allowing users to select and manually trigger confirmation sending for a single order. This builds work files for printing.

2. **Batch Order Confirmation Printing**: Automated batch processing of selected orders to generate and print order confirmations, including data sorting, validation, and output to printer queues with overrides for forms/CPI.

3. **Picking Ticket Printing**: Generation of picking tickets (standard, viscosity/EDI, or packaging plant versions) with custom sorting/grouping of details (e.g., by category/size/product), calculations for weights/quantities, and printing with special formats.

4. **Proforma Invoice Printing**: Printing of proforma invoices as an alternate output mode during the core printing process, focusing on order totals, freight estimates, and terms.

5. **Freight Purchase Order Generation and Processing**: Creation of freight-related PO files during printing, with calculations for conversions (e.g., qty to lbs/gallons), address handling, and integration calls to internal/external freight processors.

6. **Spool File Distribution and Archiving**: Post-printing handling of spool files for order confirmations, including PDF conversion, emailing to multiple recipients, saving to network folders, and error retries for locked files.

7. **Process Monitoring and Cancellation**: Display of a busy/checking screen during long-running processes, allowing user interruption/cancellation of active batch jobs.

### Function Requirement Document

This document treats each use case as a large, non-interactive function that accepts inputs (e.g., parameters like order numbers, filters, modes) and produces outputs (e.g., printed documents, files, or status). Requirements focus on business processes and rules, with concise explanations of steps and calculations. Inputs are assumed to be provided directly (e.g., via API/params), replacing screen interactions. Outputs include generated files, prints, or return codes. Business rules emphasize order eligibility, data integrity, and compliance (e.g., export terms, freight fluctuations).

#### 1. Function: OrderConfirmationInquirySelection
**Description**: Searches and selects open orders for confirmation, building work files for a single order's confirmation data.

**Inputs**:
- Company number (decimal, default 10).
- Selection filters: Order#, PO#, customer#, ship-to#, salesman#, location, product, carrier ID, country, date range (requested date, default 1 year ago to current).
- File group ('G' for production, 'Z' for test/archive).
- Order# for manual send.

**Process Steps**:
1. Apply filters to query open order headers/details (exclude deleted 'D', include status 'C'/'R' for confirmation/reprint).
2. Position/load subfile with matching records (e.g., chain to customer/ship-to/product files for names/addresses/descriptions).
3. For selected order (option 6), override files by group, read order records from BBORDR, segregate by sequence#: Header (000) to BBOCFH, details (001-899/963+) to BBOCFD, misc (900-959 to BBOCFM, 960 to BBOCFO, 961 to BBOCFI, 962 to BBOCFB).
4. Return status (e.g., "Confirmation sent") and work files for printing.

**Business Rules and Calculations**:
- Must enter at least one filter; else error "Must Enter At Least One Selection Value".
- Restrict searches: No tracking# if excluding invoices; no cancel reason without including canceled orders.
- Validate existence via chains (e.g., customer in ARCUST); highlight over-credit (authorized/unauthorized).
- Date ranges: Requested date >=1 year ago default; convert to CYMD format.
- No calculations; direct data copy to work files for integrity.

#### 2. Function: BatchOrderConfirmationPrinting
**Description**: Batch prints confirmations for selected orders, with sorting and preprocessing.

**Inputs**:
- Library prefix (e.g., 'G' for production).
- Mode ('PP' for pick print, else confirmation).
- Switch flags (e.g., bit 6 for Spoolflex).

**Process Steps**:
1. Check input file BBOCFH for records; skip if empty.
2. Sort BBOCFR records: Exclude deleted, prioritize headers > marks > details by company/order/seq/record type.
3. Preprocess details: Group/sort by type (customer-owned, ARG PKG plant, other) using custom keys (SORT1 asc, SORT2 desc, SORT3 asc); extend records to 554 bytes.
4. Load/print: Chain lookups for desc/addr/weights, accumulate totals, print headers/details/totals/notes to LIST3 (confirmations).
5. If production and switch on, Spoolflex: Split/convert to PDF, distribute/email, archive to save queue.
6. Return print status/files.

**Business Rules and Calculations**:
- Skip if no records; exclude deleted from totals.
- Sorting: Headers first (marker '1'), details '2', others '3'; custom groups (e.g., by product/container for PKG).
- Printer overrides: CPI 15, form ORCF, queue ORDCONOUTQ (test: TESTOUTQ).
- Spoolflex: Retry on open PDF; distribute to 3 points; error if no files.
- No major calcs; defer to printing function.

#### 3. Function: PickingTicketPrinting
**Description**: Prints picking tickets with custom formats/sorting for warehouse efficiency.

**Inputs**:
- Same as BatchOrderConfirmationPrinting, plus mode flags (e.g., viscosity 'Y', PKG plant).

**Process Steps**:
1. Same as BatchOrderConfirmationPrinting steps 1-3 (sorting/preprocess with BB1103 for groups).
2. Print to LIST/LIST2: Headers (company/customer/ship-to/order info), details (loc/prod/cntr/qty, grouped with lines), totals (qty/weight/misc/price/freight).
3. Include special formats (rearrange for viscosity, right-justify brand for PKG).
4. Call freight processing if needed.
5. Spoolflex if enabled.

**Business Rules and Calculations**:
- Sorting modes: Unsorted if status blank (entry order); sorted if 'C'/'S' (by group/sort keys, e.g., product asc/container desc for customer-owned).
- Conversions: Container size from GSCNTR desc if not 'N'; weights via GSCTWT/GSCTUM (lbs/gallons, revised formula).
- Exclude deleted from totals; print "ON HOLD - DO NOT SHIP" for 'H'.
- Formats: Viscosity (match Lotus, no off-edge); PKG (custom desc, part# heading).
- Printer: CPI 10, form PICK/PKGP, queue PICKNWOUTQ.

#### 4. Function: ProformaInvoicePrinting
**Description**: Prints proforma invoices focusing on totals/terms for export/pre-payment.

**Inputs**:
- Same as PickingTicketPrinting, with mode for proforma.

**Process Steps**:
1. Reuse printing core from steps above.
2. Print expanded: Order totals (product/misc/freight), incoterms, export notes/compliance.
3. Include bill-to/ship-to addresses, estimates.

**Business Rules and Calculations**:
- Prices at shipment time; freight estimates fluctuate with diesel index.
- Export rules: Comply with U.S. regs, no diversion; print MSDS link.
- Totals: Accumulate extensions (qty * price), add misc/freight; net wt includes product wt.
- Terms: Standard ARG terms, PPE/driver reqs for bulk.

#### 5. Function: FreightPurchaseOrderGeneration
**Description**: Generates freight POs and integrates with processors during printing.

**Inputs**:
- Order data (company/order#), file group, internal/external flag, bill-to address.

**Process Steps**:
1. Read order records; chain for addresses (SHIPTO/ARCUST), shipment details (BBSHP*).
2. For headers: Copy fields (rush/process code), set bill-to (customer for internal, processor for external).
3. For details: Convert qty to UM/lbs/gallons/net/gross; accumulate accessorials/totals.
4. Write to BBFPORH (header), BBFPORD (details), BBFPORA (accessorials), BBFPORF (freight, copy from BBORF).
5. Call PC program: Internal (InsertARGLMSOrder.EXE), external (SCOXML.EXE) via STRPCCMD.
6. Return PO files/status.

**Business Rules and Calculations**:
- Internal: Send deleted orders; don't use processor addr.
- External: Use processor addr; email/FTP params.
- Skip deleted details but send full deleted orders internally.
- Calcs: Qty conversions (container to UM/lbs/gallons via lookups/formulas); totals (freight/accessorial/gross gallons).
- Addresses: Compressed, include country; multiple tables notice via Spoolflex.

#### 6. Function: SpoolFileDistributionArchiving
**Description**: Handles post-print spool files for distribution/archiving.

**Inputs**:
- Output queue (e.g., ORDCONOUTQ), spool name ('ORDER CONF LIST3').

**Process Steps**:
1. Split/convert spool to PDF ('order confirmation').
2. Copy for naming/email.
3. Distribute to 3 delivery points (e.g., email).
4. Move to save queue (ORDCONSVOQ).
5. Retry on errors (e.g., open PDF prompt close/retry).
6. Return status (e.g., no files).

**Business Rules and Calculations**:
- Only process named spools; error if none ("No order confirmations selected").
- Retry indefinitely on locked PDFs; distribute to predefined points.
- No calcs; focus on archiving to network (G:\CUSTSRV\ORDER CONFIRMATIONS).

#### 7. Function: ProcessMonitoringCancellation
**Description**: Monitors active processes and allows cancellation.

**Inputs**:
- Active process name (e.g., BB111P/BB111).

**Process Steps**:
1. Load/run check program (BB110E) to display busy screen.
2. Wait for user input (cancel key).
3. If cancel, set flag and end; else continue.
4. Return status (canceled or proceeded).

**Business Rules and Calculations**:
- Allow interrupt for long runs (e.g., batch print).
- Check screen location for "/CANCEL"; reset on termination.
- No calcs; UI-focused but input-driven here.