### List of Use Cases Implemented by the Program

The call stack, consisting of the OCL program `AP780.ocl36.txt` and the RPG/36 programs `AP780P.rpg36.txt`, `AP781.rpg36.txt`, and `AP780.rpg36.txt`, implements a single primary use case:

1. **Generate IRS 1099 File for Accounts Payable**:
   - This use case involves creating an IRS-compliant 1099 file (e.g., `AP1099`) for vendor payments, supporting 1099-MISC, 1099-NEC, 1099-DIV, and 1099-INT forms. It consolidates vendor data, validates inputs, and produces structured records (Transmitter "T", Payer "A", Payee "B", End of Payer "C", and End of Transmission "F") for electronic IRS submission.

### Function Requirement Document



# Function Requirement Document: Generate IRS 1099 File

## Purpose
To generate an IRS-compliant 1099 file for Accounts Payable vendor payments, supporting 1099-MISC, 1099-NEC, 1099-DIV, and 1099-INT forms, using vendor data, company details, and a specified year.

## Inputs
- **Year**: Four-digit year for 1099 reporting (e.g., 2012).
- **File Name**: Input vendor file name (e.g., `APVN2012`).
- **Environment Flag**: Indicates production (`G`) or test environment (e.g., `TEST`).
- **Company Details**:
  - Name (30 chars), Address (30 chars), City (29 chars), State (2 chars), ZIP (9 chars).
  - Tax ID (10 chars: 2-digit part1, 7-digit part2).
  - Contact Name (20 chars), Contact Phone (10 chars).
- **Amount Threshold**: Minimum payment amount for inclusion (8-digit zoned).
- **Current/Last Indicator**: `C` (current year) or `L` (last year).
- **Form Type**: 1099 form type (e.g., `M` for MISC, `N` for NEC, `D` for DIV, `I` for INT).
- **Vendor Data File** (`APVEND`):
  - Fields: Vendor Number (5 chars), Name (30 chars), Address Lines 1–4 (30 chars each), ZIP (5-digit zoned), 1099 Code (`D`, `I`, `M`, `N`), Tax ID (11 chars), Current/Last Year Paid (6-digit packed), Box Numbers (2-digit zoned), Second Box Amount (6-digit packed), Payee Names (40 chars each), Name Overflow (`Y`/`N`), IRS Name Control (4 chars), First/Middle/Last Names, Suffix.

## Outputs
- **1099 File** (`AP1099` or environment-specific, 750 bytes/record):
  - Transmitter "T" Record: Company details, year, tax ID, contact info.
  - Payer "A" Record: Form type, company details, tax ID.
  - Payee "B" Record: Vendor data, tax ID, payment amounts, box numbers.
  - End of Payer "C" Record: Payee count, control totals.
  - End of Transmission "F" Record: Payer count.
- **Index File** (`AP1099I` or environment-specific): Indexed by vendor data keys.
- **Copied File** (`AP10<Year>` or environment-specific): Final 1099 data copy.

## Process Steps
1. **Validate Inputs**:
   - Ensure year is non-zero, form type exists in `GSTABL` table, company details (name, address, tax ID) are non-blank, and indicator is `C` or `L`.
   - Verify input file (`APVN<Year>`) exists; else, terminate with error.
2. **Consolidate Vendor Data**:
   - Read `APVEND` and sorted `AP780S` (vendor keys).
   - Group by vendor number, producing one record per vendor in `AP781`.
   - Extract city/state from address lines (highest non-blank: `VNADD4`, `VNADD3`, `VNADD2`).
   - Accumulate payment amounts based on indicator (`C`: `VNYTDP`, `L`: `VNLYDP`).
   - If second box amount (`VNB2AM`) exists, subtract from total for primary box.
3. **Generate 1099 Records**:
   - Write Transmitter "T" record with company details, tax ID, and sequence number.
   - For each vendor group:
     - Write Payer "A" record with form type (`1`: DIV, `6`: INT, `A`: MISC, `NE`: NEC).
     - Write Payee "B" record with vendor data, tax ID (EIN/SSN based on dash), and amounts.
     - Accumulate control totals by form type and box number.
     - Write End of Payer "C" record with payee count and totals.
   - Write End of Transmission "F" record with payer count.
4. **Build Index and Copy**:
   - Create index (`AP1099I`) for `AP1099` with vendor keys.
   - Copy `AP1099` to `AP10<Year>`.

## Business Rules
1. **Single Vendor Record**:
   - Consolidate multiple company records into one per vendor using `AP780S`.
2. **Amount Threshold**:
   - Include vendors only if total payment (`VNYTDP` or `VNLYDP`) meets `ENTAMT`.
3. **Amount Splitting**:
   - If `VNBOX2` exists, assign `VNB2AM` to second box, subtract from total for primary box.
   - Example: `$15,000` total, `VNBOX1=7` (MISC), `VNBOX2=1` (Rents), `VNB2AM=$5,000` → Primary=`$10,000`, Secondary=`$5,000`.
4. **Default Box**:
   - If `VNBOX1=0` and threshold met, default to box 7 (Nonemployee Compensation).
5. **Form Type Mapping**:
   - `VN1099`: `D`→DIV, `I`→INT, `M`→MISC, `N`→NEC.
   - Box assignments: MISC (1=Rents, 3=Other Income, 6=Medical, 7=Nonemployee), NEC (1=Nonemployee), DIV (1=Dividends, 2=Capital Gains), INT (1=Interest).
6. **Name Handling**:
   - Use `VNPYN1`/`VNPYN2` if provided; else, use `VNNAME`.
   - If `VNNOVF=Y`, use `VNADD1` as second name, shift `VNADD2` to `VNADD1`.
   - Name control: Use `VNNMCT` if available, else `VNNAM4`, or blank if payee names used.
7. **Tax ID**:
   - If tax ID contains `-`, use EIN; else, use SSN.
   - Vendor number as account number.
8. **City/State Extraction**:
   - Extract state (2 chars) and city from highest non-blank address line, assuming state follows ZIP and city precedes a space/comma.
9. **IRS Format**:
   - Pad records with zeros/blanks per IRS specs (e.g., `T`/`A`: 376–748 blank, `B`: 663–750 blank).
   - Include fixed codes (e.g., `28Q01` for `T`, `AMER` for `A`, `I` for in-house software).
   - Increment sequence number (`RECSEQ`) for each record.

## Calculations
- **Payment Amounts**:
  - `L1AMT1 = VNYTDP` (if `CURLST=C`) or `VNLYDP` (if `L`) – `VNB2AM` (if `VNBOX2>0`).
  - `L1AMT2 = VNB2AM` if second box exists.
- **Control Totals**:
  - DIV: `TOTGR1` (Box 1), `TOTGR2` (Box 2).
  - INT: `TOTGR1` (Box 1).
  - MISC: `TOTGR1` (Box 1/2=1), `TOTGR3` (Box 1/2=3), `TOTGR6` (Box 1/2=6).
  - NEC: `TOTGR1` (Box 1/2=1).
- **Counts**:
  - `COUNT`: Payee records per payer.
  - `TOTA`: Payer records.
  - `TOTB`: Written records meetingogels

