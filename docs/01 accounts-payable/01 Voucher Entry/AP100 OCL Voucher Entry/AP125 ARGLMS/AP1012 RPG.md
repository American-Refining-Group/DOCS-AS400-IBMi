The `AP1012.rpg` program is an RPG III program running on an IBM AS/400 (iSeries) system, designed to create detail lines for accounts payable (A/P) voucher entries in the `APTRAN` file by prorating freight charges based on sales detail or miscellaneous records. It is called from the `AP125.rpg` program (which is invoked by the `AP125.ocl36.txt` OCL script) and processes data from sales files (`SA5FIUD`, `SA5FIUM`, `SA5MOUD`, `SA5MOUM`) to allocate freight amounts. Below, I detail the process steps, business rules, tables used, and external programs called, based on the provided source code.

---

### **Process Steps of the AP1012 Program**

The program calculates freight amounts for A/P voucher detail lines by prorating the total freight amount (`FRTTOT`) across sales detail or miscellaneous records, based on gallons or miscellaneous amounts. It handles both detail (`SA5FIUD`, `SA5MOUD`) and miscellaneous (`SA5FIUM`, `SA5MOUM`) records, with adjustments for invoice dates and product codes.

1. **Initialization**:
   - **Parameters**: Receives a `SALES` data structure with:
     - `SACO`: Company number (2 digits).
     - `SAORD`: Order number (6 digits).
     - `SASRN#`: Shipping reference number (3 digits).
     - `SASEQ`: Sequence number (3 digits).
     - `FRTTOT`: Total freight to allocate (7.2 digits).
     - `VEND`: Vendor number (5 digits).
     - `ENTNUM`: A/P entry number (5 digits).
     - `EXGL`: Expense G/L (8 digits).
     - `DSPC`: Discount percentage (4.3 digits, per `MG03`).
     - `CMPDT8`: Comparison date (invoice date minus one year, per `JB06`).
     - `S@FIMO`: File indicator (`F` for `SA5FI`, `M` for `SA5MO`, per `JB06`).
     - `S@DM`: Detail/misc indicator (`D` for detail, `M` for misc, per `JB06`).
   - Initializes fields (`ZERO6`, `ZERO7`, `ZERO9`, `ZERO11`) to zero.
   - If `S@FIMO` is blank, calls `GETS@` to determine the appropriate file (`SA5FI` or `SA5MO`) and record type (`D` or `M`).

2. **Subroutine GETS@ (Determine File and Record Type)**:
   - If `SAORD` and `SASRN#` are non-zero, constructs a key (`SAIUKY`) using `SACO`, `SAORD`, and `SASRN#`.
   - Checks files in order:
     - `SA5FIUD`: If a valid record is found with `S5SHD8 >= CMPDT8`, sets `S@FIMO = 'F'`, `S@DM = 'D'`.
     - `SA5MOUD`: If found, sets `S@FIMO = 'M'`, `S@DM = 'D'`.
     - `SA5FIUM`: If found, sets `S@FIMO = 'F'`, `S@DM = 'M'`.
     - `SA5MOUM`: If found, sets `S@FIMO = 'M'`, `S@DM = 'M'`.
   - Stops when a valid record is found (`SA5FND`).

3. **Calculate Total Gallons (Detail Records)**:
   - Constructs a key (`SAKEY`) using `SACO`, `SAORD`, and `SASRN#`.
   - Sets the lower limit for `SA5FIUD` or `SA5MOUD` based on `S@FIMO`.
   - Reads through `SA5FIUD` (if `S@FIMO ≠ 'M'`) or `SA5MOUD` (if `S@FIMO = 'M'`).
   - For matching records (`S5CO# = SACO`, `S5ORD# = SAORD`, `S5SRN# = SASRN#`, `S5SHD8 >= CMPDT8`):
     - Adds `S5NGAL` (net gallons) to `TTLQTY`.
     - Increments `COUNT1` (number of detail lines).
   - If `TTLQTY` and `COUNT1` are zero, skips to miscellaneous record processing (`SKIP3`).

