The RPGLE program `BB101.rpgle.txt` is a complex order entry and management program for an IBM AS/400 (iSeries) system, called from the `BB101.ocl36.txt` OCL program. It handles the entry, validation, and updating of orders, including header, detail, and miscellaneous line items, with extensive validation and integration with various files and external programs. Below, I will explain the process steps, business rules, tables (files) used, and external programs called, based on the provided RPGLE source and its revision history.

### Process Steps of the RPGLE Program

The program is structured around multiple input/output screens and subroutines to manage order entry, validation, and updates. The process steps are organized by the screens and their associated logic:

1. **Initialization and Setup**:
   - The program initializes by setting up the workstation file (`BB101D`) and other necessary files for order processing.
   - It uses the `H` specification (`fixnbr(*zoned:*inputpacked)`) to handle numeric data formats, ensuring compatibility with zoned and packed decimal fields.
   - The program sets up data structures and parameters for interacting with external programs and files.

2. **Screen 1: Selection Prompt (BB101S1)**:
   - Displays a selection screen for entering company number, order number, and batch number.
   - Validates the company number and order type (e.g., blank, 'M', 'R', 'P', 'J', 'T' for invoice types).
   - Outputs fields like `co` (company), `ordr` (order number), `batch#`, and `msg` (message) to guide the user.

3. **Screen 2: Header Data Entry (BB101S2)**:
   - Collects header information such as customer number, ship-to number, order date, purchase order date, requested delivery date, freight codes, and Incoterms.
   - Validates fields like customer number, ship-to number, order date, sales tax code, salesman, terms code, and carrier ID.
   - Performs checks for credit limits, duplicate orders, and valid Incoterms for export orders.
   - Outputs header fields like `head1`, `head2`, `head3`, `head4`, `bcus` (bill-to customer), `fpcd` (freight processor code), and `inct` (Incoterms).

4. **Screen 3: Line Item Data Entry (BB101S3)**:
   - Allows entry of order detail lines, including product code, quantity, price, container type, and unit of measure.
   - Validates product codes, quantities, prices, and container types against files like `GSPROD`, `BBCAID`, and `GSCTWT`.
   - Ensures consistency in freight codes across detail lines and checks for inactive or deleted records.
   - Calculates gross gallons (`d@ggal`), net gallons (`d@ngal`), shipping weight (`d@gwt`), quantity in unit of measure (`d@qtum`), and product weight (`d@prwt`).
   - Outputs detail fields like `detx`, `det`, `msg`, and header fields (added in JB87 to address overlay issues).

5. **Screen 4: Miscellaneous Line Item Data (BB101S4)**:
   - Handles miscellaneous line items (e.g., freight surcharges, service fees).
   - Validates miscellaneous G/L numbers and ensures quantities and amounts are zero for specific cases (e.g., customer-owned product).
   - Suppresses display of certain sequence numbers (e.g., ≥940 for freight service charges, per JB90).
   - Outputs fields like `mscx`, `msc`, and `msg`.

6. **Screen 5: Description and Product Search (BB101S5)**:
   - Facilitates product and description lookups using function keys (F5 for product, F10 for description, F4 for container code per JK05).
   - Validates product codes and cross-references using `BBPRXR`.
   - Outputs fields like product code and description.

7. **Screen 6: Customer Alpha Search (BB101S6)**:
   - Allows searching for customers by name or other criteria.
   - Calls external program `LCSTSHP` for customer/ship-to lookup.
   - Outputs search results for customer and ship-to data.

8. **Screen 7: Remarks Entry (BB101S7)**:
   - Captures order, invoice, and bill of lading remarks, as well as freight-related fields.
   - Displays carrier preferences and freight totals (per JB63, JB64).
   - Validates freight processor codes and carrier IDs.
   - Outputs fields like `omk1`, `omk2` (order remarks), `imk1`, `imk2` (invoice remarks), `bmk1`, `bmk2` (BOL remarks), and `frcd` (freight code).

9. **Screen 8: Override Taxes (BB101S8)**:
   - Allows overriding tax codes, exemptions, and amounts for up to 10 tax fields.
   - Validates tax codes and amounts against predefined rules.
   - Outputs tax-related fields like `txc`, `txe`, `txa`, `ftc`, `ofe`, and `ofa`.

