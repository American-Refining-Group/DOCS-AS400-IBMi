The provided document is an RPGLE (Report Program Generator Language Enhanced) program named `AR900.rpgle.txt`, which is called from the OCL script `AR900.ocl36.txt` for **Customer Master Add/Update** functionality on an IBM midrange system (likely IBM System/36 or AS/400, now IBM i). Below, I will explain the **process steps**, **business rules**, **tables used**, and **external programs called** by this RPGLE program.

---

### **Process Steps of the RPGLE Program**

The `AR900` RPGLE program manages the maintenance (add, update, delete, reactivate) of customer master records through a workstation display file (`AR900D`). It interacts with multiple files and performs validations, updates, and history tracking. The process steps are structured around a main loop that displays screen formats and processes user inputs.

1. **Program Initialization (`ONETIM` Subroutine)**:
   - **Purpose**: Sets up initial variables and constants for the program.
   - **Steps**:
     - Initializes zero-filled variables (`ZERO5`, `ZERO6`, etc.) for calculations and comparisons.
     - Sets key table lookup values (`SLKEY` for salesman, `TRMKEY` for terms, `CUCLKY` for customer class, `GRUPKY` for group code).
     - Captures the system date (`UDATE`) and time (`SYSTIM`) for history file updates, formatting the date as `SYSCYM` (e.g., `20YYMMDD`).
     - Sets default values for company (`CO`) and customer number (`CUSN`) to zero and enables the initial screen format (`FMT01`) by setting indicator `81`.

2. **Main Processing Loop**:
   - **Purpose**: Continuously displays and processes screen formats (`FMT01`, `FMT02`, `FMT03`, `FMT05`) until the user exits.
   - **Steps**:
     - Enters a `DO` loop controlled by `fmtagn` (set to `*ON` initially).
     - Clears error indicators (`01`, `02`, `03`, `05`, `09`) and message fields (`MSG1`, `MSG2`).
     - Selects and displays the appropriate screen format based on indicators:
       - `FMT01` (indicator `81`): Initial customer selection screen.
       - `FMT02` (indicator `82`): Customer detail entry/update screen.
       - `FMT03` (indicator `83`): Supplemental data screen.
       - `FMT05` (indicator `85`): Additional maintenance screen.
     - Processes user input based on the selected screen and command keys (e.g., `KA`, `KD`, `KG`, `KJ`, `KK`).

3. **Command Key Processing**:
   - **KA (Rekey, No Add/Update)**:
     - Clears input fields using the `CLEAR` subroutine.
     - Sets appropriate screen indicators (`81` for update mode, `82` for entry mode).
     - Resets error and format indicators.
   - **KD (Delete/Reactivate)**:
     - If update mode (`07`) is active, checks if the customer can be deleted (subroutine `DELETE`) or reactivated (subroutine `REACTI`).
     - Prevents deletion if the customer has outstanding invoices (non-zero balance).
   - **KG (End of Job)**:
     - Sets the Last Record indicator (`LR`) to exit the program.
     - Clears format indicators and turns off `fmtagn` to exit the main loop.
   - **KJ (Entry Mode Selected)**:
     - Clears fields, sets entry mode (`08`), and displays `FMT02` for new customer entry.
   - **KK (Update Mode Selected)**:
     - Clears fields, sets update mode (`07`), and displays `FMT02` for editing an existing customer.

4. **Screen Processing Subroutines**:
   - **S1 (Customer Selection, `FMT01`)**:
     - Validates the company number (`CO`) against `ARCONT`. If invalid, displays an error (`MSG(1)`).
     - Retrieves customer data from `ARCUST` and `ARCUSP` using the key `ARKEY` (company + customer number).
     - Checks for EFT (Electronic Funds Transfer) data if `AREFT = 'Y'`, ensuring bank routing (`CSARTE`) and account (`CSABK#`) are not blank.
     - Retrieves billing instructions from `BICONT` and sets the screen header (`S4HEAD`) based on `BCINST`.
     - Calls `GETCUS` and `GETSUP` to populate screen fields with customer and supplemental data.
     - Transitions to `FMT02` (indicator `82`).
   - **S2 (Customer Detail Entry/Update, `FMT02`)**:
     - Validates company and customer numbers for new entries (`08` mode).
     - Ensures customer number is non-zero and not already in use (for new entries).
     - Calls `SC2EDT` to validate input fields (e.g., sort name, salesman, terms, state, statement code).
     - Transitions to `FMT03` (indicator `83`) if validations pass.
   - **S3 (Supplemental Data, `FMT03`)**:
     - Validates and adjusts date fields (`STDATE`, `FSDATE`, `ICDATE`) for century (Y2K compliance).
     - Calls `@DTEDT` to validate dates (month, day, leap year checks).
     - Calls `BI907AC` to maintain `ARCUPR` (customer pricing) data.
     - Transitions to `FMT05` (indicator `85`).
   - **S5 (Add/Update Customer, `FMT05`)**:
     - Updates or adds records to `ARCUST` and `ARCUSP` based on mode (`07` for update, `08` for add).
     - Updates grouped customer records in `ARCLGR` to reflect credit problem status (`ARCLPB`).
     - Writes history records to `ARCUSHS` and `ARCUSPH`.
     - Clears fields and returns to `FMT01`.

