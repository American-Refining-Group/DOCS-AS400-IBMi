The `AR137A.rpg36.txt` is an RPG III program used in IBM i (AS/400) systems, invoked by the `AR137A.ocl36.txt` OCL script as part of the EFT (Electronic Funds Transfer) draft notice filtering process. This program processes EFT transaction data to generate summary records for EFT reports, aggregating amounts by customer and company. Below, I’ll explain the process steps, business rules, tables (files) used, and external programs called, providing a clear and concise analysis.

### Process Steps of the AR137A RPG Program

The `AR137A` program reads transaction data from the `AREFTD` file, validates company and customer information using `ARCONT` and `ARCUST`, and writes summarized EFT data to the `AREFTS` file. Here’s a step-by-step breakdown of its execution:

1. **Program Header**:
   - `H P064 B AR137A`: Specifies the program identifier (`P064`), batch mode (`B`), and program name (`AR137A`).
   - Sets up the program for batch processing in the IBM i environment.

2. **File Definitions**:
   - `FAREFTD IP F 256 256 15AI 2 DISK`: Defines `AREFTD` as the primary input file (256 bytes, indexed with a key starting at position 2, length 15 bytes).
   - `FARCUST IF F 384 384 8AI 2 DISK`: Defines `ARCUST` as an input file (384 bytes, indexed with a key starting at position 2, length 8 bytes).
   - `FARCONT IF F 256 256 2AI 2 DISK`: Defines `ARCONT` as an input file (256 bytes, indexed with a key starting at position 2, length 2 bytes).
   - `FAREFTS O F 100 100 DISK A`: Defines `AREFTS` as an output file (100 bytes, append mode `A` for adding records).

3. **Input Specifications**:
   - `IAREFTD NS 01 1NCD`:
     - Defines the `AREFTD` record layout for non-deleted records (`1NCD`, not deleted).
     - Fields include:
       - `AFDEL` (position 1, delete flag).
       - `AFSEQ#` (positions 2-6, sequence number).
       - `AFCO` (positions 7-8, company number, level break `L3`).
       - `AFCUST` (positions 9-14, customer number, level break `L2`).
       - `CUSKEY` (positions 7-14, composite key: company + customer).
       - `AFINV#` (positions 15-21, invoice number, level break `L1`).
       - `AFAMT` (positions 22-26, packed, amount).
       - `AFDISC` (positions 27-30, packed, discount).
       - `AFTYPE` (position 31, transaction type).
       - `AFDATE` (positions 32-37, date).
       - `AFDESC` (positions 55-79, description).
       - `AFDUDT` (positions 80-85, due date).
       - `AFDAT8` (positions 193-200, 8-digit date).
       - `AFDUD8` (positions 201-208, 8-digit due date).
       - `AFDSDT` (positions 219-226, selection date).
       - `AFUPDT` (positions 228-233, update date).
       - `AFSLDT` (positions 234-239, selection date).
   - `IARCUST NS`: Defines `ARCUST` fields, including `ARDEL` (delete code), `ARCO` (company number), `ARCUST` (customer number), `ARNAME`, `ARADR1` to `ARADR4`, `AREFT` (EFT participant flag), and others.
   - `IARCONT NS`: Defines `ARCONT` fields, including `ACDEL` (delete code), `ACCO` (company number), `ACNAME`, and EFT-related fields like `ACEFCG` (EFT cash G/L number).
   - `I UDS`: Defines Local Data Area (LDA) fields:
     - `KYCO` (positions 101-102, company number).
     - `KYSLDT` (positions 103-108, selection date).
     - `STATUS` (position 109, status flag).
     - `KYUPDT` (positions 110-115, update date).
     - `Y2KCEN` (positions 509-510, century).
     - `Y2KCMP` (positions 511-512, company).

4. **Calculation Logic**:
   - **Initialization (ONCE Block)**:
     - If `ONCE` is zero (first iteration):
       - Captures system time (`SYTMDT`) and date (`SYSDTE`).
       - Sets `ONCE` to 1 to prevent re-execution.
       - Converts `AFSLDT` to an 8-digit format (`AFDYMD`, `AFDTD8`) by multiplying by 10000.01 and prefixing with ‘20’.
       - Converts `KYUPDT` to an 8-digit format (`KYUYMD`, `KYUPD8`) similarly.
       - Initializes accumulators (`L2IAMT`, `L2DAMT`, `L2EAMT`, `L1IAMT`, `L1DAMT`, `L1EAMT`, `SEQ#`, `ZERO9`) to zero.
       - Chains to `ARCONT` using `AFCO` to validate the company number. If not found (`*IN99` on), skips further processing (`END`).
   - **Level Break Processing (L2 - Customer Level)**:
     - For each customer (`L2` level break on `AFCUST`):
       - Resets accumulators (`L2IAMT`, `L2DAMT`, `L2EAMT`) to zero.
       - Chains to `ARCUST` using `CUSKEY` (company + customer) to retrieve customer details. If not found (`*IN99` on), continues.
       - Sets indicator `*IN11` off.
       - Writes a summary record to `AREFTS` via `EXCPTPRTL2` (see output specification).
   - **Level Break Processing (L1 - Invoice Level)**:
     - For each invoice (`L1` level break on `AFINV#`):
       - Resets working variables (`ADWRK1`, `INVAMT`, `EFTAMT`, `L1IAMT`, `L1DAMT`, `L1EAMT`) to zero.
       - For non-deleted records (`01` indicator):
         - Converts `AFSLDT` to an 8-digit format (`AFDYMD`, `AFDTD8`).
         - Calculates `EFTAMT` as `AFAMT` (amount) minus `AFDISC` (discount).
         - Accumulates `AFAMT`, `AFDISC`, and `EFTAMT` into `L1IAMT`, `L1DAMT`, and `L1EAMT` (invoice-level totals).
         - Accumulates the same into `L2IAMT`, `L2DAMT`, and `L2EAMT` (customer-level totals).

