The `BB811.rpgle.txt` document is an RPGLE (RPG IV) program called from the `BB101.ocl36.txt` OCL program in an IBM System/36 or AS/400 (now IBM i) environment. It is part of the Customer Orders and Billing system, designed for batch order inquiry, allowing users to review order details, including transaction data, customer information, and freight-related charges. Written by Dave Capo on 04/16/2012, it includes revisions to add order process status (jk02), handle field removal (jk03), and support freight collect service fees (jb04). Below is a detailed explanation of the process steps, business rules, tables (files) used, and external programs called.

---

### Process Steps of the RPGLE Program

The `BB811` program facilitates batch order inquiry by displaying order details across multiple subfiles (`sfl1`, `sfl2`, `sfl3`) on the `bb811d` display file. It retrieves data from order transaction, customer, and related files, allowing users to review orders, including freight collect service charges. The steps are as follows:

1. **Program Initialization**:
   - **File Definitions**:
     - `bb811d`: Work station file (display file) with three subfiles (`sfl1`, `sfl2`, `sfl3`) and relative record numbers (`rrn1`, `rrn2`, `rrn3`).
     - `bbortr`: Input-only file for order transactions (512 bytes, indexed, key length 11, starting at position 2).
     - `arcust`: Input-only file for customer data (keyed, user-opened).
     - `bborcl`: Input-only file for order close data (keyed, user-opened).
     - `bbota1`: Input-only file for order transaction accessorials/marks (keyed, user-opened).
     - `gsctum`: Input-only file for customer unit of measure data (keyed, user-opened).
     - `gstabl`: Input-only file for table data (keyed, user-opened).
     - `shipto`: Input-only file for ship-to data (keyed, user-opened).
   - **Data Structures and Fields**:
     - `klortr`: Data structure for `bbortr` key (11 characters, combining `p$cord` [company/order, 8 characters] and `k$seq` [sequence, 3 digits]).
     - `m@80`: Array for message data (80 elements, 1 character each).
     - `hd1`: Array for format headers (1 element, 80 characters: "Customer Tran-Batch Order Inquiry").
     - `com`: Array for error messages (1 element: "THIS ORDER IS CURRENTLY ON CREDIT HOLD", COM,01).
     - `ovg`, `ovz`: Arrays for file overrides (6 elements, 80 characters each) for `arcust`, `bborcl`, `bbota1`, `gsctum`, `gstabl`, `shipto` in libraries `garcust`, `gbborcl`, `gbbota1`, `ggsctum`, `ggstabl`, `gshipto` (ovg) or `zarcust`, `zbborcl`, `zbbota1`, `zgctum`, `zgstabl`, `zshipto` (ovz).
     - Field prefixes: `f$` (panel fields), `c$` (subfile control), `s1`/`s2`/`s3` (subfile fields), `k$` (key lists), `p$` (input parameters), `o$` (output parameters), `r$` (reposition subfile), `w$` (work fields).
   - **Key Lists**:
     - `klcust`: For `arcust` (company `f$co`, customer `bocust`).
     - `klship`: For `shipto` (company `f$co`, customer `bocust`, ship-to `csship`).
     - `klord999`: For `bbortr` (company `f$co`, order number `f$ord#`, sequence `k$999=999`).
     - `klctum`: For `gsctum` (company `boco`, product `bdprod`, container `bdcntr`, unit of measure `bdum`).
     - `klc1r1`, `klc2r1`, `klc3r1`: For `bbortr`/`bbota1` (company `f$co`, order number `f$ord#`).
     - `klc1s1`, `klc2s1`, `klc3s1`: For subfile-specific records (company `f$co`, order number `f$ord#`, sequence `c1seq`/`c2seq` or accessorial ID `c$rcid`).
   - **Indicators**:
     - 19: Panel format input change.
     - 21-39, 50-69: Screen errors.
     - 40: Subfile clear.
     - 41: Subfile display control.
     - 42: Subfile.
     - 43: Subfile end and next change.
     - 49: Message subfile display/control/initialize/end.
     - 70-79: Input field protect.
     - 80: Primary file chain.
     - 88: Read subfile.
     - 90-99: Secondary file chain/read.

2. **Parameter Input**:
   - Receives input parameters via `*ENTRY PLIST` (assumed, not shown) including company (`f$co`), order number (`f$ord#`), and possibly customer (`bocust`) or ship-to (`csship`).

3. **Order Inquiry Processing**:
   - Opens files (`arcust`, `bborcl`, `bbota1`, `gsctum`, `gstabl`, `shipto`).
   - Chains to `bbortr` using `klortr` (company/order `p$cord`, sequence `k$seq`) to retrieve order transaction details (indicator 80).
   - Chains to `arcust` using `klcust` to retrieve customer details.
   - Chains to `shipto` using `klship` to retrieve ship-to details.
   - Chains to `gsctum` using `klctum` to retrieve unit of measure data for the product/container.
   - Chains to `bbota1` using `klc3s1` to retrieve accessorials/marks, including freight collect service fees (misc sequence 940, jb04).
   - Chains to `bborcl` to check order close status.
   - Checks if the order is on credit hold, setting error message "THIS ORDER IS CURRENTLY ON CREDIT HOLD" (COM,01) if applicable.

