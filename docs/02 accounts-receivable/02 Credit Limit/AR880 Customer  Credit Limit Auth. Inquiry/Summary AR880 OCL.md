### List of Use Cases Implemented by the Program Call Stack

The call stack consists of three programs: `AR880.ocl36.txt` (OCL), `AR880.rpgle.txt` (RPGLE), and `AR880TC.clp.txt` (CLP). Together, they implement a single primary use case for the Customer Credit Limit Authorization Inquiry system on an IBM AS/400 or iSeries platform. The use case is:

1. **Customer Credit Limit Authorization Inquiry and Order Status Management**:
   - This use case allows users to inquire about a customer’s credit status, view orders exceeding credit limits, authorize or unauthorize those orders, and send email notifications to Customer Service Representatives (CSRs), Accounts Receivable (A/R) clerks, and salesmen when an order’s credit hold status changes. The system supports interactive customer selection (via a screen or pre-selected from another program, `AR800`) and manages credit-related data retrieval, validation, and updates.

### Function Requirement Document



# Customer Credit Limit Authorization Function Requirements

## Overview
The **Customer Credit Limit Authorization Function** processes customer credit inquiries and manages order authorization for orders exceeding credit limits. It retrieves customer and order data, validates authorization actions, updates order status, and sends email notifications to relevant parties (CSRs, A/R clerks, salesmen). The function assumes all inputs are provided programmatically (no interactive screen) and operates in an IBM AS/400 or iSeries environment.

## Inputs
- **Company Number** (`cono`, 2-digit numeric): Identifies the company.
- **Customer Number** (`cust`, 6-digit numeric): Identifies the customer.
- **Order Number** (`aord`, 6-digit numeric): Identifies the order to authorize/unauthorize (optional, for specific order processing).
- **Authorization Initials** (`aauin`, 3-character string): Initials of the authorizer (e.g., `ARG`, `KP`, `BZ`, `LM`, `JM`, `JS`).
- **Authorization Action** (`aunau`, 1-character string): `Y` to unauthorize, blank to authorize.
- **Environment Code** (`env`, 1-character string): Environment identifier (e.g., `G` for test environment).
- **User ID** (`auser`, 8-character string): ID of the user performing the action.

## Outputs
- **Customer Credit Details**:
  - Customer name, address, contact name, phone number.
  - Credit limit, total due, aging buckets (current, 1-30, 31-60, 61-90, over 90 days).
  - Available credit, approved/unapproved order amounts.
- **Order Details** (for orders exceeding credit limits):
  - Order number, batch number, amount, authorization status, user ID.
- **Email Notifications**:
  - Spool files (`CREMAL`, `SMEMAL`) for CSR/A/R and salesman, containing order and customer details.
- **Status Messages**: Success or error messages (e.g., invalid customer, order not found).

## Process Steps
1. **Initialize Environment**:
   - Delete existing file overrides.
   - Set up environment using `GSGENIEC` (initialization program).
   - Override database files (`BBCSR`, `BBSLSM`) to point to `QS36F` library.
   - Load files: `ARCONT` (control), `ARCUST` (customer), `ARCUSP` (customer preferences), `BBORCL` (orders), `ARCLGR` (credit group), `BBORCLAU` (order authorization).

2. **Validate Inputs**:
   - Verify company (`cono`) and customer (`cust`) exist in `ARCUST` and are not deleted (`ardel <> 'D'`).
   - If order number (`aord`) is provided, ensure it exists in `BBORCLAU` and is not deleted (`bxdel <> 'D'`).
   - Validate authorization initials (`aauin`) against allowed values (`ARG`, `KP`, `BZ`, `LM`, `JM`, `JS`).
   - Check if order requires authorization (`bxovcl = 'Y'` for over credit limit) or unauthorization (`bxauin <> *BLANKS`).

3. **Retrieve Customer Credit Data**:
   - Fetch customer details (name, address, credit limit, aging buckets) from `ARCUST`.
   - Get contact name from `ARCUSP`.
   - Retrieve credit limit ranges from `ARCONT` (e.g., 1-10, 11-20, 21-30, over 30 days).
   - Aggregate totals for related customers in credit group (`ARCLGR`).

