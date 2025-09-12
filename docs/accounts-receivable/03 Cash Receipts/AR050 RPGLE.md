The provided document is an RPGLE program named `AR050.rpgle.txt`, which is part of a cash receipts payment statement entry system for an IBM System/36 or AS/400 environment. This program is invoked by the `AR050.ocl36` OCL script, which sets up the necessary files and environment. The `AR050` program handles the entry and application of payment statements, including customer selection, invoice payment processing, and check application. Below, I’ll explain the process steps, business rules, tables (files) used, and external programs called.

### Process Steps of the AR050 RPGLE Program

The `AR050` program facilitates the entry of payment statements for cash receipts, allowing users to select customers, apply payments to invoices, and manage checks. It uses a workstation display file for user interaction and multiple disk files for data storage and retrieval. Due to the document being truncated, I’ll focus on the provided code and context from the OCL script, summarizing the key steps based on the available information.

1. **Program Initialization**:
   - **Header Specifications**:
     - `DFTACTGRP(*NO)`: Runs in a named activation group.
     - `fixnbr(*zoned:*inputpacked)`: Ensures proper handling of zoned and packed numeric data.
     - `dftname(ar050)`: Sets the default program name to `AR050`.
     - `Handler('PROFOUNDUI(HANDLER)')`: Uses Profound UI for modern user interface integration.
     - `infsr(rollsr)`: Specifies a subroutine (`rollsr`) for handling workstation file errors (e.g., roll-up/roll-down navigation).
   - **Data Definitions**:
     - Arrays (`ivd8`, `svcd`, `svpy`, `svds`, `cus`, `nam`, `ad1`, `ad2`, `cd`, `py`, `ds`, `ivno`) store data for up to 16 invoices or customers, including invoice numbers, payment codes, amounts, discounts, customer numbers, names, and addresses.
     - `com` array holds 40 error messages (e.g., "NO CUSTOMER FOUND FOR THIS SEARCH", "INVALID CUST NO OR CUSTOMER IS DELETED").
     - `infds` captures workstation file status (e.g., `*status` for error codes).
     - `kysnm` and `snm` handle keyed search names for customer lookup.

2. **Workstation File Processing**:
   - The program uses the `ar050d` display file (format `ar050fm`) to interact with users.
   - Supports roll-up (`*in18`) and roll-down (`*in19`) for navigating lists of customers or invoices.
   - Displays fields such as customer number, name, address, invoice number, due date, payment code (`cd`), payment amount (`py`), discount amount (`ds`), and error messages (`msg1`, `msg2`).

3. **Customer Selection**:
   - Uses `arcustx` (customer alternate index) and `arcust` (customer master) to retrieve customer data based on a search key (`kysnm`).
   - Validates customer existence and status, displaying errors like "NO CUSTOMER FOUND FOR THIS SEARCH" (com(01)) or "INVALID CUST NO OR CUSTOMER IS DELETED" (com(04)) if the customer is invalid.

4. **Invoice Retrieval and Display**:
   - Retrieves open invoices for the selected customer from `crdetx` (AR detail alternate index) and stores them in `crwork` (work file for open invoices).
   - Populates arrays (`ivno`, `ivd8`, `py`, `ds`, `cd`) with invoice details (e.g., invoice number, due date, amount due, payment amount, discount amount, payment code).
   - Displays up to 16 invoices on the screen, allowing users to enter payment codes (`P` for pay, `H` for hold, blank for no action) and amounts.

5. **Payment and Discount Processing**:
   - Users enter payment codes (`cd`) and amounts (`py`, `ds`) for each invoice.
   - The program validates:
     - Payment codes must be `P`, `H`, or blank (previously `J` for adjustment, but removed from the display file as of March 2024).
     - Payment amounts must not exceed the amount due.
     - Held invoices (`H`) cannot have payment amounts.
     - Discounts and payments must balance with the check’s gross amount.
   - Errors trigger messages like "INVALID CODE -- MUST BE P, J, OR H" (com(22)) or "PAY AMT CANNOT EXCEED AMT DUE" (com(28)).

6. **Check Processing**:
   - Stores check data in `crchks` (check work file), including customer key, check number (`kyckno`), check amount (`kyckam`), discount amount (`kydsam`), payment date (`kypydt`), gross amount (`kygrss`), and foreign currency amount (`kyfcam`).
   - Supports check application, deletion, and reactivation:
     - Adds check records via `crbld` output operation.
     - Deletes checks via `crdele` (marks records with `'D'`).
     - Updates checks via `crupd1` or `crupd2` (e.g., updating amounts or application status).
     - Reactivates checks using command key 9 (com(24), com(25)).

7. **File Updates**:
   - **arcont**: Updates the AR control file (`dtupdt` operation) with payment date information (`kypydt`).
   - **crwork**:
     - Adds invoice records via `wrkbld` (initial build with customer, invoice, and amount data).
     - Updates records via `wrkupd` (payment amount, discount, check number, etc.).
     - Activates records via `wrkact` (marks with `'A'`).
     - Clears records via `wrkclr`.
   - **crchks**: Manages check records as described above.

8. **Error Handling and Screen Redisplay**:
   - Validates inputs and displays errors (e.g., "CHECK HAS BEEN APPLIED" (com(07)), "INVALID DATE ENTERED" (com(15))).
   - Redisplays the screen for corrections if errors occur or if command keys (e.g., KA for rekey, KG for end of job) are used.

### Business Rules

The program enforces the following business rules for payment statement entry:

