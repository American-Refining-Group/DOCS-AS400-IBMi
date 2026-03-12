The provided document is an **OCL (Operation Control Language)** procedure from the IBM System/36 environment (running on AS/400 or later IBM i in compatibility mode). OCL is a job control/scripting language used to prepare and execute programs, typically RPG II programs, by clearing files, allocating resources (via file overrides), loading programs into memory, and running them sequentially.

The procedure appears to be part of a **credit limit exceedance report** or selection process for accounts receivable (AR) customers whose open orders push them over their credit limits. The comment at the top ("CREATE THE SELECTION TABLE TO CHOOSE ONLY CUSTOMER GROUP NUMBERS THAT HAVE ORDERS OVER THE CREDIT LIMIT") describes the overall goal.

### Process Steps

1. **Clear a temporary/work file**:
   - `CLRPFM FILE(?9?AR780I)`
     - Clears all records from the file/member `?9?AR780I` (likely a temporary intermediate file from a prior step; `?9?` is a common substitution for a library or parameter).

2. **Execute the first program (AR781P)**:
   - Comments indicate: Load program **AR781P**.
   - File allocations/overrides:
     - `BBORCL` → label `?9?BBORCL`, shared access (`DISP-SHR`)
     - `ARCUST` → `?9?ARCUST`, shared
     - `ARCUSX` → `?9?ARCUSX`, shared
     - `AR780I` → `?9?AR780I`, shared
   - Then `// RUN`
     - Runs AR781P with these file overrides. This program likely performs initial selection or calculation of customers/orders exceeding credit limits, populating the cleared `AR780I` file.

3. **Clear output files**:
   - `CLRPFM ?9?CREDREL`
     - Clears the main detail output file (`CREDREL` – likely "Credit Release" or "Credit Related" details).
   - `CLRPFM ?9?CREDRLH`
     - Clears a header or summary output file (`CREDRLH`).

4. **Execute the main processing program (AR781)**:
   - Comments indicate: Load program **AR781**.
   - File allocations/overrides (mostly shared multi-member access with `DISP-SHRMM`):
     - Input: `AR780I` (intermediate from prior step), `ARCONT`, `ARCUST`, `ARCUSP`, `BBORCL`, `ARCLGR`, `BBORDR`, `BBORCLAU`
     - Output: `CREDREL` and `CREDRLH`
   - Printer: `JBLIST` (priority 0) – output likely spools to this printer file.
   - Then `// RUN`
     - Runs AR781, which reads the intermediate data, joins with customer/order/contact files, checks credit limits vs. open orders, and populates the `CREDREL` (details) and `CREDRLH` (headers/summary) files. It probably generates a report/list of customers requiring credit review/release.

5. **Execute the final/reporting program (AR781A)**:
   - Comments indicate: Load program **AR781A**.
   - File allocations/overrides (shared multi-member):
     - `BBORCL`, `CREDREL`, `CREDRLH`
   - Then `// RUN`
     - Runs AR781A, which likely sorts, formats, and prints the final report from the populated `CREDREL`/`CREDRLH` files (e.g., a "Customers Over Credit Limit" list).

The procedure chains three related RPG programs (AR781P → AR781 → AR781A) with intermediate file clearing to produce a credit exceedance selection/report.

### External Programs Called (Loaded and Run)
- **AR781P**: Initial selection/preparation program.
- **AR781**: Main processing program (core logic for credit checks and output generation).
- **AR781A**: Final formatting/reporting program.

These are likely RPG II programs (common in System/36-style environments).

### Tables/Files Used
Many files use `?9?` prefix (parameterized library) and shared access. Common AR (Accounts Receivable) and order files:

- **Input/Master Files**:
  - `BBORCL` / `?9?BBORCL`: Open order lines or backlog.
  - `ARCUST` / `?9?ARCUST`: Customer master.
  - `ARCUSX` / `?9?ARCUSX`: Customer extensions/index.
  - `ARCONT`: Customer contacts.
  - `ARCUSP`: Customer special/pricing?
  - `ARCLGR`: AR ledger or credit group.
  - `BBORDR` / `?9?BBORDR`: Order headers.
  - `BBORCLAU`: Order clause/values?

- **Temporary/Intermediate Files**:
  - `AR780I` / `?9?AR780I`: Work file populated by AR781P, read by AR781.

- **Output Files**:
  - `CREDREL` / `?9?CREDREL`: Detail records of over-limit customers/orders.
  - `CREDRLH` / `?9?CREDRLH`: Header/summary records.

- **Printer**:
  - `JBLIST`: Spooled report output.

This is a classic batch process for identifying customers whose outstanding orders exceed credit limits, likely for credit hold/release review.