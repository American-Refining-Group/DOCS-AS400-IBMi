### List of Use Cases Implemented by the AR101/AR111 Program Call Stack

The call stack, consisting of `AR101.ocl36.txt` (OCL script), `AR101.rpgle.txt` (RPGLE program for transaction entry), and `AR111.rpg36.txt` (RPG II program for transaction editing), implements the following primary use case:

1. **Accounts Receivable Transaction Entry and Validation**:
   - This use case encompasses the entry, validation, and reporting of A/R transactions (invoices and adjustments). The OCL script orchestrates the process by initializing the environment, sorting transactions, and calling `AR101` for interactive entry and `AR111` for batch validation and reporting. Users can add or update transactions via a workstation interface, with validations ensuring data integrity, followed by a printed edit report summarizing transactions and errors.

---

### Function Requirement Document: A/R Transaction Processing Function



# A/R Transaction Processing Function Requirements

## Purpose
The A/R Transaction Processing function enables the entry, validation, and reporting of Accounts Receivable (A/R) transactions (invoices and adjustments) in a batch environment, replacing the interactive screen-based input of `AR101.rpgle` with programmatic input handling. It processes input data, validates it against business rules, updates transaction records, and generates a report summarizing valid transactions, errors, and totals.

## Inputs
- **Transaction Data**:
  - Sequence number (`seq`, numeric)
  - Company number (`co`, 2-digit numeric)
  - Customer number (`cust`, 6-digit numeric)
  - Invoice number (`inv#`, 7-digit numeric)
  - Transaction type (`type`, 1-character, `I` for invoice, `J` for adjustment)
  - Transaction amount (`amt`, 9.2 decimal)
  - Transaction date (`date`, 6-digit MMDDYY)
  - Due date (`dudt`, 6-digit MMDDYY)
  - Terms code (`term`, 2-digit numeric)
  - Salesman number (`sls`, 2-digit numeric)
  - Debit G/L account (`gldr`, 8-digit numeric)
  - Credit G/L account (`glcr`, 8-digit numeric)
  - Debit company (`codr`, 2-digit numeric)
  - Credit company (`cocr`, 2-digit numeric)
  - Transaction description (`desc`, 25-character)
  - Reference invoice number (`rfiv`, 7-digit numeric, for adjustments)
- **Files**:
  - Customer master (`ARCUST`): Customer data (name, salesman, terms, inter-company number).
  - A/R control (`ARCONT`): Company data (name, A/R G/L, sales G/L, cash G/L).
  - General ledger master (`GLMAST`): G/L account details (description, deletion/inactive status).
  - A/R detail (`ARDETL`): A/R records (amount, due date, terms, salesman).
  - General system table (`GSTABL`): Terms and salesman data.
  - A/R transaction file (`ARTRAN`): Transaction storage.

## Outputs
- **Updated ARTRAN File**: Transactions marked as valid or with error flag (`'E'`).
- **Transaction Edit Report**: Lists transactions with details (sequence, customer, invoice, amount, date, type, etc.), errors, warnings, and totals (total invoices, total A/R, net change to A/R).
- **Error/Warning Messages**: Array of validation errors (e.g., invalid customer, duplicate invoice).

## Process Steps
1. **Initialization**:
   - Retrieve current date/time for reporting.
   - Initialize counters (`PAGE`, `TOTINV`, `TOTAR`, `NETCHG`) to zero.
   - Validate company number (`co`) against `ARCONT` to retrieve company name.

