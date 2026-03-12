### List of Use Cases

Based on the call stack (OCL procedure BB850.ocl36.txt loading and running RPG program BB850.rpg36.txt, which calls BB1016.rpg36.txt for product lookups), the implemented use cases are as follows. These are derived from the program's interactive inquiry logic, data validations, and lookup functionality. Note that BB1016 is reusable (e.g., also for order entry in BB101), but in this stack, it's specifically for supporting the price history inquiry.

1. **Price History Inquiry**: Retrieve and display historical rack pricing data for a specific product based on a composite key (company, location, product code, container, unit of measure). Includes validation of input keys, paging through history records (up to 5 per page), skipping deleted records, and displaying fields like dates, times, multi-level prices, quantities, and flags (e.g., no rack price required, inactive).

2. **Product Code Lookup by Description or Customer-Specific Criteria**: When product code is not provided or needs selection, perform a lookup to populate a list of up to 15 products matching a partial description or customer/ship-to specifics. Returns product codes, descriptions (standard and alternate), and fluid codes for selection, filtering out deleted or non-sellable items.

### Function Requirement Document

Assuming each use case is reimagined as a non-interactive, batch-style function that accepts all required inputs upfront (e.g., via parameters) and returns structured output (e.g., JSON or dataset) without screen interactions. The functions focus on business requirements for data retrieval and validation, with concise steps and rules. Calculations are minimal (e.g., date/time conversions, array population) and explained where relevant.

#### Function 1: Price History Inquiry
**Purpose**: Retrieve paginated historical rack pricing data for a given product key, validating inputs and applying business filters, to support inquiries without user interaction.

**Inputs**:
- Company (2 digits, required).
- Location (3 digits, required).
- Product Code (4 chars, optional; if blank, triggers Product Code Lookup function).
- Container (3 chars, optional; must be blank for non-fluid products).
- Unit of Measure (3 chars, required).
- Page Number (integer, default 1; for pagination).
- Page Size (integer, default 5; max records per page).
- Direction (string: 'forward' or 'backward'; for paging).

**Outputs**:
- List of history records (up to page size), each with: Date (MM/DD/YY), Time (HH:MM), Rack Prices (4 levels, decimal), Quantity Levels (4 levels + min qty, integers), Flags (No Rack Price: 'N' or blank; Inactive: 'I' or blank).
- Total Pages (estimated).
- Error Message (if validation fails or no data).

**Process Steps**:
1. Validate Inputs:
   - Chain to BICONT: Company must exist and have invoicing style '5'.
   - Chain to INLOC: Location must exist for the company.
   - If Product Code blank: Call Product Code Lookup function with description search; use first matching code if available.
   - Chain to GSPROD: Product must exist; if fluid code 'N', container must be blank.
   - Chain to GSCTUM: Container and unit must match product.
   - If any fail, return error (e.g., "Invalid Company").

2. Position File Pointer:
   - Build composite key (company + location + product + container + unit + high date/time for forward; adjust for backward paging using hold date/time).
   - SETLL on BBPRCE file using key limit.

3. Retrieve Records:
   - Read records (READP for backward, READ for forward), skipping deleted (RKDEL = 'D').
   - For each valid record (up to page size): Convert date (RKDAT6 * 100.0001 to MM/DD/YY), copy time (RKTIME to HH:MM), prices (RKPRCE/RKPR02-04 to decimals), quantities (RKMINQ/RKQT01-04 to integers), flags (RKRKRQ/RKINAC).
   - Track hold date/time for next page positioning.

4. Handle Pagination and End:
   - If no more records, indicate end-of-file.
   - Calculate total pages based on record count estimate (not exact; derived from reads).

**Business Rules**:
- Read-only; no updates.
- History keyed by company/location/product/container/unit; sorted descending by date/time.
- Non-fluid products (TPFLCD = 'N') prohibit non-blank containers.
- Display multi-level pricing (up to 4) for tiered quantity-based racks.
- Flags: 'N' indicates no rack price required; 'I' indicates inactive status.
- Y2K-compliant date handling (century prefix '19').

#### Function 2: Product Code Lookup
**Purpose**: Search for product codes matching description or customer criteria, returning a list for selection, to assist in key completion without interactive screens.

**Inputs**:
- Company (2 digits, required).
- Customer (6 digits, optional; for customer-specific mode).
- Ship-To (3 digits, optional; for customer-specific mode).
- Lookup Mode ('P*' for customer-specific products; 'D*' for description search; required).
- Description (30 chars, optional; partial match for 'D*').
- Last Product/Description (for paging continuation).
- Page Size (integer, default 15).

**Outputs**:
- List of products (up to page size), each with: Line Number, Product Code, Standard Description, Customer Alternate Description, Fluid Code ('N' for non-fluid).
- Next Page Indicator (true if more records).
- Error Message (if no matches).

**Process Steps**:
1. Determine File and Mode:
   - If from order entry context (e.g., PROGRM='BB101'): Use ARCUP36/GSPRD6.
   - Otherwise: Use ARCUP16/GSPROD.
   - For 'P*': Build key with company/customer/ship-to (+ alternate desc for paging in ARCUP16).

2. Position and Read:
   - SETLL on appropriate file (ARCUP16/36 for 'P*'; GSPRD6 for 'D*').
   - Read forward, skipping deleted (CPDEL/TBDEL = 'D').
   - Filter: For 'P*', match company/customer/ship-to, skip duplicates. For 'D*', match company, require sellable (TBSELL = 'Y').

3. Populate Results:
   - For each (up to page size): Copy product code, alternate desc (from ARCUP; blank for 'D*'), standard desc and fluid code (chain to GSPROD if needed).

**Business Rules**:
- 'P*' prioritizes customer-specific (alternate desc if available); supplements with standard details.
- 'D*' searches all sellable products by partial description.
- Exclude non-sellable or deleted items.
- Fluid code enforces business distinctions (e.g., non-fluid impacts container rules in inquiries).
- Paging uses last result for continuation (e.g., last alternate desc in 'P*').