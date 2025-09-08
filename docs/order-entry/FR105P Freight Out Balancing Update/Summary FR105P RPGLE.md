### List of Use Cases Implemented in the FR105P and FR105 Call Stack

The RPG programs `FR105P` and `FR105` together form a call stack within a Traffic/Freight Invoicing system, designed for freight out bill balancing. They implement several distinct use cases for managing freight invoice records. Below are the identified use cases based on the functionality of both programs:

1. **View and Filter Freight Invoice Headers (FR105P)**:
   - **Description**: Allows users to view and filter freight invoice headers in a subfile, with options to include/exclude deleted records, filter by company, order number, customer, ship-to, invoice number, carrier, and open/closed status.
   - **Functionality**:
     - Displays invoice headers in a subfile (`sfl1`) with fields like company, order number, customer, ship-to, invoice number, carrier, and status.
     - Supports filtering (e.g., exclude deleted records, open/closed invoices) and repositioning based on user input.
     - Provides navigation via function keys (F03: Exit, F04: Prompt, F05: Refresh, F08: Toggle deleted records, F10: Reset cursor, PAGEDN: Load more records).
   - **Interaction**: Interactive via display file (`fr105pd`), with subfile control (`sflctl1`) for user input.

2. **Create Freight Invoice Detail (FR105P calling FR105)**:
   - **Description**: Enables users to create or modify a carrier invoice detail for a selected freight invoice header by calling `FR105`.
   - **Functionality**:
     - From `FR105P`, option `1` on a subfile record triggers `FR105` to process the selected invoice (company, order, shipping reference).
     - `FR105` allows adding or updating carrier invoice details, validating inputs, and calculating totals.
     - Returns a success flag to `FR105P` for confirmation messaging.
   - **Interaction**: `FR105P` passes parameters to `FR105`, which uses a display file (`fr105d`) with subfiles (`sfl1`, `sfl1b`) for detailed processing.

3. **View Order Detail and Miscellaneous Charges (FR105)**:
   - **Description**: Displays order detail and miscellaneous charge records associated with a specific freight invoice in a subfile.
   - **Functionality**:
     - Loads sales history (`sa5shy`), invoice details (`sa5fjxd`), and miscellaneous records (`sa5fjxm`) into `sfl1b`.
     - Filters records by order and shipping reference, excluding specific miscellaneous codes (e.g., `samscd = 'F'`).
     - Displays formatted details like product, description, and freight amounts.
   - **Interaction**: Interactive via `sfl1b` in the display file (`fr105d`).

4. **Manage Carrier Invoice Details (FR105)**:
   - **Description**: Allows users to add, update, or delete carrier invoice details, including carrier ID and invoice dates.
   - **Functionality**:
     - Supports adding a new carrier invoice record (F06), updating invoice dates (F09), and deleting paper invoices (F23) under specific conditions.
     - Validates carrier IDs against `bbcaid` and ensures linkage to vendors (`apveny`).
     - Updates database files (`frcinh`, `frcfbh`, `frcind`) in maintenance mode.
   - **Interaction**: Interactive via `sfl1` in the display file (`fr105d`).

5. **Calculate Freight Totals and Differences (FR105)**:
   - **Description**: Computes invoice totals, carrier freight totals, and the difference between expected and actual freight amounts for a specific order.
   - **Functionality**:
     - Calculates invoice total (`f$itot`) from detail and miscellaneous records.
     - Calculates carrier freight total (`f$cftl`) from carrier invoice headers, adjusting for overrides.
     - Computes freight difference (`f$diff`) for balancing.
   - **Interaction**: Non-interactive calculation triggered during data retrieval.

---

### Function Requirement Document: Freight Out Bill Balancing Function

**Function Name**: `FreightOutBillBalancing`

**Purpose**: To process freight invoice headers and carrier invoice details for a specified company, order, and shipping reference, calculating invoice totals, carrier freight totals, and differences, while applying business rules for validation and filtering.

