### List of Use Cases Implemented by the Program

The call stack consists of three components: the OCL program (`AR500P.ocl36.txt`), the RPGLE program (`AR500P.rpgle.txt`), and the RPG/36 program (`AR500.rpg36.txt`). Together, they implement a single primary use case:

1. **Generate Accounts Receivable (A/R) Aged Trial Balance Report**:
   - **Description**: This use case involves prompting the user for parameters, validating them, and generating a detailed A/R Aged Trial Balance report that categorizes customer balances into aging buckets (0-30, 31-60, 61-90, 91-120, over 120 days from invoice date) by customer number, with options for filtering by company, customer, salesman, and other criteria.
   - **Components**:
     - **OCL Program (`AR500P.ocl36.txt`)**: Orchestrates the process by calling initialization programs (`GSGENIEC`, `GSY2K`), loading the RPGLE program, and submitting or running the RPG/36 program.
     - **RPGLE Program (`AR500P.rpgle.txt`)**: Handles user input via a workstation interface, validates parameters, and updates the A/R control file with the aging date.
     - **RPG/36 Program (`AR500.rpg36.txt`)**: Processes customer and invoice data, calculates aging buckets, and generates the report output.

### Function Requirement Document: A/R Aged Trial Balance Report Generation



# A/R Aged Trial Balance Report Generation Function Requirements

## Purpose
The function generates an Accounts Receivable (A/R) Aged Trial Balance report, categorizing customer balances into aging buckets based on invoice dates, with flexible filtering and reporting options.

## Inputs
- **Aging Date** (`kydate`, 6-digit, MMDDYY): Date for aging calculations.
- **Company Selection** (`kyalco`, string): `'ALL'` or `'CO '` (specific companies).
- **Company Numbers** (`kyco1`, `kyco2`, `kyco3`, 2-digit each): Up to three company numbers if `kyalco = 'CO '`.
- **Customer Selection** (`kyalcs`, string): `'ALL'` or `'SEL'` (specific customers).
- **Customer Numbers** (`kycs01`, `kycs02`, `kycs03`, 6-digit each): Up to three customer numbers if `kyalcs = 'SEL'`.
- **Salesmen Selection** (`kyalsl`, string): `'ALL'` or `'SEL'` (specific salesmen).
- **Salesmen Range** (`kyfmsl`, `kytosl`, 2-digit each): From/to salesman numbers if `kyalsl = 'SEL'`.
- **Report Type** (`kyds`, string): `'D'` (detail) or `'S'` (summary).
- **Outstanding Invoices** (`kyouts`, string): `'O'` (outstanding only) or blank.
- **Report Sequence** (`kyrept`, string): `'C'` (customer), `'N'` (name), or `'S'` (salesman).
- **Credit Limit Flag** (`kyclyn`, string): `'Y'` (print) or `'N'` (don’t print).
- **NOD Report Flag** (`kynod`, string): `'Y'` (NOD only) or `'N'` (all records).
- **Salesman Copies** (`kyslcy`, string): `'Y'` (print) or `'N'` (don’t print).
- **Customer Class** (`kycucl`, string): Customer class code or blank.
- **Copy Count** (`kycopy`, 2-digit): Number of report copies (default: 1).
- **Job Queue Flag** (`kyjobq`, string): `'Y'` (batch) or `'N'` (interactive).

## Outputs
- **A/R Aged Trial Balance Report**:
  - Format: Printed report (to `PRINT`, `PRINT2`, or `PRINT3`).
  - Content: Customer details, aging buckets (0-30, 31-60, 61-90, 91-120, over 120 days), totals, and optional credit limit/comments.
  - Structure: Hierarchical by company, customer group, and customer.
- **Updated `ARCONT` File**: Records updated with aging date (`kydate`).

## Process Steps
1. **Validate Inputs**:
   - Validate `kydate` (MMDDYY format, valid month/day, leap year check).
   - Ensure `kyalco` is `'ALL'` or `'CO '`; if `'CO '`, validate `kyco1-3` against `ARCONT` (non-deleted, `acdel ≠ 'D'`); if `'ALL'`, ensure `kyco1-3` are zero.
   - Ensure `kyalcs` is `'ALL'` or `'SEL'`; if `'SEL'`, require non-zero `kycs01-3`.
   - Ensure `kyalsl` is `'ALL'` or `'SEL'`; if `'SEL'`, require `kyrept = 'S'` and `kytosl > kyfmsl`.
   - Ensure `kyds` is `'D'` or `'S'` (default: `'D'` if blank).
   - Ensure `kyouts` is `'O'` or blank.
   - Ensure `kyrept` is `'C'`, `'N'`, or `'S'`; if `kyslcy = 'Y'`, require `'S'`.
   - Ensure `kyclyn`, `kynod`, `kyslcy` are `'Y'` or `'N'`.
   - Ensure `kycopy` is non-zero (default: 1).
   - If `kynod = 'Y'`, require `kyds = 'D'`.
   - Validate `kycucl` against `GSTABL` if specified.

2. **Initialize Environment**:
   - Retrieve company names (`CONAME`) and aging limits (`ACLMT1-4`) from `ARCONT`.
   - Retrieve customer class description from `GSTABL` if `kycucl` is specified.
   - Set default parameters if needed (e.g., `kycopy = 1`, `kyds = 'D'`).

3. **Update Aging Date**:
   - Update `ARCONT` records with `kydate` for non-deleted records.

