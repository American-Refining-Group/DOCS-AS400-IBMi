The `AP125P.rpgle` program is an RPGLE program running on an IBM AS/400 (iSeries) system, designed for "Freight Invoice Import Selection" as part of an Accounts Payable (A/P) process. It is called from the `AP125.ocl36.txt` OCL script and facilitates the selection and management of freight invoices for batch processing. Below, I will explain the process steps, business rules, tables used, and external programs called, based on the provided source code.

---

### **Process Steps of the AP125P Program**

The program is structured to manage a subfile (SFL) interface for selecting freight invoices, allowing users to add or remove invoices from a batch, validate inputs, and create batches for further processing. The main process steps are:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Parameters**: Receives input parameters:
     - `a$co`: Company code (2 digits).
     - `p$inty`: Invoice type filter (`P` for paper, `N` for non-paper).
     - `p$mode`: Run mode (`MNT` for maintenance, otherwise inquiry).
     - `p$fgrp`: File group (`G` or `Z`, determining which library files to use).
   - Initializes work fields, subfile control fields (e.g., `rrn1`, `pagsz1`), and key lists for file access.
   - Sets up message handling and default headers/comments.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - Overrides database files based on the `p$fgrp` parameter (`G` or `Z`) using `QCMDEXC` to execute override commands (e.g., `ovrdbf` for `apcont`, `frbinh`, etc.).
   - Opens input files (`apcont`, `frbinh`, `frcinh`, `frcfbh`, `bbcaid`, `frcinhj4`) and the add/update file (`ap125pw`).

3. **Create or Clear Work File (`crtwrkf` Subroutine)**:
   - Calls the external program `AP125PC` to create or clear the work file `ap125pw`.
   - Overrides the `ap125pw` file to `qtemp/ap125pw` with `share(*no)`.
   - Opens the `ap125pw` file for add/update operations.

4. **Process Subfile (`srsfl1` Subroutine)**:
   - **Clear Message Subfile**: Clears any existing messages (`clrmsg`) and writes initial messages (`wrtmsg`).
   - **Initialize Subfile**: Sets the subfile mode to folded (`sfmod1 = '1'`, `*in45 = *on`) and initializes control fields (e.g., `c1co` from `a$co`, `c1mode` to `All`).
   - **Global Protection**: Sets `*in70` based on `p$mode` (`*off` for `MNT`, `*on` for inquiry mode, protecting fields in inquiry mode).
   - **Position File**: Calls `sf1rep` to position the file and load the subfile.
   - **Main Loop**:
     - Displays the subfile control (`sflctl1`) and command line (`sflcmd1`).
     - Checks for subfile records to enable display (`*in41`).
     - Handles user inputs (function keys and Enter):
       - **F03 (Exit)**: Checks for pending selections in `ap125pw`. If present, displays a confirmation window (`f03wdw`) via `sf1f03`. Exits if confirmed.
       - **F04 (Field Prompting)**: Calls `prompt` to handle field prompting (e.g., for `c1caid` using `LBBCAID`).
       - **F05 (All/Review Toggle)**: Toggles between `All` and `Review` modes, updating `c1mode` and repositioning the subfile.
       - **F06 (Create Batch and Exit)**: Displays a confirmation window (`f06wdw`) via `sf1crtbat` and creates a batch by calling `crtbat`.
       - **F22 (Add All)**: Adds all displayed invoices to the batch via `sf1prcall`.
       - **F23 (Remove All)**: Removes all displayed invoices from the batch via `sf1prcall`.
       - **Page Down**: Loads additional subfile records (`sf1lod`).
       - **Enter**: Processes subfile changes (`sf1prc`).
       - **Positioning**: Repositions the subfile if `c1caid`, `c1fmdy`, or `c1tmdy` change (`sf1rep`).
       - **F10**: Moves the cursor to the control record.

5. **Process Subfile on Enter (`sf1prc` Subroutine)**:
   - Reads changed subfile records (`readc sfl1`) and processes options (`sf1chg`).
   - Handles options:
     - **Option 1 (Add)**: Adds the invoice to the batch (`sf1s01`) if in `MNT` mode.
     - **Option 4 (Remove)**: Removes the invoice from the batch (`sf1s04`) if in `MNT` mode.
   - Updates subfile fields and colors (`sf1fmt`, `sf1col`) and writes the updated record.

