The provided documents include an OCL program (`AR135A.ocl36.txt`) and an RPG program (`AR135A.rpg36.txt`), which are part of the Accounts Receivable (AR) Electronic Funds Transfer (EFT) draft automation process. The OCL program loads and runs the RPG program, which generates a pre-upload report for EFT transactions, detailing customer payments for submission to a bank (e.g., PNC). Below, I will explain the process steps, business rules, tables/files used, and external programs called for both the OCL and RPG programs.

---

### Process Steps of the AR135A OCL Program

The OCL program (`AR135A.ocl36.txt`) sets up the environment and executes the `AR135A` RPG program. Here are the steps:

1. **Comment Block**:
   ```
   **
   ** EFT CUSTOMER(S) ACCCOUNTS RECEIVABLE DUE REPORT
   ** PRINTS THE EFT REPORT (ELECTRONIC FUNDS TRANSFER FOR EACH EFT CUSTOMER)
   **
   ```
   - Describes the purpose: generating a report listing EFT transactions for AR customers, likely for verification before bank upload.

2. **Load Program `AR135A`**:
   ```
   // LOAD AR135A
   ```
   - Loads the `AR135A` RPG program into memory for execution.

3. **Define Files**:
   ```
   // FILE NAME-CRTRAN,LABEL-?9?E?L'110,6'?,DISP-SHR
   // FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR
   // FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR
   ```
   - Specifies three files:
     - **CRTRAN**: EFT transaction workfile with a dynamic label (`?9?E?L'110,6'?`, e.g., `PRODE123456`). Opened in shared mode (`DISP-SHR`).
     - **ARCUST**: Customer master file with label `?9?ARCUST` (e.g., `PRODARCUST`), shared mode.
     - **ARCONT**: AR control file with label `?9?ARCONT` (e.g., `PRODARCONT`), shared mode.

4. **Run the Program**:
   ```
   // RUN
   ```
   - Executes the `AR135A` RPG program, which processes the files to generate the EFT report.

---

### Process Steps of the AR135A RPG Program

The `AR135A` RPG program is a System/36-style fixed-format RPG program that generates a printed report (`LIST3`) detailing EFT transactions from the `CRTRAN` file, enriched with customer and company data from `ARCUST` and `ARCONT`. It calculates totals and formats the output for pre-upload verification. Here’s a step-by-step breakdown of the RPG program’s logic:

1. **File and Data Definitions**:
   - **Files**:
     - `CRTRAN`: Input primary file (IP), 256 bytes, keyed on position 2 (likely company/customer).
     - `ARCUST`: Input chained file (IC), 384 bytes, keyed on position 2, for customer details.
     - `ARCONT`: Input chained file (IC), 256 bytes, keyed on position 2, for company details.
     - `LIST3`: Output printer file (O), 164 bytes, for the EFT report.
   - **Input Specifications**:
     - `CRTRAN`: Fields include `ATDEL` (delete flag), `ATSEQ#` (sequence), `ATCO` (company), `ATCUST` (customer), `ATINV#` (invoice), `ATAMT` (amount), `ATDISC` (discount), `ATTYPE` (type), `ATDATE` (date), `ATGLCR` (credit G/L), `ATGLDR` (debit G/L), `ATDESC` (description), `ATDUDT` (due date), `ATTERM` (terms), `ATCODI` (discount company), `ATGLDI` (discount G/L), `ATCOCR` (credit company), `ATCODR` (debit company), `ATNOD` (notification of difference), and date fields (`ADTXCN`, `ADTXYY`, `ADTXMM`, `ADTXDD`, `ADDUD8`, `DISCY`, `DISMD`, `KYUPDT`, `KYSEDT`).
     - `ARCUST`: Fields include customer details like `ARNAME` (name), `ARADR1-4` (address), `ARZIP5/9/14` (zip codes), `ARTOTD` (total due), `ARCURD` (current due), `ARPYMT` (last payment), `ARPDAT` (last payment date), `ARSLS#` (salesman), `ARTERM` (terms), `AREFT` (EFT participant flag), etc.
     - `ARCONT`: Fields include `ACNAME` (company name), `ACARGL` (AR G/L), `ACDSGL` (discount G/L), `ACCSGL` (cash G/L), `ACEFCG` (EFT cash G/L), etc.
     - UDS: Control fields like `KYCO` (company), `KYSLDT` (select date), `STATUS`, `KYUPDT` (update date), `Y2KCEN`, `Y2KCMP`.
   - **Arrays**:
     - `HD`: 3-element array for printer headings (60 bytes each).

