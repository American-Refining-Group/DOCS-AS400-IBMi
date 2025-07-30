### List of Use Cases Implemented by the AR201 Call Stack

The call stack (`AR201.ocl36.txt`, `AR200.rpg36.txt`, `AR210.rpg36.txt`, `AR211.rpg36.txt`) implements a single primary use case in the IBM AS/400 or iSeries environment:

1. **Accounts Receivable (A/R) Transaction Posting and Journal Entry Generation**:
   - This use case processes A/R transactions (invoices, adjustments, payments, and miscellaneous cash) from a sorted transaction file, updates customer and A/R detail files, generates journal entries for the general ledger, and produces sales and cash receipts journals along with a transaction posting register.

### Function Requirement Document



# A/R Transaction Posting and Journal Entry Generation

## Overview
This function processes accounts receivable transactions (invoices, adjustments, payments, miscellaneous cash) to update customer balances, A/R details, and history, generate general ledger journal entries, and produce sales/cash receipts journals and a transaction posting register.

## Inputs
- **Transaction File (ARTRAN)**: Sorted records with company code, customer number, invoice number, transaction type (I=invoice, J=adjustment, P=payment), amount, discount, G/L accounts, dates, and description.
- **Customer Master (ARCUST)**: Customer data including balances, payment info, and aged amounts.
- **A/R Detail (ARDETL)**: Invoice-level details with payment and adjustment history.
- **A/R Control (ARCONT)**: Company-level control data (journal numbers, G/L accounts).
- **G/L Master (GLMAST)**: G/L account descriptions.
- **System Parameters**: Journal date, user ID, workstation ID, Y2K century settings.

## Outputs
- **Updated Files**:
  - `ARCUST`: Updated customer balances, payment info, aged amounts.
  - `ARDETL`: Updated or new invoice, adjustment, payment records.
  - `ARHIST`: Transaction history records.
  - `ARCONT`: Updated journal numbers (pre-8/06/14).
  - `TEMGEN`: G/L journal entries.
  - `ARDALY`: Daily transaction records.
- **Reports**:
  - Transaction Posting Register (`REPORT`, `REPORTP`): Lists invoices, adjustments, payments, totals.
  - Sales/Cash Receipts Journals (`REPORT`, `REPORTP`): Summarized or detailed journal entries.

## Process Steps
1. **Initialize**:
   - Retrieve system date, time, user ID, workstation ID.
   - Chain to `ARCONT` for journal numbers and G/L accounts.
   - Convert dates to CCYYMMDD format with Y2K compliance (century = 19 if year ≥ 80, else 20).

2. **Process Transactions**:
   - Read `ARTRAN` records sequentially, grouped by company, transaction type.
   - Determine transaction type:
     - **Invoice (I)**: Update customer balances, A/R details, history; generate sales journal (SJ) entries.
     - **Adjustment (J)**: Adjust balances, details, history; generate SJ entries.
     - **Payment (P)**: Reduce balances, update payment info, details, history; generate cash receipts (CR) entries.
     - **COD Invoice (Y)**: Generate SJ entries, skip A/R updates.
     - **Miscellaneous Cash**: Adjust finance charges or record as cash; generate CR entries.
   - Skip A/R updates for inter-company customers (`ATICCD = 'IC'`), generate journal entries only.

3. **Update Files**:
   - **ARCUST**: Update total due, aged balances, period totals (month-to-date, year-to-date), payment info.
   - **ARDETL**: Add/update invoice, adjustment, payment, or prepaid records (sequence number, paid amount, sales).
   - **ARHIST**: Log transaction details (except inter-company).
   - **ARCONT**: Increment journal numbers (`ACARJ#`, `ACSLJ#`) for payments, invoices/adjustments (pre-8/06/14).

4. **Generate Journal Entries**:
   - Create debit/credit entries in `ARDIST` (from `AR200`):
     - Invoices/Adjustments: Debit A/R (`ADGLDR`), credit sales (`ADGLCR`).
     - Payments: Debit cash (`ADGLCR`), credit A/R (`ADGLDR`).
     - Discounts: Debit/credit discount account (`ADGLDI`).
     - Inter-Company: Use inter-company G/L account (`ACTRGL`).
   - Process `ARDIST` to `ARTEMG` (from `AR210`):
     - Adjust amounts for miscellaneous cash, discounts.
     - Handle negative amounts by reversing debit/credit.
     - Write inter-company entries with swapped debit/credit types.
   - Summarize entries in `TEMGEN`/`ARDALY` (from `AR211`):
     - Summarize by account if `GDSUMM = 'S'`, else write individual entries.
     - Include company, account, journal type, number, sequence, date, amount, description.

5. **Produce Reports**:
   - **Transaction Posting Register**: List invoices, adjustments, payments, totals (net change, cash, discounts).
   - **Sales/Cash Receipts Journals**: List journal entries (detailed or summarized) with G/L accounts, amounts, descriptions, and totals.

## Business Rules
1. **Transaction Types**:
   - Invoices/adjustments increase A/R balances, generate SJ entries.
   - Payments reduce A/R balances, generate CR entries.
   - COD invoices skip A/R updates, generate SJ entries.
   - Miscellaneous cash reduces finance charges or records as cash.

2. **Inter-Company**:
   - Skip `ARCUST`, `ARDETL`, `ARHIST` updates; generate journal entries with inter-company G/L account.

3. **Calculations**:
   - **Aged Balances**: Update `AGE` array (5 periods) based on transaction date and amount.
   - **Net Change**: `NETCHG = INVAMT + ADJAMT - TOTREC`.
   - **Payment Amount**: `ATCASH = ATAMT - ATDISC` for payments.
   - **Journal Entries**:
     - Standard: Debit A/R, credit sales/cash.
     - Negative amounts: Reverse debit/credit accounts.
     - Discounts: Separate entry with `ADGLDI`.
     - Inter-Company: Use `ACTRGL` with swapped debit/credit based on amount sign.

4. **Date Handling**:
   - Convert dates to CCYYMMDD using Y2K logic (century = 19 if year ≥ 80, else 20).
   - Compare invoice date to journal date if `LDRETL = 'R'` for discrepancy flagging.

5. **Summarization**:
   - Summarize journal entries by account if `GDSUMM = 'S'`; otherwise, write individual entries.

6. **Description**:
   - Retain 25-character description from cash receipts, with date in second field (per 4/20/05 change).

7. **Journal Numbers**:
   - Increment `ACARJ#` (A/R journal) for payments, `ACSLJ#` (sales journal) for invoices/adjustments (pre-8/06/14).

## Assumptions
- Input files are pre-sorted by company, transaction type.
- G/L accounts in `ARCONT`, `GLMAST` are valid.
- Shared file access (`DISP-SHR`) allows concurrent processing.
- No external program calls; all logic is internal via subroutines.

