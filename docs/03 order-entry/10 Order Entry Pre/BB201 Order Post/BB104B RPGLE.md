### Process Steps of the RPG Program (BB104B.rpgle.txt)

This RPGLE program (BB104B) handles the cancellation or reactivation of open orders by archiving cancelled orders to history files (prefixed "bbcn*") and optionally deleting them from open order files (prefixed "bbord*"). It processes transaction records from a batch file (bbortr) to determine actions. The program is parameterized: p$fgrp ('G' or 'Z' for file group/library overrides) and p$flag (e.g., '0' for delete mode). It uses data structures to copy records verbatim during archiving. The process is driven by reading the primary input (bbortr) in a loop until EOF, with subroutines for checking archival needs, performing archives, and deletions.

High-level process steps:

1. **Initialization (*INZSR Subroutine)**:
   - Receives entry parameters: p$fgrp (file group for overrides) and p$flag (return flag, controls delete behavior).
   - Executes OVRDBF commands via QCMDEXC for file overrides on 11 tables (e.g., bbordh, bbcnh) if p$fgrp is 'G' or 'Z', setting SHARE(*NO).
   - Opens all files (bbordh for input, bbcn* for update/add).
   - Defines keys (e.g., klordh for company + order#) and data structures (wkcnh, etc.) mirroring file formats with prefixes (h_, ds_, etc.).

2. **Main Processing Loop (Reading bbortr)**:
   - Reads records from bbortr (primary input, NS 01 for headers).
   - Parses record type using indicators and field values (e.g., NS 01: header; NS 02: detail; NS 03: misc; NS 04: order marks; NS 05: invoice marks; NS 06: BOL marks).
   - For headers (01): 
     - Extracts keys (toco, toordn, tocoor, toky11).
     - Calls subroutine chkordarc to check if archival is needed (sets w$arc = *ON if cancelled in bbordh but not in transaction).
     - If w$arc = *ON: Clears bbcnh record, copies transaction data (t$orh) to wkcnh, sets h_bodel = 'A' (active archive), writes to bbcnh.
     - Calls arcnonprm to archive supplemental/non-primary records (taxes, headers/details/accessorials from bbtrtx, bboths1, bbotds1, bbota1 to corresponding bbcn* files).
   - For details (02): If w$arc = *ON, clears bbcnd, copies t$ord to wkcnd, writes to bbcnd.
   - For misc (03): If w$arc = *ON, clears bbcnm, copies t$orm to wkcnm, writes to bbcnm.
   - For order marks (04): If w$arc = *ON, clears bbcno, copies t$oro to wkcno, writes to bbcno.
   - For invoice marks (05): If w$arc = *ON, clears bbcni, copies t$ori to wkcni, writes to bbcni.
   - For BOL marks (06): If w$arc = *ON, clears bbcnb, copies t$orb to wkcnb, writes to bbcnb.

3. **Archival Check Subroutine (chkordarc)**:
   - Chains to bbordh using klordh (company + order#).
   - If found and BOCANC = 'Y' (cancelled) but todel != 'D' (not deleted in transaction), sets w$arc = *ON.

4. **Non-Primary Archival Subroutine (arcnonprm)**:
   - For taxes (bbtrtx): SETLL/READ loop by toky11, copies matching records (t$ortx) to wkcntx, writes to bbcntx.
   - For supplemental headers (bboths1): SETLL/READ loop by toky11, copies (t$orhs) to wkcnhs, writes to bbcnhs.
   - For supplemental details (bbotds1): SETLL/READ loop by k$hkey (company + order#), copies (t$ords) to wkcnds, writes to bbcnds.
   - For accessorials (bbota1): SETLL/READ loop by k$hkey, copies (t$ora) to wkcna, writes to bbcna.

5. **Deletion from Open Orders (If p$flag = '0')**:
   - Calls delopenord subroutine.
   - Deletes header from bbordh (chain/delete).
   - Deletes details from bbordd (SETLL/READE loop, delete).
   - Deletes misc from bbordm (SETLL/READE loop, delete).
   - Deletes marks from bbordo, bbordi, bbordb (SETLL/READE loop, delete each).
   - Deletes taxes from bbortx (SETLL/READE loop, delete).
   - Deletes supplementals from bborhs1 (SETLL/READ loop, delete by key match).
   - Deletes from bbords1, bbora1 similarly (SETLL/READ loop, delete by key match).

6. **End of Processing**:
   - Continues until EOF on bbortr.
   - No explicit close, but files are opened with USROPN, so closed at program end.

### Business Rules

- **Archival Trigger**: Archives only if the order exists in bbordh, is marked cancelled (BOCANC = 'Y'), and is not a transaction delete (todel != 'D'). Sets archive flag (w$arc) for the order.
- **Record Copying**: Uses external data structures (EXTNAME with PREFIX) to copy entire records verbatim from transaction/open files to archive files, with minimal changes (e.g., h_bodel = 'A' for active archive).
- **Deletion Mode**: Controlled by p$flag = '0' (likely from OCL parameter '0'). Deletes all related records from open order files after archiving, ensuring no remnants.
- **File Overrides**: Applies OVRDBF SHARE(*NO) to 11 files if p$fgrp is 'G' or 'Z', preventing shared access during updates.
- **Key Matching**: All operations use company + order# (tocoor) as primary key for chaining/SETLL/READ. Supplemental files use partial keys (e.g., k$hkey for headers).
- **No Validation/Errors**: Minimal error handling (e.g., *IN99 for EOF/not found). Assumes valid input; no business validations beyond existence/cancel flag.
- **Archival Integrity**: Archives all related records (headers, details, misc, marks, taxes, supplementals) atomically per order.
- **Reactivation**: Implied but not explicit; if not cancelled, no action (skips archival/deletion).

### Tables Used (Files)

Files are disk-based, with keyed access (AIDISK). Prefixes distinguish open (bbord*/bbor*/bbot*) vs. archived (bbcn*) files.

- **bbortr**: Input primary (IP), keyed (11AI), transaction batch (headers/details/marks).
- **bbtrtx**: Input (IF), keyed (11AI), transaction taxes.
- **bboths1**: Input (IF), keyed (8AI), supplemental transaction headers.
- **bbotds1**: Input (IF), keyed (11AI), supplemental transaction details.
- **bbota1**: Input (IF), keyed (13AI), transaction accessorials.
- **bbordh**: Input (IF E), open order headers (chained for cancel check).
- **bbcnh**: Update/Add (UF A E), archived order headers.
- **bbcnhs**: Update/Add (UF A E), archived supplemental headers.
- **bbcnd**: Update/Add (UF A E), archived order details.
- **bbcnds**: Update/Add (UF A E), archived supplemental details.
- **bbcnm**: Update/Add (UF A E), archived misc lines.
- **bbcno**: Update/Add (UF A E), archived order entry marks.
- **bbcnb**: Update/Add (UF A E), archived BOL marks.
- **bbcni**: Update/Add (UF A E), archived invoice marks.
- **bbcna**: Update/Add (UF A E), archived accessorials.
- **bbcntx**: Update/Add (UF A E), archived tax overrides.

Note: Deletion logic references additional files not in F-specs (e.g., bbordd, bbordm, bbordo, bbordi, bbordb, bbortx, bborhs1, bbords1, bbora1), but implied as overridden/aliased.

### External Programs Called

- **QCMDEXC**: System program called to execute OVRDBF commands for file overrides. Parameters: command string (dbov##, 80 chars) and length (dbol##, 15.5). No other external calls.