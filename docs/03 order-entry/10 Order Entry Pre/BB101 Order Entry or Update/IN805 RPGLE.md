The `IN805.rpgle.txt` document is an RPGLE (RPG IV) program called from the `BB101.ocl36.txt` OCL program in an IBM System/36 or AS/400 (now IBM i) environment. It is part of the Customer Orders and Inventory system, designed to perform inventory balances inquiry, displaying available inventory for products and containers and checking sufficiency for orders. Written by Dave Capo on 08/25/2014, it includes revisions for tank reading date validation (jk01), screen expansion (jk02), allocated totals calculation (jk04), and container weight file integration (jb05, jk05). Below is a detailed explanation of the process steps, business rules, tables (files) used, and external programs called.

---

### Process Steps of the RPGLE Program

The `IN805` program queries inventory balances for products and containers, displays results in subfiles on the `in805d` display file, and checks if sufficient inventory exists for an order based on the input mode (`p$mode`). The steps are as follows:

1. **Program Initialization**:
   - **File Definitions**:
     - `in805d`: Work station file (display file) with subfiles `sfl1` and `sfl2`, user-opened.
     - `in805w`: Work file in `qtemp` for temporary data (user-opened).
     - `bbordh`: Input-only file for order headers (keyed, user-opened).
     - `bborpx`: Input-only file for order product cross-reference (keyed, user-opened).
     - `gscntr`: Input-only file for container data (keyed, user-opened).
     - `gsctum`: Input-only file for customer unit of measure data (keyed, user-opened).
     - `gsctwt`: Input-only file for container weight data (keyed, user-opened, added in jb05/jk05).
     - `gsprod`: Input-only file for product data (keyed, user-opened).
     - `incont`: Input-only file for inventory container data (keyed, user-opened).
     - `inloc`: Input-only file for inventory location data (keyed, user-opened).
     - `inmtaj`: Input-only file for inventory major adjustments (keyed, user-opened).
     - `inmtrdy`: Input-only file for inventory meter readings (keyed, user-opened).
     - `intank`: Input-only file for tank inventory data (keyed, user-opened).
     - `intkrv`: Input-only file for tank receiving data (keyed, user-opened).
     - `sa5movd3`: Input-only file for movement details (keyed, user-opened).
     - `sa5shz`: Input-only file for shipment history (keyed, user-opened).
     - `gsumcv`: Input-only file for unit measure conversion (keyed, user-opened).
     - `bicont`: Input-only file for bill of lading containers (keyed, user-opened).
     - `arcust`: Input-only file for customer data (keyed, user-opened).
     - `shipto`: Input-only file for ship-to data (keyed, user-opened).
     - `bborcl`: Input-only file for order close data (keyed, user-opened).
   - **Data Structures and Fields**:
     - `m@80`: Array for message data (80 elements, 1 character each).
     - `ovr`, `ovg`, `ovz`: Arrays for file overrides for `in805w` (`qtemp`), and other files in production (`g*`) or alternate (`z*`) libraries.
     - `com`: Array for error messages (4 elements: "No Results", "Totl Gal In Trn---->", "Totl Alloc Gal/Ctn->", "Not enough Prod to fill order", COM,01-04).
     - Field prefixes: `f$` (panel fields), `c$` (subfile control), `s1`/`s2` (subfile fields), `k$` (key lists), `p$` (input parameters), `o$` (output parameters), `r$` (reposition subfile), `w$` (work fields).
   - **Key Lists**:
     - `klordh`: For `bbordh` (company `boco`, customer `bocust`, order number `bordno`, jk08).
     - Other key lists (assumed, not shown) for chaining to inventory and order files.
   - **Indicators**:
     - 19: Panel format input change.
     - 21-39, 50-69: Screen errors.
     - 40: Subfile clear.
     - 41: Subfile display control.
     - 42: Subfile display.
     - 43: Subfile end and next change.
     - 49: Message subfile display/control/initialize/end.
     - 70-79: Input field protect.
     - 80: Primary file chain.
     - 81: Read subfile.
     - 90-99: Secondary file chain/read.

