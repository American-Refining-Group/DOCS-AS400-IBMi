The provided document, `AR101.rpgle.txt`, is an RPGLE (RPG IV) program for the IBM AS/400 (iSeries) platform, converted from a System/36 RPG program using the TARGET/400 tool. It is called from the main OCL program (`AR101.ocl36.txt`) to handle Accounts Receivable (A/R) transaction entry. Below, I will explain the process steps, business rules, tables (files) used, and any external programs called.

---

### **Process Steps of the AR101 RPGLE Program**

The `AR101` program is designed to facilitate A/R transaction entry through a workstation interface, allowing users to add or update transactions such as invoices (`I`) or adjustments (`J`). It performs validations, interacts with multiple files, and supports date calculations for Y2K compliance. Here’s a step-by-step breakdown of the process:

1. **Initialization**:
   - The program initializes key variables (e.g., `axcust`, `co`, `cust`, `inv#`) to zero in the `*inzsr` subroutine.
   - It sets up the workstation file (`ar101d`) for screen interaction, using the Profound UI handler for modernized UI rendering.
   - Data structures and arrays (e.g., `msg`, `tabmon`, `tabdys`) are defined for messages, month/day validations, and customer data.

2. **Workstation File Read and Mode Determination**:
   - The program checks the control field `qsctl`. If blank, it sets indicator `*in09` to initiate a read operation; otherwise, it reads one of three screen formats (`ar101s1`, `ar101s2`, `ar101s3`) based on indicators `*in81`, `*in82`, or `*in83`.
   - Two modes are supported:
     - **Entry Mode** (`*in10`): For adding new transactions.
     - **Update Mode** (`*in11`): For updating existing transactions.
   - Indicators `*in81`, `*in82`, and `*in83` control which screen format is displayed (`AR101S1`, `AR101S2`, `AR101S3`).

3. **Command Key Processing**:
   - The program processes function keys (`*inka` to `*inkk`):
     - **F1 (`*inka`)**: Switches to Update Mode (`*in11`).
     - **F2 (`*inkb`)**: Calls the `totals` subroutine to calculate and display total amounts, then displays screen `AR101S3`.
     - **F7 (`*inkg`)**: Sets `*inlr` to terminate the program.
     - **F10 (`*inkj`)**: Initiates Entry Mode (`*in10`), sets transaction type to `I` (invoice), and calls `rdtran` to get the next sequence number.
     - **F11 (`*inkk`)**: Switches to Update Mode, resets sequence number, and displays screen `AR101S1`.

4. **Screen Processing**:
   - **Screen 1 (`AR101S1`, `*in01` and `*in11`)**:
     - Executes the `s1` subroutine to retrieve and validate a transaction by sequence number (`seq`) from `artran`.
     - If the record is not found (`*in90`), it clears fields and displays error messages (`msg(1)`, `msg(2)`).
     - If the record is marked as deleted (`atdel = 'D'`), it sets `*in91` and displays `msg(5)`.
     - Calls `move` to populate screen fields and `s2edit` for validations, then displays screen `AR101S2`.
   - **Screen 2 (`AR101S2`, `*in02`)**:
     - In **Entry Mode** (`*in10`): Executes `s2add` to validate and add a new transaction. If valid, it writes to `artran` with `*in50`, increments the sequence number, clears fields, and redisplays `AR101S2`.
     - In **Update Mode** (`*in11`): Executes `s2upd` to validate and update an existing transaction. If valid, it updates `artran` with `*in51`, resets the sequence number, and displays `AR101S1`.
   - **Screen 3 (`AR101S3`, `*in03`)**: Displays total amounts calculated by the `totals` subroutine.

