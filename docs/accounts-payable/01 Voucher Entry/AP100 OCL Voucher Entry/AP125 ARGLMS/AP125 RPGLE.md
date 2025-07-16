The `AP125.rpg` program is an RPG III program running on an IBM AS/400 (iSeries) system, designed to create voucher transactions from freight invoice data, likely called from the `AP125.ocl36.txt` OCL script after the `AP125P` program has selected invoices for batch processing. It processes freight invoice headers (`FRCINH` or `FRCFBH`) to generate accounts payable transactions in the `APTRAN` file, including header and detail records. Below, I will detail the process steps, business rules, tables used, and external programs called, based on the provided source code.

---

### **Process Steps of the AP125 Program**

The program is structured to process freight invoices, validate data, calculate due dates, and create or update accounts payable transactions. The main process steps are:

1. **Initialization**:
   - **Parameters**: Receives input parameters:
     - `CO`: Company code (2 digits).
     - `CAID`: Carrier ID (6 characters).
     - `CAIN`: Invoice number (25 characters).
     - `RCD`: Record type (`FRCFBHP4` for freight billed balance header, otherwise `FRCINHP4`, per revision `JB06`).
     - `RDNO`: Reference number (numeric, defined like `FRRDNO`).
   - Initializes fields and indicators, setting indicators 81-83, 50-53, 60-69, and 51-62 to off.
   - Sets up key fields for file access (e.g., `TERMKY` for `APTERM`, `BBCAKY` for `BBCAID`).

2. **Subroutine S1 (Main Processing)**:
   - **Validate Company**: Chains to `APCONT` using `CO` to retrieve company details (e.g., `ACAPGL`, `ACCAGL`, `ACRTGL`, `ACNXTE`). If not found, sets indicator 50.
   - **Retrieve Invoice Data**:
     - Constructs a key (`FRCKEY` or `FRCK39`) using `CO`, `CAID`, `CAIN`, and `RDNO` (for `FRCFBH`).
     - Chains to `FRCINH` (if `RCD ≠ 'FRCFBHP4'`) or `FRCFBH` (if `RCD = 'FRCFBHP4'`, per `JB06`) to retrieve invoice details.
     - If found, populates transaction fields:
       - `ATINV#`: Invoice number (`FRCAIN`).
       - `INTY`: Invoice type (`FRINTY`).
       - `ATINDT`/`INDT`: Invoice date (`FRIYMD` converted to MMDDYY).
       - `ATIAMT`/`IAMT`/`ATFRTL`/`FRTL`: Invoice amount (`FRINAM - FRFBOA`, per `JB06`).
       - `ATSORN`/`SORN`: Sales order number (`FRRDNO`).
       - `ATSSRN`/`SSRN`: Sales sequence number (`FRSRN`).
       - Sets indicators 51 (`INTY = 'P'`) or 52 (`INTY = 'O'`) for invoice description.
   - **Generate Entry Number**:
     - If `ENT#` is zero, sets `RECSTS = 'ADDNEW'`, retrieves the next entry number (`ACNXTE`) from `APCONT`, increments it, and updates `APCONT`.
     - Ensures `ENT#` does not exceed 99999, adjusting `ACNXTE` accordingly.
   - **Retrieve Vendor Information**:
     - Chains to `APVENY` using `CO` and `CAID` to get the vendor number (`VYVEND`).
     - Chains to `APVEND` using `CO` and `VEND` to retrieve vendor details (`VNVNAM`, `VNAD1`, `VNAD2`, `VNAD3`, `VNAD4`, `VNHOLD`, `VNSNGL`, `VNTERM`, `VNEXGL`).
     - Sets hold description (`HLDD`) based on `VNHOLD` (per `JB02` and `MG18`):
       - `H`: "VENDOR ON HOLD" (COM,01).
       - `A`: "ON HOLD FOR ACH" (COM,02).
       - `W`: "ON HOLD FOR WIRE TRANSFER" (COM,03).
       - `U`: "ON HOLD FOR UTILITY AUTO-PAYMENT" (COM,04).
   - **Output**: Writes or updates the transaction header in `APTRAN` via the `EXCPT` operation.

3. **Subroutine S2 (Header Processing and Detail Setup)**:
   - Chains to `APTRAN` using `KEYENT` (constructed from `COENT` and "000") to check for an existing header.
   - If not found, populates header fields from `APCONT` (`APGL`, `BKGL`, `RTGL`) and vendor data.
   - Calls `S2EDIT` to validate header fields.
   - Calls `HDRADD` to write/update the transaction header.
   - Calls `ROLFWD` to initiate detail line processing.
   - Sets indicator 81 to trigger further processing.

