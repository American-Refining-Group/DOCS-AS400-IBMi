### Process Steps of the AR135 RPG Program

The `AR135` RPG program is an interactive screen-based application for entering and managing Accounts Receivable (AR) cash receipts, specifically tailored for Electronic Funds Transfer (EFT) draft automation. It uses a workstation file (`AR135FM`) for user interaction across multiple screens (S1, S2, S3, S4) and processes data against various database files. The program supports entry mode (add new records) and update mode (edit existing records), with validation, cross-referencing, and totals calculation.

The code is a mix of original System/36-style RPG and converted ILE RPGLE (via TARGET/400 tool, dated 09/23/23), using fixed-format with modern enhancements like `EVAL` and `EXSR`. Here's a step-by-step breakdown of the main process flow, based on the program's structure:

1. **Program Initialization (`*INZSR` Subroutine)**:
   - Clears key fields: Company (`CO`), Customer (`CUST`), Invoice (`INV#`), and Customer Address (`AXCUST`).
   - Sets up initial state for entry, ensuring blanks or zeros for new transactions.

2. **Read Workstation Input (Main Loop)**:
   - Checks the control field (`QSCTL`): If blank, sets it to 'R' (read mode) and turns on indicator `*IN09` (initial display).
   - Reads the appropriate screen based on indicators:
     - `*IN81`: Reads `AR135S1` (header/search screen).
     - `*IN82`: Reads `AR135S2` (detail entry screen).
     - `*IN83`: Reads `AR135S3` (totals screen).
     - `*IN84`: Reads `AR135S4` (cross-reference/address screen).
   - If last record (`LR`), returns to caller.
   - Resets error indicators (`*IN90`, `*IN91`) and screen indicators (`*IN81-*IN84`).
   - Clears display fields like company name (`COMP NM`), customer name (`CUSTNM`), descriptions, and messages.

3. **Handle Function Keys and Mode Switches**:
   - **Exit (F3, `*IN03`)**: Switches to update mode (`*IN11` on, `*IN10` off) and redisplays S1.
   - **Initial Entry (F9, `*IN09`)**: Sets company from `DSCSCO` (default company), switches to update mode, and redisplays S1.
   - **Update Mode Toggle (F4, `*INKA`)**: Switches from entry to update mode and redisplays S1.
   - **Totals Calculation (F11, `*INKB`)**: Calls `TOTALS` subroutine to compute totals, then redisplays S3.
   - **Exit Program (F23, `*INKG`)**: Sets `*INLR = *ON` to end the program.
   - **Read Transaction (F12, `*INKJ`)**: Calls `RDTRAN` subroutine to read from `CRTRAN`, then switches to entry mode and displays S2.
   - **Add New (F24, `*INKK`)**: Resets sequence to 0, switches to update mode, and displays S1.

4. **Screen S1 Processing (Enter on S1, `*IN01` and `*IN11`)**:
   - Calls `S1` subroutine to validate and process header data (company, customer, invoice, etc.).
   - If successful, redisplays S1.

5. **Screen S2 Processing (Enter on S2, `*IN02` and `*IN10` for Add)**:
   - Calls `S2ADD` subroutine to add a new transaction record to `CRTRAN`.
   - Writes the record using `EADD` (effective add with key) or `E` (update).

6. **Screen S2 Update (Enter on S2, `*IN02` and `*IN11` for Update)**:
   - Calls `S2UPD` subroutine to update an existing transaction in `CRTRAN`.

7. **Cross-Reference/Address Processing (S4, `*IN84`)**:
   - Calls `CSR` (Cross-Reference Subroutine) to build address lines from customer data.
   - Calls `S4` subroutine to handle customer search and validation.
   - If search yields no results, displays error and allows re-entry.

8. **Date Validation (`DTEDIT` Subroutine)**:
   - Validates input date (`DATE`) in MMDDYY format.
   - Breaks down into month/day/year, checks for valid month (1-12), day (accounting for leap years using Y2K logic).
   - Handles leap year: Divides year by 4 (or by 400 for century years using `Y2KCMP` and `Y2KCEN`).
   - Sets `*IN79` on error; otherwise, proceeds.

9. **Transaction Read (`RDTRAN` Subroutine)**:
   - Reads from `CRTRAN` using key (company, customer, sequence).
   - Handles duplicate keys by redefining fields as alphanumeric for comparison.
   - Loads data into screen fields.

10. **Output and Display**:
    - Displays screens with messages (e.g., "Sequence Not Found", "Invalid Date").
    - S3 shows running totals (`TOTAMT`, `TOTDIS`, `TOTMIS`).
    - S4 populates address lines (`CSAR` array).

11. **End of Program**:
    - On `*INLR`, returns control to the calling OCL program.

The program loops through reads and writes until exit, with subroutines handling most logic for modularity.

---

### Business Rules

The program enforces several business rules for data integrity, validation, and EFT/cash receipts processing. These are derived from field validations, indicators, and subroutines:

1. **Mode Switching**:
   - Starts in entry mode (`*IN10` on) for adding new receipts; toggles to update mode (`*IN11` on) via F3/F4/F9/F24.
   - Prevents adding if sequence exists (error: "Seq No Is Already Used, CMD 10 To Add").

2. **Key Field Validation**:
   - Company (`CO`): Must be valid (error if invalid: "Invalid Company Number").
   - Customer (`CUST`): Must exist in `ARCUST`/`ARCUSTX`; cross-references name/abbreviation (error: "Invalid Customer Number", "Name Search Data - Not Found").
   - Invoice (`INV#`): Cannot be blank for non-misc cash (error: "Invoice Number May Not Be Blank").
   - Sequence (`SEQ`): Auto-increments; cannot reuse existing (checks `CRTRAN` for duplicates).

3. **Transaction Rules**:
   - Type (`TYPE`): 'P' for cash receipts/misc cash; misc cash cannot have discount (`DISC > 0` triggers error: "Misc Cash cannot have Discount Amount").
   - Amount (`AMT`): Must be > 0 (error: "Invalid Amount, Must Not Be = To Zero").
   - Discount (`DISC`): Valid only for non-misc; links to discount G/L (`GLDI`).
   - G/L Accounts: Debit (`GLDR`), Credit (`GLCR`), Discount (`GLDI`) must exist in `GLMAST` (errors for invalid accounts).
   - Intercompany: Credit/Debit companies (`COCR`, `CODR`) must be valid.
   - Notification of Difference (`NOD`): Must be blank or 'Y' (error otherwise).

4. **Date Handling**:
   - Transaction Date (`DATE`), Due Date (`DUDT`): MMDDYY format; validated for month/day/leap year (using Y2K century/month params).
   - Error on invalid dates (e.g., Feb 30, non-leap Feb 29).

5. **Customer and Address**:
   - Searches `ARCUSTX` by name abbreviation; builds 11-line address array (`CSAR`).
   - Salesman (`SLSM`) from `ARCUST`.
   - Terms (`TERM`) and due date from `ARDETL`.

6. **Financial Rules**:
   - Totals: Calculates total amount (`TOTAMT`), discount (`TOTDIS`), misc (`TOTMIS`) from detail lines.
   - AR Detail Update: Applies payments to `ARDETL` (partial `ADPART`, current month `ADPAY`).
   - Deletion: Marks records with 'D' in `ATDEL`; prevents processing deleted sequences (error: "This Sequence Was Previously Deleted" or "Record Marked For Deletion").

7. **Error Handling and Messages**:
   - Uses 28 predefined messages (e.g., "Press Enter To Add/Update", "Record Activated").
   - Displays on screens; requires user acknowledgment.

8. **EFT-Specific**:
   - Designed for EFT draft automation (per comments); handles individual entries with intercompany transfers and G/L postings.

These rules ensure accurate AR postings, prevent duplicates/errors, and comply with financial standards.

---

### Tables/Files Used

The program uses the following physical and logical files (all IBM i-style DDS definitions, with keys and usage):

| File Name | Type | Record Length | Key Location | Usage/Description |
|-----------|------|---------------|--------------|-------------------|
| **AR135FM** | Display File (CF) | 900 | N/A | Workstation SFL for screens S1-S4 (header, detail, totals, address). |
| **CRTRAN** | Update File (UF A) | 256 | 2 (Company, Customer, Invoice?) | EFT transaction workfile; stores receipts (amount, disc, type, date, G/L, desc). |
| **CRTRANR** | Input File (IF) | 256 | 2 | Reference transaction file (for reads/comparisons; includes old seq, misc amounts). |
| **ARCUST** | Input File (IF) | 384 | 2 | Customer master; name, salesman (`ARSLS#`). |
| **ARCUSTX** | Input File (IF) | 384 | 1 | Customer index/extension; address lines (`AXADR1/2`), name abbr (`CXNM`). |
| **ARCONT** | Input File (IF) | 256 | 2 | AR control; company name (`ACNAME`), G/L accounts (AR `ACARGL`, disc `ACDSGL`, cash `ACCSGL`). |
| **GLMAST** | Input File (IF) | 256 | 2 | G/L master; account descriptions (`GLDESC`). |
| **ARDETL** | Input File (IF) | 128 | 2 | AR detail; amounts (`ADAMT`, partial `ADPART`, paid `ADPAY`), due date (`ADDUDT`), terms (`ADTERM`), no diff (`ADNOD`). |

Files use packed decimals (P) for amounts, zoned for dates/numbers. All are keyed for efficient access; shared in OCL caller.

---

### External Programs Called

The RPG program `AR135` does not explicitly call any other external programs or procedures via `CALL` or similar statements. All processing is internal via subroutines (`EXSR`):
- `S1`, `S2ADD`, `S2UPD`, `CSR`, `S4`, `DTEDIT`, `RDTRAN`, `TOTALS`, `*INZSR`.

It is invoked by the OCL program (`AR135.ocl36.txt` via `LOAD AR135` and `RUN`), which handles file opens and environment setup. No outbound calls; returns control on `*INLR`.