The call stack provided includes a series of programs (`BB055.rpg36.txt`, `BB607.ocl36.txt`, `BB607.rpg.txt`, `BB6002.clp.txt`, `BB6002ASN.clp.txt`, `BB6002NEW.clp.txt`, `BB620.rpg36.txt`, `BB117.rpgle.txt`, `BB201.rpg36.txt`) that form part of the invoice posting workflow in the ARGLMS (Accounts Receivable and General Ledger Management System) on an IBM System/36 environment. These programs collectively handle various aspects of invoice processing, order updates, shipment history, and electronic delivery of invoices. Below, I identify the distinct use cases implemented by this call stack and provide a concise function requirements document for each, focusing on business requirements and key calculations.

---

### List of Use Cases

The call stack implements the following distinct use cases within the invoice posting workflow:

1. **Batch Status Management** (`BB055.rpg36.txt`):
   - Manages the status of invoice batches through stages (invoice entry, edit, forms, posting) by updating the batch control file (`BBIBCH`).
   - Determines readiness for posting based on selection criteria and record count.

2. **Shipment File Preparation** (`BB607.ocl36.txt`):
   - Prepares shipment and customer files by clearing and reorganizing them for subsequent processing.

3. **Shipment Record Deletion and History Creation** (`BB607.rpg.txt`):
   - Deletes processed shipment container records from `VISSHCN` and creates history records in `VISSHCH` after validating against the order master file (`BBORDR`).

4. **Invoice Transaction Deletion** (`BB620.rpg36.txt`):
   - Removes posted invoices from transaction files (`BBTRAN`, `BBTRANU`, `BBTRHS1`, `BBTRDS1`, `BBTRA1`, `BBTRTX`) and logs deletions to an audit file (`SA5SHA`).

5. **Order History Update for Duplicate Checking** (`BB117.rpgle.txt`):
   - Adds, updates, or deletes order history records in `BBOH` and `BBOHMS` to support duplicate order checking, called by `BB201`.

6. **Order and Customer File Update** (`BB201.rpg36.txt`):
   - Updates the order master file (`BBORDR`) and customer order file (`CUSORD`) based on transactions from `BBORTR`, manages supplemental and accessorial data, and deletes historical records for duplicate checking.

7. **Electronic Invoice Delivery** (`BB6002.clp.txt`, `BB6002ASN.clp.txt`, `BB6002NEW.clp.txt`):
   - Manages electronic delivery of invoices (and ASNs for `BB6002ASN`) via email and fax by handling output queues, using SpoolFlex to split and distribute spooled files as PDFs, and cleaning up temporary queues.

---

### Function Requirements Documents

Each use case is documented as a function that processes inputs to complete its tasks, focusing on business requirements and necessary calculations, presented concisely in a business-oriented format.

#### 1. Batch Status Management Function

<xaiArtifact artifact_id="6f974936-9807-4537-bb75-7009152f44fa" artifact_version_id="53341f37-c2ca-4a51-93d5-dad106ca9d7e" title="batch_status_management_requirements.md" contentType="text/markdown">

# Batch Status Management Function

**Purpose**: Updates the status of invoice batches to track progress through invoice entry, edit, forms, and posting stages in the ARGLMS system.

**Inputs**:
- Batch Number (`BATCH#`, 2-digit numeric): Identifies the batch.
- Program Mode (`PGM`, 1 character): `I` (invoice entry), `E` (edit), `F` (forms), `P` (post).
- Selection Criteria (`KYALSL`, 3 characters): `ALL` or company-specific code.
- Record Count (`RECCNT`, 8-digit numeric): Number of records in the batch.

**Outputs**:
- Updated `BBIBCH` (batch control file) with status fields and flags.

**Process Steps**:
1. Retrieve batch record from `BBIBCH` using `BATCH#`.
2. Based on `PGM`:
   - `I` (Invoice Entry): Clear status fields (positions 6–8) and update `RECCNT`.
   - `E` (Edit): Same as invoice entry.
   - `F` (Forms): Set forms flag (position 9 = `Y`), set ready-to-post flag (`RTOPST` = `Y` if `KYALSL = 'ALL'`, else `N`), update `RECCNT`.
   - `P` (Post): If `RECCNT = 0`, set delete flag (position 1 = `D`) and posted status (position 6 = `P`); else, clear status fields and update `RECCNT`.
   - Default: Clear status fields and update `RECCNT`.

