### FUNCTIONAL REQUIREMENTS DOCUMENT  
**FRD-AR500-MODERN – Next-Generation Batch A/R Aged Trial Balance API / Service (2026+)**

#### 1. Business Objective & Summary  
Replace the 35-year-old AR500 OCL/RPG suite with a **modern, callable, fully reconciled A/R Aged Trial Balance service** that can be invoked from any platform (RPG-ILE, Python, Node.js, Java, Power Automate, etc.) via a simple program call, REST API, or message queue.

The new service will:
- Produce the **exact same trusted, penny-perfect numbers** as the current suite  
- **Permanently re-age** the live A/R files exactly as AR390 does today  
- Deliver the report in PDF + optional Excel/CSV  
- Eliminate obsolete options (NOD flag, Salesman sequence) while keeping the core accounting integrity intact

This is **not** a cosmetic rewrite – it is a **strategic modernisation** that preserves the single most important reconciliation report in the company while making it callable from anywhere.

#### 2. Modern Call Interface (Example – ILE RPG or REST)

```cobol
CALL PGM(AR500NG) PARM(
     asOfDate       /* '2025-11-30'       YYYY-MM-DD     */
     companyList    /* '01,05,ALL'               */
     custClass      /* '' or 'GOV,HIGH'          */
     sequence       /* 'C' or 'N'                */
     includeCreditLimitFlag /* 'Y' or 'N'         */
     outputFormat   /* 'PDF', 'EXCEL', 'BOTH'    */
     returnJobId    /* 10-digit job number out   */
)
```

Or via REST POST /api/ar-aged-trial-balance

#### 3. Input Parameters (Modernised & Slimmed Down)

| Position / Name               | Type     | Mandatory? | Valid Values / Meaning                                   | Notes / Changes from Legacy |
|-------------------------------|----------|------------|----------------------------------------------------------|-----------------------------|
| asOfDate                      | Date / String | Yes     | YYYY-MM-DD or YYYYMMDD                                   | Mandatory ageing date       |
| companyList                   | String   | Yes     | Comma-separated list or 'ALL'                            | Replaces 111–119            |
| custClass                     | String   | No      | Blank = all, or comma-separated list (e.g. 'GOV,HIGH')   | Replaces 130–150          |
| sequence                      | Char(1)  | Yes     | 'C' = By Customer Number<br>'N' = By Customer Name       | **'S' (Salesman) permanently removed** |
| includeCreditLimitFlag        | Char(1)  | No      | 'Y' = show & highlight over-limit customers<br>'N' = hide | Replaces position 153       |
| outputFormat                  | String   | No      | 'PDF', 'EXCEL', 'BOTH' (default = PDF)                   | New – modern formats        |
| requestId / correlationId     | String   | No      | For async calls / logging                                |                             |

**Explicitly Removed Legacy Options**  
- NOD (position 161) → **permanently removed** – zero-balance customers are now **always suppressed** (this is the 2025 business standard)  
- Salesman sequence ('S'), salesman range, salesman copies (positions 152, 154–162) → **permanently removed** – no longer supported

#### 4. Core Process Steps (Same Trusted Logic – Modern Implementation)

1. **Re-age all open items**  
   - Basis: **Days overdue from due date** (standard since 2022)**  
   - Bucket calculation:  
     `DaysOverdue = AsOfDate – DueDate`  
     Bucket 1 (Current): DaysOverdue ≤ ACLMT1  
     Bucket 2: ACLMT1 + 1 ≤ DaysOverdue ≤ ACLMT2  
     …  
     Bucket 5 (Over): DaysOverdue > ACLMT4  
   - Physically update ADAGE in every ARDETL record  
   - Update packed AGE(1)–AGE(5) and high-balance fields in every ARCUST record

2. **Apply company & customer-class filters**  
   - Extract only customers matching companyList and custClass (if supplied)

3. **Suppress zero-balance customers**  
   - Hard-coded rule: any customer with recalculated balance = 0 is excluded (former NOD=Y is now default)

4. **Sort**  
   - 'C' → by Customer Number  
   - 'N' → by Last Name Abbreviation (ARLSTN)

5. **Reconcile & generate output**  
   - Recalculate every customer balance from detail  
   - Compare to ARCUST.ARTOTD → raise alert if different  
   - If includeCreditLimitFlag = 'Y' → compare to ARCLMT → highlight over-limit with “**”  
   - Produce PDF (164-column layout identical to AR750) + optional Excel/CSV

#### 5. Key Business Rules (Clarified & Hardened)

| Rule                                      | Detail                                                                                     |
|-------------------------------------------|--------------------------------------------------------------------------------------------|
| Ageing basis                            | **Always by due date** – days overdue = AsOfDate – DueDate                                  |
| Bucket boundaries                         | Defined once per company in ARCONT (ACLMT1–ACLMT4) – 5 buckets total                       |
| Zero-balance suppression                  | **Always on** – customers with $0 balance are never shown (former NOD=Y is now mandatory)  |
| Reconciliation                            | 100% mandatory – detail total must equal ARCUST.ARTOTD or alert is raised                  |
| Credit limit highlighting                 | Optional – when requested, “**” shown if recalculated balance > ARCLMT                     |
| Grouping hierarchy                        | Child customers inherit parent’s salesman and name for sorting (preserved)                 |
| Salesman reporting                        | **Removed** – no longer available (use separate commission reports instead)               |
| Output formats                            | PDF (official audit trail) + optional Excel/CSV for analysis                               |

#### 6. Outputs  
- PDF spooled file or stream (exact replica of current AR750 layout)  
- Optional Excel/CSV file (modern replacement for old ARATBS)  
- Permanent update of ageing codes and high-balance history in live files  
- Return code + job/log ID for monitoring

**This modernised service preserves **30+ years of accounting trust** while eliminating obsolete options and enabling seamless integration into today’s enterprise systems.