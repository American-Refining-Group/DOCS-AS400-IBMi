### Process Steps of the RPG Program BB112S

This RPGLE program (BB112S) is a utility for sending customer order confirmations by populating temporary work files with data from a specific order. It reads records for a single order (passed via parameters) from the order details file and segregates them into type-specific output files (e.g., header, details, miscellaneous charges) for downstream processing (e.g., printing in BB110). The program uses keyed access for efficient reading and additive output to build the work files. It runs in a standard RPG cycle with subroutines for initialization and building. Below is a step-by-step breakdown of the execution flow based on the code structure (F, I, C, O specifications):

1. **Program Initialization (*INZSR Subroutine)**:
   - Receives entry parameters: `Pco` (company number, decimal 2.0) and `Pordn` (order number, decimal 6.0).
   - Constructs the key `KyCoOrd` by combining `Pco` and `Pordn` (e.g., company + order# as an 8-digit key).
   - No other setup; files are opened implicitly on first access.

2. **Main Processing**:
   - Executes the `BUILD` subroutine to process the order.
   - Sets on the Last Record (LR) indicator to terminate the program after processing.

3. **Build Subroutine (BUILD SR)**:
   - Constructs a starting key `KyCoOrdSq` by appending '000' to `KyCoOrd` (positions sequence number 000 for headers).
   - Positions the file pointer (SETLL) on `BBORDR` using `KyCoOrdSq` (starts at the beginning of the order's records).
   - Enters a loop (DOU *IN99 = *ON) to read all records for the order:
     - Reads the next record (READE) from `BBORDR` using `KyCoOrd` (keyed by company + order#, reads sequentially until key changes).
     - If not end-of-file (*IN99 off):
       - Selects based on sequence number (`SEQ#` or `BORSEQ`):
         - If 000: Writes header record (EXCEPT Header) to `BBOCFH`.
         - If 900-959: Writes other miscellaneous record (EXCEPT OtherMisc) to `BBOCFM`.
         - If 960: Writes miscellaneous record (EXCEPT Misc960) to `BBOCFO`.
         - If 961: Writes miscellaneous record (EXCEPT Misc961) to `BBOCFI`.
         - If 962: Writes miscellaneous record (EXCEPT Misc962) to `BBOCFB`.
         - Otherwise (details): Writes detail record (EXCEPT Detail) to `BBOCFD`.
   - Loops until no more records for the order (key mismatch triggers EOF).

4. **Output Writing**:
   - Output (O) specifications use EXCEPT to add records (EADD) to the respective files:
     - Copies data from input buffers (e.g., BOHDR1/BOHDR2 for headers, BDDTL1/BDDTL2 for details) directly to output (positions 1-256 and 257-512).
     - Adds spacing (/SPACE 1) between records for some outputs (e.g., headers, details, Misc960).

5. **Program Termination**:
   - Ends via LR indicator after the single order is processed. Files close implicitly.

The program is called once per order (from BB112SC), processes sequentially, and assumes records are pre-sorted by sequence number in BBORDR.

### Business Rules

- **Single-Order Processing**: Handles one order per invocation (keyed by company + order#), ensuring targeted confirmation generation without batching multiple orders.
- **Record Segregation**: Classifies records by sequence number (`BORSEQ`):
  - 000: Header (e.g., order info, dates, terms).
  - 001-899/963+: Details (line items with product, qty, etc.).
  - 900-959: General miscellaneous (e.g., charges, notes).
  - 960: Specific misc (e.g., order-related).
  - 961: Specific misc (e.g., invoice-related).
  - 962: Specific misc (e.g., BOL-related).
  This segregation prepares data for type-specific printing/processing in confirmations.
- **Data Copying**: Direct position-based copy from input to output (no transformations/calculations), preserving original data. Deleted records (BODEL='D') are included if present, but downstream programs (e.g., BB110) may filter them.
- **Key Usage**: Key starts at position 2 in BBORDR (company + order# + seq#), allowing efficient positioning and reading of all seq# for an order.
- **Work File Population**: Builds additive work files (BBOCF*) for temporary use in confirmation workflows (e.g., sorted/printed in BB111/BB110). Assumes files are cleared beforehand (e.g., by CLRPFM in calling OCL/BB112).
- **Assumptions**: Relies on valid input parameters (no validation); records exist in BBORDR. No error handling beyond EOF indicators.

### Tables Used

- `BBORDR`: Input keyed (IF) disk file (512 bytes, key location 2: company + order# + seq#). Primary source for order records (headers, details, misc).
- `BBOCFH`: Output additive (O A F) disk file (512 bytes). For header records.
- `BBOCFD`: Output additive (O A F) disk file (512 bytes). For detail records.
- `BBOCFO`: Output additive (O A F) disk file (512 bytes). For misc records (seq 960).
- `BBOCFI`: Output additive (O A F) disk file (512 bytes). For misc records (seq 961).
- `BBOCFB`: Output additive (O A F) disk file (512 bytes). For misc records (seq 962).
- `BBOCFM`: Output additive (O A F) disk file (512 bytes). For other misc records (seq 900-959).

All outputs are direct copies; no updates to input file.

### External Programs Called

None. The program contains no CALL opcodes, external subroutine references (EXSR to external), or APIs. It operates standalone.