The provided `AR890.rpgle.txt` is an RPGLE (Report Program Generator Language Enhanced) program used for a **Customer Master Inquiry** system on an IBM midrange platform (e.g., AS/400 or IBM i). It is called by the `AR890.ocl36` OCL script previously analyzed. The program facilitates inquiry into customer data, including account details, supplemental information, product codes, and contacts, with support for multiple display formats. Below, I’ll explain the **process steps**, **business rules**, **tables used**, and **external programs called**, based on the source code.

---

### **Process Steps of the RPGLE Program**

The `AR890` RPGLE program is structured to handle interactive customer inquiries through a workstation display file (`ar890d`). It processes user inputs, retrieves data from various files, and displays results across five screen formats (`AR890S01` to `AR890S05`). The main steps are:

1. **Program Initialization (`*inzsr` Subroutine):**
   - Initializes key fields (`cocust`, `ship`, `o$co`, `o$cust`, `o$ship`) to zeros, setting up the program for a fresh start.
   - Defines the local data area (`*lda`) to access runtime parameters (e.g., `ldacc` for company/customer number, `tstprd` for library/file group).

2. **Read Workstation Input:**
   - Checks the control field `qsctl`. If blank, sets indicator `*in09` and `*in01` to `1`, indicating a new session, and sets `qsctl` to `'R'`.
   - Reads input from one of five display formats (`AR890S01` to `AR890S05`) based on indicators `*in81` to `*in85`. If an invalid read occurs (indicator `*in09`), it calls the `rollky` subroutine to handle roll keys.

3. **Handle Function Keys:**
   - **Clear Key (`*inka`)**: Clears all fields using the `clear` subroutine, resets indicators, and displays the initial screen (`AR890S01`).
   - **End of Job (`*inkg`)**: Sets the last record indicator (`*inlr`) to terminate the program and clears relevant indicators.
   - **Roll Keys (`*in18`, `*in19`)**: Handles roll forward (up) and roll backward (down) for navigating customer records using `rollfw` and `rollbw` subroutines.
   - **Command Keys (`ke`, `kf`, `kh`, `02`, `03`, `04`)**: Trigger specific inquiries:
     - **F2 (`ke` and `02`)**: Customer history inquiry (`ARCUST` or `ARCUSP`).
     - **F3 (`ke` and `03`)**: Customer form type contacts inquiry (originally called `AR915P`, now commented out).
     - **F4 (`kf` and `04`)**: Customer product code history (originally called `GB730P`, now commented out; replaced by `BI907AC` for `ARCUPR` maintenance).
     - **F2 (`kh` and `02`)**: Supplemental file inquiry (`ARCUSP`).

4. **Screen Processing Subroutines:**
   - **s1 (Screen 1 - `AR890S01`)**:
     - Validates company number (`co`) against `ARCONT`. If invalid, displays error message (`msg1` = "INVALID COMPANY NUMBER ENTERED").
     - Chains to `ARCUST` and `ARCUSP` using `arkey` (company/customer number). If not found, displays "CUSTOMER NOT FOUND".
     - Checks for EFT (Electronic Funds Transfer) data and sets `*in60` if valid.
     - Chains to `BICONT` to get invoicing style (`bcinst`) and sets the header (`s4head`) for screen 4.
     - Calls `getcus` to retrieve customer data and `getsup` for supplemental data.
     - Sets `*in82` to display screen 2 (`AR890S02`).
   - **s2 (Screen 2 - `AR890S02`)**:
     - Sets `*in83` to display screen 3 (`AR890S03`).
   - **s3 (Screen 3 - `AR890S03`)**:
     - Prepares to display customer ship-to products but skips direct `ARCUPR` processing, instead calling `BI907AC` (commented out in the code but referenced in comments).
     - Sets `*in85` to display screen 5 (`AR890S05`).
   - **s4 (Screen 4 - `AR890S04`)**:
     - Clears arrays for product codes, descriptions, and related fields.
     - If array index `x` reaches 18, reads previous `GSPROD` record and fills arrays via `filara`.
     - If invoicing style (`bcinst`) is '5', displays screen 5; otherwise, calls `s5`.
   - **s5 (Screen 5 - `AR890S05`)**:
     - Clears fields using `clear` and sets `*in81` to return to screen 1.

5. **Data Retrieval Subroutines:**
   - **getcus**:
     - Checks if the customer is deleted (`ardel = 'D'`). If so, displays "THIS CUSTOMER WAS PREVIOUSLY DELETED".
     - Moves customer data from `ARCUST` (e.g., name, address, financials) to display fields.
     - Retrieves descriptions for salesman (`sls#`), terms (`term`), group (`grup`), and class (`cucl`) from `GSTABL`.
   - **getsup**:
     - Checks if the supplemental record is deleted (`csdel = 'D'`). If so, displays the deletion message.
     - Converts dates (`csstdt`, `csfsdt`, `csicdt`) to MMDDYY format.
     - Moves supplemental data (e.g., tax codes, comments, freight info) to display fields.
   - **filara**:
     - Fills arrays for screen 4 (`prcd`, `prds`, `glcd`, `stno`, `pfrc`, `psfr`, `pcfr`) with product data from `GSPROD` and `ARCUPR`.
     - Stops when 17 records are filled or end of file is reached (`*in70`).

6. **Roll Key Handling:**
   - **rollky**: Detects roll forward (`status = 01122`) or backward (`status = 01123`) and clears function key indicators.
   - **rollfw**: Moves to the next customer record in `ARCUST` and updates `arkey`.
   - **rollbw**: Moves to the previous customer record in `ARCUST` and updates `arkey`.

7. **Write to Display:**
   - Writes to the appropriate screen format (`AR890S01` to `AR890S05`) based on indicators `*in81` to `*in85`.

