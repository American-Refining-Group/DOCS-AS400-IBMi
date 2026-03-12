The provided document is an RPG/36 program named `AR137A.rpg36`, called by the `AR137A.ocl36` OCL script. This program is responsible for processing EFT (Electronic Funds Transfer) data to create a bank upload table and generate a report for EFT transactions for each customer. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the AR137A.rpg36 Program

1. **Program Initialization**:
   - The program defines input and output files:
     - `AREFTD` (input, 256 bytes, keyed at position 2, 15-byte key): Contains EFT transaction data.
     - `ARCUST` (input, 384 bytes, keyed at position 2, 8-byte key): Customer master file.
     - `ARCONT` (input, 256 bytes, keyed at position 2, 2-byte key): Company control file.
     - `AREFTS` (output, 100 bytes): Staging file for processed EFT data.
   - Data structures (`UDS`) define key fields:
     - `KYCO` (company number, positions 101-102).
     - `KYSLDT` (selected date, positions 103-108).
     - `STATUS` (status indicator, position 109).
     - `KYUPDT` (update date, positions 110-115).
     - `Y2KCEN` and `Y2KCMP` (Year 2000 fields, positions 509-512).
   - The `ONCE` flag is initialized to zero to control one-time setup logic.

2. **One-Time Setup (ONCE Block)**:
   - If `ONCE` is zero, the program performs initial setup:
     - Captures the system date and time (`TIME` to `SYTMDT`, moved to `SYSTME` and `SYSDTE`).
     - Sets `ONCE` to 1 to prevent re-execution.
     - Converts `AFSLDT` (selected date from `AREFTD`) to YYMMDD format (`AFDYMD`) by multiplying by 10000.01 and stores it in `AFDTD8` (8-digit date starting with 20).
     - Converts `KYUPDT` (update date) similarly to `KYUYMD` and `KYUPD8`.
     - Initializes accumulators (`L2IAMT`, `L2DAMT`, `L2EAMT`, `L1IAMT`, `L1DAMT`, `L1EAMT`, `SEQ#`, `ZERO9`) to zero.
     - Chains to `ARCONT` using `AFCO` (company number from `AREFTD`) to validate the company. If not found (`*IN99` on), processing continues (error handling is minimal).

3. **Main Processing Loop (L2 Level)**:
   - The program processes records at the level-2 break (`L2`, customer key `CUSKEY` = `AFCO` + `AFCUST`):
     - Resets accumulators (`L2IAMT`, `L2DAMT`, `L2EAMT`) to zero for the current customer.
     - Clears indicator 11 (likely used for conditional logic or reporting).
     - Chains to `ARCUST` using `CUSKEY` to retrieve customer details (e.g., `AREFT`, `ARNAME`, `ARADR1-4`, etc.). If not found (`*IN99` on), processing continues.
     - Accumulates totals for the customer across invoices (handled in the L1 loop).

4. **Invoice-Level Processing (L1 Level)**:
   - For each EFT record at the level-1 break (`L1`, invoice number `AFINV#`):
     - Resets working variables (`ADWRK1`, `INVAMT`, `EFTAMT`, `L1IAMT`, `L1DAMT`, `L1EAMT`) to zero.
     - Converts `AFSLDT` to YYMMDD format (`AFDYMD`) and stores it in `AFDTD8` (8-digit date).
     - Calculates the EFT amount: `EFTAMT` = `AFAMT` (invoice amount) - `AFDISC` (discount amount).
     - Accumulates invoice totals:
       - Adds `AFAMT`, `AFDISC`, and `EFTAMT` to level-1 accumulators (`L1IAMT`, `L1DAMT`, `L1EAMT`).
       - Adds the same values to level-2 accumulators (`L2IAMT`, `L2DAMT`, `L2EAMT`) for customer totals.

5. **Output and Reporting (L2 Break)**:
   - At the level-2 break (`L2`), the program writes a record to the `AREFTS` file using the `PRTL2` exception:
     - Fields written: `AFDEL` (delete code), `AFCO` (company), `AFCUST` (customer), `AFDESC` (description), `L2IAMT` (total invoice amount), `L2DAMT` (total discount), `L2EAMT` (total EFT amount), `AFDTD8` (8-digit selected date), `KYUPD8` (8-digit update date).
   - The program also generates an EFT report (as noted in the OCL comments), likely spooled automatically for each EFT customer, including their email address from `ARCUST`.

