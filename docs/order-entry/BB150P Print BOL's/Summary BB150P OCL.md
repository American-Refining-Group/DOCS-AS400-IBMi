### List of Use Cases Implemented by BB150P and BB150 RPG Programs

The `BB150P` and `BB150` RPG programs together implement a single primary use case in the IBM System/36 or AS/400 environment, focused on generating Bills of Lading (BOLs) for order shipments. Below is the identified use case:

1. **Generate Bills of Lading for Order Shipments**:
   - **Description**: This use case involves collecting user inputs for company, order numbers, and shipment reference numbers (SRNs), validating them, and generating formatted BOLs for printing. It supports both regular (order pick list time, `U5`) and railcar (railcar BOL time, `U6`) BOLs, including multi-load shipments and EDI 856 (Viscosity) orders. The process ensures accurate printing of customer, shipment, product, and freight details, while updating tracking files to record BOL generation.
   - **Components**:
     - `BB150P`: Prompts and validates user inputs (company, orders, SRNs, job queue, copy counts).
     - `BB150`: Processes validated inputs, retrieves data from multiple files, applies business logic, and prints BOLs to `LIST` and `LIST1` printer files.

No additional distinct use cases are implemented, as `BB150P` serves as the input/validation interface for `BB150`, which handles the core BOL printing logic.

---

### Function Requirement Document: Generate Bills of Lading



# Function Requirement Document: Generate Bills of Lading

## Purpose
The `Generate Bills of Lading` function generates formatted Bills of Lading (BOLs) for order shipments, supporting regular and railcar BOLs, single and multi-load orders, and EDI 856 (Viscosity) orders. It validates inputs, retrieves order/customer data, calculates weights/gallons, and produces printed output while updating tracking files.

## Inputs
- **Company Code** (`KYCO`, 2 chars): Identifies the company.
- **Selection Type** (`KYALSL`, 3 chars): `ALL` (all orders) or `SEL` (specific orders).
- **Order Numbers** (`KYORD1`–`KYORD0`, up to 10, 6 digits each): Specific orders for `SEL`.
- **From/To SRNs** (`KYFS`, `KYTS`, 3 digits each): Shipment reference numbers for multi-load orders.
- **Job Queue Flag** (`KYJOBQ`, 1 char): `Y`, `N`, or blank for job queue processing.
- **Copy Counts** (`KYCOPY`, `KYCOPB`, 2 digits each): Number of BOL copies.
- **BOL Type** (`U5` or `U6`, implied): Regular (order pick list) or railcar BOL.

## Outputs
- **Printed BOLs**: Formatted output to `LIST` (primary) and `LIST1` (PDF spooling) printer files, including:
  - Header: Company address, order number–SRN, customer/ship-to details, freight processor, batch number, routing codes.
  - Details: Product quantity, description, container size, gross weight, net gallons, hazmat indicators.
  - Railcar-Specific: Car capacity, outage, temperature (for `U6`).
  - EDI 856: Vendor/buyer part numbers, custom descriptions.
  - Footer: Freight terms, hazmat messages, shipper details, page number.
- **Updated Files**: `BBORTO`/`BBORTOB` (BOL printed flag), `BBSRNH` (SRN tracking).

## Process Steps
1. **Validate Inputs**:
   - Verify company code exists in `BICONT`.
   - Ensure `KYALSL` is `ALL` or `SEL`.
   - For `SEL`, validate order numbers in `BBORTR`.
   - For single-load orders (`BOMULO ≠ 'Y'`), ensure `KYFS` and `KYTS` are zero.
   - For multi-load orders (`BOMULO = 'Y'`), ensure `KYFS` and `KYTS` are both zero or both keyed, not exceeding total loads (`BOTOLO`), with `KYFS ≤ KYTS`.
   - Default `KYCOPY` and `KYCOPB` to `01` if zero.
   - Validate `KYJOBQ` as `Y`, `N`, or blank.

2. **Retrieve Order Data**:
   - Read header from `BBOTHS1` (`U5`) or `BBBLHS1` (`U6`) based on company (`BOCO`) and order number (`BORDNO`).
   - Skip deleted orders (`BODEL = 'D'`) or EDI errors (`BODEL = 'E'`).
   - Skip pick ticket printing if `BOORPR = 'H'`.
   - Retrieve customer (`ARCUST`), ship-to (`SHIPTO`), and freight processor (`BBFRPR`) details.
   - For EDI 856, read `EDICUS`, `BBASNH`, `BBASND`, `BBASNM` for shipper/product data.
   - Get routing from `TRRTCD` and container details from `GSCONT`, `BICONT`, `GSCNTR1`.