**Inputs**:
- `company` (Numeric, 2 digits): Company number.
- `orderNumber` (Numeric, 6 digits): Customer order number.
- `shippingReference` (Numeric, 3 digits): Shipping reference number.
- `fileGroup` (Character, 1): File group (`Z` or `G`) for database library selection.
- `mode` (Character, 3): Processing mode (`MNT` for maintenance, `INQ` for inquiry).
- `filters` (Structure):
  - `includeDeleted` (Boolean): Include or exclude deleted records.
  - `customer` (Numeric, 6 digits, optional): Filter by customer number.
  - `shipTo` (Numeric, 3 digits, optional): Filter by ship-to number.
  - `invoiceNumber` (Numeric, 6 digits, optional): Filter by invoice number.
  - `carrier` (Character, 10, optional): Filter by carrier ID.
  - `status` (Character, 1, optional): Filter by open (`O`) or closed (`C`) status.

**Outputs**:
- `freightInvoiceHeaders` (Array of Structures): List of filtered freight invoice headers with fields: company, orderNumber, shippingReference, customerNumber, customerName, shipToNumber, shipToName, invoiceNumber, invoiceDate, shipDate, carrierID, carrierName, freightCode, status, deletedFlag.
- `carrierInvoiceDetails` (Array of Structures): List of carrier invoice details for the selected order with fields: company, carrierID, carrierName, carrierInvoiceNumber, invoiceAmount, invoiceType, apStatus.
- `orderDetails` (Array of Structures): List of order details and miscellaneous charges with fields: type, product, description, freightAmount.
- `totals` (Structure):
  - `invoiceTotal` (Numeric, 9.2): Total freight amount from order details and miscellaneous charges.
  - `carrierFreightTotal` (Numeric, 9.2): Total carrier freight amount.
  - `freightDifference` (Numeric, 9.2): Difference between carrier freight and expected/actual freight.
- `successFlag` (Boolean): Indicates successful processing.
- `errorMessages` (Array of Strings): List of validation or processing errors.

**Process Steps**:
1. **Validate Inputs**:
   - Verify `company`, `orderNumber`, and `shippingReference` exist in `frbinh`.
   - Validate `fileGroup` is `Z` or `G`.
   - Validate `mode` is `MNT` or `INQ`.
   - Return error if any input is invalid.

2. **Apply File Overrides**:
   - Use `fileGroup` to override database files (`frbinh`, `frbinf`, `frcinh`, `frcfbh`, `frcind`, `frcinhj1`, `bbcaid`, `sa5shy`, `sa5fjxd`, `sa5fjxm`, `apveny`) to the appropriate library (`g*` or `z*`).

3. **Retrieve Freight Invoice Headers**:
   - Query `frbinh` and `frbinf` for headers matching `company`, `orderNumber`, and optional filters (`customer`, `shipTo`, `invoiceNumber`, `carrier`, `status`).
   - Exclude deleted records (`bodel = 'D'`) unless `includeDeleted` is true.
   - Exclude customer-owned products (`bocoon = 'Y'`).
   - Exclude collect freight records (`bofrcd = 'C'`) with no calculated freight (`bocafr = 'N'`).
   - Retrieve customer name from `arcust` and ship-to name from `shipto`.

4. **Retrieve Order Details and Miscellaneous Charges**:
   - Query `sa5shy`, `sa5fjxd`, and `sa5fjxm` for records matching `company`, `orderNumber`, and `shippingReference`.
   - Filter miscellaneous records to include only freight-related records (`samsty = 'F'`, `samscd <> 'F'`).
   - Format details with type, product, description, and freight amount.

5. **Retrieve Carrier Invoice Details**:
   - Query `frcinhj1` for carrier invoice headers matching `company`, `orderNumber`, and `shippingReference`.
   - Exclude deleted records (`frdel = 'D'`).
   - Validate carrier IDs against `bbcaid` and ensure linkage to vendors in `apveny`.
   - Retrieve carrier name (`cicanm`) from `bbcaid`.

