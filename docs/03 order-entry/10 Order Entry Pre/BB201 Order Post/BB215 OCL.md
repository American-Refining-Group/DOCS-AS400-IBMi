**BB215 – Overview**  
This is a very simple, focused **cleanup program** that runs near the end of the main `BB201` order-posting OCL procedure.  

Its sole purpose is to **remove order lockout fields** from the permanent order header file (`BBORDH`) for every order that was just processed in the current batch.  

These lockout fields prevent multiple users from editing the same order simultaneously. After posting is complete, the locks must be cleared so the orders become available again for inquiry, editing, or new entry.

---

### Process Steps (Combined OCL + RPG)

**OCL (BB215 OCL)**

1. Calls system utility `GSY2K` (Y2K handling / environment setup).
2. Loads the RPG program **BB215**.
3. Passes two files:
   - `BBORTR` → the temporary order batch that was just posted (input).
   - `BBORDRH` → the permanent open order header file (update).
4. Runs the program.

**RPG Program BB215** (very short and efficient)

1. **Initialization (ONCE routine)**  
   - Clears two work fields: `LOCK` (1 byte) and `WSID` (2 bytes – workstation ID).

2. **Main Processing (RPG Cycle)**  
   - Reads every **header record** from the temporary batch file `BBORTR` (primary file, indicator 01).  
   - Uses the order key (`BOCOS` = company + order number) to **CHAIN** to the matching record in the permanent file `BBORDRH`.

3. **Update Logic (Exception Output)**  
   - If the order exists in `BBORDRH` (`N90` = not "record not found"), it performs an **UPDATE**:
     - Sets the lockout code field (position 155) to blank.
     - Sets the lockout workstation ID field (positions 156-157) to blank.
   - No action is taken if the order is not found in the permanent file (rare, but safe).

4. **End of Job**  
   - All processed orders in the batch now have their locks removed.  
   - Files close automatically.

---

### Business Rules

- **Lock Removal is unconditional** for every header in the batch — even if the order was marked for deletion (`BODEL = 'D'`). The lock must still be cleared.
- **Only affects orders that were in this posting batch** — other open orders remain untouched.
- **Minimal processing** — no calculations, no totals, no status updates, no history writes. It is purely a housekeeping step.
- **Safety** — Uses `CHAIN` + conditional update so it never creates new records or corrupts data.
- **Timing** — Runs **after** the main posting (`BB201`) and after any cancel/reactivation logic (`BB104B`), ensuring locks are released only when the entire post is successful.

This step is critical for usability: without it, every posted order would remain locked forever, blocking further work by users.

---

### Tables / Files Used

| File       | Type      | Purpose                                      | Usage          | Updated? |
|------------|-----------|----------------------------------------------|----------------|----------|
| **BBORTR** | IP (Primary) | Temporary order batch headers (the batch just posted) | Input / Read   | No       |
| **BBORDRH**| UC        | Permanent Open Order Header master           | Chain + Update | **Yes**  |

- **No other files** are used.

---

### External Programs Called

**None.**

BB215 is completely self-contained. It does not call any other programs, sub-procedures, or utilities beyond the standard RPG cycle.

---

### Summary

`BB215` is a lightweight **post-posting cleanup routine**.  
It simply walks through the batch that was just posted and clears the two lockout fields (`BOLOCK` and `BOWSID`) in every corresponding order header in `BBORDH`.  

This ensures orders become editable/inquirable again immediately after posting completes, completing the full order-entry → credit-check → post → unlock cycle managed by the main OCL.  

The program is intentionally minimal and fast, reflecting its narrow, critical housekeeping role.