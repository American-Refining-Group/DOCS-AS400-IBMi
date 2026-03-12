### List of Use Cases Implemented by the Program Suite

The call stack (`AR200.ocl36.txt`, `AR200P.rpgle.txt`, `AR200.rpg36.txt`, `AR210.rpg36.txt`, `AR211.rpg36.txt`, `AR2011.ocl36.txt`, `AR2011.rpg36.txt`) implements a single primary use case in the accounts receivable (AR) cash receipts posting process. This use case is executed as a batch process, taking input files and producing outputs without direct screen interaction, aligning with the requirement for a large function that processes inputs to complete the process.

**Use Case: Post Cash Receipts and Generate General Ledger (GL) Journal Entries with Product-Level Discount Allocation**

- **Description**: This use case processes cash receipt transactions to update AR files, generate GL journal entries, produce sales and cash receipts journal reports, and allocate discounts by product for invoices. It handles invoices, adjustments, payments, miscellaneous cash, and inter-company transactions, ensuring accurate financial updates and reporting.
- **Scope**: Encompasses transaction validation, AR updates, GL entry creation, journal reporting, and product-specific discount allocation.
- **Inputs**: Transaction register (`CRTRGG`), customer master (`ARCUST`), AR detail (`ARDETL`), control (`ARCONT`), sales finder (`SA5FIND`), and general table (`GSTABL`) files.
- **Outputs**: Updated AR files (`ARDETL`, `ARCUST`), GL entries (`TEMGEN`), daily summaries (`ARDALY`), discount details (`SA5DSC`), and journal reports (`REPORT`, `REPORTP`).
- **Components**:
  - `AR200P`: Initializes and validates input data, sets journal date.
  - `AR200`: Updates AR files, generates distribution entries, and prints the transaction posting register.
  - `AR210`: Creates GL journal entries from distribution records.
  - `AR211`: Summarizes GL entries, updates GL master, and prints sales/cash receipts journals.
  - `AR2011`: Allocates discounts by product for invoices.
  - OCL scripts (`AR200.ocl36.txt`, `AR2011.ocl36.txt`): Orchestrate file sorting, program execution, and cleanup.

This is a single cohesive use case, as all programs work together to process cash receipts end-to-end, with `AR2011` providing specialized discount analysis.

---

<xaiArtifact artifact_id="ccfd71c5-0367-4ef4-92da-f2f372d37e48" artifact_version_id="aace6422-fb6d-409b-9df3-033cdb6e9f45" title="Cash Receipts Posting Function Requirements.md" contentType="text/markdown">

# Cash Receipts Posting Function Requirements

## Overview
The Cash Receipts Posting function processes cash receipt transactions to update accounts receivable (AR) balances, generate general ledger (GL) journal entries, produce journal reports, and allocate discounts by product for invoices. It operates as a batch process, taking input files and producing file and report outputs without user interaction.

## Inputs
- **Transaction Register (CRTRGG)**: Contains cash receipt transactions (company, customer, invoice, amount, discount, type, date, GL accounts, description, inter-company code).
- **Customer Master (ARCUST)**: Customer details (name, balances, payment dates).
- **AR Detail (ARDETL)**: Open invoice items (amount, aging code, payments).
- **AR Control (ARCONT)**: Company settings (GL accounts, journal numbers).
- **Sales Finder (SA5FIND)**: Invoice line items (product, quantity, price).
- **General Table (GSTABL)**: Product/discount code mappings.
- **Journal Date (JRNLDT)**: Supplied via user data structure (UDS).

## Outputs
- **Updated Files**:
  - `ARDETL`: Updated invoice amounts, payments, and aging.
  - `ARCUST`: Updated customer balances and payment dates.
  - `TEMGEN`: GL journal entries (debit/credit, account, amount).
  - `ARDALY`: Daily AR summaries (company, account, amount).
  - `SA5DSC`: Product-level discount details (invoice, product, discount amount, percentage).
