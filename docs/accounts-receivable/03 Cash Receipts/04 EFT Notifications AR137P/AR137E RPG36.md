The `AR137E.rpg36.txt` is an RPG III program used in IBM i (AS/400) systems, invoked by the `AR137.ocl36.txt` OCL script as part of the EFT (Electronic Funds Transfer) draft notice filtering process. This program counts the number of email addresses assigned to each EFT customer in the `AREFTS` file, using customer format data from `ARCUFMX`, and writes the count to the `ARDTWSC` file. Below, I’ll explain the process steps, business rules, tables (files) used, and external programs called, providing a clear and concise analysis.

### Process Steps of the AR137E RPG Program

The `AR137E` program processes EFT summary records, checks for valid email addresses in the customer format file, and records the count of email addresses per customer for EFT draft notices. Here’s a step-by-step breakdown of its execution:

1. **Program Header**:
   - `H P064 B AR137E`: Specifies the program identifier (`P064`), batch mode (`B`), and program name (`AR137E`).
   - Indicates the program is designed for batch processing to count email addresses for EFT customers.

2. **File Definitions**:
   - `FAREFTS IP F 100 100 DISK`: Defines `AREFTS` as the primary input file (100 bytes, fixed length).
   - `FARCUFMX IF F 266 266 L21AI EXTK DISK`: Defines `ARCUFMX` as an input file (266 bytes, indexed with a 21-byte key, externally defined key).
   - `FARDTWSC UF F 11 11 8AI 1 DISK A`: Defines `ARDTWSC` as an update-capable output file (11 bytes, indexed with an 8-byte key, append mode `A`).

3. **Input Specifications**:
   - `IAREFTS NS 01 1NCD`:
     - Defines the `AREFTS` record layout for non-deleted records (`1NCD`).
     - Fields include:
       - `AFDEL` (position 1, delete flag).
       - `AFCO` (positions 2-3, company number).
       - `AFCUST` (positions 4-9, customer number).
       - `AFCOCU` (positions 2-9, composite key: company + customer).
       - `AFDESC` (positions 10-39, description).
       - `AFAMT` (positions 40-44, packed, amount).
       - `AFDISC` (positions 45-49, packed, discount).
       - `AFEAMT` (positions 50-54, packed, EFT amount).
       - `AFDAT8` (positions 55-62, 8-digit date).
       - `AFUPD8` (positions 63-70, 8-digit update date).
   - `IARCUFMX NS`:
     - Defines the `ARCUFMX` record layout:
       - `FMDEL` (position 1, delete code).
       - `FMCONO` (positions 2-3, company number).
       - `FMCUST` (positions 4-9, customer number).
       - `FMFMTY` (positions 10-13, form type).
       - `FMSEQ#` (positions 14-22, sequence number).
       - `FMCNTC` (positions 23-72, contact name).
       - `FMEMLA` (positions 73-132, email address).
       - `FMFAX#` (positions 133-152, fax number).
       - `FMFMYN` (position 153, send original flag).
       - `FMRPYN` (position 154, send reprint flag).
       - `FMMLYN` (position 155, send by mail flag).
       - `FMBKYN` (position 155, send terms on back flag).
   - `IARDTWSC NS`: Defines the `ARDTWSC` record layout (not explicitly detailed but includes `AFCOCU` and `COUNT`).

4. **Calculation Logic**:
   - **Main Loop (DO for `01` Indicator)**:
     - Processes each non-deleted record in `AREFTS` (`01` indicator, `1NCD`).
     - Initializes:
       - `A` (counter) to 1.
       - `FMKEY` (21-byte key) to blanks.
       - Constructs `FMKY12` (12 bytes) by combining `AFCO` (company) and `FMKY10` (10 bytes, customer + ‘EFT’).
       - Sets `FMKEY` to `FMKY12` for indexing `ARCUFMX`.
     - Positions the `ARCUFMX` file at the key using `SETLL`.
   - **Read ARCUMFX Loop (AGNFM)**:
     - Reads `ARCUFMX` records (`READ`, indicator `08`).
     - If end of file (`08` on), jumps to `SKIP1`.
     - If counter `A` exceeds 4, jumps to `SKIP1` (limits processing to 4 email addresses per customer).
     - Validates:
       - If `FMFMTY` is not ‘EFT’, jumps to `SKIP1`.
       - If `AFCO` does not match `FMCONO`, jumps to `SKIP1`.
       - If `AFCUST` does not match `FMCUST`, jumps to `SKIP1`.
       - If `FMDEL` is ‘D’, loops back to `AGNFM` (skips deleted records).
     - If `FMEMLA` (email address) is not blank and `FMFMYN` (send original) is ‘Y’, increments `COUNT` (3 digits).
     - If `A` is 2, 3, or 4, increments `COUNT` (likely for multiple email addresses or formats).
     - Loops back to `AGNFM` to read the next `ARCUFMX` record.
   - **Output to ARDTWSC**:
     - Chains to `ARDTWSC` using `AFCOCU` (company + customer key, indicator `98`).
     - Ends the `DO` loop for the current `AREFTS` record.