2. **Parameter Input**:
   - Receives input parameters via `*ENTRY PLIST` (assumed, not shown) including:
     - `p$mode` (1 character): Mode of operation ("1", "2", "3").
     - `p$co` (company), `p$prod` (product), `p$cntr` (container), `p$ord#` (order number, jk04), `p$qty` (quantity).
   - `p$mode` determines behavior:
     - "1": Always display inquiry (jk06).
     - "2": Display inquiry only if insufficient product exists for the order (jk06).
     - "3": Send message if insufficient product exists for the order (jb07).

3. **Inventory Inquiry Processing**:
   - Opens files (`bbordh`, `bborpx`, `gscntr`, `gsctum`, `gsctwt`, `gsprod`, `incont`, `inloc`, `inmtaj`, `inmtrdy`, `intank`, `intkrv`, `sa5movd3`, `sa5shz`, `gsumcv`, `bicont`, `arcust`, `shipto`, `bborcl`).
   - Chains to `bbordh` using `klordh` to retrieve order details (company, customer, order number, jk08).
   - Queries inventory files (`incont`, `inloc`, `intank`, `intkrv`, `inmtrdy`, `inmtaj`) to calculate available inventory for the product (`p$prod`) and container (`p$cntr`).
   - Calculates in-transit totals from `sa5movd3` (jk01).
   - Calculates allocated totals from `bborpx` and `bbordh`, excluding the current order number (`p$ord#`) to avoid double-counting (jk04).
   - Converts gallons to containers using `gsctwt` (jb05, jk05).
   - Validates tank reading dates against system date (cannot be future-dated, jk01).

4. **Inventory Sufficiency Check**:
   - Compares available inventory (in gallons or containers) to the order quantity (`p$qty`).
   - If insufficient:
     - For `p$mode="2"`, displays inquiry with details (jk06).
     - For `p$mode="3"`, sets output message "Not enough Prod to fill order" (COM,04, jb07).
   - If sufficient or `p$mode="1"`, proceeds to display inquiry regardless (jk06).

5. **Subfile Population and Display**:
   - Clears subfiles (`*in40`) and initializes relative record numbers (`rrn1`, `rrn2`).
   - Populates subfiles:
     - `sfl1`: Displays inventory details (e.g., location, product, container, available quantity, in-transit totals [COM,02], allocated totals [COM,03]).
     - `sfl2`: Displays additional details (e.g., tank readings, adjustments).
   - Writes subfile control formats and displays subfiles (`*in41`, `*in42`) using `exfmt` on the expanded 27x132 screen (jk02).
   - Displays "No Results" (COM,01) if no inventory data is found.

6. **External Program Calls**:
   - Calls `IN110C` and `IN805BC` to retrieve additional inventory data or perform calculations (jk02, jk03).
   - Passes relevant parameters (e.g., company, product, container, order number).

7. **Program Termination**:
   - Sets the last record indicator (`LR`) to exit the program.
   - Returns output parameters (`o$` fields, assumed), including messages for insufficient inventory (`p$mode="3"`).

---

### Business Rules

The program enforces the following business rules:

1. **Inventory Inquiry Modes**:
   - `p$mode="1"`: Always displays inventory inquiry regardless of sufficiency (jk06).
   - `p$mode="2"`: Displays inquiry only if insufficient product exists for the order (jk06).
   - `p$mode="3"`: Sends a message ("Not enough Prod to fill order", COM,04) if insufficient product exists (jb07).

2. **Inventory Calculations**:
   - Calculates available inventory from `incont`, `intank`, `intkrv`, `inmtrdy`, and `inmtaj`.
   - Includes in-transit totals from `sa5movd3` (jk01).
   - Calculates allocated totals from `bborpx` and `bbordh`, excluding the current order (`p$ord#`, jk04).
   - Converts gallons to containers using `gsctwt` (jb05, jk05).

3. **Data Validation**:
   - Validates tank reading dates in `inmtrdy` to ensure they are not future-dated (jk01).
   - Validates company, product, container, customer, and ship-to data using `gsprod`, `gscntr`, `gsctum`, `arcust`, and `shipto`.