---

### Business Rules

1. **EFT Participant Validation**:
   - The program checks the `AREFT` field in `ARCUST` (position 302, 'Y' or 'N') to determine if the customer participates in EFT. Only EFT participants are processed for bank upload and reporting.

2. **Data Accumulation**:
   - Invoice amounts (`AFAMT`), discounts (`AFDISC`), and EFT amounts (`EFTAMT`) are accumulated at both the invoice (`L1`) and customer (`L2`) levels to provide totals for reporting and bank upload.

3. **Date Handling**:
   - Dates (`AFSLDT`, `KYUPDT`) are converted to YYMMDD format and prefixed with '20' to create 8-digit dates (`AFDTD8`, `KYUPD8`) for Year 2000 compliance and consistent reporting.

4. **Company Validation**:
   - The company number (`AFCO`) is validated against `ARCONT`. If invalid, processing continues without explicit error handling, suggesting reliance on upstream validation (e.g., `AR156P`).

5. **Output Structure**:
   - The `AREFTS` file is populated with summarized EFT data per customer, including totals and dates, formatted for bank upload.
   - The EFT report includes customer-specific details (e.g., name, email, totals) for verification and auditing.

6. **Dynamic File Naming**:
   - The program uses dynamic file labels (e.g., `?9?E?L'110,6'?`) from the OCL script, ensuring flexibility across different libraries and batch dates.

---

### Tables (Files) Used

1. **AREFTD** (Input, 256 bytes, 15-byte key at position 2):
   - Contains EFT transaction data, including:
     - `AFDEL` (delete code, position 1).
     - `AFCO` (company, positions 7-8).
     - `AFCUST` (customer, positions 9-14).
     - `AFINV#` (invoice number, positions 15-21).
     - `AFAMT` (invoice amount, positions 22-26, packed).
     - `AFDISC` (discount amount, positions 27-30, packed).
     - `AFDATE` (date, positions 32-37).
     - `AFDESC` (description, positions 55-79).
     - `AFDTD8` (8-digit date, positions 193-200).
     - `AFUPDT` (update date, positions 228-233).
     - `AFSLDT` (selected date, positions 234-239).

2. **ARCUST** (Input, 384 bytes, 8-byte key at position 2):
   - Customer master file, including:
     - `ARDEL` (delete code, position 1).
     - `ARCO` (company, positions 2-3).
     - `ARCUST` (customer number, positions 4-9).
     - `ARNAME` (name, positions 10-39).
     - `ARADR1-4` (address lines, positions 40-159).
     - `AREFT` (EFT participant, position 302, 'Y' or 'N').
     - Other fields for aging, payments, and contact details.

3. **ARCONT** (Input, 256 bytes, 2-byte key at position 2):
   - Company control file, including:
     - `ACDEL` (delete code, position 1).
     - `ACCO` (company number, positions 2-3).
     - `ACNAME` (company name, positions 4-33).
     - Aging bucket limits (`ACLMT1-4`, positions 107-125).
     - EFT cash G/L number (`ACEFCG`, positions 115-122).

4. **AREFTS** (Output, 100 bytes):
   - Staging file for processed EFT data, with fields:
     - `AFDEL` (position 1).
     - `AFCO` (positions 2-3).
     - `AFCUST` (positions 4-9).
     - `AFDESC` (positions 10-34).
     - `L2IAMT` (total invoice amount, positions 35-44, packed).
     - `L2DAMT` (total discount, positions 45-49, packed).
     - `L2EAMT` (total EFT amount, positions 50-54, packed).
     - `AFDTD8` (positions 55-62).
     - `KYUPD8` (positions 63-70).

---

### External Programs Called

- **None**:
  - The `AR137A` program does not call any external programs. It performs all processing internally using the input files and writes to the output file.

---

### Summary

The `AR137A.rpg36` program processes EFT transactions by reading data from `AREFTD`, validating and retrieving customer details from `ARCUST` and company settings from `ARCONT`, and writing summarized EFT data to `AREFTS`. It accumulates invoice and discount amounts at both invoice and customer levels, formats dates for Year 2000 compliance, and generates a report for EFT customers, likely including their email addresses for automated spooling. The program ensures only EFT participants are processed and produces a bank upload table for further use in the EFT workflow. No external programs are called, and the process relies on the input files and dynamic naming provided by the calling OCL script.