5. **Data Validation (`SC2EDT` Subroutine)**:
   - Validates key fields:
     - **Sort Name (`LSTN`)**: Must not be blank.
     - **Salesman Number (`SLS#`)**: Must exist in `GSTABL`.
     - **Terms Code (`TERM`)**: Must exist in `GSTABL`.
     - **State (`STAT`)**: Must not be blank.
     - **Statement Code (`STMT`)**: Must be `Y`, `N`, or blank.
     - **Separate Freight Code (`SFRT`)**: Must be `Y`, `N`, or blank.
     - **Finance Charge Code (`FINC`)**: Must be `Y`, `N`, or blank.
     - **Freight Code (`FRCD`)**: Must be `C` (collect), `P` (prepaid), `A` (prepaid & add), or blank.
     - **EFT Participant (`EFT`)**: Must be `Y`, `N`, or blank.
     - **EDI Participant (`EDI`)**: Must be `Y`, `N`, or blank.
     - **Credit Problem Code (`CLPB`)**: Must be `Y`, `N`, or blank.
     - **Wire Transfer Code (`WIRE`)**: Must be `Y`, `N`, or blank.
     - **Customer Class (`CCLASS`)**: Must exist in `GSTABL` under table type `ARCUCL`.
     - **Federal EIN (`FEIN`)**: Must be non-zero (required field, per `JB06`).
     - **Duplicate Order Match Type (`DUPC`)**: Must be `A` (match on customer, shipto, product), `B` (match on customer, shipto, product, PO#), or blank.
     - **Product Move (`PRMV`)**: Must be `Y`, `N`, or blank.
   - Sets error indicators and messages if validations fail, preventing progression to the next screen.

6. **Delete Processing (`DELETE` Subroutine)**:
   - Checks if the customer has outstanding invoices by summing aging fields (`AGE` array).
   - If the balance is non-zero, displays an error (`MSG(9)`, `MSG(10)`) and prevents deletion.
   - Marks the customer as inactive (`ARDEL = 'I'`, `CSDEL = 'I'`) in `ARCUST` and `ARCUSP` instead of deleting (per revision note).
   - Updates history files (`ARCUSHS`, `ARCUSPH`) and displays a success message (`MSG(4)`).

7. **Reactivate Processing (`REACTI` Subroutine)**:
   - Changes the delete flag from `I` (inactive) to `A` (active) in `ARCUST` and `ARCUSP`.
   - Updates history files and displays a success message (`MSG(12)`).

8. **Date Validation (`@DTEDT` Subroutine)**:
   - Validates dates (`MMDDYY`) for month, day, and leap year correctness.
   - Handles Y2K century adjustments and checks for valid months (1-12) and days (1-31, or 1-28/29 for February).
   - Sets error indicator `79` if the date is invalid.

9. **History File Updates**:
   - Writes records to `ARCUSHS` (customer history) and `ARCUSPH` (supplemental history) during add (`89` indicator), update (`89N91`), or delete/reactivate (`88`) operations.
   - Includes system date (`SYSCYM`), time (`SYSTIM`), user ID (`USERID`), and workstation ID (`WSID`) in history records.

10. **Program Termination**:
    - Closes all files, sets the Last Record indicator (`LR`), and returns.

---

### **Business Rules**

The program enforces several business rules to ensure data integrity and compliance with customer master maintenance requirements:

1. **Customer Number Validation**:
   - Customer number (`CUSN`) cannot be zero for new entries (`MSG(19)`).
   - Must not already exist in `ARCUST` or `ARCUSP` for new entries (`MSG(7)`, `MSG(26)`).
   - Must exist in both `ARCUST` and `ARCUSP` for updates.

2. **Company Number Validation**:
   - Company number (`CO`) must exist in `ARCONT` (`MSG(1)`).

3. **Field Validations**:
   - Mandatory fields like sort name (`LSTN`), salesman number (`SLS#`), terms code (`TERM`), state (`STAT`), and Federal EIN (`FEIN`) must be valid.
   - Codes like `STMT`, `FINC`, `SFRT`, `FRCD`, `EFT`, `EDI`, `CLPB`, `WIRE`, and `PRMV` must match specific values (`Y`, `N`, or blank).
   - Freight code (`FRCD`) must be `C`, `P`, `A`, or blank, with specific rules for separate freight (`SFRT`) based on freight type.
   - Customer class (`CCLASS`) and other codes must exist in `GSTABL` under specific table types (e.g., `ARCUCL`, `SLSMAN`, `ARTERM`).

4. **Deletion Rules**:
   - Customers with outstanding invoices (non-zero balance in `AGE`) cannot be deleted (`MSG(9)`, `MSG(10)`).
   - Deletion marks records as inactive (`ARDEL = 'I'`, `CSDEL = 'I'`) rather than physically deleting them.

5. **EFT Validation**:
   - If `AREFT = 'Y'`, bank routing (`CSARTE`) and account (`CSABK#`) must be provided.

6. **Date Handling**:
   - Dates (`STDATE`, `FSDATE`, `ICDATE`) are validated for correctness, including leap year checks.
   - Century is added for Y2K compliance if not provided, using `Y2KCEN` (19 or 20) based on the year compared to `Y2KCMP` (80).

7. **Group Account Updates**:
   - If a customer is part of a group (`ARCLGR`), the credit problem flag (`ARCLPB`) is propagated to all grouped accounts.

8. **Freight and Billing**:
   - Supports special freight scenarios:
     - `CNY` (calculate freight for freight collect, `JB09`): For non-Bradford locations (e.g., Anchor).
     - `CYY` (freight collect with service fee, `JB10`): ARG arranges shipping, customer is billed by the carrier, and a $100 service fee is charged.
   - Wire transfer instructions are printed on invoices/statements if `ARWIRE = 'Y'`.

9. **History Tracking**:
   - All add, update, and delete/reactivate operations are logged to `ARCUSHS` and `ARCUSPH` with timestamps and user details.

---

### **Tables (Files) Used**

The RPGLE program uses the following files, as defined in the File Specification (`F`) section:

1. **AR900D** (`WORKSTN`, Update/Input, `CF`): Workstation display file for user interaction, externally described with the `PROFOUNDUI` handler.
2. **ARCUST** (`UF A`, Update/Add, 384 bytes, Key at position 2): Primary customer master file.
3. **ARCUSP** (`UF A`, Update/Add, 1344 bytes, Key at position 2): Customer supplemental file.
4. **GSTABL** (`IF`, Input, 256 bytes, Key at position 2): General system table for validating salesman, terms, customer class, and group codes.
5. **ARCONT** (`IF`, Input, 256 bytes, Key at position 2): Customer contact file for company validation.
6. **BICONT** (`IF`, Input, 256 bytes, Key at position 2): Billing contact file for invoice styling.
7. **ARCLGR** (`IF`, Input, 240 bytes, Key at position 2): Customer group ledger file for grouped account updates.
8. **ARCUST2** (`UF`, Update, 384 bytes, Key at position 2): Secondary reference to `ARCUST` for group updates.
9. **ARCUSHS** (`O A`, Output/Add, 411 bytes, Key at position 2): Customer history file.
10. **ARCUSPH** (`O A`, Output/Add, 1371 bytes, Key at position 2): Customer supplemental history file.

---

### **External Programs Called**

The program calls the following external programs, as indicated in the code (some calls are commented out, reflecting updates in `JB11` and `JB08`):

1. **BI907AC**:
   - Called in the `S3` subroutine to maintain `ARCUPR` (customer pricing) data.
   - Parameters: Company (`@CPCO`), Customer (`@CPCUS`), Shipto (`@CPSHP`, set to `'000'`), Mode (`@CPMI`, set to `'MNT'`), and File Group (`@CPFGR`, set to `' '`).
   - Replaced direct `ARCUPR` processing (per `JB08`).

2. **AR915P** (Commented Out, per `DC01`):
   - Previously called in `S3` for customer form type contacts maintenance (`ARCUFM`).
   - Parameters included company (`A$CO`), customer (`A$CUST`), mode (`A$MODE = 'MNT'`), and file group (`A$FGRP = 'X'`).
   - No longer used due to `JB11` revisions.

3. **GB730P** (Commented Out, per `JK01`):
   - Previously called in `S2` for history inquiries on `ARCUST` and `ARCUSP`.
   - Parameters included file name (`O$FILE`), file group (`O$FGRP`), company (`O$CO`), customer (`O$CUST`), and shipto (`O$SHIP`).
   - No longer used due to `JB11` revisions.

---

### **Summary**

- **Process Steps**: The program initializes variables, loops through screen formats (`FMT01`, `FMT02`, `FMT03`, `FMT05`) to manage customer data, validates inputs, updates/adds records to `ARCUST` and `ARCUSP`, maintains group accounts in `ARCLGR`, logs history to `ARCUSHS` and `ARCUSPH`, and handles deletions/reactivations.
- **Business Rules**: Enforces strict validations on customer numbers, mandatory fields, codes, and dates; prevents deletion of customers with outstanding invoices; supports freight and billing scenarios; and maintains history records.
- **Tables Used**: `AR900D`, `ARCUST`, `ARCUSP`, `GSTABL`, `ARCONT`, `BICONT`, `ARCLGR`, `ARCUST2`, `ARCUSHS`, `ARCUSPH`.
- **External Programs Called**: `BI907AC` (active), `AR915P` and `GB730P` (commented out, no longer used).

This RPGLE program is a critical component of the customer master maintenance process, integrating with the OCL script to provide a robust interface for managing customer data. If you need further clarification on specific subroutines or business rules, let me know!