### Process Steps of the AR100 RPGLE Program

The `AR100.rpgle.txt` program is an RPGLE (RPG IV) application designed for individual cash receipts entry in an accounts receivable (AR) system on an IBM i (AS/400) or System/36 environment. It has been converted from older RPG II/III code using the TARGET/400 tool (as noted in the header comments). The program is invoked by the `AR100.ocl36` OCL script, which sets up the environment and files. It uses a workstation display file for interactive user input, allowing entry of single cash receipt transactions (e.g., payments applied to specific invoices or miscellaneous cash). The program handles data validation, GL account checks, and file updates. Due to the truncation in the provided code, the explanation focuses on the visible logic, including initialization, screen processing, validation subroutines, and output operations. Here's a step-by-step breakdown:

1. **Program Initialization (`*inzsr` Subroutine)**:
   - Clears key fields: Sets `co` (company number), `cust` (customer number), `inv#` (invoice number), and `axcust` (auxiliary customer number) to zero.
   - This ensures a clean starting state for each transaction entry, preventing carryover from previous runs.

2. **Workstation File Read and Control Logic**:
   - Reads the display file `ar100d` (format `ar100fm`) to capture user input.
   - Uses `qsctl` (likely a control field in the display file) to determine processing mode:
     - If blank (first-time display), sets indicators for initial screen display (e.g., `*in81` for first-time processing).
     - Otherwise, processes user-entered data (e.g., sequence number `seq`, company `co`, customer `cust`, invoice `inv#`, amount `amt`, discount `disc`, date `date`, GL accounts `gldr` (debit), `glcr` (credit), `gldi` (discount), description `desc`, and notification of difference `nod`).
   - Supports multiple screen formats:
     - `AR100S1`: Initial entry screen for sequence number.
     - `AR100S2`: Full entry screen with customer details, invoice info, amounts, dates, GL accounts, and descriptions.
     - `AR100S3`: Summary screen showing totals (`totamt`, `totdis`, `totmis` for total amount, discount, and miscellaneous).
   - Displays function keys (e.g., F10 for Add Mode, F11 for Update Mode) and error messages (`msg1`, `msg2`).

3. **Sequence Number Processing**:
   - Uses the sequence number (`seq`) to read or chain to the transaction file `crtranr` (read-only view).
   - If the sequence exists (`n97` - not found indicator off), loads existing data (e.g., company, customer, invoice, amounts, GL accounts).
   - If not found, prompts to add a new record (e.g., message "Enter CMD 10 To Add" - msg(2)).
   - Handles deletion: If marked for deletion (`atdel = 'D'`), displays "Record Marked For Deletion" (msg(3)) and allows reactivation.

4. **Customer and Invoice Validation**:
   - Chains to `arcust` using `cust` to validate the customer:
     - Retrieves customer name (`arname`), salesman number (`arsls#`), and checks deletion flag (`ardel`).
     - If invalid, displays "Invalid Customer Number" (msg(10)).
   - Chains to `ardetl` using the composite key `arrdky` (company + customer + invoice) to validate the invoice:
     - Retrieves due date (`dudt`), terms (`atterm`), and other details.
     - If invalid or blank invoice, displays "Invalid Invoice To Pay" (msg(24)) or "Invoice Number May Not Be Blank" (msg(27)).
   - For miscellaneous cash (no invoice), sets `type` to a specific value and flags "Miscellaneous Cash" (msg(25)), but prohibits discount amounts (msg(16): "Misc Cash cannot have Discount Amount").

5. **Date Validation Subroutine (`@enddt` or Date Edit Logic)**:
   - Validates transaction date (`date`) and due date (`dudt`):
     - Checks for invalid dates (e.g., day > 31, invalid month/year).
     - Uses Y2K fields (`y2kcen`, `y2kcmp`) for century and millennium processing.
     - If invalid, sets error indicator `*in79` and displays "Invalid Transaction Date" (msg(13)).
     - Converts dates to 8-digit format (`date8`, `dudt8`) for storage.
   - Ensures dates are non-blank and logical (e.g., transaction date not in the future beyond system limits).