5. **Output Specification**:
   - `OARDTWSC DADD 01 98`:
     - If `98` is on (record found in `ARDTWSC`), updates the record with:
       - `AFCOCU` (positions 1-8, company + customer key).
       - `COUNT` (positions 9-11, packed, email address count).
   - `O D 01N98`:
     - If `98` is off (no record found), adds a new record to `ARDTWSC` with:
       - `COUNT` (positions 9-11, packed, email address count).

6. **Program Termination**:
   - The program processes all non-deleted records in `AREFTS` using RPG’s implicit file processing cycle, writing or updating records in `ARDTWSC` with email address counts, and terminates when complete.

### Business Rules

The `AR137E` program enforces the following business rules:

1. **Non-Deleted Records**:
   - Only processes non-deleted records in `AREFTS` (`1NCD`) to ensure valid EFT summary data.

2. **Email Address Counting**:
   - Counts email addresses in `ARCUFMX` for each customer where:
     - `FMFMTY` is ‘EFT’ (specific to EFT forms).
     - `FMCONO` matches `AFCO` (company number).
     - `FMCUST` matches `AFCUST` (customer number).
     - `FMDEL` is not ‘D’ (not deleted).
     - `FMEMLA` is not blank and `FMFMYN` is ‘Y’ (valid email for sending original EFT drafts).
   - Limits counting to a maximum of 4 email addresses per customer (`A` ≤ 4).

3. **Counter Logic**:
   - Increments `COUNT` for each valid email address (`FMEMLA` not blank, `FMFMYN = 'Y'`).
   - Additional increments for `A` = 2, 3, or 4, possibly accounting for multiple email formats or addresses.

4. **Output to ARDTWSC**:
   - Writes or updates a record in `ARDTWSC` for each customer (`AFCOCU`) with the count of valid email addresses (`COUNT`).
   - Uses the company + customer key (`AFCOCU`) to locate or create records.

### Integration with AR137 OCL Script

The `AR137E` RPG program is invoked by the `AR137.ocl36.txt` script via `// LOAD AR137E` and `// RUN`, following the `AR137A` program. The OCL script:
- Defines `AREFTS` (input, EFT summary data from `AR137A`), `ARCUFMX` (input, customer format data), and `ARDTWSC` (output, temporary file for email counts).
- Uses the `?9?` library prefix and a temporary retention for `ARDTWSC` (50 records).
The `AR137E` program processes the `AREFTS` file generated by `AR137A`, counts email addresses per customer using `ARCUFMX`, and stores the results in `ARDTWSC` for use by subsequent programs (e.g., `AR137B` for final EFT reporting).

### Tables (Files) Used

The program uses the following files:

1. **AREFTS**:
   - Type: Primary input (100 bytes, fixed length).
   - Purpose: Contains EFT summary data from `AR137A`.
   - Fields: `AFDEL`, `AFCO`, `AFCUST`, `AFCOCU`, `AFDESC`, `AFAMT`, `AFDISC`, `AFEAMT`, `AFDAT8`, `AFUPD8`.

2. **ARCUFMX**:
   - Type: Input (266 bytes, indexed, 21-byte key).
   - Purpose: Contains customer format data, including email addresses for EFT drafts.
   - Fields: `FMDEL`, `FMCONO`, `FMCUST`, `FMFMTY`, `FMSEQ#`, `FMCNTC`, `FMEMLA`, `FMFAX#`, `FMFMYN`, `FMRPYN`, `FMMLYN`, `FMBKYN`.

3. **ARDTWSC**:
   - Type: Update-capable output (11 bytes, indexed, 8-byte key, append mode).
   - Purpose: Stores the count of email addresses per customer for EFT draft notices.
   - Fields: `AFCOCU` (company + customer key), `COUNT` (email address count).

### External Programs Called

The `AR137E` program does not call any external programs. It operates solely on the defined files and performs its logic internally.

### Summary

The `AR137E` RPG III program counts the number of valid email addresses for each EFT customer in `AREFTS`, using `ARCUFMX` to identify email addresses marked for sending original EFT drafts. It writes or updates the count in `ARDTWSC`, ensuring only non-deleted records with matching company and customer numbers are processed, with a limit of 4 email addresses per customer. The output supports the EFT draft notice process by providing email address counts for reporting.

**Tables Used**: `AREFTS`, `ARCUFMX`, `ARDTWSC`.
**External Programs Called**: None.