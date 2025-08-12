### Process Steps of the RPG Program BB1103

This RPG III program (BB1103) preprocesses order data for printing picking tickets by reading input records, determining custom grouping and sorting keys based on order characteristics, and writing extended records to an output file. It handles three record types: headers, details, and others (e.g., marks or attachments). The core logic applies business-specific sorting rules to detail lines (e.g., by product, container, size) while preserving headers first and others last. The program extends each record by 42 bytes (to 554 total) with sort indicators and keys for downstream sorting (e.g., in #GSORT as seen in calling OCL). It runs in a standard RPG cycle: read input → calculate → output → repeat until end-of-file. Below is a step-by-step breakdown:

1. **Program Initialization**:
   - Header (H) specifications define the program title and purpose.
   - File (F) specifications declare:
     - `BBORTR`: Input primary (IP) file (512 bytes, sequential read).
     - `GSCNTR`: Input keyed (IF) file (512 bytes, key length 3, alternate index 2).
     - `BB1103`: Output (O) file (554 bytes, add mode for sequential writes).
   - No explicit *INZSR; defaults to standard RPG init.

2. **Input Record Identification**:
   - Input (I) specifications define non-sequential (NS) formats for three record types from `BBORTR`:
     - 01 (Header): Identified by indicators 10, C0 (company?), 11 C0, 12 C0. Maps fields like BOCO (company), BORDNO (order#), BORSEQ (sequence#), BOCOON ('Y' for customer-owned), BOGPBY (group by, e.g., 'C' for category), BORACD (responsible area, e.g., 'PP'), BOMLCD (major location, e.g., 'PKGP'), BOORPR (process status).
     - 02 (Detail): Identified by 10NC9 (non-company?). Maps similar header fields plus BDLOC (location), BDPROD (product), BDCNTR (container), BDFLCD (flag code).
     - 03 (Other): Catch-all for remaining records.
   - Additional input for `GSCNTR` (NS) maps TCDSCS (short description for size).

3. **Calculation Logic (Main Processing)**:
   - For all records: Clears work fields (SIZE, GROUP, SORT1, SORT2, SORT3 to blanks).
   - Header-specific (01):
     - Sets indicator 50 off if BOORPR is blank (prints in line entry order); on otherwise (sorted order for 'C'/'S' codes).
   - Detail-specific (02):
     - Retrieves SIZE: If BDFLCD != 'N', chains to GSCNTR by BDCNTR (indicator 88 on failure); sets SIZE to TCDSCS (short desc) if found, else blank.
     - Determines GROUP/SORT keys based on nested conditions (see Business Rules for details):
       - If customer-owned (BOCOON='Y'): Group by product or size/container/product.
       - Else if ARG PKG PLANT (BORACD='PP' and BOMLCD='PKGP'): Similar, product or size/container/product.
       - Else (ARG all other): Product or container/product/container.
     - No processing for other records (03).
   - The program cycles through all `BBORTR` records automatically via LR (last record) indicator.

4. **Output to Extended File**:
   - Output (O) specifications write to `BB1103` in detail add (DADD) mode:
     - For headers (01): Copies original 512 bytes (DTA1/DTA2), adds '1' (record type) at pos 513, blanks in sort fields (523-553), BOORPR at 554.
     - For details (02): Copies 512 bytes, adds '2' at 513. Conditionally (based on 50 off/on):
       - If unsorted (N50): '0' at 524, BORKEY (order key) at 533.
       - If sorted (50): GROUP at 523, SORT1 at 533, SORT2 at 543, SORT3 at 553.
       - Adds BOORPR at 554.
     - For others (03): Copies 512 bytes, adds '3' at 513, blanks in sort fields, BOORPR at 554.
   - Ensures output records are extended for sorting (e.g., headers first via '1', details '2', others '3').

5. **Program Termination**:
   - Ends when `BBORTR` EOF reached (LR on implicitly).
   - No explicit cleanup; files close automatically.

The program is called from OCL (e.g., BB111) to prepare sorted data for picking ticket printing, extending records for custom sorts in subsequent steps.

### Business Rules

- **Record Typing and Prioritization**: Forces output order: Headers ('1' marker) > Details ('2') > Others ('3'). This ensures coherent picking tickets (headers first, details grouped, attachments last).
- **Sorting Modes**:
  - If BOORPR blank: Unsorted (indicator 50 off; prints in entry sequence, uses '0' and BORKEY for minimal sort).
  - If BOORPR non-blank (e.g., 'C' for confirmation, 'S' for something else): Sorted (50 on; applies custom GROUP/SORT1-3).
- **Detail Grouping/Sorting Logic** (Nested by Order Type):
  - Customer-Owned Only (BOCOON='Y', e.g., for viscosity EDI856):
    - By Category (BOGPBY='C'): GROUP/SORT1=Product, SORT2=Container, SORT3=Blank.
    - By Size: GROUP=Size Desc, SORT1=Blank, SORT2=Container, SORT3=Product.
  - ARG PKG Plant (BORACD='PP' and BOMLCD='PKGP'):
    - By Category: GROUP/SORT1=Product, SORT2=Container, SORT3=Blank.
    - By Size: GROUP=Size Desc, SORT1=Blank, SORT2=Container, SORT3=Product.
  - ARG All Other:
    - By Category: GROUP/SORT1=Product, SORT2=Blank, SORT3=Container.
    - By Size: GROUP/SORT1=Container, SORT2=Blank, SORT3=Product.
  - SIZE derived from GSCNTR short desc if BDFLCD!='N' (non-new?); blank otherwise.
- **Data Integrity**: Chains fail gracefully (88 on, sets blanks). No updates; read-only input, additive output.
- **Purpose Alignment**: Prepares for picking tickets with efficient warehouse flow (e.g., group by category for similar items, size for bin optimization). Assumes input from prior selection (e.g., BB112/BB111).

### Tables Used

- `BBORTR`: Input primary (IP) file (sequential read, 512 bytes). Source of order records (headers, details, others).
- `GSCNTR`: Input keyed (IF) file (512 bytes, key 3 bytes). Used for chaining to get container size descriptions (TCDSCS).
- `BB1103`: Output (O) file (554 bytes, add mode). Extended output with sort keys.

No database updates; all are disk files.

### External Programs Called

None. The program contains no CALL opcodes or external subroutine references (EXSR to external programs). It operates standalone.