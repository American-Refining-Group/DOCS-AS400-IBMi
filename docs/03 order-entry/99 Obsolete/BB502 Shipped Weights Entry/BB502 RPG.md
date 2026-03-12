The `BB502.rpg36.txt` is an RPG (Report Program Generator) program for the IBM System/36 or AS/400, called from the `BB502.ocl36.txt` OCL program. It handles the **Shipment Weights Entry/Update and Edit** process, focusing on managing shipment data for orders, including header and detail records, customer lookups, and validations. Below, I’ll explain the process steps, business rules, tables (files) used, and external programs called, based on the provided RPG source code.

---

### Process Steps of the RPG Program

The `BB502` RPG program is structured to manage shipment weights entry, update, and validation through a series of screens and subroutines. It processes order headers, line items, and shipment details, ensuring data integrity through validations and interactions with various files. The process steps are as follows:

1. **Program Initialization**:
   - The program defines the workstation file (`SCREEN`) for interactive screen processing and declares multiple disk files (e.g., `BICONT`, `BBORDR`, `BBSHPH`) for data access.
   - It sets up arrays (`CSAR`, `KNM`, `CXNM`) for customer lookup and error message tables (`TABLC`, `COM`) for lockout codes and error messages.
   - The program uses indicators (e.g., `11`, `12`, `80`, `81`) to control logic flow and error handling.

2. **Screen Processing**:
   - The program uses three main screens:
     - **Screen 1 (BB502S1)**: Order selection screen, capturing company number (`CO`), order number (`ORDNO`), shipping reference number (`SRN#`), and batch number (`BATCH#`).
     - **Screen 2 (BB502S2)**: Order header entry screen, displaying fields like customer number (`CUST`), ship-to (`SHIP`), order date (`RQDT`), carrier code (`CACD`), and freight details.
     - **Screen 3 (BB502S3)**: Line item entry screen, handling details like location (`LOC`), product (`PROD`), container (`CNTR`), quantities (`CTQK`, `SHPQ`), weights (`TARE`, `GVWT`), and other shipment-specific data.
   - Input fields are defined in the `I` (Input) specifications, mapping screen fields to program variables (e.g., `CO`, `ORDNO`, `CUST`, `PROD`, `CTQK`).

3. **Order Header Processing (`OOHMOV` Subroutine)**:
   - The `OOHMOV` subroutine moves data from the `BBORDR` (order master) file to screen fields for a new or existing order.
   - It transfers header fields like company (`BOCO`), order number (`BORDNO`), customer (`BOCUST`), ship-to (`BOSHIP`), and freight-related fields (`BOFRCD`, `BORTG1`) to screen variables (`CO`, `ORDNO`, `CUST`, `FRCD`, `RTG1`).
   - It initializes fields like ship date (`SHDT`) to zero and sets default values (e.g., `TEMP` to 60 if `BDTEMP` is zero).

4. **Order Detail Processing (`OODMOV` Subroutine)**:
   - The `OODMOV` subroutine moves detail data from `BBORDD` (order detail) to screen fields for line items.
   - It populates fields like item number (`BDITEM` to `LOC`, `PROD`, `TANK`), quantity (`BDQTY` to `CTQK`, `SHPQ`), price (`BDOPRC` to `OPRC`), and weights (`BDTARE`, `BDGVWT` to `TARE`, `GVWT`).
   - It validates the container code (`CNTR`) against `GSCNTR1` and determines the selling tank (`TANK`) based on carrier code (`CACD`) and other conditions.

5. **Shipment Header Processing (`SPHMOV` Subroutine)**:
   - The `SPHMOV` subroutine moves data from the `BBSHPH` (shipment header) file to screen fields, specifically updating the carrier ID (`SHCAID` to `ACTC`).
   - It ensures shipment header data is correctly displayed and updated.

6. **Shipment Detail Processing (`SPDMOV` Subroutine)**:
   - The `SPDMOV` subroutine moves data from the `BBSHPD` (shipment detail) file to screen fields, updating actual weight (`SDACTW` to `ACTW`) and shipped quantity (`SDCTQT` to `SHPQ`).

7. **Customer Lookup (`CSSRCH` and `NRLFWD` Subroutines)**:
   - The `CSSRCH` subroutine handles customer name searches, using the `KNM` array to store search keys and comparing them against `ARCUSTX` (customer extension file).
   - The `NRLFWD` subroutine reads `ARCUSTX` to populate the `CSAR` array with customer data for display, ensuring valid customer records are retrieved.