5. **Transaction Validation (`s2edit` Subroutine)**:
   - Validates input fields for both entry and update modes:
     - **Sequence Number**: Ensures the sequence number is unique (`*in31`).
     - **Transaction Type**: Must be `I` (invoice) or `J` (adjustment) (`*in31`).
     - **Company Number**: Validates against `arcont` (`*in32`) and retrieves company name.
     - **Customer Number**: Validates against `arcust` (`*in33`, `*in47`) and checks for deletion status.
     - **Invoice Number**: Ensures it’s non-zero (`*in34`) and checks for existence in `ardetl` for invoices (`I`) or non-existence for adjustments (`J`).
     - **Amount**: Must be non-zero (`*in35`).
     - **Transaction Date**: Validates format and sets to current date if zero (`*in36`). Calls `dtedit` for date validation.
     - **Due Date**: For invoices, calculates due date using `clcdue` based on terms code (`*in37`, `*in38`).
     - **Salesman**: Validates against `gstabl` (`*in39`) or defaults from `arcust`.
     - **G/L Accounts**: Validates debit (`gldr`, `*in41`) and credit (`glcr`, `*in43`) accounts against `glmast`, checking for deletion or inactive status (`'D'` or `'I'`).
     - **Debit/Credit Companies**: Validates against `arcont` (`*in40`, `*in42`).
   - If any validation fails, sets `*in90`, displays an error message, and redisplays `AR101S2`.

6. **Due Date Calculation (`clcdue` Subroutine)**:
   - If terms code is zero, uses customer’s terms code (`arterm`) or defaults to 30 days.
   - Validates terms code against `gstabl` (`*in38`).
   - Calculates due date based on net days (`tbnetd`) or prox days (`tbprxd`) using `tmdatn` or `tmdatp` subroutines.
   - Ensures due date is not earlier than transaction date (`*in37`).

7. **G/L Account Defaults (`invgl` Subroutine)**:
   - Sets default G/L accounts if not provided:
     - Debit account (`gldr`): Uses `acargl` (A/R G/L) or inter-company number (`arintr`).
     - Credit account (`glcr`): Uses `acslgl` (sales G/L) or inter-company number.
     - Sets debit/credit company numbers (`codr`, `cocr`) to transaction company (`co`) if not specified.

8. **Sequence Number Retrieval (`rdtran` Subroutine)**:
   - Reads `artranr` to find the last sequence number, adds 10 to it, and clears fields for a new transaction.

9. **Total Calculation (`totals` Subroutine)**:
   - Reads `artranr` sequentially, summing non-deleted transaction amounts (`atamtr`) to display on `AR101S3`.

10. **Date Handling**:
    - The program handles Y2K compliance using `y2kcen` (century) and `y2kcmp` (comparison year).
    - Subroutines `@dte1` and `@dte2` convert between Gregorian and Julian dates for due date calculations.
    - The `dtedit` subroutine validates dates, checking for valid months, days, and leap years.

11. **File Updates**:
    - New transactions are added to `artran` (`*in50`).
    - Existing transactions are updated in `artran` (`*in51`).
    - Deleted records are marked with `atdel = 'D'` or cleared (`' '`).

12. **Screen Output**:
    - Writes to `ar101s1`, `ar101s2`, or `ar101s3` based on the active screen and mode, displaying fields like sequence number, company, customer, invoice number, amount, dates, and G/L accounts.

---

### **Business Rules**

The program enforces the following business rules for A/R transaction entry:

1. **Transaction Types**:
   - Transactions must be either invoices (`I`) or adjustments (`J`). Invalid types trigger error `msg(3)`.

2. **Sequence Number**:
   - Each transaction requires a unique sequence number. Duplicates trigger `msg(7)`.
   - Sequence numbers increment by 10 for new transactions.

3. **Company Validation**:
   - Company number (`co`) must exist in `arcont`. Invalid companies trigger `msg(9)`.
   - Debit (`codr`) and credit (`cocr`) companies must also exist in `arcont` (`msg(22)`, `msg(21)`).

4. **Customer Validation**:
   - Customer number (`cust`) must exist in `arcust` and not be deleted (`ardel = 'D'`). Invalid customers trigger `msg(10)`.
   - If customer is zero, displays `msg(25)` ("MISCELLANEOUS CASH").
   - Inter-company customers are flagged with `iccd = 'IC'`.

5. **Invoice Number**:
   - Invoice number (`inv#`) must be non-zero (`msg(27)`).
   - For invoices (`I`), the invoice must not already exist in `ardetl` (`msg(6)`).
   - For adjustments (`J`), the referenced invoice must exist in `ardetl` (`msg(23)`).

6. **Amount**:
   - Transaction amount (`amt`) must be non-zero (`msg(11)`).