**Business Rules**:
- Only mark batch as posted (`D`, `P`) if `RECCNT = 0` (all records processed).
- Set `RTOPST = Y` for forms if all orders selected (`KYALSL = 'ALL'`); else `N`.
- Clear status fields for non-posted stages to reset batch state.
- Assume valid input; skip processing if batch not found.

**Calculations**:
- No complex calculations; updates involve setting flags and copying `RECCNT`.

</xaiArtifact>

#### 2. Shipment File Preparation Function

<xaiArtifact artifact_id="f116ebfa-84f0-4579-b7ad-e7ba4d059bc8" artifact_version_id="18bd1d1b-cd8d-4acc-9d47-0dfbd1eac9cd" title="shipment_file_preparation_requirements.md" contentType="text/markdown">

# Shipment File Preparation Function

**Purpose**: Prepares shipment and customer files by clearing and reorganizing them for invoice posting in the ARGLMS system.

**Inputs**:
- Library Name (resolved at runtime, e.g., `?9?`): Specifies file location.

**Outputs**:
- Cleared `VISSHSM` file (empty).
- Reorganized `VISSHCN` file (optimized storage).

**Process Steps**:
1. Allocate files for shared read access: `VISSHCN`, `BBORDR`, `VISSHCH`, `ARCUFMX`.
2. Clear all records from `VISSHSM` (shipment summary file).
3. Reorganize `VISSHCN` (shipment container file) to remove deleted records and optimize access.

**Business Rules**:
- Clear `VISSHSM` to ensure no residual shipment summary data.
- Reorganize `VISSHCN` to maintain efficient storage and access.
- Use shared disposition (`DISP-SHR`) for concurrent read access.
- Resolve library names at runtime using `?9?` placeholders.

**Calculations**:
- None; performs file maintenance tasks only.

</xaiArtifact>

#### 3. Shipment Record Deletion and History Creation Function

<xaiArtifact artifact_id="9c82bada-48fd-4cf5-9be6-4419a01d78ef" artifact_version_id="42574b43-e81a-42c9-9b71-980538c6b079" title="shipment_record_deletion_requirements.md" contentType="text/markdown">

# Shipment Record Deletion and History Creation Function

**Purpose**: Deletes processed shipment container records and creates history records in the ARGLMS system.

**Inputs**:
- `VISSHCN` records with fields: `BCOORD` (company + order, 8 chars), `VDSEQ` (sequence, 3 digits), full record (249 bytes).

**Outputs**:
- Deleted records from `VISSHCN`.
- New history records in `VISSHCH` with system date and time.

**Process Steps**:
1. Generate system date (`SYSD8`, 8-digit `20YYMMDD`) and time (`TIME06`, 6-digit `HHMMSS`).
2. For each `VISSHCN` record:
   - Construct key (`KEY11` = `BCOORD` + `VDSEQ`).
   - Validate against `BBORDR` using `KEY11`.
   - If not found in `BBORDR`, delete `VISSHCN` record and write to `VISSHCH` with:
     - Entire record (1–249).
     - `SYSD8` (129–136).
     - `TIME06` (135–140).

**Business Rules**:
- Delete `VISSHCN` records only if not found in `BBORDR` (processed shipments).
- Archive deleted records in `VISSHCH` with system date and time.
- Assume valid input files and no errors during write/delete.

**Calculations**:
- System date: `SYSDAT * 10000.01` to `SYSYMD`, prepend `20` for `SYSD8`.
- System time: Extract `TIME06` from `TIMDAT` (12-digit timestamp).

</xaiArtifact>

#### 4. Invoice Transaction Deletion Function

<xaiArtifact artifact_id="37e4fab4-410f-4c99-9493-21c2fd3f786a" artifact_version_id="c9921469-974d-4bf9-b04c-0b589ff02728" title="invoice_transaction_deletion_requirements.md" contentType="text/markdown">

# Invoice Transaction Deletion Function

**Purpose**: Removes posted invoices from transaction files and logs deletions for auditing in the ARGLMS system.

**Inputs**:
- `BBTRAN` records with fields: `COORD` (order key, 8 chars), `BOINV#` (invoice number, 7 digits), `BOSHDT` (ship date, 6 digits), `BOLKEY` (transaction key, 21 bytes), `BOLKYH` (header key, 18 bytes).