8. **Order Locking/Unlocking (`UNLORD` Subroutine)**:
   - The `UNLORD` subroutine manages order locking by updating the `BBORDRH` file, setting the lock code (`BOLOCK`) to `'S'` and storing the batch number (`BATCH#`).
   - It also handles unlocking by clearing the lock code.

9. **Carrier and Routing Validation (`RLOOK` Subroutine)**:
   - The `RLOOK` subroutine validates carrier codes by calling the external program `VSCDS1R`, which retrieves carrier details (`@RTCOD`, `@RTDSC`) and maps them to screen fields (`VSCODE`, `VSDESC`, `VSNAME`).
   - It updates the shipping address (`SHPADR`) if the order is customer-owned (`BOCOON = 'Y'`).

10. **Data Validation and File Updates**:
    - The program validates inputs against files like `GSCNTR1` (container codes), `BBCAID` (carrier IDs), and `GSPROD` (product codes).
    - It updates `BBSHPH` and `BBSHPD` files with header and detail data, respectively, using `EADD` (add) and `E` (update) operations.
    - It updates `BBASND` (advanced shipping notice) with batch numbers and other details.

11. **Build Routine (`BUILD` Subroutine)**:
    - The `BUILD` subroutine reads `BBORDR` to retrieve open orders matching the company and order number (`COORD`).
    - It chains to `BBORDRH` and `BBSHPH` to validate order headers and shipment headers, calling `OOHMOV` and `SPHMOV` to populate screen data.
    - For detail records, it chains to `BBORDD` and `BBSHPD`, calling `OODMOV` and `SPDMOV` to process line items.
    - It loops through records until all relevant orders are processed (`BLDEND` tag).

12. **Output Processing**:
    - The program writes to output files (`BBSHPH`, `BBSHPD`, `BBORDRH`, `SHPADR`, `BBASND`) using `O` (Output) specifications.
    - It generates screen outputs for user interaction, displaying errors or messages from the `COM` table (e.g., "INVALID COMPANY NO.", "INVALID ORDER #").

---

### Business Rules

The RPG program enforces several business rules, as indicated by the error messages and logic in the code and change history:

1. **Order and Shipment Validation**:
   - Orders must have a valid company number (`CO`), order number (`ORDNO`), customer (`CUST`), and ship-to (`SHIP`) (Error codes 01, 03, 04, 20).
   - Order types must be valid (e.g., not obsolete types 'M' or 'C'; new 'M' for product move, 'B' for rebill) (Error 02, JB17, JK02).
   - Shipping reference numbers (`SRN#`) must exist in `BBSRNH` (Error 79, JB09).
   - Freight codes must be 'C' (collect), 'P' (prepaid), or 'A' (PPD&ADD), with specific rules for delivery and separate freight (Errors 39, 56–67, JB10).
   - Railcar orders require specific carrier codes ('RC') and valid railcar numbers (Errors 40, 53–55).
   - Quantities, tare, and gross weights must follow rules (e.g., both must be zero or non-zero together; cannot all be zero) (Errors 24, 45–47).

2. **Freight Processing**:
   - For non-packaging plant orders, only orders with a valid freight processor in `BBFRPR` are allowed (JB10).
   - Internal freight processors do not use the processor’s address as the freight bill address, while external processors do (JB10).
   - Freight collect orders must have blank delivery, while prepaid or PPD&ADD require delivery set to 'Y' (Errors 57–59).
   - Calculate freight (`CAFR`) must be 'Y', 'N', or blank, with specific rules for collect vs. prepaid (Errors 64–67, JB19).

3. **Container and Product Validation**:
   - Container codes (`CNTR`) must exist in `GSCNTR1` (Error 43, JK01).
   - Product codes (`PROD`) must exist in `GSPROD` (Error 29, JK04).
   - Unit of measure (`IUM`) must be valid, and container codes may need to be blank for certain products (Errors 15, 44, 77, VV04).
   - Container weight entries are required in the container weight file for certain conditions (Error 78, VV04).

4. **Packaging Plant vs. Viscosity Shipments**:
   - If called from the PPMENU (packaging plant menu), the program restricts orders to packaging plant orders; otherwise, it processes viscosity shipments (JB08, MG01).
   - For viscosity shipments, it skips the `BB5009` quantity conversion and manually copies `CTQK` to `CTQT` (MG01).

