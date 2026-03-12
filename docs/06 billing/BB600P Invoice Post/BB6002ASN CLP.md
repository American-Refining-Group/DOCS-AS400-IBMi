The `BB6002ASN.clp.txt` CLP (Control Language Program) is called by the main OCL program (likely `BB600.ocl36.txt`) within the invoice posting workflow on an IBM System/36 environment, specifically for handling Viscosity Advanced Shipping Notices (ASNs). Its primary function is to manage output queues for spooled files related to invoice processing, split and distribute them as PDFs using SpoolFlex for email or fax delivery, and clean up temporary output queues. The program accepts parameters for batch and freight group, constructs queue names, clears existing queues, and deletes them after processing. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the BB6002ASN CLP Program

The `BB6002ASN` program manages output queues for invoice-related spooled files, leveraging SpoolFlex to split and distribute them as PDFs, and performs cleanup by clearing and deleting temporary queues.

1. **Parameter Declaration**:
   - Declares input parameters:
     - `&P$BAT` (2 characters): Batch number.
     - `&P$FGRP` (1 character): Freight group.
   - Declares variables for output queue names:
     - `&FILE` (10 characters): Main output queue (e.g., `ZBBSxxOUTQ`).
     - `&FILE1` (10 characters): Send output queue (e.g., `ZBBSNDxxOQ`).
     - `&FILE2` (10 characters): Fax output queue (e.g., `ZFAXxxOUTQ`).
     - `&FILE3` (9 characters): Email output queue (e.g., `ZTCEMOUTQ`).
     - `&FILE4` (9 characters): Additional output queue (not fully specified due to truncation).
     - `&QDTAQ` (1024 characters): Data queue for SpoolFlex processing (not used in provided snippet).

2. **Construct Output Queue Names**:
   - Builds queue names dynamically using `CHGVAR`:
     - `&FILE = &P$FGRP + 'BBS' + &P$BAT + 'OUTQ'` (e.g., `ZBBS01OUTQ` for freight group `Z`, batch `01`).
     - `&FILE1 = &P$FGRP + 'BBSND' + &P$BAT + 'OQ'` (e.g., `ZBBSND01OQ`).
     - `&FILE2 = &P$FGRP + 'FAX' + &P$BAT + 'OUTQ'` (e.g., `ZFAX01OUTQ`).
     - `&FILE3 = &P$FGRP + 'TCEMOUTQ'` (e.g., `ZTCEMOUTQ`).
     - `&FILE4 = &P$FGRP + ...` (incomplete due to truncation, likely another queue name).

3. **Clear Output Queues**:
   - Uses `CLROUTQ` to clear spooled files from the following queues in `QUSRSYS`:
     - `&FILE1` (e.g., `ZBBSNDxxOQ`).
     - `&FILE2` (e.g., `ZFAXxxOUTQ`).
     - `&FILE3` (e.g., `ZTCEMOUTQ`).
     - `&FILE4` (unspecified queue).
     - `INVCELDELV`: A fixed invoice delivery queue.
   - Monitors for error messages (`MONMSG`) to handle cases where queues are empty or do not exist:
     - `CPF2105`: Object not found.
     - `CPF3357`: Output queue not found.
     - `CPF2207`: Not authorized.
     - `CPF2189`: Output queue not found in library.

4. **Delete Output Queues**:
   - Uses `DLTOUTQ` to delete the following queues from `QUSRSYS`:
     - `&FILE1` (e.g., `ZBBSNDxxOQ`).
     - `&FILE2` (e.g., `ZFAXxxOUTQ`).
   - Monitors for the same error messages (`CPF2105`, `CPF3357`, `CPF2207`, `CPF2189`) to handle cases where queues do not exist.

5. **SpoolFlex Integration**:
   - The program relies on SpoolFlex (a third-party tool) to split and distribute spooled files as PDFs for email (`ZTCEMOUTQ`) or fax (`ZFAXxxOUTQ`).
   - Queues `ZINV99OUTQ` and `ZBBS99OUTQ` are created in `BB5103` and `BB5107` (called by `BB510`), but their processing is not detailed in this snippet.
   - `ZBBSNDxxOQ` is created but appears unused (`ZBBSND99OUTQ` noted as not used).

6. **Program Termination**:
   - Ends with `ENDPGM` after clearing and deleting queues.

---

### Business Rules

1. **Output Queue Management**:
   - Dynamically constructs output queue names based on freight group (`&P$FGRP`) and batch number (`&P$BAT`).
   - Clears spooled files from temporary queues (`&FILE1`, `&FILE2`, `&FILE3`, `&FILE4`, `INVCELDELV`) to prepare for new processing.
   - Deletes temporary queues (`&FILE1`, `&FILE2`) after processing to clean up.