**Outputs**:
- Deleted records from `BBTRAN`, `BBTRANU`, `BBTRHS1`, `BBTRDS1`, `BBTRA1`, `BBTRTX`.
- Audit records in `SA5SHA` with invoice and ship date.

**Process Steps**:
1. Read `BBTRAN` records (header, detail, marks, miscellaneous).
2. For each posted invoice:
   - Delete header from `BBTRANU` using `BOLKEY`.
   - Delete supplemental header from `BBTRHS1` using `BOLKYH`.
   - Delete supplemental detail from `BBTRDS1`.
   - Delete tax records from `BBTRTX`.
   - Process `BBTRA1` (accessorial/marks) using `XXKY23` (`BOLKYH` + `00000`):
     - Delete matching records; release non-matching.
   - Log to `SA5SHA`: record data, `BOINV#`, `BOSHDT`, program ID (`BB620`).
3. Verify no errors (`ERRCNT = 0`) before deletions.

**Business Rules**:
- Delete only posted invoices based on status or `BOINV#`.
- Log all deletions to `SA5SHA` for auditing.
- Handle supplemental and accessorial data (per `JB01`).
- Assume valid input; skip if errors detected.

**Calculations**:
- Construct `XXKY23` by concatenating `BOLKYH` and `00000` for `BBTRA1` access.

</xaiArtifact>

#### 5. Order History Update for Duplicate Checking Function

<xaiArtifact artifact_id="2c138b51-1ad0-46c8-829a-ca0dc0562fdf" artifact_version_id="d4115ede-9ad2-4bde-af51-1ed00a3afb42" title="order_history_update_requirements.md" contentType="text/markdown">

# Order History Update for Duplicate Checking Function

**Purpose**: Manages order history records to support duplicate order checking in the ARGLMS system.

**Inputs**:
- Parameters: `p$co` (company, 2 digits), `p$rdno` (order number, 6 digits), `p$rseq` (sequence, 3 digits), `p$cust` (customer, 6 digits), `p$ship` (ship-to, 3 digits), `p$pord` (purchase order, 15 chars), `p$ord8` (order date, 8 digits), `p$prod` (product, 4 chars), `p$cntr` (container, 4 chars), `p$qty` (quantity, 5 digits), `p$btch` (batch, 4 chars), `p$dpok` (duplicate OK, 1 char), `p$mode` (mode, 3 chars), `p$fgrp` (freight group, 1 char), `p$flag` (flag, 1 char).

**Outputs**:
- Updated or deleted records in `BBOH` and `BBOHMS`.

**Process Steps**:
1. Chain to `BBOH` using key (`p$co`, `p$rdno`, `p$rseq`).
2. Based on `p$mode`:
   - Add: Write new record to `BBOH` with input parameters.
   - Update: Update existing `BBOH` record.
   - Delete: Delete `BBOH` record if `p$mode` indicates deletion (e.g., code `D`).
3. For `BBOHMS` (marks supplemental), chain using key (`p$co`, `p$rdno`, `p$rseq`, ship/status dates), and add/update/delete similarly.
4. Include system timestamp in records.

**Business Rules**:
- Delete records only when `p$mode` indicates deletion and code is `D` (per `BB201 JB12`).
- Maintain `BBOH` for duplicate order checking.
- Update `BBOHMS` for supplemental marks data.
- Assume valid input parameters.

**Calculations**:
- Timestamp conversion: Extract hours/minutes/seconds from 14-digit timestamp.

</xaiArtifact>

#### 6. Order and Customer File Update Function

<xaiArtifact artifact_id="ef903978-eeca-4fb1-9dbc-13ff14c4ce05" artifact_version_id="b6a035ce-32df-4b54-9301-ca8c90d5cc50" title="order_customer_update_requirements.md" contentType="text/markdown">

# Order and Customer File Update Function

**Purpose**: Updates order and customer files with transaction data, manages supplemental data, and deletes history for duplicate checking in the ARGLMS system.

**Inputs**:
- `BBORTR` records with fields: `BOCUST` (customer, 6 digits), `BOSHIP` (ship-to, 3 digits), `RUSH` (rush flag), `OPCODE` (order process code), `INCOTM` (Incoterms), `IN11` (fluid code/IMS UOM), detail fields (`BDQTUM`, `BDPRWT`, `BMFAMT`), tax fields.
- Delete code (`D` for deletion).

