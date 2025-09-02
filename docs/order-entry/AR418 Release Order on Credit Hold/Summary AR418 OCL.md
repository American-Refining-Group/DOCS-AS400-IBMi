Below, I will first identify the **use cases** implemented in the call stack consisting of the `AR418.ocl36` (OCL), `AR418.rpgle` (RPGLE), and `AR418TC.clp` (CLP) programs. Then, I will provide a **Function Requirement Document** for one of the primary use cases, reimagined as a large function that processes inputs programmatically rather than through screen interactions. The document will focus on business requirements, process steps, and relevant calculations.

---

### **Use Cases Implemented in the Call Stack**

The call stack (`AR418.ocl36`, `AR418.rpgle`, `AR418TC.clp`) implements a system for managing order authorization and credit limit overrides, with notifications sent to relevant parties. Based on the code and comments, the following use cases are implemented:

1. **Authorize an Order Over Credit Limit**:
   - **Description**: Allows authorized users to override the credit limit for an order that exceeds the customer’s credit limit, marking it as authorized in the system.
   - **Details**:
     - Validates company, customer, and order number.
     - Requires authorization initials (`kyauin`) to proceed.
     - Updates the `bborcl` file with authorization details (initials and user ID).
     - Generates email notifications to the CSR, salesman, and potentially a credit manager with order details and status ("RELEASED FROM CREDIT HOLD").
   - **Files Involved**: `bicont`, `bborcl`, `bbcsr`, `bbslsm`, `arcust`, `cremal`, `smemal`.
   - **Programs Involved**: `AR418.rpgle` (core logic), `AR418TC.clp` (notification distribution when `?9? = 'G'`).
   - **Conditions**: Blocked if the order is already authorized or does not need authorization.

2. **Place an Order on Credit Hold**:
   - **Description**: Allows authorized users to place an order on credit hold, typically when it exceeds the credit limit or other criteria warrant it.
   - **Details**:
     - Validates company, customer, and order number.
     - Sets the `blovcl` flag to `'Y'` in `bborcl` to indicate the hold status.
     - Generates email notifications to the CSR, salesman, and potentially a credit manager with order details and status ("PLACED ON CREDIT HOLD").
   - **Files Involved**: `bicont`, `bborcl`, `bbcsr`, `bbslsm`, `arcust`, `cremal`, `smemal`.
   - **Programs Involved**: `AR418.rpgle` (core logic), `AR418TC.clp` (notification distribution when `?9? = 'G'`).
   - **Conditions**: Blocked if the order is already on hold.

3. **Unauthorize an Order**:
   - **Description**: Allows authorized users to remove authorization from a previously authorized order, effectively resetting its status.
   - **Details**:
     - Validates company, customer, and order number.
     - Requires the user to confirm unauthorization (`kyunau = 'Y'`).
     - Clears authorization fields (`blauin`, `blusid`) in `bborcl`.
     - Generates email notifications to the CSR, salesman, and potentially a credit manager with order details and status ("UNAU DENIED").
   - **Files Involved**: `bicont`, `bborcl`, `bbcsr`, `bbslsm`, `arcust`, `cremal`, `smemal`.
   - **Programs Involved**: `AR418.rpgle` (core logic), `AR418TC.clp` (notification distribution when `?9? = 'G'`).
   - **Conditions**: Blocked if the order was not previously authorized.

4. **Restrict Program Access During Business Hours**:
   - **Description**: Limits program execution to specific users during normal business hours (06:00–20:00) to prevent disruptions.
   - **Details**:
     - Only authorized users (`BRIANZ`, `OTTO`, `MARTY`, `FRANCO`, `GSSZTEST`, `JLBZTEST`, `JBTEST`, `MIDLA`, `SPAHR`) can run the program during business hours.
     - Non-authorized users are restricted to non-business hours.
     - Displays an error message ("CANNOT RUN DURING NORMAL BUSINESS HOURS") if access is denied.
   - **Files Involved**: None directly (uses `kyuser` from data area).
   - **Programs Involved**: `AR418.rpgle` (access check in `onetim` subroutine).
   - **Conditions**: Applies to all other use cases.