4. **Subroutine S2EDIT (Header Validation)**:
   - **Validate Invoice Date**:
     - Moves `INDT` to `MMDDYY` and calls `DTEDIT` to validate the date.
   - **Calculate Due Date**:
     - If `DUDT` is zero and `VEND` is non-zero, calls `CLCDUE` to calculate the due date based on vendor terms (`VNTERM`).
     - Otherwise, sets `DUDT = INDT`.
   - **Validate Due Date**:
     - Converts `DUDT` to `MMDDYY` and validates via `DTEDIT`.
     - Adjusts for century (`Y2KCEN`) to create `INDT8` and `DUDT8` (8-digit dates).
     - Checks `APDATE` (per `MG17`) to replace `DUDT8` with a non-holiday/non-weekend date (`ADNED8`).
   - **Validate G/L Accounts**:
     - Chains to `GLMAST` to validate `APGL`, `BKGL`, and `RTGL`, retrieving descriptions (`APGLNM`, `BKGLNM`, `RTGLNM`). Sets indicators 61, 62, or 63 if invalid.

5. **Subroutine CLCDUE (Calculate Due Date)**:
   - If `VNTERM` is non-zero, chains to `GSTABL` (table `APTERM`) to retrieve terms (`TBNETD`, `TBPRXD`, `TBDISC`, `TBDISD`).
   - If valid, retrieves discount percentage (`TBDISC` to `SVDSPC`, per `MG03`).
   - Calls `TMDATN` (net days) or `TMDATP` (prox days) to calculate `DUDT`.

6. **Subroutine TMDATN (Net Days Calculation)**:
   - Converts `INDT` to Julian format (`G$JD`) via `@DTE1`.
   - Adds net days (`TBNETD`) to `G$JD`.
   - Converts back to Gregorian format (`$MDY`) via `@DTE2` to set `DUDT`.

7. **Subroutine TMDATP (Prox Days Calculation)**:
   - Adjusts `INDT` by incrementing the month (and year if December) and sets the day to `TBPRXD` to calculate `DUDT`.

8. **Subroutine DTEDIT (Date Validation)**:
   - Validates `MMDDYY` by breaking it into month, day, and year.
   - Checks:
     - Month (1-12).
     - Day (1-31, or 1-28/29 for February, accounting for leap years).
   - Sets indicator 99 if invalid.

9. **Subroutine @DTE1 (Gregorian to Julian Conversion)**:
   - Converts a Gregorian date (`$MDY`, `$CN`) to Julian days (`G$JD`) for dates between March 1, 1900, and February 28, 2100.
   - Calculates the day of the week (`G$JW`).

10. **Subroutine @DTE2 (Julian to Gregorian Conversion)**:
    - Converts a Julian date (`G$JD`) back to Gregorian format (`$MDY`, `$CN`).

11. **Subroutine ROLFWD (Detail Line Processing)**:
    - If `SORN` and `SSRN` are non-zero and `RECSTS = 'ADDNEW'`, populates a `SALES` data structure with `CO`, `SORN`, `SSRN`, `ENT#`, `VEND`, `FRTL`, `SVDSPC`, and `CMPDT8` (invoice date minus one year, per `JB05`).
    - Calls `AP1012` to populate freight detail lines with calculated percentage amounts.
    - Resets `RECSTS` and sets `KEYENT` to "001" for the first detail line.

12. **Subroutine S3 (Detail Line Processing)**:
    - If `FRTL` and `PRAM` are non-zero, calculates the freight amount (`FRAM = PRAM * FRTL`) and adds it to `HOLDAM` and `SVTTL`.
    - Adjusts `FRAM` if `IAMT > SVTTL` to balance the invoice amount.
    - Combines `PRAM` and `FRAM` to set `AMT`.
    - If `AMT` is zero, clears the header (`HDRCLR`) and exits.
    - Calls `S3EDIT` to validate detail fields.
    - Calls `DETADD` to write the detail line.
    - Calls `DETCLR` to clear detail fields.
    - Calls `ROLFWD` to process additional lines.
    - Updates `FRCINH` or `FRCFBH` (setting `FRAPST = 'Y'`) based on `RCD`.
    - Writes the updated record via `EXCPT` (`APINST` for `FRCINH`, `APINSF` for `FRCFBH`).

13. **Subroutine S3EDIT (Detail Validation)**:
    - Sets default expense G/L (`EXGL`) from `VNEXGL` if not specified.
    - Chains to `GLMAST` to validate `EXGL`, setting indicator 50 if invalid.

14. **Subroutine HDRADD (Add/Update Header)**:
    - Chains to `APTRAN` and writes/updates the header record using `EXCPT`.

15. **Subroutine DETADD (Add Detail Line)**:
    - Writes the detail line to `APTRAN` using `EXCPT`.

