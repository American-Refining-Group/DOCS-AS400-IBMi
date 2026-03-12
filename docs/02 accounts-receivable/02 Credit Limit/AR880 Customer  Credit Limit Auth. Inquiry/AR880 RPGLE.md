The provided RPGLE program, `AR880.rpgle.txt`, is an RPG IV program designed for a Customer Credit Limit Authorization Inquiry on an IBM AS/400 or iSeries system. It is called from the OCL program `AR880.ocl36.txt` and handles the interactive process of displaying customer credit information, managing order authorizations, and generating email notifications for credit status changes. Below, I’ll explain the process steps, business rules, tables used, and external programs called.

### Process Steps of the RPGLE Program

The program is structured to handle customer credit limit inquiries, display order details, and allow users to authorize or unauthorize orders that exceed credit limits. It uses a workstation file with subfiles for user interaction and updates files to reflect authorization changes. Here’s a step-by-step breakdown of the process:

1. **Program Initialization**:
   - **Header Specifications**: The program uses `DFTACTGRP(*NO)` to run in a named activation group and `DFTNAME(AR880)` to set the default program name. The `FIXNBR(*ZONED:*INPUTPACKED)` option ensures proper handling of numeric fields.
   - **File Declarations**: Defines input files (`ARCONT`, `ARCUST`, `ARCUSP`, `BBORCL`, `ARCLGR`, `BBCSR`, `BBSLSM`, `GSCONT`), an update file (`BBORCLAU`), and printer files (`CREMAL`, `SMEMAL`) for output. The workstation file `AR880D` uses a subfile (`SFL1`) for displaying order details.
   - **Data Structures and Variables**: Defines data structures for message handling, display file information, and customer/order data. Arrays and fields store customer credit details, order amounts, and messages.
   - **Initial Setup**:
     - Checks if the control field `qsctl` is blank to set indicators for screen display (`*IN01`, `*IN09`, `*IN02`).
     - If `qsctl` is not blank, reads the customer selection screen (`AR880S1`) and sets indicators accordingly.
     - Initializes message handling fields and sets the default company number from `GSCONT` if available.

2. **Main Logic Flow**:
   - **Check for End of Job**: If the F3 key (`*INKG`) is pressed, the program sets the last record indicator (`*INLR`) and exits.
   - **Customer Selection Mode**:
     - If `U8` is off (`*INU8 = *OFF`), the program uses Screen 1 (`AR880S1`) for customer selection. If the F12 key (`*IN88`) is pressed, it redisplays Screen 1.
     - If `U8` is on (`*INU8 = *ON`), the program bypasses Screen 1, using customer data from `AR800` (stored in `ldacc`), and proceeds directly to Screen 2.
   - **Screen 1 Processing** (`S1` Subroutine):
     - Retrieves customer data from `ARCONT` and `ARCUST` using the company/customer number (`cocust`).
     - Validates the customer exists and is not deleted (`ardel <> 'D'`).
     - Populates Screen 2 fields with customer details (name, address, credit limit, aging buckets, etc.) from `ARCUST` and `ARCUSP`.
     - Checks authorization initials (`aauin`) against valid values (`ARG`, `KP`, `BZ`, `LM`, `JM`, `JS`). If invalid, displays an error message.
     - Calls `S2FILL` to preprocess order credit limits and calculate totals.
     - Reads the credit limit group file (`ARCLGR`) to accumulate totals for related customers.
     - Sets up the subfile for order display by calling `READFW`.

3. **Subfile Processing (Screen 2)**:
   - **Clear Subfile** (`SF1CLR` Subroutine): Clears the subfile (`SFL1`) and resets the relative record number (`rrn1`).
   - **Load Subfile** (`SF1LOD` Subroutine): Loads order details (customer, order number, batch, amount, status) into the subfile from `BBORCL`.
   - **Read Forward** (`READFW` Subroutine): Reads orders from `BBORCL` for the selected customer or group, filtering out deleted records (`bldel = 'D'`) and non-over-limit orders (`blovcl <> 'Y'`). Populates the subfile with up to 9 records per page.
   - **Display Subfile**: Writes the subfile control record (`SFLCTL1`) and displays the subfile if records exist (`*IN41`). Handles user input via the `EXFMT` operation.

4. **Order Authorization Processing**:
   - **Process Subfile** (`SF1PRC` Subroutine): Reads subfile records to process user selections (`A` for authorize, `U` for unauthorize).
   - **Authorize/UnAuthorize** (`S2AUTH` Subroutine):
     - Validates the selected order exists in `BBORCLAU` and is not deleted (`bxdel <> 'D'`).
     - Checks if the order needs authorization (`bxovcl = 'Y'` and `aunau <> 'Y'`) or unauthorization (`aunau = 'Y'` and `bxauin <> *BLANKS`).
     - Validates authorization initials (`aauin`) against allowed values.
     - If authorizing, updates `BBORCLAU` with the initials and user ID (`auser`) via the `AUTH` exception output.
     - If unauthorizing, clears the authorization fields in `BBORCLAU` via the `UNAU` exception output.
     - Generates email notifications via printer files `CREMAL` (CSR/A/R copy) and `SMEMAL` (salesman copy) with order details, customer info, and credit status.
     - Sends error messages for invalid cases (e.g., order not found, already authorized, invalid initials).

