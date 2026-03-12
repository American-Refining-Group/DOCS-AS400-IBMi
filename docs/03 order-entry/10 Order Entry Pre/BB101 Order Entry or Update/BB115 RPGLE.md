The `BB115.rpgle.txt` document is an RPGLE (RPG IV) program called from the `BB101.ocl36.txt` OCL program in an IBM System/36 or AS/400 (now IBM i) environment. It is part of the Customer Order Entry system, designed to check for duplicate customer orders based on two options: Option A (company/customer/ship-to/product) or Option B (company/customer/ship-to/product/PO number, default). Written by Dave Capo on 04/02/2011, it includes revisions to set Option B as the default (jb01) and replace the `gstabl` file with `gsprod` (jk01). Below is a detailed explanation of the process steps, business rules, tables (files) used, and external programs called.

---

### Process Steps of the RPGLE Program

The `BB115` program checks for duplicate orders by querying order header and detail files, displaying potential duplicates in a subfile on the `bb115d` display file for user review. The steps are as follows:

1. **Program Initialization**:
   - **File Definitions**:
     - `bb115d`: Work station file (display file) with subfile `sfl1` and control format, user-opened.
     - **Group 1 (Duplicate Check)**:
       - `arcusp`: Input-only file for customer data (keyed, user-opened).
       - `bicont`: Input-only file for container data (keyed, user-opened).
       - `bboh2`: Input-only file for order headers with renamed format `bbohp2` (keyed, user-opened).
       - `bboh3`: Input-only file for order headers with renamed format `bbohp3` (keyed, user-opened).
     - **Group 2 (Post-Duplicate Check)**:
       - `arcust`: Input-only file for customer data (keyed, user-opened).
       - `bbordd`: Input-only file for order details (keyed, user-opened).
       - `gsprod`: Input-only file for product data (keyed, user-opened, replaced `gstabl` in jk01).
       - `sa5fiud`: Input-only file for sales agreement data (keyed, user-opened).
       - `sa5shz`: Input-only file for shipment history (keyed, user-opened).
       - `shipto`: Input-only file for ship-to data (keyed, user-opened).
   - **Data Structures and Fields**:
     - `dspf_ds`: Display file data structure for feedback (record name, key, cursor location, page RRN).
     - `ovg1`, `ovz1`: Arrays for file overrides for Group 1 files (`arcusp`, `bboh2`, `bboh3`, `bicont`) in libraries `garcusp`, `gbboh2`, `gbboh3`, `gbicont` (ovg1) or `zarcusp`, `zbboh2`, `zbboh3`, `zbicont` (ovz1).
     - `ovg2`, `ovz2`: Arrays for file overrides for Group 2 files (`arcust`, `bbordd`, `gsprod`, `sa5fiud`, `sa5shz`, `shipto`) in libraries `garcust`, `gbbordd`, `ggsprod`, `gsa5fiud`, `gsa5shz`, `gshipto` (ovg2) or `zarcust`, `zbbordd`, `zgsprod`, `zsa5fiud`, `zsa5shz`, `zshipto` (ovz2).
     - `com`: Array for error messages (1 element defined: "No Records", COM,01).
     - Field prefixes: `f$` (panel fields), `c$` (subfile control), `s1` (subfile fields), `s2` (unused), `k$` (key lists), `p$` (input parameters), `o$` (output parameters), `r$` (reposition subfile), `w$` (work fields).
   - **Key Lists**:
     - `kloh2r1`, `kloh3r1`: For `bboh2`/`bboh3` (company, customer, ship-to, product, [PO number for Option B]).
     - `klcust`: For `arcusp`/`arcust` (company, customer).
     - `klship`: For `shipto` (company, customer, ship-to).
     - `klordh`: For order headers (company, order number).
     - `klprod`: For `gsprod` (company, product, jk01).
     - `klshz`: For `sa5shz` (company, customer, invoice number, order number).
   - **Indicators**:
     - 19: Panel format input change.
     - 21-39, 50-69: Screen errors.
     - 40: Subfile clear.
     - 41: Subfile display control.
     - 42: Subfile display.
     - 43: Subfile end and next change.
     - 49: Message subfile display/control/initialize/end.
     - 70: Global protect in inquiry mode.
     - 80: Primary file chain.
     - 88: Read subfile.
     - 90-99: Secondary file chain/read.

2. **Parameter Input**:
   - Receives input parameters (assumed via `*ENTRY PLIST`, not shown) including company (`c$co`), customer (`c$cust`), ship-to (`c$ship`), product (`c$prod`), PO number (`c$pord`), order date (`c$odat`), and checking option (Option A or B, default B, jb01).

3. **Duplicate Order Check**:
   - Opens Group 1 files (`arcusp`, `bicont`, `bboh2`, `bboh3`).
   - **Option A (Company/Customer/Ship-to/Product)**:
     - Uses key list `kloh2r1` to query `bboh2` for matching records (company, customer, ship-to, product).
   - **Option B (Company/Customer/Ship-to/Product/PO Number, default, jb01)**:
     - Uses key list `kloh3r1` to query `bboh3` for matching records (company, customer, ship-to, product, PO number).
   - Chains to `arcusp` and `bicont` using `klcust` to validate customer and container data.
   - For each matching record:
     - Populates subfile `sfl1` with order details (e.g., order number, date, customer, ship-to, product, PO number).
     - Increments relative record number (`rrn1`).

