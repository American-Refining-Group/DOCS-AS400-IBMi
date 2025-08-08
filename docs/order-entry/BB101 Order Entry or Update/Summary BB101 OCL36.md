### Use Cases Implemented by the Program Call Stack

The call stack consists of the following RPGLE programs, all part of the Customer Order Entry and Inventory system, called from the main OCL program `BB101.ocl36.txt`: `BB1033`, `BB106`, `BB115`, `BB104A`, `BB811`, `IN805`, and `MBBQTY`. Below is a list of use cases derived from these programs, representing the primary business functions they support without screen interaction, focusing on processing inputs to produce outputs.

1. **Validate Responsible Area and Major Location (BB1033)**:
   - Validates product/container combinations against location data to determine valid shipping locations and displays them for further processing.
   - Inputs: Company, product, container, order number.
   - Outputs: Valid responsible areas and major locations or error messages.

2. **Calculate Freight Charges (BB106)**:
   - Calculates freight charges for orders or invoices, updating transaction records with freight totals and handling collect service charges.
   - Inputs: Order/invoice type ('O' or 'I'), company, order number, freight table (optional), mode ('CALC' or 'TOP5').
   - Outputs: Freight totals, carrier choices (for 'TOP5'), updated order/invoice records.

3. **Check for Duplicate Orders (BB115)**:
   - Identifies duplicate orders based on company, customer, ship-to, product, and optionally PO number.
   - Inputs: Company, customer, ship-to, product, PO number (for Option B), order date.
   - Outputs: List of matching orders or confirmation of no duplicates.

4. **Track Order Cancellation or Reactivation (BB104A)**:
   - Records or updates cancellation/reactivation details for open orders, including reason codes and dates.
   - Inputs: Company, order number, reason code, date.
   - Outputs: Updated cancellation/reactivation records, validation status.

5. **Perform Batch Order Inquiry (BB811)**:
   - Retrieves and returns detailed order transaction information, including customer, ship-to, and freight collect service fees.
   - Inputs: Company, order number.
   - Outputs: Order details (header, transactions, accessorials), credit hold status.

6. **Check Inventory Balances (IN805)**:
   - Queries inventory balances for products and containers, checking sufficiency for orders and returning totals or insufficiency messages.
   - Inputs: Company, product, container, order number, quantity, mode ('1', '2', '3').
   - Outputs: Available inventory, in-transit/allocated totals, sufficiency status.

7. **Convert Order Quantity to Pounds and Gallons (MBBQTY)**:
   - Converts order quantities from various units of measure to pounds, gallons, and shipping weight.
   - Inputs: Company, product, container, unit of measure, quantity, specific gravity.
   - Outputs: Net pounds, net gallons, shipping weight.

---

### Function Requirement Document

<xaiArtifact artifact_id="71f78c7a-0f30-4d0b-ad35-15d42d3417c8" artifact_version_id="1d552cbb-f3a2-4ab6-a07b-1f01cf2d2fb8" title="OrderProcessingFunctions.md" contentType="text/markdown">

# Customer Order Processing Functions

This document outlines the business requirements for key functions in the Customer Order Entry and Inventory system, focusing on processing inputs to produce outputs without screen interaction. Each function corresponds to a use case implemented by the programs `BB1033`, `BB106`, `BB115`, `BB104A`, `BB811`, `IN805`, and `MBBQTY`.

## 1. Validate Responsible Area and Major Location
**Purpose**: Validate product/container combinations against location data to identify valid shipping locations.

**Inputs**:
- Company (2 characters)
- Product (4 characters)
- Container (3 alphanumeric characters)
- Order Number (variable length)

**Outputs**:
- List of valid responsible areas and major locations
- Error messages (e.g., "Prod/Cntr not in Prd Load File")

**Process Steps**:
1. Query product load files (`prdlody`, `prdlodx`, `prdlodz`) using company, product, and container.
2. Retrieve order details from `bbortr`/`bbortrx` using company and order number.
3. Validate locations, ignoring inactive or 'I' status records.
4. If no valid locations, return error message allowing override.
5. Return list of valid locations or error.

**Business Rules**:
- Ignore inactive or 'I' status records in product load files.
- Check alternate headers/majors if primary locations are invalid.
- Allow override for missing product/container records.
- Use file overrides for production (`g*`) or alternate (`z*`) libraries.