6. **Calculate Totals**:
   - **Invoice Total (`invoiceTotal`)**:
     - For detail records (`sa5fjxd`, `samscd = *blanks`, `sasfrt <> 'Y'`): `freightAmount = saqty * safrrt`.
     - For miscellaneous records (`sa5fjxm`, `samsty = 'F'`, `samscd <> *blanks`): `freightAmount = samqty * samamt`.
     - Sum all `freightAmount` values.
   - **Carrier Freight Total (`carrierFreightTotal`)**:
     - Sum `frinam` from `frcinhj1` where `frdel <> 'D'`, subtracting `frfboa` (freight balancing override).
   - **Freight Difference (`freightDifference`)**:
     - If `cftfam` (expected freight) is non-zero, calculate `carrierFreightTotal - cftfam`.
     - Otherwise, calculate `carrierFreightTotal - bftfam` (actual freight).

7. **Process Carrier Invoice Updates (Maintenance Mode Only)**:
   - If `mode = 'MNT'`, allow adding new carrier invoice records, updating invoice dates, or deleting paper invoices (`s1inty = 'P'`, `s1apst = *blanks`).
   - Validate carrier IDs and update `frcinh`, `frcfbh`, and `frcind` accordingly.

8. **Return Results**:
   - Populate `freightInvoiceHeaders`, `carrierInvoiceDetails`, `orderDetails`, and `totals`.
   - Set `successFlag = true` if no errors, else include `errorMessages`.

**Business Rules**:
- **Validation**:
  - Carrier IDs must exist in `bbcaid` and be linked to a vendor in `apveny`.
  - Only paper invoices (`s1inty = 'P'`) with no A/P status (`s1apst = *blanks`) can be deleted.
  - Dates must be valid and within or after the shipped month.
- **Filtering**:
  - Exclude deleted records unless explicitly included.
  - Exclude customer-owned products (`bocoon = 'Y'`).
  - Exclude collect freight records with no calculated freight.
  - Miscellaneous records must have `samsty = 'F'` and `samscd <> 'F'` for freight inclusion.
- **Calculations**:
  - Invoice total excludes detail records with `sasfrt = 'Y'`.
  - Carrier freight total subtracts freight balancing overrides (`frfboa`).
  - Freight difference uses expected (`cftfam`) or actual (`bftfam`) freight based on availability.
- **Mode Restrictions**:
  - Inquiry mode (`INQ`) restricts updates and deletes.
  - Maintenance mode (`MNT`) allows full CRUD operations on carrier invoices.

**Error Handling**:
- Return specific error messages for invalid carrier IDs (`ERR0010`), unlinked vendors (`ERR0037`), invalid dates (`ERR0003`), and other validation failures.
- Ensure all database operations are atomic and rolled back on failure.

<xaiArtifact artifact_id="cc24b70d-28cc-4726-b440-9827c9681a9e" artifact_version_id="0fc7cb91-906c-44a0-a057-9f2f8ace6deb" title="FreightOutBillBalancing_Requirements.md" contentType="text/markdown">

# Freight Out Bill Balancing Function Requirements

## Function Name
`FreightOutBillBalancing`

## Purpose
Process freight invoice headers and carrier invoice details, calculating totals and differences for a specified company, order, and shipping reference.

## Inputs
- `company` (Numeric, 2): Company number.
- `orderNumber` (Numeric, 6): Customer order number.
- `shippingReference` (Numeric, 3): Shipping reference number.
- `fileGroup` (Character, 1): `Z` or `G` for database library.
- `mode` (Character, 3): `MNT` (maintenance) or `INQ` (inquiry).
- `filters` (Structure):
  - `includeDeleted` (Boolean): Include/exclude deleted records.
  - `customer` (Numeric, 6, optional): Customer number filter.
  - `shipTo` (Numeric, 3, optional): Ship-to number filter.
  - `invoiceNumber` (Numeric, 6, optional): Invoice number filter.
  - `carrier` (Character, 10, optional): Carrier ID filter.
  - `status` (Character, 1, optional): Open (`O`) or closed (`C`) status.