4. **Calculate Credit Metrics**:
   - **Total Due** (`s2totd`): Sum of `artotd` from `ARCUST` for all customers in the credit group.
   - **Aging Buckets**:
     - Current Due (`s2curd`): Sum of `arcurd`.
     - 1-30 Days (`s20110`): Sum of `ar0110`.
     - 31-60 Days (`s21120`): Sum of `ar1120`.
     - 61-90 Days (`s22130`): Sum of `ar2130`.
     - Over 90 Days (`s2ov30`): Sum of `arov30`.
   - **Order Amounts**:
     - Approved Orders (`s2orap`): Sum of `bltamt` or `bloamt` from `BBORCL` where `blovcl <> 'Y'` or `blauin <> *BLANKS`.
     - Unapproved Orders (`s2onap`): Sum of `bltamt` or `bloamt` where `blovcl = 'Y'` and `blauin = *BLANKS`.
   - **Available Credit** (`s2avcl`): `s2clmt - s2totd - s2orap`.

5. **Process Order Authorization**:
   - If `aord` is provided:
     - **Authorize**: If `aunau <> 'Y'`, update `BBORCLAU` with `aauin` and `auser` for the order.
     - **Unauthorize**: If `aunau = 'Y'`, clear authorization fields in `BBORCLAU`.
   - If no `aord`, retrieve all orders from `BBORCL` where `blovcl = 'Y'` for display.

6. **Generate Notifications**:
   - Create spool files (`CREMAL`, `SMEMAL`) with order details (number, customer, amount, status), customer details, and credit metrics.
   - If `env = 'G'`, call `AR880TC` to process spool files:
     - `CREMAL` to `CSROUTQ` (CSR/A/R) and `DLTOUTQ` (additional copy).
     - `SMEMAL` to `SLMNOUTQ` (salesman).
     - Delete original spool files after processing.

7. **Return Results**:
   - Output customer credit details, order details, and status messages.

## Business Rules
1. **Customer Validation**:
   - Customer must exist in `ARCUST` and not be deleted (`ardel <> 'D'`).
   - Invalid customer returns error: "INVALID CUSTOMER".

2. **Order Validation**:
   - Order must exist in `BBORCLAU`, not be deleted (`bxdel <> 'D'`), and exceed credit limit (`bxovcl = 'Y'`).
   - Errors:
     - Order not found: "Order# [order] NOT FOUND FOR THIS CUSTOMER".
     - Order deleted: "Order# [order] HAS BEEN DELETED FROM FILE".
     - No authorization needed: "Order# [order] DOES NOT NEED AUTHORIZATION".
     - Already authorized: "Order# [order] HAS ALREADY BEEN AUTHORIZED".
     - No unauthorization needed: "Order# [order] DOES NOT NEED TO BE UNAUTHORIZED".

3. **Authorization Rules**:
   - Authorization requires valid initials (`ARG`, `KP`, `BZ`, `LM`, `JM`, `JS`). Invalid initials return error: "INVALID AUTHORIZATION INITIALS USED".
   - Authorization updates `BBORCLAU` with initials and user ID.
   - Unauthorization clears authorization fields.

4. **Notification Rules**:
   - Notifications are sent for each authorization/unauthorization action.
   - `CREMAL` notifies CSR/A/R; `SMEMAL` notifies salesman.
   - Notifications include order number, customer details, credit limit, aging buckets, and timestamp.

5. **Environment-Specific Processing**:
   - If `env = 'G'`, spool files are processed by `AR880TC` for email distribution.
   - Spool files are deleted after processing to avoid duplication.

## Calculations
- **Total Due**: `s2totd = Σ(artotd)` for all customers in credit group (`ARCLGR`).
- **Aging Buckets**: Sum `arcurd`, `ar0110`, `ar1120`, `ar2130`, `arov30` across credit group customers.
- **Order Amounts**:
  - Approved: `s2orap = Σ(bltamt or bloamt)` where `blovcl <> 'Y'` or `blauin <> *BLANKS`.
  - Unapproved: `s2onap = Σ(bltamt or bloamt)` where `blovcl = 'Y'` and `blauin = *BLANKS`.
- **Available Credit**: `s2avcl = s2clmt - s2totd - s2orap`.

## Error Handling
- Return descriptive error messages for invalid inputs or failed validations.
- Ignore spool file processing errors (`SF00136`) to ensure completion.

## External Dependencies
- **Programs Called**:
  - `GSGENIEC`: Initializes environment.
  - `AR880TC`: Processes spool files for email (if `env = 'G'`).
  - `QMHSNDPM`, `QMHRMVPM`: Handle message queue operations.
  - `SFASPLIT`, `SFARDST`: Third-party commands for spool file processing.
- **Files**:
  - Input: `ARCONT`, `ARCUST`, `ARCUSP`, `BBORCL`, `ARCLGR`, `BBCSR`, `BBSLSM`, `GSCONT`.
  - Update: `BBORCLAU`.
  - Output: `CREMAL`, `SMEMAL` (spool files).