## 2. Calculate Freight Charges
**Purpose**: Calculate freight charges for orders or invoices, updating transaction records.

**Inputs**:
- Order/Invoice Type ('O' for order, 'I' for invoice, 1 character)
- Company (2 characters)
- Order Number (variable length)
- Freight Table (optional, for 'CALC' mode)
- Mode ('CALC' for single table, 'TOP5' for top five carriers)

**Outputs**:
- Freight totals
- Carrier choices (for 'TOP5')
- Updated order/invoice records

**Process Steps**:
1. If 'O', read order transactions from `bbortr`; if 'I', read invoice transactions from `bbtranin`.
2. For 'CALC' mode, calculate freight for specified table; for 'TOP5', calculate for up to five tables with tender sequences.
3. Call `MBBFRT` to compute freight for each load.
4. For multi-load orders/invoices, adjust quantities (single load to total loads).
5. Skip calculations for override freight rates.
6. Update `bbortr`/`bbtranin` with freight totals; add/update `frofc` for charges.
7. Handle collect service charges ($100, misc sequence 940 for invoices).
8. Return freight totals and carrier data.

**Business Rules**:
- Skip calculations for override freight rates.
- Do not update records if no freight table is found.
- For collect service: delete for orders, add/update/delete for invoices (code '943').
- Pricing: Freight collect (customer=1.00, freight=0.00); PPD&ADD (freight=0.25); prepaid with rack price (freight=0.25); prepaid with sales agreement (variable freight).
- Use file overrides for production (`g*`) or alternate (`z*`) libraries.

**Calculations**:
- Multi-load orders: Calculate freight for one load, then scale to total loads.
- Multi-load invoices: Multiply flat-rate freight by detail line count.
- Freight collect: Add $100 service charge (misc sequence 940).

## 3. Check for Duplicate Orders
**Purpose**: Identify duplicate orders to prevent redundant entries.

**Inputs**:
- Company (2 characters)
- Customer (variable length)
- Ship-to (variable length)
- Product (4 characters)
- PO Number (variable length, for Option B)
- Order Date (6 digits, MMDDYY)
- Option ('A' for company/customer/ship-to/product, 'B' for company/customer/ship-to/product/PO, default 'B')

**Outputs**:
- List of matching orders or confirmation of no duplicates

**Process Steps**:
1. Query `bboh2` (Option A) or `bboh3` (Option B) using input fields.
2. Validate customer and container via `arcusp` and `bicont`.
3. Retrieve details from `arcust`, `bbordd`, `gsprod`, `sa5fiud`, `sa5shz`, `shipto` for matching orders.
4. Return list of duplicates or "No Records" message.

**Business Rules**:
- Option B is default, including PO number for stricter matching.
- Validate customer, ship-to, product, and container data.
- Use file overrides for production (`g*`) or alternate (`z*`) libraries.
- Return "No Records" if no duplicates are found.

## 4. Track Order Cancellation or Reactivation
**Purpose**: Record or update cancellation/reactivation details for open orders.

**Inputs**:
- Company (2 characters)
- Order Number (variable length)
- Reason Code (variable length)
- Date (6 digits, MMDDYY)

**Outputs**:
- Updated cancellation/reactivation record
- Validation status

**Process Steps**:
1. Validate order via `bbordh` using company and order number.
2. Validate customer via `arcust`.
3. Validate reason code via `gstabl` (type 'BBORCN').
4. Call `dtp010r` to validate date (convert to CCYYMMDD).
5. Check `bbcnor` for existing record; add new or update existing with reason and date.
6. Return validation status.

**Business Rules**:
- Require valid company, order number, and reason code.
- Validate date to ensure it is not future-dated.
- Use file overrides for production (`g*`) or alternate (`z*`) libraries.
- Protect inputs in inquiry mode.

## 5. Perform Batch Order Inquiry
**Purpose**: Retrieve detailed order transaction information for review.

**Inputs**:
- Company (2 characters)
- Order Number (variable length)

**Outputs**:
- Order details (header, transactions, accessorials)
- Credit hold status