3. **Process Order Details**:
   - Read details from `BBTRAN` (non-deleted, `BDDEL ≠ 'D'`).
   - Retrieve product data from `GSTABL`/`GSPROD` (description, hazmat, gravity).
   - Determine container size from `GSCNTR1` (`BDCNTR`): bulk uses unit of measure (`BDIUM`), packaged uses short description (`TBSHDS`).
   - Calculate weights/gallons using `MINLBGL1` (pounds ↔ gallons conversion).
   - Read marks from `BBOTA1` (`U5`) or `BBBLA1` (`U6`) for remarks, filtering by print flags (`BAPICK`, `BABOL`, `BADSPH`).

4. **Generate BOL Output**:
   - Print to `LIST`/`LIST1` using formats (`PRTHDG`, `PRTDTL`, `PRTRIL`, `PRTEID`, `PRTTF`).
   - Include header (order–SRN, customer, ship-to, freight processor), details (quantity, product, weight, gallons), railcar data (`U6`), EDI data, and footer (freight terms, hazmat messages).
   - Handle multi-load orders with additional BOLs (`PRTHD2`, `PRTDT2`, etc.).
   - Suppress freight processor address for EDI orders.

5. **Update Tracking**:
   - Set `BOPBOL = 'Y'` in `BBORTO`/`BBORTOB` and `SRPBOL = 'Y'` in `BBSRNH`.

## Business Rules
1. **Input Validation**:
   - Company code must exist in `BICONT`.
   - Selection type must be `ALL` or `SEL`.
   - Order numbers must exist in `BBORTR` for `SEL`.
   - Single-load orders require zero `KYFS`/`KYTS`.
   - Multi-load orders require `KYFS`/`KYTS` both zero or both valid, ≤ `BOTOLO`, with `KYFS ≤ KYTS`.
   - `KYCOPY`/`KYCOPB` default to `01`.
   - `KYJOBQ` must be `Y`, `N`, or blank.

2. **Order Processing**:
   - Skip deleted orders (`BODEL = 'D'`) or EDI errors (`BODEL = 'E'`).
   - Skip pick ticket printing if `BOORPR = 'H'`.
   - Multi-load orders use order–SRN for unique identification.
   - Freight processor address prints only for external/customer processors.

3. **Calculations**:
   - Convert weights/gallons using `MINLBGL1` (e.g., `BDGVWT` → `BDNGAL`).
   - Container size: bulk (`BDIUM`), packaged (`TBSHDS` from `GSCNTR1`).

4. **Output Formatting**:
   - Regular BOLs (`U5`) print at order pick list time, railcar BOLs (`U6`) at railcar BOL time.
   - Include hazmat messages only for applicable products.
   - EDI 856 orders include vendor/buyer part numbers and custom descriptions.
   - Railcar BOLs include capacity, outage, and temperature when entered.

## External Dependencies
- **Files** (33): `BBTRAN`, `BBOTHS1`, `BBOTDS1`, `BBBLHS1`, `BBORTO`, `BBORTOB`, `BBOTA1`, `BBBLA1`, `BBORCL`, `ARCUST`, `ARCUPR`, `SHIPTO`, `GSCONT`, `BICONT`, `GSTABL`, `GSCTWT`, `GSCTUM`, `GSUMCV`, `GSHAZM`, `BBFRPR`, `TRRTCD`, `BOLEDIY`, `BBSRNH`, `SHPADR`, `EDICUS`, `BBASNH`, `BBASND`, `BBASNM`, `GSCNTR1`, `GSPRCT`, `BBCAID`, `GSPROD`, `LIST`, `LIST1`.
- **Programs**:
  - `MSHPADR`: Returns compressed ship-to address.
  - `MINLBGL1`: Converts pounds to gallons or vice versa.

## Notes
- The function assumes inputs are provided programmatically (e.g., via API or batch) rather than interactively.
- Error handling returns specific messages for invalid inputs (e.g., "INVALID COMPANY #", "FROM SRN MUST BE LESS THAN TO SRN").
- Output is formatted for printing, with PDF spooling support (`LIST1`).