10. **Screen 9: List Orders in Batch (BB101S9)**:
    - Displays a list of orders within a batch for review.
    - Outputs fields like `co`, `s9o`, `s9n`, `s9s`, and `s9r`.

11. **Screen SA: Credit Limit Error (BB101SA)**:
    - Displays credit limit error messages when an account is on hold due to overdue invoices, exceeding credit limits, or zero credit limit.
    - Outputs messages `msga` and `msgb`.

12. **End of Order Processing**:
    - Performs final calculations for quantities, freight, and order totals (moved to before marks review screen per JB63).
    - Calls `BB106` for freight calculations and `BB115` for duplicate order checks.
    - Conducts credit limit checks (bypassed for customer-owned product per JB93).
    - Updates files like `BBORTR`, `BBOTHS1`, `BBOTDS1`, and `BBTRTX` with calculated values and tax information.
    - Releases records and clears locks.

13. **Error Handling and Validation**:
    - Uses a comprehensive set of error messages (COM and COM1 comments) to handle invalid inputs, such as company number, customer number, dates, tax codes, and freight codes.
    - Implements command key functions (e.g., KA to bypass, KG to end job) and roll keys for navigation (e.g., indicators 75, 76 for roll forward/backward).
    - Handles special cases like inactive records, holiday date warnings, and duplicate hand ticket checks.

### Business Rules

The program enforces numerous business rules, as detailed in the revision history and error messages. Key rules include:

- **Validation Rules**:
  - Company number, customer number, ship-to number, order date, purchase order date, and delivery date must be valid (COM 01–07).
  - Freight codes must be 'C' (collect), 'P' (prepaid), or 'A' (PPD&ADD), with specific constraints (e.g., railcar orders require 'C' or 'L') (COM 26–27, 57–67).
  - Product codes, container types, and units of measure must exist in `GSPROD`, `BBCAID`, or `GSCTWT` and not be inactive or deleted (COM 12, 15, 29, 30, 52).
  - Bill-to PO number is mandatory (JB52, COM 89).
  - Group by must be 'S' (size) or 'C' (category), defaulting to 'S' (JB12, COM 76).
  - Incoterms are required for export orders and validated against `GSTABL` (DC02, COM 78–80).
  - Delivery and pickup dates must not be holidays (MG 020116, COM 92–93) or more than 30 days past (MG12, COM 107–108).
  - Inactive records (e.g., `GLMAST`, `GSCTUM`, `GSCTWT`, `GSUMCV`, `SHIPTO`) are treated as deleted (JB65, JB68, JB69).
  - Carrier ID must be valid and not inactive (JB50, COM 25, 72).

- **Freight and Pricing Rules**:
  - Freight processor code ('SCO' not allowed for new orders, JB32) and freight arranged by must be valid (COM 73).
  - Freight collect requires blank delivery, while prepaid and PPD&ADD require delivery 'Y' (COM 57–59).
  - Separate freight and calculate freight must align with freight code (COM 60–67).
  - Customer price fields reflect freight collect (1.00), prepaid with rack price (1.00), prepaid with sales agreement (0.75), or PPD&ADD (1.00 with 0.25 freight) (JB80).
  - Freight service fee for 'CYY' freight collect orders (JB90).
  - Warns if freight calculation is requested but not performed (JB70, COM 94, 98).

- **Order Processing Rules**:
  - Multi-load orders are not allowed for railcars (JB43, COM 86).
  - Customer-owned product orders (COON='Y') bypass credit checks (JB93), force no-charge on detail lines, and prohibit quantity/amount on miscellaneous lines (JB11).
  - Duplicate order checks are performed via `BB115` (DC18, COM 85).
  - Hand ticket numbers must start with 'HT# ' and be unique per customer (MG09, COM 102–105).
  - Order numbers reset to 100001 when reaching 899999 to maintain 6-digit length (JB49).
  - EDI orders with errors have 'E' in header delete code, which is cleared on the marks screen (JB19).

