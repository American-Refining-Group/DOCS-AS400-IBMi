### Overview of AR781A

**AR781A** is the **final program** in the three-program batch chain (AR781P → AR781 → **AR781A**). It is an **RPG II** program designed to run after AR781 has populated the **CREDREL** (detail) and **CREDRLH** (header) files with customers and orders that are **eligible for credit hold release**.

The program appears to have a dual purpose in its naming and comments:
- The header says "CUSTOMER CREDIT LIMIT AUTHORIZATION INQUIRY", but the actual logic is **batch update** rather than inquiry.
- Its primary function is to **automatically release** orders that were identified as eligible in AR781 by updating the open order lines in **BBORCL** and adjusting the available credit in **CREDRLH**.

This program performs an **automated credit release** for orders that pass the eligibility criteria, without requiring manual intervention.

### Process Steps (Execution Flow)

1. **One-time initialization (ONCE block)**:
   - Executes only on the first record.
   - Captures the current time and formats it into fields (`TIME12`, `SYDATE`, `TIMEOF`).
   - Sets a flag (`ONCE = 1`) to prevent re-execution.

2. **Main processing loop**:
   - Reads **CREDREL** sequentially (input primary file, indicator 01 on each record).
   - Each record in CREDREL represents **one order line previously determined to be eligible for release** by AR781.

3. **Chain to the header record in CREDRLH**:
   - Uses the group key (`CDKEY` = company + group customer) to chain to the corresponding record in **CREDRLH**.
   - If found (indicator 99 off):
     - Compares the order amount (`CDAMT`) to the current available credit in the header (`CHAVCL`).
     - If `CHAVCL >= CDAMT` (indicator 50 on):
       - Calculates new available credit: `NWAVCL = CHAVCL - CDAMT`
       - Outputs an **UPDATE** exception record to **CREDRLH** to reduce the available credit by the order amount.
     - If not enough available credit (50 off), skips the release and goes to end.

4. **Chain to the actual open order line in BBORCL**:
   - Uses `CDORKY` (composite key: company + order number + batch?) to chain to the corresponding record in **BBORCL**.
   - If found (indicator 98 off):
     - Outputs an **ORCLUP** exception record to **BBORCL** with update fields:
       - Hard-coded values: position 39–41 = 'ARG', 47–53 = 'OTTO    '
       - These appear to be **authorization initials or user ID** indicating automated release (possibly "ARG" = auto-release group, "OTTO" = system user or code for Otto?).
     - This effectively **releases the credit hold** on the order line by populating the authorization fields (`BLAUIN` and/or `BLUSID`).

5. **Print confirmation**:
   - Outputs a line to the **REPORT** printer file showing:
     - Delete code, company, customer, order number
   - This creates a simple audit trail of which orders were automatically released.

6. **End of record processing**:
   - Loops to the next record in CREDREL.

### Business Rules Implemented

1. **Automated release only for pre-approved eligible orders**:
   - Only processes orders already flagged as eligible by AR781 (i.e., over limit, not authorized, not a problem customer, and enough available credit at group level).

2. **Re-check available credit before release**:
   - Even though AR781 already checked, AR781A **re-validates** that the group still has sufficient available credit (`CHAVCL >= order amount`).
   - This prevents race conditions or changes since AR781 ran.

3. **Sequential processing with update**:
   - As each order is released, the group's **available credit is reduced** in CREDRLH to reflect the commitment.
   - This ensures subsequent orders in the same run see the updated available balance.

4. **Authorization marking**:
   - Released orders are marked with a system identifier ('ARG' / 'OTTO') in the authorization fields of BBORCL.
   - This distinguishes **auto-released** orders from those manually authorized by credit staff.

5. **Audit trail**:
   - Prints a simple log of released orders.

### Tables/Files Used

| File       | Type   | Description                                              | Access Mode              | Key Usage |
|------------|--------|----------------------------------------------------------|--------------------------|-----------|
| **CREDREL**    | Input (Primary) | Detail file from AR781 – eligible order lines to process | Sequential read (EXTK?)  | Driver file |
| **CREDRLH**    | Update | Header/summary file from AR781 – group credit totals     | Random chain + update    | Group key |
| **BBORCL**     | Update | Open order lines (backlog) file                          | Random chain + update    | Order key (CDORKY) |
| **REPORT**     | Output | Printer file – audit report of auto-released orders      | EXCPT                    | Simple log |

### External Programs Called

- **None**.
  - There are no `CALL` statements.
  - All operations are internal: reading, chaining, updating files, and printing.

### Summary and Role in the Overall Process

**AR781A** is the **automated execution step** in a three-phase credit release workflow:

1. **AR781P**: Pre-filters to find group customers with pending over-limit orders.
2. **AR781**: Performs detailed group-level credit analysis and identifies **specifically which orders are safe to release** → populates CREDREL/CREDRLH.
3. **AR781A**: **Automatically releases** those safe orders by:
   - Updating the order line in BBORCL to mark it as authorized.
   - Reducing available credit in the header.
   - Producing an audit report.

This design allows credit managers to:
- First review the full report from AR781 (manual decision).
- Then optionally run AR781A to auto-release the low-risk ones.

Or run the entire chain (as in the OCL) for fully automated release of clearly eligible orders.

The use of hard-coded 'ARG'/'OTTO' suggests this was customized for a specific company (possibly named Arg or run by someone named Otto).