2. **SpoolFlex Distribution**:
   - Uses SpoolFlex to split spooled files in `ZFAXxxOUTQ` for faxing and `ZTCEMOUTQ` for emailing as PDFs.
   - Relies on queues created by `BB5103` (`ZINV99OUTQ`) and `BB5107` (`ZBBS99OUTQ`) for invoice-related spooled files.

3. **Error Handling**:
   - Monitors for common errors (`CPF2105`, `CPF3357`, `CPF2207`, `CPF2189`) to ensure robust queue clearing and deletion, ignoring cases where queues are missing or inaccessible.

4. **Queue Usage**:
   - `ZBBSNDxxOQ` is created but not used, indicating potential redundancy or future expansion.
   - `INVCELDELV` is a fixed queue for invoice delivery, cleared alongside dynamic queues.

5. **Integration with ARGLMS**:
   - Part of the invoice posting workflow, managing spooled output for ASNs to support fax and email distribution of invoice documents.

---

### Tables (Files) Used

The program does not directly interact with database files (tables) but manages output queues for spooled files. The relevant queues are:

1. **&FILE (e.g., ZBBSxxOUTQ)**:
   - **Description**: Main output queue for invoice spooled files.
   - **Purpose**: Holds spooled files from `BB5103` or `BB5107` for processing.

2. **&FILE1 (e.g., ZBBSNDxxOQ)**:
   - **Description**: Send output queue (unused).
   - **Purpose**: Created but not utilized in the current process.
   - **Usage**: Cleared (`CLROUTQ`) and deleted (`DLTOUTQ`).

3. **&FILE2 (e.g., ZFAXxxOUTQ)**:
   - **Description**: Fax output queue.
   - **Purpose**: Holds spooled files for fax distribution via SpoolFlex.
   - **Usage**: Cleared (`CLROUTQ`) and deleted (`DLTOUTQ`).

4. **&FILE3 (e.g., ZTCEMOUTQ)**:
   - **Description**: Email output queue.
   - **Purpose**: Holds spooled files for email distribution as PDFs via SpoolFlex.
   - **Usage**: Cleared (`CLROUTQ`).

5. **&FILE4 (unspecified)**:
   - **Description**: Additional output queue (name truncated).
   - **Purpose**: Likely another temporary queue for spooled files.
   - **Usage**: Cleared (`CLROUTQ`).

6. **INVCELDELV**:
   - **Description**: Fixed invoice delivery output queue.
   - **Purpose**: Holds spooled files for invoice delivery.
   - **Usage**: Cleared (`CLROUTQ`).

---

### External Programs Called

The `BB6002ASN` CLP program does not explicitly call other programs in the provided snippet. However, it relies on:
- **SpoolFlex**: A third-party tool (not a program called directly but integrated for splitting and distributing spooled files as PDFs for email or fax).
- **BB5103** and **BB5107**: CLP programs (called by `BB510`) that create `ZINV99OUTQ` and `ZBBS99OUTQ`, respectively, which are referenced but not directly managed in this program.

---

### Summary

The `BB6002ASN` CLP program, called by the main OCL (e.g., `BB600.ocl36.txt`), manages invoice-related spooled files by:
- Accepting batch (`&P$BAT`) and freight group (`&P$FGRP`) parameters.
- Constructing output queue names (e.g., `ZBBSxxOUTQ`, `ZFAXxxOUTQ`, `ZTCEMOUTQ`).
- Clearing spooled files from queues (`&FILE1`, `&FILE2`, `&FILE3`, `&FILE4`, `INVCELDELV`) using `CLROUTQ`.
- Deleting temporary queues (`&FILE1`, `&FILE2`) using `DLTOUTQ`.
- Supporting SpoolFlex for splitting and distributing spooled files as PDFs for email (`ZTCEMOUTQ`) or fax (`ZFAXxxOUTQ`).
- Handling errors robustly with `MONMSG`.
- Terminating after queue cleanup.

**Tables Used**: None (operates on output queues: `ZBBSxxOUTQ`, `ZBBSNDxxOQ`, `ZFAXxxOUTQ`, `ZTCEMOUTQ`, `INVCELDELV`, and an unspecified queue).
**External Programs Called**: None directly (relies on SpoolFlex and references `BB5103`, `BB5107` for queue creation).

This program ensures efficient management of spooled invoice documents in the ARGLMS system, supporting PDF distribution for ASNs and cleanup of temporary queues.