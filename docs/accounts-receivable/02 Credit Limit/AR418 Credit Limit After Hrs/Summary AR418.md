# Function Requirement Document  
**FRD-AR418 – Credit Hold Override & Order Release Automation (Headless/Batch/API Use Case)**

### Functional Overview  
**Business Capability:**  
Authoritatively release an order from credit hold, place an order on credit hold, or remove prior authorization — with full audit trail and automatic email notification to CSR and Salesman — **without using the interactive green-screen**.

This is the **only major use case** implemented by the entire AR418 stack (OCL + RPG + CL).

### Use Case Implemented  
| Use Case ID | Name                                    | Trigger                                 | Actor (intended)                     |
|-------------|-----------------------------------------|-----------------------------------------|--------------------------------------|
| UC-AR418-01 | Credit-Hold Override & Order Authorization | Need to release or re-hold an order     | Credit Manager, EDI, API, Night Job |

All other behaviors (time restriction, concurrency lock, email) are **supporting rules** of this single use case.

### Required Inputs (to fully replace the screen)  
| Input Field       | Format   | Required? | Source in original screen |
|-------------------|----------|-----------|---------------------------|
| Company           | 2-digit  | Yes       | kyco                      |
| Customer Number   | 6-digit  | Yes       | kycust                    |
| Order Number      | 6-digit  | Yes       | kyord#                    |
| Action            | Char(1)  | Yes       | One of: 'A'=Authorize/Release, 'H'=Place on Hold, 'U'=Unauthorize |
| Authorizer Initials (3 chars) | Char(3) | Yes only if Action='A' | kyauin |
| Requesting User ID (for audit) | Char(10) | Yes    | Passed from job (kyuser)  |

### Process Steps (Headless Execution Flow)

1. **Concurrency & Authority Check**  
   - Fail immediately if another job is actively running AR418 or AR880  
   - Fail if Action='A' and current time is 06:00–19:59 and requesting user is not in the whitelist (ED, BRIAN, CATHY, etc.)

2. **Validation**  
   - Company must exist in BICONT and not be deleted  
   - Customer must exist in ARCUST and not be deleted  
   - Order must exist in BBORCL (Open Order Credit file)

3. **Business Rule Enforcement**

| Action | Current BBORCL State Allowed? | Result on BBORCL Record                              | Message Returned               |
|--------|-------------------------------|-------------------------------------------------------|--------------------------------|
| 'A'    | blauin = blanks OR '   '      | → blauin = supplied 3 initials<br>→ blusid = requesting user<br>→ blovcl = 'N' | “ORDER HAS BEEN AUTHORIZED” |
| 'A'    | blauin already populated      | → Rejected                                           | “ORDER HAS ALREADY BEEN AUTHORIZED” |
| 'H'    | Any                           | → blovcl = 'Y'<br>→ blauin = blanks<br>→ blusid = blanks | “ORDER HAS BEEN PLACED ON HOLD” |
| 'U'    | blauin populated              | → blauin = blanks<br>→ blusid = blanks               | “ORDER HAS BEEN UNAUTHORIZED” |
| 'U'    | blauin already blank          | → Rejected                                           | “ORDER DOES NOT NEED AUTHORIZATION” |

4. **Audit & Notification (always executed on success)**  
   - Generate two spool files using exact same layout as original CREMAL/SMEMAL  
     - One named “ORDER CREDIT STATUS”  → routed to CSROUTQ → emails CSR & Credit team  
     - One named “ORDER CREDIT STATUS2” → routed to SLMNOUTQ → emails Salesman  
   - Content includes: order #, customer name/address, credit limit, full aging buckets, last payment date, authorizer initials, timestamp

5. **Return Status**  
   Success → return code 0 + descriptive message  
   Failure → return code >0 + exact error text from COM array (01–11)

### Calculations Performed (none complex)
- No monetary calculations – all dollar fields are read directly from ARCUST and BBORCL  
- Aging buckets are pre-calculated nightly in ARCUST (fields arcurd, ar0110, ar1120, ar2130, arov30)

### Non-Functional / Supporting Rules (must be preserved)
| Rule | Enforcement Point |
|------|-------------------|
| Only one user/system may process a given order at a time | OCL ACTIVE-AR418 check |
| During 6 AM – 8 PM only specific users may authorize | RPG onetim subroutine |
| Full email notification on every status change | RPG exception output + spool files |
| Test mode never sends real emails | AR418TC CLP deletes/distributes safely |

### Success Criteria  
The order’s credit-hold status in BBORCL is changed exactly as it would have been via the green-screen, and all affected parties receive the identical emailed report — with zero user interaction required.

This is now ready to be wrapped as an API, called from EDI, or used in overnight automation.