4. **File Overrides**:
   - Uses override arrays (`ovr`, `ovg`, `ovz`) to redirect files to `qtemp` (`in805w`) or production (`g*`) or alternate (`z*`) libraries.
   - Ensures correct file access based on the environment.

5. **Error Handling**:
   - Displays "No Results" (COM,01) if no inventory data is found.
   - Displays in-transit and allocated totals (COM,02, COM,03) in the subfile.
   - Sends "Not enough Prod to fill order" (COM,04) for insufficient inventory in `p$mode="3"`.

6. **Inquiry Mode**:
   - Protects input fields (`*in70`) in inquiry mode, allowing only review of data.

---

### Tables (Files) Used

The program interacts with the following files:

1. **in805d**: Work station file (display file) with subfiles `sfl1` and `sfl2`.
   - Fields: `f$` (panel), `c$` (subfile control), `s1`/`s2` (subfile data).
2. **in805w**: Temporary work file in `qtemp`.
   - Used for intermediate data storage.
3. **bbordh**: Input-only order header file (keyed, user-opened).
   - Fields: `boco` (company), `bocust` (customer), `bordno` (order number).
4. **bborpx**: Input-only order product cross-reference file (keyed, user-opened).
   - Used for allocated quantity calculations.
5. **gscntr**: Input-only container file (keyed, user-opened).
   - Contains container data.
6. **gsctum**: Input-only customer unit of measure file (keyed, user-opened).
   - Contains unit of measure data.
7. **gsctwt**: Input-only container weight file (keyed, user-opened, jb05/jk05).
   - Used for gallon-to-container conversion.
8. **gsprod**: Input-only product file (keyed, user-opened).
   - Contains product data.
9. **incont**: Input-only inventory container file (keyed, user-opened).
   - Contains container inventory data.
10. **inloc**: Input-only inventory location file (keyed, user-opened).
    - Contains location data.
11. **inmtaj**: Input-only inventory major adjustments file (keyed, user-opened).
    - Contains adjustment data.
12. **inmtrdy**: Input-only inventory meter readings file (keyed, user-opened).
    - Contains tank reading data.
13. **intank**: Input-only tank inventory file (keyed, user-opened).
    - Contains tank inventory data.
14. **intkrv**: Input-only tank receiving file (keyed, user-opened).
    - Contains receiving data.
15. **sa5movd3**: Input-only movement details file (keyed, user-opened).
    - Contains in-transit data.
16. **sa5shz**: Input-only shipment history file (keyed, user-opened).
    - Contains shipment history data.
17. **gsumcv**: Input-only unit measure conversion file (keyed, user-opened).
    - Contains conversion factors.
18. **bicont**: Input-only bill of lading container file (keyed, user-opened).
    - Contains container data for BOL.
19. **arcust**: Input-only customer file (keyed, user-opened).
    - Contains customer data.
20. **shipto**: Input-only ship-to file (keyed, user-opened).
    - Contains ship-to data.
21. **bborcl**: Input-only order close file (keyed, user-opened).
    - Contains order close status.

---

### External Programs Called

The program calls the following external programs:

1. **IN110C**:
   - Called to retrieve additional inventory data or perform related calculations (jk02).
   - Parameters: Assumed to include company, product, container, etc.
2. **IN805BC**:
   - Called for specific inventory balance calculations or data retrieval (jk02, jk03).
   - Parameters: Assumed to include company, product, container, order number, etc.

---

### Summary

The `IN805` RPGLE program, called from the `BB101.ocl36.txt` OCL program, performs inventory balances inquiry for the Customer Orders and Inventory system. It checks product/container availability, calculates in-transit and allocated totals, and displays results in subfiles, with modes to always display (`p$mode="1"`), display on insufficiency (`p$mode="2"`), or send messages (`p$mode="3"`). Business rules enforce inventory calculations, date validation, and mode-specific behavior. The program interacts with 21 files and calls `IN110C` and `IN805BC` for additional processing, ensuring accurate inventory checks for order fulfillment.