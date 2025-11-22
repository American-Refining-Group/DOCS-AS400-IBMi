**AR500P – Interactive A/R Aged Trial Balance Prompt Program (RPG IV)**  
This is the **interactive front-end** that is called by the OCL wrapper you posted earlier.  
It displays a single prompt screen (originally 5250, now converted to Profound UI web), validates all user entries, updates the last ageing date in the company control files, and then returns control to the OCL which decides whether to run the batch report (AR500).

### Overall Process Flow (Step-by-Step)

| Step | What the program does | Key Indicators / Actions |
|------|-----------------------|--------------------------|
| 1    | First-time entry (*IN09 = ON) | Runs **ONETIM** subroutine once per session |
| 2    | **ONETIM** – initialization | • Gets current time/date<br>• Defaults ageing date to today<br>• Reads first 3 active companies from **ARCONT** → displays company names (dco1/dco2/dco3)<br>• Sets defaults: ALL companies, Detail report, No job queue, etc.<br>• Reads **GSCONT** (global control) to see if a default company is forced (CO xx) |
| 3    | Normal screen cycle (*IN01 = ON) | Runs **SCREEN1** subroutine |
| 4    | **SCREEN1** – full validation | Validates **every** field the user entered, in this order:<br>1. Ageing date (calls **@DTEDT** subroutine – full leap-year logic including Y2K century handling)<br>2. Company selection (ALL or CO + up to 3 valid company numbers)<br>3. Customer class selection (ALL or SEL + ranges)<br>4. Outstanding invoices flag (O or blank)<br>5. Report sequence (C=Customer, N=Name, S=Salesman)<br>6. Salesman selection (ALL/SEL + from/to if SEL, only allowed with S report)<br>7. Job queue flag (Y/N)<br>8. Print credit limit (Y/N)<br>9. Number of copies (defaults to 1 if zero)<br>10. NOD flag (Y/N) and related rules<br>11. Salesman copies flag (Y/N)<br>12. Customer class code (if used) |
| 5    | If any validation fails | Sets **IN81** (error) + **IN90** (redisplay screen), turns on the relevant error indicator (50–62), puts message text in **MSG40**, and loops back to screen |
| 6    | If all validations pass | Sets **IN15** = ON (valid entry), updates **ACDATE** (last ageing date) in every active **ARCONT** record (loops through file with indicator 71) |
| 7    | On F3/F12 or Cancel (KG) | Moves “CANCEL” into positions 124-129 of the parameter area (**KYCANC**) so the calling OCL knows to skip batch submission |
| 8    | Normal accept (Enter with no errors) | Returns to OCL with all **KYxxx** parameters filled in positions 105–163 of the display file record (the same parameter string the OCL passed in and reads back) |
| 9    | OCL then decides | If KYCANC = “CANCEL” → end<br>Else if KYJOBQ = “Y” → submit AR500 batch job<br>Else call AR500 synchronously |

### Business Rules Implemented

| Field / Option | Valid Values | Business Rule |
|----------------|--------------|---------------|
| Ageing Date    | MMDDYY       | Must be valid date, full leap-year logic including century rule |
| Company        | ALL or CO + 1–3 company numbers | If CO selected, all entered companies must exist and not be deleted |
| Customer Class | ALL or SEL + 3 ranges | If SEL, at least one range must be filled |
| Outstanding Invoices | O or blank | Anything else = error |
| Report Sequence | C, N, or S | C = Customer#, N = Alpha name, S = Salesman |
| Salesman Selection | ALL or SEL | SEL only allowed when Report Sequence = S |
| From/To Salesman | Numeric | To must be ≥ From |
| Job Queue | Y or N | Determines if batch job is submitted after interactive run |
| Print Credit Limit | Y or N | |
| NOD (Not on Detail?) | Y or N | Only allowed with Detail report |
| Salesman Copies | Y or N | |
| Copies | 00 → forced to 01 | |

### Physical Files (Tables) Used

| Logical/Physical File | Description | Key fields / Usage |
|-----------------------|-------------|--------------------|
| **AR500PFM** (display file) | The prompt screen (originally AR500P, converted to Profound UI) | Returns all parameters in fixed positions |
| **ARCONT** | A/R Company Control file (one record per company) | Key = company number (ACCO), updated with last ageing date (ACDATE), used for company validation and name display |
| **GSCONT** | Global system control file | Record “00” contains default company (GXCONO) if single-company site |

### External Programs / Subprocedures Called

| Program | Type | Purpose |
|---------|------|---------|
| **PROFOUNDUI(HANDLER)** | Display file handler | Specified on the F-spec – this is the web/GUI interface layer |
| None | — | No CALL, CALLP, or /EXEC SQL to other RPG programs – all logic is self-contained |

### Parameter Area Layout (positions the OCL reads/writes)

| Positions | Field | Meaning |
|-----------|-------|---------|
| 105–110   | KYDATE | Ageing date (MMDDYY) |
| 111–113   | KYALCO | Company selection: ALL or CO |
| 114–119   | KYCO1–KYCO3 | Company numbers when CO |
| 120       | KYJOBQ | Y = submit batch job |
| 121–122   | KYCOPY | Number of copies |
| 124–129   | KYCANC | “CANCEL” if user pressed F3/F12 |
| 130–132   | KYALCS | Customer class: ALL/SEL |
| 133–150   | KYCS01–KYCS03 | Customer class ranges |
| 151       | KYOUTS | Outstanding invoices flag |
| 152       | KYREPT | Report sequence C/N/S |
| 153       | KYCLYN | Print credit limit Y/N |
| 154–156   | KYALSL | Salesman selection ALL/SEL |
| 157–160   | KYFMSL/KYTOSL | Salesman from/to |
| 161       | KYNOD | NOD flag |
| 162       | KYSLYC | Salesman copies Y/N |
| 163       | KYCUCL | Customer class code (rare) |

### Summary

**AR500P** is a classic interactive parameter prompt program whose sole job is to:

1. Show the user a nicely formatted ageing criteria screen.
2. Enforce dozens of business rules with clear error messages.
3. Update the “last aged” date in every company’s control record.
4. Pass back a fully-validated parameter block to the calling OCL.
5. Allow clean cancellation.

After it returns, the OCL (AR500P.ocl36) decides whether to run the actual report program **AR500** interactively or submit it to batch — exactly the pattern used in hundreds of old AS/400 third-party accounting packages in the 1990s–2000s.