5. **Send Email Notifications for Credit Status Changes**:
   - **Description**: Sends email notifications to CSRs, salesmen, and potentially credit managers when an order’s credit status changes (authorized, unauthorized, or placed on hold).
   - **Details**:
     - Processes spooled files (`cremal`, `smemal`, and potentially another file for `'ORDER CREDIT STATUS3'`) to generate emails.
     - Uses output queues (`QUSRSYS/CSROUTQ`, `SLMNOUTQ`, `DLTOUTQ`) for distribution.
     - Deletes spooled files after processing to maintain system cleanliness.
   - **Files Involved**: `cremal`, `smemal` (spooled files).
   - **Programs Involved**: `AR418TC.clp` (notification processing), `AR418.rpgle` (generates spooled files).
   - **Conditions**: Executed only when `?9? = 'G'` in the OCL program.

---

### **Function Requirement Document**

Below is a **Function Requirement Document** for the primary use case: **Authorize an Order Over Credit Limit**, reimagined as a programmatic function that accepts inputs directly rather than through a screen. This document outlines the business requirements, process steps, and calculations concisely.

<xaiArtifact artifact_id="cf2dbe96-c50e-41d9-ae4c-1e93e04cc74e" artifact_version_id="9df6f160-0254-49ed-a822-579b9732277d" title="AuthorizeOrderCreditLimitOverride.md" contentType="text/markdown">

# Function Requirement Document: Authorize Order Credit Limit Override

## Overview
The `AuthorizeOrderCreditLimitOverride` function processes a request to authorize an order that exceeds a customer’s credit limit, updating the order status and sending email notifications to relevant parties (CSR, salesman, credit manager). The function takes inputs programmatically and validates them before processing.

## Business Requirements
1. **Authorization Eligibility**:
   - Only valid company, customer, and order numbers are processed.
   - The order must exist, not be deleted, and require authorization (exceeds credit limit, not already authorized).
   - Authorization requires valid user initials.
2. **Access Control**:
   - Only specific users (`BRIANZ`, `OTTO`, `MARTY`, `FRANCO`, `GSSZTEST`, `JLBZTEST`, `JBTEST`, `MIDLA`, `SPAHR`) can process during business hours (06:00–20:00). Others are restricted to non-business hours.
3. **Data Updates**:
   - Update the order record with authorization initials and user ID.
   - Clear any hold status if applicable.
4. **Notifications**:
   - Send email notifications to the CSR, salesman, and credit manager with order details, customer information, and credit status.
   - Notifications include order number, customer details, credit limit, and payment history.
5. **Error Handling**:
   - Return error messages for invalid inputs, unauthorized access, or already authorized orders.
   - Ensure system integrity by preventing duplicate or conflicting updates.

## Inputs
- **Company Number** (`kyco`, 2-digit numeric): Identifies the company.
- **Customer Number** (`kycust`, 6-digit numeric): Identifies the customer.
- **Order Number** (`kyord#`, 6-digit character): Identifies the order.
- **Authorization Initials** (`kyauin`, 3-character): Initials of the authorizing user.
- **User ID** (`kyuser`, 8-character): ID of the user processing the request.
- **Environment Flag** (`env`, 1-character): `'G'` for test environment, otherwise production.

## Outputs
- **Status Code**: Success or error code (e.g., `SUCCESS`, `INVALID_COMPANY`, `ORDER_ALREADY_AUTHORIZED`).
- **Error Message**: Descriptive message for errors (e.g., "INVALID COMPANY NUMBER ENTERED").
- **Spooled Files**: Notification files (`cremal`, `smemal`) for email distribution.

## Process Steps
1. **Validate User Access**:
   - If current time is between 06:00 and 20:00 and `kyuser` is not in the authorized list, return error: "CANNOT RUN DURING NORMAL BUSINESS HOURS".
2. **Validate Inputs**:
   - Check `kyco` exists in `bicont` and is not deleted. If invalid, return error: "INVALID COMPANY NUMBER ENTERED".
   - Check `kyco`, `kycust`, `kyord#` exist in `bborcl` and are not deleted. If invalid, return error: "INVALID CO/CUST/ORDER# ENTERED".
   - If `kyauin` is blank, return error: "MUST ENTER AUTHORIZATION INITIALS".
3. **Check Order Status**:
   - If `bborcl.blovcl <> 'Y'` (not over limit) and `bborcl.blauin` is not blank, return error: "ORDER DOES NOT NEED AUTHORIZATION".
   - If `bborcl.blauin` and `bborcl.blusid` are not blank, return error: "ORDER HAS ALREADY BEEN AUTHORIZED".
4. **Update Order**:
   - Update `bborcl`:
     - Set `blauin = kyauin` (authorization initials).
     - Set `blusid = kyuser` (user ID).
     - Clear `blovcl` if on hold.
   - Capture current date and time for audit (`sytime`, `sydate`).
