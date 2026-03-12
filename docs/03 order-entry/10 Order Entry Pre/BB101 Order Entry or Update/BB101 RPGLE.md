The `BB101.rpgle.txt` document is an RPGLE (RPG IV) program used in conjunction with the `BB101.ocl36.txt` OCL program in an IBM System/38 or AS/400 (now IBM i) environment. This program handles the entry and update of open orders in the `BBORDR` file, supporting order entry, validation, and maintenance processes. Below is a detailed explanation of the process steps, business rules, tables (files) used, and external programs called.

---

### Process Steps of the RPGLE Program

The `BB101` RPGLE program manages the interactive entry and updating of order data through multiple screens, performing validations, calculations, and updates to various files. The process steps are organized around screen interactions, data validation, and file updates, as outlined below:

1. **Program Initialization**:
   - The program is initialized with the `H` specification (`fixnbr(*zoned:*inputpacked)`) to handle zoned decimal and packed decimal data formats.
   - It defines the workstation file `bb101d` (previously `SCREEN`) for interactive screen processing, using `infds(statrk)` for status tracking and `infsr(rollky)` for handling roll keys.
   - Comments indicate that the program is a converted version from an earlier system (TARGET/400 conversion on 11/25/19), with added, deleted, and modified source lines.

2. **Screen Processing**:
   The program uses multiple screens for data entry and display, each with specific purposes:
   - **Screen 1 (BB101S1, Output: 21)**: Selection prompt for entering company number, order number, batch number, and other initial data.
   - **Screen 2 (BB101S2, Output: 22)**: Captures header data, including customer, ship-to, order dates, freight codes, and other order-level details.
   - **Screen 3 (BB101S3, Output: 23)**: Handles line item data, including product, quantity, price, and container details. Header fields were added to this screen in revision JB87 to address overlay issues in Profound Genie.
   - **Screen 4 (BB101S4, Output: 24)**: Manages miscellaneous line items (e.g., non-product charges).
   - **Screen 5 (BB101S5, Output: 25)**: Supports description and product search.
   - **Screen 6 (BB101S6, Output: 26)**: Facilitates customer alpha search.
   - **Screen 7 (BB101S7, Output: 27)**: Captures remarks for orders, invoices, and bills of lading (BOL).
   - **Screen 8 (BB101S8, Output: 28)**: Allows overriding tax fields.
   - **Screen 9 (BB101S9, Output: 29)**: Lists orders in a batch.
   - **Screen Z (BB101SZ, Output: 38)**: Displays a remarks review screen.
   - **Screen A (BB101SA, Output: 43)**: Shows credit limit error messages.
   - **Screen B (BB101SB, Output: 94)**: Blank overlay for the line item screen.

3. **Data Validation and Business Logic**:
   - The program performs extensive validations on input fields, using error messages (COM and COM1 comments) to enforce business rules. Examples include:
     - Validating company number, customer number, ship-to number, order dates, sales tax code, salesman number, terms code, and general ledger (G/L) numbers.
     - Checking product codes, quantities, prices, unit of measure, and container codes.
     - Ensuring freight codes are valid (e.g., "C", "P", "A" for collect, prepaid, or PPD&ADD).
     - Validating multi-load orders and railcar restrictions.
   - Specific validations include:
     - **JB01**: Forces a valid carrier ID on Screen 2.
     - **JB06**: Adds 'S' order locking code for orders being shipped in IMS.
     - **JB11**: Forces 'Y' no-charge code for customer-owned product orders and restricts quantity/amount on miscellaneous lines.
     - **JB12**: Validates group-by field as 'S' (size) or 'C' (category), defaulting to 'S'.
     - **JB19**: Restricts EDI orders to specific plants by comparing LDA RACD to order RACD and ensures COON='Y'.
     - **JB31**: Prohibits 'CC' or 'CT' carrier codes.
     - **JB32**: Prevents 'SCO' freight processor for new orders.
     - **JB48**: Validates order process status (blank, 'C', or 'S').
     - **JB52**: Makes bill-to PO# a required field.
     - **JB55**: Makes responsible area/major location a required field.
     - **JB93**: Bypasses credit checking for customer-owned product orders (COON='Y').
   - The program uses command keys (e.g., KA for bypass, KG to end job, F1 to F12 for specific functions like lookups and overrides).

4. **File Operations**:
   - **Add/Update Records**:
     - Adds or updates header records (`OUTHDR`, indicators 80/N80).
     - Adds or updates transaction line items (`OUTDTL`, indicators 81/N81).
     - Adds or updates miscellaneous transaction lines (`OUTMSC`, indicators 82/N82).
     - Updates freight processor (`upfpcd`), calculated quantities/weights (`updd@`, `updd@s`), and tax fields (`addtax`, `updtax`, `addmtx`, `deldtx`).
   - **Delete Records**:
     - Flags records for deletion rather than physically deleting them (`deldtx`, revision DC16).
   - **Release Records**:
     - Releases records in files like `BBORTR`, `BBOTHS1`, and `BBOTDS1` (`nulbbortr`, `nulbboths1`, `nulbbotds1`).
   - **File Updates**:
     - Updates files like `BICONT` (indicator 87) and `SHIPTO` (indicator 79).
     - Writes specific fields to `BBORHS1` and `BBOTHS1` individually instead of as 256-byte fields (revision JB85).