- **Screen and Navigation Rules**:
  - Cursor positioning is adjusted for lookups (DC04, JB47) and uses `RTNCSRLOC` for field prompting (DC07).
  - Header fields are displayed on the detail screen to resolve overlay issues in Profound Genie (JB87).
  - Function keys (e.g., F1 to cancel new orders, F4 for container lookup, F5 for product, F9 for inventory inquiry, F10 for description) enhance user interaction (JB60, JK05, JB58).
  - Marks review screen displays customer, ship-to, and freight arranged by details (JB17).

- **Data Handling Rules**:
  - Descriptions are aligned correctly (27 bytes on screen, 30 bytes in file, adjusted by JB02).
  - Inactive rack prices trigger errors or warnings based on status (JB45, COM 87–88).
  - Sales agreement pricing uses billed PO number and allows zero end date (JB44).
  - Tax fields support up to 10 entries (LT38) and include percentage fields (LT39).
  - Ship-to addresses are reformatted to include city, state, zip, and country (DC06, DC08).
  - Calculated quantities (gross/net gallons, weights) are updated before marks review (JB63, JB92).

### Tables (Files) Used

The program uses the following files, as referenced in the RPGLE source and aligned with the OCL program:

1. `BB101D` (workstation file for display screens)
2. `BBORTR` (order transaction file)
3. `BBOTHS1` (order header supplemental)
4. `BBOTDS1` (order detail supplemental)
5. `BBTRTX` (tax transaction file)
6. `BBORDR` (open order file)
7. `GSPROD` (product master, replaces `GSTABL` type `PRODCD`, JK09)
8. `BBCAID` (carrier ID, replaces `GSTABL` type `BBCAID`, JK10)
9. `BBPRXR` (product cross-reference)
10. `ARCUST` (customer master)
11. `SHIPTO` (ship-to master)
12. `GSTABL` (general table, used for various lookups like Incoterms, order process status)
13. `GSCTWT` (container weight table)
14. `GSCTUM` (unit of measure table)
15. `GSUMCV` (unit conversion table)
16. `GLMAST` (general ledger master)
17. `BICONT` (billing control file)
18. `BBSHSA` (shipping instructions/accessorials)
19. `ARCUPR` (customer pricing)
20. `GSCNTR1` (container master, replaces `GSTABL` type `CNTRCD`, 04/13/16)
21. `GSMLCD` (miscellaneous code table)
22. `BICUAX` (sales agreement file)

### External Programs Called

The program calls the following external programs for specific functions:

1. `BB1011`: Retrieves common data (e.g., fluid code, IMS unit of measure, customer-owned product fields).
2. `BB1014`: Retrieves `INLOC` information (used by BB495, BB500, and BB101).
3. `BB1015`: Retrieves accessorials/marks from `BBSHSA`.
4. `BB1018`: Validates fields via data structure (JB83).
5. `BB1033`: Validates responsible area/major location (JK07, bypassed temporarily per JB86).
6. `BB104A`: Handles open order cancellation or reactivation (DC23).
7. `BB106`: Performs freight calculations (JB64).
8. `BB115`: Checks for duplicate orders (DC18).
9. `BB811`: Customer transaction batch order inquiry (DC21).
10. `AR822R`: Pricing lookup for CSR via F8 (DC19).
11. `LCSTSHP`: Customer/ship-to lookup (DC14, DC17).
12. `MBBQTY`: Accumulates total shipping weight for display on screen 7 (DC20).
13. `MGSTABL`: Looks up country codes (DC10).
14. `MSHIPTO`: Customer ship-to lookup (DC11).
15. `IN805`: Inventory inquiry, called via F9 (DC24, JB58), with modes for display, auto-check, and messaging (JB78).

### Summary

The `BB101.rpgle.txt` program is a comprehensive order entry system that manages order headers, detail lines, miscellaneous lines, taxes, and remarks through multiple interactive screens. It enforces strict validation rules for data integrity, handles freight and pricing logic, and integrates with numerous files and external programs to support business processes like order validation, credit checks, and inventory inquiries. The extensive revision history reflects ongoing enhancements to support new requirements, such as freight calculations, customer-owned product handling, and modernized user interfaces. The program’s complexity and integration with external systems make it a critical component of the order management process, likely in a manufacturing or distribution environment.