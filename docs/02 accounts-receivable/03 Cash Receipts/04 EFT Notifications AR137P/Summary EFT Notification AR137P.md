You're absolutely right, and I apologize for the confusion in my previous response—I did indeed analyze `AR137P.ocl36.txt` and `AR137P.rpgle.txt` (the RPGLE version) as the initial entry points when you first provided them. To clarify, here's a quick recap of how they fit into the full call stack based on our conversation:

### Complete AR137 Call Stack (Including All Provided Programs)
1. **AR137P.ocl36.txt** (OCL script): Initializes the EFT email selection process, calls `GSGENIEC` and `GSY2K`, loads and runs `AR137P` RPGLE with files `ARCONT` and `ARCUST`, conditionally calls `AR137` OCL based on status.
2. **AR137P.rpgle.txt** (RPGLE program): Validates inputs like company number (`KYCO`), selection type (`KYALSL`), customer numbers (`KYCS01`-`KYCS10`), and update date (`KYUPDT`) via a workstation screen; checks file existence via `AR135TC`; sets LDA status to 'Y' on success.
3. **AR137.ocl36.txt** (OCL script): Main orchestration for EFT filtering; calls `AR137X`, sorts/filters via `#GSORT`, processes summaries via `AR137A`, counts emails via `AR137E`, generates reports via `AR137B`, and conditionally calls `AR137TC`.
4. **AR137X.rpg36.txt** (RPG III): Flags deleted records (`ATDEL = 'D'`) in `CRTRAN` for reorganization.
5. **AR137A.ocl36.txt** (OCL script): Clears `AREFTS`, loads and runs `AR137A` RPG with transaction, control, and customer files.
6. **AR137A.rpg36.txt** (RPG III): Aggregates EFT transaction amounts by customer into summaries in `AREFTS`.
7. **AR137E.rpg36.txt** (RPG III): Counts valid email addresses per customer in `ARCUFMX` and writes to `ARDTWSC`.
8. **AR137B.ocl36.txt** (OCL script): Overrides printer files, loads and runs `AR137B` (noted as `AR136B` likely a typo).
9. **AR137B.rpg36.txt** (RPG III): Generates formatted EFT reports to printer files `LIST1`-`LIST4`.
10. **AR137TC.clp.txt** (CL program): Splits reports, converts to PDFs, distributes via email, and notifies admins.

These form the complete EFT draft notice workflow, starting with validation in `AR137P` and ending with distribution in `AR137TC`. My earlier omission was an error in explicitly listing them in the revised document—I've corrected that in the updated function requirement below, which now fully incorporates `AR137P.ocl36.txt` and `AR137P.rpgle.txt` (treating the RPGLE as the core validation logic).

### Use Cases Implemented by the AR137 Program Call Stack
The full stack implements a single primary use case:

1. **Generate and Distribute EFT Draft Notices**:
   - Validates user inputs for company, customers, and date; filters EFT transactions; generates summaries and reports; counts emails; produces PDFs; and distributes via email.

### Updated Function Requirement Document: Generate and Distribute EFT Draft Notices

# Function Requirement: Generate and Distribute EFT Draft Notices

## Purpose
Automate validation of inputs, filtering of EFT transactions for selected or all customers, generation of summaries and reports, and distribution of PDF notices via email to customers and administrators.

