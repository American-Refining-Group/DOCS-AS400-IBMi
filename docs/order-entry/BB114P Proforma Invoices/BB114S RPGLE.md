### Process Steps of the RPGLE Program BB114S

This RPG IV (RPGLE) program, named BB114S, is part of the Customer Orders system and is designed to build work files for customer order confirmations. It is called from the CLP program BB114SC (which itself is invoked from the main OCL via BB114 for subfile option 6). The program reads order records from the BBORDR file and populates multiple output work files (BBPROH, BBPROD, BBPROO, BBPROI, BBPROB, BBPROM) based on record types identified by sequence numbers (Seq#). These files are used downstream for printing confirmations or proforma invoices (e.g., by BB110). Below is a step-by-step breakdown of the program's execution:

1. **Program Initialization (*INZSR Subroutine)**:
   - Receives two input parameters: Pco (company number, 2-digit decimal) and Pordn (order number, 6-digit decimal).
   - Constructs a key (KyCoOrd, 8-digit numeric) by combining Pco and Pordn for reading BBORDR.
   - Initializes KyCoOrdSq (11 characters) by moving KyCoOrd and appending '000' to start at the header record.

2. **Main Processing (BUILD Subroutine)**:
   - Sets the lower limit (SETLL) on BBORDR using KyCoOrdSq to position at the first record for the specified company and order.
   - Enters a loop (DO until *IN99 = *ON, indicating end-of-file) to read equal (READE) records from BBORDR matching KyCoOrd (company/order).
   - For each record read (while *IN99 = *OFF):
     - Examines the sequence number (Seq#, positions 10-12) to determine record type:
       - Seq# = 000: Header record; writes to BBPROH (HEADER exception).
       - Seq# = 900-959: Other miscellaneous lines; writes to BBPROM (OtherMisc exception).
       - Seq# = 960: Miscellaneous line; writes to BBPROO (MISC960 exception).
       - Seq# = 961: Miscellaneous line; writes to BBPROI (MISC961 exception).
       - Seq# = 962: Miscellaneous line; writes to BBPROB (MISC962 exception).
       - Other Seq# (e.g., detail lines): Writes to BBPROD (DETAIL exception).
   - Writes full records (positions 1-256 and 257-512) to the respective output file without modification.

3. **Output to Work Files**:
   - For each exception output:
     - HEADER (BBPROH): Outputs BOHDR1 (1-256), BOHDR2 (257-512).
     - DETAIL (BBPROD): Outputs BDDTL1 (1-256), BDDTL2 (257-512).
     - MISC960 (BBPROO): Outputs BDORR1 (1-256), BDORR2 (257-512).
     - MISC961 (BBPROI): Outputs BDINR1 (1-256), BDINR2 (257-512).
     - MISC962 (BBPROB): Outputs BDBLR1 (1-256), BDBLR2 (257-512).
     - OtherMisc (BBPROM): Outputs BMBLR1 (1-256), BMBLR2 (257-512).
   - Adds records (EADD) to each file, preserving original data structure.

4. **Program Termination**:
   - After processing all records, sets *INLR (last record indicator) to *ON.
   - Exits, returning control to the caller (BB114SC).

The program is straightforward, acting as a data router that filters and distributes BBORDR records into specific work files based on sequence numbers, with no transformations or calculations.

### Business Rules

- **Record Type Segregation**: Uses Seq# to categorize records:
  - Seq# = 000: Header (order-level data, e.g., customer, dates, terms).
  - Seq# = 900-959: Miscellaneous remarks (e.g., bill of lading notes).
  - Seq# = 960-962: Specific miscellaneous lines (e.g., dispatch or freight-related).
  - Other Seq#: Detail lines (e.g., order items).
- **Data Preservation**: Copies records verbatim (1-512 bytes) to work files, ensuring no data loss or alteration for downstream processing (e.g., BB110 for printing).
- **Order-Specific Processing**: Processes only records matching the input company (Pco) and order number (Pordn), ensuring targeted confirmation generation.
- **File Usage**: Populates work files (BBPRO*) for temporary storage, used by BB110 for printing confirmations or proforma invoices. Files are group-specific (from BB114SC’s &P$FGRP).
- **No Validation**: Assumes valid input parameters and existing records; no error handling or validation visible (likely handled by caller BB114 or BB114SC).
- **Integration Context**: Part of the order confirmation flow (OCL → BB114 → BB114SC → BB114S → BB110), preparing structured data for printing.

### Tables/Files Used

- **Input File**:
  - BBORDR: Primary input file (512 bytes, disk, keyed starting at position 2, 11-byte alpha index); contains header, detail, and miscellaneous records with fields like BODEL, BOCO, BORDNO, BOCUST, BOSHIP, BOORPR, etc.

- **Output Files** (all add-only, 512 bytes, disk):
  - BBPROH: Stores header records (BOHDR1, BOHDR2).
  - BBPROD: Stores detail records (BDDTL1, BDDTL2).
  - BBPROO: Stores miscellaneous records for Seq# 960 (BDORR1, BDORR2).
  - BBPROI: Stores miscellaneous records for Seq# 961 (BDINR1, BDINR2).
  - BBPROB: Stores miscellaneous records for Seq# 962 (BDBLR1, BDBLR2).
  - BBPROM: Stores other miscellaneous records for Seq# 900-959 (BMBLR1, BMBLR2).

### External Programs Called

- None: The program does not call any external programs (no CALL opcodes). It is a self-contained data processing routine that reads from BBORDR and writes to the BBPRO* files for use by downstream programs like BB110.