**Process Steps**:
1. Retrieve order transactions from `bbortr` using company and order number.
2. Retrieve customer and ship-to details from `arcust` and `shipto`.
3. Retrieve unit of measure from `gsctum` and accessorials from `bbota1` (including misc sequence 940).
4. Check order close status in `bborcl`.
5. Check credit hold status, returning "THIS ORDER IS CURRENTLY ON CREDIT HOLD" if applicable.
6. Return order details.

**Business Rules**:
- Include freight collect service fees ($100, misc sequence 940).
- Validate company, customer, ship-to, product, and container data.
- Use file overrides for production (`g*`) or alternate (`z*`) libraries.
- Protect inputs in inquiry mode.

## 6. Check Inventory Balances
**Purpose**: Verify inventory availability for products and containers for order fulfillment.

**Inputs**:
- Company (2 characters)
- Product (4 characters)
- Container (3 alphanumeric characters)
- Order Number (variable length)
- Quantity (numeric)
- Mode ('1' for always display, '2' for display on insufficiency, '3' for message on insufficiency)

**Outputs**:
- Available inventory, in-transit/allocated totals
- Sufficiency status or insufficiency message

**Process Steps**:
1. Query inventory files (`incont`, `inloc`, `intank`, `intkrv`, `inmtrdy`, `inmtaj`) for product/container availability.
2. Calculate in-transit totals from `sa5movd3`.
3. Calculate allocated totals from `bborpx`/`bbordh`, excluding current order.
4. Convert gallons to containers using `gsctwt`.
5. Validate tank reading dates (not future-dated).
6. Compare available inventory to order quantity.
7. Call `IN110C` and `IN805BC` for additional calculations.
8. For mode '1', return all details; for '2', return if insufficient; for '3', return insufficiency message.
9. Return inventory details and status.

**Business Rules**:
- Mode '1': Always return inventory details.
- Mode '2': Return details if insufficient inventory.
- Mode '3': Return "Not enough Prod to fill order" if insufficient.
- Exclude current order from allocated totals.
- Validate dates and data using `gsprod`, `gscntr`, `gsctum`, `arcust`, `shipto`.
- Use file overrides for production (`g*`) or alternate (`z*`) libraries.

**Calculations**:
- In-transit totals: Sum quantities from `sa5movd3`.
- Allocated totals: Sum quantities from `bborpx`/`bbordh`, excluding current order.
- Gallons to containers: Use conversion factors from `gsctwt`.

## 7. Convert Order Quantity to Pounds and Gallons
**Purpose**: Convert order quantities to pounds, gallons, and shipping weight.

**Inputs**:
- Company (2 characters)
- Product (4 characters)
- Container (3 alphanumeric characters)
- Unit of Measure (e.g., 'LBS', 'GAL', 'KG', 'ECH')
- Quantity (numeric)
- Specific Gravity (numeric)

**Outputs**:
- Net Pounds (7 digits)
- Net Gallons (7 digits)
- Shipping Weight (7 digits)

**Process Steps**:
1. If `UM='LBS'`, set net pounds = quantity; convert to gallons via `MINLBGL1`.
2. If `UM='GAL'`, set net gallons = quantity; convert to pounds via `MINLBGL1`.
3. If `UM='KG'`, convert to pounds (quantity * 2.20462), then to gallons via `MINLBGL1`.
4. If `UM='ECH'`, query `gsctum` for gallons per unit; if not found, query `gsumcv` for factor or use 1.
5. For other units, query `gsumcv` for factor or use 1; convert to gallons, then pounds via `MINLBGL1`.
6. Calculate shipping weight from `gsctwt` (quantity * gross weight) or use net pounds if no record.
7. Return net pounds, net gallons, and shipping weight.

**Business Rules**:
- Use `gsctum` for 'ECH' conversions, `gsumcv` for other units; default to factor 1 if no record.
- Prioritize pounds-to-unit over gallons-to-unit for GVW-Tare calculations.
- For non-LBS/GAL/ECH, use gallon-based factor for conversions.
- Validate container as alphanumeric.
- Check for deleted/inactive records in `gsctum`, `gsumcv`, `gsctwt`.
- Use file overrides for production (`g*`) or alternate (`z*`) libraries.

**Calculations**:
- Kilograms to pounds: Quantity * 2.20462.
- Shipping weight: Quantity * `WTGROS` (from `gsctwt`) or net pounds.
- Gallons/pounds conversion: Via `MINLBGL1` using specific gravity.

</xaiArtifact>