2. **Initialization (`ONCE` Block)**:
   ```
   C           ONCE      IFEQ *ZERO                      B1
   C                     TIME           SYTMDT 120
   C                     MOVELSYTMDT    SYSTME  60
   C                     MOVE SYTMDT    SYSDTE  60
   C           ATCO      CHAINARCONT               99
   C                     EXCPTPRTHD3
   C                     Z-ADD1         ONCE    10
   C                     Z-ADD*ZEROS    LRIAMT 102
   C                     Z-ADD*ZEROS    LRDAMT 102
   C                     Z-ADD*ZEROS    LREAMT 102
   C                     Z-ADD*ZEROS    SEQ#    50
   C                     Z-ADD*ZEROS    ZERO9   90
   C                     END                             E1
   ```
   - Runs once (`ONCE = 0`):
     - Captures system time (`SYTMDT`) and splits into `SYSTME` (time) and `SYSDTE` (date).
     - Chains to `ARCONT` using `ATCO` (company from `CRTRAN`) to retrieve company data (e.g., `ACNAME`).
     - Outputs header (`PRTHD3`) to `LIST3`.
     - Initializes totals: `LRIAMT` (invoice amount), `LRDAMT` (discount amount), `LREAMT` (EFT amount), `SEQ#` (sequence), `ZERO9` (zero field).
     - Sets `ONCE = 1` to prevent re-execution.

3. **Main Processing Loop**:
   ```
   C   01                DO
   C           ATKEY     CHAINARCUST               99
   C           ATAMT     SUB  ATDISC    EFTAMT 102
   C                     ADD  ATAMT     LRIAMT
   C                     ADD  ATDISC    LRDAMT
   C                     ADD  EFTAMT    LREAMT
   C                     EXCPTPRTDTL
   C                     END
   ```
   - For each `CRTRAN` record (indicator `*IN01` on, as primary file):
     - Chains to `ARCUST` using `ATKEY` (company/customer) to retrieve customer details (e.g., `ARNAME`).
     - Calculates `EFTAMT = ATAMT - ATDISC` (net EFT amount).
     - Accumulates totals: `LRIAMT += ATAMT`, `LRDAMT += ATDISC`, `LREAMT += EFTAMT`.
     - Outputs detail line (`PRTDTL`) to `LIST3`.

4. **Output Totals and End**:
   ```
   C                     EXCPTPRTLRT
   ```
   - After processing all `CRTRAN` records, outputs totals (`PRTLRT`) to `LIST3`, including `LRIAMT`, `LRDAMT`, `LREAMT`.

5. **Output Specifications**:
   - **PRTHD3 (Header)**:
     - Prints company name (`ACNAME`), report title ("ELECTRONIC FUNDS WORKFILE PRE-UPLOAD TO PNC EDIT REPORT"), contact info, and column headers (e.g., "SEQ#", "CUST#", "NAME", "DATE", "INV#", "AMOUNT", "DISCOUNT", "EFT AMOUNT").
     - Includes `KYUPDT` (bank upload date) and `SYSDTE`/`SYSTME` (run date/time).
   - **PRTDTL (Detail)**:
     - Prints transaction details: `ATSEQ#`, `ATCUST`, `ARNAME`, `KYSEDT`, `ATINV#`, `ATAMT`, `ATDISC`, `EFTAMT`.
   - **PRTLRT (Totals)**:
     - Prints total invoice amount (`LRIAMT`), discount (`LRDAMT`), and EFT amount (`LREAMT`).

6. **Program Termination**:
   - Ends after processing all `CRTRAN` records and outputting totals, returning control to the OCL caller.

---

### Business Rules

The OCL and RPG programs enforce the following business rules:

1. **OCL Program (AR135A.ocl36.txt)**:
   - **Dynamic File Labeling**: Uses `?9?` (e.g., `PROD`) and `?L'110,6'?` (e.g., `123456`) for `CRTRAN`, `?9?ARCUST`, and `?9?ARCONT`, ensuring environment flexibility (e.g., production, test).
   - **Shared Access**: All files (`CRTRAN`, `ARCUST`, `ARCONT`) are opened in shared mode (`DISP-SHR`), supporting concurrent processing in multi-user environments.
   - **Report Purpose**: Prepares data for a pre-upload EFT report, likely for verification before bank submission.

