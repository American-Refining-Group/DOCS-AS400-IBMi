Based on the provided OCL (`AP290.ocl36.txt`) and RPG (`AP290.rpg36.txt`) program details, I’ll identify the use case implemented by the program and create a concise function requirement document for it, assuming the use case is implemented as a large function that processes inputs programmatically rather than through screen interactions.

### Use Case Implemented

The program implements a single primary use case:

1. **Post Cancelled Checks for Accounts Payable Reconciliation**:
   - This use case processes a set of cancelled check transactions to update the check reconciliation file, accumulate totals by bank general ledger (G/L) account and company, and generate a report summarizing the reconciled checks. It includes Y2K-compliant date handling to ensure accurate date representation.

### Function Requirement Document



# Function Requirements: Post Cancelled Checks for A/P Reconciliation

## Purpose
Automate the posting of cancelled checks to update the Accounts Payable (A/P) reconciliation file, calculate totals by bank G/L account and company, and generate a summary report.

## Inputs
- **Transaction Data** (`APCRTR` equivalent):
  - Company Number (2 chars)
  - Bank G/L Number (8 chars)
  - Check Number (6 chars)
  - Clear Amount (11 digits, 2 decimal places)
  - Clear Date (6 digits, MMDDYY format)
- **Company Data** (`APCONT` equivalent):
  - Company Number (2 chars, key)
  - Company Name (30 chars)
- **Check Reconciliation Data** (`APCHKR` equivalent, for update):
  - Key (16 chars, composite of company and check number)
- **System Parameters**:
  - System Date (6 digits, MMDDYY)
  - System Time (6 digits, HHMMSS)
  - Y2K Century (2 digits, e.g., 19 for 1900s)
  - Y2K Comparison Year (2 digits, e.g., 80)

## Outputs
- **Updated Check Reconciliation File** (`APCHKR` equivalent):
  - Updated records with reconciled status ('R'), clear date (6 digits), and 8-digit date (CCYYMMDD).
- **Report** (`LIST` equivalent):
  - Header: Company name, page number, system date/time, bank G/L number, report title.
  - Detail Lines: Check number, clear date, clear amount.
  - Totals: Bank G/L total and company total clear amounts.
- **Cleared Work File** (`APCHKUP` equivalent): Emptied file.
- **Deleted Work File** (`APCRTR` equivalent): Deleted after processing.

## Process Steps
1. **Initialize**:
   - Retrieve system date and time.
   - Initialize report page counter to 0 and separator lines with asterisks.
   - Initialize accumulators for bank G/L (`L1CLAM`) and company (`L2CLAM`) totals to 0.

2. **Process by Company** (group by Company Number):
   - Retrieve company name from company data using Company Number.
   - If company not found, skip name in report but continue processing.

3. **Process by Bank G/L** (within company):
   - Group transactions by Bank G/L Number.
   - Initialize bank G/L total (`L1CLAM`) to 0 for each new G/L.

4. **Process Each Transaction**:
   - Convert clear date to 8-digit format (CCYYMMDD):
     - If 2-digit year ≥ 80, use century 19 (e.g., 1980–1999).
     - Else, use century 20 (e.g., 2000–2079).
   - Update check reconciliation file with:
     - Status 'R' (reconciled).
     - Clear date (MMDDYY).
     - 8-digit date (CCYYMMDD).
   - Add clear amount to bank G/L total (`L1CLAM`).

5. **Accumulate Totals**:
   - At bank G/L break, add `L1CLAM` to company total (`L2CLAM`).

6. **Generate Report**:
   - Print header (company name, page, date/time, bank G/L, title).
   - Print column headings (Check #, Clear Date, Clear Amount).
   - For each transaction: Print check number, clear date, clear amount.
   - At bank G/L break: Print bank G/L total.
   - At company break: Print company total.

7. **Cleanup**:
   - Delete transaction work file.
   - Clear output work file.

## Business Rules
- **Hierarchical Processing**: Group checks by company, then by bank G/L account.
- **Y2K Date Handling**: Convert 2-digit year to 4-digit year based on comparison year (80):
  - Year ≥ 80 → 19xx (e.g., 80 → 1980).
  - Year < 80 → 20xx (e.g., 79 → 2079).
- **File Updates**: Update check reconciliation file only if record exists; otherwise, skip update but continue processing.
- **Reporting**: Include company name only if found; format bank G/L number with dashes (e.g., XXXXXX-XX); report totals at bank G/L and company levels.
- **Error Handling**: Handle missing company or check records gracefully (skip updates or report fields).
- **Cleanup**: Ensure temporary transaction file is deleted and output file is cleared post-processing.

## Calculations
- **Date Conversion**:
  - `CLDT = ATCLDT * 10000.01` (converts MMDDYY to internal format).
  - `CLDT8 = CN (century) + CLDT` (e.g., 19 + MMDDYY → CCYYMMDD).
- **Totals**:
  - `L1CLAM += ATCLAM` (accumulate clear amount for bank G/L).
  - `L2CLAM += L1CLAM` (accumulate bank G/L total for company at L1 break).

## Assumptions
- Input transaction data is valid and sorted by company and bank G/L number.
- Check reconciliation file allows updates with shared access.
- Report formatting uses fixed positions for fields and totals.

