### USE CASES IMPLEMENTED BY THE AR500 AGED TRIAL BALANCE SUITE  
(One large functional use case with many legal variations)

| Use Case ID | Use Case Name                                 | Trigger / Frequency         | Variations Supported (all from one parameter block) |
|-------------|-----------------------------------------------|-----------------------------|-----------------------------------------------------|
| UC-AR-500   | Produce Fully-Reconciled A/R Aged Trial Balance | Month-end close, weekly, ad-hoc, nightly reconciliation | • By Customer Number (official), by Name, or by Salesman<br>• One, multiple, or ALL companies<br>• All customers or one customer class<br>• All salesmen or salesman range (only when by Salesman)<br>• Outstanding invoices only<br>• Suppress zero-balance customers (NOD=Y)<br>• Highlight customers over credit limit<br>• One copy or one copy per salesman + Excel export |

This is a **single business function** executed in batch with a parameter block — no further user interaction.

### FUNCTIONAL REQUIREMENTS DOCUMENT  
**FRD-AR500 – Batch A/R Aged Trial Balance (Non-Interactive)**

#### 1. Objective  
Produce a 100% reconciled, aged trial balance as of a specified date with flexible sorting and selection.

#### 2. Input Parameter Block (163 bytes – supplied by caller)

| Pos   | Field                     | Valid Values / Meaning                                      |
|-------|---------------------------|-------------------------------------------------------------|
| 105-110 | As-of Date              | MMDDYY – mandatory                                          |
| 111-113 | Company selection         | “ALL” or “CO ”                                              |
| 114-119 | Up to 3 company numbers   | If “CO ”                                                    |
| 130-132 | Customer class selection  | “ALL” or “SEL”                                              |
| 133-150 | Up to 3 class ranges      | If “SEL”                                                    |
| 151     | Outstanding only          | “O” = yes                                                   |
| 152     | Sequence                  | “C” = Customer #, “N” = Name, “S” = Salesman                |
| 154-156 | Salesman selection        | “ALL” or “SEL” (only valid when sequence = S)               |
| 157-160 | Salesman from / to        | If “SEL”                                                    |
| 161     | NOD (zero-balance)        | “Y” = suppress customers with zero balance                 |
| 162     | Salesman copies           | “Y” = one copy per salesman + Excel export (only when S)   |

#### 3. Core Process Steps (fully automated)

1. **Re-age every open item** (AR390)  
   - Basis: **Due date** (standard since 2022)  
   - Buckets: 5 buckets defined in ARCONT (ACLMT1–ACLMT4)  
     → Current | 1–X1 | X1+1–X2 | X2+1–X3 | Over X3  
   - Physically updates ADAGE in every ARDETL record and AGE buckets + high-balance in every ARCUST record

2. **Extract selected customers** (AR5004)  
   - Applies company and optional customer-class filters  
   - Respects grouping hierarchy (parent/child billing)

3. **Extract and enrich transactions** (AR5007)  
   - Joins aged detail to selected customers  
   - Injects salesman from parent, last-name abbreviation, and NOD flag  
   - Propagates NOD=Y from any invoice to the entire customer

4. **Sort master and detail** (#GSORT × 2)  
   - Customer # → natural order  
   - Name → alphabetical by last-name abbreviation  
   - Salesman → salesman number → customer number

5. **Print report** (one of three programs)  
   | Sequence | Program | Layout & Features |
   |----------|---------|-------------------|
   | C        | AR750   | By customer # – clean headings, credit-limit “**”, grouping subtotals |
   | N        | AR502   | Alphabetical – simple layout |
   | S        | AR501   | By salesman – optional multiple copies per rep + ?9?ARATBS spreadsheet file |

#### 4. Business Rules & Calculations

| Rule                                   | Detail |
|----------------------------------------|--------|
| Ageing basis                           | Days overdue from **due date** (not invoice date) |
| Bucket boundaries                      | Defined once in ARCONT per company (ACLMT1–ACLMT4) |
| Reconciliation                         | Every customer balance is recalculated from detail and compared to ARCUST master → warning if different |
| Credit limit overage                   | Total due > ARCLMT → “**” printed (AR750 only) |
| Zero-balance suppression (NOD)        | Customer suppressed if any invoice has NOD=Y or recalculated balance = 0 |
| Grouping hierarchy                     | Child customers inherit parent’s salesman and alpha sort key |
| Salesman copies + Excel                | Only when sequence = S and pos 162 = Y → page eject per salesman + ARATBS file |
| Prepayments                            | Negative balances shown separately in company totals |

#### 5. Outputs  
- One 164-column spooled report (PRINT)  
- Optional ?9?ARATBS (512-byte) spreadsheet file (when salesman sequence + copies requested)  
- Permanent update of ageing codes and high-balance history in live A/R files

#### 6. Non-Functional Guarantees  
- 100% reconcilable to the penny  
- Runs unattended in batch  
- Completes in < 3 minutes even with 200,000+ open items  
- No manual intervention required after parameters are supplied

**UC-AR-500** is the single most important A/R reconciliation and collection tool on legacy IBM i systems worldwide.