## Call Stack

| Order | Program     | Main Purpose                                                                 | Tables / Files Used                                                                 | Outputs / Side Effects |
|-------|-------------|-------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|------------------------|
| 1     | **BB001**   | Allows user to select or create the order batch to be posted                 | BBBTCH, BBBTCHX                                                                    | Selected batch number (?20?) |
| 2     | **#GSORT**  | Sorts the order batch so headers come before marks and details               | BBOR?20? (input) → BB198S (output)                                                 | Sorted work file BB198S |
| 3     | **BB198**   | Generates Credit Authorization Report and updates order status               | BBORTR, BBORTA, BICONT, ARCUST, GSCTUM, GSTABL, CUSORD, SHIPTO, GSPROD            | Credit Authorization Report (printer)<br>Updated CUSORD statuses |
| 4     | **BB201**   | Core posting: moves batch data into live open-order files                    | BBORTR, BBOTHS1, BBOTDS1, BBOTA1, BBTRTX, BBORDH/R/D/M/O/I/B, BBORHS1/ORDS1, BBORA1, BBORTX, BICONT, ARCUST, GSCTUM, GSTABL, CUSORD, BBORCL, GSPROD, BBOH, BBOHMS | Updated permanent order files (BBORDH, BBORDD, BBORDM, BBORDO/I/B, BBORTX, BBORA1)<br>Audit history files (BBORHHS, BBORDHS, BBORMHS)<br>Updated CUSORD<br>Called BB117 for duplicate history |
| 5     | **BB104B**  | Handles archiving of cancelled orders and reactivation of open orders        | BBORTR, BBTRTX, BBOTHS1, BBOTDS1, BBOTA1 (via OVRDBF)                             | Archived cancelled orders<br>Reactivated orders |
| 6     | **BB202**   | Identifies new credit holds and generates email notification spools          | BBORTR, BICONT, ARCUST, BBORCL, BBCSR, BBSLSM                                     | Spool files CREMAL (Credit/CSR) and SMEMAL (Salesman)<br>Resets hold flags in BBORCL |
| 7     | **BB202TC** | Post-processing of hold notification spools (routing/email)                  | (Uses spools created by BB202)                                                     | Emails or routes hold notifications |
| 8     | **BB215**   | Removes editing locks from all posted orders                                 | BBORTR, BBORDRH                                                                    | Cleared lock fields (BOLOCK, BOWSID) in BBORDRH |
| 9     | **BB005**   | Marks the batch as posted and updates control record                         | BBBTCH                                                                             | Updated BBBTCH with posted flags ('P'/'D') and final record count |
| 10    | **GSDELETE**| System utility that deletes temporary batch files                            | BBOR?20?, BBOX?20?                                                                 | Deleted temporary batch files |


## Programs and Tables Used


| Program      | BBBTCH          | BBOR?20? / BBORTR | BB198S     | BBORDH / BBORHS1          | BBORDD / BBORDS1          | BBORDM / BBORMHS | BBORDO/I/B | BBORTX     | BBORA1     | BBORCL     | CUSORD          | BBOH / BBOHMS | Audit History (HHS/DHS/MHS) | BICONT / ARCUST / Others | BBCSR / BBSLSM | CREMAL / SMEMAL |
|--------------|-----------------|-------------------|------------|---------------------------|---------------------------|------------------|------------|------------|------------|------------|-----------------|---------------|-----------------------------|--------------------------|----------------|-----------------|
| **BB001**    | Read only      | -                 | -          | -                         | -                         | -                | -          | -          | -          | -          | -               | -             | -                           | Read only               | -              | -               |
| **#GSORT**   | -              | Read              | **Create** (sorted output) | -                         | -                         | -                | -          | -          | -          | -          | -               | -             | -                           | -                       | -              | -               |
| **BB198**    | -              | Read              | Read       | -                         | -                         | -                | -          | -          | -          | -          | **Update** (status) | -             | -                           | Read only               | -              | -               |
| **BB201**    | -              | Read              | -          | **Create/Update**         | **Create/Update**         | **Create/Update**| **Create/Update** | **Create/Update** | **Delete/Create/Update** | **Update** (reset) | **Update** (status) | **Update/Delete** (via BB117) | **Create** (snapshots)     | Read only               | -              | -               |
| **BB104B**   | -              | Read              | -          | -                         | -                         | -                | -          | -          | -          | -          | -               | -             | -                           | -                       | -              | -               |
| **BB202**    | -              | Read              | -          | -                         | -                         | -                | -          | -          | -          | **Update** (reset hold flags) | -               | -             | -                           | Read only               | Read only      | **Create** (spools) |
| **BB202TC**  | -              | -                 | -          | -                         | -                         | -                | -          | -          | -          | -          | -               | -             | -                           | -                       | -              | **Process** (email/route) |
| **BB215**    | -              | Read              | -          | **Update** (clear locks)  | -                         | -                | -          | -          | -          | -          | -               | -             | -                           | -                       | -              | -               |
| **BB005**    | **Update** (posted flags + count) | -                 | -          | -                         | -                         | -                | -          | -          | -          | -          | -               | -             | -                           | -                       | -              | -               |
| **GSDELETE** | -              | **Delete**        | -          | -                         | -                         | -                | -          | -          | -          | -          | -               | -             | -                           | -                       | -              | -               |