4. **Prorate Freight for Detail Records**:
   - Resets `SAKEY` to "000" and re-reads `SA5FIUD` or `SA5MOUD`.
   - For each matching record:
     - Calculates the percentage (`PCTHLD = S5NGAL / TTLQTY`) if `TTLQTY` is non-zero.
     - Computes freight amount (`AMT,Y = PCTHLD * FRTTOT`).
     - Stores in `AMTITM` and `LINAMT`.
     - Increments `COUNT2` (line counter) and array index `Y`.
   - For the last record (`COUNT2 = COUNT1`):
     - Calculates total computed amount (`CLCAMT` = sum of `AMT`).
     - Adjusts `AMTITM` and `LINAMT`:
       - If `FRTTOT = CLCAMT`, no adjustment.
       - If `FRTTOT > CLCAMT`, adds difference (`DIFF1`) to `AMTITM`.
       - If `FRTTOT < CLCAMT`, subtracts difference (`DIFF2`) from `AMTITM`.
   - Calls `GOOD` to write the detail line to `APTRAN`.

5. **Subroutine MFRTO (Miscellaneous Freight Total)**:
   - For invoices with only miscellaneous records (no detail lines, per `JB08`):
     - Reads `SA5FIUM` or `SA5MOUM` based on `S@FIMO`.
     - For matching records (`SMCO# = SACO`, `SMORD# = SAORD`, `SMSRN# = SASRN#`, `SMMSTY = 'F'`, `SMGLNO ≠ 0`, `SMSHD8 >= CMPDT8`):
       - Adds `SMMAMT` to `TTLMFT` (total miscellaneous freight).
       - Increments `COUNTM` (miscellaneous line count).

6. **Prorate Freight for Miscellaneous Records**:
   - If no detail records were found (`TTLQTY = 0`, `COUNT1 = 0`):
     - Reads `SA5FIUM` or `SA5MOUM` based on `S@FIMO`.
     - For matching records (`SMCO# = SACO`, `SMORD# = SAORD`, `SMSRN# = SASRN#`, `SMMSTY = 'F'`, `SMGLNO ≠ 0`, `SMSHD8 >= CMPDT8`):
       - Calculates miscellaneous amount (`CLCAMT = SMMAMT * SMMQTY`).
       - For non-last records (`COUNT3 < COUNTM`):
         - Computes percentage (`PCTHLD = CLCAMT / TTLMFT`).
         - Calculates freight amount (`FRTAMT = FRTTOT * PCTHLD`).
         - Adds `FRTAMT` to `CLCTOT` (running total).
       - For the last record:
         - Sets `FRTAMT = FRTTOT - CLCTOT` to ensure total matches `FRTTOT`.
       - Calls `GETFRT` to write the miscellaneous detail line to `APTRAN`.

7. **Subroutine GOOD (Write Detail Line for Detail Records)**:
   - Retrieves freight G/L (`FEGL`):
     - If product code (`S5PROD`) contains an alpha character (per `JB02`), chains to `GSCTUM` using `S5CO#`, `S5PROD`, `S5CNTR`, and `S5UM` to get `CUFEGL`.
     - Otherwise, chains to `GSTABL` (table `CNTRPF`) using `S5TANK` to get `TBFEG4`, appending `S5PROD`.
     - If still zero, chains to `BICONT` to get default `BCFRGL`.
   - Constructs `APKEY` using `S5CO#` and `COUNT2`.
   - Chains to `APTRAN` to check for an existing record.
   - Sets description (`DDES = 'XXXXXXXXX XXXX XXX FRTCHG'`).
   - Writes (`ADDT`) or updates (`UPDT`) the detail line with `LINAMT`, `FEGL`, `DSPC`, etc.

8. **Subroutine GETFRT (Write Detail Line for Miscellaneous Records)**:
   - Sets freight G/L (`FEGL = SMGLNO`).
   - Constructs `APKEY` using `SMCO#` and `COUNT3`.
   - Chains to `APTRAN` to check for an existing record.
   - Sets description (`DDES = 'MISC CHARGE'`).
   - Writes (`ADDTM`) or updates (`UPDTM`) the detail line with `FRTAMT`, `FEGL`, `DSPC`, etc.

