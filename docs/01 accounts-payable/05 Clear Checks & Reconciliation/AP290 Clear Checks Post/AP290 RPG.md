The provided document is an RPG III (or RPG/36) program named `AP290`, which is called from the OCL program described previously. This program handles the Accounts Payable (A/P) Cancelled Checks Post process. Below, I’ll explain the process steps, business rules, tables (files) used, and any external programs called, based on the RPG code provided.

### Process Steps of the RPG Program (AP290)

The RPG program `AP290` processes cancelled checks for Accounts Payable reconciliation, updating relevant files and generating a report. Here’s a step-by-step breakdown of the process:

1. **File and Data Definitions**:
   - **Input Files**:
     - `APCRTR` (Primary Input, Line 0009): A transaction file containing check data (80 bytes, indexed by a 16-byte key).
     - `APCONT` (Input, Chained, Line 0010): A control file with company information (256 bytes, indexed by a 2-byte key).
     - `APCHKR` (Update, Chained, Line 0011): A check reconciliation file (128 bytes, indexed by a 16-byte key).
   - **Output File**:
     - `LIST` (Output, Line 0012): A printer file for generating a report (132 characters wide).
   - **Data Structures**:
     - `SEP` (Line 0013): An array of 66 elements (2 bytes each) used for formatting report separators.
     - `APCRTR NS 01` (Lines 0015–0020): Defines fields from the `APCRTR` file, including:
       - `ATCONOL2` (Company Number, positions 2–3).
       - `ATBKGLL1` (Bank G/L Number, positions 4–11).
       - `ATCHK#` (Check Number, positions 12–17).
       - `ATCLAM` (Clear Amount, positions 23–33, 2 decimal places).
       - `ATCLDT` (Clear Date, positions 40–45).
       - `ATCLYY` (Clear Date Year, positions 44–45).
       - `ATKEY` (Key field, positions 2–17, likely a composite key).
     - `APCONT NS` (Line 0026): Defines `ACNAME` (Company Name, positions 4–33).
     - `UDS` (Line 0029): User Data Structure with:
       - `Y2KCEN` (Century, positions 509–510, value 19 for 1900s).
       - `Y2KCMP` (Comparison Year, positions 511–512, value 80 for Y2K logic).
   - **Variables**:
     - `L1CLAM`, `L2CLAM` (112 zoned decimal, initialized to zero): Accumulators for totals at level 1 (Bank G/L) and level 2 (Company).
     - `TIMDAT`, `SYSTIM`, `SYSDAT` (6-digit fields): Used for system time and date.
     - `CLDT`, `CLDT8` (6-digit and 8-digit fields): Used for date manipulation.
     - `CN` (2-digit): Century for date processing.

2. **Initialization** (Lines 0036–0040):
   - Retrieves the system time (`TIME` to `TIMDAT`, 12 digits).
   - Moves `TIMDAT` to `SYSTIM` (time, positions 1–6) and `SYSDAT` (date, positions 7–12).
   - Initializes the `SEP` array with asterisks (`* `) for report formatting.
   - Sets `PAGE` to zero for report pagination.

3. **Level 2 (L2) Processing – Company Level** (Lines 0034–0045):
   - The `L2` indicator represents a control break at the company level (`ATCONOL2`).
   - For each new company:
     - Performs a `CHAIN` operation on `APCONT` using `ATCONO` (Company Number) to retrieve the company name (`ACNAME`). If not found, indicator 92 is set.
     - Initializes `L2CLAM` (Company Total Clear Amount) to zero.
   - The `DO` loop (`L2 DO`) processes all records for a company until the company number changes.

4. **Level 1 (L1) Processing – Bank G/L Level** (Lines 0047–0049):
   - The `L1` indicator represents a control break at the Bank G/L level (`ATBKGLL1`).
   - For each new Bank G/L number:
     - Initializes `L1CLAM` (Bank G/L Total Clear Amount) to zero.
   - This loop processes all checks for a specific Bank G/L account within a company.

5. **Date Processing for Y2K Compliance** (Lines 0051–0058):
   - Converts the clear date (`ATCLDT`) to a 6-digit format by multiplying by 10000.01 (e.g., MMDDYY format).
   - Handles Y2K logic for the year (`ATCLYY`):
     - If `ATCLYY` (2-digit year) is greater than or equal to `Y2KCMP` (80), assumes the century is `Y2KCEN` (19, for 1900s).
     - Otherwise, adds 1 to `Y2KCEN` (e.g., 20 for 2000s).
   - Constructs an 8-digit date (`CLDT8`) by combining the century (`CN`) and `CLDT` (e.g., CCYYMMDD).