5. **Output Specification**:
   - `OAREFTS EADD PRTL2`:
     - Writes a record to `AREFTS` on customer-level break (`L2`) using the `PRTL2` exception output.
     - Fields written:
       - `AFDEL` (position 1, delete flag).
       - `AFCO` (positions 2-3, company number).
       - `AFCUST` (positions 4-9, customer number).
       - `AFDESC` (positions 10-34, description).
       - `L2IAMT` (positions 35-44, packed, invoice amount total).
       - `L2DAMT` (positions 45-49, packed, discount amount total).
       - `L2EAMT` (positions 50-54, packed, EFT amount total).
       - `AFDTD8` (positions 55-62, 8-digit date).
       - `KYUPD8` (positions 63-70, 8-digit update date).

6. **Program Termination**:
   - The program processes all records in `AREFTD` using RPG’s implicit file processing cycle, writing summary records to `AREFTS` for each customer, and terminates when complete.

### Business Rules

The `AR137A` program enforces the following business rules:

1. **Non-Deleted Records**:
   - Only processes records in `AREFTD` where `AFDEL` is not ‘D’ (not deleted, enforced by `1NCD`).

2. **Company Validation**:
   - Validates the company number (`AFCO`) against `ARCONT`. If not found, skips processing for that company.

3. **Customer Validation**:
   - Retrieves customer details from `ARCUST` using `CUSKEY` (company + customer). If not found, continues processing but may skip customer-specific details.

4. **Amount Aggregation**:
   - Calculates EFT amounts (`EFTAMT = AFAMT - AFDISC`) for each invoice.
   - Accumulates totals at the invoice level (`L1IAMT`, `L1DAMT`, `L1EAMT`) and customer level (`L2IAMT`, `L2DAMT`, `L2EAMT`).
   - Writes customer-level totals to `AREFTS` when the customer changes (`L2` break).

5. **Date Formatting**:
   - Converts `AFSLDT` and `KYUPDT` to 8-digit formats (e.g., `200YYYYMMDD`) for consistency in output.
   - Uses `Y2KCEN` and `Y2KCMP` from the LDA for Y2K compliance.

6. **Output Structure**:
   - Generates summary records in `AREFTS` with company, customer, description, totals, and dates, formatted for EFT reporting.

### Integration with AR137A OCL Script

The `AR137A` RPG program is invoked by the `AR137A.ocl36.txt` script via `// LOAD AR137A` and `// RUN`. The OCL script:
- Clears the `AREFTS` file before execution to ensure a fresh dataset.
- Defines input files `AREFTD` (as `?9?E?L'110,6'?` or `?9?AREFTX`), `ARCONT`, and `ARCUST`, and the output file `AREFTS`, matching the RPG file definitions.
- Uses the update date (`?L'110,6'?`, `KYUPDT` from the LDA) set by the `AR137P` program to select the correct transaction file.
The `AR137A` program processes these files to produce summarized EFT data in `AREFTS`, which is used by subsequent programs (`AR137E`, `AR137B`) for detailed processing and final report generation.

### Tables (Files) Used

The program uses the following files:

1. **AREFTD**:
   - Type: Primary input (256 bytes, indexed, key at position 2, length 15).
   - Purpose: Contains EFT transaction data (e.g., invoices, amounts, dates).
   - Fields: `AFDEL`, `AFSEQ#`, `AFCO`, `AFCUST`, `AFINV#`, `AFAMT`, `AFDISC`, `AFTYPE`, `AFDATE`, `AFDESC`, `AFDUDT`, `AFDAT8`, `AFDUD8`, `AFDSDT`, `AFUPDT`, `AFSLDT`.

2. **ARCUST**:
   - Type: Input (384 bytes, indexed, key at position 2, length 8).
   - Purpose: Contains customer data for validation and enrichment.
   - Fields: `ARDEL`, `ARCO`, `ARCUST`, `ARNAME`, `ARADR1` to `ARADR4`, `AREFT`, and others.

3. **ARCONT**:
   - Type: Input (256 bytes, indexed, key at position 2, length 2).
   - Purpose: Contains accounts receivable control data for company validation.
   - Fields: `ACDEL`, `ACCO`, `ACNAME`, `ACEFCG`, and others.

4. **AREFTS**:
   - Type: Output (100 bytes, append mode).
   - Purpose: Stores summarized EFT data for each customer.
   - Fields: `AFDEL`, `AFCO`, `AFCUST`, `AFDESC`, `L2IAMT`, `L2DAMT`, `L2EAMT`, `AFDTD8`, `KYUPD8`.

### External Programs Called

The `AR137A` program does not call any external programs. It operates solely on the defined files and performs its logic internally.

### Summary

The `AR137A` RPG III program processes EFT transactions from `AREFTD`, validates company and customer data using `ARCONT` and `ARCUST`, and generates summarized EFT data in `AREFTS`. It aggregates invoice amounts, discounts, and EFT amounts by customer, ensuring non-deleted records and proper date formatting. The output supports subsequent EFT reporting steps in the `AR137` process.

**Tables Used**: `AREFTD`, `ARCUST`, `ARCONT`, `AREFTS`.
**External Programs Called**: None.