9. **Program Termination**:
   - Sets `*INLR = *ON` to end the program.

---

### **Business Rules**

1. **Freight Proration**:
   - For detail records (`SA5FIUD` or `SA5MOUD`), prorates freight (`FRTTOT`) based on net gallons (`S5NGAL / TTLQTY`).
   - For miscellaneous records (`SA5FIUM` or `SA5MOUM`), prorates freight based on miscellaneous amount (`SMMAMT * SMMQTY / TTLMFT`, per `JB08`).
   - Ensures the sum of prorated amounts equals `FRTTOT` by adjusting the last record.

2. **File Selection**:
   - Uses `S@FIMO` (`F` for `SA5FI`, `M` for `SA5MO`) and `S@DM` (`D` for detail, `M` for misc) to determine the correct file (`SA5FIUD`, `SA5MOUD`, `SA5FIUM`, `SA5MOUM`).
   - If `S@FIMO` is blank, `GETS@` determines the file by checking for valid records (per `JB07`).

3. **Date Restriction**:
   - Only processes records with a ship date (`S5SHD8` or `SMSHD8`) within one year of the invoice date (`CMPDT8`, per `JB06`).

4. **G/L Account Determination**:
   - For detail records with alpha product codes, retrieves freight G/L (`CUFEGL`) from `GSCTUM` (per `JB02`).
   - For numeric product codes, uses `TBFEG4` from `GSTABL` (table `CNTRPF`) with `S5PROD` appended.
   - Defaults to `BCFRGL` from `BICONT` if no G/L is found.

5. **Miscellaneous Records**:
   - Only processes miscellaneous records with `SMMSTY = 'F'` (freight) and non-zero `SMGLNO` (per `MG05`, `JB08`).
   - Handles invoices with only miscellaneous lines (no detail records, per `JB08`).

6. **Discount Application**:
   - Applies discount percentage (`DSPC`) from the `SALES` data structure to detail lines (per `MG03`).

7. **Detail Line Creation**:
   - Creates `APTRAN` detail lines with fields like `FEGL`, `LINAMT` (or `FRTAMT`), `DSPC`, and default values (e.g., `CLCD = 'C'`, `POSQ = '000'`).
   - Uses fixed descriptions: `XXXXXXXXX XXXX XXX FRTCHG` for detail records, `MISC CHARGE` for miscellaneous records.

---

### **Tables (Files) Used**

1. **Update Files**:
   - `APTRAN`: Accounts Payable transaction file (404 bytes, keyed by company and entry number).

2. **Input Files**:
   - `SA5FIUD`: Sales detail file (1024 bytes, keyed by company, customer, order, ship to, SRN).
   - `SA5FIUM`: Sales miscellaneous file (1024 bytes, keyed by company, customer, order, SRN).
   - `SA5MOUD`: Move detail file (1024 bytes, keyed by company, customer, order, ship to, SRN, per `JB06`).
   - `SA5MOUM`: Move miscellaneous file (1024 bytes, keyed by company, customer, order, SRN, per `JB06`).
   - `GSTABL`: General table file (256 bytes, keyed by table type `CNTRPF` for freight G/L).
   - `BICONT`: Billing control file (256 bytes, keyed by company for default freight G/L).
   - `GSCTUM`: Customer table file (64 bytes, keyed by company, product code, container code, unit of measure, per `JB02`).

---

### **External Programs Called**

- None. The program does not call any external programs.

---

### **Summary**

The `AP1012.rpg` program creates A/P voucher detail lines in `APTRAN` by prorating freight charges (`FRTTOT`) across sales detail (`SA5FIUD`, `SA5MOUD`) or miscellaneous (`SA5FIUM`, `SA5MOUM`) records. It calculates freight amounts based on gallons for detail records or miscellaneous amounts for misc records, ensuring the total matches `FRTTOT`. It retrieves freight G/L accounts from `GSCTUM`, `GSTABL`, or `BICONT`, applies discounts, and restricts records to within one year of the invoice date. The program handles invoices with only miscellaneous lines and writes detail lines with appropriate descriptions.

If you need further details or clarification on specific subroutines, business rules, or file structures, please let me know!