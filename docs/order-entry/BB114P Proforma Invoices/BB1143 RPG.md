### Process Steps of the RPG Program BB1143

This RPG III program (indicated by the .rpg extension and syntax) is a preprocessing utility for printing picking tickets, specifically designed to prepare and sort order detail records for the main printing program (e.g., BB110). It is called from the OCL program BB114O and processes order data to assign sorting fields based on specific criteria, such as customer-owned products, packaging plant specifics, or general conditions. The program reads input records, applies business logic to determine sorting fields, and writes extended records to an output file for subsequent sorting (by #GSORT in BB114O). Below is a step-by-step breakdown of the program's execution:

1. **File and Record Initialization**:
   - Defines input files: BBORTR (primary input, 512 bytes, disk) and GSCNTR (secondary input for country/container data, 512 bytes, keyed with 3-byte alpha index).
   - Defines output file: BB1143 (554 bytes, disk, add-only).
   - Specifies input formats for BBORTR:
     - NS 01: Header records (indicators 10, 11, 12 on; fields like BOCO, BORDNO, BORSEQ, BOCOON, BOGPBY, BORACD, BOMLCD, BOORPR).
     - NS 02: Detail records (indicator 10 off, 9 off; fields like BOCO, BORDNO, BORSEQ, BORKEY, BDLOC, BDPROD, BDCNTR, BDFLCD).
     - NS 03: Other record types (catch-all for non-header/detail).
     - GSCNTR: Reads short description (TCDSCS) for container codes.

2. **Header Record Processing (Indicator 01)**:
   - For header records, evaluates order process status (BOORPR):
     - If BOORPR ≠ ' ', sets indicator 50 on (indicating line entry order printing).
     - Otherwise, sets indicator 50 off (sorted order for codes 'C' or 'S').
   - Writes to BB1143 (DADD 01):
     - Outputs DTA1 (pos 1-256) and DTA2 (pos 257-512) as-is.
     - Adds fixed fields: '1' at pos 513 (record type), blanks at 523-553 (sort fields), BOORPR at 554.

3. **Detail Record Processing (Indicator 02)**:
   - Clears sort fields (SIZE, GROUP, SORT1, SORT2, SORT3) to blanks.
   - Determines SIZE (container size description):
     - If BDFLCD ≠ 'N', chains BDCNTR to GSCNTR to get TCDSCS (short description) into SIZE (7 chars).
     - If chain fails (*IN88 on) or BDFLCD = 'N', sets SIZE to blanks.
   - Applies sorting logic based on conditions:
     - **Customer-Owned Product (BOCOON = 'Y')**:
       - If BOGPBY = 'C' (by category): GROUP/SORT1 = BDPROD (product), SORT2 = BDCNTR (container), SORT3 = blank.
       - Else (by size): GROUP = SIZE, SORT1 = blank, SORT2 = BDCNTR, SORT3 = BDPROD.
     - **ARG Packaging Plant (BORACD = 'PP' and BOMLCD = 'PKGP')**:
       - If BOGPBY = 'C': GROUP/SORT1 = BDPROD, SORT2 = BDCNTR, SORT3 = blank.
       - Else: GROUP = SIZE, SORT1 = blank, SORT2 = BDCNTR, SORT3 = BDPROD.
     - **ARG - All Other**:
       - If BOGPBY = 'C': GROUP/SORT1 = BDPROD, SORT2 = blank, SORT3 = BDCNTR.
       - Else: GROUP/SORT1 = BDCNTR, SORT2 = blank, SORT3 = BDPROD.
   - Writes to BB1143 (DADD 02):
     - Outputs DTA1 (1-256), DTA2 (257-512).
     - Sets pos 513 to '2' (record type).
     - If *IN50 off (sorted order): Outputs GROUP (523-532), SORT1 (533-542), SORT2 (543-552), SORT3 (553) as sort fields.
     - If *IN50 on (line entry order): Outputs '0' at 524, BORKEY (order key) at 533, blanks elsewhere.
     - Outputs BOORPR at 554.

4. **Other Record Types (Indicator 03)**:
   - Writes to BB1143 (DADD 03):
     - Outputs DTA1 (1-256), DTA2 (257-512).
     - Sets pos 513 to '3' (record type), blanks at 523-553, BOORPR at 554.

5. **Program Flow**:
   - Processes all records from BBORTR sequentially.
   - For each record, applies header, detail, or other logic based on indicators.
   - Outputs extended records (554 bytes) to BB1143, adding sort fields for details or placeholders for headers/others.
   - No explicit loop or termination logic; RPG cycle handles record-by-record processing until end-of-file.

### Business Rules

- **Sorting Customization**: Sorting fields (GROUP, SORT1, SORT2, SORT3) are assigned based on:
  - **Customer-Owned Products (e.g., Viscosity EDI856)**: Prioritizes product (by category) or size/container (by size), ensuring customer-specific sorting for picking tickets.
  - **ARG Packaging Plant (PP/PKGP)**: Similar to customer-owned, sorts by product or size/container, reflecting plant-specific grouping.
  - **ARG - All Other**: Sorts by product or container, with broader grouping for non-specialized cases.
- **Order Process Status**: BOORPR determines sort mode:
  - Codes 'C' or 'S' (e.g., confirmed/shipped) use custom sorting (*IN50 off).
  - Blank (' ') uses line entry order (*IN50 on), preserving original sequence (BORKEY).
- **Container Size Lookup**: Uses GSCNTR to fetch container descriptions (TCDSCS) unless BDFLCD = 'N' (no lookup needed).
- **Record Type Handling**: Preserves header (type 1), detail (type 2), and other (type 3) records, ensuring headers are not sorted out of sequence in downstream processing (e.g., by #GSORT).
- **Data Integrity**: Passes original data (DTA1, DTA2) unchanged, adding sort fields (513-554) for flexibility in subsequent sorting (by BB114O).
- **Use Case**: Prepares data for picking ticket printing, ensuring logical grouping (e.g., by product or container size) for warehouse efficiency, tailored to customer or plant requirements.

### Tables/Files Used

- **BBORTR**: Input file (512 bytes, disk); contains order records (header, detail, others) with fields like BOCO, BORDNO, BORSEQ, BOCOON, BOGPBY, BORACD, BOMLCD, BOORPR, BDLOC, BDPROD, BDCNTR, BDFLCD.
- **GSCNTR**: Input file (512 bytes, keyed with 3-byte alpha index); used for container short descriptions (TCDSCS).
- **BB1143**: Output file (554 bytes, disk, add-only); stores extended records with sort fields and BOORPR.

### External Programs Called

- None: The program does not call external programs (no CALL opcodes). It is a standalone preprocessor that relies on the RPG cycle for processing and writes to BB1143 for use by downstream programs (e.g., #GSORT, BB110).