6. **GL Account Validation**:
   - Chains to `glmast` for debit (`gldr`), credit (`glcr`), and discount (`gldi`) accounts using a composite key (company + GL account).
     - Checks deletion/inactive flag (`gldel`): Treats inactive (`'I'`) the same as deleted (`'D'`) per JB01 update (08/24/2016).
     - Retrieves descriptions (`gldesc`) for display (`drdesc`, `crdesc`, `dsdesc`).
     - If invalid, displays errors like "Invalid Debit G/L Account" (msg(15)), "Invalid Credit G/L Account" (msg(14)), or "Invalid Discount G/L Account" (msg(23)).
   - Retrieves defaults from `arcont` (e.g., `dsarco` for AR company, `dsargl` for AR GL, `dscsgl` for cash GL, `dsdsgl` for discount GL).
     - Applies defaults if GL fields are zero (e.g., credit GL defaults to AR GL if applicable).
   - Validates company numbers (`codr`, `cocr`, `codi`): Ensures valid intercompany codes (msg(19), msg(21), msg(22)).

7. **Amount and Discount Validation**:
   - Checks `amt` > 0 (msg(11): "Invalid Amount, Must Not Be = To Zero").
   - For discounts: Ensures `disc` = 0 for miscellaneous cash; validates against invoice terms.
   - Notification of difference (`nod`) must be blank or `'Y'` (msg(28): "Notif. of Diff. must be Blank or 'Y'").
   - Calculates totals for display on `AR100S3`.

8. **File Output and Update**:
   - **Add Operation (`eadd 50`)**: Adds a new record to `crtran` with fields like `seq`, `co`, `cust`, `inv#`, `amt`, `disc`, `type`, `date`, GL accounts, `desc`, `dudt`, `term`, `slsm` (salesman), and 8-digit dates.
   - **Update Operation (`e 51`)**: Updates an existing record with similar fields; sets deletion flag to `'D'` if needed (`n97 kd` logic).
   - Displays confirmation: "Press Enter To Add/Update" (msg(26)) or "Record Activated" (msg(4)).
   - Clears error messages and redisplays the screen if validation fails.

9. **Error Handling and Screen Redisplay**:
   - Uses indicators (e.g., `*in90` for customer errors, `*in33` for customer invalid, `*in31` for sequence used) to control flow.
   - Displays up to two messages (`msg1`, `msg2`) from the `msg` array (28 predefined messages).
   - Loops back to read the screen until valid or end-of-job (e.g., command key for exit).

10. **Program Termination**:
    - Sets `*inLR = *on` after successful add/update or user exit.

### Business Rules

The program enforces the following business rules for individual cash receipts entry:

1. **Transaction Sequencing**:
   - Each transaction must have a unique sequence number (`seq`). If duplicate, error "Seq No Is Already Used, CMD 10 To Add" (msg(7)).
   - Sequence not found prompts for add mode (F10); existing sequences default to update mode (F11).

2. **Customer and Invoice Requirements**:
   - Customer must exist and not be deleted (`ardel <> 'D'`); otherwise, "Invalid Customer Number" (msg(10)).
   - Invoice must be valid and open (exists in `ardetl`); blank invoices are allowed only for miscellaneous cash (`type` set accordingly), but no discounts permitted (msg(16), msg(24), msg(27)).

3. **Amount and Discount Constraints**:
   - Amount (`amt`) must be greater than zero (msg(11)).
   - Discounts (`disc`) are prohibited for miscellaneous cash (msg(16)).
   - Payment cannot exceed invoice amount due (implied in validation, though not explicitly shown in truncated code).

4. **Date Rules**:
   - Transaction date (`date`) and due date (`dudt`) must be valid (day ≤ 31, valid month/year) and non-blank (msg(13)).
   - Uses Y2K-compliant century/month processing (`y2kcen`, `y2kcmp`) to handle dates post-2000.

5. **GL Account Validation**:
   - Debit (`gldr`), credit (`glcr`), and discount (`gldi`) accounts must exist in `glmast` and not be deleted (`gldel <> 'D'`) or inactive (`gldel <> 'I'`)—inactive treated as deleted per JB01 (msg(14), msg(15), msg(23)).
   - Defaults from `arcont`: Credit GL to AR GL (`dsargl`), debit to cash GL (`dscsgl`), discount to discount GL (`dsdsgl`).
   - Intercompany codes (`codr`, `cocr`, `codi`) must be valid (msg(19), msg(21), msg(22)).

