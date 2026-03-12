### List of Use Cases

Based on the analyzed programs and their interactions in the call stack, the following use cases are implemented. Each represents a distinct business process handled by one or more programs, focusing on order entry and posting in a legacy system.

1. **Order Batch Selection and Management**: Allows users to select, create, view, update, or delete order batches before processing, ensuring batches are locked/unlocked appropriately (implemented in BB001).

2. **Credit Authorization for Orders**: Sorts and validates orders for credit holds, calculates totals, updates statuses, and generates authorization reports (implemented in BB198 with #GSORT).

3. **Order Posting and Updates**: Processes transaction records to add/update/delete order headers, details, marks, taxes, and supplementals in open order files, with history logging and freight/tax calculations (implemented in BB201).

4. **Archiving Cancelled Orders**: Checks and archives cancelled open orders to history files, optionally deleting from open files (implemented in BB104B).

5. **Notifying for Orders on Credit Hold**: Identifies newly held orders and generates notifications (spool files for email) with details like amounts, aging, and contacts (implemented in BB202).

6. **Unlocking Open Orders**: Removes lockout codes and workstation IDs from order headers post-processing (implemented in BB215).

7. **Releasing or Posting Batch**: Updates batch control records to release locks or mark as posted/deleted after processing (implemented in BB005).

8. **Updating Order History**: Maintains logs of order line items for duplicate prevention, handling adds/updates/deletes/reactivations with timestamps and messages (implemented in BB117).

### Function Requirement Document

This document outlines requirements for each use case as a single, non-interactive function. Each function accepts inputs (e.g., batch ID, order data) to perform the process end-to-end. Focus is on business requirements: data handling, validations, updates, and calculations (e.g., totals, taxes, freight). Outputs include success/failure indicators, updated data, or reports. Processes are concise, assuming batch/transaction data as inputs; no user screens.

#### 1. Function: ManageOrderBatch
**Purpose**: Select/create/update/delete order batches for processing.

**Inputs**:
- Batch ID (numeric, optional for new).
- Mode (string: 'select', 'create', 'update', 'delete').
- Source code (string, e.g., 'PP' for Viscosity ASN).
- User ID (string).
- Workstation ID (string).
- Lock status (string, optional for update).

**Process Steps**:
1. If mode='select': Retrieve batch by ID; return details if exists and matches source.
2. If mode='create': Generate new batch ID; insert record with defaults (lock='X', created date=CYMD, record count=0, source=input).
3. If mode='update': Retrieve by ID; update lock/WSID/user/date if not locked.
4. If mode='delete': Retrieve by ID; set delete flag='D' if not locked.

**Business Rules**:
- Batches must match source code (e.g., 'PP' skips mismatches).
- Locked batches (status: 'O'=order entry, 'P'=posting) cannot be updated/deleted/selected for post.
- Soft delete only ('D' flag); skip deleted in selections.
- Record count updated on create/update (initial 0).
- No calculations; timestamps use current CYMD.

**Outputs**: Batch details JSON (ID, status, dates, count); success/failure message.

#### 2. Function: AuthorizeOrderCredit
**Purpose**: Validate orders for credit, calculate totals, update statuses, generate report.

**Inputs**:
- Sorted transaction batch (array of records: headers/details with company, order#, customer, products, quantities, prices).
- Customer master data (array: limits, aging balances).

**Process Steps**:
1. Process headers: Skip deleted; retrieve terms/ship-to details.
2. For each detail: Skip deleted/misc (>899 seq); negate qty for credits ('R' type); convert qty using UOM factors; calculate line total (converted qty * price).
3. Accumulate order/company totals; update status in customer order file ('AO'/'AB' for active/backorder, 'RO'/'RB' for revised, 'CN' for cancel).
4. Generate report with orders, values, statuses.

**Business Rules**:
- Credit types ('R') negate quantities for returns.
- No-charge items zero totals.
- Status: 'Active' for new, 'Revised' for existing; append 'Backorder' if shipment#=1; 'Cancelled' overrides.
- Descriptions prioritize custom/alternate/standard from product master.
- Totals exclude no-charge/misc; use sorted input (headers first).
- Calculations: Qty conversion (multiply/divide by factor from units table); line total = converted qty * price.

**Outputs**: Authorization report (orders, totals, statuses); updated statuses; success/failure.

#### 3. Function: PostOrderUpdates
**Purpose**: Post transactions to open orders: add/update/delete headers/details/marks/taxes.

**Inputs**:
- Transaction batch (array: headers/details/marks/taxes with delete flags, quantities, prices, freight/tax codes).
- Product/customer masters (for validations/conversions).

**Process Steps**:
1. Process headers: Add/update (dates/terms/freight); calculate totals (product/misc/freight); delete if flagged.
2. Details: Add/update (qty/price/taxes/weights); convert UOM/gallons; delete if 'D' (call history delete).
3. Marks/misc/taxes: Add/update/delete (remarks, charges, overrides).
4. If no freight: Delete specific misc records (seq 943/950).
5. Update statuses; log history changes.

**Business Rules**:
- Delete only on 'D' flag; negate for credits ('R').
- Freight codes: 'C'=collect, 'P'=prepaid, 'A'=allowance; no calc ('N') deletes freight misc.
- Taxes: Up to 10 codes; overrides store exemptions/amounts; accumulate in arrays.
- Quantities: Convert IMS/UOM; adjust gross/net gallons (temp/gravity).
- History: Log all changes with timestamps/user; delete history on line delete.
- Calculations: Line total = qty * price; tax = totals * rates (overrides apply); freight per code/terms.

**Outputs**: Updated open orders; history logs; success/failure.

#### 4. Function: ArchiveCancelledOrders
**Purpose**: Archive and optionally delete cancelled open orders.

**Inputs**:
- Transaction batch (array: records with cancel flags).
- File group ('G'/'Z').
- Delete flag ('0' for delete).

**Process Steps**:
1. For each header: If cancelled in open but not transaction delete, archive all related records (headers/details/marks/taxes/supplementals) to history files.
2. Copy records verbatim; set archive active ('A').
3. If delete flag='0': Delete from open files (headers/details/marks/taxes/supplementals).

**Business Rules**:
- Archive trigger: Exists in open, cancelled ('Y'), not transaction delete.
- Copy all related records (e.g., taxes via SETLL/READ).
- Hard deletes from open; archives as active ('A').
- Exclusive access via overrides (SHARE(*NO)).
- No calculations; key-based matching (company + order#).

**Outputs**: Success/failure; archived count.

#### 5. Function: NotifyCreditHoldOrders
**Purpose**: Generate notifications for newly held orders.

**Inputs**:
- Transaction headers (array: with hold status, amounts, dates, contacts).
- Customer data (aging, limits).

**Process Steps**:
1. Skip deleted headers.
2. Calculate order amount (prefer open file; fallback transaction).
3. If hold changed to 'Y' and unauthorized: Retrieve emails (CSR/salesman/A/R); calculate days to request date.
4. Generate dual reports (credit/salesman) with details (order/customer, amounts, aging, notifies).

**Business Rules**:
- Notify only on new holds (status change) and unauthorized.
- Amount: Open file if non-zero; else transaction sum.
- Aging buckets: Current, 1-30, 31-60, etc., from customer master.
- Emails: Lookup from tables; defaults (e.g., CSR='SJM'); fixed notifies (e.g., bzolkos@amref.com).
- Dates: CYMD; days = request - current.
- No calculations beyond amount/days.

**Outputs**: Spool reports for email; success/failure.

#### 6. Function: UnlockOpenOrders
**Purpose**: Clear locks from order headers post-processing.

**Inputs**:
- Transaction batch (array: headers with keys).

**Process Steps**:
1. For each header: Retrieve open header by key (company + order#).
2. If found: Clear lock code (blank) and WSID (blanks); update record.

**Business Rules**:
- Only headers processed; skip non-matches.
- Clears specific fields (lock position 155, WSID 156-157).
- No validations; assumes post-processing context.
- No calculations.

**Outputs**: Success/failure; unlocked count.

#### 7. Function: ReleaseOrPostBatch
**Purpose**: Update batch after processing (release/post/delete).

**Inputs**:
- Batch ID (numeric).
- Mode (string: 'O'/'L'/'B'/'P').
- Record count (numeric).

**Process Steps**:
1. Retrieve batch by ID.
2. If found: Update based on mode (clear locks/WSID, set printed 'Y' for 'B', set 'D'/'P' for 'P'); always update count.

**Business Rules**:
- Modes: 'O'/'L'/default=clear locks; 'B'=clear + printed; 'P'=delete + posted.
- Update positions: 6 (lock=' '/'P'), 8 (WSID='  '), 9 (printed='Y'), 1 (delete='D'), 39 (count).
- Skip if not found.
- No calculations.

**Outputs**: Success/failure.

#### 8. Function: UpdateOrderHistory
**Purpose**: Log order line changes for duplicate checks.

**Inputs**:
- Line details (company, order#, seq#, customer, ship-to, PO, date, product, container, qty, batch).
- Mode ('UPD'/'DEL'/'ACT').
- Flags (duplicate override 'Y', file group 'G'/'Z').

**Process Steps**:
1. For 'UPD': Add/update main history (timestamps/user); if override, add/update message in supplemental.
2. For 'DEL': Delete main/supplemental (all for order if seq=0; line-specific otherwise).
3. For 'ACT': Update status to 'A' and dates in main/supplemental.

**Business Rules**:
- 'UPD': Add if new; update timestamps if exists. Message: 'Possible Duplicates Accepted' on override.
- 'DEL': Hard delete by key (company + order# + seq# + dates for messages).
- 'ACT': Set active ('A'); update dates.
- Timestamps: Current CYMD/user.
- Exclusive overrides for environments.
- No calculations; key-based.

**Outputs**: Success/failure.