4. **Subfile Population**:
   - Clears subfiles (`*in40`) and initializes relative record numbers (`rrn1`, `rrn2`, `rrn3`).
   - Populates subfiles based on format `FMT01` (jk02):
     - `sfl1`: Displays order header details (e.g., company, order number, customer, ship-to, order process status, jk02).
     - `sfl2`: Displays order transaction details (e.g., product, container, quantity, unit of measure).
     - `sfl3`: Displays accessorials/marks, including freight collect service fees (misc sequence 940, jb04).
   - Increments `rrn1`, `rrn2`, `rrn3` for each record written to the respective subfiles.

5. **Display and User Interaction**:
   - Displays the `bb811d` panel with format header "Customer Tran-Batch Order Inquiry".
   - Writes subfile control formats and displays subfiles (`*in41`, `*in42`) using `exfmt`.
   - Allows user to review order details, including freight collect service fees (jb04).
   - In inquiry mode, protects input fields (`*in70`).
   - Supports navigation and exit (e.g., F12, assumed).

6. **Program Termination**:
   - Sets the last record indicator (`LR`) to exit the program.
   - Returns output parameters (`o$` fields, assumed) to the calling program.

---

### Business Rules

The program enforces the following business rules:

1. **Order Inquiry**:
   - Retrieves and displays order details from `bbortr`, including header, transaction, and accessorial data.
   - Includes order process status in the display format `FMT01` (jk02).
   - Supports inquiry of freight collect service fees (misc sequence 940, $100 charge, jb04) when shipping is arranged by ARG but billed to the customer by the carrier.

2. **Credit Hold Check**:
   - Checks if the order is on credit hold, displaying error message "THIS ORDER IS CURRENTLY ON CREDIT HOLD" (COM,01) if applicable.

3. **Data Validation**:
   - Validates company, customer, ship-to, product, container, and unit of measure data using `arcust`, `shipto`, `gsctum`, and `gstabl`.
   - Retrieves accessorials/marks from `bbota1`, including freight-related charges (jb04).
   - Checks order close status in `bborcl`.

4. **File Overrides**:
   - Uses override arrays (`ovg`, `ovz`) to redirect files to production (`g*`) or alternate (`z*`) libraries (e.g., test environments).
   - Ensures correct file access based on the environment.

5. **Inquiry Mode**:
   - Protects input fields (`*in70`) in inquiry mode, allowing only review of data.

6. **Error Handling**:
   - Displays error messages via the `com` array in the subfile, primarily for credit hold status.

---

### Tables (Files) Used

The program interacts with the following files:

1. **bb811d**: Work station file (display file) with subfiles `sfl1`, `sfl2`, `sfl3`.
   - Fields: `f$co` (company), `f$ord#` (order number), `s1seq`/`c1seq` (sequence for `sfl1`), `c2seq` (sequence for `sfl2`), `c$rcid` (accessorial ID for `sfl3`).
2. **bbortr**: Input-only order transaction file (512 bytes, indexed, key length 11, starting at position 2).
   - Fields: `boco` (company), `bdprod` (product), `bdcntr` (container), `bdum` (unit of measure).
3. **arcust**: Input-only customer file (keyed, user-opened).
   - Fields: `bocust` (customer number).
4. **bborcl**: Input-only order close file (keyed, user-opened).
   - Contains order close status.
5. **bbota1**: Input-only order transaction accessorials/marks file (keyed, user-opened).
   - Fields: Accessorial ID, type, code, description, including misc sequence 940 for freight collect (jb04).
6. **gsctum**: Input-only customer unit of measure file (keyed, user-opened).
   - Fields: Unit of measure for product/container combinations.
7. **gstabl**: Input-only table file (keyed, user-opened).
   - Contains reference data (e.g., codes, types).
8. **shipto**: Input-only ship-to file (keyed, user-opened).
   - Fields: `csship` (ship-to number).

---

### External Programs Called

The `BB811` RPGLE program does not call any external programs. All processing is handled internally through file operations and subroutines (assumed for subfile processing, not shown in the snippet).

---

### Summary

The `BB811` RPGLE program, called from the `BB101.ocl36.txt` OCL program, provides batch order inquiry for the Customer Orders and Billing system. It retrieves and displays order details from `bbortr`, `arcust`, `shipto`, `bborcl`, `bbota1`, `gsctum`, and `gstabl` across three subfiles, including order process status (jk02) and freight collect service fees (jb04). Business rules ensure valid data retrieval, credit hold checks, and inquiry mode protection. The program interacts with eight files and does not call external programs, relying on internal logic to facilitate user review of order transactions.