5. **Freight and Quantity Calculations**:
   - Calculates quantities and weights (e.g., gross gallons, net gallons, shipping weight) using subroutine `getslb` when KA is pressed (revision JB92).
   - Performs freight calculations by calling `BB101F` (JB30) and `BB106` (JB64), with logic to auto-populate carrier ID from top 5 preferences if blank (JB71).
   - Displays estimated carrier freight for product moves (JB82).

6. **End-of-Order Processing**:
   - Moved to before the remarks review screen (JB63), including:
     - Quantity calculations for all order details.
     - Freight calculations.
     - Order total calculations.
     - Credit limit checks (bypassed for customer-owned products, JB93).
   - Displays top 5 logistic carrier ID preferences and allows carrier ID updates on Screen 7 and Z.

7. **Error Handling and Messages**:
   - Uses a comprehensive set of error messages (COM and COM1) to notify users of invalid inputs or conditions (e.g., invalid company, customer, dates, or freight codes).
   - Issues warnings for specific conditions, such as missing container weight records (JB18, upgraded to error in JB21) or holiday pickup/delivery dates (MG, 02/01/16).
   - Displays credit hold messages (COM1) for accounts exceeding limits or with overdue invoices.

8. **Screen Navigation and Command Keys**:
   - Supports roll keys for navigating detail and miscellaneous screens (indicators 75, 76, 83, 84, 85, 86).
   - Uses function keys for specific actions:
     - **F1**: Jumps to header screen or cancels new orders not yet in the transaction file (JB60).
     - **F4**: Container code lookup (JK05).
     - **F5**: Product lookup (JK05).
     - **F6**: Overrides product restrictions (JB61).
     - **F7/F9**: Inventory inquiry via `IN805` (DC24, JB58).
     - **F8**: Pricing lookup via `AR822R` (DC19).
     - **F10**: Product description lookup (JK05) or allows holiday dates (MG, 02/01/16).
     - **F11**: Accepts inactive rack price warnings (JB45).
     - **F12**: Overrides specific errors (e.g., freight code mismatches).