6. **Check Reconciliation Update** (Lines 0060–0064):
   - Performs a `CHAIN` on `APCHKR` using `ATKEY` (likely a composite key of company and check number) to locate the check record. If not found, indicator 90 is set.
   - If the record is found (not 90), updates `APCHKR` with:
     - A status of 'R' (position 1, likely indicating "Reconciled").
     - The clear date (`ATCLDT`, positions 2–45).
     - The 8-digit clear date (`CLDT8`, positions 46–91).
   - Adds the clear amount (`ATCLAM`) to `L1CLAM` (Bank G/L total).
   - At the L1 control break, adds `L1CLAM` to `L2CLAM` (Company total).

7. **Report Generation** (Lines 0067–0107):
   - Outputs to the `LIST` printer file:
     - **Detail Lines** (Lines 0098–0101, triggered for each `APCRTR` record):
       - Check Number (`ATCHK#`, zoned, position 10).
       - Clear Date (`ATCLDT`, formatted, position 25).
       - Clear Amount (`ATCLAM`, formatted, position 45).
     - **Headers** (Lines 0072–0097):
       - Company Name (`ACNAME`, if not indicator 92, position 30).
       - Page number (`PAGE`, zoned, position 108).
       - System Date (`SYSDAT`, formatted, position 129).
       - Bank G/L Number (`ATBKGL`, formatted with dashes, position 15).
       - Report title (“A/P CANCELLED CHECKS POST”, position 77–78).
       - System Time (`SYSTIM`, formatted as HH:MM:SS, position 129).
       - Column headings (“CHECK #”, “CLEAR DATE”, “CLEAR AMOUNT”).
       - Separator lines (`SEP` array, position 132).
     - **Totals** (Lines 0102–0107):
       - At L1 break: “BANK G/L # TOTAL” with `L1CLAM` (position 45).
       - At L2 break: “COMPANY TOTAL” with `L2CLAM` (position 45).

### Business Rules

1. **Control Breaks**:
   - The program processes records hierarchically:
     - **L2 (Company Level)**: Groups records by company number (`ATCONOL2`).
     - **L1 (Bank G/L Level)**: Within each company, groups records by Bank G/L number (`ATBKGLL1`).
   - Totals (`L1CLAM`, `L2CLAM`) are accumulated and reported at each level.

2. **Y2K Date Handling**:
   - The program adjusts the century for the clear date based on a comparison year (`Y2KCMP` = 80):
     - Years ≥ 80 are assumed to be 19xx (e.g., 1980–1999).
     - Years < 80 are assumed to be 20xx (e.g., 2000–2079).
   - This ensures correct date representation in the `CLDT8` field (CCYYMMDD).

3. **File Updates**:
   - The `APCHKR` file is updated for each valid check record (not indicator 90) with a reconciled status ('R'), the clear date, and the 8-digit date.
   - The program assumes that `APCRTR` provides valid transaction data, and `APCONT` provides company details.

4. **Reporting**:
   - Generates a detailed report with headers, check details, and totals for each Bank G/L and company.
   - Skips company name printing if the `APCONT` record is not found (indicator 92).
   - Formats dates, amounts, and other fields for readability.

5. **Error Handling**:
   - Uses indicators (90 for `APCHKR` chain failure, 92 for `APCONT` chain failure) to handle missing records gracefully.
   - Does not update `APCHKR` if the record is not found (indicator 90).

### Tables (Files) Used

1. **APCRTR** (Line 0009):
   - Primary input file (transaction file).
   - Fields: Company Number (`ATCONOL2`), Bank G/L Number (`ATBKGLL1`), Check Number (`ATCHK#`), Clear Amount (`ATCLAM`), Clear Date (`ATCLDT`, `ATCLYY`), Key (`ATKEY`).

2. **APCONT** (Line 0010):
   - Input file (company control file).
   - Field: Company Name (`ACNAME`).

3. **APCHKR** (Line 0011):
   - Update file (check reconciliation file).
   - Updated with reconciled status ('R'), clear date (`ATCLDT`), and 8-digit date (`CLDT8`).

4. **LIST** (Line 0012):
   - Output printer file for the report.

**Note**: The OCL program referenced an additional file, `APCHKUP`, which is cleared (`CLRPFM`) but not used in the RPG code. It may be used by another program or implicitly by `AP290` (e.g., as a work file).

### External Programs Called

- **None**: The RPG program `AP290` does not explicitly call any external programs. It performs all processing internally using the defined files and generates a report.

### Summary

- **Process Overview**: The `AP290` RPG program processes cancelled checks by reading transaction data from `APCRTR`, retrieving company details from `APCONT`, updating the `APCHKR` file with reconciled check information, and generating a report (`LIST`) with check details and totals at Bank G/L and company levels. It includes Y2K-compliant date handling.
- **Business Rules**: Hierarchical processing (company and Bank G/L levels), Y2K date conversion, file updates for reconciled checks, and detailed reporting with error handling for missing records.
- **Files Used**: `APCRTR` (input), `APCONT` (input), `APCHKR` (update), `LIST` (output).
- **External Programs**: None.

If you have additional OCL or RPG snippets, or need further details (e.g., specific field formats or interactions with `APCHKUP`), let me know! I can also search for related information if required.