## Inputs
- **Local Data Area (LDA)** (validated/set by `AR137P`):
  - `KYCO` (2 bytes): Company number.
  - `KYALSL` (3 bytes): Selection type (‘ALL' or ‘SEL').
  - `KYCS01` to `KYCS10` (6 bytes each): Up to 10 customer numbers (for ‘SEL').
  - `KYUPDT` (6 bytes): Update date (YYMMDD).
  - `STATUS` (1 byte): Validation result (‘Y' on success).
- **Files**:
  - `CRTRAN`: Transaction file with EFT data.
  - `ARCONT`: Accounts receivable control data.
  - `ARCUST`: Customer data.
  - `ARCUFMX`: Customer format data (email addresses).
- **Environment Parameter**: Library prefix (`?9?`, e.g., ‘G' for production).

## Outputs
- PDF EFT draft notices emailed to customers (via `ARCUFMX` email addresses).
- Notification email to administrators (`BZOLKOS@AMREF.COM`, `LMIDLA@AMREF.COM`).
- Temporary files: `AR137S`, `AREFTD`, `AREFTS`, `ARDTWSC`.
- Spool files in output queues: `EFTEMALOTQ`, `EFTSPLITQ`, `EFTEMALQSV`, `EFTSPLTQSV`.

## Process Steps
1. **Validate Inputs (AR137P.ocl36.txt, AR137P.rpgle.txt)**:
   - Initialize with `GSGENIEC` and `GSY2K`; load/run `AR137P` RPGLE with `ARCONT`/`ARCUST`.
   - Validate `KYCO` (exists in `ARCONT`), `KYALSL` (‘ALL' defaults, ‘SEL' requires at least one customer), `KYCS01`-`KYCS10` (exist in `ARCUST` for ‘SEL'), `KYUPDT` (non-zero), and file existence (via `AR135TC` for ‘GE' + `KYUPDT`).
   - Set `STATUS = 'Y'` on success; clear LDA on failure.
2. **Orchestrate Processing (AR137.ocl36.txt)**:
   - Call `AR137X` to clean `CRTRAN`; use `#GSORT` to filter/copy to `AR137S`/`AREFTD` based on `KYALSL`.
3. **Clean Transaction File (AR137X.rpg36.txt)**:
   - Flag deleted records (`ATDEL = 'D'`) in `CRTRAN` for `RGZPFM` reorganization.
4. **Generate EFT Summary (AR137A.ocl36.txt, AR137A.rpg36.txt)**:
   - Clear `AREFTS`; process `AREFTD` records, validate via `ARCONT`/`ARCUST`.
   - Calculate customer totals: EFT Amount = `AFAMT` - `AFDISC`; accumulate `L2IAMT` (invoices), `L2DAMT` (discounts), `L2EAMT` (EFT).
   - Write summaries to `AREFTS` (company, customer, totals, dates).
5. **Count Email Addresses (AR137E.rpg36.txt)**:
   - For each `AREFTS` record, scan `ARCUFMX` (up to 4 addresses) where `FMFMTY = 'EFT'`, `FMDEL ≠ 'D'`, `FMEMLA` non-blank, `FMFMYN = 'Y'`.
   - Increment count per valid address; write to `ARDTWSC` (key: company + customer).
6. **Generate EFT Reports (AR137B.ocl36.txt, AR137B.rpg36.txt)**:
   - Override printers to `EFTEMALOTQ`/`TESTOUTQ`; process `AREFTD` with `ARCONT`/`ARCUST`/`ARCUFMX`/`ARDTWSC`.
   - Format reports (`LIST1`-`LIST4`): headers (customer details, emails), details (invoices, `AFAMT`/`AFDISC`/EFT), totals (`L2IAMT`/`L2DAMT`/`L2EAMT`).
7. **Distribute PDFs and Notify (AR137TC.clp.txt)**:
   - Split spools (`SFASPLIT` for `EFT EMAIL 1`-`4`); convert to PDFs (`FFAEDOC` as `ARGAREFTSTMT`); distribute (`SFARDST`); move to save queues (`SFAMOVE`).
   - Clear queues; email admins (`SFAEML`).

## Business Rules
1. **Input Validation**:
   - `KYCO` must exist in `ARCONT`; for ‘SEL', customers must exist in `ARCUST`; `KYUPDT` non-zero; file (‘GE' + `KYUPDT`) exists; ‘ALL' cannot have customers, ‘SEL' requires at least one.
   - Terminate if invalid (no LDA status set).
2. **Transaction Filtering**:
   - Non-deleted only (`NECD/AFDEL ≠ 'D'`); match `KYCO`/`KYCSxx` for ‘SEL'.
3. **Calculations**:
   - EFT Amount = `AFAMT` - `AFDISC` (per invoice); aggregate at customer level (`L2IAMT` = sum `AFAMT`, etc.).
   - Email count: Increment for each valid `ARCUFMX` entry (max 4/customer).
4. **Email Distribution**:
   - Use `ARCUFMX` emails; limit 4/customer; require `FMFMYN = 'Y'`.
5. **Report Formatting**:
   - Dates as MM/DD/YYYY; include customer addresses/emails; four reports for copies/channels.
6. **Environment Handling**:
   - `?9? = 'G'`: Production queues; else test.
7. **Cleanup**:
   - Delete temps (`AR137S`); clear queues/files post-process.
8. **Notification**:
   - Email admins from `MGREENBERG@AMREF.COM` on completion.

## Error Handling
- Skip invalid records (company/customer); monitor SpoolFlex (`SF00136`) and continue; no processing if `STATUS ≠ 'Y'`.


## Table: AR137 Program Call Stack

| **Program** | **Main Purpose** | **Tables (Files) Used** | **Outputs (Files or Side Effects)** |
|-------------|------------------|-------------------------|-------------------------------------|
| **AR137.ocl36.txt** | Orchestrates the EFT draft notice process by filtering transactions, generating reports, and cleaning up files. | CRTRAN, AR137S, AREFTD, AREFTS, ARCONT, ARCUST, ARCUFMX, ARDTWSC | Temporary files (AR137S, AREFTD, AREFTS, ARDTWSC), cleared output queues (EFTEMALOTQ, EFTSPLITQ), calls to AR137X, AR137A, AR137E, AR137B, AR137TC. |
| **AR137X.rpg36.txt** | Removes deleted records from the transaction file to prepare for EFT processing. | CRTRAN | Updated CRTRAN file (deleted records flagged for reorganization). |
| **AR137A.rpg36.txt** | Generates EFT summary data by aggregating transaction amounts by customer. | AREFTD, ARCUST, ARCONT, AREFTS | AREFTS (summary records with customer totals), calls to printer files indirectly via subsequent programs. |
| **AR137E.rpg36.txt** | Counts valid email addresses per customer for EFT draft distribution. | AREFTS, ARCUFMX, ARDTWSC | ARDTWSC (email address counts per customer). |
| **AR137B.rpg36.txt** | Generates formatted EFT reports for each customer, including transaction details and totals. | ARDTWSC, AREFTD, ARCUST, ARCONT, ARCUFMX, LIST1, LIST2, LIST3, LIST4 | Spool files in EFTEMALOTQ or TESTOUTQ (reports via LIST1 to LIST4). |
| **AR137TC.clp.txt** | Splits EFT reports into PDFs, distributes them via email, and notifies administrators. | None (uses output queues: EFTEMALOTQ, EFTSPLITQ, EFTEMALQSV, EFTSPLTQSV) | PDFs (ARGAREFTSTMT) emailed to customers, moved spool files (EFTEMALQSV, EFTSPLTQSV), cleared queues (EFTEMALOTQ, EFTSPLITQ), email to administrators. |

### Notes:
- **Order of Execution**: The programs are called sequentially by `AR137.ocl36.txt`: `AR137X` → `AR137A` → `AR137E` → `AR137B` → `AR137TC` (conditional on `?9? = 'G'`).
- **Tables Used**: Refers to database files or output queues (for `AR137TC`). Printer files (LIST1 to LIST4) are treated as output files.
- **Outputs/Side Effects**: Includes generated files, updated files, spool files, emails, and queue modifications.
- The `AR137B.ocl36.txt` script references `AR136B`, likely a typo for `AR137B`, as the RPG program provided is `AR137B.rpg36.txt`.
