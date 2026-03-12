The `AR137B.rpg36.txt` is an RPG III program used in IBM i (AS/400) systems, invoked by the `AR137B.ocl36.txt` OCL script (despite the OCL referencing `AR136B`, likely a typo) as the final step in the EFT (Electronic Funds Transfer) draft notice filtering process. This program generates detailed EFT reports for each EFT customer, printing transaction details, customer information, and email addresses to multiple printer files. Below, I’ll explain the process steps, business rules, tables (files) used, and external programs called, ensuring a clear and concise analysis.

### Process Steps of the AR137B RPG Program

The `AR137B` program processes EFT transaction data, retrieves customer and email information, and generates formatted reports for EFT draft notices. Here’s a step-by-step breakdown of its execution:

1. **Program Header**:
   - `H P064 B AR137B`: Specifies the program identifier (`P064`), batch mode (`B`), and program name (`AR137B`).
   - Indicates the program is designed for batch processing to produce EFT reports.

2. **File Definitions**:
   - `FARDTWSC IF F 11 11 8AI 1 DISK`: Defines `ARDTWSC` as an input file (11 bytes, indexed, 8-byte key starting at position 1).
   - `FAREFTD IP F 256 256 15AI 2 DISK`: Defines `AREFTD` as the primary input file (256 bytes, indexed, 15-byte key starting at position 2).
   - `FARCUST IF F 384 384 8AI 2 DISK`: Defines `ARCUST` as an input file (384 bytes, indexed, 8-byte key starting at position 2).
   - `FARCONT IF F 256 256 2AI 2 DISK`: Defines `ARCONT` as an input file (256 bytes, indexed, 2-byte key starting at position 2).
   - `FARCUFMX IF F 266 266 L21AI EXTK DISK`: Defines `ARCUFMX` as an input file (266 bytes, indexed, 21-byte key, externally defined).
   - `FLIST1 O F 120 120 OF PRINTER`, `FLIST2 O F 120 120 OA PRINTER`, `FLIST3 O F 120 120 OB PRINTER`, `FLIST4 O F 120 120 OC PRINTER`: Defines four printer files (`LIST1` to `LIST4`) for report output, each 120 bytes, with different overflow indicators (`OF`, `OA`, `OB`, `OC`).

3. **Extension and Input Specifications**:
   - `E EIFM 5 60`: Defines an array `EIFM` with 5 elements, each 60 bytes, for storing email addresses.
   - `E HD 1 3 60`: Defines an array `HD` with 1 element of 3 subfields, each 60 bytes, for printer headings.
   - `IAREFTD NS 01 1NCD`: Defines the `AREFTD` record layout for non-deleted records (`1NCD`):
     - Fields: `AFDEL` (delete flag), `AFSEQ#` (sequence number), `AFCO` (company number, `L3`), `AFCUST` (customer number, `L2`), `CUSKEY` (company + customer), `AFINV#` (invoice number, `L1`), `AFAMT` (amount, packed), `AFDISC` (discount, packed), `AFTYPE`, `AFDATE`, `AFDESC`, `AFDUDT`, `AFDAT8`, `AFDUD8`, `AFDSD8`, `AFDSDT`, `AFUPDT`, `AFSLDT`.
   - `IARCUST NS`: Defines `ARCUST` fields: `ARDEL`, `ARCO`, `ARCUST`, `ARNAME`, `ARADR1` to `ARADR4`, `AREFT` (EFT participant), and others for customer details.
   - `IARCONT NS`: Defines `ARCONT` fields: `ACDEL`, `ACCO`, `ACNAME`, `ACEFCG` (EFT cash G/L number), and others for control data.

4. **Calculation Logic** (Truncated):
   - The provided source is truncated, but based on the context and output specifications, the program:
     - Processes `AREFTD` records in a loop (`L1`, `L2`, `L3` level breaks for invoice, customer, and company).
     - Chains to `ARCUST` using `CUSKEY` to retrieve customer details (e.g., `ARNAME`, `ARADR1` to `ARADR3`).
     - Chains to `ARCONT` using `AFCO` to validate company information.
     - Chains to `ARDTWSC` to retrieve the email address count for each customer.
     - Chains to `ARCUFMX` to retrieve email addresses (`EIFM` array) for EFT notices.
     - Calculates EFT amounts (`EFTAMTK = AFAMT - AFDISC`) and accumulates totals (`L2IAMTK`, `L2DAMTK`, `L2EAMTK`) at the customer level (`L2`).
     - Formats dates (e.g., `AFDATEY`, `AFDSD6Y`) for display.
   - The program iterates through non-deleted `AREFTD` records, grouping by company (`L3`) and customer (`L2`), and processes invoices (`L1`).

5. **Output Specifications**:
   - Outputs to four printer files (`LIST1`, `LIST2`, `LIST3`, `LIST4`), each with similar structures but different overflow indicators:
     - `PRTHDR` (Header):
       - Prints selection date (`SLDT8`, formatted as MM/DD/YYYY).
       - Prints company (`AFCO`) and customer number (`AFCUSTZ`).
       - Prints customer details (`ARNAME`, `ARADR1` to `ARADR3`).
       - Prints email addresses (`EIFM,2` to `EIFM,4`) from `ARCUFMX`.
       - Prints heading lines (`HD,2`, `HD,3`) from the `HD` array (e.g., “ELECTRONIC FUNDS TRANSFER NOTIFICATION”, contact information).
     - `PRTDTL` (Detail):
       - Prints transaction details: date (`AFDATEY`), selection date (`AFDSD6Y`), invoice number (`AFINV#Z`), amount (`AFAMT K`), discount (`AFDISCK`), EFT amount (`EFTAMTK`).
     - `PRTL2T` (Customer Totals):
       - Prints dashed lines for formatting.
       - Prints customer-level totals: invoice amount (`L2IAMTK`), discount (`L2DAMTK`), EFT amount (`L2EAMTK`).
   - Each printer file (`LIST1` to `LIST4`) corresponds to different output destinations or formats, possibly for multiple copies or distribution channels.