2. **Transaction Validation**:
   - For each transaction record:
     - **Sequence Number**: Ensure unique (`ARTRAN` chain). If duplicate, return error.
     - **Customer**: Validate `co` + `cust` exists in `ARCUST`, not deleted. If zero, treat as miscellaneous cash.
     - **Invoice Number**: Non-zero. For invoices (`I`), check non-existence in `ARDETL` (duplicate check). For adjustments (`J`), check existence in `ARDETL`.
     - **Transaction Type**: Must be `I` (invoice) or `J` (adjustment).
     - **Amount**: Non-zero.
     - **Transaction Date**: Validate format (MMDDYY). If zero, default to current date. Adjust for Y2K compliance (century calculation).
     - **Due Date (Invoices)**: If provided, validate format and ensure not earlier than transaction date. If terms provided, calculate due date (see calculations). If neither, default to 30 days.
     - **Terms Code**: If non-zero, validate against `GSTABL`.
     - **Salesman**: If non-zero, validate against `GSTABL`. Default from `ARCUST` if zero.
     - **G/L Accounts**: Validate `gldr` and `glcr` against `GLMAST` (not deleted/inactive). Default to `ACARGL` (A/R G/L) or `ACSLGL` (sales G/L) if zero, or inter-company number if applicable.
     - **Debit/Credit Companies**: Validate `codr` and `cocr` against `ARCONT`.

3. **Calculations**:
   - **Due Date (Invoices)**:
     - If terms code exists, retrieve net days (`tbnetd`) or prox days (`tbprxd`) from `GSTABL`.
     - For net days: Add `tbnetd` to transaction date (Julian conversion: add days, convert back to Gregorian).
     - For prox days: Set due date to `tbprxd` of the next month.
     - If no terms, use customer’s terms (`ARTERM`) or default to 30 days.
   - **Y2K Date Handling**:
     - If year < comparison year (`y2kcmp`), add 1 to century (`y2kcen`). Otherwise, use `y2kcen`.
     - Convert dates to 8-digit format (CCYYMMDD) for storage.
   - **Totals**:
     - For A/R detail records: Prior month balance = `ADAMT - ADPART`, Current month balance (`ARLEFT`) = `ADAMT - ADPAY`.
     - Total invoices (`TOTINV`) += `ARLEFT` (invoices only).
     - Total A/R (`TOTAR`) += `ATAMT`.
     - Net change to A/R (`NETCHG`) += `ATAMT`.

4. **Transaction Update**:
   - If no errors, write/update `ARTRAN` with transaction data.
   - If errors, mark `ARTRAN` record with `'E'` flag.

5. **Report Generation**:
   - Generate report with headers (company, date, time, page).
   - For each transaction, print: sequence, customer number/name, invoice number, amount, date, type (invoice/adjustment/credit), description, reference invoice, G/L accounts, terms, due date, salesman.
   - Print errors/warnings: customer not found, G/L not found, duplicate invoice, invoice not found (adjustments), invalid type, invalid terms.
   - At end, print totals: total invoices, total A/R, net change to A/R.

## Business Rules
1. **Transaction Types**: Must be `I` (invoice) or `J` (adjustment).
2. **Sequence Number**: Must be unique in `ARTRAN`.
3. **Company**: Must exist in `ARCONT`. Debit/credit companies (`codr`, `cocr`) must also exist.
4. **Customer**: Must exist in `ARCUST`, not deleted. Zero allowed for miscellaneous cash.
5. **Invoice Number**: Non-zero. Invoices must not exist in `ARDETL`; adjustments must reference existing invoices.
6. **Amount**: Non-zero.
7. **Transaction Date**: Valid MMDDYY format, defaults to current date if zero.
8. **Due Date**: For invoices, valid and not earlier than transaction date. Calculated from terms or defaults to 30 days.
9. **Terms Code**: If non-zero, must exist in `GSTABL`.
10. **Salesman**: If non-zero, must exist in `GSTABL`. Defaults from `ARCUST`.
11. **G/L Accounts**: Must exist in `GLMAST`, not deleted (`'D'`) or inactive (`'I'`). Defaults from `ARCONT` or inter-company number.
12. **Error Handling**: Invalid data marks `ARTRAN` with `'E'` and includes error message in report.
13. **Reporting**: Detailed transaction report with errors/warnings and totals.

## Dependencies
- **Files**: `ARTRAN` (update), `AR111S` (sorted input), `ARCUST`, `ARDETL`, `GLMAST`, `ARCONT`, `GSTABL` (input), `PRINT` (output).
- **No External Programs**: All logic is self-contained.

## Notes
- Replaces interactive screen input (`AR101D`) with programmatic input.
- Maintains Y2K compliance for date handling.
- Report format mirrors `AR111` output for consistency.