### Legend / Notes
- **Create/Update** = Adds new records or modifies existing ones.
- **Delete** = Physically removes records.
- **Update (reset)** = Clears or changes specific flags (e.g., hold status, locks).
- **Read only** = The program reads the table but does not modify it.
- **Via BB117** = BB201 calls BB117 internally to handle updates to BBOH.
- **Others** = Includes GSPROD, GSCTUM, SHIPTO, GSTABL, etc. — all read-only across programs.
- Empty cells ("-") mean the program does not access that table.

**Use Cases Implemented**  
The entire call stack implements **one primary use case** (with minor batch-type variations):

**Use Case: Post Order Batch**  
A batch-oriented function that accepts a batch identifier and library prefix as input, performs full credit review, posts orders to the live system, handles cancellations/reactivations, sends hold notifications, releases locks, marks the batch complete, and cleans up temporary data — all without user screens or interactive input.

---

**Function Requirements Document**  
**FRD-ORD-POST: Order Entry Batch Posting Function**

**1. Purpose**  
Accept a prepared order batch and move it safely into the live open-order system while enforcing credit controls, generating required notifications, maintaining audit history, and releasing the batch for downstream processing (invoicing, shipping, etc.).

**2. Inputs**  
- Batch number (`?20?`)  
- Library prefix (`?9?`)  
- Batch type flag (`?13?`: blank = normal orders, `PP` = Viscosity ASN, `PM` = Product Moves)  
- Pre-populated temporary batch files created by order entry

**3. High-Level Process Steps (in execution order)**

1. **Validate & Select Batch**  
   Confirm batch exists and contains orders; obtain user confirmation to proceed.

2. **Credit Authorization**  
   Sort batch and generate Credit Authorization Report; update each order’s lifecycle status (`CUSORD`).

3. **Core Posting**  
   Merge temporary batch data into permanent open-order files (`BBORDH`, `BBORDD`, etc.); apply add/update/delete; create full audit history snapshots; call duplicate-order history maintenance; remove stale freight misc lines when freight calculation is disabled.

4. **Handle Cancellations & Reactivations**  
   Archive cancelled orders and reactivate open orders as required.

5. **Generate Hold Notifications** (conditional)  
   Identify *new* credit holds; produce email-ready spools for Credit/CSR team and responsible salesperson.

6. **Release Locks**  
   Clear editing lock fields on all posted order headers.

7. **Mark Batch Complete & Cleanup**  
   Update batch control record with posted flags and final record count; delete temporary batch files.

**4. Business Rules**  
- Only non-deleted records are processed.  
- Returns (`BOTYPE = 'R'`) negate quantities for correct value and inventory impact.  
- No-charge lines post with zero value.  
- **New-hold-only notifications**: Email is sent only when an order changes from “not on hold” to “on hold” in this posting (prevents duplicate alerts).  
- **Freight rule**: If freight calculation flag = 'N', automatically delete freight-related misc lines (seq 943 & 950).  
- **Accessorials**: Completely replaced on every post (old records deleted, new records copied).  
- **Tax records**: Created only when tax-exempt numbers or amounts exist.  
- **Locks**: Cleared for every order in the batch after posting succeeds.  
- Batch types (`PP`/`PM`) affect only report banners and skip certain notification steps; core posting logic is identical.  
- Full audit trail: every change writes a timestamped snapshot with user/WSID.

**5. Key Calculations**  
- **UOM conversion** (pricing quantity):  
  `QTY = BDQTY × or ÷ GSCTUM conversion factor` (multiply if operand='M', divide if 'D').  
  Extended value = `QTY × BDPRCE`.  
- **Order totals**: Sum of line extended values + misc charges + freight.  
- **Days until pickup** (for hold notifications):  
  `DAYS = (Requested Date Julian) – (Current Date Julian)`.  
- **Credit hold detection**: `BLOVCL = 'Y' AND BLAUIN = blank AND previous hold flag (BLOVSV) was not 'Y'`.

**6. Outputs**  
- Updated live order master files (headers, details, misc, marks, taxes, accessorials).  
- Audit history records.  
- Updated customer order status.  
- Updated batch control record (posted flags + final count).  
- Optional email spools for new credit holds (`CREMAL` + `SMEMAL`).  
- Deleted temporary batch files.

**7. Success Criteria**  
- All orders in the batch are posted or archived.  
- Credit holds trigger exactly one notification set.  
- Orders are unlocked and batch is marked complete.  
- No temporary files remain.

This function is fully batch-driven and designed for unattended execution after order entry.