4. **Post-Duplicate Check Processing**:
   - Opens Group 2 files (`arcust`, `bbordd`, `gsprod`, `sa5fiud`, `sa5shz`, `shipto`).
   - If duplicates are found:
     - Chains to `arcust` (`klcust`) for customer details.
     - Chains to `shipto` (`klship`) for ship-to details.
     - Chains to `gsprod` (`klprod`) for product details (jk01).
     - Chains to `bbordd` for order details.
     - Chains to `sa5fiud` and `sa5shz` (`klshz`) for sales agreement and shipment history.
     - Displays duplicates in the subfile for user review.
   - If no duplicates are found, sets error message "No Records" (COM,01) and displays it in the subfile.

5. **Subfile Display and User Interaction**:
   - Clears the subfile (`*in40`) and initializes `rrn1`.
   - Writes subfile records (`sfl1`) with duplicate order details.
   - Displays the subfile control format (`sflctl1`) using `exfmt`.
   - Allows user to review duplicates or exit (e.g., via F12, not shown in snippet).
   - In inquiry mode, protects input fields (`*in70`).

6. **Program Termination**:
   - Sets the last record indicator (`LR`) to exit the program.
   - Returns results (e.g., selected order or confirmation to proceed) via output parameters (`o$` fields, assumed).

---

### Business Rules

The program enforces the following business rules:

1. **Duplicate Order Checking**:
   - **Option A**: Checks for duplicate orders based on company, customer, ship-to, and product.
   - **Option B (default, jb01)**: Checks for duplicates based on company, customer, ship-to, product, and PO number, providing more specific matching.
   - Displays potential duplicates in a subfile for user review to prevent redundant order entry.

2. **Data Validation**:
   - Validates customer, ship-to, product, and container data using `arcusp`, `arcust`, `shipto`, `gsprod`, and `bicont`.
   - Retrieves order details from `bbordd` and sales/shipment data from `sa5fiud` and `sa5shz` for comprehensive duplicate checking.

3. **File Overrides**:
   - Uses override arrays (`ovg1`, `ovz1`, `ovg2`, `ovz2`) to redirect files to production (`g*`) or alternate (`z*`) libraries (e.g., test environments).
   - Ensures correct file access based on the environment.

4. **Error Handling**:
   - Displays "No Records" (COM,01) if no duplicates are found, informing the user that the order is unique.
   - Relies on the calling program (`BB101`) to handle further validation or errors.

5. **Inquiry Mode**:
   - Protects input fields (`*in70`) when in inquiry mode, ensuring users cannot modify data during review.

---

### Tables (Files) Used

The program interacts with the following files:

1. **bb115d**: Work station file (display file) with subfile `sfl1` and control format `sflctl1`.
   - Fields: Subfile fields (`s1*`), control fields (`c$co`, `c$cust`, `c$ship`, `c$prod`, `c$pord`, `c$odat`).
2. **arcusp**: Input-only customer file (keyed, user-opened, Group 1).
   - Used for customer validation during duplicate check.
3. **bicont**: Input-only container file (keyed, user-opened, Group 1).
   - Used for container validation.
4. **bboh2**: Input-only order header file with format `bbohp2` (keyed, user-opened, Group 1).
   - Used for Option A duplicate check (company/customer/ship-to/product).
5. **bboh3**: Input-only order header file with format `bbohp3` (keyed, user-opened, Group 1).
   - Used for Option B duplicate check (company/customer/ship-to/product/PO number).
6. **arcust**: Input-only customer file (keyed, user-opened, Group 2).
   - Used for customer details post-duplicate check.
7. **bbordd**: Input-only order detail file (keyed, user-opened, Group 2).
   - Used for order detail retrieval.
8. **gsprod**: Input-only product file (keyed, user-opened, Group 2, replaced `gstabl` in jk01).
   - Used for product validation.
9. **sa5fiud**: Input-only sales agreement file (keyed, user-opened, Group 2).
   - Used for sales agreement data.
10. **sa5shz**: Input-only shipment history file (keyed, user-opened, Group 2).
    - Used for shipment history data.
11. **shipto**: Input-only ship-to file (keyed, user-opened, Group 2).
    - Used for ship-to details.

---

### External Programs Called

The `BB115` RPGLE program does not call any external programs. All processing is handled internally through file operations and subroutines (assumed, e.g., for subfile processing, not shown in the snippet).

---

### Summary

The `BB115` RPGLE program, called from the `BB101.ocl36.txt` OCL program, checks for duplicate customer orders in the Customer Order Entry system using Option A (company/customer/ship-to/product) or Option B (company/customer/ship-to/product/PO number, default). It queries `bboh2` or `bboh3` for duplicates, validates data with `arcusp`, `bicont`, `arcust`, `gsprod`, `shipto`, `sa5fiud`, and `sa5shz`, and displays results in a subfile for user review. Business rules ensure accurate duplicate detection, proper file overrides, and inquiry mode protection. The program interacts with 11 files and does not call external programs, relying on internal logic for all operations.