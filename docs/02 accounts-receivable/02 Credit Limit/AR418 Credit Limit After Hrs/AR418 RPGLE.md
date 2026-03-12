**AR418 – Order Authorization / Credit Limit Override Program (RPG IV version)**  
This is the **modernized IBM i (AS/400) RPGLE** version of the classic System/36 program whose OCL you saw earlier.  
It is an **interactive green-screen (or Profound UI) program** used by Credit Managers to:

- Release orders that are on credit hold  
- Put orders back on credit hold (unauthorize)  
- Temporarily override credit limits for a specific order  
- Automatically send email notifications (via spool files) to CSR and Salesman

### Overall Process Flow (step-by-step)

| Step | Indicator / Condition | What the program does |
|-------|-----------------------|-----------------------|
| 1     | First screen load (`*IN09`) | Runs **onetim** subroutine → checks user ID and time-of-day restriction |
| 2     | **onetim** subroutine | • If user is NOT one of the authorized people (ED, BRIAN, CATHY, etc.) AND current time is 06:00–20:00 → shows message **“CANNOT RUN DURING NORMAL BUSINESS HOURS”** and forces exit. This is the famous MG03 restriction (Dec 2017). |
| 3     | User enters Company / Customer / Order# → presses Enter (`*IN01`) | Chains to **ARCUST** (customer master) to validate customer exists |
| 4     | User enters one of these options: <br>• `kyplhd = 'Y'` → Put on hold <br>• `kyunau = 'Y'` → Unauthorize (remove authorization) <br>• `kyauin` = initials → Authorize/release | Program builds key `orky14` (Co + Cust + Order) and chains to **BBORCL** (Open Order Credit Limit file) |
| 5     | **screen1** subroutine performs all business rule checks: <br>• Company invalid → msg 01 <br>• Order not found → msg 02 <br>• Order already deleted → msg 08 <br>• Trying to release an order that is already authorized → msg 05 <br>• Trying to unauthorize an order that isn’t authorized → msg 06 <br>• Order is over limit but no initials entered → msg 03 <br>• etc. | Shows appropriate error message and redisplays screen |
| 6     | If all checks pass and user entered initials → **AUTH** exception output <br>• Updates BBORCL record: <br>  `blauin` = entered initials <br>  `blusid` = user ID <br>  `blovcl` = 'N' (no longer over limit flag) <br>• Prints spool file **CREMAL** (Credit Manager email) and **SMEMAL** (Salesman email) with full order/customer aging info | Order is RELEASED from credit hold |
| 7     | If user entered `kyplhd = 'Y'` → **HOLD** exception output <br>• Sets `blovcl = 'Y'`, clears initials/userid | Order is put BACK on credit hold |
| 8     | If user entered `kyunau = 'Y'` → **UNAU** exception output <br>• Clears `blauin` and `blusid` | Authorization removed |
| 9     | Email notification logic (MG02 – Nov 2017) <br>• Looks up CSR who took the order (`bltkby`) → gets email from **BBCSR** <br>• Looks up Salesman (`blslmn`) → gets email from **BBSLSM** <br>• Looks up person who is authorizing/unauthorizing → gets their email <br>• Spools two reports (CREMAL and SMEMAL) that go to outqueues → external spool-to-email product (SpooFlex) sends the actual emails | Automatic notification to CSR, Salesman, and Credit staff |

### Key Business Rules Implemented

| Rule | Description |
|------|-----------|
| Only specific users can run this program during 6 AM – 8 PM (MG03) | Prevents regular staff from overriding credit outside hours |
| Only one person can work on a customer/order at a time | Handled by the OCL (ACTIVE-AR418 check) – program itself assumes exclusive access |
| Cannot release an order that is already released | Shows “ORDER HAS ALREADY BEEN AUTHORIZED” |
| Cannot unauthorize an order that isn’t authorized | Shows “ORDER DOES NOT NEED AUTHORIZATION” |
| Must enter 3-character initials to authorize | Enforced with message 03 |
| Full audit trail via spool files (JBLIST was commented out, now uses CREMAL/SMEMAL) | Every release/hold generates emailed report with aging, credit limit, etc. |
| Automatic email to CSR and Salesman on status change | Modern addition (2017) |

### Files / Tables Used

| File     | Physical File (typical) | Purpose |
|----------|-------------------------|--------------------------------------------|
| **ARCUST**   | ARCUST                  | Customer master – name, address, credit limit, aging buckets |
| **BBORCL**   | BBORCL (update)         | Open Order Credit Limit file – holds orders on credit hold, authorization fields |
| **BBCSR**    | BBCSR                   | Customer Service Rep master – contains email address (fields 37-86) |
| **BBSLSM**   | BBSLSM                  | Salesman master – contains email address (fields 36-85) |
| **BICONT**   | BICONT                  | Company/branch control (validates company number) |
| **GSCONT**   | GSCONT                  | Global control file – used to default company number on first entry |
| **AR418D**   | Display file            | Workstations screen (converted to Profound UI handler) |
| **CREMAL**   | Spool file (140 chars)  | Email notification to Credit/AR staff |
| **SMEMAL**   | Spool file (140 chars)  | Email notification to Salesman |

### External Programs / Objects Called

| Object          | Type      | Called from where? | Purpose |
|-----------------|-----------|---------------------|-------------------------------------------|
| **PROFOUNDUI(HANDLER)** | Display file handler | F-spec of AR418D | Modern web GUI (replaces 5250 screen) |
| **SpooFlex** (external product) | Not called directly | Consumes CREMAL/SMEMAL spool files from outqueues | Actually sends the emails |
| No other RPG programs are called from this source | — | — | All logic is self-contained |

### Summary – What AR418 Actually Does in Plain English

> “A credit manager (or one of the few authorized people) types in a customer order number.  
> The system shows the customer’s current balance, aging, and credit limit.  
> The manager can:
> - Type their 3 initials → the order is released from credit hold and can ship  
> - Type ‘Y’ in “Place on Hold” → the order goes back on hold  
> - Type ‘Y’ in “Unauthorize” → removes previous authorization  
> Every action automatically emails the CSR, the salesman, and the credit team with full details.  
> Regular employees are blocked from using this option during normal business hours.”

This combination of the original OCL (concurrency protection) + this RPGLE program is a very typical, extremely robust credit-override solution used in wholesale/distribution companies running on IBM i for the last 30+ years.