## Outputs
- `freightInvoiceHeaders` (Array): Headers with company, orderNumber, shippingReference, customerNumber, customerName, shipToNumber, shipToName, invoiceNumber, invoiceDate, shipDate, carrierID, carrierName, freightCode, status, deletedFlag.
- `carrierInvoiceDetails` (Array): Details with company, carrierID, carrierName, carrierInvoiceNumber, invoiceAmount, invoiceType, apStatus.
- `orderDetails` (Array): Details with type, product, description, freightAmount.
- `totals` (Structure):
  - `invoiceTotal` (Numeric, 9.2): Total freight from details/miscellaneous.
  - `carrierFreightTotal` (Numeric, 9.2): Total carrier freight.
  - `freightDifference` (Numeric, 9.2): Difference between carrier and expected/actual freight.
- `successFlag` (Boolean): Processing success.
- `errorMessages` (Array): Validation/processing errors.

## Process Steps
1. **Validate Inputs**: Ensure `company`, `orderNumber`, `shippingReference` exist in `frbinh`, `fileGroup` is `Z`/`G`, `mode` is `MNT`/`INQ`.
2. **Apply File Overrides**: Override files to `g*` or `z*` based on `fileGroup`.
3. **Retrieve Freight Invoice Headers**: Query `frbinh`/`frbinf` with filters, exclude deleted/customer-owned/collect records, retrieve names from `arcust`/`shipto`.
4. **Retrieve Order Details/Miscellaneous**: Query `sa5shy`, `sa5fjxd`, `sa5fjxm` for matching records, filter miscellaneous by `samsty = 'F'`, `samscd <> 'F'`.
5. **Retrieve Carrier Invoice Details**: Query `frcinhj1`, exclude deleted records, validate carrier IDs against `bbcaid`/`apveny`.
6. **Calculate Totals**:
   - `invoiceTotal`: Sum `saqty * safrrt` (details, `sasfrt <> 'Y'`) and `samqty * samamt` (miscellaneous, `samsty = 'F'`).
   - `carrierFreightTotal`: Sum `frinam - frfboa` from `frcinhj1` where `frdel <> 'D'`.
   - `freightDifference`: `carrierFreightTotal - cftfam` (if non-zero) or `carrierFreightTotal - bftfam`.
7. **Process Carrier Invoice Updates**: In `MNT` mode, allow add/update/delete of carrier invoices, validate inputs, update `frcinh`, `frcfbh`, `frcind`.
8. **Return Results**: Populate outputs, set `successFlag`, include any `errorMessages`.

## Business Rules
- **Validation**: Carrier IDs must exist in `bbcaid` and link to `apveny`. Only paper invoices (`s1inty = 'P'`, `s1apst = *blanks`) can be deleted. Dates must be valid and within/after shipped month.
- **Filtering**: Exclude deleted (`bodel = 'D'`), customer-owned (`bocoon = 'Y'`), collect freight (`bofrcd = 'C'`, `bocafr = 'N'`) unless specified. Miscellaneous records require `samsty = 'F'`, `samscd <> 'F'`.
- **Calculations**: Exclude `sasfrt = 'Y'` from invoice total. Subtract `frfboa` from carrier freight. Use `cftfam` or `bftfam` for difference calculation.
- **Mode**: `INQ` restricts updates/deletes; `MNT` allows full CRUD.

## Error Handling
- Return errors for invalid carrier IDs (`ERR0010`), unlinked vendors (`ERR0037`), invalid dates (`ERR0003`), etc.
- Ensure atomic database operations.

</xaiArtifact>