- **Reports**:
  - Transaction Posting Register (`REPORT`, `REPORTP`): Details invoices, adjustments, payments, and totals.
  - Sales/Cash Receipts Journals (`REPORT`, `REPORTP`): Summarized GL entries by account.

## Process Steps
1. **Validate Input and Initialize**:
   - Validate `CRTRGG` existence and journal date format (CCYYMMDD).
   - Initialize accumulators, journal numbers, and Y2K century settings.
2. **Sort Transactions**:
   - Sort `CRTRGG` by company, customer, and invoice for processing.
3. **Process Transactions**:
   - Read `CRTRGG` records (invoices, adjustments, payments, misc cash).
   - Match to `ARDETL`, `ARCUST`, `ARCONT` for validation and updates.
   - Update `ARDETL` (invoice amounts, payments), `ARCUST` (balances, payment dates), and write to `ARHIST` (audit trail) unless inter-company.
   - Generate GL distribution entries (`ARDIST`) with debit/credit accounts.
4. **Generate GL Journal Entries**:
   - Process `ARDIST` to create `TEMGEN` entries (debit/credit, account, amount, description).
   - Handle inter-company transactions with separate GL accounts (`ACTRGL`).
   - Create discount entries using discount GL account (`ADGLDI`).
5. **Summarize and Post to GL**:
   - Summarize `TEMGEN` entries by account for sales (`SJ`) or cash receipts (`CR`) journals.
   - Update `GLMAST` descriptions and write daily summaries to `ARDALY`.
   - Produce journal reports with headers (company, date, user) and totals.
6. **Allocate Discounts by Product**:
   - Match `CRTRGG` discounts to `SA5FIND` invoice lines by company, customer, invoice.
   - Calculate total invoice price (quantity * price per line).
   - Allocate discount percentage (line price / total price) and amount per product.
   - Adjust final line to match total discount.
   - Write discount details to `SA5DSC` (product, amount, percentage, journal data).
7. **Clean Up**:
   - Delete temporary files (`CRTRGG`, `CRDIGG`, `CRTGGG`, `AR211S`).

## Business Rules
1. **Transaction Types**:
   - **Invoices**: Add to `ARDETL`, `ARCUST` balances, and aging; generate GL entries.
   - **Adjustments**: Modify `ARDETL`, `ARCUST` (e.g., credit memos); generate GL entries.
   - **Payments**: Reduce `ARDETL` amounts, update `ARCUST` payment dates, apply discounts; generate GL entries.
   - **Miscellaneous Cash**: Generate GL entries without AR updates.
2. **Inter-Company Transactions**:
   - Skip `ARDETL`, `ARCUST`, `ARHIST` updates; generate GL entries using inter-company account (`ACTRGL`).
3. **Discount Allocation**:
   - Calculate discount percentage per invoice line: `(quantity * price) / total invoice price`.
   - Multiply percentage by total discount (`ATDISC`) to get line discount.
   - Adjust final line to ensure sum equals `ATDISC`.
   - Handle credit/rebill cases: Set percentage to 1 (positive price) or -1 (negative price) if total price is zero.
4. **GL Entries**:
   - Positive amounts: Debit AR (`ADGLDR`), credit GL (`ADGLCR`).
   - Negative amounts: Reverse accounts (debit `ADGLCR`, credit `ADGLDR`).
   - Discounts: Separate debit/credit entry to `ADGLDI`.
   - Inter-company: Use `ACTRGL` for AR side, flip debit/credit based on amount sign.
5. **Y2K Compliance**:
   - Convert dates to CCYYMMDD using century (`Y2KCEN`, e.g., 19/20) and comparison year (`Y2KCMP`, e.g., 80).
6. **Reporting**:
   - Transaction register: Lists invoices, adjustments, payments, totals, with audit fields (user, workstation, date).
   - Journals: Summarize by account, include debits/credits, support paperless output.
7. **Error Handling**:
   - Validate file existence and matches (company, customer, invoice).
   - Ensure GL entries balance (debits = credits).
   - Adjust discounts to prevent rounding discrepancies.