6. **Company and Description Rules**:
   - Company number (`co`) must be valid (msg(9)).
   - Description (`desc`) is mandatory and limited to 25 characters (`desc` DS overlay).
   - Notification of difference (`nod`) must be blank or `'Y'` (msg(28)).

7. **Deletion and Activation**:
   - Records can be marked for deletion (`'D'`) but not reviewed if deleted (implied). Reactivation clears the flag and displays "Record Activated" (msg(4)).
   - Previously deleted sequences cannot be reused without reactivation (msg(5)).

8. **Miscellaneous Cash**:
   - Allowed without invoice, but flagged as "Miscellaneous Cash" (msg(25)) and no discount permitted.

9. **User Interface Rules**:
   - Supports add/update modes via function keys (F10/F11).
   - Errors trigger screen redisplay with messages; totals shown on summary screen.

### Tables (Files) Used

The program interacts with the following files (all prefixed with `?9?` from the OCL script):
1. **ar100d**:
   - Type: Workstation (display) file (`cf e` - combined file extended).
   - Usage: Interactive screen for input/output (formats `AR100S1`, `AR100S2`, `AR100S3`). Displays entry fields, totals, messages, and function key prompts. Uses Profound UI handler for modern rendering.

2. **crtranr**:
   - Type: Input file (`if`), disk-based, logical view.
   - Record Length: 256 bytes.
   - Key: 5-byte arrival sequence key (keyloc(2)).
   - Usage: Read-only access to transaction records for validation/loading existing data.

3. **crtran**:
   - Type: Update file (`uf a`), disk-based, allows additions.
   - Record Length: 256 bytes.
   - Key: 5-byte arrival sequence key (keyloc(2)).
   - Usage: Adds/updates cash receipt transactions (fields like `atco`, `atcust`, `atinv#`, `atamt`, `atdisc`, `atdate`, GL accounts, description).

4. **arcust**:
   - Type: Input file (`if`), disk-based.
   - Record Length: 384 bytes.
   - Key: 8-byte customer key (keyloc(2)).
   - Usage: Validates customers; retrieves name (`arname`), deletion flag (`ardel`), salesman (`arsls#`).

5. **arcont**:
   - Type: Input file (`if`), disk-based.
   - Record Length: 256 bytes.
   - Key: 2-byte company key (keyloc(2)).
   - Usage: Provides company name (`acname`) and default GL accounts (e.g., `dsargl`, `dscsgl`, `dsdsgl`).

6. **glmast**:
   - Type: Input file (`if`), disk-based.
   - Record Length: 256 bytes.
   - Key: 11-byte GL key (company + account, keyloc(2)).
   - Usage: Validates GL accounts; retrieves deletion flag (`gldel`) and description (`gldesc`).

7. **ardetl**:
   - Type: Input file (`if`), disk-based.
   - Record Length: 128 bytes.
   - Key: 18-byte detail key (company + customer + invoice, keyloc(2)).
   - Usage: Validates invoices; retrieves due date (`atdudt`), terms (`atterm`).

### External Programs Called

The `AR100` RPGLE program does not explicitly call any external programs or procedures within the provided code. It operates as a standalone interactive application, relying on the invoking OCL script (`AR100.ocl36`) for environment setup (e.g., `SCPROCP`, `GSY2K`). All processing, including validations and file I/O, is handled internally via RPGLE operations (e.g., `chain`, `eadd`, `e`). The Profound UI handler (`PROFOUNDUI(HANDLER)`) is used for UI rendering but is not an external program call.

### Summary

The `AR100` program provides an interactive screen for entering individual cash receipts, validating customers, invoices, dates, amounts, and GL accounts against master files. It supports add/update modes, deletion/reactivation, and miscellaneous cash entries while enforcing strict rules for data integrity (e.g., no discounts on misc cash, inactive GLs treated as deleted). Integrated with the OCL script, it focuses on single-transaction processing in a multi-user AR environment, with error-driven screen redisplay for user guidance. The truncation limits visibility into some loops or totals calculations, but the core flow emphasizes validation before file updates.