7. **Transaction Date**:
   - Must be valid (checked by `dtedit`). Invalid dates trigger `msg(13)`.
   - If zero, defaults to the current date (`udate`).

8. **Due Date**:
   - For invoices, due date (`dudt`) is calculated based on terms code (`term`) or defaults to 30 days.
   - Must be valid and not earlier than the transaction date (`msg(16)`).
   - Terms code must exist in `gstabl` (`msg(17)`).

9. **Salesman**:
   - Salesman number (`sls`) defaults from `arcust` (`arsls#`) if not provided.
   - Must exist in `gstabl` (`msg(18)`).

10. **G/L Accounts**:
    - Debit (`gldr`) and credit (`glcr`) accounts must exist in `glmast` and not be deleted (`gldel = 'D'`) or inactive (`gldel = 'I'`) (`msg(15)`, `msg(14)`).
    - Defaults are set from `arcont` (`acargl`, `acslgl`) or inter-company number (`arintr`).

11. **Y2K Compliance**:
    - Dates are adjusted for century using `y2kcen` and `y2kcmp`.
    - Julian date conversions ensure accurate due date calculations.

12. **Deletion Handling**:
    - Deleted transactions are marked with `atdel = 'D'` and flagged with `msg(5)`.
    - Inactive G/L accounts are treated as deleted (per modification `JB01`).

13. **Error Handling**:
    - Validation errors set `*in90`, display an error message, and redisplay `AR101S2` for correction.
    - Messages are stored in the `msg` array (e.g., `msg(1)` to `msg(44)`).

---

### **Tables (Files) Used**

The program interacts with the following files:

1. **ar101d** (Workstation File):
   - Screen file for user interaction, with formats `AR101S1`, `AR101S2`, and `AR101S3`.
   - Uses Profound UI handler for rendering.

2. **artran** (Update File, `UF A`):
   - A/R transaction file (256 bytes, 5-byte key at position 2).
   - Stores transaction details (e.g., `atco`, `atcust`, `atinv#`, `atamt`, `atdate`).
   - Used for adding (`*in50`) and updating (`*in51`) transactions.

3. **artranr** (Input File, `IF`):
   - Read-only A/R transaction file (256 bytes, 5-byte key at position 2).
   - Used to retrieve sequence numbers and calculate totals.

4. **arcust** (Input File, `IF`):
   - Customer master file (384 bytes, 8-byte key at position 2).
   - Contains customer data (e.g., `arname`, `arsls#`, `arterm`, `arintr`).

5. **arcont** (Input File, `IF`):
   - A/R control file (256 bytes, 2-byte key at position 2).
   - Stores company details (e.g., `acname`, `acargl`, `acslgl`, `acdsgl`, `accsgl`).

6. **glmast** (Input File, `IF`):
   - General ledger master file (256 bytes, 11-byte key at position 2).
   - Contains G/L account details (e.g., `gldesc`, `gldel`).

7. **ardetl** (Input File, `IF`):
   - A/R detail file (128 bytes, 18-byte key at position 2).
   - Stores detailed A/R records (e.g., `adamt`, `addudt`, `adterm`, `adrfiv`).

8. **gstabl** (Input File, `IF`):
   - General system table (256 bytes, 12-byte key at position 2).
   - Contains terms and salesman data (e.g., `tbnetd`, `tbprxd`).

---

### **External Programs Called**

The `AR101` RPGLE program does not explicitly call any external programs. All processing is handled within the program using subroutines (e.g., `s1`, `s2add`, `s2upd`, `s2edit`, `clcdue`, `invgl`, `rdtran`, `totals`, `dtedit`, `@dte1`, `@dte2`, `rdcust`, `tmdatp`, `tmdatn`).

---

### **Summary**

The `AR101` RPGLE program, called from the OCL program, provides a user interface for entering and updating A/R transactions (invoices and adjustments). It enforces strict business rules for data validation, including checks on company, customer, invoice, amount, dates, salesman, and G/L accounts. The program uses multiple files (`artran`, `artranr`, `arcust`, `arcont`, `glmast`, `ardetl`, `gstabl`) to retrieve and store data, and handles Y2K-compliant date calculations. No external programs are called, and all logic is contained within subroutines. The program’s robust error handling ensures data integrity, with clear error messages guiding users to correct invalid inputs.