2. **RPG Program (AR135A.rpg36.txt)**:
   - **Report Generation**:
     - Produces a formatted report (`LIST3`) for EFT transactions, including customer details, transaction amounts, discounts, and net EFT amounts.
     - Includes headers with company name, run date/time, bank upload date (`KYUPDT`), and contact info for verification.
   - **Data Validation**:
     - Chains to `ARCUST` to validate customer data; indicator `*IN99` flags errors (though not explicitly handled).
     - Chains to `ARCONT` for company name and G/L accounts.
   - **Calculations**:
     - Computes `EFTAMT = ATAMT - ATDISC` for each transaction.
     - Accumulates totals: invoice amount (`LRIAMT`), discount (`LRDAMT`), EFT amount (`LREAMT`).
   - **EFT-Specific**:
     - Only processes records where `AREFT = 'Y'` (EFT participant flag in `ARCUST`), implied by context.
     - Uses `KYSEDT` (select date) and `KYUPDT` (update date) for transaction timing, critical for bank uploads.
   - **Deletion Handling**:
     - Skips records with `ATDEL = 'D'` (delete flag) via `*IN01` and `NCD` (non-deleted condition).
   - **Y2K Compliance**:
     - Uses `Y2KCEN` and `Y2KCMP` for date handling, ensuring correct century for `KYSEDT` and other dates.
   - **Report Format**:
     - Structured for clarity with headers, detail lines, and totals.
     - Includes customer name (`ARNAME`) and sequence (`ATSEQ#`) for traceability.

---

### Tables/Files Used

| File Name | Type | Record Length | Key Location | Usage/Description |
|-----------|------|---------------|--------------|-------------------|
| **CRTRAN** | Input Primary (IP) | 256 | 2 | EFT transaction workfile; contains transaction details (sequence, company, customer, invoice, amount, discount, type, dates, G/L accounts). |
| **ARCUST** | Input Chained (IC) | 384 | 2 | Customer master; provides name, address, EFT flag (`AREFT`), terms, payment history, etc. |
| **ARCONT** | Input Chained (IC) | 256 | 2 | AR control; provides company name (`ACNAME`), G/L accounts (`ACARGL`, `ACDSGL`, `ACCSGL`, `ACEFCG`). |
| **LIST3** | Output Printer (O) | 164 | N/A | Printer file for the EFT pre-upload report, with headers, details, and totals. |

- **OCL Context**: Files use dynamic labels (`?9?E?L'110,6'?`, `?9?ARCUST`, `?9?ARCONT`) and shared mode.
- **RPG Context**: `CRTRAN` is primary, driving the loop; `ARCUST` and `ARCONT` are chained for lookup.

---

### External Programs Called

- **OCL Program (AR135A.ocl36.txt)**:
  - Calls the RPG program `AR135A` via `LOAD` and `RUN`.
  - No other external programs or procedures are invoked.

- **RPG Program (AR135A.rpg36.txt)**:
  - Does not call any external programs or procedures (no `CALL` operations).
  - Uses internal exception output (`EXCPT`) to write to `LIST3`.
  - Relies on the OCL caller for file setup and control.

---

### Summary

**OCL Program (`AR135A.ocl36.txt`)**:
- **Process Steps**: Loads `AR135A`, opens `CRTRAN`, `ARCUST`, and `ARCONT` with dynamic labels in shared mode, and runs the program.
- **Business Rules**: Ensures flexible file access and concurrent processing, preparing data for the EFT report.
- **Files Used**: `CRTRAN`, `ARCUST`, `ARCONT`.
- **External Programs**: Calls `AR135A`.

**RPG Program (`AR135A.rpg36.txt`)**:
- **Process Steps**: Initializes with system date/time and company data, processes `CRTRAN` records, chains to `ARCUST` and `ARCONT` for details, calculates EFT amounts and totals, and outputs a formatted report to `LIST3`.
- **Business Rules**: Generates a pre-upload EFT report for verification, validates customer/company data, skips deleted records, and ensures Y2K-compliant dates. Accumulates totals for invoice, discount, and EFT amounts.
- **Files Used**: `CRTRAN`, `ARCUST`, `ARCONT`, `LIST3`.
- **External Programs**: None.

The `AR135A` programs produce a critical report for EFT processing, ensuring accurate customer and transaction data before bank submission. They rely on `AR135` and `AR135B` for prior data entry and updates. If you need further details on the report format or integration with PNC, additional documentation would be required.