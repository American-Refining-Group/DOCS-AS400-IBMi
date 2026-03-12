### Use Cases Implemented

This batch program chain implements **one primary use case**:

**UC-01: Batch Identification and Optional Automated Release of Credit-Held Orders Eligible for Release**

- Purpose: Identify open orders on credit hold that can safely be released without exceeding the customer's group credit limit, produce a report for review, and optionally auto-release low-risk ones.

### Function Requirement Document

**Function Name:** Batch Credit Hold Release Processing  
**Function ID:** FUNC-AR781-CHAIN  
**Objective:**  
Automatically identify open orders currently on credit hold due to exceeding credit limit, determine which are eligible for release based on group-level available credit and customer status, generate a review report, and (in auto mode) release eligible orders by marking them as authorized.

**Inputs (Trigger & Data Sources):**
- Trigger: Scheduled batch job or manual submission of the OCL procedure containing AR781P → AR781 → AR781A.
- Data Files:
  - BBORCL (Open Order Lines – backlog)
  - ARCUST (Customer Master)
  - ARCLGR (Credit Limit Group Membership)
  - BBORDR (Order Headers)
  - ARCONT (Company Control – for aging bucket definitions)
  - ARCUSP (Customer Contacts – optional)

**Outputs:**
- Printed Report: "CREDIT HOLD ELIGIBLE FOR RELEASE REPORT" (spooled via JBLIST/REPORT)
- Temporary Files:
  - AR780I (intermediate list of candidate group customers)
  - CREDREL (detail records of eligible orders)
  - CREDRLH (group-level summary records)
- Side Effects (when AR781A runs):
  - Updates to BBORCL: authorization fields populated on released orders (credit hold removed)
  - Updates to CREDRLH: available credit reduced
  - Audit printout of auto-released orders

**Process Steps (High-Level):**

1. **Pre-Filtering (AR781P)**  
   Scan all open order lines in BBORCL.  
   Select group (parent) customers having at least one order line where:  
   - Order number exists  
   - BLOVCL = 'Y' (flagged over credit limit)  
   - No authorization yet (BLAUIN and BLUSID blank)  
   Write one record per qualifying group customer to AR780I.

2. **Detailed Analysis & Report Generation (AR781)**  
   For each group customer in AR780I:  
   a. Retrieve all member customers in the credit group (via ARCLGR).  
   b. Accumulate group-level totals from ARCUST:  
      - Total A/R due (ARTOTD)  
      - Aging buckets (current, 1-10/1-30, etc.)  
      - Credit limit (use non-zero limit from any member)  
   c. Accumulate open order values from BBORCL for all group members:  
      - Approved orders (S2ORAP): orders not over limit OR over limit but authorized  
      - Pending/not-approved (S2ONAP): over limit and not authorized  
   d. Calculate Available Credit:  
      **Available Credit = Credit Limit – Total A/R Due – Approved Open Orders**  
   e. For each pending (over-limit, unauthorized) order line:  
      - If customer not flagged as credit problem (ARCLPB ≠ 'Y')  
      - AND (Available Credit – this order amount) ≥ 0  
      → Mark as eligible: write detail to CREDREL, ensure header exists in CREDRLH  
   f. Print group summary and eligible order details to report.

3. **Automated Release (AR781A – optional)**  
   For each eligible order in CREDREL:  
   a. Re-validate: group available credit (in CREDRLH) ≥ order amount  
   b. If yes:  
      - Update BBORCL record: set authorization fields ('ARG' / 'OTTO') → releases credit hold  
      - Reduce available credit in CREDRLH by order amount  
      - Print audit line confirming release

**Business Rules:**

1. Credit limits and exposure are evaluated at **group/parent customer level** (shared limit).
2. An order is **eligible for release** only if:  
   - Currently over limit and unauthorized  
   - Customer not marked as chronic credit problem (ARCLPB = 'Y')  
   - Sufficient available credit exists after considering A/R and approved open orders
3. **Available Credit Calculation:**  
   Credit Limit minus (Total A/R Due + Value of Approved Open Orders)
4. Orders marked with authorization initials/user ID are treated as approved regardless of limit.
5. Auto-release (AR781A) uses system identifier ('ARG'/'OTTO') to distinguish from manual authorizations.
6. Past-due balances are reported but do not automatically block release unless ARCLPB = 'Y'.

**Non-Functional Notes:**
- Fully batch-oriented (no interactive screens)
- Designed for overnight or off-peak execution
- Two-phase design allows review of report (after AR781) before committing releases (running AR781A)