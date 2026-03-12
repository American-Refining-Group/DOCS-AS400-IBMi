The `BB6002.clp.txt` CLP (Control Language Program) is called by the main OCL program (likely `BB600.ocl36.txt`) within the invoice posting workflow on an IBM System/36 environment. Its primary function is to manage the electronic delivery of invoices via email and fax by handling output queues for spooled files, using SpoolFlex to split and distribute them as PDFs, and cleaning up temporary queues after processing. The program accepts parameters for batch and freight group, constructs queue names, clears existing queues, and deletes temporary queues. It is nearly identical to `BB6002ASN.clp.txt`, but focused on general invoice delivery rather than Viscosity ASNs. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the BB6002 CLP Program

The `BB6002` program manages output queues for invoice-related spooled files, leveraging SpoolFlex to split and distribute them as PDFs for email or fax delivery, and performs cleanup by clearing and deleting temporary queues.

1. **Parameter Declaration**:
   - Declares input parameters:
     - `&P$BAT` (2 characters): Batch number.
     - `&P$FGRP` (1 character): Freight group.
   - Declares variables for output queue names:
     - `&FILE` (10 characters): Main output queue (e.g., `ZBBSxxOUTQ`).
     - `&FILE1` (10 characters): Send output queue (e.g., `ZBBSNDxxOQ`).
     - `&FILE2` (10 characters): Fax output queue (e.g., `ZFAXxxOUTQ`).
     - `&FILE3` (9 characters): Email output queue (e.g., `ZTCEMOUTQ`).
     - `&FILE4` (9 characters): Additional output queue (name truncated, likely similar to `BB6002ASN`).
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
     - `INVCELDELV`: Fixed invoice delivery queue.
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
   - Relies on SpoolFlex to split and distribute spooled files as PDFs for email (`ZTCEMOUTQ`) or fax (`ZFAXxxOUTQ`).
   - Queues `ZINV99OUTQ` and `ZBBS99OUTQ` are created in `BB5103` and `BB5107` (called by `BB510`), but their processing is not detailed in this snippet.
   - `ZBBSNDxxOQ` is created but appears unused (`ZBBSND99OUTQ` noted as not used).

6. **Program Termination**:
   - Ends with `ENDPGM` after clearing and deleting queues.

---

### Business Rules

1. **Output Queue Management**:
   - Dynamically constructs output queue names using `&P$FGRP` and `&P$BAT`.
   - Clears spooled files from temporary queues (`&FILE1`, `&FILE2`, `&FILE3`, `&FILE4`, `INVCELDELV`) to prepare for new processing.
   - Deletes temporary queues (`&FILE1`, `&FILE2`) after processing to clean up.

2. **SpoolFlex Distribution**:
   - Uses SpoolFlex to split spooled files in `ZFAXxxOUTQ` for faxing and `ZTCEMOUTQ` for emailing as PDFs.
   - Relies on `ZINV99OUTQ` and `ZBBS99OUTQ` created by `BB5103` and `BB5107` for invoice spooled files.

3. **Error Handling**:
   - Monitors for common errors (`CPF2105`, `CPF3357`, `CPF2207`, `CPF2189`) to ensure robust queue clearing and deletion, ignoring cases where queues are missing or inaccessible.

4. **Queue Usage**:
   - `ZBBSNDxxOQ` is created but not used, indicating potential redundancy.
   - `INVCELDELV` is a fixed queue for invoice delivery, cleared alongside dynamic queues.

5. **Integration with ARGLMS**:
   - Part of the invoice posting workflow, managing spooled output for electronic invoice delivery (email and fax) in the ARGLMS system.

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

The `BB6002` CLP program does not explicitly call other programs in the provided snippet. However, it relies on:
- **SpoolFlex**: A third-party tool (not called directly but integrated for splitting and distributing spooled files as PDFs for email or fax).
- **BB5103** and **BB5107**: CLP programs (called by `BB510`) that create `ZINV99OUTQ` and `ZBBS99OUTQ`, respectively, which are referenced but not directly managed in this program.

---

### Summary

The `BB6002` CLP program, called by the main OCL (e.g., `BB600.ocl36.txt`), manages electronic invoice delivery by:
- Accepting batch (`&P$BAT`) and freight group (`&P$FGRP`) parameters.
- Constructing output queue names (e.g., `ZBBSxxOUTQ`, `ZFAXxxOUTQ`, `ZTCEMOUTQ`).
- Clearing spooled files from queues (`&FILE1`, `&FILE2`, `&FILE3`, `&FILE4`, `INVCELDELV`) using `CLROUTQ`.
- Deleting temporary queues (`&FILE1`, `&FILE2`) using `DLTOUTQ`.
- Supporting SpoolFlex for splitting and distributing spooled files as PDFs for email (`ZTCEMOUTQ`) or fax (`ZFAXxxOUTQ`).
- Handling errors robustly with `MONMSG`.
- Terminating after queue cleanup.

**Tables Used**: None (operates on output queues: `ZBBSxxOUTQ`, `ZBBSNDxxOQ`, `ZFAXxxOUTQ`, `ZTCEMOUTQ`, `INVCELDELV`, and an unspecified queue).
**External Programs Called**: None directly (relies on SpoolFlex and references `BB5103`, `BB5107` for queue creation).

This program ensures efficient management of spooled invoice documents in the ARGLMS system, supporting electronic delivery via email and fax and cleanup of temporary queues.