16. **Subroutine HDRCLR (Clear Header Fields)**:
    - Clears header fields (e.g., `SVAPGL`, `SVBKGL`, `HDEL`, `ENT#`, `VNAM`, etc.).

17. **Subroutine DETCLR (Clear Detail Fields)**:
    - Clears detail fields (e.g., `SVLNGL`, `SVLNCO`, `DDEL`, `AMT`, `DISC`, etc.).

18. **Subroutine ROLLBK (Rollback Processing)**:
    - Rolls back detail lines by decrementing `NXLINE` and chaining to `APTRAN`.
    - If a header is reached, calls `S2EDIT` to validate and sets indicator 82.

19. **Program Termination**:
    - Sets `*INLR = *ON` to end the program.

---

### **Business Rules**

1. **Invoice Processing**:
   - Processes invoices from `FRCINH` or `FRCFBH` based on `RCD` (`FRCFBHP4` for freight billed balance, per `JB06`).
   - Adjusts invoice amount (`ATIAMT = FRINAM - FRFBOA`, per `JB06`) to account for freight balancing order override total.

2. **Hold Status**:
   - Supports hold codes (`VNHOLD`): `H` (hold), `A` (ACH), `W` (wire transfer), `U` (utility auto-payment, per `MG18`).
   - Assigns corresponding descriptions to `HLDD` (per `JB02`, `MG18`).

3. **Due Date Calculation**:
   - Calculates due date (`DUDT`) based on vendor terms (`VNTERM`) using net days (`TBNETD`) or prox days (`TBPRXD`) from `GSTABL`.
   - Adjusts due date to avoid holidays/weekends using `APDATE` (`ADNED8`, per `MG17`).
   - Defaults to invoice date (`INDT`) if no terms are specified.

4. **Discount Handling**:
   - Retrieves discount percentage (`TBDISC`) from `GSTABL` and applies it to `SVDSPC` (per `MG03`).

5. **Date Validation**:
   - Validates invoice and due dates for valid months, days, and leap years.
   - Handles century adjustments for Y2K compliance.

6. **G/L Account Validation**:
   - Validates accounts payable (`APGL`), bank (`BKGL`), and retention (`RTGL`) G/L accounts against `GLMAST`.

7. **Detail Line Calculation**:
   - Calculates freight amount (`FRAM`) as a proportion of `PRAM` and `FRTL`.
   - Ensures total line amount (`AMT = PRAM + FRAM`) aligns with the invoice amount (`IAMT`).

8. **Freight Detail Lines**:
   - Calls `AP1012` to populate detail lines with calculated freight percentages, using sales order (`SORN`), sequence number (`SSRN`), and invoice date minus one year (`CMPDT8`, per `JB05`).

9. **Transaction Creation**:
   - Creates header and detail records in `APTRAN` with vendor, G/L, and freight details.
   - Updates `FRCINH` or `FRCFBH` to mark invoices as processed (`FRAPST = 'Y'`).

---

### **Tables (Files) Used**

1. **Update Files**:
   - `APTRAN`: Accounts Payable transaction file (header and detail records, 404 bytes, keyed by company and entry number).
   - `APCONT`: Accounts Payable control file (256 bytes, keyed by company).
   - `FRCINH`: Freight Invoice Header (206 bytes, keyed by company, carrier ID, invoice number).
   - `FRCFBH`: Freight Billed Balance Header (206 bytes, keyed by company, carrier ID, invoice number, reference number, per `JB06`).

2. **Input Files**:
   - `APVEND`: Vendor master file (579 bytes, keyed by company and vendor number).
   - `APVENY`: Vendor cross-reference file (320 bytes, keyed by company and carrier ID).
   - `GLMAST`: General Ledger master file (256 bytes, keyed by company and G/L account).
   - `GSTABL`: General table file (256 bytes, keyed by table type `APTERM` for terms).
   - `APDATE`: Due date adjustment file (19 bytes, keyed by company and due date, per `MG17`).

---

### **External Programs Called**

1. **AP1012**:
   - Called in `ROLFWD` to populate freight detail lines with calculated percentage amounts, passing the `SALES` data structure.

---

### **Summary**

The `AP125.rpg` program processes freight invoices from `FRCINH` or `FRCFBH` to create accounts payable transactions in `APTRAN`. It validates company, vendor, and G/L data, calculates due dates (adjusted for holidays/weekends), applies discounts, and generates header and detail records. The program handles freight balancing adjustments, hold statuses, and detail line calculations via `AP1012`. It ensures data integrity through extensive validation and updates the source invoice records upon completion.

If you need further details or clarification on specific subroutines, business rules, or file structures, please let me know!