9. **Special Processing**:
   - Handles customer-owned product orders by bypassing certain validations (e.g., `INTANK` existence, JB15).
   - Supports EDI orders with specific restrictions (JB19).
   - Validates in-transit and destination locations/tanks (JB72).
   - Checks for duplicate hand tickets (MG09, JB88) and ensures hand ticket format (HT# prefix, JK07).
   - Bypasses product restriction tests for carrier codes 'PT' or 'XS' (MG11).
   - Ensures delivery and pickup dates are not more than 30 days past (MG12).

---

### Business Rules

The program enforces numerous business rules, primarily through validations and error messages. Key rules include:

1. **Order Validation**:
   - Company, customer, ship-to, and order numbers must be valid (errors 01, 03, 04, 46).
   - Order types must be valid (blank, 'M', or specific invoice types like 'R', 'P', 'J', 'T') (error 02).
   - Dates (order, P/O, pickup, delivery) must be valid and not on holidays unless overridden (errors 05, 06, 07, 68, 92, 93, 107, 108).
   - Sales tax codes, salesman numbers, terms codes, and G/L numbers must exist in respective tables (errors 08, 09, 10, 11, 65).
   - Bill-to PO# is required (JB52, error 89).
   - Responsible area/major location is required (JB55, error 74).

2. **Product and Inventory**:
   - Products must have valid codes, containers, and units of measure (errors 12, 15, 22, 29, 30, 52).
   - Inactive or deleted products/tanks trigger errors or warnings (JB45, JB65, JB68, errors 87, 88).
   - Customer-owned products bypass certain validations (e.g., `INTANK`, credit checks) (JB11, JB15, JB93).
   - Product moves require specific validations (errors 96, 97, 99, 100).

3. **Freight Rules**:
   - Freight codes must be 'C' (collect), 'P' (prepaid), or 'A' (PPD&ADD) (error 26).
   - Railcar orders restrict freight codes to 'C' or 'L' and prohibit multi-loads (JB43, error 27).
   - Freight collect requires blank delivery and no separate freight (errors 57, 61, 65).
   - Prepaid and PPD&ADD require delivery='Y' and specific separate freight settings (errors 58, 59, 62, 63, 66, 67).
   - Carrier ID must be valid and not inactive (JB50, error 72).
   - 'SCO' freight processor is not allowed for new orders (JB32, error 84).
   - Freight calculations must be enabled ('Y') or disabled ('N') appropriately (errors 64, 94, 98).
   - Default carrier ID to 'BPRR' for railcar orders (JB81).

4. **Quantity and Pricing**:
   - Quantities must be valid and non-zero unless multi-load settings allow it (errors 13, 42, 43, 44).
   - Prices must be valid, with special handling for rack prices and sales agreements (JB45, JB80).
   - Customer-owned product orders force no-charge codes and restrict miscellaneous line quantities/amounts (JB11, error 75).
   - Calculated quantities (e.g., gross/net gallons, weights) are updated at key points (JB59, JB92).

5. **Credit and Duplicates**:
   - Credit checks are bypassed for customer-owned product orders (JB93).
   - Duplicate order checking is performed via `BB115` (DC18, error 85).
   - Hand ticket numbers must be unique per customer and start with "HT# " (MG09, JB88, errors 102, 103, 104, 105).

6. **EDI and Special Orders**:
   - EDI orders require plant-specific access (JB19).
   - Multi-load orders have specific quantity and freight code restrictions (errors 40, 41, 42, 43, 44, 45, 51).
   - Railcar orders prohibit multi-loads (JB43, error 86).

7. **Screen and Data Handling**:
   - Protects certain fields for EDI orders (JB14).
   - Ensures proper cursor positioning after lookups (JB47).
   - Handles one-time ship-to deletions correctly (JB67).
   - Displays top 5 carrier preferences and allows updates on specific screens (JB63).
   - Bypasses certain validations temporarily (e.g., responsible area/major location, JB86).

---

### Tables (Files) Used

The program interacts with numerous files for data storage, retrieval, and validation. Based on the RPGLE source and cross-referencing with the OCL program, the following files are used:

1. **BB101D**: Workstation display file for interactive screens (replaces `SCREEN`).
2. **BBORTR**: Order transaction file (update, add, release records).
3. **BBOTHS1**: Order transaction history supplemental file.
4. **BBOTDS1**: Order detail supplemental file.
5. **BBTRTX**: Tax transaction file.
6. **BBORDR**: Open order detail file (primary file for order entry/update).
7. **BICONT**: Customer contract file.
8. **SHIPTO**: Ship-to address file.
9. **GSPREX**: Product exception file (validation removed in JK08, moved to `BB116`).
10. **INLOC**: Inventory location file.
11. **INTANK**: Inventory tank file.
12. **GSCTWT**: Contract weight table for non-fluid validation.
13. **BBORDD**: Order detail file.
14. **GSTABL**: General system table (replaced for `PRODCD` and `BBCAID` in JK09, JK10).
15. **GSPROD**: Product file (replaces `GSTABL` for `PRODCD`, JK09).
16. **BBCAID**: Carrier ID file (replaces `GSTABL` for `BBCAID`, JK10).
17. **GSMLCD**: System calendar file (length changed in JB53).
18. **GSCTUM**: Customer master file.
19. **GSUMCV**: Customer summary file.
20. **ARCUST**: Customer master file.
21. **ARCUPR**: Customer pricing file (key includes container type in JB62).
22. **BBSHSA**: Shipment file (used for accessorials/marks in JB13).
23. **BBPRXR**: Product cross-reference file.
24. **GLMAST**: General ledger master file.
25. **SHPADR**: Shipping address file.

---

### External Programs Called

The program calls several external programs for specific functions, as noted in the revision history and comments:

1. **BB1011**: Retrieves product, pricing, and customer data (common data structure, JB74, JK04).
2. **BB1014**: Retrieves inventory location (`INLOC`) information (used by BB495, BB500, and BB101).
3. **BB1015**: Retrieves accessorials/marks from `BBSHSA` (JB13).
4. **BB101F**: Performs freight calculations (JB30).
5. **BB106**: Performs advanced freight calculations (JB64).
6. **BB115**: Checks for duplicate customer orders (DC18).
7. **BB811**: Customer transaction batch order inquiry (DC21).
8. **BB104A**: Handles open order cancellation or reactivation (DC23).
9. **BB1033**: Validates responsible area/major location (JK07, temporarily bypassed in JB86).
10. **BB1018**: Handles accessorials/marks with data structure parameters (JB83).
11. **AR822R**: Pricing lookup for CSR (called via F8, DC19).
12. **IN805**: Inventory inquiry (called via F9, DC24, JB58, JK02, JK03, JB78).
13. **LCSTSHP**: Customer ship-to lookup (replaces internal search, DC14, DC17).
14. **MSHIPTO**: Customer ship-to lookup (replaces `SHIPS1R`, DC11).
15. **MGSTABL**: Country code lookup (DC10).
16. **MBBQTY**: Accumulates total shipping weight for display on Screen 7 (DC20).

---

### Summary

The `BB101` RPGLE program, called from the `BB101.ocl36.txt` OCL program, manages the entry and update of open orders in the `BBORDR` file. It uses multiple screens for order header, line items, miscellaneous items, remarks, and tax overrides, with extensive validations to enforce business rules. Key processes include order entry, product and freight validation, quantity and weight calculations, credit checks, and file updates. The program interacts with numerous files for order, customer, inventory, and pricing data and calls external programs for specialized functions like freight calculations, inventory inquiries, and duplicate order checks. Business rules ensure data integrity, proper freight handling, and compliance with customer and system requirements, with specific accommodations for EDI, railcar, and customer-owned product orders.