5. **Locking and Authorization**:
   - Orders are locked during processing (`BOLOCK = 'S'`) and unlocked when complete (JB05, `UNLORD` subroutine).
   - Users must be authorized to process specific orders (Error 80).
   - Lockout codes (e.g., "ORDER IS BEING INVOICED") prevent concurrent modifications (TABLC/TABD).

6. **Tax and Pricing**:
   - Tax fields are expanded to 1–10 (LT11), and invalid tax codes or amounts are rejected (Errors 33–35).
   - For order type 'J', prices must be greater than zero (Errors 31–32).
   - Freight GL numbers require a miscellaneous type 'F' (Error 68).

7. **EDI and Special Cases**:
   - EDI fields must be 'Y' or 'N', and non-EDI customers require 'N' (Errors 73–74).
   - Special freight calculations for freight collect from non-Bradford locations (e.g., Anchor) are handled (JB19).

8. **Error Handling**:
   - The program uses a comprehensive error message table (`COM`) with 83 messages to guide users on invalid inputs or conditions (e.g., "INVALID ITEM NUMBER", "FREIGHT CODE MUST BE C, P, OR A").
   - Over- or under-shipment requires user acceptance via F6 (Errors 81–82).

---

### Tables (Files) Used

The RPG program interacts with the following files, as defined in the `F` (File) specifications:

1. **SCREEN**: Workstation file for interactive screen processing (1112 bytes, keyed by `WSID`).
2. **BICONT**: Container master file (256 bytes, indexed, read-only).
3. **BBORDR**: Order master file (512 bytes, indexed, read-only).
4. **BBORDD**: Order detail file (512 bytes, indexed, read-only).
5. **BBORDRH**: Order header file (512 bytes, indexed, update-capable).
6. **BBSHPH**: Shipment header file (128 bytes, indexed, update-capable).
7. **BBSHPD**: Shipment detail file (256 bytes, indexed, update-capable).
8. **SHIPTO**: Ship-to address file (2048 bytes, indexed, read-only).
9. **VSCARR**: Carrier file (92 bytes, indexed, read-only).
10. **SHPADR**: Shipping address file (448 bytes, indexed, update-capable).
11. **BBASND**: Advanced shipping notice file (512 bytes, indexed, update-capable).
12. **ARCUST**: Customer master file (384 bytes, indexed, read-only).
13. **ARCUSTX**: Customer extension file (384 bytes, indexed, read-only).
14. **ARCUSP**: Customer profile file (1344 bytes, indexed, read-only).
15. **GLMAST**: General ledger master file (256 bytes, indexed, read-only).
16. **GSTABL**: General system table (256 bytes, indexed, read-only).
17. **BBFRPR**: Freight processor file (256 bytes, indexed, read-only).
18. **BBORH1**: Order header extension file (512 bytes, indexed, read-only, JB09).
19. **BBORD1**: Order detail extension file (512 bytes, indexed, read-only, JB06).
20. **BBSRNH**: Shipping reference number file (128 bytes, indexed, read-only, JB09).
21. **GSCNTR1**: Counter/container file (512 bytes, indexed, read-only, JK01).
22. **BBCAID**: Carrier ID file (200 bytes, indexed, read-only, JK03).
23. **GSPROD**: Product master file (512 bytes, indexed, read-only, JK04).

---

### External Programs Called

The RPG program calls the following external programs:

1. **BB5003**: Used to determine the selling tank (`SELLTK`) based on container and carrier data (called in `OODMOV`).
2. **VSCDS1R**: Retrieves carrier routing details (e.g., `@RTCOD`, `@RTDSC`) for validation and display (called in `RLOOK`).
3. **MBBQTY**: Handles quantity conversion, replacing `BB5009` for certain orders (JB12, called implicitly based on change history).

---

### Summary

The `BB502` RPG program, called from the `BB502` OCL program, manages shipment weights entry and updates through interactive screens, processing order headers, line items, and shipment details. It enforces strict business rules for order types, freight processing, container/product validation, and locking mechanisms. The program interacts with 23 files for data retrieval and updates and calls three external programs (`BB5003`, `VSCDS1R`, `MBBQTY`) for specific tasks. Error handling is robust, with 83 error messages ensuring data integrity and user guidance.

If you need further details on specific subroutines, business rules, or related content (e.g., analysis of X posts or web searches), let me know!