**Outputs**:
- Updated `BBORDR`, `CUSORD`, `BBORCL`, `BBORHS1`, `BBORDS1`, `BBORHHS`, `BBORMHS`, `BBORDH`.
- Deleted `BBOH` records via `BB117`.

**Process Steps**:
1. Read `BBORTR` records.
2. Update `BBORDR` with transaction data (e.g., `BOCUST`, `BOSHIP`, `RUSH`, `OPCODE`, `INCOTM`, `IN11`, detail fields).
3. Update `CUSORD` with status (`XXSTAT`), company (`CN`), date (`YMD`); mark deleted (`D`) if applicable.
4. Update `BBORCL` with credit totals (`CRTOTL`) at level break.
5. Add/update/delete supplemental records in `BBORHS1`, `BBORDS1`, `BBORHHS`, `BBORMHS`, `BBORDH` with data (`BODTA1`, `BDDTA1`, `BMDTA1`, etc.) and metadata (`SYCYMD`, `SYSTIM`, `USERID`, `WSID`).
6. Call `BB117` to delete `BBOH` records with delete code `D`.

**Business Rules**:
- Update `BBORDR` and `CUSORD` with transaction data; mark deleted orders as `D`.
- Manage supplemental/accessorial data (per `JB06`, `JB09`, `MG19`).
- Delete `BBOH` records only when delete code is `D` (per `DC02`, `JB12`).
- Initialize fields (blank/zero-fill) and add tax/freight data (per `LT13`, `JB14`).
- Assume valid input; no error handling.

**Calculations**:
- None; updates involve field transfers and flag settings.

</xaiArtifact>

#### 7. Electronic Invoice Delivery Function

<xaiArtifact artifact_id="52471455-5b99-4d80-b20b-e872974f7e6f" artifact_version_id="34c96529-274d-4d8c-a7e4-91a577bee2c6" title="electronic_invoice_delivery_requirements.md" contentType="text/markdown">

# Electronic Invoice Delivery Function

**Purpose**: Manages electronic delivery of invoices (and ASNs for specific cases) via email and fax using SpoolFlex in the ARGLMS system.

**Inputs**:
- Batch Number (`P$BAT`, 2 chars).
- Freight Group (`P$FGRP`, 1 char).

**Outputs**:
- Cleared and deleted output queues (`ZBBSNDxxOQ`, `ZFAXxxOUTQ`, `ZTCEMOUTQ`, `INVCELDELV`, etc.).
- Spooled files distributed as PDFs via email/fax.

**Process Steps**:
1. Construct queue names:
   - Main: `P$FGRP + 'BBS' + P$BAT + 'OUTQ'` (e.g., `ZBBS01OUTQ`).
   - Send: `P$FGRP + 'BBSND' + P$BAT + 'OQ'` (e.g., `ZBBSND01OQ`).
   - Fax: `P$FGRP + 'FAX' + P$BAT + 'OUTQ'` (e.g., `ZFAX01OUTQ`).
   - Email: `P$FGRP + 'TCEMOUTQ'` (e.g., `ZTCEMOUTQ`).
2. Clear queues: `ZBBSNDxxOQ`, `ZFAXxxOUTQ`, `ZTCEMOUTQ`, `INVCELDELV`, and others.
3. Delete temporary queues: `ZBBSNDxxOQ`, `ZFAXxxOUTQ`.
4. Use SpoolFlex to split and distribute spooled files from `ZFAXxxOUTQ` (fax) and `ZTCEMOUTQ` (email).

**Business Rules**:
- Clear all specified queues to remove old spooled files.
- Delete temporary queues after processing.
- Use SpoolFlex for PDF distribution; `ZBBSNDxxOQ` is unused.
- Handle errors (e.g., queue not found) gracefully.
- Support `ZINV99OUTQ` and `ZBBS99OUTQ` from `BB5103`/`BB5107`.

**Calculations**:
- Queue name construction via string concatenation.

</xaiArtifact>

---

### Summary

The call stack implements seven use cases: batch status management, shipment file preparation, shipment record deletion/history creation, invoice transaction deletion, order history update, order/customer file update, and electronic invoice delivery. Each function processes specific inputs to update files, manage batch status, or distribute invoices, ensuring data integrity and workflow progression in the ARGLMS system. The requirements documents outline concise business rules and calculations, focusing on functional outcomes without screen interactions.