## Calculations
- **Discount Percentage**: `PCT = (SAQTY * SAPRCE) / TOTPRC`, where `TOTPRC = Σ(SAQTY * SAPRCE)` across invoice lines. If `TOTPRC = 0`, set `PCT = 1` (positive) or `-1` (negative).
- **Discount Amount**: `AMT = PCT * ATDISC`.
- **Final Adjustment**: If `ΣAMT ≠ ATDISC`, adjust last line’s `AMT` by difference (`ATDISC - ΣAMT`).
- **GL Amounts**: For payments, debit = `ADAMT ± ADDISC` (subtract if positive, add if negative); credit = `ADAMT`. For negative amounts, reverse accounts and adjust signs.

## Constraints
- Requires sorted `CRTRGG` from prior steps.
- Assumes valid `SA5FIND` data for invoice-product mappings.
- No direct user interaction; relies on pre-populated files.
- No updates to `ARCONT` journal numbers (per 8/06/14 change).

</xaiArtifact>


##  Call stack Summary
| **Program** | **Main Purpose** | **Tables Used** | **Outputs (Files or Side Effects)** |
|-------------|------------------|----------------|-------------------------------------|
| **AR200.ocl36.txt** | Orchestrates the cash receipts posting process by sorting transactions, calling RPG programs, and cleaning up temporary files. | Input: CRIEGG, CRPSGG, ARCUST, ARCONT<br>Output: CRTRGG, CRDIGG, CRTGGG, AR211S | CRTRGG (sorted transactions), CRDIGG (distributions), CRTGGG (GL entries), AR211S (sorted GL entries); deletes temporary files; routes reports to ARPOST/TESTOUTQ. |
| **AR200P.rpgle.txt** | Validates and initializes the cash receipts process by setting journal date and checking input file existence. | Input: CRIEGG, CRPSGG, ARCONT | Sets journal date in UDS; sets U1 switch for cancellation; no file outputs. |
| **AR200.rpg36.txt** | Processes cash receipt transactions, updates AR files, generates GL distribution entries, and prints the transaction posting register. | Input: ARTRAN (?9?CRTRGG), ARDETL, ARCUST, ARCONT<br>Output: ARHIST, ARDIST (?9?CRDIGG), REPORT, REPORTP | Updates ARDETL (invoice amounts), ARCUST (balances, payment dates); writes ARHIST (audit trail), ARDIST (GL distributions); prints transaction register (REPORT, REPORTP). |
| **AR210.rpg36.txt** | Generates GL journal entries from AR distribution records for invoices, adjustments, payments, and discounts. | Input: ARDIST (?9?CRDIGG), ARCONT<br>Output: ARTEMG (?9?CRTGGG) | Writes ARTEMG (GL journal entries); no reports or side effects. |
| **AR211.rpg36.txt** | Summarizes GL journal entries, updates GL master, generates daily AR summaries, and prints sales/cash receipts journals. | Input: ARTEMG (?9?CRTGGG), AR211S (?9?AR211S), GLMAST, ARCONT<br>Output: TEMGEN, ARDALY, REPORT, REPORTP | Writes TEMGEN (GL postings), ARDALY (daily summaries); prints sales/cash receipts journals (REPORT, REPORTP); no ARCONT updates (per 8/06/14). |
| **AR2011.ocl36.txt** | Calls AR2011 RPG to process product-level discount allocations, ensuring transaction file existence. | Input: ARTRAN (?9?CRTRGG), SA5FIND, GSTABL, ARCUST, SA5DSC | No direct outputs; invokes AR2011 RPG; no file creation/deletion. |
| **AR2011.rpg36.txt** | Allocates cash receipt discounts across invoice line items by product, writing detailed discount records. | Input: ARTRAN (?9?CRTRGG), SA5FIND, GSTABL, ARCUST<br>Output: SA5DSC | Writes SA5DSC (product-level discount details); no reports or side effects. |