5. **Message Handling**:
   - **Add Message** (`ADDMSG` Subroutine): Sends error or status messages to the program message queue using `QMHSNDPM`.
   - **Write Message** (`WRTMSG` Subroutine): Displays messages in the message subfile.
   - **Clear Message** (`CLRMSG` Subroutine): Clears the message subfile using `QMHRMVPM`.

6. **Subfile Edit** (`EDITSFL` Subroutine):
   - Validates subfile selections to ensure no invalid options are entered when a customer/order is keyed manually.

7. **Output Generation**:
   - Generates spool files for `CREMAL` and `SMEMAL` when an order is authorized or unauthorized, containing details like order number, customer info, credit limit, aging buckets, and notification recipients (CSR, salesman, A/R clerk).

8. **Program Exit**:
   - Exits when the user presses F3 (`*INKG`) or completes processing, setting `*INLR` and returning.

### Business Rules

1. **Customer Selection**:
   - If `U8` is off, users must select a customer via Screen 1 (`AR880S1`).
   - If `U8` is on, customer data is passed from `AR800`, bypassing Screen 1.
   - The customer must exist in `ARCUST` and not be marked as deleted (`ardel <> 'D'`).

2. **Authorization Validation**:
   - Orders are only displayed if they exceed the credit limit (`blovcl = 'Y'`).
   - Authorization requires valid initials (`ARG`, `KP`, `BZ`, `LM`, `JM`, `JS`). Invalid initials trigger an error.
   - An order cannot be authorized if it’s already authorized (`bxauin <> *BLANKS`) or doesn’t need authorization (`bxovcl <> 'Y'`).
   - An order cannot be unauthorized if it’s not authorized (`bxauin = *BLANKS`).

3. **Credit Limit and Aging**:
   - Credit limits and aging buckets (current, 1-30, 31-60, 61-90, over 90 days) are retrieved from `ARCUST`.
   - Totals for related customers in a credit limit group (`ARCLGR`) are accumulated for display.
   - Available credit is calculated as `s2clmt - s2totd - s2orap`.

4. **Order Processing**:
   - Orders are read from `BBORCL` and updated in `BBORCLAU`.
   - The program accumulates approved (`s2orap`) and unapproved (`s2onap`) order amounts for display.

5. **Email Notifications**:
   - When an order is authorized or unauthorized, notifications are sent via `CREMAL` (to CSR/A/R) and `SMEMAL` (to salesman).
   - Notifications include order details, customer info, credit limit, aging buckets, and timestamps.

6. **Error Handling**:
   - Errors (e.g., invalid customer, order not found, invalid initials) are displayed via the message subfile.
   - The program prevents invalid subfile selections when manual customer/order input is provided.

7. **Modifications** (per comments):
   - **MG02 (11/09/2017)**: Added email notifications via spool files for CSR and salesman when order status changes.
   - **MG03 (03/21/2022)**: Removed beep for successful changes due to user annoyance.
   - **MG04 (04/08/2022)**: Removed `JBLIST` printer file output, no longer needed.

### Tables (Files) Used

The program uses the following files:
1. **AR880D**: Workstation file (display file) with subfile `SFL1` for interactive user interface.
2. **ARCONT**: Input file for accounts receivable control data (credit limit ranges).
3. **ARCUST**: Input file for customer master data (name, address, credit limit, aging buckets).
4. **ARCUSP**: Input file for customer-specific data (contact name).
5. **BBORCL**: Input file for order credit limit data.
6. **ARCLGR**: Input file for credit limit group data (related customers).
7. **BBORCLAU**: Update file for order authorization data.
8. **BBCSR**: Input file for customer service representative data (CSR ID, name, email).
9. **BBSLSM**: Input file for salesman data (salesman ID, name, email).
10. **GSCONT**: Input file for system control data (default company number).
11. **CREMAL**: Printer file for CSR and A/R email notifications.
12. **SMEMAL**: Printer file for salesman email notifications.

### External Programs Called

The program explicitly calls two external programs for message handling:
1. **QMHSNDPM**: Sends messages to the program message queue for error/status display.
2. **QMHRMVPM**: Removes messages from the program message queue to clear the message subfile.

### Summary

The `AR880.rpgle` program is a customer credit limit authorization inquiry tool that:
- Allows users to select customers and view credit details (via Screen 1 or `AR800`).
- Displays orders exceeding credit limits in a subfile, allowing authorization or unauthorization.
- Validates inputs (e.g., initials) and updates order authorization data in `BBORCLAU`.
- Generates email notifications for CSR, A/R, and salesmen when order status changes.
- Handles errors and user interactions via a message subfile.

**Tables Used**: `AR880D`, `ARCONT`, `ARCUST`, `ARCUSP`, `BBORCL`, `ARCLGR`, `BBORCLAU`, `BBCSR`, `BBSLSM`, `GSCONT`, `CREMAL`, `SMEMAL`.
**External Programs Called**: `QMHSNDPM`, `QMHRMVPM`.