6. **Reposition Subfile (`sf1rep` Subroutine)**:
   - Clears the subfile (`sf1clr`) and resets the relative record number (`rrn1`).
   - Validates control fields (`sf1cte`) and positions the file (`frcinhj4`) using `setll`.
   - Loads subfile records (`sf1lod`) and retains control field values for repositioning.

7. **Edit Subfile Control Input (`sf1cte` Subroutine)**:
   - Validates:
     - **Company (`c1co`)**: Chains to `apcont` to verify the company code and retrieve the company name (`c1conm`).
     - **Carrier ID (`c1caid`)**: Chains to `bbcaid` to verify the carrier ID and retrieve the carrier name (`c1crnm`). If blank, sets to `*All`.
     - **From/To Invoice Dates (`c1fmdy`, `c1tmdy`)**: Calls `GSDTEDIT` to validate dates and convert to `c1fdat`/`c1tdat`. Errors trigger messages (`ERR0020`).
   - Adds error messages to the message subfile if validation fails.

8. **Load Subfile Records (`sf1lod` Subroutine)**:
   - Reads records from `frcinhj4` (replacing `frcinh4`) for the specified company (`c1co`).
   - Retrieves the order ship date (`boshd8`) from `frbinh` (replacing invoice date per revision `dc01`).
   - Applies filters (`sf1fltr`) and skips records that don’t match criteria.
   - Formats (`sf1fmt`) and color-codes (`sf1col`) subfile lines, then writes them to the subfile.
   - Sets `s1nosflrecs` if no records are found.

9. **Apply Filters (`sf1fltr` Subroutine)**:
   - Filters records based on:
     - Approval status (`fralst ≠ 'Y'` or `frapst ≠ 'N'`).
     - Invoice type (`frinty` must match `p$inty`).
     - Non-zero invoice amount (`s1inam ≠ 0`, per revision `jb03`).
     - Carrier ID (`c1caid` must match `frcaid` if specified).
   - Sets `s1fltr` to `*on` to skip records that fail filters.

10. **Format Subfile Line (`sf1fmt` Subroutine)**:
    - Populates subfile fields (`s1caid`, `s1cain`, `s1inty`, `s1rdno`, `s1srn`, `s1inam`, `s1smdy`) from `frcinhj4` or `frbinh`.
    - Adjusts invoice amount (`s1inam = frinam - frfboa`, per revision `jb02`).
    - Checks if the record is already in `ap125pw` and sets `s1flag` to `'*'` if present.

11. **Color Coding (`sf1col` Subroutine)**:
    - Placeholder for color-coding subfile records (currently empty but reserved for visual indicators).

12. **Add Invoice to Batch (`sf1s01` Subroutine)**:
    - Chains to `ap125pw` to check if the record exists.
    - If not, writes a new record with company, carrier ID, invoice number, record type (`s1rcd`), and reference number (`s1rdno`).
    - Updates the total invoice amount (`c1inam`).

13. **Remove Invoice from Batch (`sf1s04` Subroutine)**:
    - Chains to `ap125pw` to locate the record.
    - If found, deletes it and subtracts the invoice amount from `c1inam`.

14. **Process All Records (`sf1prcall` and `sf1prcall2` Subroutines)**:
    - Displays a window (`addwdw` for adding, `rmvwdw` for removing) and processes all displayed records.
    - Iterates through `frcinhj4`, applies filters, and calls `sf1s01` (add) or `sf1s04` (remove) based on `w$add`.

15. **Create Batch (`crtbat` Subroutine)**:
    - Reads `ap125pw` records and chains to `frcinh` or `frcfbh` based on `w1rcd`.
    - For approved (`fralst = 'Y'`, `frapst = 'N'`) records, calls `AP125` with parameters (`o$co`, `o$caid`, `o$cain`, `o$rcd`, `o$rdno`) to create the batch.

16. **Field Prompting (`prompt` Subroutine)**:
    - For the `C1CAID` field, calls `LBBCAID` to prompt for a carrier ID, passing `c1co`, `c1caid`, and `p$fgrp`.