5. **Generate Notifications**:
   - Retrieve CSR email (`qqcrem`) from `bbcsr` using `bborcl.bltkby`. Default to `SJM` if not found.
   - Retrieve salesman email (`qqsmem`) from `bbslsm` using `bborcl.blslmn`.
   - Retrieve credit manager email (`qqcrem`) from `bbcsr` using `kyauin` or `blauin`.
   - Generate `cremal` and `smemal` spooled files with:
     - Order details: `kyord#`, `kycust`, `blslmn`, `bltkby`.
     - Customer details: `arname`, `aradr1`, `aradr2`, `aradr3`, `arclmt`, `arpdat`, `artotd`, `arcurd`, `ar0110`, `ar1120`, `ar2130`, `arov30`.
     - Status: "RELEASED FROM CREDIT HOLD".
     - Timestamp: Current date and time.
6. **Distribute Notifications (if `env = 'G'`) **:
   - Process `cremal` to `QUSRSYS/CSROUTQ` and `smemal` to `SLMNOUTQ` using `SFASPLIT`.
   - Distribute notifications using `SFARDST` for `'ORDER CREDIT STATUS'` and `'ORDER CREDIT STATUS2'`.
   - Handle `'ORDER CREDIT STATUS3'` to `DLTOUTQ` for additional recipients.
   - Delete spooled files after processing.
7. **Return Result**:
   - Return `SUCCESS` with message: "ORDER HAS BEEN AUTHORIZED".

## Calculations
- **No Financial Calculations**: The function does not perform calculations on amounts (e.g., `bltamt`, `bloamt`, `arclmt`). It only validates and updates fields.
- **Timestamp Generation**:
  - Current date and time (`timdat`, 12-digit numeric) is split into `sytime` (HHMMSS) and `sydate` (MMDDYY) for audit and notification purposes.
- **Access Time Check**:
  - Extract current hour (`hour`, 2-digit numeric) from `timdat`. If `hour >= 06` and `hour <= 20`, enforce user restrictions.

## Data Sources
- **bicont**: Validates company number (`bcdel`, delete code).
- **bborcl**: Order details (`bldel`, `blcono`, `blcust`, `blordr`, `blovcl`, `blauin`, `blusid`, `blslmn`, `bltkby`).
- **bbcsr**: CSR email (`qqcrem`, `crcaid`, `crcanm`).
- **bbslsm**: Salesman email (`qqsmem`, `smcaid`, `smcanm`).
- **arcust**: Customer details (`qqarcu`, `arname`, `aradr1`, `aradr2`, `aradr3`, `arclmt`, `arpdat`, `artotd`, `arcurd`, `ar0110`, `ar1120`, `ar2130`, `arov30`).
- **gscont**: Retrieves default company number (`gxcono`).

## Error Messages
- "INVALID COMPANY NUMBER ENTERED" (invalid `kyco`).
- "INVALID CO/CUST/ORDER# ENTERED" (invalid order).
- "MUST ENTER AUTHORIZATION INITIALS" (blank `kyauin`).
- "ORDER DOES NOT NEED AUTHORIZATION" (order not over limit).
- "ORDER HAS ALREADY BEEN AUTHORIZED" (already authorized).
- "ORDER HAS BEEN DELETED FROM FILE" (order deleted).
- "CANNOT RUN DURING NORMAL BUSINESS HOURS" (access restricted).

## Assumptions
- The function assumes a test environment when `env = 'G'`, triggering additional notification processing via `AR418TC.clp`-like logic.
- Email distribution relies on external tools (e.g., SpoolFlex) configured for output queues.
- No direct database writes are performed for notifications; they are handled via spooled files.

</xaiArtifact>

---

### **Notes on Other Use Cases**

The other use cases (Place Order on Credit Hold, Unauthorize Order, Restrict Access, Send Notifications) could be similarly reimagined as programmatic functions. For brevity, I focused on the primary use case (Authorize Order). Key differences for the others:
- **Place Order on Credit Hold**: Similar inputs, but sets `bborcl.blovcl = 'Y'` and generates "PLACED ON CREDIT HOLD" notifications.
- **Unauthorize Order**: Requires `kyunau = 'Y'` input, clears `blauin` and `blusid`, and generates "UNAU DENIED" notifications.
- **Restrict Access**: Embedded as a validation step in all functions, checking `kyuser` and time.
- **Send Notifications**: Handled as a sub-process within each function, with conditional execution based on `env = 'G'`.

If you need function requirement documents for these additional use cases, please let me know!