6. **Program Termination**:
   - The program processes all non-deleted `AREFTD` records using RPG’s implicit file processing cycle, generating reports for each EFT customer, and terminates when complete.

### Business Rules

The `AR137B` program enforces the following business rules:

1. **Non-Deleted Records**:
   - Only processes records in `AREFTD` where `AFDEL` is not ‘D’ (`1NCD`), ensuring valid transaction data.

2. **Company and Customer Validation**:
   - Validates company number (`AFCO`) against `ARCONT`.
   - Validates customer number (`AFCUST`) against `ARCUST` using `CUSKEY`.
   - Retrieves customer details (e.g., `ARNAME`, `ARADR1` to `ARADR3`) for report headers.

3. **Email Address Integration**:
   - Retrieves email addresses from `ARCUFMX` (stored in `EIFM` array) for EFT notice distribution.
   - Uses email address counts from `ARDTWSC` to determine the number of email addresses per customer.

4. **Amount Calculations**:
   - Calculates EFT amounts (`EFTAMTK = AFAMT - AFDISC`) for each invoice.
   - Accumulates totals at the customer level (`L2IAMTK`, `L2DAMTK`, `L2EAMTK`) for reporting.

5. **Report Formatting**:
   - Generates headers with company, customer, and selection date information.
   - Includes contact information (e.g., “Lisa at (412)826-3011”) and a static email (`mgreenberg@amref.com`, commented out).
   - Prints transaction details (date, invoice, amounts) and customer totals.
   - Uses four printer files (`LIST1` to `LIST4`) for multiple output copies or formats.

6. **Date Formatting**:
   - Formats dates (`AFDATEY`, `AFDSD6Y`, `SLDT8`) for readability in reports (e.g., MM/DD/YYYY).

### Integration with AR137B OCL Script

The `AR137B` RPG program is invoked by the `AR137B.ocl36.txt` script via `// LOAD AR136B` and `// RUN` (likely a typo for `AR137B`). The OCL script:
- Defines input files `AREFTD` (as `?9?E?L'110,6'?` and `?9?AREFTX`), `ARCONT`, `ARCUST`, `ARCUFMX`, and `ARDTWSC` (as `?9?ARDTGGC`), matching the RPG file definitions.
- Overrides printer files `LIST1` to `LIST4` to direct output to `EFTEMALOTQ` (production) or `TESTOUTQ` (test) with 12 CPI.
The `AR137B` program uses these files to generate final EFT reports, incorporating transaction data (`AREFTD`), customer details (`ARCUST`), control data (`ARCONT`), email addresses (`ARCUFMX`), and email counts (`ARDTWSC`) from prior steps (`AR137A`, `AR137E`).

### Tables (Files) Used

The program uses the following files:

1. **ARDTWSC**:
   - Type: Input (11 bytes, indexed, 8-byte key).
   - Purpose: Contains email address counts per customer, generated by `AR137E`.
   - Fields: `AFCOCU` (company + customer key), `COUNT` (email count).

2. **AREFTD**:
   - Type: Primary input (256 bytes, indexed, 15-byte key).
   - Purpose: Contains EFT transaction data (invoices, amounts, dates).
   - Fields: `AFDEL`, `AFSEQ#`, `AFCO`, `AFCUST`, `CUSKEY`, `AFINV#`, `AFAMT`, `AFDISC`, `AFTYPE`, `AFDATE`, `AFDESC`, `AFDUDT`, `AFDAT8`, `AFDUD8`, `AFDSD8`, `AFDSDT`, `AFUPDT`, `AFSLDT`.

3. **ARCUST**:
   - Type: Input (384 bytes, indexed, 8-byte key).
   - Purpose: Contains customer data for report headers.
   - Fields: `ARDEL`, `ARCO`, `ARCUST`, `ARNAME`, `ARADR1` to `ARADR4`, `AREFT`, and others.

4. **ARCONT**:
   - Type: Input (256 bytes, indexed, 2-byte key).
   - Purpose: Contains accounts receivable control data for company validation.
   - Fields: `ACDEL`, `ACCO`, `ACNAME`, `ACEFCG`, and others.

5. **ARCUFMX**:
   - Type: Input (266 bytes, indexed, 21-byte key).
   - Purpose: Contains customer format data, including email addresses for EFT notices.
   - Fields: `FMDEL`, `FMCONO`, `FMCUST`, `FMFMTY`, `FMEMLA`, and others.

6. **LIST1, LIST2, LIST3, LIST4**:
   - Type: Output (printer files, 120 bytes).
   - Purpose: Generate formatted EFT reports with headers, transaction details, and customer totals.

### External Programs Called

The `AR137B` program does not call any external programs. It operates solely on the defined files and performs its logic internally.

### Summary

The `AR137B` RPG III program generates final EFT reports for each customer, using transaction data (`AREFTD`), customer details (`ARCUST`), control data (`ARCONT`), email addresses (`ARCUFMX`), and email counts (`ARDTWSC`). It produces formatted output to four printer files (`LIST1` to `LIST4`), including headers, transaction details, and customer totals, for printing or electronic distribution. The program completes the EFT draft notice process initiated by the `AR137` OCL script.

**Tables Used**: `ARDTWSC`, `AREFTD`, `ARCUST`, `ARCONT`, `ARCUFMX`, `LIST1`, `LIST2`, `LIST3`, `LIST4`.
**External Programs Called**: None.