17. **Message Handling (`addmsg`, `wrtmsg`, `clrmsg` Subroutines)**:
    - `addmsg`: Sends error messages (e.g., `ERR0010`, `ERR0020`) to the program message queue using `QMHSNDPM`.
    - `wrtmsg`: Displays the message subfile (`msgctl`).
    - `clrmsg`: Clears the message subfile using `QMHRMVPM`.

18. **Program Termination**:
    - Closes all files and sets `*inlr = *on` to end the program.

---

### **Business Rules**

The program enforces the following business rules:
1. **Invoice Filtering**:
   - Only approved invoices (`fralst = 'Y'`, `frapst = 'N'`) are eligible for selection.
   - Invoices must match the specified invoice type (`p$inty`, `P` or `N`).
   - Invoices with zero amounts are excluded (revision `jb03`).
   - Carrier ID (`c1caid`) must match if specified, otherwise all carriers are included.

2. **Invoice Amount Adjustment**:
   - The displayed invoice amount (`s1inam`) is calculated as `frinam - frfboa` (Freight Balancing Order Override Total, revision `jb02`).

3. **Date Usage**:
   - Uses the order ship date (`boshd8` from `frbinh`) instead of the invoice date (`frindt`, revision `dc01`) for filtering and display.

4. **Batch Processing**:
   - Invoices are added to or removed from `ap125pw` based on user selections (options 1 or 4, or F22/F23).
   - Batch creation (`crtbat`) only processes approved invoices with valid status codes.

5. **Mode-Based Access**:
   - In `MNT` mode, users can add/remove invoices and create batches.
   - In inquiry mode, fields are protected (`*in70 = *on`), preventing modifications.

6. **Validation**:
   - Company code (`c1co`) must exist in `apcont`.
   - Carrier ID (`c1caid`) must exist in `bbcaid` (revision `jk01`).
   - Dates (`c1fmdy`, `c1tmdy`) must be valid, checked via `GSDTEDIT`.

7. **Subfile Behavior**:
   - Supports `All` (all eligible invoices) and `Review` (only invoices in `ap125pw`) modes.
   - Displays a "No Records Found" message if no invoices meet the criteria.

---

### **Tables (Files) Used**

1. **Display File**:
   - `ap125pd`: Display file with subfile `sfl1`, used for the user interface.

2. **Input Files**:
   - `apcont`: Accounts Payable control file (company data).
   - `frbinh`: Freight Out Balancing Customer Invoice Header (for order ship date).
   - `frcinh`: Freight Invoice Header.
   - `frcfbh`: Freight Billed Balance Header (revision `jb02`).
   - `bbcaid`: Carrier ID table (replacing `gstabl`, revision `jk01`).
   - `frcinhj4`: Multi-file logical file combining `frcinh` and `frcfbh` (replacing `frcinh4`, revision `jb02`).

3. **Add/Update File**:
   - `ap125pw`: Work file in `qtemp` for storing selected invoices.

---

### **External Programs Called**

1. **AP125PC**:
   - Called in `crtwrkf` to create or clear the `ap125pw` work file.

2. **GSDTEDIT**:
   - Called in `sf1cte` to validate and convert dates (`c1fmdy`, `c1tmdy`).

3. **AP125**:
   - Called in `crtbat` to create the invoice batch, passing company, carrier ID, invoice number, record type, and reference number.

4. **LBBCAID**:
   - Called in `prompt` to provide carrier ID prompting, replacing `LGSTABL` (revision `jk01`).

5. **QCMDEXC**:
   - System program to execute override commands (`ovrdbf`) in `opntbl` and `crtwrkf`.

6. **QMHSNDPM**:
   - System program to send error messages to the program message queue in `addmsg`.

7. **QMHRMVPM**:
   - System program to clear the message subfile in `clrmsg`.

---

### **Summary**

The `AP125P` program is a user-interactive RPGLE program for selecting freight invoices for batch processing. It uses a subfile interface to display invoices from `frcinhj4`, allows users to add/remove invoices to/from `ap125pw`, and creates batches via `AP125`. Key features include filtering by approval status, invoice type, and non-zero amounts, using order ship dates, and supporting maintenance/inquiry modes. The program integrates with multiple files and external programs to validate data and manage the batch process.

If you need further details or clarification on specific subroutines, business rules, or file structures, please let me know!