4. **Process Data**:
   - **Read `ARCUST` and `ARDETL`**:
     - Process hierarchically by company (`ARCO`), customer group (`ADGCO`, `ADGCUS`), and customer (`ARCUST`).
     - Match `ARDETL` records to `ARCUST` using `ADCOCU`.
   - **Assign Aging Buckets**:
     - Use `ADDATE` (invoice date) to categorize `ADAMT` into buckets (0-30, 31-60, 61-90, 91-120, over 120 days) based on `kydate`.
     - For invoices (`ADTYPE = 'I'`), calculate balance (`ADAMT - ADPART`); if `kyouts = 'O'`, include only if balance ≠ 0.
     - Adjust buckets for credits (`C`), adjustments (`J`), and payments (`P`).
     - If `kynod = 'Y'`, process only records with `ADNOD = 'Y'`.
   - **Accumulate Totals**:
     - Customer totals (`L3TOT`, `L3CUR`, `L30110`, `L31120`, `L32130`, `L3OV30`).
     - Group totals (`L4TOT`, `L4CUR`, `L40110`, `L41120`, `L42130`, `L4OV30`).
     - Company totals (`L5TOT`, `L5CUR`, `L50110`, `L51120`, `L52130`, `L5OV30`).
     - Prepay balance (`L5CPRE`) for prepaid cash invoices (`ADINV1 = 9`).
   - **Validate Balances**:
     - Compare `ARDETL` totals (`DTA` array) with `ARCUST` totals (`ARTOTD`, `ARCURD`, etc.).
     - Crossfoot `ARCUST` aging fields (`ARCURD + AR0110 + AR1120 + AR2130 + AROV30 = ARTOTD`).
     - Increment `OUTBAL` for mismatches.

5. **Generate Report**:
   - **Headers**: Include company name, date, time, aging date, customer class, and bucket ranges (0-30, 31-60, 61-90, 91-120, over 120 days).
   - **Details**: Print customer number, name, salesman, totals, last payment, finance charges, and invoice details (invoice number, date, amount, reference invoice).
   - **Optional Fields**: Include credit limit (`ARCLMT`) if `kyclyn = 'Y'`, terms, and comments (`CSCMT1-3`) from `ARCUSP`.
   - **Subtotals/Totals**: Print customer, group, and company totals; highlight out-of-balance conditions (`OUTBAL > 0`) and credit limit violations.
   - **Output Streams**: Write to `PRINT`, `PRINT2`, or `PRINT3` based on report type and errors.

6. **Execution Mode**:
   - If `kyjobq = 'Y'`, submit to batch; otherwise, run interactively.

## Business Rules
- **Aging**: Use invoice date (`ADDATE`) for buckets: 0-30, 31-60, 61-90, 91-120, over 120 days.
- **Filters**:
  - Company: `'ALL'` or specific (`kyco1-3` must exist in `ARCONT`).
  - Customer: `'ALL'` or specific (`kycs01-3` non-zero if `'SEL'`).
  - Salesmen: `'ALL'` or range (`kytosl > kyfmsl` if `'SEL'`).
  - Outstanding: Include only non-zero balances if `kyouts = 'O'`.
  - NOD: Process only `ADNOD = 'Y'` records if `kynod = 'Y'`, requiring `kyds = 'D'`.
- **Report Options**:
  - Sequence: `'C'` (customer), `'N'` (name), or `'S'` (salesman; required if `kyslcy = 'Y'`).
  - Type: `'D'` (detail) or `'S'` (summary).
  - Copies: Non-zero `kycopy` (default: 1).
  - Credit Limit: Print if `kyclyn = 'Y'`.
- **Validations**:
  - Ensure valid date, company, customer, salesman, and parameter values.
  - Report out-of-balance conditions between `ARDETL` and `ARCUST`.
  - Flag customers exceeding credit limit (`ARCLMT < ARTOTD`).

## Calculations
- **Aging Buckets**:
  - For each `ARDETL` record:
    - Calculate days from `ADDATE` to `kydate`.
    - Assign `ADAMT` to bucket based on `ADAGE` (1=0-30, 2=31-60, 3=61-90, 4=91-120, 5=over 120).
    - Adjust for partial payments (`ADPART`) and record type (`ADTYPE`).
  - Invoice (`I`): `DTA(I) += ADAMT - ADPART` (if `ADBAL ≠ 0` for `kyouts = 'O'`).
  - Credit (`C`)/Adjustment (`J`): `DTA(I) += ADAMT`.
  - Payment (`P`): `DTA(I) -= ADAMT`.
- **Totals**:
  - Customer: Sum `DTA` into `L3TOT`, `L3CUR`, `L30110`, `L31120`, `L32130`, `L3OV30`.
  - Group/Company: Aggregate customer totals into `L4*` and `L5*` variables.
  - Prepay: Adjust `L5CPRE` for prepaid invoices (`ADINV1 = 9`).
- **Balance Check**:
  - `DTA` vs. `ARCUST` totals; `ARTOTD` vs. sum of `ARCURD`, `AR0110`, `AR1120`, `AR2130`, `AROV30`.
  - Increment `OUTBAL` for mismatches.

## Data Sources
- **ARCUST**: Customer master (balances, aging, credit limit).
- **ARDETL**: Invoice details (amount, date, type, NOD).
- **ARCUSP**: Credit comments, contact info.
- **GSTABL**: Terms, salesman, and class descriptions.
- **ARCONT**: Company data, aging limits.

## Constraints
- Skip deleted records (`ARDEL = 'D'`, `ADDEL = 'D'`, `TBDEL = 'D'`).
- Release locked `ARCONT` records after updates.
- Handle Y2K-compliant dates using century checks.