8. **Termination:**
   - If `*inu8` and `*in81` are on, sets `*inlr` to terminate the program.

---

### **Business Rules**

The program enforces the following business rules based on the code and comments:

1. **Customer Validation**:
   - Validates company number (`co`) against `ARCONT`. If invalid, displays "INVALID COMPANY NUMBER ENTERED".
   - Checks if customer exists in `ARCUST` and `ARCUSP`. If not found, displays "CUSTOMER NOT FOUND".
   - Checks for deletion status (`ardel` or `csdel = 'D'`). If deleted, displays "THIS CUSTOMER WAS PREVIOUSLY DELETED".

2. **EFT Validation**:
   - If `areft = 'Y'` (EFT participant), verifies that `csarte` (ACH bank routing code) and `csabk#` (ACH bank account number) are non-blank/zero, setting `*in60` to indicate valid EFT data.

3. **Invoicing Style**:
   - Uses `bcinst` from `BICONT` to determine the header for screen 4 (`s4head`). If `bcinst = '5'`, uses alternate header ("To Bill Gross Gallons Enter 'G'"); otherwise, uses default ("To Bill Net Gallons Enter 'N'").

4. **Product Code Handling**:
   - Originally processed `ARCUPR` directly for product codes but now calls `BI907AC` for maintenance (per revision `JB06`).
   - Limits array filling to 17 records for product codes (`prcd`), descriptions (`prds`), and related fields.

5. **Navigation**:
   - Supports roll forward/backward to navigate customer records.
   - Command keys (F2, F3, F4) trigger specific inquiries, with F3 and F4 linked to external programs (`AR915P`, `BI907AC`) for detailed processing.

6. **Data Display**:
   - Formats financial data (e.g., `artotd`, `arcurd`, `arfin$`) with specific decimal places.
   - Converts dates (e.g., `arhidt`, `csstdt`) from YYMMDD to MMDDYY for display.
   - Retrieves descriptive text for salesman, terms, group, and class codes from `GSTABL`.

7. **Error Handling**:
   - Displays error messages (`msg1`, `msg2`) for invalid inputs or missing records.
   - Sets `*in90` for error conditions, triggering error message display.

---

### **Tables (Files) Used**

The program uses the following files, defined in the file specifications (`F-specs`):

1. **`ar890d` (Workstation File)**:
   - Display file for interactive user interface, using formats `AR890S01` to `AR890S05`.
   - Handled by `PROFOUNDUI(HANDLER)` for modern UI rendering.

2. **`arcust` (Customer Master File)**:
   - Input file, 384 bytes, keyed by company/customer number (`keyloc(2)`).
   - Contains core customer data (e.g., name, address, financials, credit limit).

3. **`arcusp` (Customer Supplemental File)**:
   - Input file, 1344 bytes, keyed by company/customer number.
   - Stores supplemental data (e.g., tax codes, comments, freight details).

4. **`arcupr` (Customer Product Master)**:
   - Input file, 80 bytes, keyed by company/customer/ship-to/product.
   - Holds product-specific data (e.g., product code, freight codes).

5. **`gsprod` (Product File)**:
   - Input file, 512 bytes, keyed by company/product code.
   - Contains product codes and descriptions.

6. **`arcont` (Contact File)**:
   - Input file, 256 bytes, keyed by company number.
   - Stores ageing period limits (`aclmt1` to `aclmt4`).

7. **`bicont` (Billing Contact File)**:
   - Input file, 256 bytes, keyed by company number.
   - Contains invoicing style (`bcinst`).

8. **`gstabl` (Table File)**:
   - Input file, 256 bytes, keyed by table type/code.
   - Stores descriptions for salesman, terms, group, and class codes.

9. **`gscont` (General Contact File)**:
   - Input file, externally defined, keyed (assumed by company number).
   - Provides default company number (`gxcono`).

---

### **External Programs Called**

The program references the following external programs, though some calls are commented out, indicating they may be invoked conditionally or replaced:

1. **`GB730P`** (Commented Out):
   - Previously called for inquiries on `ARCUST`, `ARCUSP`, and `ARCUPR`.
   - Parameters included `parmst` (data structure with file, company, customer, ship-to).

2. **`AR915P`** (Commented Out):
   - Previously called for customer form type contacts inquiry (`ARCUFM`, `ARCUFMX`).
   - Parameters included company (`a$co`), customer (`a$cust`), mode (`'INQ'`), and file group (`a$fgrp`).

3. **`BI907AC`** (Commented Out but Referenced):
   - Called to maintain `ARCUPR` records instead of direct processing in `s3` (per revision `JB06`).
   - Parameters included company (`@cpco`), customer (`@cpcus`), ship-to (`@cpshp`), mode (`'INQ'`), and file group (`@cpfgr`).

---

### **Summary**

The `AR890` RPGLE program is an interactive customer inquiry application that retrieves and displays customer data across multiple screens. It:
- Validates company and customer data, checks deletion status, and handles EFT data.
- Supports navigation through roll keys and command keys for specific inquiries.
- Retrieves data from multiple files (`ARCUST`, `ARCUSP`, `ARCUPR`, etc.) and formats it for display.
- Calls external programs (`BI907AC`, potentially `AR915P`, `GB730P`) for specialized inquiries.

**Tables Used**: `ar890d`, `arcust`, `arcusp`, `arcupr`, `gsprod`, `arcont`, `bicont`, `gstabl`, `gscont`.  
**External Programs Called**: `GB730P` (commented out), `AR915P` (commented out), `BI907AC` (commented out but referenced).

If you need further analysis of the screen formats, specific field mappings, or the logic of `BI907AC` or other called programs, please provide additional source code or context.