1. **Customer Validation**:
   - Customers must exist in `arcust` and not be deleted, or errors like "INVALID CUST NO OR CUSTOMER IS DELETED" (com(04)) are displayed.
   - If no balances are due for a customer, displays "NO BALANCES DUE FOR THIS CUSTOMER" (com(05)).

2. **Payment Code Validation**:
   - Valid codes are `P` (pay), `H` (hold), or blank (no action). The `J` (adjustment) code is disabled as of March 2024 (com(22), com(31), com(32)).
   - Held invoices (`H`) cannot have payment amounts (com(29)).
   - Invalid codes trigger "INVALID CODE -- MUST BE P, J, OR H" (com(22)).

3. **Payment and Discount Rules**:
   - Payment amounts (`py`) cannot exceed the invoice amount due (com(28)).
   - Applied amounts must balance with the check’s gross amount (com(08)).
   - Applied amounts must not be negative (com(21)).
   - A check amount or discount amount must be entered (com(09)).
   - Unapplied cash is flagged with "ALL OR PART OF PAYMENT IS UNAPPLIED CASH" (com(06)).

4. **Check Management**:
   - Checks can be applied, deleted, or reactivated (com(17), com(23), com(25)).
   - Deleted checks cannot be reviewed (com(11)), and records must exist for deletion (com(12), com(14)).
   - Applied checks are flagged (com(07)), and previously applied checks require review via command key 2 (com(33), com(34)).
   - Applied amounts must be zero when the check is fully applied (com(35)).

5. **Date Validation**:
   - Payment dates must be valid and non-blank (com(15), com(16)).

6. **GL Account Validation**:
   - GL accounts must be valid for non-blank or non-`H` codes (com(30)).
   - `J` codes (if enabled) require a valid GL account and amount (com(31), com(32)).

7. **Navigation and Commands**:
   - Supports roll-up/roll-down for invoice lists (indicators 18, 19).
   - Command key KA allows rekeying without adding/updating.
   - Command key KG ends the job.
   - Command key 9 reactivates deleted checks (com(24), com(25)).
   - Command key 2 reviews applied check details (com(34)).

8. **Notification of Difference**:
   - The notification field (`appds$`) must be blank or `'Y'` (com(36)).

### Tables (Files) Used

The program interacts with the following files:
1. **ar050d**:
   - Type: Workstation (display) file.
   - Format: `ar050fm`.
   - Usage: Displays customer data, invoice details, payment codes, amounts, discounts, and error messages. Supports roll-up/roll-down navigation.
   - Handler: Uses `PROFOUNDUI(HANDLER)` for UI integration.

2. **arcustx**:
   - Type: Input file (`if`), disk-based.
   - Record Length: 384 bytes.
   - Key: 18-byte external key (`extk`, keyloc(1)).
   - Usage: Alternate index for customer master, used for customer lookup.

3. **arcust**:
   - Type: Input file (`if`), disk-based.
   - Record Length: 384 bytes.
   - Key: 8-byte key (keyloc(2)).
   - Usage: Customer master file, provides core customer data (e.g., name, address).

4. **crdetx**:
   - Type: Input file (`if`), disk-based.
   - Record Length: 128 bytes.
   - Key: 23-byte external key (`extk`, keyloc(1)).
   - Usage: Alternate index for AR detail file, used to retrieve open invoices.

5. **arcont**:
   - Type: Update file (`uf`), disk-based.
   - Record Length: 256 bytes.
   - Key: 2-byte key (keyloc(2)).
   - Usage: AR control file, updated with payment date information (`kypydt`).

6. **gstabl**:
   - Type: Input file (`if`), disk-based.
   - Record Length: 256 bytes.
   - Key: 12-byte key (keyloc(2)).
   - Usage: General system table, likely provides configuration or reference data.

7. **crwork**:
   - Type: Update file (`uf a`), disk-based, allows additions.
   - Record Length: 96 bytes.
   - Key: 23-byte external key (`extk`, keyloc(1)).
   - Usage: Work file for open invoices, stores customer, invoice number, amount due, payment, and discount data. Supports add (`wrkbld`), update (`wrkupd`), activate (`wrkact`), and clear (`wrkclr`) operations.

8. **crchks**:
   - Type: Update file (`uf a`), disk-based, allows additions.
   - Record Length: 96 bytes.
   - Key: 14-byte key (keyloc(2)).
   - Usage: Work file for checks, stores check number, amount, discount, payment date, and application status. Supports add (`crbld`), delete (`crdele`), and update (`crupd1`, `crupd2`) operations.

### External Programs Called

The `AR050` RPGLE program does not explicitly call external programs within the provided code. It is a standalone program invoked by the `AR050.ocl36` script, which may call other programs (e.g., `AR050P`, `AR100`, `AR105`, `AR110`) in the broader workflow. The `rollsr` subroutine handles workstation file errors internally, and no external program calls are evident in the truncated code.

### Summary

The `AR050` RPGLE program is a core component of the cash receipts payment statement entry process. It:
- Manages user interaction via the `ar050d` display file, displaying customer and invoice data.
- Validates customer selection, payment codes (`P`, `H`, blank), payment amounts, discounts, and check data.
- Updates `arcont`, `crwork`, and `crchks` files to record payments and check applications.
- Enforces business rules for valid codes, amounts, dates, and GL accounts, with detailed error messaging.
- Supports navigation (roll-up/roll-down), check deletion/reactivation, and review of applied checks.

The program integrates with the `AR050.ocl36` script, which sets up the necessary files (`arcustx`, `arcust`, `arcont`, `gstabl`, `crdetx`, `crwork`, `crchks`) to enable efficient payment processing. The truncation of the code limits visibility into some subroutines, but the provided structure and comments outline a robust system for managing cash receipts.